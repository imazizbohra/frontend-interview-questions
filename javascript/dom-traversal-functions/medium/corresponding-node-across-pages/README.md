# Corresponding Node Across Pages

Real-time collaboration apps like Google Docs and Figma often sync actions between two webpages that render the same DOM structure. For example, hovering, selecting, or inspecting a node on one page might require highlighting the corresponding node on the other page.

Implement `correspondingNodeAcrossPages(rootA, rootB, target)`, which takes the root nodes of two pages with the same DOM structure and a `target` node inside `rootA`, then returns the node in `rootB` at the same relative position.

## Argument(s)

`correspondingNodeAcrossPages(rootA, rootB, target)`

- `rootA (Node)`: The root node of the first page.
- `rootB (Node)`: The root node of the second page. It has the same structure as rootA.
- `target (Node)`: A node that is guaranteed to be rootA itself or one of its descendants.

## Returns

Returns the node in `rootB` that appears at the same relative position as target in `rootA`.

## Example(s)

```ts
const pageA = new DOMParser().parseFromString(
  `
    <main>
      <header>
        <h1>Workspace</h1>
      </header>
      <section>
        <form>
          <button>Save</button>
        </form>
      </section>
    </main>
  `,
  'text/html',
);
const pageB = new DOMParser().parseFromString(
  `
    <main>
      <header>
        <h1>Workspace</h1>
      </header>
      <section>
        <form>
          <button>Save</button>
        </form>
      </section>
    </main>
  `,
  'text/html',
);

const target =
  pageA.body.children[0].children[1].firstElementChild.firstElementChild;

correspondingNodeAcrossPages(pageA.body, pageB.body, target);
// pageB.body.children[0].children[1].firstElementChild.firstElementChild
```

When identical subtrees repeat, matching by text or selectors is not enough. The node must be found by its position in the tree.

```ts
const pageA = new DOMParser().parseFromString(
  `
    <main>
      <section class="card">
        <header><h2>Todo</h2></header>
        <div><button>Sync</button></div>
      </section>
      <section class="card">
        <header><h2>Todo</h2></header>
        <div><button>Sync</button></div>
      </section>
    </main>
  `,
  'text/html',
);
const pageB = new DOMParser().parseFromString(
  `
    <main>
      <section class="card">
        <header><h2>Todo</h2></header>
        <div><button>Sync</button></div>
      </section>
      <section class="card">
        <header><h2>Todo</h2></header>
        <div><button>Sync</button></div>
      </section>
    </main>
  `,
  'text/html',
);

const target =
  pageA.body.children[0].children[1].lastElementChild.firstElementChild;

correspondingNodeAcrossPages(pageA.body, pageB.body, target);
// pageB.body.children[0].children[1].lastElementChild.firstElementChild
```

## Techniques

- Recursion
- DOM APIs
- Traversing descendants with `node.childNodes`

## Reference(s)

- Web API [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node)

## Solution(s)

```ts
// Approach 1: Recursion

export default function correspondingNodeAcrossPages(
  rootA: Node,
  rootB: Node,
  target: Node,
): Node {
  function traverse(rootA: Node, rootB: Node, target: Node): Node | null {
    if (rootA === target) {
      return rootB;
    }

    for (let i = 0; i < rootA.childNodes.length; i++) {
      const relativeNode = traverse(
        rootA.childNodes[i],
        rootB.childNodes[i],
        target,
      );

      if (relativeNode) {
        return relativeNode;
      }
    }

    return null;
  }

  return traverse(rootA, rootB, target) as Node;
}
```

```ts
// Approach 1: Path replay

export default function correspondingNodeAcrossPages(
  rootA: Node,
  rootB: Node,
  target: Node,
): Node {
  const path: Array<number> = [];
  let node = target;

  while (node !== rootA) {
    const parent = node.parentNode as Node;
    // Record each child index taken while walking from the target back to rootA.
    path.push(Array.prototype.indexOf.call(parent.childNodes, node));
    node = parent;
  }

  let correspondingNode = rootB;

  for (let index = path.length - 1; index >= 0; index--) {
    // Replay the recorded child indexes from rootB to land on the mirrored node.
    correspondingNode = correspondingNode.childNodes[path[index]];
  }

  return correspondingNode;
}
```
