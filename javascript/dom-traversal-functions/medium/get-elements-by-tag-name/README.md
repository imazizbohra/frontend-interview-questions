# getElementsByTagName

`getElementsByTagName()` is a method that exists on the Document and Element objects and returns an `HTMLCollection` of descendant elements within the `Document/Element` given a tag name.

Implement a custom `Element.getElementsByTagName()` that is similar but slightly different:

- It is a pure function that takes an element and a tag name string. For example, `getElementsByTagName(document.body, 'div')`.
- Similar to `Element.getElementsByTagName()`, only descendants of the element argument are searched, not the element itself.
- Returns an array of `Elements` instead of an `HTMLCollection` of `Elements`.
- Do not use `document.querySelectorAll()`, which would make the problem trivial.

## Examples

```ts
const doc = new DOMParser().parseFromString(
  `<div id="foo">
    <span>Span</span>
    <p>Paragraph</p>
    <div id="bar">Div</div>
  </div>`,
  'text/html',
);

getElementsByTagName(doc.body, 'div');
// [div#foo, div#bar] <-- This is an array of elements.
```

## Techniques

- Recursion
- DOM APIs
- Reading tagName
- Reading localName
- Traversing descendants with `element.children`

## References

- Web API [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element)
- Element [tagName](https://developer.mozilla.org/en-US/docs/Web/API/Element/tagName)
- Element [localName](https://developer.mozilla.org/en-US/docs/Web/API/Element/localName)

## Solutions

```ts
export default function getElementsByTagName(
  el: Element,
  tagName: string,
): Array<Element> {
  const elements: Array<Element> = [];

  function traverse(elem: Element) {
    if (elem.localName === tagName.toLowerCase()) {
      elements.push(elem);
    }

    for (const child of elem.children) {
      traverse(child);
    }
  }

  for (const child of el.children) {
    traverse(child);
  }

  return elements;
}
```
