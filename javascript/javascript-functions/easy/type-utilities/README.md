# Type Utilities

JavaScript is a dynamically typed language, which means the types of variables can change at runtime. Many interview questions involve recursively traversing values that contain different value types, and each type may require different handling (e.g. different code is needed to iterate over an array vs. an object). Understanding JavaScript types is crucial to solving questions like Deep Clone and Deep Equal.

Implement the following utility functions to determine the types of primitive values.

- isBoolean(value): Return true if value is a boolean, false otherwise.
- isNumber(value): Return true if value is a number, false otherwise. Note that NaN is considered a number.
- isNull(value): Return true if value is null, false otherwise.
- isString(value): Return true if value is a string, false otherwise.
- isSymbol(value): Return true if value is a symbol primitive, false otherwise.
- isUndefined(value): Return true if value is undefined, false otherwise.

## Reference(s)

- Operators [typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)
- Object [getPrototypeOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf)
- Object [toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)
- String [toLowerCase](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/toLowerCase)
- Jest [getType](https://github.com/jestjs/jest/blob/main/packages/jest-get-type/src/index.ts)


## Solution(s)

```ts
// Approach 1

export function isBoolean(value: unknown): boolean {
  return typeof value === 'boolean';
}

export function isNumber(value: unknown): boolean {
  return typeof value === 'number';
}

export function isNull(value: unknown): boolean {
  return value === null;
}

export function isString(value: unknown): boolean {
  return typeof value === 'string';
}

export function isSymbol(value: unknown): boolean {
  return typeof value === 'symbol';
}

export function isUndefined(value: unknown): boolean {
  return value === undefined;
}
```

```ts
// Approach 2

type ValueType =
  | 'array'
  | 'bigint'
  | 'boolean'
  | 'function'
  | 'null'
  | 'number'
  | 'object'
  | 'regexp'
  | 'map'
  | 'set'
  | 'date'
  | 'string'
  | 'symbol'
  | 'undefined';

const getType = (value: unknown): ValueType => {
  // explicitly handle null & undefined
  if (value === null) return "null";

  if (value === undefined) return "undefined";

  return Object.getPrototypeOf(value)?.constructor?.name?.toLowerCase() ?? "object";
};

export function isBoolean(value: unknown): boolean {
  return getType(value) === "boolean";
}

export function isNumber(value: unknown): boolean {
  return getType(value) === "number";
}

export function isNull(value: unknown): boolean {
  return getType(value) === 'null';
}

export function isString(value: unknown): boolean {
  return getType(value) === "string";
}

export function isSymbol(value: unknown): boolean {
  return getType(value) === "symbol";
}

export function isUndefined(value: unknown): boolean {
  return getType(value) === 'undefined';
}
```

```ts
// Approach 3

type ValueType =
  | '[object Array]'
  | '[object BigInt]'
  | '[object Boolean]'
  | '[object Function]'
  | '[object Null]'
  | '[object Number]'
  | '[object Object]'
  | '[object RegExp]'
  | '[object Map]'
  | '[object Set]'
  | '[object Date]'
  | '[object String]'
  | '[object Symbol]'
  | '[object Undefined]';

const getType = (value: unknown): ValueType => {
  return Object.prototype.toString.call(value) as ValueType;
};

export function isBoolean(value: unknown): boolean {
  return getType(value) === "[object Boolean]";
}

export function isNumber(value: unknown): boolean {
  return getType(value) === "[object Number]";
}

export function isNull(value: unknown): boolean {
  return getType(value) === '[object Null]';
}

export function isString(value: unknown): boolean {
  return getType(value) === "[object String]";
}

export function isSymbol(value: unknown): boolean {
  return getType(value) === "[object Symbol]";
}

export function isUndefined(value: unknown): boolean {
  return getType(value) === '[object Undefined]';
}
```

```ts
// getType will handle all the types  

console.log(getType([])); // 'array'
console.log(getType(BigInt(9007199254740991))); // 'bigint'
console.log(getType(true)); // 'boolean'
console.log(getType(() => {})); // 'function'
console.log(getType(null)); // 'null'
console.log(getType(42)); // 'number'
console.log(getType({})); // 'object'
console.log(getType(/ab+c/i)); // 'regexp'
console.log(getType(new Map())); // 'map'
console.log(getType(new Set())); // 'set'
console.log(getType(new Date())); // 'date'
console.log(getType("hello world")); // 'string'
console.log(getType(Symbol("hello"))); // 'symbol'
console.log(getType(undefined)); // 'undefined'
console.log(getType(Object.create(null))); // 'object'
```
