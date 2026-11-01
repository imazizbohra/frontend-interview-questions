# Sleep

In languages like Java and Python, a sleep function is available to suspend execution of the calling thread. However, unlike those languages, JavaScript is single-threaded, and blocking the main thread is not a good idea. Therefore, let's implement an asynchronous version of the sleep function that works similarly but does not block the main thread.

## Argument(s)

duration (number): Duration to sleep for.

## Returns

(number): Returns an empty promise.

## Example(s)

```ts
async function greeting() {
  console.log('Hello!');
  await sleep(2000);
  console.log('Bye.'); // Only logs after 2000 milliseconds (2 seconds)
}

greeting();
// t = 0: Hello!
// t = 2000: Bye.
```

```ts
console.log('Hello!');
sleep(2000).then(() => {
  console.log('Bye.'); // Only logs after 2000 milliseconds (2 seconds)
});
```

## Reference(s)

- JavaScript [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- How to use promises [Promises](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises)

## Solution(s)

```ts
export default async function sleep(duration: number): Promise<void> {
  const promise: Promise<void> = new Promise<void>((res) => {
    setTimeout(() => {
      res();
    }, duration);
  });

  return promise;
}
```
