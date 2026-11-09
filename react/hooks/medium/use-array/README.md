# useArray

Implement a `useArray` hook that manages an array of items with additional utility methods.

It is more convenient to use `useArray` than plain `useState`; with `useState`, you have to derive a new array and set state with that array for every update, which can be cumbersome.

The hook should work generically with arrays of any type.

## Argument(s)

defaultValue: The initial array of items

## Returns

The hook returns an object with the following properties:

- array: The current array of items
- set: (newArray) => void: A function that sets the array of items. It must have the same type as the setter returned by useState
- push: (item) => void: A function that adds an item to the end of the array
- remove: (index: number) => void: A function that removes an item from the array by index
- filter: (predicate) => void: A function that filters the array based on a predicate function. predicate must have the same type as the callback passed to `Array.prototype.filter`
- update: (index: number, newItem) => void: A function that replaces an item in the array at index
- clear: () => void: A function that clears the array

## Notes

- Each update should publish a new array instance so React can observe the state change.
- Do not mutate the current array directly.
- set should behave like the setter returned by useState, including accepting updater functions.
- clear() should replace the array with an empty array.

## Example(s)

```ts
const defaultValue = ['apple', 'banana'];

export default function Component() {
  const { array, push, update, remove, filter, set, clear } =
    useArray(defaultValue);

  return (
    <div>
      <p>Fruits: {array.join(', ')}</p>
      <button onClick={() => push('orange')}>Add orange</button>
      <button onClick={() => update(1, 'grape')}>
        Change second item to grape
      </button>
      <button onClick={() => remove(0)}>Remove first</button>
      <button onClick={() => filter((fruit) => fruit.includes('a'))}>
        Keep fruits containing 'a'
      </button>
      <button onClick={() => set(defaultValue)}>Reset</button>
      <button onClick={clear}>Clear list</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { Dispatch, SetStateAction, useState } from "react";

interface UseArrayReturn<T> {
  array: T[];
  set: Dispatch<SetStateAction<T[]>>;
  push: (element: T) => void;
  filter: (callback: (value: T, index: number, array: T[]) => boolean) => void;
  update: (index: number, newElement: T) => void;
  remove: (index: number) => void;
  clear: () => void;
}

export default function useArray<T>(defaultValue: T[]): UseArrayReturn<T> {
  const [state, setState] = useState<T[]>(defaultValue ?? []);

  const returnValue: UseArrayReturn<T> = {
    array: state,
    set: setState,
    push: (item: T) => {
      setState((prevState) => [...prevState, item]);
    },
    remove: (index: number) => {
      setState((prevState) => [
        ...prevState.slice(0, index),
        ...prevState.slice(index + 1),
      ]);
    },
    filter: (predicate) => {
      setState((prevState) => {
        return prevState.filter(predicate);
      });
    },
    update: (index: number, newItem) => {
      setState((prevState) => {
        prevState[index] = newItem;
        return prevState;
      });
    },
    clear: () => setState([]),
  };

  return returnValue;
}
```
