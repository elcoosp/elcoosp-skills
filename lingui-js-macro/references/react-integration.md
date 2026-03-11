# React Integration with Macros

Lingui provides specific macros for React to make JSX translation seamless.

## `<Trans>` Component

The primary way to translate JSX trees.

```jsx
import { Trans } from '@lingui/react/macro';

// Simple text
<Trans>Hello World</Trans>

// With variables
<Trans>Hello {username}</Trans>

// With rich text (components)
<Trans>
  Read the <a href="/docs">documentation</a>.
</Trans>
```

**Why use `<Trans>` over `t`?**

- `t` is a tagged template literal (strings only).
- `<Trans>` creates React components and preserves the React tree structure during translation.

## `useLingui` Macro

A macro wrapper around the `useLingui` hook that allows using `t` inside component logic.

```jsx
import { useLingui } from "@lingui/react/macro";

function Profile({ user }) {
  const { t } = useLingui();

  return <span>{t`Welcome back, ${user.name}`}</span>;
}
```

## Plural and Select in JSX

```jsx
import { Plural, Select } from "@lingui/react/macro";

function Notifications({ count }) {
  return (
    <Plural
      value={count}
      one="You have # notification"
      other="You have # notifications"
    />
  );
}
```
