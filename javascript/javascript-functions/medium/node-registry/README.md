# Node Registry

Browser extensions, visual editors, and DOM inspectors often need an in-memory registry that can associate state or data with specific DOM nodes. For example, an inspector panel might want to remember whether a node is highlighted or which inspector tab is open for it without relying on selectors or mutating the HTML markup.

Implement a `NodeRegistry` class that acts as that registry: it stores values keyed by DOM `Node` objects.

Values should be looked up by node identity, not by text, tag name, attributes, or position in the tree.

## Examples

```ts
const registry = new NodeRegistry();
const button = document.createElement('button');

registry.set(button, { panel: 'styles', highlighted: true });

registry.has(button); // true
registry.get(button); // { panel: 'styles', highlighted: true }
```

The registry should also work for non-element nodes such as text and comment nodes.

```ts
const registry = new NodeRegistry();
const textNode = document.createTextNode('Draft');

registry.set(textNode, { warning: 'stale copy' });

registry.get(textNode); // { warning: 'stale copy' }
```

Two different nodes can look identical and still need separate entries.

```ts
const registry = new NodeRegistry();
const first = document.createElement('div');
const second = document.createElement('div');

registry.set(first, 'left panel');
registry.set(second, 'right panel');

registry.get(first); // 'left panel'
registry.get(second); // 'right panel'
```

## Notes

- Keys are DOM `Node` objects matched by reference identity.
- Values can be any type and should be stored as-is.
- Storing `undefined` is valid, so `has()` is needed to distinguish "missing" from "present but undefined".
- Inputs are guaranteed valid.
- All `Node` subtypes are in scope.
- Do not use `Map` or `WeakMap`.
- You may attach internal hidden bookkeeping to nodes.
- You do not need `delete()`, `clear()`, iteration, size tracking, serialization, or garbage-collection behavior.

## References

- Concept [Middleware](https://developer.mozilla.org/en-US/docs/Glossary/Middleware)
- Design Pattern [middleware](https://www.patterns.dev/vanilla/mediator-pattern/)

## Solutions

```ts
type NodeWithPrivateId = Node & Record<string, number | undefined>;

interface INodeRegistry {
  has(node: Node): boolean;
  get(node: Node): unknown | undefined;
  set(node: Node, value: unknown): void;
}

let nextRegistryInstanceId = 0;

export default class NodeRegistry implements INodeRegistry {
  _entries: Record<number, unknown>;
  _present: Record<number, boolean>;
  _nextNodeId: number;
  _nodeIdKey: string;

  constructor() {
    this._entries = Object.create(null);
    // Track presence separately so `set(node, undefined)` still counts as existing.
    this._present = Object.create(null);
    this._nextNodeId = 0;
    // Give each registry instance its own hidden property to avoid cross-registry collisions.
    this._nodeIdKey = `__nodeRegistry${nextRegistryInstanceId++}__`;
  }

  has(node: Node): boolean {
    const nodeId = this._getNodeId(node);

    return nodeId !== undefined && this._present[nodeId] === true;
  }

  get(node: Node): unknown | undefined {
    const nodeId = this._getNodeId(node);

    if (nodeId === undefined || this._present[nodeId] !== true) {
      return undefined;
    }

    return this._entries[nodeId];
  }

  set(node: Node, value: unknown): void {
    let nodeId = this._getNodeId(node);

    if (nodeId === undefined) {
      nodeId = this._nextNodeId++;

      Object.defineProperty(node, this._nodeIdKey, {
        value: nodeId,
        configurable: true,
      });
    }

    this._entries[nodeId] = value;
    this._present[nodeId] = true;
  }

  _getNodeId(node: Node): number | undefined {
    const nodeWithPrivateId = node as NodeWithPrivateId;

    return nodeWithPrivateId[this._nodeIdKey];
  }
}
```
