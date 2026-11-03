# Min By

Implement a function `minBy(array, iteratee)` that finds the element inside array with the minimum value returned by iteratee.

## Argument(s)

- array (Array): The array to iterate over.
- iteratee (Function): The iteratee invoked per element.

## Returns

(*): Returns the element whose iteratee result is the minimum.

## Example(s)

```ts
minBy([2, 3, 1, 4], (num) => num); // => 1
minBy([{ n: 1 }, { n: 2 }], (o) => o.n); // => { n: 1 }
```

The function should ignore elements where iteratee produces null or undefined.

If multiple elements produce the same minimum iteratee result, return the first one.

```ts
minBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => undefined
```

## Reference(s)

- Array [from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- Array [forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
- Array [sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- Javascript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- Lodash [_.minBy](https://lodash.com/docs/4.18.1#minBy)

## Solution(s)

```ts
// Approach 1

export default function minBy<T>(
  array: Array<T>,
  iteratee: (value: T) => any,
): any {
  const map = new Map<any, T>();

  array.forEach((elem) => {
    const rank = iteratee(elem);

    if (rank && !map.has(rank)) {
      map.set(rank, elem);
    }
  });

  const sortedEnteries = Array.from(map.entries()).sort(([a], [b]) => {
    if (typeof a === "string" && typeof b === "string") {
      return a.localeCompare(b);
    }

    if (a instanceof Date && b instanceof Date) {
      return a.getTime() - b.getTime();
    }

    return a - b;
  });

  return sortedEnteries.length ? sortedEnteries[0][1] : undefined;
}
```

```ts
// Approach 2

export default function minBy<T>(
  array: Array<T>,
  iteratee: (value: T) => any,
): any {
  let result: any, prevRank: any;

  for (const value of array) {
    const nextRank: any = iteratee(value);

    if (nextRank != null && (prevRank === undefined || nextRank < prevRank)) {
      prevRank = nextRank;
      result = value;
    }
  }

  return result;
}
```
