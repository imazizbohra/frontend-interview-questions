# Techniques

## Check out-of-bound index

```ts
if (index < -length || index >= length) {
  // out-of-bound index
}
```

## Normalizing array index

```ts
const nIndex =
    index < 0 ? Math.max(index + length, 0) : Math.min(index, length);
```

## Find type of a value

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

## Check if a value is primitive

```ts
const isPrimitive = (
  value: unknown,
): value is null | undefined | string | number | boolean | symbol | bigint => {
  return Object(value) !== value;
};
```

## Check if a value is a POJO (Plain Old JavaScript Object)

```ts
function isPlainObject(value: unknown): boolean {
  if (value === null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);

  return proto === Object.prototype || proto === null;
}
```

## Count no of items in a range

```ts
// if step = 0, handle divide by zero
// if range is negative then step also becomes negative; 
// division by negative produces positive value

const count = (end - start) / (step || 1);
```

## Convert array to object

```ts
const obj = { ...array };
```

## Check empty slot in an array

```ts
if (index in array) {
  // non-empty slot
}
```

## Preserve empty slots in an array

```ts
const arr = [1, , 2, , ,];
const newArr = [];

for(let i = 0; i < arr.length; i++) {
  if (i in arr) {
    newArr[i] = arr[i];
  } else {
    newArr.length++;
  }
}

console.log(newArr) // [ 1, <1 empty item>, 2, <2 empty items> ]
```

## Check if argument is explicitly undefined

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

## Preserve arguments without bind

```ts
function collect(this: any, args: any[]): any {
  ...

  return function next(arg: any): any {
    const nextArgs = arguments.length === 0 ? args : [...args, arg];
    
    return collect(this, nextArgs);
  };
}

const args = arguments.length === 0 ? [] : [arg]
return collect(this, args);
```

## Traverse nodes of a DOM tree bottom-up

```ts
// used internally by event bubble and capture phase

let current: Node | null = node.parentNode;
while (current) {
  current = current.parentNode;
}
```

## Traverse nodes of a DOM subtree in document order

```ts
// Approach 1

const iterator = document.createNodeIterator(
      root,
      NodeFilter.SHOW_ELEMENT,
    );
    
let node: Node | null = iterator.nextNode();
while (node) {
  // do some ops

  node = iterator.nextNode();
}
```

```ts
// Approach 2

function traverse(el: Element) {
  if (el == null) {
    return;
  }

  // do some ops

  for (const child of el.children) {
    traverse(child);
  }
}

for (const child of element.children) {
  traverse(child);
}
```

## Collect all the paths of the object

```ts
function traverse(
    obj: Object,
    path: Array<string>,
  ) {
    for (const [key, value] of Object.entries(obj)) {
      if (typeof value !== 'object' || value === null) {
        // if value is primite or null
        // hit the leaf 
        console.log(path.concat(key).filter(Boolean), value) // [a, b, c] 3
      } else {
        traverse(value, path.concat(key));
      }
    }
  }

traverse(obj, []);
```

## Check if throttle window has already elapsed

```ts
const remaining = interval - (Date.now() - lastPublishedAt);
if (remaining <= 0) {
  // do something
  return;
}

// or

if (interval < (Date.now() - lastPublishedAt)) {
  // do something
  return;
}
```

## Cancel Promise with AbortController

```ts
const controller = new AbortController();
const signal = controller.signal;

new Promise<T>((resolve, reject) => {
  let isAborted = false;

  signal.addEventListener("abort", () => {
    isAborted = true;
  });

  // after running some async task
  if (isAborted) {
    reject(new Error("Aborted"));
  };

  resolve(result);
});

controller.abort();
```
