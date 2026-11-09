# Curry

Currying is the technique of converting a function that takes multiple arguments into a sequence of functions that each take a single argument.

Implement `curry(fn)` to return a curried version of `fn`. Collect one argument at a time until at least `fn.length` arguments have been provided, then call `fn` with the collected arguments.

Each call contributes at most its first argument; additional arguments passed in the same call are ignored. Calls with no arguments should also be ignored and return another function, while an explicit `undefined` counts as an argument. Preserve the `this` value from the first call in the chain when `fn` is finally invoked.

## Arguments

fn (Function): The function to curry.

## Returns

(Function): Returns the curried function.

## Example(s)

```ts
function add(a, b) {
  return a + b;
}

const curriedAdd = curry(add);
curriedAdd(3)(4); // 7

const alreadyAddedThree = curriedAdd(3);
alreadyAddedThree(4); // 7
```

```ts
function multiplyThreeNumbers(a, b, c) {
  return a * b * c;
}

const curriedMultiplyThreeNumbers = curry(multiplyThreeNumbers);
curriedMultiplyThreeNumbers(4)(5)(6); // 120
curriedMultiplyThreeNumbers()(4)()(5)(6); // 120

const containsFour = curriedMultiplyThreeNumbers(4);
const containsFourMulFive = containsFour(5);
containsFourMulFive(6); // 120
```

## Reference(s)

- JavaScript [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- Function [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## Solution(s)

```ts
// Approach 1

export default function curry(func: Function): Function {
  let args: any[] = [];

  return function inner(this: any, arg: any) {
    if (arguments.length >= 1) {
      args.push(arg);
    }

    if (args.length === func.length) {
      const result = func.call(this, ...args);
      args = [];
      return result;
    }

    return inner.bind(this);
  };
}
```

```ts
// Approach 2

export default function curry(func: Function): Function {
  function collect(receiver: any, args: Array<any>): any {
    if (args.length >= func.length) {
      return func.call(receiver, ...args);
    }

    return function next(arg: any): any {
      const nextArgs = arguments.length === 0 ? args : [...args, arg];
      return collect(receiver, nextArgs);
    };
  }

  return function (this: any, arg: any) {
    const args = arguments.length === 0 ? [] : [arg];
    return collect(this, args);
  };
}
```
