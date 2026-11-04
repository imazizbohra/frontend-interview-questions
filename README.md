# frontend-interview-questions

Collection of frontend interview questions

## Javascript

- [Easy - Design Pattern](./javascript/design-pattern/easy/)
- [Easy - Javascript Functions](./javascript/javascript-functions/easy/)
- [Easy - jQuery Functions](./javascript/jquery-functions/easy/)
- [Easy - Lodash Functions](./javascript/lodash-functions/easy/)
- [Easy - Promise Functions](./javascript/promise-functions/easy/)
- [Easy - Prototype Functions](./javascript/prototype-functions/easy/)

## React

- [Easy - Hooks](./react/hooks/easy/)

## Techniques

### Check out-of-bound index

```ts
if (index < -length || index >= length) {
  // out-of-bound index
}
```

### Normalizing array index

```ts
const nIndex =
    index < 0 ? Math.max(index + length, 0) : Math.min(index, length);
```

### Find type of a value

```ts
// output format: "array"

const getType = (value) => {
  if (value === null) return "null";

  if (value === undefined) return "undefined";

  return Object.getPrototypeOf(value)?.constructor?.name?.toLowerCase() ?? "object";
};
```

```ts
// output format: "[object Array]"

const getType = (value) => {
  return Object.prototype.toString.call(value);
};
```

### Check if a value is primitive

```ts
const isPrimitive = (
  value: unknown,
): value is null | undefined | string | number | boolean | symbol | bigint => {
  return Object(value) !== value;
};
```

### Count no of items in a range

```ts
// if step = 0, handle divide by zero
// if range is negative then step also becomes negative; 
// division by negative produces positive value

const count = (end - start) / (step || 1);
```

### Check empty slot in an array

```ts
if (index in array) {
  // non-empty slot
}
```

### Check if argument is explicitly undefined

```ts
// used in Array.prototype.reduce 

(function (arg) {
  console.log(arguments.length) // 0
})()
```

```ts
(function (arg) {
  console.log(arguments.length) // 1
})(undefined)
```

### Tree traversal bottom-up

```ts
// used internally by event bubble and capture phase

let current: Node | null = node.parentNode;
while (current) {
  current = current.parentNode;
}
```

## Credits

- [greatfrontend.com](https://www.greatfrontend.com/)
- [patterns.dev](https://www.patterns.dev/)
