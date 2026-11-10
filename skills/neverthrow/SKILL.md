---
name: neverthrow
description: >-
  Project convention and reference for error handling with the neverthrow library
  (Result / ResultAsync / ok / err / fromPromise / fromThrowable / fromAsyncThrowable /
  safeTry / combine). Consult whenever writing, refactoring, or reviewing .ts/.tsx code
  that can fail: a promise that may reject, a call that may throw (native modules, expo-*,
  firebase, Linking, JSON.parse), a chain of dependent fallible steps, a fallback value on
  failure, a fire-and-forget side effect, or any spot where you are about to write
  try/catch, .catch() or throw outside react-query. Also use it to decide when a Result is
  overkill (queryFn/mutationFn, nullable results, zod safeParse) and how to consume one
  (match / unwrapOr / isErr / orTee). This project bans try/catch in src, so prefer Result here.
---

# neverthrow in this project

`neverthrow` (`^8.2.0`) is how failure is represented outside react-query. Already a dependency:

```ts
import { fromPromise, fromThrowable, fromAsyncThrowable, ok, err, safeTry } from "neverthrow";
```

## Why we bother

A `throw` is invisible in a signature: the caller cannot see that the function fails, and a
missed `catch` is a crash — there is **no error boundary above the navigator**, so a throw
escaping a handler or effect takes the app down. `Result<T, E>` / `ResultAsync<T, E>` put the
failure into the type: the value cannot be read without deciding what happens on `Err`, and
that decision lives at the call site. `try/catch` does not appear in `src`.

## Where a Result comes from

Convert at the boundary — the moment you touch something that throws or rejects — and never
re-wrap further up.

| Source | Use |
| --- | --- |
| a promise that may reject (`Linking.openURL`, `fetch`, expo / firebase async APIs) | `fromPromise(p, errorFn)` |
| a sync call that may throw (native module getter, `JSON.parse`) | `fromThrowable(fn, errorFn)` — returns a wrapped **function** |
| a function that may throw **synchronously before** returning its promise (async method on a possibly-missing native module) | `fromAsyncThrowable(fn, errorFn)` — `fromPromise(fn())` lets the sync throw escape |
| your own fallible helper | return `ok(value)` / `err(reason)` (`okAsync` / `errAsync` for async) |
| a promise that cannot reject (`setTimeout` wrapper) | `fromSafePromise(p)` — rare |

`errorFn` maps the `unknown` rejection to `E`. Keep it `(error) => error` when the consumer only
logs; narrow (`isAxiosError`, `instanceof`) only when a consumer branches on the reason.

```ts
// a missing native module must not throw above the navigator
const readInstaller = fromThrowable(() => DeviceInfoModule.installerPackageName);
const store = readInstaller().map(toStore).unwrapOr("ruStore");
```

## Every chain ends in a consumer

`map` / `mapErr` / `andThen` transform; they do not handle. A chain that stops on them silently
drops the error. Finish with one of:

| Need | Consumer |
| --- | --- |
| both branches yield a value | `.match(onOk, onErr)` (on `ResultAsync` returns a `Promise`) |
| a fallback value | `.unwrapOr(fallback)` |
| imperative early return | `if (r.isErr()) return; use(r.value)` — `isOk()` / `isErr()` narrow |
| fire-and-forget | `void fromPromise(p, errorFn)` — log inside `errorFn`, nothing else to do |

`void` marks a `ResultAsync` that is deliberately dropped. `_unsafeUnwrap` never appears in `src`.

## Chaining

| Method | Does |
| --- | --- |
| `map(f)` | transform the value |
| `mapErr(f)` | transform the error — not a logger, see mistakes |
| `andThen(f)` | next fallible step; `f` returns `Result` / `ResultAsync`, errors union, nesting flattens |
| `orElse(f)` | recover: turn an `Err` into `ok(...)` or a different `err(...)` |
| `andThrough(f)` | a step that may fail but whose value you do not need (validate, persist) — keeps the original value |
| `andTee(f)` / `orTee(f)` | side effect on the ok / err track; its outcome never changes the chain |
| `Result.combine([...])` / `ResultAsync.combine` | `Promise.all` for Results: first `Err` wins; `combineWithAllErrors` collects every error |

`ResultAsync` is thenable: `await` yields a `Result`, not the value. Step from sync to async with
`Result.asyncAndThen` / `asyncMap`; `ResultAsync.andThen` accepts either kind.

## safeTry — early return without the ladder

Three or more dependent steps where each `Err` must abort the whole thing: `yield*` unwraps an
`Ok` or returns the `Err` from the block.

```ts
const loadProfile = (token: string) =>
  safeTry(async function* () {
    const user = yield* fetchUser(token); // ResultAsync<User, ApiError>
    const prefs = yield* fetchPrefs(user.id); // ResultAsync<Prefs, ApiError>
    return ok({ user, prefs });
  }); // ResultAsync<{ user; prefs }, ApiError>
```

`function*` → `Result`, `async function*` → `ResultAsync`. Two steps → plain `andThen`.

## Where a Result is wrong or overkill

- **`queryFn` / `mutationFn`.** react-query is the Result there: a rejection drives `isError`,
  retries, the `meta.errorMessage` toast and `mutate`'s `onError`. Let the promise reject; a
  Result returned from a `queryFn` is a "success" with a hidden failure inside.
- **Absence is not failure.** A parse or lookup with nothing to say about *why* returns
  `T | null` (`toPushLink`); reach for `Result` when a caller acts on the reason.
- **`zod` `safeParse`** already returns a `{ success, data | error }` union. Lift it into
  `ok` / `err` only to chain with other Results.
- **A value you unwrap immediately.** `ok(x)` followed by `.value` two lines later is an `if`.
  `Result` is a return type across a function boundary, not a local-variable style.
- **Error taxonomies.** No `AppError` class hierarchy. `E = unknown` (`(error) => error`) when it
  is only logged; a literal union / tagged object when consumers branch — then `match` from
  ts-pattern on it.
- **Re-wrapping.** A helper that already returns a `Result` is not passed through `fromPromise`
  again — `andThen` it.

## Common mistakes

- `const v = await fromPromise(p, f)` then using `v` as the value — it is a `Result`; `match`,
  `unwrapOr` or `isOk()` first.
- `fromPromise(nativeModule.fn(), …)` — a missing module throws synchronously at `fn()`, outside
  the wrapper. `fromAsyncThrowable(() => nativeModule.fn(), …)()`.
- A chain ending in `map` / `mapErr` — nothing consumed the error.
- `mapErr(console.error)` — turns `E` into `void`; log inside `errorFn` or via `orTee`.
- `unwrapOr` on a failure the user should see. A toast belongs in the handler's `match` at the
  call site, never inside a hook or util.

Full API for the installed version: `node_modules/neverthrow/README.md`.
