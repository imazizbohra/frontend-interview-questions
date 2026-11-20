# Deep Merge

Implement a function `deepMerge(objA, objB)` that takes two objects and returns a new object after deep merging them:

- The resulting object should contain a union of the keys and values of both objects.
- If the same key is present in both objects, the merged value will be from `objB`, unless:
  - Both values are arrays: the elements from `objB` will be appended after `objA's`.
  - Both values are objects: merge the objects using the same rules for `deepMerge`.
- Every array or plain object produced by an actual merge should be a new instance. Values copied from only one side, and items inside concatenated arrays, may retain their original references.

The input objects should not be modified.

## Behavior guide

| objA value | objB value | Merged value |
| --- | --- | --- |
| Plain object | Plain object | Recursively merge both objects |
| Array | Array | A new array with objA items followed by objB items |
| Any other overlap | Any value | The value from objB |
| Key only in objA | Missing | The value from objA |
| Missing | Key only in objB | The value from objB |

Only plain objects are recursively merged. Values such as `null`, `Date`, `RegExp`, `Map`, and `Set` should be treated as leaf values.

## Example(s)

```ts
deepMerge({ a: 1 }, { b: 2 }); // { a: 1, b: 2 }
deepMerge({ a: 1 }, { a: 2 }); // { a: 2 }
deepMerge({ a: 1, b: [2] }, { b: [3, 4] }); // { a: 1, b: [2, 3, 4] }
```

## Notes

- Non-plain values like `null`, `Date`, `RegExp`, and `Symbol` values should be treated as non-objects.
- Non-primitive values in the merged object may reference the original objects, as long as the original objects are not modified.
- If a full copy of every nested object is desired, deep clone both objects before merging or deep clone the resulting object.
- Recursing only on overlapping object keys keeps keys that exist only in `objA` intact and copies keys that exist only in `objB` directly.

## Reference(s)

- Concept [recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion)

## Solution(s)

```ts
const isPlainObject = (value: unknown): value is Record<string, unknown> => {
  if (value == null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);
  return proto == null || proto == Object.prototype;
};

export default function deepMerge(target: unknown, source: unknown): unknown {
  if (Array.isArray(target) && Array.isArray(source)) {
    return [...target, ...source];
  }

  if (isPlainObject(target) && isPlainObject(source)) {
    const clonedTarget = { ...target };
    for (const key in source) {
      clonedTarget[key] = deepMerge(clonedTarget[key], source[key]);
    }
    return clonedTarget;
  }

  // Return the source value as it will "win" in case of an overlap.
  return source;
}
```
