# Array.prototype.apply

Implement your own `Function.prototype.apply` without calling the native apply method. To avoid overwriting the actual `Function.prototype.apply`, implement the function as `Function.prototype.myApply`.

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

multiplyAge.myApply(mary); // 21
multiplyAge.myApply(john, [2]); // 84
```

## Reference(s)

- Operators [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- Operators [delete](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete)
- Global Object [globalThis](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis)
- Global Object [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)
- Global Object [defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- Global Object [Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
- Function [apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

## Solution(s)

```ts
Function.prototype.myApply = function (thisArg, args) {
  const context: Record<string | symbol, any> =
    thisArg == null ? globalThis : Object(thisArg);
  const originalFun: Function = this;
  const originalArgs: any[] = args || [];

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
