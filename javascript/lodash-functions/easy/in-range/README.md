# Get

Implement a function `inRange(value, [start=0], end)` to check if a number value is greater than or equal to start and less than end. If only two arguments are specified, the second argument becomes end and start is set to 0. If start is greater than end, the parameters are swapped to support reversed ranges.

## Argument(s)

- value (number): The number to check.
- [start=0] (number): The start of the range.
- end (number): The end of the range (exclusive).

## Returns

(boolean): Returns true if value is in the range, or false otherwise.

## Example(s)

```ts
inRange(3, 2, 4); // => true
inRange(4, 8); // => true
inRange(4, 2); // => false
inRange(2, 2); // => false
inRange(1.2, 2); // => true
inRange(5.2, 4); // => false
inRange(-3, -2, -6); // => true
```

## Reference(s)

- Lodash [_.inRange](https://lodash.com/docs/4.18.1#inRange)

## Solution(s)

```ts
export default function inRange(
  value: number,
  start: number,
  end?: number,
): boolean {
  // if only 2 arguments
  if (!end) {
    end = start;
    start = 0;
  }

  if (start > end) {
    [end, start] = [start, end];
  }

  if (value >= start && value < end) return true;

  return false;
}
```
