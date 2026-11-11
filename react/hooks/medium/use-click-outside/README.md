# useClickOutside

Implement a `useClickOutside` hook that detects clicks outside of a specified element.

## Argument(s)

- `ref: RefObject<T>`: The ref object of the element to detect clicks outside of
- `handler: (event) => void`: The event handler function to call when a click outside is detected
- `eventType: 'mousedown' | 'touchstart'`: The event type to listen for. Defaults to `mousedown`
- `eventOptions: boolean | AddEventListenerOptions`: The AddEventListenerOptions options object that specifies characteristics about the event listener. This is an optional argument

## Returns

Nothing.

## Notes

- Only call handler when the event target is a connected DOM node outside `ref.current`; ignore missing, non-node, or disconnected targets.
- After a rerender, outside events should call the latest handler passed to the hook.
- Remove the listener when the component unmounts or when the event type, ref, or listener options change.

## Example(s)

```tsx
export default function Component() {
  const target = useRef(null);
  useClickOutside(target, () => console.log('Clicked outside'));

  return (
    <div>
      <div ref={target}>Click outside me</div>
      <div>Maybe here?</div>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
// Approach 1

import { RefObject, useEffect } from "react";

export default function useClickOutside<T extends HTMLElement>(
  ref: RefObject<T>,
  handler: (event: MouseEvent | TouchEvent | FocusEvent) => void,
  eventType: "mousedown" | "touchstart" = "mousedown",
  eventListenerOptions?: boolean | AddEventListenerOptions,
) {
  useEffect(() => {
    const handleClick = (event: MouseEvent | TouchEvent | FocusEvent) => {
      const target = event.target;

      if (target instanceof Node && !ref.current.contains(target)) {
        handler(event);
      }
    };

    window.addEventListener(eventType, handleClick, eventListenerOptions);

    return () =>
      window.removeEventListener(eventType, handleClick, eventListenerOptions);
  }, []);
}
```

```ts
// Approach 2

import { RefObject, useEffect, useRef } from 'react';

export default function useClickOutside<T extends HTMLElement>(
  ref: RefObject<T | null>,
  handler: (event: MouseEvent | TouchEvent | FocusEvent) => void,
  eventType: 'mousedown' | 'touchstart' = 'mousedown',
  eventListenerOptions: boolean | AddEventListenerOptions = {},
) {
  // Keep the DOM listener stable while still invoking the newest callback.
  const latestHandler = useRef(handler);
  latestHandler.current = handler;

  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent | FocusEvent) => {
      const target = event.target as Node;
      // Ignore events from nodes React has already detached from the document.
      if (!target || !target.isConnected) {
        return;
      }

      const outside = ref.current && !ref.current.contains(target);
      if (!outside) {
        return;
      }

      latestHandler.current(event);
    };

    window.addEventListener(eventType, listener, eventListenerOptions);

    return () => {
      window.removeEventListener(eventType, listener, eventListenerOptions);
    };
  }, [ref, eventType, eventListenerOptions]);
}
```
