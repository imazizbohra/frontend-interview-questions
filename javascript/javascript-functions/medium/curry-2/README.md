# Curry II

Note: This is an advanced version of [Curry](../curry/); you should complete that first before attempting this question.

Currying is the technique of converting a function that takes multiple arguments into a sequence of functions that each take a single argument.

Implement the `curry` function, which accepts a function as its only argument. It returns a function that accepts any number of arguments (rather than only one argument at a time, as in [Curry](../curry/)) and can be repeatedly called until at least the minimum number of arguments has been provided, as determined by how many arguments the original function accepts. The original function is then invoked with the provided arguments.

## Example(s)

```ts
function addTwo(a, b) {
  return a + b;
}
const curriedAddTwo = curry(addTwo);
curriedAddTwo(3)(4); // 7
curriedAddTwo(3, 4); // 7
const alreadyAddedThree = curriedAddTwo(3);
alreadyAddedThree(4); // 7
```

```ts
function multiplyThree(a, b, c) {
  return a * b * c;
}
const curriedMultiplyThree = curry(multiplyThree);
curriedMultiplyThree(4)(5)(6); // 120
curriedMultiplyThree(4)(5, 6); // 120
curriedMultiplyThree(4, 5)(6); // 120
curriedMultiplyThree(4, 5, 6); // 120

const containsFour = curriedMultiplyThree(4);
const containsFourMulFive = containsFour(5);
containsFourMulFive(6); // 120
```

## Reference(s)

- JavaScript [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- Function [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

## Solution(s)

```ts
export default function curry(func: Function): Function {
  return function curried(this: any, ...args: any[]): Function {
    if (args.length >= func.length) {
      return func.call(this, ...args);
    }

    return curried.bind(this, ...args);
  };
}
```
