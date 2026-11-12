---
name: react-code-style
description: Recommendations for writing React code - derived state, Effects, re-renders and memoization. MUST be consulted whenever writing, refactoring, reviewing or debugging React components or Hooks.
---

# React code style

**Derive** what can be calculated, **colocate** what must be stored, and write an Effect only to **synchronize** with a system outside React.

## How to apply

- New code follows these recommendations even inside a file that breaks them: they outrank the style of the surrounding code.
- Existing code changes only as far as the task needs: the lines the task must alter, and code the new lines cannot build on without breaking a recommendation. List the other violations you saw in your reply, each with the fix you would make.
- Behavior the user sees stays as specified. When a recommendation can only be met by changing it, keep the behavior and propose the change.
- **Hard guardrail** marks a recommendation without exceptions. When you cannot satisfy one, stop and ask the user, explaining why.

## Project facts

Read them from `package.json` and the bundler or framework config before the first React edit in a project.

- **React version.** `useEffectEvent` exists from 19.2. `ref` is a plain prop from 19; below 19 pass a named prop such as `inputRef`.
- **React Compiler.** On when `babel-plugin-react-compiler` is installed or the config enables it (`reactCompilerPreset` in Vite, `reactCompiler` in Next.js).
- **Data layer.** Fetch through what the project already has: a framework loader, TanStack Query, SWR. Adding a fetching library is the user's decision. With none, fetch in an Effect inside a custom Hook, with a cleanup that sets an `ignore` flag.

## State

- State is only what changes over time and cannot be calculated. Everything else is a `const` computed during rendering.
- **Hard guardrail:** an Effect never sets state calculated from props or other state. A setter called only inside an Effect marks state to delete and derive.
- What the component remembers but never renders (a timer ID, a DOM node, a connection opened by an Effect) lives in a ref, read and written in handlers and Effects only.
- State lives in the component that uses it; shared state lives in the closest common parent and no higher.
- To reset state when a prop changes, in order of preference: `key` on the component; a derived value in place of the state; a comparison with the previous prop during rendering, as below. An Effect is not on the list.

```tsx
const [prevItems, setPrevItems] = useState(items);
if (items !== prevItems) {
  setPrevItems(items);
  setPage(0);
}
```

## Effects

- An Effect synchronizes with an external system: a browser API, the network, a non-React widget. Work caused by a specific interaction goes in its event handler; a value needed for rendering is computed in the render body.
- One Effect is one synchronization process: the body starts it, the cleanup stops it. Unrelated processes get separate Effects.
- **Hard guardrail:** an Effect survives start → stop → start, which Strict Mode runs in development. A broken double run is fixed by a cleanup, not by a ref or flag that skips the second run.
- **Hard guardrail:** every `react-hooks` lint error is fixed by changing the code, and the dependency array always matches what the Effect reads. To drop a dependency: an updater function (`setCount((count) => count + 1)`), an object or function created inside the Effect, or an Effect Event.
- An Effect Event (`useEffectEvent`) holds the part of an Effect that must see fresh props and state without re-running it. Declare it next to its Effect, call it only from Effects, keep it out of dependency arrays. The value whose change is the event stays a dependency and is passed in as an argument, as below.

```tsx
const onVisit = useEffectEvent((visited: string) => {
  logVisit(visited, cart.length);
});

useEffect(() => {
  onVisit(url);
}, [url]);
```

## Re-renders

- Plain code is the default. `memo`, `useMemo` and `useCallback` answer a known cost: an interaction measured slow, or a component known to be expensive. With React Compiler on, write none.
- Restructure first. Move state down into the component that reads it. When state belongs to a wrapper around expensive content, the wrapper owns the state and takes the content as `children`: elements created by the parent are skipped when the wrapper re-renders.
- `memo` skips a render only when every prop is identical: memoize each object, array and function prop of a `memo` component, and nothing else.
- When an expensive component's props do change on rapid input, wrap it in `memo` and pass it `useDeferredValue(value)` (React 18 and later). The input updates at once; the expensive render follows in the background.
- **Hard guardrail:** every component is declared at module level. One declared inside another is a new type on each render, so React remounts it and its state is lost.
- **Hard guardrail:** a `key` is a stable ID from the data, assigned once when the item is loaded or created. It is never produced in the render body, and the array index serves only a list that never changes order or length.
