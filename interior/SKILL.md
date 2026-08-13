---
name: interior.dev Component Library Expert
description: Comprehensive guide for AI agents on using the interior.dev React micro-interaction library. It is a copy‑paste library where each component is a single self-contained file. Lists all ready components, their props, hooks, and usage. One dependency: motion.
---

# interior.dev Micro‑Interaction Library Expert Skill

## Core Concept

interior.dev is not an installable package. Every component is a single file you copy into your project (e.g., `components/interior/<slug>.tsx`). The file exports two things:

1.  A **headless hook** (`useX`) – owns all behaviour, touches zero class names.
2.  A **styled component** (`X`) – a ready‑to‑use example built on the hook.

The hook is the product; the component is an example. You can keep the styled component, modify it, or build your own UI on top of the hook.

### Installation

Add the only runtime dependency:

```bash
pnpm add motion
# or
bun add motion
```

Then copy any component file from the docs into your project. The component is yours after that.

### Shadcn‑style Registry

You can also use the shadcn CLI to copy a component:

```bash
pnpx shadcn@latest add https://www.interior.dev/r/<slug>.json
```

---

## Component Reference (Ready Components)

Below is every ready component grouped by category. For each component, we list:
- the hook’s name (if any) and its key return values,
- the styled component’s main props (with types and defaults),
- a minimal usage example.

### 1. Action Feedback

#### CopyButton
- **Purpose:** Copy text to clipboard. Icon morphs from copy to checkmark.
- **Hook:** `useCopyToClipboard` – returns `{ copy, status, copied }`.
- **Component Props:**
  - `value: string` – text to copy.
  - `label?: string` (default `"Copy"`)
  - `copiedLabel?: string` (default `"Copied"`)
  - `errorLabel?: string` (default `"Failed"`)
  - `timeout?: number` (default `2000`) – ms before reverting to idle.
  - `onCopy?: (value: string) => void`
  - `onError?: (reason: unknown) => void`
  - `disabled?: boolean`
- **Example:**
  ```tsx
  <CopyButton value="npx interior add copy-button" />
  ```

#### LoadingButton
- **Purpose:** Button that shows loading, success, and error states while an async action runs.
- **Hook:** `useAsyncAction` – returns `{ status, run, reset, pending }`.
- **Component Props:**
  - `onAction: () => unknown` – async function.
  - `children: string` – label for idle state.
  - `pendingLabel?: string` (default: `children`)
  - `successLabel?: string` (default: `"Done"`)
  - `errorLabel?: string` (default: `"Try again"`)
  - `resetAfter?: number` (default `1400`) – ms before returning to idle.
  - `disabled?: boolean`
- **Example:**
  ```tsx
  <LoadingButton onAction={publish}>Publish</LoadingButton>
  ```

#### HoldToConfirm
- **Purpose:** Requires holding for a set duration to confirm a destructive action.
- **Hook:** `useHoldToConfirm` – returns `{ bind, step, progress, phase, reset }`.
- **Component Props:**
  - `onConfirm: () => void`
  - `children: ReactNode`
  - `onAbort?: () => void`
  - `confirmLabel?: string` (default `"Confirmed"`)
  - `duration?: number` (default `1800`) – ms to hold.
  - `resetAfter?: number` (default `1600`) – ms after confirm before resetting.
  - `disabled?: boolean`
- **Example:**
  ```tsx
  <HoldToConfirm onConfirm={deleteWorkspace}>
    Hold to delete workspace
  </HoldToConfirm>
  ```

#### LikeBurst
- **Purpose:** Optimistic like button with burst particle effect.
- **Hook:** `useOptimisticLike` – returns `{ liked, count, pending, toggle, burst }`.
- **Component Props:**
  - `initialLiked?: boolean` (default `false`)
  - `initialCount?: number` (default `0`)
  - `onCommit?: (liked: boolean, signal: AbortSignal) => Promise<unknown>`
  - `onError?: (error: unknown) => void`
  - `onToggle?: (liked: boolean) => void`
  - `settle?: number` (default `400`) – ms to wait before committing.
  - `label?: string` (default `"Like"`)
  - `activeLabel?: string` (default `"Liked"`)
  - `format?: (value: number) => string`
- **Example:**
  ```tsx
  <LikeBurst initialCount={128} />
  ```

#### Ripple
- **Purpose:** Touch ripple effect from the pointer origin.
- **Hook:** `useRipple` – returns `{ bind, ripples, fadeDuration }`.
- **Component Props:**
  - `children: ReactNode`
  - `onPress?: () => void`
  - `disabled?: boolean`
  - `max?: number` (default `4`) – max concurrent ripples.
  - `tintClassName?: string` (default `"bg-stone-800/15 dark:bg-white/20"`)
- **Example:**
  ```tsx
  <Ripple>Tap anywhere</Ripple>
  ```

#### IconMorph
- **Purpose:** Morph between two icons (e.g., menu↔close, play↔pause).
- **Hook:** `useIconMorph` – returns `{ index, toggle, setIndex, label }`.
- **Component Props:**
  - `preset?: "menu-close" | "play-pause" | "plus-minus" | "check-close"`
  - `shapes?: MorphShape[]` – custom paths; each shape is `{ d: string[]; rotate?: number }`.
  - `mode?: "stroke" | "fill"`
  - `labels?: string[]`
  - `active?: number | boolean` – controlled index.
  - `defaultActive?: number | boolean` (default `0`)
  - `onActiveChange?: (index: number) => void`
  - `size?: number` (default `20`)
  - `showLabel?: boolean` (default `false`)
  - `semantics?: "label" | "pressed" | "expanded"`
- **Example:**
  ```tsx
  <IconMorph preset="menu-close" semantics="expanded" />
  ```

#### PressDepth
- **Purpose:** Button that physically depresses and tilts toward the finger.
- **Hook:** `usePressDepth` – returns `{ pressed, origin, ref, bind }`.
- **Component Props:**
  - `children: ReactNode`
  - `depth?: number` (default `4`) – px depression.
  - `tilt?: number` (default `7`) – deg tilt.
  - `disabled?: boolean`
  - `type?: "button" | "submit" | "reset"`
  - `onClick?: React.MouseEventHandler`
- **Example:**
  ```tsx
  <PressDepth>Press me</PressDepth>
  ```

---

### 2. Input

#### FloatingLabelInput
- **Purpose:** Label that floats above the field when focused or filled.
- **Hook:** `useFloatingLabel` – returns `{ ref, raised, focused, filled, fieldProps }`.
- **Component Props:**
  - `label: string`
  - `value?: string` / `defaultValue?: string`
  - `onChange?: (value: string, event: ChangeEvent) => void`
  - `hint?: string`
  - `invalid?: boolean`
  - `id?: string`, `name?: string`
  - `type?: "text" | "email" | "password" | "search" | "tel" | "url"`
  - `autoComplete?: string`, `inputMode?: string`
  - `maxLength?: number`
  - `required?: boolean`, `disabled?: boolean`, `readOnly?: boolean`
- **Example:**
  ```tsx
  <FloatingLabelInput label="Account reference" hint="Printed on statement." maxLength={16} />
  ```

#### InlineValidation
- **Purpose:** Input that validates on blur/debounce and shows success/error inline.
- **Hook:** `useInlineValidation` – returns `{ status, error, message, touched, commit, reset, fieldProps }`.
- **Component Props:**
  - `label: string`
  - `value: string`
  - `onChange: (value: string) => void`
  - `validate: (value: string) => string | null` – returns error message or null.
  - `hint?: string`
  - `debounce?: number` (default `400`)
  - `reserveLines?: number` (default `1`)
  - `type`, `placeholder`, `autoComplete`, `inputMode`, `disabled`, `required`
- **Example:**
  ```tsx
  <InlineValidation
    label="Email"
    value={email}
    onChange={setEmail}
    validate={(v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) ? null : "Invalid email"}
    hint="Work email only"
  />
  ```

#### PasswordStrength
- **Purpose:** Password strength meter with rule list.
- **Hook:** `usePasswordStrength` – returns `{ score, max, label, rules, guessable, announcement }`.
- **Component Props:**
  - `value: string`
  - `rules?: PasswordRule[]` – default includes length, case, digit, symbol.
  - `labels?: string[]` – default `["Empty", "Weak", "Fair", "Good", "Strong"]`.
  - `showRules?: boolean` (default `true`)
- **Example:**
  ```tsx
  <PasswordStrength value={password} />
  ```

#### OtpInput
- **Purpose:** One‑time password input with auto‑advance, paste, and error recovery.
- **Hook:** `useOtpInput` – returns `{ chars, value, complete, focusedIndex, getCellProps, clear }`.
- **Component Props:**
  - `length?: number` (default `6`)
  - `mode?: "numeric" | "alphanumeric"` (default `"numeric"`)
  - `defaultValue?: string`
  - `onChange?: (value: string) => void`
  - `onComplete?: (value: string) => void`
  - `status?: "idle" | "error" | "success"`
  - `errorMessage?: string`, `successMessage?: string`
  - `hint?: string`, `label?: string` (default `"Verification code"`)
  - `groupEvery?: number` (default `3`)
  - `disabled?: boolean`, `autoFocus?: boolean`
- **Example:**
  ```tsx
  <OtpInput onComplete={handleComplete} hint="Enter the 6-digit code" />
  ```

#### TagInput
- **Purpose:** Add and remove tags with keyboard support (Enter adds, Backspace removes).
- **Hook:** `useTagInput` – returns `{ tags, draft, add, removeAt, armedIndex, inputProps }`.
- **Component Props:**
  - `value?: string[]`, `defaultValue?: string[]`
  - `onChange?: (tags: string[]) => void`
  - `max?: number` – max tags.
  - `separators?: string[]` (default `[","]`)
  - `allowDuplicates?: boolean` (default `false`)
  - `validate?: (candidate: string, tags: string[]) => boolean`
  - `label?: string`, `placeholder?: string` (default `"Add a tag"`), `hint?: string`
- **Example:**
  ```tsx
  <TagInput defaultValue={["motion", "focus"]} max={6} />
  ```

#### ExpandingSearch
- **Purpose:** Icon expands into a search field; collapses on blur if empty.
- **Hook:** `useExpandingSearch` – returns `{ open, query, expand, collapse, toggle, clear, inputRef, triggerRef, inputProps, triggerProps }`.
- **Component Props:**
  - `value?: string`, `defaultValue?: string`
  - `onChange?: (value: string) => void`
  - `onSearch?: (value: string) => void` – debounced.
  - `onSubmit?: (value: string) => void` – on Enter.
  - `open?: boolean`, `defaultOpen?: boolean`, `onOpenChange?: (open: boolean) => void`
  - `debounce?: number` (default `220`)
  - `collapseOnBlur?: boolean` (default `true`)
  - `disabled?: boolean`
  - `label?: string` (default `"Search"`), `placeholder?: string` (default `"Search"`)
  - `resultCount?: number` – shows count next to clear button.
  - `align?: "left" | "right"` (default `"right"`)
- **Example:**
  ```tsx
  <ExpandingSearch resultCount={3} />
  ```

---

### 3. Async

#### SkeletonSwap
- **Purpose:** Smoothly swap skeleton for content with no layout shift.
- **Hook:** `useSkeletonSwap` – returns `{ showSkeleton, busy }`.
- **Component Props:**
  - `ready: boolean`
  - `children: ReactNode`
  - `lines?: number` (default `3`) – number of skeleton lines.
  - `lineHeight?: number` (default `21`)
  - `barHeight?: number` (default `9`)
  - `reserve?: number` – fixed height override.
  - `delay?: number` (default `120`) – ms before showing skeleton.
  - `minVisible?: number` (default `380`) – minimum skeleton visible time.
  - `label?: string`, `skeleton?: ReactNode`
- **Example:**
  ```tsx
  <SkeletonSwap ready={dataLoaded} lines={4}>
    <p>{content}</p>
  </SkeletonSwap>
  ```

#### ProgressBar
- **Purpose:** Determinate/indeterminate progress bar with label and percentage.
- **Props:**
  - `value: number | null` – null for indeterminate.
  - `max?: number` (default `100`)
  - `label?: string` (default `"Progress"`)
  - `pendingLabel?: string` (default `"Working"`)
  - `completeLabel?: string` (default `"Complete"`)
- **Example:**
  ```tsx
  <ProgressBar value={uploadProgress} label="Uploading" />
  ```

#### LoadMore
- **Purpose:** Sentinel that triggers loading when scrolled into view.
- **Hook:** `useLoadMore` – returns `{ status, sentinelRef, load }`.
- **Component Props:**
  - `onLoad: () => unknown` – returns `false` to indicate end.
  - `hasMore?: boolean` (default `true`)
  - `auto?: boolean` (default `true`)
  - `rootRef?: RefObject<Element>` – scroll container.
  - `rootMargin?: string` (default `"600px 0px"`)
  - `maxAutoLoads?: number` (default `3`) – number of auto‑loads before pausing.
  - `labels?: Partial<Record<"idle" | "loading" | "error" | "end", string>>`
- **Example:**
  ```tsx
  <LoadMore onLoad={fetchMore} hasMore={hasNextPage} />
  ```

#### StreamingText
- **Purpose:** Stream text token by token with a caret.
- **Hook:** `useStreamingText` – returns `{ visible, status, start, pause, skip, reset }`.
- **Component Props:**
  - `text: string`
  - `tokensPerSecond?: number` (default `18`)
  - `autoStart?: boolean` (default `true`)
  - `showSkip?: boolean` (default `true`)
  - `label?: string`, `onDone?: () => void`
- **Example:**
  ```tsx
  <StreamingText text={longText} tokensPerSecond={9} />
  ```

#### TaskSteps
- **Purpose:** Show a multi‑step progress list with status icons.
- **Hook:** `useTaskSteps` – returns `{ rows, complete, failed, sentence }`.
- **Component Props:**
  - `steps: TaskStep[]` – `{ id, label, meta? }`
  - `current: number` – index of active step.
  - `failed?: boolean` (default `false`)
  - `label?: string` (default `"Task progress"`)
- **Example:**
  ```tsx
  <TaskSteps steps={STEPS} current={stepIndex} failed={hasError} />
  ```

---

### 4. Notification

#### LiveActivity
- **Purpose:** Display a persistent activity status (running, success, error) with progress.
- **Hook:** `useLiveActivity` – returns `{ activity, start, update, succeed, fail, dismiss }`.
- **Component Props:**
  - `activity: Activity | null`
  - `onDismiss?: () => void`
  - `width?: number` (default `300`)
  - `dismissLabel?: string`, `label?: string`
- **Example:**
  ```tsx
  <LiveActivity activity={pod.activity} onDismiss={pod.dismiss} />
  ```

#### CollapsibleBanner
- **Purpose:** Banner with expand/collapse and dismiss.
- **Hook:** `useCollapsibleBanner` – returns `{ state, open, fold, expand, dismiss, restore }`.
- **Component Props:**
  - `title: ReactNode`
  - `description?: ReactNode`
  - `children?: ReactNode`
  - `action?: ReactNode`
  - `icon?: ReactNode`
  - `dismissible?: boolean` (default `true`)
  - `state?: BannerState`, `defaultState?: BannerState` (default `"open"`)
  - `onStateChange?: (state: BannerState) => void`, `onDismiss?: () => void`
- **Example:**
  ```tsx
  <CollapsibleBanner title="Storage full" description="94% used" action={<button>Upgrade</button>} />
  ```

#### PresenceAvatars
- **Purpose:** Show who is present with overflow count.
- **Hook:** `usePresence` – returns `{ ordered, visible, hidden, overflow, summary, announcement }`.
- **Component Props:**
  - `people: PresencePerson[]` – `{ id, name, src? }`
  - `max?: number` (default `5`)
  - `size?: number` (default `28`)
  - `overlap?: number` (default `9`)
  - `label?: string`, `announceAfter?: number`
  - `onOverflowSelect?: (hidden: PresencePerson[]) => void`
- **Example:**
  ```tsx
  <PresenceAvatars people={activeUsers} max={4} size={48} overlap={14} />
  ```

#### TypingIndicator
- **Purpose:** Show typing dots with names.
- **Hook:** `useTypingPresence` – returns `{ typists, sending, ping, send, clear, reset }`.
- **Component Props:**
  - `typists: string[]` – list of typing names.
  - `sending?: boolean` – whether a message is being sent (hides dots).
  - `max?: number` (default `2`) – max names shown in label.
  - `size?: number` (default `34`) – container height.
  - `showLabel?: boolean` (default `true`)
  - `announceAfter?: number` (default `700`)
- **Example:**
  ```tsx
  <TypingIndicator typists={["Nadia", "Ravi"]} sending={false} />
  ```

#### NewItemsPill
- **Purpose:** Pill that appears when new items are added above the current scroll position.
- **Hook:** `useNewItems` – returns `{ scrollProps, unread, pinned, jump }`.
- **Component Props:**
  - `count: number` – number of new items.
  - `onJump: () => void` – called when pill is clicked.
  - `anchor?: "top" | "bottom"` (default `"top"`)
  - `label?: (count: number) => string`, `max?: number` (default `99`)
- **Example:**
  ```tsx
  <NewItemsPill count={unread} onJump={jumpToTop} />
  ```

---

### 5. Overlay

#### Modal
- **Purpose:** Full‑featured modal with backdrop, scroll lock, focus trap.
- **Hook:** `useModal` – returns `{ target, titleId, descriptionId, overlayProps, panelProps, close }`.
- **Component Props:**
  - `open: boolean`, `onClose: () => void`
  - `title: ReactNode`, `description?: ReactNode`
  - `children?: ReactNode`, `footer?: ReactNode`
  - `closeLabel?: string` (default `"Close dialog"`)
  - `showClose?: boolean` (default `true`)
  - `closeOnEscape?: boolean` (default `true`)
  - `closeOnBackdrop?: boolean` (default `true`)
  - `lockScroll?: boolean` (default `true`)
  - `initialFocusRef?: RefObject<HTMLElement>`
  - `container?: HTMLElement | null` – portal target (default `document.body`)
  - `maxWidth?: number` (default `440`), `maxHeight?: string` (default `"min(78vh, 620px)"`)
- **Example:**
  ```tsx
  <Modal open={open} onClose={() => setOpen(false)} title="Delete project" description="Cannot be undone.">
    <p>Are you sure?</p>
  </Modal>
  ```

#### Drawer
- **Purpose:** Side panel with drag‑to‑dismiss.
- **Hook:** `useDrawer` – returns `{ open, side, width, x, veil, setOpen, close, panelProps, gripProps }`.
- **Component Props:**
  - `open: boolean`, `onOpenChange: (open: boolean) => void`
  - `title: string`
  - `children: ReactNode`
  - `description?: string`, `footer?: ReactNode`
  - `side?: "left" | "right"` (default `"right"`)
  - `width?: number` (default `320`)
  - `container?: "viewport" | "parent"` (default `"viewport"`)
  - `closeLabel?: string` (default `"Close panel"`)
  - `dismissOnScrimClick?: boolean` (default `true`)
- **Example:**
  ```tsx
  <Drawer open={open} onOpenChange={setOpen} title="Edit profile" side="right">
    <form>...</form>
  </Drawer>
  ```

#### Popover
- **Purpose:** Floating panel anchored to a trigger with collision‑aware placement.
- **Hook:** `usePopover` – returns `{ anchorRef, floatingRef, panelRef, contentRef, arrowRef, side, update }`.
- **Component Props:**
  - `trigger: ReactNode` – the element that opens the popover.
  - `children: ReactNode` – popover content.
  - `label: string` – accessible name.
  - `open?: boolean`, `defaultOpen?: boolean`, `onOpenChange?: (open: boolean) => void`
  - `side?: "top" | "right" | "bottom" | "left"` (default `"bottom"`)
  - `align?: "start" | "center" | "end"` (default `"center"`)
  - `offset?: number` (default `10`)
  - `padding?: number` (default `8`)
  - `arrowSize?: number` (default `9`)
  - `boundary?: RefObject<HTMLElement>`
- **Example:**
  ```tsx
  <Popover trigger="Open" label="Profile" side="top">
    <div>Profile content</div>
  </Popover>
  ```

#### CommandPalette
- **Purpose:** Modal command palette with fuzzy search and keyboard navigation.
- **Hook:** `useCommandPalette` – returns `{ query, setQuery, results, activeId, listRef, onKeyDown, ... }`.
- **Component Props:**
  - `items: CommandItem[]` – `{ id, label, hint?, keywords?, shortcut? }`
  - `onSelect: (item: CommandItem) => void`
  - `onDismiss?: () => void`
  - `open?: boolean`
  - `placeholder?: string` (default `"Search commands"`)
  - `emptyLabel?: string` (default `"No command matches"`)
  - `label?: string` (default `"Command palette"`)
  - `maxRows?: number` (default `6`)
  - `autoFocus?: boolean`
- **Example:**
  ```tsx
  <CommandPalette open={open} items={commands} onSelect={handleSelect} onDismiss={() => setOpen(false)} />
  ```

#### ContextMenu
- **Purpose:** Right‑click context menu with keyboard access.
- **Hook:** `useContextMenu` – returns `{ isOpen, placement, openAt, close, triggerProps, menuProps, getItemProps }`.
- **Component Props:**
  - `items: ContextMenuItem[]` – `{ id, label, shortcut?, icon?, disabled?, onSelect? }` and `{ type: "separator" }`.
  - `children: ReactNode` – the element that triggers the menu.
  - `onSelect?: (id: string) => void`
  - `label?: string` (default `"Context menu"`)
  - `width?: number` (default `224`)
  - `disabled?: boolean`
- **Example:**
  ```tsx
  <ContextMenu items={actions} label="File options">
    <div>Right‑click me</div>
  </ContextMenu>
  ```

#### Dropdown
- **Purpose:** Select dropdown with typeahead and keyboard navigation.
- **Hook:** `useDropdown` – returns `{ open, selectedIndex, activeIndex, triggerProps, listProps, getItemProps }`.
- **Component Props:**
  - `items: DropdownItem[]` – `{ value, label, hint?, disabled? }`
  - `value?: string`, `defaultValue?: string`, `onChange?: (value: string) => void`
  - `label?: string` (default `"Options"`)
  - `placeholder?: string` (default `"Select an option"`)
  - `disabled?: boolean`
  - `emptyLabel?: string` (default `"Nothing to choose"`)
- **Example:**
  ```tsx
  <Dropdown items={options} value={selected} onChange={setSelected} label="Visibility" />
  ```

#### TooltipGroup
- **Purpose:** Group of tooltips with shared delay and warm‑up logic (second tooltip appears instantly).
- **Hook:** `useTooltip` – returns `{ open, warm, skipped, travel, tooltipId, triggerProps }`.
- **Component Props:**
  - `children: ReactNode`
  - `openDelay?: number` (default `200`)
  - `closeDelay?: number` (default `120`)
  - `skipDelay?: number` (default `400`) – ms before the group becomes "warm".
  - `onWarmChange?: (warm: boolean) => void`
  - `className?: string`
- **Example:**
  ```tsx
  <TooltipGroup>
    <Tooltip label="Bold (⌘B)"><button>B</button></Tooltip>
    <Tooltip label="Italic (⌘I)"><button>I</button></Tooltip>
  </TooltipGroup>
  ```

---

### 6. Navigation

#### Tabs
- **Purpose:** Tabs with an animated indicator.
- **Hook:** `useTabs` – returns `{ value, select, direction, tabListProps, getTabProps, getPanelProps }`.
- **Component Props:**
  - `items: TabItem[]` – `{ value, label, disabled? }`
  - `value?: string`, `defaultValue?: string`, `onValueChange?: (value: string) => void`
  - `activation?: "automatic" | "manual"` (default `"automatic"`)
  - `renderPanel?: (value: string) => ReactNode`
  - `label?: string` (default `"Tabs"`)
- **Example:**
  ```tsx
  <Tabs items={tabs} value={current} onValueChange={setTab} renderPanel={renderContent} />
  ```

#### SegmentedControl
- **Purpose:** Radio‑style segmented control with sliding thumb.
- **Props:**
  - `options: SegmentedOption[]` – `{ value, label, disabled? }`
  - `label: string`
  - `value?: string`, `defaultValue?: string`, `onValueChange?: (value: string) => void`
- **Example:**
  ```tsx
  <SegmentedControl options={ranges} value={range} onValueChange={setRange} label="Range" />
  ```

#### Accordion
- **Purpose:** Accessible accordion with auto‑height animation.
- **Hook:** `useAccordion` – returns `{ open, isOpen, toggle, headerProps, panelProps }`.
- **Component Props:**
  - `items: AccordionItem[]` – `{ id, title, content, meta? }`
  - `type?: "single" | "multiple"` (default `"single"`)
  - `defaultOpen?: string[]`, `open?: string[]`, `onOpenChange?: (open: string[]) => void`
  - `collapsible?: boolean` (default `true`) – for single mode, allow closing all.
  - `maxPanelHeight?: number` (default `220`)
  - `headingLevel?: number` (default `3`)
- **Example:**
  ```tsx
  <Accordion items={faqItems} defaultOpen={["shipping"]} />
  ```

#### WizardSteps
- **Purpose:** Multi‑step wizard with progress bar and animated panel transitions.
- **Hook:** `useWizard` – returns `{ index, direction, furthest, total, isFirst, isLast, next, back, goTo }`.
- **Component Props:**
  - `steps: WizardStep[]` – `{ id, label, content }`
  - `index?: number`, `defaultIndex?: number`, `onIndexChange?: (index: number, direction: WizardDirection) => void`
  - `onComplete?: () => void`
  - `complete?: boolean` – show completion state.
  - `height?: number` (default `184`)
  - `backLabel?: string` (default `"Back"`)
  - `nextLabel?: string` (default `"Next"`)
  - `finishLabel?: string` (default `"Finish"`)
  - `completeLabel?: string` (default `"All set"`)
  - `completeHint?: string` (default `"Step back to change anything"`)
- **Example:**
  ```tsx
  <WizardSteps steps={steps} index={step} onIndexChange={setStep} onComplete={handleComplete} />
  ```

#### Pagination
- **Purpose:** Pagination with sliding window and thumb.
- **Hook:** `usePagination` – returns `{ page, count, items, direction, thumbIndex, canPrev, canNext, goTo, prev, next }`.
- **Component Props:**
  - `count: number`
  - `page?: number`, `defaultPage?: number`, `onPageChange?: (page: number) => void`
  - `siblings?: number` (default `1`)
  - `boundaries?: number` (default `1`)
  - `label?: string` (default `"Pagination"`)
- **Example:**
  ```tsx
  <Pagination count={12} page={currentPage} onPageChange={setPage} />
  ```

#### TreeView
- **Purpose:** Tree with disclosure and full keyboard APG pattern.
- **Hook:** `useTreeView` – returns `{ rows, openSet, selectedId, tabStop, register, focusRow, toggle, select, handleKey }`.
- **Component Props:**
  - `nodes: TreeNode[]` – `{ id, label, meta?, children? }`
  - `label: string`
  - `expanded?: string[]`, `defaultExpanded?: string[]`, `onExpandedChange?: (expanded: string[]) => void`
  - `selected?: string | null`, `defaultSelected?: string | null`, `onSelectedChange?: (selected: string) => void`
- **Example:**
  ```tsx
  <TreeView nodes={fileTree} label="Project files" defaultExpanded={["src"]} />
  ```

---

### 7. Scroll

#### StickyHeader
- **Purpose:** Header that condenses on scroll, with separate compact and expanded states.
- **Hook:** `useCondense` – returns `{ ref, progress, condensed }`.
- **Component Props:**
  - `title: string`
  - `children: ReactNode`
  - `subtitle?: string`
  - `leading?: ReactNode`, `actions?: ReactNode`
  - `expandedHeight?: number` (default `68`)
  - `compactHeight?: number` (default `48`)
  - `maxHeight?: number` (default `320`)
- **Example:**
  ```tsx
  <StickyHeader title="Inbox" subtitle="12 messages">
    <ul>...</ul>
  </StickyHeader>
  ```

#### ReadingProgress
- **Purpose:** Progress bar with estimated time remaining.
- **Hook:** `useReadingProgress` – returns `{ step, steps, progress, percent, minutesLeft, totalMinutes, complete }`.
- **Component Props:**
  - `target?: ScrollRef` – element to track.
  - `scroller?: ScrollRef` – scroll container.
  - `steps?: number` (default `24`)
  - `words?: number` – total word count for time estimate.
  - `wordsPerMinute?: number` (default `220`)
  - `label?: string` (default `"Reading progress"`)
  - `doneLabel?: string` (default `"End"`)
- **Example:**
  ```tsx
  <ReadingProgress scroller={containerRef} words={1200} />
  ```

#### ScrollSpy
- **Purpose:** Navigation that highlights the currently visible section.
- **Hook:** `useScrollSpy` – returns `{ activeId, activeIndex, scrollTo, getLinkProps, announce }`.
- **Component Props:**
  - `sections: ScrollSpySection[]` – `{ id, label }`
  - `offset?: number` (default `96`)
  - `root?: RefObject<HTMLElement>`
  - `onChange?: (id: string) => void`
  - `label?: string` (default `"On this page"`)
- **Example:**
  ```tsx
  <ScrollSpy sections={sections} root={containerRef} />
  ```

#### SnapCarousel
- **Purpose:** Carousel with snap scrolling and momentum.
- **Hook:** `useSnapCarousel` – returns `{ index, count, target, dragging, slideWidth, step, goTo, next, prev, viewportRef, viewportProps, trackProps }`.
- **Component Props:**
  - `children: ReactNode`
  - `label: string`
  - `index?: number`, `defaultIndex?: number`, `onIndexChange?: (index: number) => void`
  - `gap?: number` (default `12`)
  - `peek?: number` – px visible on each side.
  - `momentum?: number` (default `0.14`)
  - `maxFlick?: number` (default `1`)
- **Example:**
  ```tsx
  <SnapCarousel label="Rooms" peek={48}>
    {slides}
  </SnapCarousel>
  ```

#### HideOnScroll
- **Purpose:** Hide a toolbar when scrolling down, reveal when scrolling up.
- **Hook:** `useHideOnScroll` – returns `{ ref, hidden, atTop }`.
- **Component Props:**
  - `bar: ReactNode` – the toolbar content.
  - `children: ReactNode` – scrollable content.
  - `barHeight?: number` (default `44`)
  - `hideAfter?: number` (default `14`) – px scrolled down to hide.
  - `revealAfter?: number` (default `10`) – px scrolled up to reveal.
  - `topGuard?: number` (default `24`) – keep visible near top.
  - `pinned?: boolean` – keep visible.
  - `maxHeight?: number` (default `320`)
  - `label?: string`, `onHiddenChange?: (hidden: boolean) => void`
- **Example:**
  ```tsx
  <HideOnScroll bar={<h2>Longitude</h2>} maxHeight={300}>
    <p>...</p>
  </HideOnScroll>
  ```

---

### 8. Data

#### SortableTable
- **Purpose:** Table with sortable columns and animated row reordering.
- **Hook:** `useSortableRows` – returns `{ sort, ordered, toggle, ariaSort }`.
- **Component Props:**
  - `rows: T[]`
  - `columns: SortableColumn<T>[]` – `{ id, header, width?, align?, numeric?, sortable?, value?, cell? }`
  - `getRowId: (row: T) => string`
  - `label: string`
  - `rowHeight?: number` (default `44`)
  - `maxHeight?: number`
  - `sort?: SortState | null`, `defaultSort?: SortState | null`, `onSortChange?: (next: SortState | null) => void`
  - `markable?: boolean` – enable row selection.
  - `onMarkChange?: (id: string | null) => void`
  - `getRowLabel?: (row: T) => string`
- **Example:**
  ```tsx
  <SortableTable rows={reviewers} columns={cols} getRowId={r => r.id} label="Reviewers" markable />
  ```

#### FilterGrid
- **Purpose:** Filterable grid of items with animated transitions.
- **Hook:** `useFilterGrid` – returns `{ active, activeLabel, select, visible, counts, total }`.
- **Component Props:**
  - `items: T[]`
  - `filters: FilterDefinition<T>[]` – `{ id, label, match: (item) => boolean }`
  - `getKey: (item: T) => string`
  - `renderItem: (item: T) => ReactNode`
  - `label: string`
  - `value?: string`, `defaultValue?: string`, `onValueChange?: (id: string) => void`
  - `columns?: number` (default `3`)
  - `rowHeight?: number` (default `72`)
  - `maxRows?: number` (default `4`)
  - `gap?: number` (default `8`)
  - `emptyLabel?: string` (default `"Nothing matches this filter"`)
- **Example:**
  ```tsx
  <FilterGrid items={assets} filters={filters} getKey={a => a.id} renderItem={renderAsset} label="Asset type" />
  ```

#### ValueFlash
- **Purpose:** Highlight a numeric value change with direction and pulse.
- **Hook:** `useValueFlash` – returns `{ direction, from, changeId, flashing }`.
- **Component Props:**
  - `value: number`
  - `format?: (value: number) => string` – default `String`
  - `label?: string` – announced label.
  - `hold?: number` (default `900`) – ms to show flash.
- **Example:**
  ```tsx
  <ValueFlash value={requestsPerSecond} format={n => n.toLocaleString()} label="RPS" />
  ```

#### PollResults
- **Purpose:** Interactive poll with live results and animated bars.
- **Hook:** `usePollResults` – returns `{ rows, total, chosen, revealed, vote }`.
- **Component Props:**
  - `options: PollOption[]` – `{ id, label, votes }`
  - `label: string` – the question.
  - `value?: string | null`, `defaultValue?: string | null`, `onVote?: (id: string) => void`
- **Example:**
  ```tsx
  <PollResults options={pollOptions} label="Floor for the front room?" />
  ```

---

### 9. Gesture

#### SliderDetents
- **Purpose:** Range slider with detents (snap points) and haptic feedback.
- **Hook:** `useSliderDetents` – returns `{ trackRef, trackProps, detents, activeDetent, percent, dragging, valueText }`.
- **Component Props:**
  - `value: number`, `onValueChange: (value: number) => void`
  - `min?: number` (default `0`), `max?: number` (default `100`), `step?: number` (default `1`)
  - `detents?: (number | { value: number; label?: string })[]`
  - `pull?: number` – attraction radius to detents.
  - `label?: string`, `format?: (value: number) => string`
  - `disabled?: boolean`, `haptic?: boolean` (default `true`)
- **Example:**
  ```tsx
  <SliderDetents value={speed} onValueChange={setSpeed} min={0.25} max={2} detents={[0.5, 1, 1.5, 2]} />
  ```

#### SwipeDeck
- **Purpose:** Tinder‑style card deck with left/right decisions and undo.
- **Hook:** `useSwipeDeck` – returns `{ index, count, decisions, flow, intent, steps, armed, canUndo, decide, undo, report, release, deckProps }`.
- **Component Props:**
  - `items: T[]`
  - `itemKey: (item: T) => string`
  - `itemLabel: (item: T) => string`
  - `children: (item: T) => ReactNode`
  - `onDecide?: (item: T, choice: "left" | "right") => void`
  - `onUndo?: (item: T) => void`
  - `label?: string` (default `"Card deck"`)
  - `leftLabel?: string` (default `"Skip"`), `rightLabel?: string` (default `"Keep"`), `undoLabel?: string` (default `"Undo"`)
  - `emptyLabel?: string` (default `"Deck cleared"`)
  - `height?: number` (default `180`)
  - `threshold?: number` (default `92`)
  - `steps?: number` (default `6`)
  - `peek?: number` (default `3`)
- **Example:**
  ```tsx
  <SwipeDeck items={queue} itemKey={l => l.id} itemLabel={l => l.name} />
  ```

#### ReorderList
- **Purpose:** Drag‑and‑drop reorder list with keyboard support.
- **Hook:** `useReorderList` – returns `{ grabbed, dragging, spoken, grab, drop, cancel, step, rowKeyDown, onDragStart, onDragEnd }`.
- **Component Props:**
  - `items: T[]`
  - `getId: (item: T) => string`
  - `getLabel: (item: T) => string`
  - `onReorder: (next: T[]) => void`
  - `onCommit?: (next: T[]) => void` – final order after drag ends.
  - `disabled?: boolean`
  - `children: (item: T) => ReactNode`
  - `label: string`
- **Example:**
  ```tsx
  <ReorderList items={agenda} getId={t => t.id} getLabel={t => t.title} onReorder={setAgenda} label="Agenda" />
  ```

#### LongPress
- **Purpose:** Detect long press with progress feedback.
- **Hook:** `useLongPress` – returns `{ bind, step, steps, holding, fired, progress }`.
- **Component Props:**
  - `onLongPress: () => void`
  - `children: ReactNode`
  - `duration?: number` (default `550`)
  - `steps?: number` (default `12`)
  - `disabled?: boolean`
- **Example:**
  ```tsx
  <LongPressButton onLongPress={archive}>Hold to archive</LongPressButton>
  ```

#### Lightbox
- **Purpose:** Image lightbox with zoom, pan, and origin animation.
- **Hook:** `useLightbox` – returns `{ frameRef, contentRef, bind, scale, x, y, step, steps, zoom, zoomed, reset, zoomAt }`.
- **Component Props:**
  - `open: boolean`, `onClose: () => void`
  - `src: string`, `alt: string`
  - `originRef?: RefObject<HTMLElement>` – element to animate from.
  - `caption?: string`
  - `width?: number`, `height?: number`
  - `maxScale?: number` (default `4`)
- **Example:**
  ```tsx
  <Lightbox open={open} onClose={() => setOpen(false)} src={imgSrc} alt="..." originRef={thumbRef} />
  ```

---

### 10. Content

#### TextReveal
- **Purpose:** Reveal text word by word or character by character on scroll/trigger.
- **Hook:** `useTextReveal` – returns `{ ref, groups, step, count, started, reduced, duration }`.
- **Component Props:**
  - `text: string`
  - `by?: "word" | "character"` (default `"word"`)
  - `stagger?: number` (default `0.055`)
  - `maxDuration?: number` (default `1.6`)
  - `startOnView?: boolean` (default `true`)
  - `play?: boolean` (default `true`)
  - `once?: boolean` (default `true`)
  - `amount?: number` (default `0.35`) – intersection ratio.
- **Example:**
  ```tsx
  <TextReveal text="Hello world" by="character" stagger={0.08} />
  ```

#### LogoMarquee
- **Purpose:** Infinite scrolling logo marquee that pauses on hover/focus.
- **Hook:** `useLogoMarquee` – returns `{ viewportRef, trackRef, groupRef, copies, paused, reduced, bind }`.
- **Component Props:**
  - `items: LogoMarqueeItem[]` – `{ id, label, href?, mark? }`
  - `label?: string` (default `"Logos"`)
  - `speed?: number` (default `44`) – px/s.
  - `direction?: "left" | "right"` (default `"left"`)
  - `gap?: number` (default `40`)
  - `paused?: boolean`
  - `onSelect?: (item: LogoMarqueeItem) => void`
- **Example:**
  ```tsx
  <LogoMarquee items={brands} label="Customers" />
  ```

#### BlurUpImage
- **Purpose:** Image with blur‑up placeholder and smooth reveal.
- **Hook:** `useBlurUpImage` – returns `{ ref, status, instant, loaded }`.
- **Component Props:**
  - `src?: string`
  - `alt: string`
  - `width: number`, `height: number`
  - `placeholder?: string` – low‑res image (base64).
  - `color?: string` – background colour.
  - `blur?: number` (default `14`) – placeholder blur.
  - `radius?: 5 | 6 | 9 | 11 | 14` (default `11`)
  - `srcSet?: string`, `sizes?: string`
  - `loading?: "lazy" | "eager"` (default `"lazy"`)
  - `fetchPriority?: "high" | "low" | "auto"`
  - `onReady?: () => void`, `onError?: () => void`
- **Example:**
  ```tsx
  <BlurUpImage src={photo} alt="..." width={320} height={200} placeholder={lqip} color="#8e977f" />
  ```

#### ShowMore
- **Purpose:** Expand/collapse long text with height animation.
- **Hook:** `useShowMore` – returns `{ contentRef, expanded, open, toggle, setExpanded, height, collapsedHeight, fullHeight, expandable, capped, scrollable }`.
- **Component Props:**
  - `children: ReactNode`
  - `lines?: number` (default `3`) – number of visible lines when collapsed.
  - `maxHeight?: number` (default `320`) – max expanded height.
  - `defaultExpanded?: boolean`, `expanded?: boolean`, `onExpandedChange?: (expanded: boolean) => void`
  - `moreLabel?: string` (default `"Show more"`), `lessLabel?: string` (default `"Show less"`), `label?: string`
- **Example:**
  ```tsx
  <ShowMore lines={3} maxHeight={200}>{longText}</ShowMore>
  ```

---

## Hooks vs Components

Always prefer the hook if you need custom styling. The styled component is a reference implementation. Use the hook’s returned props to attach to your own elements.

```tsx
// Using the hook directly
const { open, toggle, headerProps, panelProps } = useAccordion({ items });

<div>
  <button {...headerProps("item-1")}>Title</button>
  <div {...panelProps("item-1")}>Content</div>
</div>
```

## Final Notes

- All components respect `prefers-reduced-motion`. They use `useReducedMotion` from `motion`.
- Components are designed for zero layout shift – widths and heights are reserved before content arrives.
- Gesture components handle all abandonment cases – pointercancel, lostpointercapture, window blur, Escape.
- Icons are inline – no icon library dependency.

Use this reference to pick the right component, understand its props, and integrate it seamlessly. Copy the source file, install `motion`, and you’re ready.
