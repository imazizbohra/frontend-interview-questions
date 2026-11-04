# jQuery.contains

`jQuery.contains()` is a small utility that checks whether one DOM element is a descendant of another.

Implement a jQuery object with a `contains(container, contained)` method that returns true if contained is a descendant of container, and false otherwise.

Unlike `Node.contains()`, this jQuery helper is self-exclusive, which means an element does not contain itself.

## Argument(s)

`jQuery.contains(container, contained)`

- container (Element): The DOM element that may contain another element.
- contained (Node | null): The DOM node that may be contained by container.

## Returns

Returns true if contained is an element descendant of container, and false otherwise.

## Example(s)

```ts
const doc = new DOMParser().parseFromString(
  `<div id="parent">
    <section>
      <button id="child">Click me</button>
    </section>
  </div>`,
  'text/html',
);

const parent = doc.getElementById('parent');
const child = doc.getElementById('child');

jQuery.contains(parent, child); // true
jQuery.contains(child, parent); // false
jQuery.contains(parent, parent); // false
```

## Notes

- container is guaranteed to be a DOM element.
- Only element nodes count as contained descendants. If contained is a text node or comment node, return false.
- Return false for the same element, ancestors, siblings, unrelated nodes, and disconnected trees.
- Do not use Node.contains() or compareDocumentPosition().

## Reference(s)

- jQuery [contains](https://api.jquery.com/jQuery.contains/)

## Techniques

- DOM tree traversal.
- Parent-pointer walking.

## Solution(s)

```ts
// Approach 1: bottom-up

interface JQueryStatic {
  contains: (container: Element, contained: Node | null) => boolean;
}

const jQuery: JQueryStatic = {
  contains(container: Element, contained: Node | null): boolean {
    if (container === contained) return false;

    if (contained == null || contained.nodeType !== Node.ELEMENT_NODE) {
      return false;
    }

    if (!container.isConnected || !contained.isConnected) {
      return false;
    }

    let current: Node | null = contained.parentNode;
    while (current != null) {
      if (current === container) return true;

      current = current.parentNode;
    }

    return false;
  },
};

export default jQuery;
```

```ts
// Approach 2: top-down

interface JQueryStatic {
  contains: (container: Element, contained: Node | null) => boolean;
}

const jQuery: JQueryStatic = {
  contains(container: Element, contained: Node | null): boolean {
    if (container === contained) return false;

    if (contained == null || contained.nodeType !== Node.ELEMENT_NODE) {
      return false;
    }

    if (!container.isConnected || !contained.isConnected) {
      return false;
    }

    const iterator = document.createNodeIterator(
      container,
      NodeFilter.SHOW_ELEMENT,
    );
    
    let node: Node | null = iterator.nextNode();
    while (node) {
      if (node === contained) return true;

      node = iterator.nextNode();
    }

    return false;
  },
};

export default jQuery;
```
