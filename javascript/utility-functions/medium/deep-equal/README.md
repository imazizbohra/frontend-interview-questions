# Deep Equal

Implement a function `deepEqual` that performs a deep comparison between two values. It returns `true` if the two input values are deemed equal and `false` otherwise.

- Inputs are limited to `undefined`, `numbers`, `strings`, `booleans`, `null`, `arrays`, and plain `objects`.
- Arrays may be sparse. Their length and present indices are part of the structure being compared.
- There will not be cyclic objects, i.e. objects with circular references.

**Note**: Similar to [deep clone](../deep-clone-2/)

## Examples

```ts
deepEqual('foo', 'foo'); // true
deepEqual({ id: 1 }, { id: 1 }); // true
deepEqual([1, 2, 3], [1, 2, 3]); // true
deepEqual([{ id: '1' }], [{ id: '2' }]); // false
```

## Notes

- `Object.is` treats `NaN` as equal to itself and distinguishes +0 from -0.
- null, plain objects, and arrays are handled explicitly by the type checks.
- Cyclic objects, i.e. objects with circular references, are not handled.
- Property descriptors are not taken into account when comparing properties.
- Non-enumerable properties and symbol-keyed properties are not compared.
- Prototypes are not compared. Two plain-object-shaped values with different prototypes can still compare equal if their enumerable entries match.

## References

- Concept [deep-equal](https://developer.mozilla.org/en-US/docs/Glossary/Deep_equality)
- Lodash [_.isEqual](https://lodash.com/docs/4.18.1#isEqual)

## Solutions

```ts
function isPlainObject(value: unknown): boolean {
  if (value === null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);

  return proto === Object.prototype || proto === null;
}

export default function deepEqual(valueA: unknown, valueB: unknown): boolean {
  // handle array
  if (Array.isArray(valueA) && Array.isArray(valueB)) {
    if (valueA.length !== valueB.length) {
      return false;
    }

    for (let i = 0; i < valueA.length; i++) {
      if (!deepEqual(valueA[i], valueB[i])) {
        return false;
      }
    }

    return true;
  }

  // handle object
  if (isPlainObject(valueA) && isPlainObject(valueB)) {
    if (
      Object.entries(valueA as any).length !==
      Object.entries(valueB as any).length
    ) {
      return false;
    }

    for (let key in valueA as any) {
      if (
        !(key in (valueB as any)) ||
        !deepEqual((valueA as any)[key], (valueB as any)[key])
      ) {
        return false;
      }
    }

    return true;
  }

  return valueA === valueB;
}
```
