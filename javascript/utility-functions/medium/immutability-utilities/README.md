# Immutability Utilities

**Note**: Similar to [squash-object](../squash-object/) and [unsquash-object](../unsquash-object/)

Libraries like [immutability-helper](https://github.com/kolodny/immutability-helper) make it easier to update nested state without writing long chains of object spreads by hand.

Implement a simplified `update(target, spec)` utility for plain objects.

This version is intentionally interview-scoped and only needs to support:

- nested object traversal through the `spec`
- `{ $set: value }` to replace the current target
- `{ $apply: fn }` to derive the next value from the current target

The important constraint is structural sharing:

- Clone only the objects along the updated path.
- Keep untouched branches as the exact same references.
- Do not mutate the input object.

This question is inspired by `immutability-helper`, but it supports only a small subset of the real library API.

## Arguments

`update(target, spec)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `target` | `Object` | The plain object being updated. |
| `spec` | `Object` | A nested spec object describing what should be updated. |

## Spec rules

A spec node is either:

A traversal object whose keys point to nested specs.
A command object with exactly one of the commands below.

`{ $set: value }`
Replaces the current target entirely with value.

`{ $apply: fn }`
Calls `fn(currentValue)` and uses the return value as the replacement.

## Examples

```ts
const state = {
  profile: {
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

const next = update(state, {
  profile: {
    user: {
      name: {
        $set: 'Bob',
      },
    },
  },
});

next.profile.user.name; // 'Bob'
next !== state; // true
next.profile !== state.profile; // true
next.profile.user !== state.profile.user; // true
next.profile.meta === state.profile.meta; // true
next.theme === state.theme; // true
```

```ts
const state = {
  counters: {
    openPanels: 1,
  },
};

const next = update(state, {
  counters: {
    openPanels: {
      $apply: (count) => count + 1,
    },
  },
});

next;
// {
//   counters: {
//     openPanels: 2,
//   },
// }
```

```ts
const state = {
  status: 'draft',
  profile: {
    title: 'Planning notes',
  },
};

const next = update(state, {
  status: {
    $set: 'published',
  },
});

next;
// {
//   status: 'published',
//   profile: {
//     title: 'Planning notes',
//   },
// }
```

## Notes

- `target` is always a plain object at the root.
- Every traversed key already exists.
- Only the touched path needs to be cloned.
- Arrays, function shorthand specs, missing-branch creation, and invalid-input handling are out of scope.

## References

- Github [immutability-helper](https://github.com/kolodny/immutability-helper)

## Techniques

- Recursion
- Command dispatch
- Structural sharing
- Immutable updates

## Solutions

```ts
interface CommandSpec {
  $set?: unknown;
  $apply?: (currentValue: any) => unknown;
}

interface TraversalSpec {
  [key: string]: Spec;
}

type Spec = CommandSpec | TraversalSpec;

function squashObject(
  obj: Record<string, any>,
  path: string[],
  output: Record<string, any>,
) {
  for (const key in obj) {
    const value = obj[key];

    if (typeof value !== "object" || value === null) {
      output[path.concat(key).join(".")] = value;
    } else {
      squashObject(value, path.concat(key), output);
    }
  }
}

function objectUpdate(obj: Record<string, any>, path: string, value: unknown) {
  let root = { ...obj };
  let current = root;
  const pathArray = path.split(".");

  for (let i = 0; i < pathArray.length - 2; i++) {
    const path = pathArray[i];

    current[path] = { ...current[path] };
    current = current[path];
  }

  const lastPath = pathArray[pathArray.length - 2];
  const setOrApply = pathArray[pathArray.length - 1];
  current[lastPath] = setOrApply === "$set" ? value : value(current[lastPath]);

  return root;
}

export default function update<T extends Record<string, unknown>>(
  target: T,
  spec: Spec,
): T {
  const squashedObj: Record<string, any> = {};

  squashObject(spec, [], squashedObj);

  for (const key in squashedObj) {
    const value = squashedObj[key];

    return objectUpdate(target, key, value) as T;
  }
}
```
