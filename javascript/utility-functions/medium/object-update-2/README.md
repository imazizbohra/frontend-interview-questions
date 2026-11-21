# Object Update II

**Note**: Similar to [unsquash object](../unsquash-object/)

This is a follow-up to [Object Update](../object-update/).

In the first question, `path` was always a `.` delimited string, every intermediate object already existed, and arrays were out of scope. In this question, implement the same core idea but with more practical path traversal.

Implement `objectUpdate(source, path, valueOrUpdater)`.

Additional behavior for this question:

- `path` may be either a non-empty dot-delimited string or a non-empty array of keys/indices.
- Numeric path segments can access existing array items.
- The final argument may be either a direct value or an updater function that receives the current leaf value.
- Missing or nullish intermediate object branches should be created automatically as plain objects.

Keep the problem scoped:

- `source` is still a plain object at the root.
- Arrays may already exist in the data, but you do not need to create new arrays.
- Array indices used in the path are guaranteed to point to existing items.
- Existing intermediate values are guaranteed to be `objects`, `arrays`, `null`, or `undefined`.
- Any function passed as `valueOrUpdater` is treated as an updater. Storing a function as the direct leaf value is out of scope.
- Inputs are guaranteed valid.
- Path escaping and validation are out of scope.

## Arguments

`objectUpdate(source, path, value)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| source | Object | The plain object to update. |
| path | string | A non-empty dot-delimited path pointing to an existing nested property. |
| value | unknown | The new value to store at the final path segment. |

## Returns

Returns a new object with the updated value written at `path`.

## Examples

```ts
const state = {
  todos: [
    { text: 'Ship feature', done: false },
    { text: 'Write docs', done: false },
  ],
  filters: {
    showCompleted: true,
  },
};

const next = objectUpdate(state, 'todos.0.done', (done) => !done);

next.todos[0].done; // true
next.todos !== state.todos; // true
next.todos[0] !== state.todos[0]; // true
next.todos[1] === state.todos[1]; // true
next.filters === state.filters; // true
```

```ts
const state = {
  draft: {
    title: 'Trip ideas',
  },
};

const next = objectUpdate(state, 'draft.address.city', 'Singapore');

next; // {
//   draft: {
//     title: 'Trip ideas',
//     address: {
//       city: 'Singapore',
//     },
//   },
// }
```

```ts
const state = {
  form: {
    sections: [{ title: 'Profile' }],
  },
};

const next = objectUpdate(state, ['form', 'sections', 0, 'title'], 'Account');

next.form.sections[0].title; // 'Account'
```

## Notes

- Clone only the touched objects and arrays along the updated path.
- Untouched branches should keep the same references as the original input.
- Missing, `null`, or `undefined` intermediate branches are only created as plain objects.
- Storing a function as the direct leaf value is out of scope because every function is treated as an updater.
- You do not need to support invalid-path errors, sparse-array behavior, or escaped path tokens.

## References

- Lodash [_.set](https://lodash.com/docs/4.18.1#set)

## Solutions

```ts
export default function objectUpdate<T extends Record<string, unknown>>(
  source: T,
  path: string | Array<string | number>,
  valueOrUpdater: unknown,
): T {
  let root = { ...source };
  let current = root as Record<string, any>;

  const pathArray = Array.isArray(path) ? path : path.split(".");

  for (let i = 0; i < pathArray.length - 1; i++) {
    const key = pathArray[i];
    const container = current[key];

    if (container) {
      current[key] = Array.isArray(container)
        ? [...container]
        : { ...container };
    } else {
      current[key] = {};
    }

    current = current[key];
  }

  const lastPath = pathArray[pathArray.length - 1];
  const value = current[lastPath];
  current[pathArray[pathArray.length - 1]] =
    typeof valueOrUpdater === "function"
      ? valueOrUpdater(value)
      : valueOrUpdater;

  return root;
}
```
