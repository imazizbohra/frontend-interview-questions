# Range Right

Implement a function `rangeRight([start=0], end, [step=1])` that creates an array of numbers progressing from start up to (but not including) end with a specified step, similar to [range](../range/), but in descending order.

## Argument(s)

- start (Number): The start of the range.
- end (Number): The end of the range.
- step (Number): The value to increment or decrement by.

## Returns

(Array): Returns the range of numbers.

## Example(s)

```ts
rangeRight(4); // => [3, 2, 1, 0]
rangeRight(-4); // => [-3, -2, -1, 0]
rangeRight(1, 5); // => [4, 3, 2, 1]
rangeRight(0, 20, 5); // => [15, 10, 5, 0]
rangeRight(0, -4, -1); // => [-3, -2, -1, 0]
rangeRight(1, 4, 0); // => [1, 1, 1]
```

The function should return an empty array if start is equal to end.

```ts
range(0); // => []
```

## Reference(s)

- Lodash [_.rangeRight](https://lodash.com/docs/4.18.1#rangeRight)

## Solution(s)

```ts
export default function range(
  start: number = 0,
  end?: number,
  step?: number = 1,
): Array<number> {
  if (end === undefined) {
    end = start;
    start = 0;
  }

  if (end < start && step === 1) {
    step = -1;
  }

  const returnValue: number[] = [];
  const count = (end - start) / (step || 1);
  for (let i = 0; i < count; i++) {
    returnValue.push(start + i * step);
  }

  return returnValue.reverse();
}
```
