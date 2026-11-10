# Count By

Implement `countBy(array, iteratee)` so it creates an object whose keys are the results of calling `iteratee` on each element of `array`. The value of each key is the number of elements that produced that result. Do not modify the original array.

## Argument(s)

- array (Array): The array to iterate over.
- iteratee (Function): The function used to produce a key for each element. It is invoked with one argument: (value).

## Returns

(Object): Returns the aggregate object.

## Example(s)

```ts
countBy([6.1, 4.2, 6.3], Math.floor);
// => { '4': 1, '6': 2 }

countBy([{ n: 3 }, { n: 5 }, { n: 3 }], (o) => o.n);
// => { '3': 2, '5': 1 }

countBy([], (o) => o); // => {}

countBy([{ n: 1 }, { n: 2 }], (o) => o.m); // => { undefined: 2 }
```

## Reference(s)

- Lodash [_.countBy](https://lodash.com/docs/4.18.1#countBy)

## Solution(s)

```ts
export default function countBy<T>(
  array: Array<T>,
  iteratee: (value: T) => number | string,
): Record<string, number> {
  const map: Record<string, number> = Object.create(null);

  for (let elem of array) {
    const key = String(iteratee.call(null, elem));

    if (key in map) {
      map[key]++;
    } else {
      map[key] = 1;
    }
  }

  return map;
}
```
