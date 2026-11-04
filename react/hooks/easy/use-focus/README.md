# useFocus

Implement a `useFocus` hook that enables focusing an element programmatically.

## Argument(s)

Nothing.

## Returns

The hook returns an array with two elements:

- A ref object that will be attached to the ref prop of the element to focus. This has the type `RefObject<T>`, where `T extends HTMLElement`
- A function `() => void` that can be called to programmatically focus the element

## Example(s)

```ts
export default function Component() {
  const [ref, focus] = useFocus();

  return (
    <div>
      <input ref={ref} type="text" />
      <button onClick={focus}>Focus input</button>
    </div>
  );
}
```

## Notes

Calling the function while no element is attached should do nothing.

## Reference(s)

- React [useRef](https://react.dev/reference/react/useRef)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { RefObject, useRef, useCallback } from "react";

export default function useFocus<T extends HTMLElement>(): [
  RefObject<T>,
  () => void,
] {
  const ref = useRef<T>(null);

  const focus = useCallback(() => {
    requestAnimationFrame(() => {
      ref.current?.focus();
    });
  }, []);

  return [ref, focus] as const;
}
```
