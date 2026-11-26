# Classnames II

The classnames question introduced `classnames`, a commonly used utility in modern frontend applications to conditionally join CSS class names together. However, there are some cases the library does not handle:

| Case | Example | Original | Improved |
| --- | --- | --- | --- |
| De-duplicate classnames | `classNames('foo', 'foo')` | `'foo foo'` | `'foo'` |
| Turn off classnames | `classNames('foo', 'bar', { foo: false })` | `'foo bar'` | `'bar'` |
| Function values | `classNames(() => 'foo')` | Unsupported | `'foo'` |

Implement an improved version of the classnames function that handles the above cases.

## Examples

```ts
classNames('foo', 'foo'); // 'foo'
classNames({ foo: true }, { foo: true }); // 'foo'
classNames({ foo: true, bar: true }, { foo: false }); // 'bar'
classNames('foo', () => 'bar'); // 'foo bar'
classNames('foo', () => 'foo'); // 'foo'
```

## References

- Github [classnames](https://github.com/JedWatson/classnames)
- Github [clsx](https://github.com/lukeed/clsx)

## Solutions

```ts
export type ClassValue =
  | ClassArray
  | ClassDictionary
  | Function
  | string
  | number
  | null
  | boolean
  | undefined;
export type ClassDictionary = Record<string, any>;
export type ClassArray = Array<ClassValue>;

export default function classNames(...args: Array<ClassValue>): string {
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

  function clcx(token: unknown): any {
    // handle primitives
    if (isPrimitive(token)) {
      if (typeof token === "string" && Boolean(token)) {
        return token;
      } else if (typeof token === "number" && Boolean(token)) {
        return String(token);
      } else if (typeof token === "boolean") {
        return token;
      }
    }

    // handle function
    if (typeof token === "function") {
      const className = clcx(token());

      if (Boolean(className)) {
        return className;
      }
    }

    // handle arrays
    if (Array.isArray(token)) {
      for (const elem of token) {
        const className = clcx(elem);

        if (Boolean(className)) {
          acc.add(className);
        }
      }
    }

    // handle objects
    if (isPojo(token)) {
      for (const [key, value] of Object.entries(token as ClassDictionary)) {
        const className = clcx(value);

        if (Boolean(className)) {
          acc.add(key);
        } else {
          acc.delete(key);
        }
      }
    }
  }

  let acc: Set<string> = new Set();

  clcx([...args]);

  return Array.from(acc).join(" ").trim().replace(/\s+/g, " ");
}
```
