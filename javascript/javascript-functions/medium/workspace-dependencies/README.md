# Workspace Dependencies

Monorepos often contain many workspace packages that depend on one another. A package's direct dependencies may have their own workspace dependencies, so it can be useful to compute the full dependency list in build order.

Implement `workspaceDependencies(workspace, packageName)` to return the unique workspace-local dependencies for `packageName` in a deterministic topological order.

The `workspace` input is an adjacency map where each key is a workspace package name and each value is an array of that package's direct dependency names.

- Only traverse dependency names that also exist as keys in `workspace`.
- Ignore external packages such as `react` or `lodash`.
- Return each workspace package at most once.
- Do not include `packageName` itself in the result.
- Traverse dependencies depth first: process each dependency array from left to right, finishing each dependency's entire reachable subtree before moving to the next dependency.
- Add each package to the result only after all of its own workspace-local dependencies (post-order).
- If a reachable cycle exists, `throw Error('Cycle detected')`.

## Arguments

`workspaceDependencies(workspace, packageName)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `workspace` | `Record<string, Array<string>>` | An adjacency map of workspace package names to their direct dependency names. |
| `packageName` | `string` | The workspace package whose transitive workspace-local dependencies should be returned. |

## Returns

Returns `Array<string>`, containing the unique workspace-local dependencies of `packageName` in left-to-right depth-first post-order.

## Notes

- `packageName` is guaranteed to exist as a key in `workspace`.
- The input shape is guaranteed valid.
- This question is intentionally limited:
- `workspace` is already parsed for you.
- You do not need to handle package.json files, version ranges, or different dependency types.
- Production-hardening details are out of scope.

## Examples

Basic transitive dependency lookup:

```ts
const workspace = {
  'web-app': ['ui-kit'],
  'ui-kit': ['design-tokens', 'shared-utils'],
  'design-tokens': [],
  'shared-utils': [],
};

workspaceDependencies(workspace, 'web-app');
// ['design-tokens', 'shared-utils', 'ui-kit']
```

Shared dependencies should only appear once:

```ts
const workspace = {
  'web-app': ['ui-kit', 'shared-utils'],
  'ui-kit': ['shared-utils'],
  'shared-utils': [],
};

workspaceDependencies(workspace, 'web-app');
// ['shared-utils', 'ui-kit']
```

External packages should be ignored:

```ts
const workspace = {
  'web-app': ['ui-kit', 'react'],
  'ui-kit': ['shared-utils', 'lodash'],
  'shared-utils': ['date-fns'],
};

workspaceDependencies(workspace, 'web-app');
// ['shared-utils', 'ui-kit']
```

Reachable cycles should throw:

```ts
const workspace = {
  'web-app': ['ui-kit'],
  'ui-kit': ['shared-utils'],
  'shared-utils': ['ui-kit'],
};

workspaceDependencies(workspace, 'web-app');
// throws Error('Cycle detected')
```

## References

- Topological sort [geeksforgeeks.org](https://www.geeksforgeeks.org/dsa/topological-sorting/)

## Solutions

```ts
export default function workspaceDependencies(
  workspace: Record<string, string[]>,
  packageName: string,
): string[] {
  const visiting = new Set<string>();
  const resolved = new Set<string>();

  function traverse(workspace: Record<string, string[]>, packageName: string) {
    if (!workspace[packageName]) {
      return;
    }

    if (visiting.has(packageName)) {
      throw Error("Cycle detected");
    }

    visiting.add(packageName);

    for (const deps of workspace[packageName]) {
      traverse(workspace, deps);
    }

    resolved.add(packageName);
    visiting.delete(packageName);
  }

  for (const deps of workspace[packageName]) {
    traverse(workspace, deps);
  }

  return Array.from(resolved);
}
```
