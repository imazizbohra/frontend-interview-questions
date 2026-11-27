# Undo / Redo Manager II

This is a follow-up to [Undo / Redo Manager](../../medium/undo-redo-manager/).

In many products, successive updates should be grouped into a single undo step. For example, typing three characters into a text editor often should not require three separate undo presses. Instead, the app may keep updating the visible value while only committing a history checkpoint once the typing burst is done.

In this question, implement a reusable `UndoRedoManager` class with that behavior:

- `set()` updates the live draft immediately.
- `commit()` records the current draft as one undo/redo history step.

## Examples

```ts
const manager = new UndoRedoManager('');

manager.getCurrent(); // ''
manager.canUndo(); // false
manager.canRedo(); // false

manager.set('h');
manager.set('he');
manager.set('hel');

manager.getCurrent(); // 'hel'
manager.canUndo(); // true because undo() would discard the pending draft
manager.canRedo(); // false

manager.commit();
manager.getCurrent(); // 'hel'

manager.undo();
manager.getCurrent(); // ''
manager.canRedo(); // true

manager.redo();
manager.getCurrent(); // 'hel'

manager.set('hell');
manager.set('hello');
manager.getCurrent(); // 'hello'

manager.undo(); // Discards the uncommitted draft.
manager.getCurrent(); // 'hel'
manager.canRedo(); // false

manager.set('hello');
manager.commit();
manager.getCurrent(); // 'hello'

manager.reset();
manager.getCurrent(); // ''
manager.canUndo(); // false
manager.canRedo(); // false
```

## UndoRedoManager API

Implement the following APIs on the `UndoRedoManager`:

## 1. `new UndoRedoManager(initialValue)`

Creates an instance of the `UndoRedoManager` class with `initialValue` as the first committed history entry. History is isolated within each instance.

- Values should be stored as-is. Deep cloning is not required.

Parameter:

| Parameter | Type | Description |
| --- | --- | --- |
| `initialValue` | `unknown` | The initial value to seed the history with. |

## 2. `manager.getCurrent()`

Returns the current visible value.

- If there is a pending draft started by `set()`, return that draft value.
- Otherwise, return the current committed history value.

## 3. `manager.set(value)`

Updates the live draft to `value`.

- Repeated `set()` calls before `commit()` belong to the same batch, so they should not create multiple committed history steps.
- If some committed values had been undone before the first `set()` in a new batch, all redo history should be discarded immediately when that new batch starts.

Parameter:

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | `unknown` | The new live draft value. |

## 4. `manager.commit()`

Commits the current draft as exactly one new history entry.

- If there is no pending draft, this method should do nothing.

## 5. `manager.undo()`

Moves backward if possible.

- If there is a pending draft, `undo()` should discard that draft and restore the latest committed value instead of traversing committed history.
- Otherwise, if the manager is already at the earliest committed history entry, this method should do nothing.

## 6. `manager.redo()`

Moves the current pointer one committed step forward if possible.

- Discarded uncommitted drafts are not redoable.
- If the manager is already at the latest committed history entry, this method should do nothing.

## 7. `manager.reset()`

Restores the manager to its original `initialValue`, clears committed history back to one entry, and discards any pending draft.

## 8. `manager.canUndo()`

Returns `true` if `undo()` would change the current state.

This includes both:

- Discarding a pending draft.
- Moving to an earlier committed history entry.

## 9. `manager.canRedo()`

Returns `true` if there is a later committed history entry to move to; otherwise, returns `false`.

## Notes

- Calling `commit()` after several `set()` calls should still create only one committed history step.
- A committed batch should still create a history step even if it ends on the same value it started with.
- You do not need to implement timers, nested batches, async batching, or history inspection APIs such as `getHistory()` or `getIndex()`.

## References

- javascriptbit.com [undo-redo](https://javascriptbit.com/frontend-system-design-undo-redo/)

## Solutions

```ts
interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  commit(): void;
  undo(): void;
  redo(): void;
  reset(): void;
  canUndo(): boolean;
  canRedo(): boolean;
}

export default class UndoRedoManager<
  T = unknown,
> implements IUndoRedoManager<T> {
  _initialValue: T;
  _history: Array<T>;
  _currentIndex: number;
  _draftValue: T | undefined;
  _hasDraft: boolean;

  constructor(initialValue: T) {
    this._initialValue = initialValue;
    this._history = [initialValue];
    this._currentIndex = 0;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  getCurrent(): T {
    return this._hasDraft
      ? (this._draftValue as T)
      : this._history[this._currentIndex];
  }

  set(value: T): void {
    if (!this._hasDraft) {
      // Starting a new draft branch immediately invalidates redo history.
      this._history = this._history.slice(0, this._currentIndex + 1);
    }

    this._draftValue = value;
    this._hasDraft = true;
  }

  commit(): void {
    if (!this._hasDraft) {
      return;
    }

    this._history.push(this._draftValue as T);
    this._currentIndex = this._history.length - 1;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  undo(): void {
    if (this._hasDraft) {
      this._draftValue = undefined;
      this._hasDraft = false;
      return;
    }

    if (!this.canUndo()) {
      return;
    }

    this._currentIndex -= 1;
  }

  redo(): void {
    if (!this.canRedo()) {
      return;
    }

    this._currentIndex += 1;
  }

  reset(): void {
    this._history = [this._initialValue];
    this._currentIndex = 0;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  canUndo(): boolean {
    return this._hasDraft || this._currentIndex > 0;
  }

  canRedo(): boolean {
    return this._currentIndex < this._history.length - 1;
  }
}
```
