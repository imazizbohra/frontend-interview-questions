# useObject

Implement a `useObject` hook that manages JavaScript Object state. A plain object, also known as a POJO (Plain Old JavaScript Object), is a key-value store initialized with {}.

When the state setter is called with an object, it is merged with the existing object.

It is more convenient to use `useObject` instead of plain `useState` because with `useState`, you would always have to create a new object, mutate it, then set state to use the new object (or create a new object via object spread), which can be quite cumbersome.

## Argument(s)

`initialValue`: The initial value of the object

## Returns

The hook returns an array with two elements:

- The current object state
- An updater function that can update the object state

The updater function can receive either:

- An object that will be merged with the current state object
- An updater function that receives the current state object and returns an object that will be merged with the current state object

## Notes

- Updates should shallow-merge into the current object; they should not replace the object entirely.
- The updater-function form should derive its patch from the latest previous state.
- Only plain object patches are valid. If a direct patch or the value returned by an updater is an array, null, Date, or another non-plain object, throw an error with the message `Invalid new state`.

## Example(s)

```tsx
export default function Component() {
  const [record, setRecord] = useObject({ a: 1, b: 2 });

  return (
    <div>
      <pre>{JSON.stringify(record, null, 2)}</pre>
      <button onClick={() => setRecord((prev) => ({ a: prev.a + 1 }))}>
        Increase a
      </button>
      <button onClick={() => setRecord((prev) => ({ b: prev.b + 1 }))}>
        Increase b
      </button>
      <button onClick={() => setRecord(() => ({ c: 3 }))}>Add c</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState } from "react";

function isPlainObject(value: unknown): boolean {
  if (value === null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);

  return proto === Object.prototype || proto === null;
}

type UseObjectUpdater<T extends Record<string | number | symbol, any>> = (
  partialOrUpdaterFunction: Partial<T> | ((prev: T) => Partial<T>),
) => void;

export default function useObject<
  T extends Record<string | number | symbol, any>,
>(initialValue: T): [T, UseObjectUpdater<T>] {
  const [pojo, setPojo] = useState(initialValue);

  const updaterFn: UseObjectUpdater<T> = (partialOrUpdaterFunction) => {
    if (typeof partialOrUpdaterFunction === "function") {
      setPojo((prevState) => {
        const newState = partialOrUpdaterFunction(prevState);

        if (!isPlainObject(newState)) {
          throw Error("Invalid new state");
        }

        return { ...prevState, ...newState };
      });
      return;
    }

    if (isPlainObject(partialOrUpdaterFunction)) {
      setPojo((prevState) => {
        return { ...prevState, ...partialOrUpdaterFunction };
      });
    } else {
      throw Error("Invalid new state");
    }
  };

  return [pojo, updaterFn] as const;
}
```
