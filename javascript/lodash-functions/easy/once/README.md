# Once

Sometimes it's helpful to ensure a function runs only once during the lifecycle of a website, e.g. for setting up logging, initializing an environment, etc.

Implement a function that accepts a callback and restricts its invocation to at most once. Subsequent calls to the returned function will return the result of the first invocation of the callback function. The callback function is invoked with the this binding and arguments of the created function.

## Example(s)

```ts
let i = 1;

function incrementBy(value) {
  i += value;
  return i;
}

const incrementByOnce = once(incrementBy);
incrementByOnce(2); // i is now 3; The function returns 3.
incrementByOnce(3); // i is still 3; The function returns the result of the first invocation, which is 3.
i = 4;
incrementByOnce(2); // i is still 4 as it is not modified. However, the function returns the result of the first invocation, which is 3.
```

## Notes

- Only the first invocation calls the original function.
- Later calls return the first result, even when called with different arguments or a different receiver.
- Use a separate boolean flag instead of checking the cached value because the first result can be falsy.
- Side effects from func should happen once as well. This is why later calls must skip func entirely rather than calling it again and discarding its result.

## Reference(s)

- Javascript [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- Operators [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- Lodash [_.once](https://lodash.com/docs/4.18.1#once)

## Solution(s)

```ts
type Fn<T> = (this: any, args: Array<any>) => T;

export default function once<T>(func: Fn<T>): Fn<T> {
  let isFirstInvocation: boolean = true;
  let returnValue: T;

  return function (...args) {
    if (isFirstInvocation) {
      isFirstInvocation = false;
      returnValue = func.apply(this, args);
    }

    return returnValue;
  };
}
```
