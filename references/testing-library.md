# Testing Library

## Overview

This guide covers Testing Library with a focus on writing tests that resemble how users interact with your UI. The core philosophy is **test behavior, not implementation** — query by what's visible or accessible, interact the way a user would, and assert on outcomes rather than internals.

## Best Practices

1. **Query by accessibility first**: prefer `getByRole` (highest priority) → `getByLabelText` → `getByText`. Only use `getByTestId` as a last resort.
2. **Use `userEvent` over `fireEvent`** — `userEvent` simulates realistic browser interactions (focus/blur, keyboard/pointer events); `fireEvent` is a low-level escape hatch.
3. **Always `await userEvent` calls** — `@testing-library/user-event` v14+ is fully async.
4. **Set up `userEvent` once per test** with `userEvent.setup()` rather than calling `userEvent.click()` directly on the static API.
5. **Avoid querying by implementation details** (class names, component names, internal state).
6. **Prefer `findBy*` over `waitFor` + `getBy*`** when waiting for async elements to appear.

## Patterns

### Standard `userEvent` Setup

**When to use:** Every test that involves user interaction. The `setup()` call creates an
isolated instance with its own event state (pointer position, keyboard modifiers, etc.).

```typescript
import userEvent from '@testing-library/user-event';

test('submits the form', async () => {
  const user = userEvent.setup();
  render(<MyForm />);

  await user.type(screen.getByLabelText('Email'), 'hello@example.com');
  await user.click(screen.getByRole('button', { name: /submit/i }));

  expect(screen.getByText('Success')).toBeInTheDocument();
});
```

### `userEvent` with Fake Timers

**When to use:** When your test suite uses fake timers (`e.g. vi.useFakeTimers()` for Vitest) and you have user interactions. Without `advanceTimers`, `userEvent` internally awaits real-time delays that never resolve under fake timers, causing tests to hang or behave incorrectly.

Pass the timer advancement function via the `advanceTimers` option so `userEvent` can
drive fake time forward itself during interactions:

```typescript
// Vitest
import { vi } from 'vitest';
import userEvent from '@testing-library/user-event';

beforeEach(() => { vi.useFakeTimers(); });
afterEach(() => { vi.useRealTimers(); });

test('shows tooltip after delay', async () => {
  const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
  render(<TooltipButton />);

  await user.hover(screen.getByRole('button'));
  expect(screen.getByRole('tooltip')).toBeInTheDocument();
});
```

### Async Element Queries

**When to use:** The element doesn't exist yet at render time (appears after a fetch,
animation, or timeout).

```typescript
test('loads user data', async () => {
  render(<UserProfile id="1" />);

  // findBy* returns a promise — no need for a separate waitFor
  const heading = await screen.findByRole('heading', { name: /alice/i });
  expect(heading).toBeInTheDocument();
});
```

## Pitfalls

### Using fake timers without `advanceTimers`

❌ **Avoid:** `userEvent` delays hang because fake timers never advance automatically.

```typescript
vi.useFakeTimers();

const user = userEvent.setup(); // missing advanceTimers
await user.hover(button); // may hang or silently misbehave
```

✅ **Prefer:**

```typescript
vi.useFakeTimers();

const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
await user.hover(button); // fake time advances correctly during the interaction
```

### Calling `userEvent.*` statically instead of via `setup()`

❌ **Avoid:** The static API doesn't carry interaction state between calls.

```typescript
await userEvent.type(input, "hello");
await userEvent.click(button);
```

✅ **Prefer:**

```typescript
const user = userEvent.setup();
await user.type(input, "hello");
await user.click(button);
```

### Using `getBy*` for async content

❌ **Avoid:** Throws immediately if the element isn't in the DOM yet.

```typescript
await fetchData();
expect(screen.getByText("Loaded")).toBeInTheDocument(); // may fail before render
```

✅ **Prefer:**

```typescript
expect(await screen.findByText("Loaded")).toBeInTheDocument();
```

### Querying by test ID when a better query exists

❌ **Avoid:** Couples tests to implementation details, not user-visible structure.

```typescript
screen.getByTestId("submit-btn");
```

✅ **Prefer:**

```typescript
screen.getByRole("button", { name: /submit/i });
```
