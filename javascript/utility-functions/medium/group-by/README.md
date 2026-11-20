# Group By

Implement `groupBy(array, iteratee)` so it creates an object whose keys are the results of calling `iteratee` on each element of `array`. Each key stores an array of the original elements that produced that result. Do not modify the original array.

## Argument(s)

- array (Array): The array to iterate over.
- iteratee (Function): The function used to produce a key for each element. It is invoked with one argument: (value).

## Returns

(Object): Returns the composed aggregate object.

## Example(s)

```ts
groupBy([6.1, 4.2, 6.3], Math.floor);
// => { '4': [4.2], '6': [6.1, 6.3] }

groupBy([{ n: 3 }, { n: 5 }, { n: 3 }], (o) => o.n);
// => { '3': [{ n: 3 }, { n: 3 }], '5': [{ n: 5 }] }

groupBy([], (o) => o); // => {}

groupBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => { undefined: [{ n: 1 }, { n: 2 }] }
```

## Reference(s)

- Lodash [_.groupBy](https://lodash.com/docs/4.18.1#groupBy)

## Solution(s)

```ts
export default function groupBy<T>(
  array: Array<T>,
  iteratee: (value: T) => number | string,
): Record<string, Array<T>> {
  const map: Record<string, T[]> = Object.create(null);

  for (let elem of array) {
    const key = String(iteratee.call(null, elem));

    if (!(key in map)) {
      map[key] = [];
    }

    map[key].push(elem);
  }

  return map;
}
```
