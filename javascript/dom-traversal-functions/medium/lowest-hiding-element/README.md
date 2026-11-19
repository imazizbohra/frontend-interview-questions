# Lowest Hiding Element

Sometimes UI behavior is controlled by hiding a container instead of hiding each descendant individually. If several elements need to disappear together, it is useful to know the deepest element in the DOM tree that contains all of them.

Implement `lowestHidingElement(elements)`, which takes a non-empty array of DOM elements and returns the deepest element that would hide all of them if that element were hidden.

## Argument(s)

`lowestHidingElement(elements)`

`elements (Element[])`: A non-empty array of elements from the same document tree.

## Returns

Returns the deepest `Element` that contains every element in `elements`.

## Notes

- An element counts as hiding itself and all of its descendants.
- If one input element contains all the others, return that element.
- Do not mutate the DOM.
- Do not worry about Shadow DOM, iframes, detached trees, or existing CSS visibility state.
“Hidden” is only the motivation for the problem. The task is to find the shared deepest containing element.

## Example(s)

```ts
const page = new DOMParser().parseFromString(
  `
    <main>
      <section>
        <button>Save</button>
        <button>Cancel</button>
      </section>
      <aside>
        <button>Archive</button>
      </aside>
    </main>
  `,
  'text/html',
);

const section = page.body.firstElementChild.firstElementChild;
const saveButton = section.firstElementChild;
const cancelButton = section.lastElementChild;

lowestHidingElement([saveButton, cancelButton]);
// section
```

```ts
const page = new DOMParser().parseFromString(
  `
    <main>
      <nav>
        <ul>
          <li><a href="/docs">Docs</a></li>
          <li><a href="/api">API</a></li>
        </ul>
      </nav>
    </main>
  `,
  'text/html',
);

const list = page.body.firstElementChild.firstElementChild.firstElementChild;
const firstLink = list.firstElementChild.firstElementChild;

lowestHidingElement([list, firstLink]);
// list
```

## Reference(s)

- Array [every](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
- Node [parentElement](https://developer.mozilla.org/en-US/docs/Web/API/Node/parentElement)
- Node [contains](https://developer.mozilla.org/en-US/docs/Web/API/Node/contains)

## Solution(s)

```ts
// Approach 1

export default function lowestHidingElement(elements: Array<Element>): Element {
  let lca: Element = elements[0];

  elements.forEach((element) => {
    let node: Element | null = element;

    while (node) {
      if (elements.every((item) => node!.contains(item))) {
        lca = node; // found Lowest Common Ancestor, early exit
        break;
      }

      node = node.parentElement;
    }
  });

  return lca;
}
```

```ts
// Approach 2 

export default function lowestHidingElement(elements: Array<Element>): Element {
  let lowest = elements[0];

  for (let index = 1; index < elements.length; index++) {
    const element = elements[index];

    // Once lowest contains everything seen so far, only its ancestors can contain element too.
    while (!lowest.contains(element)) {
      lowest = lowest.parentElement as Element;
    }
  }

  return lowest;
}
```
