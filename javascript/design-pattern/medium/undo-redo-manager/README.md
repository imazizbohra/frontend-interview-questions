# Undo / Redo Manager

Undo/redo functionality is common in text editors, drawing tools, settings panels, and multi-step workflows. Instead of tying this logic to a UI, implement a reusable `UndoRedoManager` class that stores a history of values and can move backward and forward through that history.

## Example(s)

```ts
const manager = new UndoRedoManager(0);

manager.getCurrent(); // 0
manager.canUndo(); // false
manager.canRedo(); // false

manager.set(1);
manager.set(2);
manager.getCurrent(); // 2

manager.undo();
manager.getCurrent(); // 1
manager.canRedo(); // true

manager.redo();
manager.getCurrent(); // 2

manager.undo();
manager.set(10);
manager.getCurrent(); // 10
manager.canRedo(); // false

manager.reset();
manager.getCurrent(); // 0
manager.canUndo(); // false
manager.canRedo(); // false
```

## Notes

- Calling `set()` with the same value as the current value should still create a new history step.
- You do not need to implement history inspection APIs such as `getHistory()` or `getIndex()`.

## Reference(s)

- javascriptbit.com [undo-redo](https://javascriptbit.com/frontend-system-design-undo-redo/)

## Solution(s)

```ts
// Approach 1: Array + Current Pointer

interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  undo(): void;
  redo(): void;
  reset(): void;
  canUndo(): boolean;
  canRedo(): boolean;
}

export default class UndoRedoManager<
  T = unknown,
> implements IUndoRedoManager<T> {
  #initialValue: T;
  #history: T[];
  #currentIndex: number;

  constructor(initialValue: T) {
    this.#initialValue = initialValue;
    this.#history = [initialValue];
    this.#currentIndex = 0;
  }

  getCurrent(): T {
    return this.#history[this.#currentIndex];
  }

  set(value: T): void {
    const oldhistory = this.#history.slice(0, this.#currentIndex + 1);
    this.#history = [...oldhistory, value];
    this.#currentIndex++;
  }

  undo(): void {
    this.#currentIndex = Math.max(0, this.#currentIndex - 1);
  }

  redo(): void {
    this.#currentIndex = Math.min(
      this.#history.length - 1,
      this.#currentIndex + 1,
    );
  }

  reset(): void {
    this.#history = [this.#initialValue];
    this.#currentIndex = 0;
  }

  canUndo(): boolean {
    return this.#currentIndex > 0;
  }

  canRedo(): boolean {
    return this.#history.length - 1 > this.#currentIndex;
  }
}
```

```ts
// Approach 2: Two Stacks

interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  undo(): void;
  redo(): void;
  reset(): void;
  canUndo(): boolean;
  canRedo(): boolean;
}

export default class UndoRedoManagerStacks<
  T = unknown,
> implements IUndoRedoManager<T> {
  _initialValue: T;
  _past: Array<T>;
  _current: T;
  _future: Array<T>;

  constructor(initialValue: T) {
    this._initialValue = initialValue;
    this._past = [];
    this._current = initialValue;
    this._future = [];
  }

  getCurrent(): T {
    return this._current;
  }

  set(value: T): void {
    // Setting a new value promotes the current value into past history and
    // invalidates any redo path.
    this._past.push(this._current);
    this._current = value;
    this._future = [];
  }

  undo(): void {
    if (!this.canUndo()) {
      return;
    }

    // The current value becomes redo-able, and the latest past value becomes
    // the new current value.
    this._future.push(this._current);
    this._current = this._past.pop() as T;
  }

  redo(): void {
    if (!this.canRedo()) {
      return;
    }

    // Redo is the reverse transfer: current goes back into past, and the most
    // recent future value becomes current again.
    this._past.push(this._current);
    this._current = this._future.pop() as T;
  }

  reset(): void {
    this._past = [];
    this._current = this._initialValue;
    this._future = [];
  }

  canUndo(): boolean {
    return this._past.length > 0;
  }

  canRedo(): boolean {
    return this._future.length > 0;
  }
}
```

```ts
// Approach 3: LinkedList

interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  undo(): void;
  redo(): void;
  reset(): void;
  canUndo(): boolean;
  canRedo(): boolean;
}

type HistoryNode<T> = {
  value: T;
  prev: HistoryNode<T> | null;
  next: HistoryNode<T> | null;
};

function createNode<T>(value: T): HistoryNode<T> {
  return {
    value,
    prev: null,
    next: null,
  };
}

export default class UndoRedoManagerLinkedList<
  T = unknown,
> implements IUndoRedoManager<T> {
  _initialValue: T;
  _head: HistoryNode<T>;
  _current: HistoryNode<T>;

  constructor(initialValue: T) {
    const node = createNode(initialValue);
    this._initialValue = initialValue;
    this._head = node;
    this._current = node;
  }

  getCurrent(): T {
    return this._current.value;
  }

  set(value: T): void {
    const node = createNode(value);
    node.prev = this._current;
    // Overwriting `next` drops any redo branch that used to hang off the
    // current node, then the new node becomes the active tail.
    this._current.next = node;
    this._current = node;
  }

  undo(): void {
    if (!this.canUndo()) {
      return;
    }

    this._current = this._current.prev as HistoryNode<T>;
  }

  redo(): void {
    if (!this.canRedo()) {
      return;
    }

    this._current = this._current.next as HistoryNode<T>;
  }

  reset(): void {
    const node = createNode(this._initialValue);
    // Rebuild the list from scratch so both undo and redo history are cleared.
    this._head = node;
    this._current = node;
  }

  canUndo(): boolean {
    return this._current.prev !== null;
  }

  canRedo(): boolean {
    return this._current.next !== null;
  }
}
```
