# Union By

Implement a function `unionBy(iteratee, ...arrays)` that creates an array of unique values, in order, from all given arrays and accepts an `iteratee`, which is invoked for each element of each array to generate the criterion by which uniqueness is computed.

## Argument(s)

- iteratee (Function): The iteratee invoked per element. The function is invoked with one argument: value.
- [arrays] (...Array): The arrays to inspect.

## Returns

- (Array): Returns the new array of combined values.

## Example(s)

```ts
unionBy((value: any) => value, [2], [1, 2]); // => [2, 1]
unionBy(Math.floor, [2.1], [1.2, 2.3]); // => [2.1, 1.2]
unionBy((o) => o.x, [{ x: 1 }], [{ x: 2 }, { x: 1 }]); // => [{ 'x': 1 }, { 'x': 2 }]
```

The function should return an empty array if the provided arrays are empty and treat falsy values as-is.

```ts
unionBy((o) => o.m, []); // => []

unionBy((o) => o.m, [{ n: 1 }], [{ m: 2 }]); // => [{ n: 1 }, { m: 2 }]
```

## Reference(s)

- Lodash [_.unionBy](https://lodash.com/docs/4.18.1#unionBy)

## Solution(s)

```ts
// Approach 1

export default function unionBy<T>(
  iteratee: (value: T) => any,
  ...arrays: Array<any>
): Array<T> {
  if (arrays.length === 0) {
    return [];
  }

  if (arrays.length === 1) {
    return Array.from(new Set(arrays[0]));
  }

  const set = new Set<T>();
  const visited = new Set<any>();

  const addToSet = (elem: T) => {
    const value = iteratee(elem);

    if (!visited.has(value)) {
      visited.add(value);
      set.add(elem);
    }
  };

  for (let elemA of arrays) {
    if (Array.isArray(elemA)) {
      for (let elemB of elemA) {
        addToSet(elemB);
      }
    } else {
      addToSet(elemA);
    }
  }

  return Array.from(set);
}
```

```ts
// Approach 2

export default function unionBy<T>(
  iteratee: (value: T) => any,
  ...arrays: Array<any>
): Array<T> {
  if (arrays.length === 0) {
    return [];
  }

  if (arrays.length === 1) {
    return Array.from(new Set(arrays[0]));
  }

  const map = new Map<any, T>();

  const addToMap = (elem: T) => {
    const value = iteratee(elem);

    if (!map.has(value)) {
      map.set(value, elem);
    }
  };

  for (let elemA of arrays) {
    if (Array.isArray(elemA)) {
      for (let elemB of elemA) {
        addToMap(elemB);
      }
    } else {
      addToMap(elemA);
    }
  }

  return Array.from(map.values());
}
```
