# TypeScript

## Overview

This guide covers idiomatic TypeScript with strict mode enabled. The philosophy is
**type safety without ceremony** — use the type system to catch real bugs and communicate
intent, but avoid over-engineering types for their own sake. Prefer simple, readable
types that compose well over complex, clever ones.

## Best Practices

1. Always enable strict mode (`"strict": true` in `tsconfig*json`). It catches the most bugs.
2. Prefer `interface` for object shapes, `type` for unions/intersections/aliases.
3. Never use `any`. Use `unknown` when the type is genuinely unknown, then narrow it.
4. Avoid type assertions (`as X`) unless you have no alternative — they silence the compiler.
5. Let TypeScript infer where it's obvious; annotate function signatures and public APIs explicitly.
6. Use `readonly` for data that shouldn't be mutated.
7. Use `satisfies` to validate a value matches a type without widening it.
8. Prefer discriminated unions over optional fields for modeling state variants.
9. Use `const` assertions (`as const`) for literal types and enums-as-objects.
10. Keep generics as simple as possible — add constraints only when needed.

## Patterns

### Discriminated Union for State

**When to use:** Modeling mutually exclusive states (loading/error/success, different entity variants). Much safer than a flat object with many optional fields.

```typescript
type RequestState<T> =
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function render(state: RequestState<User>) {
  switch (state.status) {
    case "success":
      return state.data.name;
    case "error":
      return state.error.message;
    default:
      return null;
  }
}
```

### Exhaustive Switch with `never`

**When to use**: Ensure every union case is handled at compile time.

```typescript
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "rect"; w: number; h: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.r ** 2;
    case "rect":
      return shape.w * shape.h;
    default:
      assertNever(shape.kind);
  }
}

function assertNever(value: never) {
  throw new Error("Expected never but received: " + value);
}
```

### `satisfies` for Validated Literals

**When to use:** You want the value to match a type contract but keep the narrowest possible inferred type.

```typescript
type RouteConfig = Record<string, { path: string }>;

// routes.home.path is still typed as '/' (literal), not string
const routes = { home: { path: "/" as const } } satisfies RouteConfig;
```

### Generic Utility with Constraints

**When to use:** Reusable helpers that work across types but need a guarantee about shape.

```typescript
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  return keys.reduce((acc, k) => ({ ...acc, [k]: obj[k] }), {} as Pick<T, K>);
}
```

### Readonly Domain Object

**When to use**: Enforcing immutability for core business entities or value objects. This prevents accidental side effects in complex logic and ensures state transitions are explicit.

```typescript
interface User {
  readonly id: UserId;
  readonly email: string;
  name: string; // mutable — display name can change
}

function updateName(user: Readonly<User>, name: string): User {
  return { ...user, name }; // returns a new object, never mutates
}
```

## Pitfalls

### Widening with `as const` forgotten

❌ **Avoid:** The type is `string[]`, losing the literal values.

```typescript
const ROLES = ["admin", "editor", "viewer"];
type Role = (typeof ROLES)[number]; // string
```

✅ **Prefer**

```typescript
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = (typeof ROLES)[number]; // 'admin' | 'editor' | 'viewer'
```

### Over-annotating obvious types

❌ **Avoid:** Redundant and noisy.

```typescript
const count: number = 0;
const name: string = "Alice";
const users: User[] = getUsers();
```

✅ **Prefer:** Let inference do the work.

```typescript
const count = 0;
const name = "Alice";
const users = getUsers();
```

### Type assertions hiding bugs

❌ **Avoid:** Asserts silence the compiler; if the assumption is wrong, you get incorrect behavior or unclear runtime errors.

```typescript
const user = JSON.parse(data) as User;
user.profile.avatar;
```

✅ **Prefer:** Use a parsing library (e.g. zod); fails locally and explictly.

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.string(),
  profile: z.object({ avatar: z.string() }),
});

const user = UserSchema.parse(JSON.parse(data));
```
