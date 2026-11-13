# Memoize

A memoize function is a higher-order function that takes a function and returns a memoized version of it. The memoized function caches the results of expensive function calls and returns the cached result when it receives the same inputs again. This can significantly improve the performance of functions that involve complex processing or significant latency and are called with the same arguments repeatedly.

Implement a function `memoize(func)` that takes a function parameter `func` and returns a memoized version of the function. You may assume that func only accepts a string or number as its only argument.

When the memoized function computes a result, it should invoke `func` with the memoized function's `this` binding.

## Example(s)

```ts
function expensiveFunction(n) {
  console.log('Computing...');
  return n * 2;
}

// Create a memoized version of the function.
const memoizedExpensiveFunction = memoize(expensiveFunction);

// First call (computes and caches the result).
console.log(memoizedExpensiveFunction(5)); // Output: Computing... 10

// Second call with the same argument (returns the cached result).
console.log(memoizedExpensiveFunction(5)); // Output: 10

// Third call with a different argument (computes and caches the new result).
console.log(memoizedExpensiveFunction(10)); // Output: Computing... 20

// Fourth call with the same argument as the third call (returns the cached result).
console.log(memoizedExpensiveFunction(10)); // Output: 20
```

## Reference(s)

- Javascript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- Function [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## Solution(s)

```ts
type Fn = (this: any, arg: string | number) => unknown;

export default function memoize(func: Fn): Fn {
  const cache = new Map<string | number, any>();

  const fn: Fn = function memoized(this, arg) {
    // cache hit
    if (cache.has(arg)) {
      return cache.get(arg);
    }

    // cache miss
    const result = func.call(this, arg);
    cache.set(arg, result);
    return result;
  };

  return fn;
}
```
