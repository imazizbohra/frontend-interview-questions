# List Format

Given a list of strings, implement a function `listFormat` that returns the items concatenated into a single `string`. A common use case is summarizing reactions on social media posts.

The function should support a few options as the second parameter:

- `sorted`: Sorts the items alphabetically.
- `length`: Shows only the first length items, using "and X other(s)" for the remaining. Ignore invalid values (negative, 0, etc.).
- `unique`: Removes duplicate items.

## Example(s)

```ts
listFormat([]); // ''

listFormat(['Bob']); // 'Bob'
listFormat(['Bob', 'Alice']); // 'Bob and Alice'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John']);
// 'Bob, Ben, Tim, Jane and John'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 3,
}); // 'Bob, Ben, Tim and 2 others'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 4,
}); // 'Bob, Ben, Tim, Jane and 1 other'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 3,
  sorted: true,
}); // 'Ben, Bob, Jane and 2 others'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John', 'Bob'], {
  length: 3,
  unique: true,
}); // 'Bob, Ben, Tim and 2 others'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 3,
  unique: true,
}); // 'Bob, Ben, Tim and 2 others'

listFormat(['Bob', 'Ben', '', '', 'John']); // 'Bob, Ben and John'
```

## Reference(s)

- Array [sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- Array [slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
- Array [from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- Javascript [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)

## Solution(s)

```ts
export default function listFormat(
  items: Array<string>,
  options?: { sorted?: boolean; length?: number; unique?: boolean },
): string {
  const { sorted = false, length, unique = false } = options || {};
  const SEPARATOR = ", ";
  let array = items.slice();

  array = array.filter(Boolean);

  if (unique) {
    array = Array.from(new Set(array));
  }

  if (sorted) {
    array = array.sort();
  }

  if (array.length === 0) {
    return "";
  }

  if (array.length === 1) {
    return array[0];
  }

  if (!length || length < 0 || length >= array.length) {
    const firstSlice = array.slice(0, array.length - 1);
    const lastElem = array[array.length - 1];

    return firstSlice.join(SEPARATOR).concat(` and ${lastElem}`);
  }

  const firstSlice = array.slice(0, length);
  const lastSlice = array.slice(length);
  const suffix = lastSlice.length === 1 ? "other" : "others";

  return firstSlice
    .join(SEPARATOR)
    .concat(` and ${lastSlice.length} ${suffix}`);
}
```
