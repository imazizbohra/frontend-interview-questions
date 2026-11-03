# Promise.race

Implement a custom version of `Promise.race()`, a `promiseRace` function, except that the function takes an array instead of an iterable. Be sure to read the description carefully and implement accordingly!

## Example(s)

```ts
const p0 = new Promise((resolve) => {
  setTimeout(() => {
    resolve(42);
  }, 100);
});
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('Err!');
  }, 400);
});

await promiseRace([p0, p1]); // 42
```

```ts
const p0 = Promise.resolve(42);
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(21);
  }, 100);
});

await promiseRace([p0, p1]); // 42
```

```ts
const p0 = new Promise((resolve) => {
  setTimeout(() => {
    resolve(42);
  }, 400);
});
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('Err!');
  }, 100);
});

try {
  await promiseRace([p0, p1]);
} catch (err) {
  console.log(err); // 'Err!'
}
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)
- Promise [race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

## Solution(s)

```ts
export default function promiseRace<T extends readonly unknown[] | []>(
  iterable: T,
): Promise<Awaited<T[number]>> {
  const promise = new Promise<Awaited<T[number]>>((res, rej) => {
    iterable
      .map((elem) => {
        return Promise.resolve(elem);
      })
      .forEach((promise) => {
        promise.then(
          (result) => {
            res(result as Awaited<T[number]>);
          },
          (reason) => {
            rej(reason);
          },
        );
      });
  });

  return promise;
}
```
