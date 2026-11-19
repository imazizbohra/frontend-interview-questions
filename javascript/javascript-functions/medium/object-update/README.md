# Object Update

**Note**: Similar to [unsquash object](../unsquash-object/)

Reducers and state containers often need to update a deeply nested field without mutating the existing state. Writing the full object-spread chain by hand works, but it quickly becomes noisy.

An easy way to update an object while leaving the original untouched would be to [deeply clone](../deep-clone/) it, but that uses much more memory because each update creates an entirely new object, which is wasteful when most values in the original object usually stay the same.

Implement a function `objectUpdate(source, path, value)` that updates a nested object path in an immutable fashion and returns a new object instance while sharing the original inner structure where possible.

This question is intentionally scoped:

- `source` is a plain JavaScript object.
- `path` is a non-empty dot-delimited string such as `'draft.user.name'`.
- Every intermediate object already exists.
- Arrays are out of scope.
- Return a new object where the value at `path` has been replaced with `value`.

The important constraint is structural sharing:

- Clone only the objects along the touched path.
- Keep all untouched branches as the exact same references.
- Do not mutate the input object.

## Argument(s)

`objectUpdate(source, path, value)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| source | Object | The plain object to update. |
| path | string | A non-empty dot-delimited path pointing to an existing nested property. |
| value | unknown | The new value to store at the final path segment. |

## Returns

Returns a new object with the updated value written at `path`.

## Example(s)

```ts
const state = {
  draft: {
    user: {
      name: 'Alice',
      role: 'admin',
    },
    meta: {
      saved: false,
    },
  },
  theme: {
    mode: 'dark',
  },
};

const next = objectUpdate(state, 'draft.user.name', 'Bob');

next.draft.user.name; // 'Bob'
next !== state; // true
next.draft !== state.draft; // true
next.draft.user !== state.draft.user; // true
next.draft.meta === state.draft.meta; // true
next.theme === state.theme; // true
```

```ts
const state = {
  status: 'draft',
  draft: {
    title: 'Object Update',
  },
};

const next = objectUpdate(state, 'status', 'published');

console.log(next); // { status: 'published', draft: { title: 'Object Update' } }
console.log(state.status); // 'draft'
```

## Notes

- The returned object should reuse untouched nested references from source.
- Only the objects on the updated path need to be shallow-cloned.
- You do not need to support updater callbacks, arrays, missing-branch creation, or path validation.

## Solution(s)

```ts
export default function objectUpdate<T extends Record<string, unknown>>(
  source: T,
  path: string,
  value: unknown,
): T {
  let root = { ...source };
  let current = root as Record<string, any>;

  const pathArray = path.split(".");

  for (let i = 0; i < pathArray.length - 1; i++) {
    const key = pathArray[i];
    current[key] = { ...current[key] };
    current = current[key];
  }

  current[pathArray[pathArray.length - 1]] = value;

  return root;
}
```
