# Chunk

Implement a function chunk(array, [size=1]) that splits the input array into groups of length size and returns them in a new array. If the array can't be split evenly, the final chunk will be the remaining elements. The function should not modify the original input array.

## Argument(s)

- array (Array): The array to process.
- [size=1] (number): The length of each chunk.

## Returns

(Array): Returns the new array of chunks.

## Example(s)

```ts
chunk(['a', 'b', 'c', 'd']); // => [['a'], ['b'], ['c'], ['d']]
chunk([1, 2, 3, 4], 2); // => [[1, 2], [3, 4]]
chunk([1, 2, 3, 4], 3); // => [[1, 2, 3], [4]]
```

## Note

The function should return an empty array if the array argument is empty.

## Reference(s)

- Array [slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- Lodash [_.chunk](https://lodash.com/docs/4.18.1#chunk)

## Solution(s)

```ts
// Approach 1

export default function chunk<T>(array: Array<T>, size = 1): Array<Array<T>> {
  if (array.length === 0) {
    return [];
  }

  const length: number = array.length;
  const returnValue: Array<Array<T>> = [];

  let i = 0;
  while (i < length) {

    const chunk: Array<T> = [];
    while (i < length && chunk.length < size) {
      chunk.push(array[i]);
      i++;
    }

    returnValue.push(chunk);
  }

  return returnValue;
}

```

```ts
// Approach 2

export default function chunk<T>(array: Array<T>, size = 1): Array<Array<T>> {
  if (array.length === 0) {
    return [];
  }

  const length: number = array.length;
  const returnValue: Array<Array<T>> = [];

  // hop on to each chunk 1 by 1
  for (let i = 0; i < length; i = i + size) {
    const chunk = array.slice(i, i + size);
    returnValue.push(chunk);
  }

  return returnValue;
}
```

```ts
// Approach 3

export default function chunk<T>(array: Array<T>, size = 1): Array<Array<T>> {
  if (array.length === 0) {
    return [];
  }

  const length: number = array.length;
  const chunkCount: number = Math.floor(length / size);
  const returnValue: Array<Array<T>> = [];

  // collect each chunk
  for (let i = 0; i < chunkCount; i++) {
    const startIdx = i * size;
    const endIdx = startIdx + size;
    const chunk = array.slice(startIdx, endIdx);
    returnValue.push(chunk);
  }

  // collect remaining elements into a new chunk
  if (chunkCount * size < length) {
    const startIdx = chunkCount * size;
    const endIdx = startIdx + size;
    const chunk = array.slice(startIdx, endIdx);
    returnValue.push(chunk);
  }

  return returnValue;
}
```
