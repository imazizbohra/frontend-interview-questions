# Compact

Implement compact(array) so it creates a new array with all falsy values removed. This includes false, null, 0, '', undefined, NaN, and empty slots in sparse arrays. Do not modify the original array.

## Argument(s)

array (Array): The array to compact.

## Returns

(Array): Returns the new array of filtered values.

## Example(s)

```ts
compact([0, 1, false, 2, '', 3, null]); // => [1, 2, 3]
compact(['hello', 123, [], {}]); // => ['hello', 123, [], {}]
compact([1, , null, 2, , 3]); // => [1, 2, 3]
```

## Reference(s)

- Javascript [falsy values](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)
- Javascript [truthy values](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)
- Operators [in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in)
- Lodash [_.compact](https://lodash.com/docs/4.18.1#compact)

## Solution(s)

```ts
// Approach 1

export default function compact<T>(
  array: Array<T | null | undefined | false | 0 | "">,
): Array<T> {
  const length: number = array.length;
  const returnValue: Array<T> = [];

  for (let i = 0; i < length; i++) {
    const elem = array[i];

    // if non-empty slot and elem is truthy
    if (i in array && Boolean(elem) === true) {
      returnValue.push(elem as T);
    }
  }

  return returnValue;
}
```

```ts
// Approach 2

export default function compact<T>(
  array: Array<T | null | undefined | false | 0 | ''>,
): Array<T> {
  return array.filter((value): value is T => Boolean(value));
}
```
