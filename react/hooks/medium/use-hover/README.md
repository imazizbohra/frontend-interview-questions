# useHover

Implement a `useHover` hook that tracks whether an element is being hovered.

## Argument(s)

Nothing.

## Returns

The hook returns an array with two elements:

- A ref callback function that will be attached to the `ref` prop of the element to track. This has the type `(element: T | null) => void`, where `T extends Element`
- A boolean that indicates whether the element is currently being hovered

## Notes

- The hook should track the element currently assigned to the returned ref callback.
- If React later assigns a different element, remove listeners from the old element and attach them to the new one.
- The boolean should start as false.
- Only mouse hover behavior is required; touch, pointer, and focus states are out of scope.

## Example(s)

```tsx
export default function Component() {
  const [ref, hovered] = useHover();

  return (
    <div>
      <div ref={ref}>{hovered ? 'Hovered' : 'Not hovered'}</div>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { RefCallback, useState, useRef, useEffect, useCallback } from "react";

export default function useHover<T extends Element>(): [
  RefCallback<T>,
  boolean,
] {
  const [element, setElement] = useState<T | null>(null);
  const [isHovered, setIsHovered] = useState(false);

  const refCallback: RefCallback<T> = useCallback((element: T | null) => {
    setElement(element);
  }, []);

  const handleMouseOver = useCallback(() => setIsHovered(true), []);

  const handleMouseLeave = useCallback(() => setIsHovered(false), []);

  useEffect(() => {
    if (!element || !(element instanceof EventTarget)) return;

    element.addEventListener("mouseenter", handleMouseOver);
    element.addEventListener("mouseleave", handleMouseLeave);

    return () => {
      element.removeEventListener("mouseenter", handleMouseOver);
      element.removeEventListener("mouseleave", handleMouseLeave);
    };
  }, [element]);

  return [refCallback, isHovered] as const;
}
```
