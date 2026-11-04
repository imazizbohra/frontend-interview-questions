# Compose

Implement a function `compose` that takes multiple functions as arguments and returns a new function that applies those functions in reverse order. The output of one function becomes the input of the next function, creating a chain of function compositions.

If no functions are passed to `compose`, return a new function that simply returns the input it receives. In other words, `compose` without any functions returns an identity function.

You may assume that all the functions accept a single parameter.

## Example(s)

```ts
const add1 = (num) => num + 1;
const double = (num) => num * 2;
const subtract10 = (num) => num - 10;

const composedFn = compose(subtract10, double, add1);
composedFn(3); // (3 + 1) * 2 - 10 => -2
```

## Reference(s)

- Array [reduceRight](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight)

## Solution(s)

```ts
// Approach 1 

export default function compose(...fns: Array<Function>): Function {
  return function (arg: unknown) {
    if (fns.length === 0) {
      return arg;
    }

    return fns.reduceRight((acc, fun) => {
      return fun.call(null, acc);
    }, arg);
  };
}
```

```ts
// Approach 2

export default function compose(...fns: Array<Function>): Function {
  return function (x: any) {
    if (fns.length === 0) {
      return x;
    }

    function apply(...fns: Array<Function>): any {
      const [fn, ...rest] = fns;
      
      if (rest.length === 0) {
        return fn(x);
      }

      // Recurse to the end first, then unwind from right to left.
      return fn(apply(...rest));
    }

    return apply(...fns);
  };
}
```
