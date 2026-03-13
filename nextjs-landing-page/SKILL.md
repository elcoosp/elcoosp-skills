---
name: nextjs-landing-page
description: Create production‑ready Next.js landing pages with modern patterns – i18n, A/B testing, waitlist capture, analytics, and polished UI components.
version: 1.0.0
license: MIT
author: "@codebridge-community"
tags:
  - nextjs
  - landing-page
  - react
  - tailwindcss
  - i18n
  - a/b-testing
---

## Instructions

Use this skill to scaffold or learn how to build a high‑quality Next.js landing page for any product. It captures the architecture, component patterns, and best practices from the [CodeBridge landing page](https://codebridge.dev) – a fully functional, production‑grade example.

### When to Use

- You need a new landing page for a SaaS product, open‑source project, or mobile app.
- You want to implement modern features like internationalisation (i18n), A/B testing, UTM tracking, and a waitlist signup form.
- You’re looking for a clean, maintainable Next.js (App Router) codebase with Tailwind CSS and shadcn/ui.
- You want to understand how to structure a large Next.js application with server components, client components, and a well‑organized file hierarchy.

### How to Use

1. **Start a new Next.js project** (if not already done):
   ```bash
   npx create-next-app@latest my-landing --typescript --tailwind --app
   ```

2. **Install core dependencies** (as seen in the CodeBridge `package.json`):
   ```bash
   npm install @lingui/core @lingui/react framer-motion lucide-react @icons-pack/react-simple-icons canvas-confetti
   npm install -D @lingui/cli @lingui/swc-plugin
   ```
   For i18n, set up Lingui (see [Lingui docs](https://lingui.dev)).

3. **Structure your app** following the CodeBridge pattern:
   ```
   app/
   ├── [lang]/                 # Dynamic language segment
   │   ├── layout.tsx          # Root layout with fonts, providers
   │   ├── page.tsx            # Homepage – composes sections
   │   ├── globals.css         # Tailwind + custom variables
   │   ├── ABTestLayoutClient.tsx
   │   ├── blog/               # Blog pages
   │   ├── changelog/
   │   ├── faq/
   │   ├── features/
   │   ├── pricing/
   │   ├── privacy/
   │   ├── security/
   │   └── terms/
   ├── components/
   │   ├── sections/           # Large page sections (Hero, Features, Pricing…)
   │   ├── shared/             # Reusable components (Header, Footer, Terminal…)
   │   ├── ui/                 # shadcn/ui components (button, card, …)
   │   └── mascot/             # Custom animated mascot
   ├── hooks/                   # Custom React hooks
   ├── lib/                     # Utilities, config, constants
   ├── contexts/                # React context providers (e.g., UTMContext)
   └── types/                   # Global TypeScript definitions
   ```

4. **Copy key patterns** from the CodeBridge codebase:

   - **i18n**: Use `@lingui` with a `[lang]` dynamic route. Server components fetch translations, client components use `<Trans>`.
   - **A/B testing**: Assign variants via cookies/middleware (`proxy.ts`) and inject them via `ABTestProvider`.
   - **UTM tracking**: Parse UTM parameters, store in cookies (first‑touch), and make available via `UTMContext`.
   - **Analytics**: Umami (or similar) with custom event tracking hooks (`useAnalytics`).
   - **Waitlist**: Database (Turso) + API route + live counter.
   - **Animations**: `framer-motion` with `AnimatedSection` for scroll‑triggered entrances.
   - **Theming**: CSS variables with dark mode support.

5. **Build sections** as composable, client‑side components (using `'use client'` where needed). Each section lives in `components/sections/` and is imported into `page.tsx`.

6. **Add a mascot** (optional) – see `AnimatedMascot.tsx` for a fun, scroll‑aware character.

### Key Components (from the CodeBridge landing page)

| Component | Purpose |
|-----------|---------|
| `Hero` | Main headline, CTA buttons, live waitlist counter, terminal demo |
| `FeatureBelt` | Three core features with icons and info popovers |
| `ComparisonTable` | Feature comparison vs. competitors |
| `WorktreeSection` | Explains parallel worktrees with animated diagram |
| `HowItWorks` | Three‑step installation guide with visuals |
| `PricingCards` / `PricingTable` | A/B‑tested pricing display |
| `LaunchCountdown` | Countdown timer + waitlist form |
| `TestimonialsSection` | Carousel of user quotes |
| `Footer` | Links, social, copyright |

### Best Practices

- **Server Components by default** – move client logic to leaf components or use `'use client'` sparingly.
- **Centralised configuration** – `lib/constants.ts`, `lib/ab-tests.ts`, `lib/roadmap-data.ts`.
- **Custom hooks** – encapsulate logic (e.g., `useAnalytics`, `useChannelExperiment`).
- **Context providers** – wrap in `layout.tsx` (e.g., `UTMProvider`, `ABTestProvider`).
- **Environment variables** – use `NEXT_PUBLIC_*` for client‑side values.
- **Accessibility** – focus indicators, ARIA labels, keyboard navigation (see `InfoPopover.tsx`).
- **Performance** – lazy load below‑fold images, use `next/image`, preload fonts.

### Output

After following this skill, you will have:

- A Next.js project with App Router, TypeScript, Tailwind CSS, and shadcn/ui.
- A fully localised landing page (English, French, Spanish) with dynamic routing.
- A/B testing infrastructure with cookie‑based variant assignment.
- UTM tracking with first‑touch attribution.
- Waitlist signup with Turso database and real‑time counter.
- A polished, responsive UI with animations and a mascot.
- Blog, changelog, FAQ, pricing, and legal pages.

### Examples

**Example 1: Create a landing page for a new SaaS product**

```
User: I need a landing page for my new project management tool called "TaskFlow".
Agent: I'll help you build a Next.js landing page using the patterns from CodeBridge.

First, let's create the project:
npx create-next-app@latest taskflow-landing --typescript --tailwind --app

Then we'll install dependencies and set up the structure as documented in the skill.
```

**Example 2: Add a waitlist form to an existing landing page**

```
User: How do I add a waitlist form with a live counter?
Agent: Copy the `WaitlistForm` component, the `/api/subscribe` route, and the database schema from the skill. Then set up a Turso database and add the environment variables.
```

### Limitations

- This skill provides architectural guidance, not an automatic code generator. You must manually copy and adapt the patterns.
- The i18n setup assumes Lingui; you can replace it with `next-intl` or another library if preferred.
- The A/B testing implementation is cookie‑based and may not scale to millions of users without a dedicated service.

### Dependencies

- Next.js 14+ (App Router)
- TypeScript
- Tailwind CSS
- shadcn/ui (optional but used heavily)
- Lingui (for i18n)
- Framer Motion (for animations)
- Turso / libSQL (for waitlist database)
- Umami (or any analytics provider)
