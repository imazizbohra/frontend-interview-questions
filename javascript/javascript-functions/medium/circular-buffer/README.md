# Circular Buffer

When storing recent logs, analytics events, or UI interaction history, applications often need to keep only the latest `N` items without letting memory usage grow forever.

Implement a reusable `CircularBuffer` class with that behavior.

This API is inspired by [cirbuf](https://github.com/jamiebuilds/cirbuf), but for this question you only need a minimal fixed-size buffer that supports appending values and reading them back in logical order.

## Example(s)

```ts
const buffer = new CircularBuffer(3);

buffer.maxSize; // 3
buffer.size; // 0
buffer.toArray(); // []

buffer.push('a');
buffer.push('b');

buffer.size; // 2
buffer.toArray(); // ['a', 'b']
```

```ts
const buffer = new CircularBuffer(3);

buffer.push('a');
buffer.push('b');
buffer.push('c');
buffer.toArray(); // ['a', 'b', 'c']

buffer.push('d');
buffer.toArray(); // ['b', 'c', 'd']
```

```ts
const buffer = new CircularBuffer(2);

buffer.push({ id: 1 });
buffer.push({ id: 2 });
buffer.push({ id: 3 });

buffer.toArray(); // [{ id: 2 }, { id: 3 }]
```

## Reference(s)

- Github [cirbuf](https://github.com/jamiebuilds/cirbuf)

## Solution(s)

```ts
// Approach 1

interface ICircularBuffer<T = unknown> {
  readonly maxSize: number;
  readonly size: number;
  push(value: T): void;
  toArray(): Array<T>;
}

export default class CircularBuffer<T = unknown> implements ICircularBuffer<T> {
  _data: Array<T>;
  _maxSize: number;
  _size: number;
  _nextIndex: number;

  constructor(maxSize: number) {
    this._data = new Array(maxSize);
    this._maxSize = maxSize;
    this._size = 0;
    // _nextIndex always points at the slot that the next push will overwrite.
    this._nextIndex = 0;
  }

  get maxSize(): number {
    return this._maxSize;
  }

  get size(): number {
    return this._size;
  }

  // O(1)
  push(value: T): void {
    // Once the buffer is full, writing at _nextIndex replaces the oldest item
    // and then advances the cursor in a ring.
    this._data[this._nextIndex] = value;
    this._nextIndex = (this._nextIndex + 1) % this._maxSize;
    this._size = Math.min(this._size + 1, this._maxSize);
  }

  toArray(): Array<T> {
    if (this._size < this._maxSize) {
      return this._data.slice(0, this._size);
    }

    // After wrapping, the logical order starts at _nextIndex and continues to
    // the end before looping back to the front.
    return this._data
      .slice(this._nextIndex)
      .concat(this._data.slice(0, this._nextIndex));
  }
}
```

```ts
// Approach 2

interface ICircularBuffer<T = unknown> {
  readonly maxSize: number;
  readonly size: number;
  push(value: T): void;
  toArray(): Array<T>;
}

export default class CircularBuffer<T = unknown> implements ICircularBuffer<T> {
  #maxSize: number;
  #array: T[];

  constructor(maxSize: number) {
    this.#maxSize = maxSize;
    this.#array = [];
  }

  get maxSize(): number {
    return this.#maxSize;
  }

  get size(): number {
    return this.#array.length;
  }

  // O(n)
  push(value: T): void {
    if (this.#array.length === this.#maxSize) {
      this.#array.shift();
    }

    this.#array.push(value);
  }

  toArray(): Array<T> {
    return Array.from(this.#array);
  }
}
```
