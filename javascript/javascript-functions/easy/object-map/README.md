# Object Map

Implement a function `objectMap(obj, fn)` to return a new object containing the results of calling a provided function on every value in the object. The function `fn` is called with a single argument, the value that is being mapped or transformed.

Map only the input object's own enumerable properties, keep the same property names, and do not recurse into nested values. Invoke `fn` with the original input object as its this value, and leave the input object unchanged.

## Argument(s)

- object (object): Object that is being mapped and transformed.
- function (Function): Transformation function

## Returns

(object): Returns new object with the same keys and transformed values.

## Example(s)

```ts
const double = (x) => x * 2;
objectMap({ foo: 1, bar: 2 }, double); // => { foo: 2, bar: 4}
```

## Reference(s)

- Object [hasOwn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn)
- Function [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## Solution(s)

```ts
export default function objectMap<V, R>(
  obj: Record<string, V>,
  fn: (val: V) => R,
): Record<string, R> {
  const returnValue: Record<string, R> = {};

  for (const key in obj) {
    if (Object.hasOwn(obj, key)) {
      const value = obj[key];

      returnValue[key] = fn.call(obj, value);
    }
  }

  return returnValue;
}
```
