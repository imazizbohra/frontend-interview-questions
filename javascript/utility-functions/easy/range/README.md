# Range

Implement a function `range([start=0], end, [step=1])` that creates an array of numbers (positive or negative) progressing from start up to, but not including, end. A step of -1 is used if a negative start is specified without an end or step. If end is not specified, it's set to start, and start is then set to 0.

When step is 0, return Math.max(Math.ceil(end - start), 0) copies of start. This repeats start when start < end and returns [] when start >= end. An explicit zero step does not switch to a descending step.

## Argument(s)

- start (Number): The start of the range.
- end (Number): The end of the range.
- step (Number): The value to increment or decrement by.

## Returns

(Array): Returns the range of numbers.

## Example(s)

```ts
range(4); // => [0, 1, 2, 3]
range(-4); // => [0, -1, -2, -3]
range(1, 5); // => [1, 2, 3, 4]
range(0, 20, 5); // => [0, 5, 10, 15]
range(0, -4, -1); // => [0, -1, -2, -3]
range(1, 4, 0); // => [1, 1, 1]
range(-4, -2, 0); // => [-4, -4]
range(-2, -4, 0); // => []
```

The function should return an empty array if start is equal to end.

```ts
range(0); // => []
```

## Reference(s)

- Lodash [_.range](https://lodash.com/docs/4.18.1#range)

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

  return returnValue;
}
```
