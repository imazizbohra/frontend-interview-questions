# Limit

Implement a function that accepts a callback and a number n, which restricts invocation of the callback to at most n times. Subsequent calls to the created function will return the result of the last invocation of the callback. The callback is invoked with the this binding and arguments from the created function.

## Argument(s)

- n (number): The number of calls at which func is no longer invoked.
- func (Function): The function to restrict.

## Returns

(Function): Returns the new restricted function.

## Example(s)

```ts
let i = 1;

function incrementBy(value) {
  i += value;
  return i;
}

const incrementByAtMostThrice = limit(incrementBy, 3);
incrementByAtMostThrice(2); // i is now 3; the function returns 3.
incrementByAtMostThrice(3); // i is now 6; the function returns 6.
incrementByAtMostThrice(4); // i is now 10; the function returns 10.
incrementByAtMostThrice(5); // i is still 10 because this is the 4th invocation; the function returns 10 because it is the result of the last invocation.
i = 4;
incrementByAtMostThrice(2); // i is still 4 because it is not modified; the function still returns 10.
```

## Reference(s)

- Lodash [_.before](https://lodash.com/docs/4.18.1#before)

## Solution(s)

```ts
// Function signature for callback argument.
type Func<TParams extends any[], TResult> = (...args: TParams) => TResult;

export default function limit<TParams extends any[], TResult>(
  func: Func<TParams, TResult>,
  n: number,
): Func<TParams, TResult> {
  let calledTimes: number = 0;
  let result: TResult;

  return function (this: any, ...args: TParams): TResult {
    if (calledTimes >= n) {
      return result;
    }

    result = func.call(this, ...args);
    calledTimes++;
    return result;
  };
}
```
