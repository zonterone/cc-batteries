---
name: ts-code-style
description: Opinionated TypeScript/JavaScript code style guide. MUST be consulted whenever writing, refactoring, reviewing, debugging, or generating any .ts, .tsx, .js, or .jsx code.
---

You prioritize readable, explicit code over overly compact solutions. This is a balance that you have mastered as a result your years as an expert software engineer.

# Knowledge

- follow existing code style in the codebase

# Coding Style

## Comments

- NEVER write comments in code. The only exceptions are `TODO` and `FIXME`
- names, types and structure must carry the intent
- every stub, mock or unwired handler carries a one-line `TODO` naming what completes it

## Style Guide

### General Principles

- Prefer types over interfaces
- Keep things in one function unless composable or reusable
- Use proper error handling patterns (avoid try/catch when possible)
- Prefer single word variable names where possible
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- Prefer guard clauses over nested conditionals
- Prefer arrow functions over `function` keyword
- Reducing unnecessary complexity and nesting
- Eliminating redundant code and abstractions
- Improving readability through clear variable and function names
- Consolidating related logic
- Removing unnecessary comments that describe obvious code
- IMPORTANT: Avoid nested ternary operators - prefer switch statements or if/else chains for multiple conditions or use ts-pattern if available
- Choose clarity over brevity - explicit code is often better than overly compact code

### Type Safety

Every type error and lint error is fixed at its root cause: correct the types, narrow with type guards, validate input at the boundary (`unknown` + parsing), or restructure the code. The checker and linter are always right until proven otherwise.

Escape hatches are a hard guardrail — zero tolerance, including in tests:

- `any` (explicit or implicit)
- `@ts-ignore`, `@ts-expect-error`, `@ts-nocheck`
- `eslint-disable` in any form
- `as` casts that bypass checking (`as unknown as T`, `as any`); `as const` and `satisfies` are fine
- non-null assertion `!`

- `?? ''` / `|| ''` fallbacks that mask a nullable value — fix the upstream type, branch explicitly, or throw early

If the root cause cannot be fixed (broken third-party types, a genuine checker limitation), stop and ask the user, explaining why — never ship an escape hatch on your own.

Compare with `===` / `!==` only; write `=== null` / `=== undefined` per the value's actual type.

Derive types from the data that produces them (`keyof typeof`, `ReturnType<typeof …>`, `(typeof X)[number]`) and export them from the same file.

### Naming

Maintain consistent naming conventions
Prefer single word names for variables and functions. Only use multiple words if necessary.

```ts
// Good
const foo = 1;
function journal(dir: string) {}

// Bad
const fooBar = 1;
function prepareJournal(dir: string) {}
```

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json();

// Bad
const journalPath = path.join(dir, "journal.json");
const journal = await Bun.file(journalPath).json();
```

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2;

// Bad
let foo;
if (condition) foo = 1;
else foo = 2;
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
  if (condition) return 1;
  return 2;
}

// Bad
function foo() {
  if (condition) return 1;
  else return 2;
}
```

**Maintain Balance**: Avoid over-simplification that could:

- Reduce code clarity or maintainability
- Create overly clever solutions that are hard to understand
- Combine too many concerns into single functions or components
- Remove helpful abstractions that improve code organization
- Prioritize "fewer lines" over readability (e.g., nested ternaries, dense one-liners)
- Make the code harder to debug or extend
