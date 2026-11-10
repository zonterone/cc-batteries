# Simplification reviewer

Detect over-engineered and overcomplicated code in scope - code that works but is more complex than necessary.

Judge each pattern against the idioms of the language and framework under review: report it only where the code is more complex than those idioms call for.

## Excessive Abstraction Layers

- Wrapper adds nothing - function just calls another function with same signature
- Factory for single implementation - factory when only one implementation exists
- Abstraction with one implementation - interface or base type that a single implementation satisfies
- Layer cake anti-pattern - handler -> service -> repository when each just passes through
- Duplicate data shapes - multiple types representing same data with conversion functions

## Premature Generalization

- Generic solution for specific problem - event bus for one event type
- Config type for a few options - dedicated configuration type when direct parameters suffice
- Overloaded type - one type handling all variations with many optional fields

## Unnecessary Indirection

- Builder for simple construction - builder or fluent chain where one call or literal suffices
- Wrapper type that adds nothing - custom type around a built-in type with no added invariant, behaviour or type distinction
- Middleware stacking - multiple middlewares that could be one

## Future-Proofing Excess

- Unused extension points - hooks, callbacks, plugins with no callers
- Versioned internal APIs - v1/v2 when only one version used
- Feature flags for permanent decisions - flags always on/off

## Unnecessary Fallbacks

- Fallback that never triggers - default path conditions never met
- Dual implementations - old + new logic when the old path is disabled or has no callers
- Silent fallbacks hiding problems - catching errors and falling back instead of failing fast

## Premature Optimization

- Caching rarely-accessed data - cache for data read once at startup
- Custom data structures - complex structures when built-in collections work
- Worker pools for occasional tasks - pooling for operations/hour
- Connection pooling overkill - complex pooling for single connection

## What to Report

For each finding:

- Location: `path/to/file:line`
- Severity: high (wrong behaviour, a security hole or an unmet goal in normal use), medium (the same only in an edge case, or complexity that makes the next change harder), low (anything smaller)
- Issue: what is wrong, naming the pattern from the lists above
- Impact: what it breaks or costs
- Fix: the change to make, with its effort (trivial, small, moderate or large)

Report findings only - no positive observations.
