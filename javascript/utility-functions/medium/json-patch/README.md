# JSON Patch

**Note**: Similar to [unsquash object](../unsquash-object/) and [Object Update](../object-update/).

JSON Patch is a format for describing updates to a JSON document as a list of operations.

For a quick overview, see [jsonpatch.com](jsonpatch.com).

It is often used when systems want to send incremental updates instead of resending a full object every time. For example, a server can stream small patches over SSE or WebSockets to update a client-side document, chat state, or tool result in place. This pattern also shows up in streaming AI chat UIs, where the server may send structured deltas as new content arrives.

In this question, implement `applyJsonPatch(document, operations)` which applies a list of patch operations to a nested object and returns the patched result.

This question is intentionally simplified:

- Only `add`, `remove`, and `replace` operations need to be supported.
- The `document` only contains plain objects and JSON primitive values. Arrays are out of scope.
- Every `path` uses `/`-delimited object keys such as `/user/name`.
- Inputs are guaranteed valid.

## Arguments

`applyJsonPatch(document, operations)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `document` | `Object` | A nested object containing JSON primitive values or other plain objects. |
| `operations` | `Array` | A list of patch operations to apply in order. |

Each operation has one of the following shapes:

```ts
{ op: 'add', path: string, value: unknown }
{ op: 'remove', path: string }
{ op: 'replace', path: string, value: unknown }
```

## Returns

Returns a new object with all operations applied in order.

## Examples

```ts
applyJsonPatch(
  {
    user: {
      name: 'Alice',
      role: 'admin',
    },
  },
  [
    { op: 'replace', path: '/user/name', value: 'Bob' },
    { op: 'add', path: '/user/active', value: true },
  ],
);
// {
//   user: {
//     name: 'Bob',
//     role: 'admin',
//     active: true,
//   },
// }
```

```ts
applyJsonPatch(
  {
    settings: {
      theme: 'dark',
      locale: 'en-US',
    },
  },
  [{ op: 'remove', path: '/settings/locale' }],
);
// {
//   settings: {
//     theme: 'dark',
//   },
// }
```

## Notes

- `add` creates or overwrites the final property at `path`.
- `remove` deletes the final property at `path`.
- `replace` overwrites the existing property at `path`.
- Paths in this question are always non-empty and always point to properties inside the object.
- Do not mutate `document` or `operations`.
- The returned object should not share nested object references with `document` or with values supplied by `operations`.

## References

- [jsonpatch.com](https://jsonpatch.com/)

## Solutions

```ts
type JSONPrimitive = null | boolean | number | string;
type JSONValue = JSONPrimitive | JSONObject;
type JSONObject = { [key: string]: JSONValue };

type AddOperation = {
  op: "add";
  path: string;
  value: JSONValue;
};

type RemoveOperation = {
  op: "remove";
  path: string;
};

type ReplaceOperation = {
  op: "replace";
  path: string;
  value: JSONValue;
};

type PatchOperation = AddOperation | RemoveOperation | ReplaceOperation;

function performOperation(document: JSONObject, operation: PatchOperation) {
  let current = document;

  const pathArray = operation.path.slice(1).split("/");

  for (let i = 0; i < pathArray.length - 1; i++) {
    const key = pathArray[i];

    current = current[key] as JSONObject;
  }

  const lastPath = pathArray[pathArray.length - 1];

  if (operation.op === "add" || operation.op === "replace") {
    current[lastPath] = structuredClone(operation.value);
  } else if (operation.op === "remove") {
    delete current[lastPath];
  }
}

export default function applyJsonPatch(
  document: JSONObject,
  operations: Array<PatchOperation>,
): JSONObject {
  const obj = structuredClone(document);

  if (operations.length === 0) {
    return obj;
  }

  for (const operation of operations) {
    performOperation(obj, operation);
  }

  return obj;
}
```
