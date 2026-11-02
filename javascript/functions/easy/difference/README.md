# Difference

Implement difference(array, values) so it returns a new array containing the values from array that do not appear in values. Preserve the order from array and do not modify the original arrays.

For this question, treat NaN values as equal and skip empty slots in sparse arrays.

## Argument(s)

- array (Array): The array to inspect.
- values (Array): The values to exclude.

## Returns

(Array): Returns the new array of filtered values.

## Example(s)

```ts
difference([1, 2, 3], [2, 3]); // => [1]
difference([1, 2, 3, 4], [2, 3, 1]); // => [4]
difference([1, NaN, 2], [NaN]); // => [1, 2]
difference([1, 2, 3], [2, 3, 1, 4]); // => []
difference([1, , 3], [1]); // => [3]
difference([1, 2, 3], []); // => [1, 2, 3]
```

## Reference(s)

- Operators [in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in)
- Array [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
- Array [includes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)
- Lodash [_.difference](https://lodash.com/docs/4.18.1#difference)

## Solution(s)

```ts
// Approach 1

export default function difference<T>(
  array: Array<T>,
  values: Array<T>,
): Array<T> {
  const returnValue: Array<T> = [];
  const length: number = array.length;
  const valuesSet: Set<T> = new Set(values);

  for (let i = 0; i < length; i++) {
    const elem: T = array[i];

    // if slot is non-empty and elem does't exist in set
    if (i in array && !valuesSet.has(elem)) {
      returnValue.push(elem);
    }
  }

  return returnValue;
}

```

```ts
// Approach 2

export default function difference<T>(
  array: Array<T>,
  values: Array<T>,
): Array<T> {
  return array.filter((value) => !values.includes(value));
}
```
