# Classnames

`classnames` is a commonly used utility in modern frontend applications to conditionally join CSS class names together. If you've written React applications, you likely have used a similar library.

Implement the `classnames` function.

## Example(s)

```ts
classNames('foo', 'bar'); // 'foo bar'
classNames('foo', { bar: true }); // 'foo bar'
classNames({ 'foo-bar': true }); // 'foo-bar'
classNames({ 'foo-bar': false }); // ''
classNames({ foo: true }, { bar: true }); // 'foo bar'
classNames({ foo: true, bar: true }); // 'foo bar'
classNames({ foo: true, bar: false, qux: true }); // 'foo qux'

// Arrays will be recursively flattened as per the rules above.
classNames('a', ['b', { c: true, d: false }]); // 'a b c'

// Values can be mixed.
classNames(
  'foo',
  {
    bar: true,
    duck: false,
  },
  'baz',
  { quux: true },
); // 'foo bar baz quux'

// Falsy values are ignored.
classNames(null, false, 'bar', undefined, { baz: null }, ''); // 'bar'
```

## Reference(s)

- Github [classnames](https://github.com/JedWatson/classnames)
- Github [clsx](https://github.com/lukeed/clsx)

## Solution(s)

```ts
export type ClassValue =
  | ClassArray
  | ClassDictionary
  | string
  | number
  | null
  | boolean
  | undefined;
export type ClassDictionary = Record<string, any>;
export type ClassArray = Array<ClassValue>;

export default function classNames(...args: Array<ClassValue>): string {
  const array: string[] = [];

  function isPrimitive(value: unknown) {
    return Object(value) !== value;
  }

  function isPojo(value: unknown) {
    if (value == null || typeof value !== "object") {
      return false;
    }

    const prototype = Object.getPrototypeOf(value);

    return prototype === Object.prototype || prototype === null;
  }

  function clcx(token: unknown): string {
    // handle primitives
    if (isPrimitive(token)) {
      if (typeof token === "string" && Boolean(token)) {
        return token;
      } else if (typeof token === "number" && Boolean(token)
      ) {
        return String(token);
      } else if (typeof token === "boolean" && Boolean(token)) {
        return String(" ");
      }
    }

    // handle arrays
    if (Array.isArray(token)) {
      let classNames = "";
      for (const elem of token) {
        const className = clcx(elem);
        if (Boolean(className)) {
          classNames = classNames + " " + className;
        }
      }
      return classNames;
    }

    // handle objects
    if (isPojo(token)) {
      let classNames = "";
      for (const [key, value] of Object.entries(token as ClassDictionary)) {
        const className = clcx(value);
        if (Boolean(className)) {
          classNames = classNames + " " + key;
        }
      }
      return classNames;
    }

    return "";
  }

  for (const elem of args) {
    array.push(clcx(elem));
  }

  return array.join(" ").trim().replace(/\s+/g, " ");
}
```
