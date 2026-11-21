# From Pairs

Implement a function `fromPairs(pairs)` that transforms a list of key-value pairs into an object.

Process pairs in order. If a key appears more than once, the later pair overwrites the earlier value.

## Arguments

pairs (Array): An array of key-value pairs.

## Returns

(Object): Returns the object composed from the key-value pairs.

## Examples

```ts
const pairs = [
  ['a', 1],
  ['b', 2],
  ['c', 3],
];

fromPairs(pairs); // => { a: 1, b: 2, c: 3 }
```

## References

- Object [fromEntries](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries)
- Lodash [_.fromPairs](https://lodash.com/docs/4.18.1#fromPairs)

## Solutions

```ts
export default function fromPairs<T>(
  pairs: Array<[string | number, T]>,
): Record<string | number, T> {
  const returnValue: Record<string | number, T> = {};

  const length: number = pairs.length;
  for (let i = 0; i < length; i++) {
    if (i in pairs) {
      const [key, value] = pairs[i];
      returnValue[String(key)] = value;
    }
  }

  return returnValue;
}
```
