# useMap

Implement a `useMap` hook that manages a JavaScript `Map` of items with additional convenient utility methods.

It is more convenient to use `useMap` instead of plain `useState` because with `useState`, you would always have to create a new `Map`, mutate it, then set state to use the new map, which can be quite cumbersome.

The hook should work generically with keys and values of any type.

## Argument(s)

initialState: The initial map of items. This can be either a Map instance or an array of key-value pairs

## Returns

The hook returns an object with the following properties:

- map: The current map of items
- set: (key, value) => void: A function that sets a key to value in the map
- setAll: (entries) => void: A function that sets the key-value pairs in entries in the map
- remove: (key) => void: A function that removes a key-value pair from the map by key
- reset: () => void: A function that resets the map to an empty map

## Notes

- Each update should publish a new Map instance so React can observe the state change.
- Do not mutate the current map object directly.
- setAll(entries) replaces the whole map; it does not merge with the existing map.
- reset() returns to an empty map.

## Example(s)

```tsx
export default function Component() {
  const { map, set, setAll, reset, remove } = useMap([['key', '🆕']]);

  return (
    <div>
      <button onClick={() => set(String(Date.now()), '📦')}>Add</button>
      <button
        onClick={() =>
          setAll([
            ['hello', '👋'],
            ['data', '📦'],
          ])
        }>
        Set new data
      </button>
      <button onClick={reset}>Reset</button>
      <button onClick={() => remove('hello')} disabled={!map.has('hello')}>
        Remove "hello"
      </button>
      <pre>
        Map (
        {Array.from(map.entries()).map(([key, value]) => (
          <Fragment key={key}>{`\n  ${key}: ${value}`}</Fragment>
        ))}
        <br />)
      </pre>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState } from "react";

type MapOrEntries<K, V> = Map<K, V> | [K, V][];

interface UseMapReturn<K, V> {
  /** The map object. */
  map: Readonly<Map<K, V>>;
  /** Set a key-value pair in the map. */
  set: (key: K, value: V) => void;
  /** Set all key-value pairs in the map. */
  setAll: (entries: MapOrEntries<K, V>) => void;
  /** Remove a key-value pair from the map. */
  remove: (key: K) => void;
  /** Reset the map to an empty state. */
  reset: () => void;
}

export default function useMap<K, V>(
  initialState: MapOrEntries<K, V> = new Map(),
): UseMapReturn<K, V> {
  const [map, setMap] = useState<Map<K, V>>(new Map(initialState));

  const returnValue: UseMapReturn<K, V> = {
    map: map,
    set: (key, value) => {
      map.set(key, value);
      setMap(new Map(map.entries()));
    },
    setAll: (entries: MapOrEntries<K, V>) => {
      const arr = Array.isArray(entries)
        ? entries
        : Array.from(entries.entries());

      for (let [key, value] of arr) {
        map.set(key, value);
      }

      setMap(new Map(map.entries()));
    },
    remove: (key) => {
      map.delete(key);
      setMap(new Map(map.entries()));
    },
    reset: () => {
      setMap(new Map());
    },
  };

  return returnValue;
}
```
