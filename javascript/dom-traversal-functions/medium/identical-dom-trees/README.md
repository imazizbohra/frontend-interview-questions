# Identical DOM Trees

Implement a function `identicalDOMTrees` that checks if two DOM trees are identical. The function takes two DOM nodes as the root nodes of two DOM trees and returns a boolean result.

Two DOM trees are considered identical if they are structurally similar, and the DOM nodes in one tree have the exact same attributes as the nodes in the same relative positions in the other tree.

## Examples

Tree A and Tree B are considered the same.

```tsx
<!-- Tree A -->
<div>Hello World</div>

<!-- Tree B -->
<div>Hello World</div>
```

Tree C and Tree D are considered different.

```tsx
<!-- Tree C -->
<div class="header">Hello World</div>

<!-- Tree D -->
<div id="foo">Hello World</div>
```

## References

- Web API [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node)
- Node [isEqualNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/isEqualNode)
- Node [nodeType](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType)
- Web API [NamedNodeMap](https://developer.mozilla.org/en-US/docs/Web/API/NamedNodeMap)
- Web API [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element)
- Web API [Text](https://developer.mozilla.org/en-US/docs/Web/API/Text)
- Web API [CharacterData](https://developer.mozilla.org/en-US/docs/Web/API/CharacterData)

## Solutions

```ts
const isEqualNamedNodeMap = (map1: NamedNodeMap, map2: NamedNodeMap) => {
  if (map1.length !== map2.length) {
    return false;
  }

  for (const attr of map1) {
    const other = map2.getNamedItem(attr.name);

    if (!other || other.value !== attr.value) {
      return false;
    }
  }

  return true;
};

export default function identicalDOMTrees(nodeA: Node, nodeB: Node): boolean {
  // check for ref
  if (nodeA !== nodeB) {
    return false;
  }

  // check node type
  if (nodeA.nodeType !== nodeB.nodeType) {
    return false;
  }

  // check element node
  if (
    nodeA.nodeType === Node.ELEMENT_NODE &&
    nodeB.nodeType === Node.ELEMENT_NODE
  ) {
    // check tag name
    if ((nodeA as Element).localName !== (nodeB as Element).localName) {
      return false;
    }

    // check attributes
    const nodeAAttr = (nodeA as Element).attributes;
    const nodeBAttr = (nodeB as Element).attributes;
    if (!isEqualNamedNodeMap(nodeAAttr, nodeBAttr)) {
      return false;
    }
  }

  // check text node
  if (nodeA.nodeType === Node.TEXT_NODE && nodeB.nodeType === Node.TEXT_NODE) {
    if ((nodeA as Text).data !== (nodeB as Text).data) {
      return false;
    }
  }

  // check children length
  if (nodeA.childNodes.length !== nodeB.childNodes.length) {
    return false;
  }

  // check all the childs
  for (let i = 0; i < nodeA.childNodes.length; i++) {
    if (!identicalDOMTrees(nodeA.childNodes[i], nodeB.childNodes[i])) {
      return false;
    }
  }

  return true;
}
```
