# Deep Set II

**Note**: Similar to [unsquash object](../unsquash-object/)

This is a follow-up to [Deep Set](../deep-set/).

In the first question, `deepSet(obj, path, value)` always overwrote the value at the target path. In this question, keep the same API but merge object and array values at the leaf when possible.

## Merge rules

When `deepSet(obj, path, value)` reaches the final segment:

- If the existing `value` and `value` are both plain objects, recursively merge them by key.
- If the existing `value` and `value` are both arrays, recursively merge them by index.
- Otherwise, overwrite the existing `value` with `value`.

These merge rules apply recursively within the leaf value. When nested values have compatible container types, merge them using the - same rules. Otherwise, overwrite the nested existing value.

Intermediate traversal rules stay the same as the previous question:

- Missing containers should be created automatically.
- Numeric next segments should create arrays.
- Primitive intermediate branches may be replaced if traversal must continue.

## Arguments

`deepSet(obj, path, value)`

- `obj` (`object | Array<unknown>`): The object to mutate.
- `path (string | Array<string | number>)`: The non-empty path where the value should be written. Array indices may be non-negative integers or canonical numeric strings such as `'0'` and `'12'`.
- `value (unknown)`: The value to write or merge at the target path.

## Examples

Merge plain objects at the leaf.

```ts
const object = {
  settings: {
    theme: {
      mode: 'light',
      contrast: 'normal',
    },
  },
};

deepSet(object, 'settings.theme', { mode: 'dark' });

// {
//   settings: {
//     theme: {
//       mode: 'dark',
//       contrast: 'normal',
//     },
//   },
// }
```

Merge arrays by index.

```ts
const object = {
  items: [{ done: false }, { done: false }, { done: true }],
};

deepSet(object, 'items', [{ done: true }]);

// {
//   items: [{ done: true }, { done: false }, { done: true }],
// }
```

Nested values inside merged arrays should also merge recursively.

```ts
const object = {
  items: [{ info: { count: 1, label: 'a' } }],
};

deepSet(object, 'items', [{ info: { count: 2 } }]);

// {
//   items: [{ info: { count: 2, label: 'a' } }],
// }
```

If the values cannot be merged, overwrite as usual.

```ts
const object = {
  settings: {
    theme: { mode: 'light' },
  },
};

deepSet(object, 'settings.theme', 'dark');

// {
//   settings: {
//     theme: 'dark',
//   },
// }
```

## Notes

- This question only changes the behavior of the final write.
- Paths are guaranteed to be non-empty.
- Array merging is by index, not concatenation.
- This is still an interview-sized subset:
  - Bracket syntax is out of scope.
  - Numeric strings with leading zeroes, such as `'01'`, are out of scope.
  - Immutable writes are out of scope.
  - Special object types and cycle handling are out of scope.

## References

- Github [dset](https://github.com/lukeed/dset)
- Lodash [_.set](https://lodash.com/docs/4.18.1#set)

## Solutions

```ts
type PathSegment = string | number;
type Path = string | Array<PathSegment>;
type Container = Record<PathSegment, unknown> | Array<unknown>;

const isPrimitive = (value: unknown) => Object(value) !== value;

const isPlainObject = (value: unknown): value is Record<string, unknown> => {
  if (value == null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);
  return proto == null || proto == Object.prototype;
};

function merge(target: unknown, source: unknown): unknown {
  if (Array.isArray(target) && Array.isArray(source)) {
    for (let i = 0; i < source.length; i++) {
      target[i] = merge(target[i], source[i]);
    }
    return target;
  }

  if (isPlainObject(target) && isPlainObject(source)) {
    for (const key in source) {
      target[key] = merge(target[key], source[key]);
    }
    return target;
  }

  return source;
}

export default function deepSet(
  obj: Container,
  path: Path,
  value: unknown,
): void {
  const pathArr = Array.isArray(path) ? path : path.split(".");
  let current = obj;

  for (let i = 0; i < pathArr.length - 1; i++) {
    const firstPath = pathArr[i] as number;
    const nextPath = pathArr[i + 1];

    if (typeof current[firstPath] === "object" && current[firstPath] !== null) {
      // reuse existing container
      current = current[firstPath] as Container;
    } else {
      // create new container
      current[firstPath] = Number.isNaN(Number(nextPath)) ? {} : [];
      current = current[firstPath] as Container;
    }
  }

  const lastPath = pathArr[pathArr.length - 1] as number;
  const leafValue = current[lastPath];

  if (Array.isArray(leafValue) && Array.isArray(value)) {
    current[lastPath] = merge(leafValue, value);
    return;
  } else if (isPlainObject(leafValue) && isPlainObject(value)) {
    current[lastPath] = merge(leafValue, value);
    return;
  }

  current[lastPath] = value;
}
```
