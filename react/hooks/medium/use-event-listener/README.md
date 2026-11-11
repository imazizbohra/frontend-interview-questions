# useEventListener

Implement a `useEventListener` hook that subscribes to browser events by attaching event listeners to DOM elements, the window, or media query lists.

## Argument(s)

- `eventName: string`: The event type to listen for
- `handler: (event) => void`: The event handler function
- `element: RefObject<T>`: The ref object of the element to attach the event listener to. If not provided, it defaults to window
- `options: boolean | AddEventListenerOptions`: The optional `AddEventListenerOptions` value used when attaching the listener

## Returns

Nothing.

## Notes

- If `element` is omitted, listen on `window`.
- The subscribed listener should call the latest `handler` passed to the hook.
- Clean up the `listener` when the component unmounts or when the event name, target ref, or options change.
- If the resolved target does not support `addEventListener`, do nothing.

## Example(s)

```tsx
export default function Component() {
  const buttonRef = useRef(null);

  useEventListener('click', () => console.log('Button clicked'), buttonRef, {
    once: true,
  });

  return (
    <div>
      <button ref={buttonRef}>Click me</button>
    </div>
  );
}
```

## Reference(s)

React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
import { RefObject, useEffect, useRef } from "react";

export default function useEventListener<
  KW extends keyof WindowEventMap,
  KH extends keyof HTMLElementEventMap & keyof SVGElementEventMap,
  KM extends keyof MediaQueryListEventMap,
  T extends HTMLElement | SVGAElement | MediaQueryList = HTMLElement,
>(
  eventName: KW | KH | KM,
  handler: (
    event:
      | WindowEventMap[KW]
      | HTMLElementEventMap[KH]
      | SVGElementEventMap[KH]
      | MediaQueryListEventMap[KM]
      | Event,
  ) => void,
  element?: RefObject<T>,
  options?: boolean | AddEventListenerOptions,
) {
  const handlerRef = useRef(handler);
  handlerRef.current = handler;

  useEffect(() => {
    const target: T | Window = element?.current ?? window;

    if (!(target instanceof EventTarget)) {
      return;
    }

    const listener: typeof handler = (event) => {
      handlerRef.current(event);
    };

    target.addEventListener(eventName, listener, options);

    return () => {
      target?.removeEventListener(eventName, listener, options);
    };
  }, [eventName, element, options]);
}
```
