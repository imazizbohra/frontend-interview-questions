# Template Engine II

This is a follow-up to [Template Engine](../template-engine/).

Implement the same `renderTemplate(template, data)` API, but now with behavior that is closer to a real template renderer:

- Tags can use dotted paths such as `{{user.name}}`.
- Normal `{{...}}` tags should HTML-escape their output.
- Triple-brace tags like `{{{html}}}` should render raw output.
- Missing paths should still render as an empty string.

This question still keeps the engine intentionally small:

- data only contains plain objects and primitive values.
- Arrays are out of scope.
- Inputs are guaranteed valid.

## Example(s)

```ts
renderTemplate('Hello {{user.name}}!', {
  user: { name: 'Alice' },
});
// 'Hello Alice!'
```

Normal tags should escape HTML.

```ts
renderTemplate('{{content}}', {
  content: '<strong>Hello</strong>',
});
// '&lt;strong&gt;Hello&lt;/strong&gt;'
```

Triple-brace tags should render raw output.

```ts
renderTemplate('{{{content}}}', {
  content: '<strong>Hello</strong>',
});
// '<strong>Hello</strong>'
```

## Notes

- Missing paths and resolved `null` / `undefined` values should render as an empty string.
- A path that resolves to an object should also render as an empty string.
- `false`, `0`, and `''` are valid resolved values and should still render.
- You do not need to support sections, loops, or malformed templates in this question.

## Reference(s)

- String [replace](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)

## Solution(s)

```ts
export type TemplatePrimitive = null | undefined | boolean | number | string;
export type TemplateValue = TemplatePrimitive | TemplateObject;
export type TemplateObject = { [key: string]: TemplateValue };

const TOKEN_REGEX = /{{{\s*([^{}]+?)\s*}}}|{{\s*([^{}]+?)\s*}}/g;
const HTML_ESCAPE_MAP: Record<string, string> = {
  "&": "&amp;",
  "<": "&lt;",
  ">": "&gt;",
  '"': "&quot;",
  "'": "&#39;",
};

function resolvePath(
  path: string,
  obj: TemplateObject,
): TemplateValue | undefined {
  if (path.length === 0) {
    return "";
  }

  let current: any = obj;
  const pathArr = path.trim().replace(/\s+/g, " ").split(".").filter(Boolean);

  while (current != null && pathArr.length > 0) {
    const key = pathArr.shift()!;

    current = current[key];
  }

  return current;
}

function escapeHtml(value: string): string {
  return value.replace(/[&<>"']/g, (character) => HTML_ESCAPE_MAP[character]);
}

function formatValue(value: TemplateValue | undefined): string {
  if (value == null || typeof value === "object") {
    return "";
  }

  return String(value);
}

export default function renderTemplate(
  template: string,
  data: TemplateObject,
): string {
  return template.replace(
    TOKEN_REGEX,
    (_, rawPath: string | undefined, escapedPath: string | undefined) => {
      // Triple braces land in `rawPath`; double braces land in `escapedPath`.
      const path = (rawPath ?? escapedPath ?? "").trim();
      const renderedValue = formatValue(resolvePath(path, data));

      return rawPath != null ? renderedValue : escapeHtml(renderedValue);
    },
  );
}
```
