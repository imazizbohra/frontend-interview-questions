# Function Length

Implement a function functionLength to return the number of parameters a function expects. Note that this is a static value defined by the function, not the number of arguments the function is called with (which is determined by arguments.length).

## Argument(s)

- value (function): The function to operate on.

## Returns

(number): Returns the number of parameters a function expects.

## Example(s)

```ts
function foo() {}
function bar(a) {}
function baz(a, b) {}

functionLength(foo); // 0
functionLength(bar); // 1
functionLength(baz); // 2

```

## Reference(s)

Function [length](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/length)

## Solution(s)

```ts
export default function functionLength(fn: Function): number {
  return fn.length;
}
```
