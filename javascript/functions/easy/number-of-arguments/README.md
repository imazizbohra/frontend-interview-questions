# Number of Arguments

Implement a function numberOfArguments that returns the number of arguments it was called with. Note that this value is the actual number of arguments, which can be more or less than the defined parameter count (which is determined by Function.prototype.length).

## Returns

(Number): Returns the count of arguments.

## Example(s)

```ts
numberOfArguments(); // 0
numberOfArguments(1); // 1
numberOfArguments(2, 3); // 2
numberOfArguments('a', 'b', 'c'); // 3
```

## Reference(s)

- JavaScript [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- JavaScript [array-like Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections#working_with_array-like_objects)
- Functions [Arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)

## Solution(s)

```ts
export default function mean(array: number[]): number {
  return arguments.length;
}
```
