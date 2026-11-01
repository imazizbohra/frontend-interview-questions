# Clamp

Implement clamp(value, lower, upper) to restrict value to the inclusive range between lower and upper.

## Argument(s)

- value (number): The number to clamp.
- lower (number): The lower bound.
- upper (number): The upper bound.

## Returns

(number): Returns value if it is within the range, lower if it is smaller, or upper if it is larger.

## Example(s)

```ts
// Within the bounds, return as-is.
clamp(3, 0, 5); // => 3

// Smaller than the lower bound.
clamp(-10, -3, 5); // => -3

// Bigger than the upper bound.
clamp(10, -5, 5); // => 5
```

## Reference(s)

Lodash [_.clamp](https://lodash.com/docs/4.18.1#clamp)

## Solution(s)

```ts
// Approach 1

export default function clamp(
  value: number,
  lower: number,
  upper: number,
): number {
  // smaller than lower bound
  if (value < lower) {
    return lower;
  }

  // bigger than upper bound
  if (value > upper) {
    return upper;
  }

  // within the bounds
  return value;
}
```

```ts
// Approach 2

export default function clamp(
  value: number,
  lower: number,
  upper: number,
): number {
  return Math.min(upper, Math.max(lower, value));
}
```
