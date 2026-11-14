# Conforms To

Implement a function `conformsTo(object, source)` that checks whether `object` conforms to `source` by invoking the predicate properties of `source` with the corresponding property values of `object`.

## Argument(s)

- `object (Object)`: The object to inspect.
- `source (Object)`: The object of property predicates to conform to.

## Returns

`(boolean)`: Returns true if object conforms; otherwise, returns false.

## Example(s)

```ts
conformsTo({ a: 1, b: 2 }, { b: (n) => n > 1 });
// => true

conformsTo({ a: 1, b: 2 }, { b: (n) => n > 2 });
// => false
```

The function should return false when object is empty.

```ts
conformsTo({}, { b: (n) => n > 1 }); // => false
```

## Reference(s)

- Lodash [_.conformsTo](https://lodash.com/docs/4.18.1#conformsTo)

## Solution(s)

```ts
export default function conformsTo<T>(
  object: Record<string, T>,
  source: Record<string, (value: T) => boolean>,
): boolean {
  for (const key in source) {
    if (Object.hasOwn(source, key)) {
      const value = object[key];
      const predicate = source[key];

      if (!(key in object) || !predicate(value)) {
        return false;
      }
    }
  }

  return true;
}
```
