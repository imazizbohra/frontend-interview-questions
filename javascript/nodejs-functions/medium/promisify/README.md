# Promisify

Before `promises/async/await` became the standard, it was a convention for async APIs in JavaScript to accept callbacks as the last argument. Many async versions of Node.js APIs (e.g. `fs.readFile` and `fs.rm`) have such signatures. Node.js' `util.promisify` function was created to wrap callback-based functions in versions that return Promises so they can be used with `async/await`.

Implement a function promisify that takes a function following the common callback-last, error-first style, i.e. taking a `(err, value) => ...` callback as the last argument, and returns a version that returns promises.

When invoking the original function, the returned function should preserve its call-time `this` value, forward all supplied arguments, and append the callback as the final argument.

## Example(s)

```ts
// Example function with callback as last argument
// The callback has the signature `(err, value) => any`
function foo(url, options, callback) {
  apiCall(url, options)
    .then((data) => callback(null, data))
    .catch((err) => callback(err));
}

const promisifiedFoo = promisify(foo);
const data = await promisifiedFoo('example.com', { foo: 1 });
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- NodeJs [util.promisify(original)](https://nodejs.org/api/util.html#utilpromisifyoriginal)

## Solution(s)

```ts
export default function promisify<T>(
  func: (...args: any[]) => void,
): (this: any, ...args: any[]) => Promise<T> {
  return function (this: any, ...args: any[]) {
    const promise = new Promise<T>((res, rej) => {
      const callback = (error: any, value: T) => {
        if (error) {
          rej(error);
        }

        res(value);
      };

      func.call(this, ...args, callback);
    });

    return promise;
  };
}
```
