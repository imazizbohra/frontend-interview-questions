# Array.prototype.call

Implement your own `Function.prototype.call` without calling the native call method. To avoid overwriting the actual `Function.prototype.call`, implement the function as `Function.prototype.myCall`.

## Example(s)

```ts
function multiplyAge(multiplier = 1) {
  return this.age * multiplier;
}

const mary = {
  age: 21,
};

const john = {
  age: 42,
};

multiplyAge.myCall(mary); // 21
multiplyAge.myCall(john, 2); // 84
```

## Reference(s)

- Operators [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- Operators [delete](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete)
- Global Object [globalThis](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis)
- Global Object [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)
- Global Object [defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- Global Object [Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
- Function [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## Solution(s)

```ts
Function.prototype.myCall = function (thisArg, ...argArray) {
  const context: Record<string | symbol, any> =
    thisArg == null ? globalThis : Object(thisArg);
  const originalFun: Function = this;
  const originalArgs: any[] = argArray || [];

  const fnKey = Symbol();
  Object.defineProperty(context, fnKey, {
    value: originalFun,
    enumerable: false,
    configurable: true,
  });

  try {
    const result = context[fnKey](...originalArgs);
    delete context[fnKey];
    return result;
  } finally {
    delete context[fnKey];
  }
};
```
