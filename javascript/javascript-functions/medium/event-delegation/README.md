# Event Delegation

Event delegation lets you attach a single listener to a parent element and handle interactions for matching descendants as events bubble up through the DOM. This is useful when a container has many interactive children, such as action lists, menus, tables, or todo items, because it avoids attaching one listener per child. It also works well when matching elements are added or removed dynamically, since the parent listener can continue handling future descendants without needing to be rewired.

Implement `delegateEvent(container, eventName, selector, listener)`.

The utility should attach one listener to `container` and invoke `listener(event, matchedElement)` whenever an event of type `eventName` originates from:

- A descendant that matches selector.
- A descendant inside an element that matches selector.

## Returns

Returns a cleanup function that removes the delegated listener from container.

## Examples

```ts
const actions = document.querySelector('#actions');

const cleanup = delegateEvent(
  actions,
  'click',
  'button[data-action]',
  (event, matchedElement) => {
    console.log(matchedElement.getAttribute('data-action'));
  },
);

// Clicking this nested <span>:
// <button data-action="delete"><span>Delete</span></button>
// should log: "delete"

cleanup();
```

## Notes

- Only descendants of `container` should trigger `listener`.
- If `container` itself matches `selector`, direct events on `container` should still be ignored.
- Ignore events whose original target is not an `Element`.
- `container` may belong to another document or window, such as an iframe.
- You do not need to support multiple selectors, capture-phase listeners, or non-bubbling events in this question.

## References

- Node [ownerDocument](https://developer.mozilla.org/en-US/docs/Web/API/Node/ownerDocument)
- Document [defaultView](https://developer.mozilla.org/en-US/docs/Web/API/Document/defaultView)

## Solutions

```ts
export type DelegatedEventListener = (
  event: Event,
  matchedElement: Element,
) => void;

export default function delegateEvent(
  container: Element,
  eventName: string,
  selector: string,
  listener: DelegatedEventListener,
): () => void {
  function handler(this: any, event: Event) {
    const { target } = event;

    const ElementConstructor = container.ownerDocument.defaultView?.Element;
    if (
      ElementConstructor == null ||
      !(event.target instanceof ElementConstructor)
    ) {
      return;
    }

    const matchedElement = (target as Element).closest(selector);
    if (
      matchedElement == null ||
      matchedElement === container ||
      !container.contains(matchedElement)
    ) {
      return;
    }

    listener.call(this, event, matchedElement);
  }

  container.addEventListener(eventName, handler);

  return () => container.removeEventListener(eventName, handler);
}
```
