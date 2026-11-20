# Compact 2

Implement a function `compact(value)` that returns a new object with all falsy values removed, including falsy values that are deeply nested. You can assume value only contains JSON-serializable values (`null`, `boolean`, `number`, `string`, `Array`, `Object`) and will not contain any other objects like `Date`, `RegExp`, `Map`, or `Set`.

The values `false`, `null`, `0`, `''`, `undefined`, and `NaN` are falsy.

## Argument(s)

`value (Array|Object)`: The array/object to compact.

## Returns

`(Array|Object)`: Returns the new compact array/object.

## Example(s)

```ts
compact([0, 1, false, 2, '', 3, null]); // => [1, 2, 3]
compact({ foo: true, bar: null }); // => { foo: true }
```

## Reference(s)

- Array [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- Lodash [_.compact](https://lodash.com/docs/4.18.1#compact)

## Solution(s)

```ts
export default function compact<T extends Array<any> | Object>(value: T): T {
  if (Array.isArray(value)) {
    return value.reduce((acc, elem) => {
      if (Boolean(elem)) {
        acc.push(compact(elem));
      }

      return acc;
    }, []);
  }

  if (typeof value === "object" && value !== null) {
    return Object.entries(value).reduce(
      (acc, [key, val]) => {
        if (Boolean(val)) {
          acc[key] = compact(val);
        }

        return acc;
      },
      {} as Record<string, unknown>,
    ) as T;
  }

  return value;
}
```
