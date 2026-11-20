# Template Engine

Templating engines turn a string like `Hello {{name}}!` into rendered output by replacing placeholders with data.

In this question, implement `renderTemplate(template, data)`, a simplified Mustache-like renderer.

This question is intentionally small:

- Only simple variable tags such as `{{name}}` and `{{ name }}` need to be supported.
- `data` is a flat object of primitive values.
- Missing keys should render as an empty string.
- Inputs are guaranteed valid.

## Example(s)

```ts
renderTemplate('Hello {{name}}!', { name: 'Alice' });
// 'Hello Alice!'

renderTemplate('{{greeting}}, {{name}}!', {
  greeting: 'Hi',
  name: 'Sam',
});
// 'Hi, Sam!'
```

Whitespace inside tags should be ignored.

```ts
renderTemplate('{{ count }} items left', { count: 3 });
// '3 items left'
```

Missing keys should render as empty strings.

```ts
renderTemplate('Hello {{name}} {{surname}}!', { name: 'Alice' });
// 'Hello Alice !'
```

## Notes

- `false`, `0`, and `''` are valid values and should still render.
- Resolved `null` and `undefined` values should render as an empty string.
- You do not need to support nested paths, sections, or malformed templates in this question.

## Reference(s)

- String [replace](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)

## Solution(s)

```ts
export type TemplateValue = null | undefined | boolean | number | string;
export type TemplateData = Record<string, TemplateValue>;

export default function renderTemplate(
  template: string,
  data: TemplateData,
): string {
  return template.replace(/\{\{(.*?)\}\}/g, (match, key: string) => {
    if (!Object.hasOwn(data, key)) {
      return "";
    }

    const value = data[key.trim()];
    return value == null ? "" : String(value);
  });
}
```
