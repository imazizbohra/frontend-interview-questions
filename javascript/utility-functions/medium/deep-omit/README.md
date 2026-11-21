# Deep Omit

Implement a function `deepOmit(obj, keys)` that removes specified keys and their corresponding values from an object, including nested objects or arrays. It works recursively to traverse the entire object structure, ensuring that all occurrences of the specified keys are removed at all levels. The function takes an object (`obj`) and an array of string keys (`keys`).

Behavior guide

- Return a new object or array for every traversed object or array.
- Remove matching keys from plain objects at every depth.
- Recurse into arrays, but do not remove array elements just because their index appears in `keys`.
- Leave non-plain objects such as `Date`, `RegExp`, `Map`, and `Set` as-is.
- Ignore keys that do not exist.

## Examples

```ts
deepOmit({ a: 1, b: 2, c: 3 }, ['b']); // { a: 1, c: 3 }
```

A more complicated example with nested objects:

```ts
const obj = {
  a: 1,
  b: 2,
  c: {
    d: 3,
    e: 4,
  },
  f: [5, 6],
};
deepOmit(obj, ['b', 'c', 'e']); // { a: 1, f: [5, 6] }
```

## References

- Lodash [_.omit](https://lodash.com/docs/4.18.1#omit)

## Solutions

```ts
export default function deepOmit(val: unknown, keys: Array<string>): unknown {
  if (Array.isArray(val)) {
    return val.slice().map((elem) => deepOmit(elem, keys));
  }

  if (typeof val === "object" && val !== null) {
    const obj = { ...val } as Record<string, unknown>;

    for (const key in obj) {
      if (keys.includes(key)) {
        delete obj[key];
        continue;
      }

      obj[key] = deepOmit(obj[key], keys);
    }

    return obj;
  }

  return val;
}
```
