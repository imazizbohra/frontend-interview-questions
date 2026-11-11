# useBreakpoint

Implement a `createBreakpoint` function that takes an object whose keys are breakpoint names and whose values are minimum widths, then returns a `useBreakpoint` hook that reports the current breakpoint name based on `window.innerWidth`.

## Argument(s)

The `createBreakpoint` function takes an object of key-value pairs where each key is a breakpoint name (string) and each value is the breakpoint's minimum width (number) in pixels.

## Returns

The `createBreakpoint` function returns the actual `useBreakpoint` hook. This hook takes no arguments and returns the current breakpoint name (string) whose minimum width is closest to, but not greater than, the current window width. Resizing the window will cause the hook to re-run and potentially return a different value depending on the window size.

## Notes

- Breakpoint object insertion order should not determine the result; compare the numeric widths.
- The active breakpoint is the largest minimum width that is less than or equal to `window.innerWidth`.
- Listen for `resize` events and clean up the listener when the hook unmounts.
- The breakpoint map is guaranteed to be non-empty.

## Example(s)

```tsx
const useBreakpoint = createBreakpoint({
  mobile: 0,
  tablet: 768,
  desktop: 1024,
});

export default function Component() {
  const breakpoint = useBreakpoint();

  return (
    <div>
      {breakpoint === 'mobile' && <p>Mobile</p>}
      {breakpoint === 'tablet' && <p>Tablet</p>}
      {breakpoint === 'desktop' && <p>Desktop</p>}
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
// Approach 1

import { useEffect, useState, useCallback } from "react";

export default function createBreakpoint<T extends Record<string, number>>(
  breakpoints: T,
): () => keyof T {
  const sortedBreakpoints = Object.entries(breakpoints).sort(
    (a, b) => a[1] - b[1],
  );

  const computeBreakpoint = (): string => {
    const innerWidth = window.innerWidth;

    return sortedBreakpoints.reduce((acc, [name, minWidth]) => {
      return innerWidth >= minWidth ? name : acc;
    }, "");
  };

  return function useBreakpoint() {
    const [breakpoint, setBreakpoint] = useState<keyof T>(computeBreakpoint);

    const handleResize = useCallback(() => {
      const breakpoint = computeBreakpoint();
      setBreakpoint(breakpoint);
    }, []);

    useEffect(() => {
      window.addEventListener("resize", handleResize);

      return () => window.removeEventListener("resize", handleResize);
    }, []);

    return breakpoint;
  };
}
```

```ts
// Approach 2

import { useEffect, useMemo, useState } from 'react';

export default function createBreakpoint<T extends Record<string, number>>(
  breakpoints: T,
): () => keyof T {
  return function (): keyof T {
    const [width, setWidth] = useState(window.innerWidth);

    useEffect(() => {
      function resize() {
        setWidth(window.innerWidth);
      }

      resize();
      window.addEventListener('resize', resize);

      return () => {
        window.removeEventListener('resize', resize);
      };
    }, []);

    const sortedBreakpoints = useMemo(
      // Sort once so breakpoint selection is deterministic even if the config
      // object was declared in a different order.
      () => Object.entries(breakpoints).sort((a, b) => a[1] - b[1]),
      [breakpoints],
    );

    return useMemo(
      () =>
        sortedBreakpoints.reduce(
          // Each match replaces the previous one, leaving the largest minimum
          // width that still fits the current viewport.
          (acc, [name, size]) => (width >= size ? name : acc),
          sortedBreakpoints[0][0],
        ),
      [sortedBreakpoints, width],
    );
  };
}
```
