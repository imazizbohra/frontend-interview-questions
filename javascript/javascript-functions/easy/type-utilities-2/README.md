# Type Utilities 2

JavaScript is a dynamically typed language, which means the types of variables can change at runtime. Many interview questions involve recursively traversing objects that can hold different value types, and each type may require different handling (e.g. different code is needed to iterate over an array vs. an object). Understanding JavaScript types is crucial to solving questions like `Deep Clone` and `Deep Equal`.

The [Type Utilities](../type-utilities/) question covered utility functions for primitive values. Implement the following utility functions to determine the types of non-primitive values.

- isArray(value): Return true if value is an array, false otherwise.
- isFunction(value): Return true if value is a function, false otherwise.
- isObject(value): Return true if value is an object (e.g. arrays, functions, plain objects, etc., excluding null and undefined), false otherwise.
- isPlainObject(value): Return true if value is a plain object, false otherwise (for arrays, functions, etc).
- A plain object, or what is commonly known as a Plain Old JavaScript Object (POJO), is any object whose prototype is Object.prototype or an object created via Object.create(null).

## Techniques

- Familiarity with JavaScript types.
- Objects and prototypes.

## Solution(s)

```ts
export function isArray(value: unknown): boolean {
  return Array.isArray(value);
}

export function isFunction(value: unknown): boolean {
  return typeof value === "function";
}

export function isObject(value: unknown): boolean {
  return (typeof value === "object" && value !== null) || isFunction(value);
}

export function isPlainObject(value: unknown): boolean {
  if (value === null || typeof value !== "object") {
    return false;
  }

  const proto = Object.getPrototypeOf(value);

  return proto === Object.prototype || proto === null;
}
```
