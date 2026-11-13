# getElementsByStyle

Implement a method `getElementsByStyle()` that finds DOM elements that are rendered by the browser using the specified style. It is similar to `Element.getElementsByClassName()` but with some differences:

- It is a pure function that takes an element, a property string, and a value string representing the style property/value pair to match on the element's descendants. For example, `getElementsByStyle(document.body, 'font-size', '12px')`.
- Similar to `Element.getElementsByClassName()`, only descendants of the element argument are searched, not the element itself.
- Returns an array of Elements instead of an `HTMLCollection` of `Elements`.

Do not use `document.querySelectorAll()`, which would make the problem trivial. You will not be allowed to use it during real interviews.

## Example(s)

```ts
const doc = new DOMParser().parseFromString(
  `<div>
    <span style="font-size: 12px">Span</span>
    <p style="font-size: 12px">Paragraph</p>
    <blockquote style="font-size: 14px">Blockquote</blockquote>
  </div>`,
  'text/html',
);

getElementsByStyle(doc.body, 'font-size', '12px');
// [span, p] <-- This is an array of elements.
```

## Techniques

- Recursion
- DOM APIs
- Reading computed styles with `getComputedStyle()`
- Traversing descendants with `element.children`

## Reference(s)

- HTMLElement [style](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)
- Web API [NodeIterator](https://developer.mozilla.org/en-US/docs/Web/API/NodeIterator)
- Web API [CSSStyleProperties](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleProperties)

## Solution(s)

```ts
// Approach 1

export default function getElementsByStyle(
  element: Element,
  property: string,
  value: string,
): Array<Element> {
  const elements: Array<Element> = [];

  function traverse(el: Element) {
    if (el == null) {
      return;
    }

    // Computed styles include values coming from stylesheets, not just inline styles.
    const computedStyles = getComputedStyle(el);
    if (computedStyles.getPropertyValue(property) === value) {
      elements.push(el);
    }

    for (const child of el.children) {
      traverse(child);
    }
  }

  for (const child of element.children) {
    traverse(child);
  }

  return elements;
}
```

```ts
// Approach 2

export default function getElementsByStyle(
  element: Element,
  property: string,
  value: string,
): Array<Element> {
  const returnValue: Element[] = [];

  const root = element;
  const iterator = document.createNodeIterator(root, NodeFilter.SHOW_ELEMENT);

  let node: Node | null = iterator.nextNode();
  while (node) {
    const element = node as HTMLElement;
    const styles = element.style;

    if (element !== root && styles.getPropertyValue(property) === value) {
      returnValue.push(element);
    }

    node = iterator.nextNode();
  }

  return returnValue;
}
```
