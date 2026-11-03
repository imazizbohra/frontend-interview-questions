# Max By

Implement a function `maxBy(array, iteratee)` that finds the element inside array with the maximum value returned by iteratee. The iteratee is invoked with one argument: (value).

## Argument(s)

- array (Array): The array to iterate over.
- iteratee (Function): The iteratee invoked per element.

## Returns

(*): Returns the element whose iteratee result is the maximum.

## Example(s)

```ts
maxBy([{ n: 1 }, { n: 2 }], (o) => o.n); // => { n: 2 }
maxBy([1, 2], (o) => -o); // => 1
```

The function should ignore elements where iteratee produces null or undefined.

If multiple elements produce the same maximum iteratee result, return the first one.

```ts
maxBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => undefined
```

## Reference(s)

- Array [from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- Array [forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
- Array [sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- Javascript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- Lodash [_.maxBy](https://lodash.com/docs/4.18.1#maxBy)

## Solution(s)

```ts
// Approach 1

export default function maxBy<T>(
  array: Array<T>,
  iteratee: (value: T) => any,
): any {
  const map = new Map<any, T[]>();

  array.forEach((elem) => {
    const rank = iteratee(elem);

    if (rank) {
      if (!map.has(rank)) {
        const bucket = [];
        bucket.push(elem);
        map.set(rank, bucket);
      }
    }
  });

  const sortedEnteries = Array.from(map.entries()).sort(([a], [b]) => {
    if (typeof a === "number" && typeof b === "number") {
      return b - a;
    }

    if (typeof a === "string" && typeof b === "string") {
      return b.localeCompare(a);
    }

    if (a instanceof Date && b instanceof Date) {
      return b.getTime() - a.getTime();
    }

    return 0;
  });

  return sortedEnteries.length ? sortedEnteries[0][1][0] : undefined;
}
```

```ts
// Approach 2

export default function maxBy(array: Array<any>, iteratee: Function): any {
  let result: any, prevRank: any;

  for (const value of array) {
    const nextRank: any = iteratee(value);

    if (nextRank != null && (prevRank === undefined || nextRank > prevRank)) {
      prevRank = nextRank;
      result = value;
    }
  }

  return result;
}
```
