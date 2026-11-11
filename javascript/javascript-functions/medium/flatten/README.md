# Flatten

Flattening an array means removing its nested array structure while keeping the values in their original order. For example, `[1, [2, [3]]]` becomes `[1, 2, 3]`.

This is useful when combining grouped records or collecting results produced in batches. The arrays may be nested at different depths, and their values can include numbers, strings, objects, or other data. Flattening should preserve those values while producing one sequence that callers can iterate over.

Implement a function `flatten` that returns a newly created array with all subarray elements concatenated recursively into a single level.

## Example(s)

```ts
// Single-level arrays are unaffected.
flatten([1, 2, 3]); // [1, 2, 3]

// Inner arrays are flattened into a single level.
flatten([1, [2, 3]]); // [1, 2, 3]
flatten([
  [1, 2],
  [3, 4],
]); // [1, 2, 3, 4]

// Flattens recursively.
flatten([1, [2, [3, [4, [5]]]]]); // [1, 2, 3, 4, 5]
```

## Reference(s)

- Array [flat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)

## Solution(s)

```ts
// Approach 1: Recursive

type ArrayValue<T> = T | Array<ArrayValue<T>>;

export default function flatten<T>(value: Array<ArrayValue<T>>): Array<T> {
  const array: T[] = [];

  for (let elem of value) {
    if (Array.isArray(elem)) {
      const flattenArray = flatten(elem);
      array.push(...flattenArray);
    } else {
      array.push(elem);
    }
  }

  return array;
}
```

```ts
// Approach 2: Using Queue

type ArrayValue<T> = T | Array<ArrayValue<T>>;

export default function flatten<T>(value: Array<ArrayValue<T>>): Array<T> {
  const res: Array<T> = [];
  // Work on a copy so the queue-style unshift/shift logic does not mutate the input.
  const copy = value.slice();

  while (copy.length) {
    const elem = copy.shift()!;
    
    if (Array.isArray(elem)) {
      // Put nested elements back at the front so their order is preserved.
      copy.unshift(...elem);
    } else {
      res.push(elem as T);
    }
  }

  return res;
}
```

```ts
// Approach 3: Using Splice

type ArrayValue<T> = T | Array<ArrayValue<T>>;

export default function flatten<T>(value: Array<ArrayValue<T>>): Array<T> {
  for (let i = 0; i < value.length; ) {
    const item = value[i];
    if (Array.isArray(item)) {
      // Stay on the same index because splicing may reveal another nested array there.
      value.splice(i, 1, ...item);
    } else {
      i++;
    }
  }

  return value as Array<T>;
}
```
