# Memoize II

A memoize function is a higher-order function that takes a function and returns a memoized version of it. The memoized function caches the results of expensive function calls and returns the cached result when it receives the same inputs again. This can significantly improve the performance of functions that involve complex processing or significant latency and are called with the same arguments repeatedly.

Implement a function `memoize(func)` that takes a function parameter `func` and returns a memoized version of the function. You may assume that `func` only accepts strings or numbers as arguments.

When the memoized function computes a result, it should invoke `func` with the memoized function's `this` binding.

## Examples

```ts
function expensiveMul(a, b) {
    console.log('Computing...');
    return a * b;
}

// Create a memoized version of the function.
const memoizedExpensiveMul = memoize(expensiveMul);

// First call (computes and caches the result).
console.log(memoizedExpensiveMul(3, 7)); // Output: Computing... 21

// Second call with the same arguments (returns the cached result).
console.log(memoizedExpensiveMul(3, 7)); // Output: 21

// Third call with different arguments (computes and caches the new result).
console.log(memoizedExpensiveMul(5, 8)); // Output: Computing... 40

// Fourth call with the same arguments as the third call (returns the cached result).
console.log(memoizedExpensiveMul(5, 8)); // Output: 40
```

## References

- Javascript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- Function [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
- Lodash [_.memoize](https://lodash.com/docs/4.18.1#memoize)

## Solutions

```ts
type Fn<T> = (this: any, ...args: Array<any>) => T;

export default function memoize<T>(func: Fn<T>): Fn<T> {
  const cache = new Map<string, T>();

  const fn: Fn<T> = function memoized(this: any, ...args: Array<any>) {
    const cacheKey = JSON.stringify(args);

    // cache hit
    if (cache.has(cacheKey)) {
      return cache.get(cacheKey) as T;
    }

    // cache miss
    const result = func.call(this, ...args);
    cache.set(cacheKey, result);

    return result as T;
  };

  return fn;
}
```
