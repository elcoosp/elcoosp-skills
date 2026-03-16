# Infrastructure-as-Code Patterns

EL uses these patterns when writing Terraform and Kubernetes manifests.
All infrastructure changes must be in IaC — no manual console modifications.

---

## Terraform Conventions

### Module Structure

```
infrastructure/terraform/
├── main.tf           # Root module — orchestrates child modules
├── variables.tf      # Input variable declarations
├── outputs.tf        # Output value declarations
├── backend.tf        # Remote state config (S3 + DynamoDB lock)
├── providers.tf      # Provider versions pinned explicitly
└── modules/
    ├── networking/   # VPC, subnets, security groups
    ├── compute/      # ECS/EKS, Lambda
    ├── data/         # RDS, ElastiCache, S3
    └── observability/ # CloudWatch, alerting
```

### State Management (Non-Negotiable)

Remote state must be configured from the first `terraform init`. Local state is
never acceptable for anything beyond local development.

```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "product/env/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"    # Pin to major version; allow minor updates
    }
  }
  required_version = ">= 1.6.0"
}
```

### Secrets — Never in Terraform State

Terraform state is stored in S3 and may be readable by anyone with S3 access.
Never pass secrets as variables to Terraform. Reference secrets by ARN.

```hcl
# WRONG — secret value ends up in state
resource "aws_db_instance" "main" {
  password = var.db_password  # This stores the password in state
}

# RIGHT — reference the secret ARN; the value is fetched at runtime
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "/product/prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### Tagging — Required on All Resources

```hcl
locals {
  common_tags = {
    Environment = var.environment          # prod | staging | dev
    Product     = "product-name"
    ManagedBy   = "terraform"
    GoalId      = var.goal_id             # G-YYYY-NNN from CDR-001
    ArtifactId  = var.artifact_id         # EL-001-... from current task
  }
}

resource "aws_s3_bucket" "data" {
  bucket = "company-product-${var.environment}-data"
  tags   = local.common_tags
}
```

---

## RDS (PostgreSQL) — Standard Configuration

```hcl
resource "aws_db_instance" "main" {
  identifier        = "${var.product}-${var.environment}-postgres"
  engine            = "postgres"
  engine_version    = "16.2"
  instance_class    = var.db_instance_class   # db.t3.medium for staging; db.r6g.large for prod
  allocated_storage = 100
  storage_type      = "gp3"
  storage_encrypted = true                     # Always encrypt

  db_name  = var.db_name
  username = var.db_username
  # Password via secrets manager — see above

  multi_az               = var.environment == "prod"  # Multi-AZ in prod only
  deletion_protection    = var.environment == "prod"  # Prevent accidental deletion in prod
  skip_final_snapshot    = var.environment != "prod"  # Snapshot on deletion in prod
  final_snapshot_identifier = "${var.product}-${var.environment}-final"

  backup_retention_period = var.environment == "prod" ? 30 : 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "Mon:04:00-Mon:05:00"

  performance_insights_enabled = true
  monitoring_interval          = 60  # Enhanced monitoring every 60s

  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  tags = local.common_tags
}

resource "aws_security_group" "db" {
  name   = "${var.product}-${var.environment}-db-sg"
  vpc_id = var.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]  # Only from app tier — never 0.0.0.0/0
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## S3 Bucket — Secure Defaults

```hcl
resource "aws_s3_bucket" "artifacts" {
  bucket = "${var.product}-${var.environment}-artifacts"
  tags   = local.common_tags
}

resource "aws_s3_bucket_versioning" "artifacts" {
  bucket = aws_s3_bucket.artifacts.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "artifacts" {
  bucket = aws_s3_bucket.artifacts.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "artifacts" {
  bucket                  = aws_s3_bucket.artifacts.id
  block_public_acls       = true    # Always block public ACLs
  block_public_policy     = true    # Always block public policies
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

---

## Kubernetes — Pod Security Defaults

Every workload manifest should include these security contexts:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-api
  labels:
    app: product-api
    version: "{{ .Values.image.tag }}"
    goal-id: "{{ .Values.goalId }}"
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: product-api
  template:
    metadata:
      labels:
        app: product-api
    spec:
      serviceAccountName: product-api-sa  # Least-privilege SA; not default
      securityContext:
        runAsNonRoot: true          # Never run as root
        runAsUser: 1000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          securityContext:
            allowPrivilegeEscalation: false  # Cannot escalate privileges
            readOnlyRootFilesystem: true      # Immutable container filesystem
            capabilities:
              drop: ["ALL"]                   # Drop all Linux capabilities
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          env:
            - name: DB_URL
              valueFrom:
                secretKeyRef:
                  name: product-secrets
                  key: db-url
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```

---

## Common Cost Control Patterns

EL includes monthly cost impact in EL-001. Use these patterns to avoid surprises.

**NAT Gateway** is the most common unexpected cost. In AWS, NAT Gateway charges
$0.045/GB processed. Use VPC endpoints for S3, DynamoDB, and other AWS services
to avoid routing traffic through NAT.

```hcl
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = var.vpc_id
  service_name = "com.amazonaws.${var.region}.s3"
  route_table_ids = [aws_route_table.private.id]
}
```

**RDS Multi-AZ** doubles the cost. Enable in production; disable in staging.
Use `var.environment == "prod"` guards as shown above.

**CloudWatch Log retention** defaults to never expire. Always set retention:
```hcl
resource "aws_cloudwatch_log_group" "api" {
  name              = "/product/api"
  retention_in_days = var.environment == "prod" ? 90 : 14
}
```
