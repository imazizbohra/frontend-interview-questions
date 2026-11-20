# Size

Implement `size(collection)` so it returns the number of items in a supported collection.

Arrays and strings use length, plain objects use the number of own enumerable properties, and Map and Set use size. Return 0 for null, undefined, or unsupported values.

## Argument(s)

collection (Array | Object | string | Map | Set | null | undefined): The collection to inspect.

## Returns

(number): Returns the collection size.

## Example(s)

```ts
// Arrays.
size([1, 2, 3, 4, 5]); // => 5
// Objects.
size({ a: 1, b: 2 }); // => 2
// Strings.
size('peanut'); // => 6
// Sets.
size(new Set([1, 2, 3])); // => 3
// Maps.
size(
  new Map([
    [1, 2],
    [3, 4],
  ]),
); // => 2

size(null); // => 0
```

## Edge cases

Your function should handle null, undefined, and unsupported inputs by returning 0.

## Reference(s)

- Lodash [_.size](https://lodash.com/docs/4.18.1#size)

## Solution(s)

```ts
export default function size(
  collection:
    | Array<any>
    | Object
    | Map<any, any>
    | Set<any>
    | string
    | null
    | undefined,
): number {
  if (typeof collection === "string") return collection.length;

  if (Array.isArray(collection)) return collection.length;

  if (collection instanceof Map || collection instanceof Set) {
    return collection.size;
  }

  if (typeof collection === "object" && collection !== null) {
    return Object.keys(collection).length;
  }

  return 0;
}
```
