---
name: neverthrow
description: >-
  Project convention and reference for error handling with the neverthrow library
  (Result / ResultAsync / ok / err / fromPromise / fromThrowable / fromAsyncThrowable /
  safeTry / combine). Consult whenever writing, refactoring, or reviewing .ts/.tsx code
  that can fail: a promise that may reject (fetch, any async API), a call that may throw
  (JSON.parse, a getter), a chain of dependent fallible steps, a fallback value on failure,
  a fire-and-forget side effect, or any spot where you are about to write try/catch, .catch()
  or throw. Also use it to decide when a Result is overkill (code a framework expects to
  reject, nullable results, an existing success/failure union) and how to consume one
  (match / unwrapOr / isErr). This project bans try/catch in src, so use Result here.
---

# neverthrow in this project

`neverthrow` is how failure is represented in this codebase. Already a dependency:

```ts
import { fromPromise, fromThrowable, fromAsyncThrowable, ok, err, safeTry } from "neverthrow";
```

## Why we bother

A `throw` is invisible in a signature: the caller cannot see that the function fails, and a
missed `catch` is a crash at runtime. `Result<T, E>` / `ResultAsync<T, E>` put the failure into
the type: the value cannot be read without deciding what happens on `Err`, and that decision
lives at the call site. `try/catch` does not appear in `src`.

## Where a Result comes from

Convert at the boundary — the moment you touch something that throws or rejects — and never
re-wrap further up.

| Source | Use |
| --- | --- |
| a promise that may reject (`fetch`, any async API) | `fromPromise(p, errorFn)` |
| a sync call that may throw (`JSON.parse`, a getter) | `fromThrowable(fn, errorFn)` — returns a wrapped **function** |
| a function that may throw **synchronously before** returning its promise (argument validation, a method on an object that may be missing) | `fromAsyncThrowable(fn, errorFn)` — `fromPromise(fn())` lets the sync throw escape |
| your own fallible helper | return `ok(value)` / `err(reason)` (`okAsync` / `errAsync` for async) |
| a promise that cannot reject (`setTimeout` wrapper) | `fromSafePromise(p)` — rare |

`errorFn` only maps the caught `unknown` to `E`; logging goes through `orTee`. Keep it
`(error) => error` when the caller only logs; narrow (`instanceof`, a type guard) only when a
caller branches on the reason.

```ts
const parseJson = fromThrowable((text: string): unknown => JSON.parse(text), (error) => error);
const settings = parseJson(raw).map(toSettings).unwrapOr(defaults);
```

## Every chain is returned or consumed

`map` / `mapErr` / `andThen` transform; they do not handle. A helper returns its chain and the
caller decides; a chain that is neither returned nor consumed silently drops the error. At the
call site finish with one of:

| Need | Consumer |
| --- | --- |
| both branches yield a value | `.match(onOk, onErr)` (on `ResultAsync` returns a `Promise`) |
| a fallback value | `.unwrapOr(fallback)` |
| imperative early return | `if (r.isErr()) return; use(r.value)` — `isOk()` / `isErr()` narrow |
| fire-and-forget | `void fromPromise(p, errorFn).orTee(log)` — log on the error track, nothing else to do |

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
    const feed = yield* fetchFeed(user.id, prefs.locale); // ResultAsync<Feed, ApiError>
    return ok({ user, prefs, feed });
  }); // ResultAsync<{ user; prefs; feed }, ApiError>
```

`function*` → `Result`, `async function*` → `ResultAsync`. Two steps → plain `andThen`.

## Where a Result is wrong or overkill

- **Code a framework expects to reject or throw.** Where a framework reads failure from a
  rejected promise or a thrown error (a data-fetching callback, a route loader, a test), let it
  reject; a Result returned there is a "success" with a hidden failure inside.
- **Absence is not failure.** A parse or lookup with nothing to say about *why* returns
  `T | null`; reach for `Result` when a caller acts on the reason.
- **A value that is already a success/failure union** (a schema parser's
  `{ success, data | error }`). Lift it into `ok` / `err` only to chain with other Results.
- **A value you unwrap immediately.** `ok(x)` followed by `.value` two lines later is an `if`.
  `Result` is a return type across a function boundary, not a local-variable style.
- **Error taxonomies.** No `AppError` class hierarchy. `E = unknown` (`(error) => error`) when it
  is only logged; a literal union / tagged object when callers branch.
- **Re-wrapping.** A helper that already returns a `Result` is not passed through `fromPromise`
  again — `andThen` it.

## Common mistakes

- `const v = await fromPromise(p, f)` then using `v` as the value — it is a `Result`; `match`,
  `unwrapOr` or `isOk()` first.
- `fromPromise(client.fn(), …)` — when `fn()` throws synchronously, it throws outside the
  wrapper. `fromAsyncThrowable(() => client.fn(), …)()`.
- A chain ending in `map` / `mapErr` that is not returned — nothing consumed the error.
- `mapErr(console.error)` — turns `E` into `void`; log via `orTee`.
- `unwrapOr` on a failure the user should see. Showing the error belongs in `match` at the call
  site, never inside a shared helper.

The installed version decides what exists: the full API is in
`node_modules/neverthrow/README.md` — verify there before using a method the codebase does not
use yet, rather than guessing.
