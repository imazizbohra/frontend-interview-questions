# Get

Before the optional chaining operator `(?.)` existed, it was sometimes troublesome to access deeply nested properties in huge JavaScript objects when some of the intermediate properties might not be present.

```ts
const john = {
  profile: {
    name: { firstName: 'John', lastName: 'Doe' },
    age: 20,
    gender: 'Male',
  },
};

const jane = {
  profile: {
    age: 19,
    gender: 'Female',
  },
};

function getFirstName(user) {
  return user.profile.name.firstName;
}
```

Doing `getFirstName(john)` works, but `getFirstName(jane)` will throw an error because the name property doesn't exist for jane.profile.

## Argument(s)

- object (Object): The object to query.
- path (string | Array<string | number>): The path to the property to get.
- [defaultValue] (*): The value to return when the resolved value is undefined.

## Returns

(*): Returns the resolved value, defaultValue, or undefined.

## Example(s)

```ts
// Using the `john` and `jane` objects from above.
get(john, 'profile.name.firstName'); // => 'John'
get(john, 'profile.gender'); // => 'Male'
get(jane, 'profile.name.firstName'); // => undefined

// An example where `path` is provided as an array.
get({ a: { c: { foo: 2 } } }, ['a', 'c']); // => { foo: 2 }

// Arrays inside the object can also be accessed with indices.
get({ a: [{ b: { c: 3 } }] }, 'a.0.b.c'); // => 3

// Accessing through a non-object value stops the lookup.
get({ a: { b: true } }, 'a.b.c'); // => undefined

// Return the default value when the resolved value is `undefined`.
get({}, 'a.b', 'fallback'); // => 'fallback'
```

## Reference(s)

- Jest [getType](https://github.com/jestjs/jest/blob/main/packages/jest-get-type/src/index.ts)
- Lodash [_.get](https://lodash.com/docs/4.18.1#get)

## Solution(s)

```ts
const isPrimitive = (
  value: unknown,
): value is null | undefined | string | number | boolean | symbol | bigint => {
  return Object(value) !== value;
};

const getPathAsArray = (
  pathParam: string | Array<string | number>,
): Array<string> => {
  if (typeof pathParam === "number") {
    return [String(pathParam)];
  }

  if (typeof pathParam === "string") {
    return pathParam.split(".");
  }

  if (Array.isArray(pathParam)) {
    return pathParam.map((elem) => String(elem));
  }

  return [];
};

export default function get<T>(
  objectParam: Record<string, any>,
  pathParam: string | Array<string | number>,
  defaultValue?: T,
): T | undefined {
  const defaultReturnValue: T | undefined = defaultValue ?? undefined;
  const pathArray: Array<string> = getPathAsArray(pathParam);

  if (pathArray.length === 0) {
    return defaultReturnValue;
  }

  let currentObject = objectParam;
  let i = 0;
  while (i < pathArray.length && !isPrimitive(currentObject)) {
    const key: string = pathArray[i];
    const value = currentObject[key];

    currentObject = value;
    i++;
  }

  // if intermediate value is primitive
  if (i < pathArray.length) return defaultReturnValue;

  // if last resolved value is undefined
  if (currentObject === undefined) return defaultReturnValue;

  // else return resolved value
  return currentObject as T;
}
```
