# Promisify 2

Note: This is an advanced version of [Promisify](../promisify/), which you should complete before attempting this question.

In Promisify, the promisify function returns a promise-returning version of a function following the common callback-last, error-first style, i.e. taking a `(err, value) => ...` callback as the last argument. However, promisify does not work for functions that do not follow that exact signature.

In Node.js, a function can use the `util.promisify.custom` symbol to override the return value of `util.promisify()`. This is useful when the original function does not follow the standard format of taking an error-first callback as the last argument, especially for legacy formats that are incompatible with `util.promisify's` callback-last convention.

## Example(s)

```ts
// Example usage in Node.js.
const util = require('node:util');

function doSomething(callback, foo) {
  // ...
}

doSomething[util.promisify.custom] = (foo) => {
  return getPromiseSomehow();
};

const promisified = util.promisify(doSomething);
console.log(promisified === doSomething[util.promisify.custom]);
// prints 'true'
```

Implement a `promisify` function that supports custom return values. Use the symbol `Symbol.for('util.promisify.custom')` as the key for the overridden value.

```ts
// Example function with callback as the first argument.
// The callback has the signature `(err, value) => any`.
function foo(callback, url, options) {
  apiCall(url, options)
    .then((data) => callback(null, data))
    .catch((err) => callback(err));
}

foo[Symbol.for('util.promisify.custom')] = (url, options) => {
  return new Promise((resolve, reject) => {
    foo(
      (err, data) => {
        if (err) {
          reject(err);
        }
        resolve(data);
      },
      url,
      options,
    );
  });
};

const promisifiedFoo = foo[Symbol.for('util.promisify.custom')]; // true
const data = await promisifiedFoo('example.com', { foo: 1 });
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- NodeJs [util.promisify.custom](https://nodejs.org/api/util.html#custom-promisified-functions)

## Solution(s)

```ts
const promisifyCustomSymbol = Symbol.for("util.promisify.custom");

type Func = (...args: any[]) => void;
type Return<T> = (this: any, ...args: any[]) => Promise<T>;
type WithSymbol<T> = { [promisifyCustomSymbol]: Return<T> };
type Param<T> = Func | WithSymbol<T>;

export default function promisify<T>(func: Param<T>): Return<T> {
  if ((func as WithSymbol<T>)[promisifyCustomSymbol]) {
    return (func as WithSymbol<T>)[promisifyCustomSymbol];
  }

  return function (this: any, ...args: any[]) {
    const promise = new Promise<T>((res, rej) => {
      const callback = (error: any, value: T) => {
        if (error) {
          rej(error);
        }

        res(value);
      };

      if (typeof func === "function") {
        func.call(this, ...args, callback);
      }
    });

    return promise;
  };
}
```
