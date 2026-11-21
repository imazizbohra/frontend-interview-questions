# Immutable Array

In state containers, undo stacks, and other history-aware workflows, applications often need to derive a new array state without mutating the previous one.

Implement an `ImmutableArray` class that wraps a plain array and exposes a small set of array-like update methods. Every update should return a new `ImmutableArray` instance, leaving the current instance unchanged so branching from old versions stays safe.

This question is intentionally interview-scoped. Use familiar array behavior, but you do not need to match every native `Array` edge case.

## Examples

```ts
const base = new ImmutableArray(['draft', 'review']);

const next = base.push('published');

base.toArray(); // ['draft', 'review']
next.toArray(); // ['draft', 'review', 'published']
base !== next; // true
```

```ts
const history = new ImmutableArray([1, 2, 3]);

const withoutLast = history.pop();
const withReplacement = history.set(1, 20);

history.toArray(); // [1, 2, 3]
withoutLast.toArray(); // [1, 2]
withReplacement.toArray(); // [1, 20, 3]
```

```ts
const base = new ImmutableArray(['a', 'b']);

const left = base.push('c');
const right = base.set(0, 'z');

left.toArray(); // ['a', 'b', 'c']
right.toArray(); // ['z', 'b']
base.toArray(); // ['a', 'b']
```

## Notes

- Do not mutate the input array or the current `ImmutableArray` instance.
- Store item values by reference. You do not need deep cloning.
- You may assume `set(index, value)` receives a valid non-negative index.
- You do not need to support sparse arrays, negative indices, iterators, or bracket access like `array[0]`.

## References

- Concept [Immutable](https://developer.mozilla.org/en-US/docs/Glossary/Immutable)

## Solutions

```ts
// Approach 1

interface IImmutableArray<T = unknown> {
  readonly length: number;
  get(index: number): T | undefined;
  push(value: T): ImmutableArray<T>;
  pop(): ImmutableArray<T>;
  set(index: number, value: T): ImmutableArray<T>;
  toArray(): Array<T>;
}

export default class ImmutableArray<T = unknown> implements IImmutableArray<T> {
  #array: Array<T>;

  constructor(items: Array<T> = []) {
    this.#array = [...items];
  }

  get length(): number {
    return this.#array.length;
  }

  get(index: number): T | undefined {
    return this.#array[index];
  }

  push(value: T): ImmutableArray<T> {
    const clone = this.toArray();
    clone.push(value);
    return new ImmutableArray(clone);
  }

  pop(): ImmutableArray<T> {
    const clone = this.toArray();
    clone.pop();
    return new ImmutableArray(clone);
  }

  set(index: number, value: T): ImmutableArray<T> {
    const clone = this.toArray();
    clone[index] = value;
    return new ImmutableArray(clone);
  }

  toArray(): Array<T> {
    return Array.from(this.#array);
  }
}
```

```ts
// Approach 2

export interface IImmutableArray<T = unknown> {
  readonly length: number;
  get(index: number): T | undefined;
  push(value: T): ImmutableArray<T>;
  pop(): ImmutableArray<T>;
  set(index: number, value: T): ImmutableArray<T>;
  toArray(): Array<T>;
}

export default class ImmutableArray<T = unknown> implements IImmutableArray<T> {
  _items: Array<T>;

  constructor(items: Array<T> = []) {
    this._items = items.slice();
  }

  get length(): number {
    return this._items.length;
  }

  get(index: number): T | undefined {
    return this._items[index];
  }

  push(value: T): ImmutableArray<T> {
    return new ImmutableArray(this._items.concat(value));
  }

  pop(): ImmutableArray<T> {
    return new ImmutableArray(this._items.slice(0, -1));
  }

  set(index: number, value: T): ImmutableArray<T> {
    const nextItems = this._items.slice();
    nextItems[index] = value;

    return new ImmutableArray(nextItems);
  }

  toArray(): Array<T> {
    return this._items.slice();
  }
}
```
