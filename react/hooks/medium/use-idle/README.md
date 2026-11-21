# useIdle

Implement a `useIdle` hook that detects user inactivity for a specified duration. It listens for user interactions like mouse movements, keyboard presses, and touch events, and resets a timer whenever activity is detected.

The user is considered idle if none of the configured `events` are dispatched within `ms` milliseconds. Additionally, if the user has just returned to this tab from another tab, the user should not be considered idle.

## Arguments

- `ms: number`: The number of milliseconds to wait before considering the user idle. This defaults to 60_000 (1 minute)
- `initialState: boolean`: The initial `idle` state. This defaults to `false`
- `events: (keyof WindowEventMap)[]`: An array of event types that count as user activity. The default value is provided as the `DEFAULT_EVENTS` constant

## Returns

The hook returns a boolean value that indicates whether the user is idle.

## Notes

- Any configured activity event should mark the user active and restart the idle timer.
- When the document becomes visible again after a visibilitychange, treat that as activity.
- Switching away from the tab should not by itself mark the user active.
- Clean up all timers and event listeners when the component unmounts.

## Examples

```tsx
export default function Component() {
  const idle = useIdle();

  return (
    <div>
      <p>Status: {idle ? 'Idle' : 'Active'}</p>
    </div>
  );
}
```

## References

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solutions

```ts
import { useEffect, useRef, useState } from "react";

const DEFAULT_EVENTS: (keyof WindowEventMap)[] = [
  "mousemove",
  "mousedown",
  "resize",
  "keydown",
  "touchstart",
  "wheel",
];

export default function useIdle(
  ms = 60_000,
  initialState = false,
  events: (keyof WindowEventMap)[] = DEFAULT_EVENTS,
): boolean {
  const [isIdle, setIsIdle] = useState(initialState);
  const timerIdRef = useRef<number | undefined>(undefined);

  const listener = () => {
    if (document.visibilityState === "hidden") {
      return;
    }

    setIsIdle(false);

    clearTimeout(timerIdRef.current);
    timerIdRef.current = setTimeout(() => {
      timerIdRef.current = undefined;
      setIsIdle(true);
    }, ms);
  };

  useEffect(() => {
    listener();

    events.forEach((event) => {
      window.addEventListener(event, listener);
    });

    document.addEventListener("visibilitychange", listener);

    return () => {
      clearTimeout(timerIdRef.current);

      events.forEach((event) => {
        window.removeEventListener(event, listener);
      });

      document.removeEventListener("visibilitychange", listener);
    };
  }, []);

  return isIdle;
}
```
