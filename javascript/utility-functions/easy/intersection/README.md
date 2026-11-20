# Intersection

Implement `intersection(...arrays)` so it returns a new array containing the unique values that appear in every provided array. Preserve the order from the first array and do not modify the original arrays.

If no arrays are provided, return an empty array.

## Argument(s)

arrays (...Array): The arrays to compare.

## Returns

(Array): Returns a new array containing the unique values present in all given arrays.

## Example(s)

```ts
const arr1 = [1, 2, 3];
const arr2 = [2, 3, 4];
const arr3 = [3, 4, 5];

intersection(arr1, arr2, arr3); // => [3]

const arr4 = [1, 5, 7, 9, 7];
intersection(arr4); // => [1, 5, 7, 9]

intersection(); // => []
```

## Reference(s)

- Array [from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- Array [every](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
- Array [includes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)
- Array [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- Array [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- Javascript [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/Set)
- Lodash [_.intersection](https://lodash.com/docs/4.18.1#intersection)

## Solution(s)

```ts
export default function intersection<T>(...arrays: Array<Array<T>>): Array<T> {
  if (arrays.length === 0) return [];

  if (arrays.length === 1) return Array.from(new Set(arrays[0]));

  const rootArr: T[][] = Array.from(arrays).map((array) =>
    Array.from(new Set(array)),
  );

  const result = rootArr[0].reduce<T[]>((acc: T[], elem: T) => {
    const isPresent: boolean = rootArr.every((array: T[]) => {
      return array.includes(elem);
    });

    if (isPresent) {
      return [...acc, elem];
    }

    return acc;
  }, []);

  return result as T[];
}
```
