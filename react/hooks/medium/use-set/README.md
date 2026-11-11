# useSet

Implement a `useSet` hook that manages a JavaScript `Set` of items with additional utility methods.

It is more convenient to use `useSet` instead of plain useState because with useState, you would always have to create a new `Set`, mutate it, then set state to use the new set, which can be quite cumbersome.

The hook should work generically with items of any type.

## Argument(s)

`initialState`: The initial Set of items

## Returns

The hook returns an object with the following properties:

- `set`: The current set of items
- `add: (item) => void`: A function that adds item to the set
- `remove: (item) => void`: A function that removes item from the set
- `toggle: (item) => void`: A function that toggles the presence of item in the set
- `reset: () => void`: A function that resets the set to initialState
- `clear: () => void`: A function that removes all items from the set

## Notes

- Each update should publish a new `Set` instance so React can observe the state change.
- Do not mutate the current `Set` object directly.
- `reset()` restores the exact supplied `initialState` object
- `clear()` replaces the state with a new empty set.
- `add`, `remove`, and `toggle` should derive their next state from the latest previous set.

## Example(s)

```tsx
export default function Component() {
  const { set, add, remove, toggle, reset, clear } = useSet(new Set(['hello']));

  return (
    <div>
      <button onClick={() => add(Date.now().toString())}>Add</button>
      <button onClick={() => remove('hello')} disabled={!set.has('hello')}>
        Remove 'hello'
      </button>
      <button onClick={() => toggle('hello')}>Toggle hello</button>
      <button onClick={reset}>Reset</button>
      <button onClick={clear}>Clear</button>
      <pre>{JSON.stringify(Array.from(set), null, 2)}</pre>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useCallback } from "react";

export interface UseSetReturn<T> {
  set: Readonly<Set<T>>;
  add: (key: T) => void;
  remove: (key: T) => void;
  toggle: (key: T) => void;
  reset: () => void;
  clear: () => void;
}

export default function useSet<T>(
  initialState = new Set<T>(),
): UseSetReturn<T> {
  const [uniqueSet, setUniqueSet] = useState<Set<T>>(initialState);

  const add = useCallback((key: T) => {
    setUniqueSet((prevState) => {
      const newSet = new Set(prevState);
      newSet.add(key);
      return newSet;
    });
  }, []);

  const remove = useCallback((key: T) => {
    setUniqueSet((prevState) => {
      const newSet = new Set(prevState);
      newSet.delete(key);
      return newSet;
    });
  }, []);

  const toggle = useCallback((key: T) => {
    setUniqueSet((prevState) => {
      const newSet = new Set(prevState);

      if (prevState.has(key)) {
        newSet.delete(key);
      } else {
        newSet.add(key);
      }

      return newSet;
    });
  }, []);

  const reset = useCallback(() => {
    setUniqueSet(initialState);
  }, [initialState]);

  const clear = useCallback(() => {
    setUniqueSet(new Set());
  }, []);

  return {
    set: uniqueSet,
    add,
    remove,
    toggle,
    reset,
    clear,
  };
}
```
