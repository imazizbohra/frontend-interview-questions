# Function.prototype.bind

Implement your own `Function.prototype.bind` without calling the native bind method. To avoid overwriting the actual `Function.prototype.bind`, implement the function as `Function.prototype.myBind`.

## Example(s)

```ts
const john = {
  age: 42,
  getAge: function () {
    return this.age;
  },
};

const unboundGetAge = john.getAge;
console.log(unboundGetAge()); // undefined

const boundGetAge = john.getAge.myBind(john);
console.log(boundGetAge()); // 42
```

## Reference(s)

- Operators [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- Operators [delete](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete)
- Global Object [globalThis](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis)
- Global Object [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)
- Global Object [defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- Global Object [Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
- Function [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

## Solution(s)

```ts
Function.prototype.myBind = function (
  this: Function,
  thisArg: any,
  ...argArray: any[]
) {
  const context = thisArg == null ? globalThis : Object(thisArg);
  const originalFun: Function = this;

  if (typeof originalFun !== "function") {
    throw new TypeError("Bind must be called on a function");
  }

  return function (...args: any[]) {
    const fnKey = Symbol();
    Object.defineProperty(context, fnKey, {
      value: originalFun,
      enumerable: false,
      configurable: true,
    });

    try {
      const result = context[fnKey](...[...argArray, ...args]);
      delete context[fnKey];
      return result;
    } finally {
      delete context[fnKey];
    }
  };
};
```
