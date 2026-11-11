# useWindowSize

Implement a `useWindowSize` hook that returns the current height and width of the window (`window.innerHeight` and `window.innerWidth`). It should re-render the component if the window size changes.

## Argument(s)

Nothing.

## Returns

The hook returns an object with the following properties:

- `height: number`: Current height of the window
- `width: number`: Current width of the window

## Notes

- Read from `window.innerHeight` and `window.innerWidth`, not from screen.
- Measure once on mount so callers do not have to wait for the first resize event.
- Listen for `resize` events and clean up the listener when the hook unmounts.
- The hook assumes a browser-like environment where `window` exists.

## Example(s)

```tsx
export default function Component() {
  const screen = useWindowSize();

  return (
    <div>
      <p>The current window dimensions are:</p>
      <code>{JSON.stringify(screen, null, 2)}</code>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { useState, useEffect, useCallback } from "react";

interface WindowSize {
  height: number;
  width: number;
}

export default function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>({
    height: 0,
    width: 0,
  });

  const handleResize = useCallback(() => {
    setWindowSize({
      height: window.innerHeight,
      width: window.innerWidth,
    });
  }, []);

  useEffect(() => {
    handleResize();

    window.addEventListener("resize", handleResize);

    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return windowSize;
}
```
