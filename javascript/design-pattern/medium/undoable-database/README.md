# Undoable Database

**Note**: Similar to [UndoRedoManager](../undo-redo-manager/)

`Undo/redo` is often applied to more than a single value. In admin tools, editors, and dashboards, one history step might represent creating, editing, or deleting a full record.

In this question, implement a reusable `Database` class that stores users and supports `undo/redo` for CRUD operations.

Each user has an `id` and may contain arbitrary additional properties.

## Examples

```ts
const database = new Database();

database.getUsers(); // []

database.addUser({ id: 1, name: 'Alice', role: 'admin' });
database.addUser({ id: 2, name: 'Bob' });
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob' },
// ]

database.updateUser(2, { role: 'editor', active: true });
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'editor', active: true },
// ]

database.deleteUser(1);
database.getUsers();
// [{ id: 2, name: 'Bob', role: 'editor', active: true }]

database.undo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'editor', active: true },
// ]

database.undo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob' },
// ]

database.redo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'editor', active: true },
// ]
```

## Notes

- Every successful `addUser()`, `updateUser()`, and `deleteUser()` call should create exactly one undoable history step.
- A successful CRUD call after `undo()` should clear redo history.
- Calling `updateUser()` with values identical to the current top-level values should still create a history step.
- You may assume `addUser()` is only called with a unique id, and `updateUser()` / `deleteUser()` are only called with existing IDs.
- You do not need to deep clone nested objects or arrays inside user records.

## References

- javascriptbit.com [undo-redo](https://javascriptbit.com/frontend-system-design-undo-redo/)

## Solutions

```ts
export type UserId = string | number;
export type UserRecord = {
  id: UserId;
  [key: string]: unknown;
};
export type UserUpdates = Partial<Omit<UserRecord, "id">>;

interface IDatabase {
  getUsers(): Array<UserRecord>;
  addUser(user: UserRecord): void;
  updateUser(id: UserId, updates: UserUpdates): void;
  deleteUser(id: UserId): void;
  undo(): void;
  redo(): void;
}

function cloneUser(user: UserRecord): UserRecord {
  return { ...user };
}

function cloneUsers(users: Array<UserRecord>): Array<UserRecord> {
  return users.map(cloneUser);
}

export default class Database implements IDatabase {
  _history: Array<Array<UserRecord>>;
  _currentIndex: number;

  constructor() {
    this._history = [[]];
    this._currentIndex = 0;
  }

  getUsers(): Array<UserRecord> {
    return cloneUsers(this._getCurrentSnapshot());
  }

  addUser(user: UserRecord): void {
    const nextUsers = cloneUsers(this._getCurrentSnapshot());
    nextUsers.push(cloneUser(user));
    this._commitSnapshot(nextUsers);
  }

  updateUser(id: UserId, updates: UserUpdates): void {
    const currentUsers = this._getCurrentSnapshot();
    const userIndex = currentUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const nextUsers = cloneUsers(currentUsers);
    const currentUser = nextUsers[userIndex];
    nextUsers[userIndex] = {
      ...currentUser,
      ...updates,
      id: currentUser.id,
    };

    this._commitSnapshot(nextUsers);
  }

  deleteUser(id: UserId): void {
    const currentUsers = this._getCurrentSnapshot();
    const userIndex = currentUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const nextUsers = cloneUsers(currentUsers);
    nextUsers.splice(userIndex, 1);
    this._commitSnapshot(nextUsers);
  }

  undo(): void {
    if (this._currentIndex === 0) {
      return;
    }

    this._currentIndex -= 1;
  }

  redo(): void {
    if (this._currentIndex === this._history.length - 1) {
      return;
    }

    this._currentIndex += 1;
  }

  _getCurrentSnapshot(): Array<UserRecord> {
    return this._history[this._currentIndex];
  }

  _commitSnapshot(users: Array<UserRecord>): void {
    // Any new mutation after undo should drop the redo branch first.
    this._history = this._history.slice(0, this._currentIndex + 1);
    this._history.push(users);
    this._currentIndex = this._history.length - 1;
  }
}
```
