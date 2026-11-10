# Is Empty

Implement `isEmpty(value)` so it returns true when value has no items to inspect.

- Arrays and strings are empty when their length is 0.
- Maps and sets are empty when their size is 0.
- Plain objects are empty when they have no own enumerable properties.
- For this question, non-collection values such as null, booleans, numbers, symbols, and regular expressions should also be treated as empty.

You do not need to handle DOM or jQuery-like collections.

## Argument(s)

value (*): The value to check.

## Returns

(boolean): Returns true if value is empty, else false.

## Example(s)

```ts
isEmpty(null); // => true
isEmpty(true); // => true
isEmpty(1); // => true
isEmpty(new Map()); // => true
isEmpty([1, 2, 3]); // => false
isEmpty({ a: 1 }); // => false
```

## Reference(s)

- Lodash [_.isEmpty](https://lodash.com/docs/4.18.1#isEmpty)

## Solution(s)

```ts
// Approach 1

const getType = (value: unknown) => {
  if (value === null) return "null";

  if (value === undefined) return "undefined";

  return (
    Object.getPrototypeOf(value)?.constructor?.name?.toLowerCase() ?? "object"
  );
};

export default function isEmpty(value: unknown): boolean {
  if (getType(value) === "array" || getType(value) === "string") {
    return (value as unknown[] | string).length === 0;
  }

  if (getType(value) === "map" || getType(value) === "set") {
    return (value as Map<unknown, unknown> | Set<unknown>).size === 0;
  }

  if (getType(value) === "object") {
    return Object.values(value as Record<string, unknown>).length === 0;
  }

  return true;
}
```

```ts
// Approach 2

export default function isEmpty(value: unknown): boolean {
  if (value == null) {
    return true;
  }

  // Arrays/Strings.
  if (Array.isArray(value) || typeof value === 'string') {
    return value.length === 0;
  }

  // Maps/Sets.
  if (value instanceof Map || value instanceof Set) {
    return value.size === 0;
  }

  // Plain objects.
  const prototype = Object.getPrototypeOf(value);
  if (prototype === null || prototype === Object.prototype) {
    return Object.keys(value).length === 0;
  }

  return true;
}
```
