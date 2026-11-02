# Fill

Implement a function `fill(array, value, [start=0], [end=array.length])` that fills an array with values from start up to, but not including, end.

Note: This method mutates array.

## Argument(s)

- array (Array): The array to fill.
- value (*): The value to fill array with.
- [start=0] (number): The start position.
- [end=array.length] (number): The end position.

## Returns

(Array): Returns array.

## Example(s)

```ts
fill([1, 2, 3], 'a'); // ['a', 'a', 'a']
fill([4, 6, 8, 10], '*', 1, 3); // [4, '*', '*', 10]

// out-of-bounds indices
fill([4, 6, 8, 10, 12], '*', 1, 8); // [4, '*', '*', '*', '*']
fill([4, 6, 8, 10, 12], '*', 8, 10); // [4, 6, 8, 10, 12]

// negative in-bounds indices
fill([4, 6, 8, 10, 12], '*', -3, -1); // [4, 6, '*', '*', 12]

// negative out-of-bounds indices
fill([4, 6, 8, 10, 12], '*', -10, 2); // ['*', '*', 8, 10, 12]
fill([4, 6, 8, 10, 12], '*', -10, -8); // [4, 6, 8, 10, 12]
```

Make sure to handle negative indices and out-of-bounds indices as detailed in the examples above. In general, follow these principles:

- Translate negative indices to their corresponding positive indices.
- If the translated end index is smaller than the translated start index, no fill will take place.
- If a translated index is < 0, clamp it to 0.
- If a translated index is > array.length, clamp it to array.length.

## Reference(s)

Lodash [_.fill](https://lodash.com/docs/4.18.1#fill)

## Solution(s)

```ts
export default function fill<T>(
  array: Array<T>,
  value: any,
  start: number = 0,
  end: number = array.length,
): Array<T> {
  if (array.length === 0) return [];

  const length: number = array.length;
  
  // normalizing indexes
  let nStart: number =
    start < 0 ? Math.max(start + length, 0) : Math.min(start, length);
  let nEnd: number =
    end < 0 ? Math.max(end + length, 0) : Math.min(end, length);

  // check index cross each other
  if (nStart > nEnd) return array;

  // check out-of-bound index
  if(nStart < 0 || nStart >= length || nEnd < 0 || nEnd > length) return array;

  while (nStart < nEnd) {
    array[nStart] = value;
    nStart++;
  }

  return array;
}
```
