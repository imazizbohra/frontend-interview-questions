# Curry III

**Note**: This is an advanced version of [Curry II](../../medium/curry-2/); you should complete that first before attempting this question. This is a hard question and resembles a brain teaser more than a typical interview question candidates are expected to solve.

Currying is the technique of converting a function that takes multiple arguments into a sequence of functions that each take a single argument.

Implement the `curry` function, which accepts a function as its only argument and returns a function that accepts any number of arguments (rather than only one argument at a time in Curry or a fixed number of arguments in Curry II). The returned function can be called repeatedly.

## Expected output behavior

When the returned function is in an expression that suggests the value should be a string or a number, the original function is invoked with the provided arguments and the result is used as the value.

## Examples

```ts
function multiply(...numbers) {
  return numbers.reduce((a, b) => a * b, 1);
}
const curriedMultiply = curry(multiply);
const multiplyByThree = curriedMultiply(3);
console.log(+multiplyByThree); // 3
console.log(+multiplyByThree(4)); // 12

const multiplyByFifteen = multiplyByThree(5);
console.log(+multiplyByFifteen); // 15
console.log(+multiplyByFifteen(2)); // 30

console.log(+curriedMultiply(1)(2)(3)(4)); // 24
console.log(+curriedMultiply(1, 2, 3, 4)); // 24
```

The unary `+` triggers numeric coercion and evaluates the accumulated arguments. Passing the returned function directly to `console.log` does not trigger this coercion.

## References

- JavaScript [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- Function [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

## Solutions

```ts
export default function curry(func: Function): Function {
  function curried(this: any, ...args: any[]) {
    const fn: any = curried.bind(this, ...args);

    fn[Symbol.toPrimitive] = () => {
      return func.call(this, ...args);
    };

    return fn;
  }

  return curried;
}
```
