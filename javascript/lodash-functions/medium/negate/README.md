# Negate

Implement a function `negate(predicate)` that creates a wrapper that negates the result of the predicate function. The predicate is invoked with the this binding and arguments of the created function.

## Argument(s)

predicate (Function): The predicate to negate.

## Returns

(Function): Returns the new negated function.

## Example(s)

```ts
const isOdd = negate(isEven);
isOdd(1); // => true
isOdd(2); // => false
```

## Reference(s)

- Lodash [_.chunk](https://lodash.com/docs/4.18.1#negate)

## Techniques

- Closures
- Higher-order functions
- Handling variadic arguments
- Forwarding `this` and arguments before negating the predicate result

## Solution(s)

```ts
export default function negate<P extends Array<any>>(
  predicate: (this: any, ...args: P) => boolean,
): (...args: P) => boolean {
  return function (this: any, ...args: P): boolean {
    const result: boolean = predicate.call(this, ...args);

    return !result;
  };
}
```
