# getElementsByClassName

`getElementsByClassName()` is a method that exists on HTML `Documents` and `Elements` to return an `HTMLCollection` of descendant elements within the `Document`/`Element` that have the specified class names.

Implement a custom `Element.getElementsByClassName()` that is similar but slightly different:

- It is a pure function that takes an `element` and a `classNames` string containing one or more class names to match, separated by whitespace. For example, `getElementsByClassName(document.body, 'foo bar')`.
- Similar to `Element.getElementsByClassName()`, only descendants of the element argument are searched, not the element itself.
- Returns an array of Elements instead of an `HTMLCollection` of Elements.

Do not use `document.querySelectorAll()`, which would make the problem trivial. You will not be allowed to use it during real interviews.

## Examples

```ts
const doc = new DOMParser().parseFromString(
  `<div class="foo bar baz">
    <span class="bar baz">Span</span>
    <p class="foo baz">Paragraph</p>
    <div class="foo bar"></div>
  </div>`,
  'text/html',
);

getElementsByClassName(doc.body, 'foo bar');
// [div.foo.bar.baz, div.foo.bar] <-- This is an array of elements.
```

## Techniques

- Recursion
- DOM APIs
- Reading classes of element with `className` property
- Traversing descendants with `element.children`

## References

- Document [getElementsByClassName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByClassName)
- Element [getElementsByClassName](https://developer.mozilla.org/en-US/docs/Web/API/Element/getElementsByClassName)
- Element [className](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)

## Solutions

```ts
export default function getElementsByClassName(
  element: Element,
  classNames: string,
): Array<Element> {
  const returnValue: Array<Element> = [];

  function traverse(element: Element, classNames: string) {
    const classNamesSet = new Set(
      element.className.trim().replace(/\s+/g, " ").split(" "),
    );
    const classNameExists = classNames
      .trim()
      .replace(/\s+/g, " ")
      .split(" ")
      .every((className) => {
        return classNamesSet.has(className);
      });

    if (classNameExists) {
      returnValue.push(element);
    }

    for (const child of element.children) {
      traverse(child, classNames);
    }
  }

  for (const child of element.children) {
    traverse(child, classNames);
  }

  return returnValue;
}
```
