# HTML Sanitizer

The HTML Sanitizer API lets browsers turn untrusted HTML strings into DOM content. This exercise implements a subset of HTML sanitization by parsing and transforming a DOM tree.

Implement `sanitizeHTML(input)`, a simplified sanitizer inspired by the platform's safe HTML sanitization behavior.

Parse the HTML into a detached DOM tree, sanitize it, and return the resulting HTML string.

The sanitizer should:

- Remove these element subtrees entirely: `script`, `iframe`, `object`, `embed`.
- Remove all HTML comment nodes.
- Remove any attribute whose name starts with `on`.
- Remove `href` and `src` attributes whose trimmed, case-insensitive value starts with `javascript:`.
- Preserve all other parsed HTML.

## Arguments

`sanitizeHTML(input)`

| Argument | Type | Description |
| --- | --- | --- |
| `input` | `string` | The HTML string to sanitize. |

## Returns

Returns a sanitized HTML string.

## Examples

```ts
sanitizeHTML('<p>Hello <strong>world</strong></p>');
// '<p>Hello <strong>world</strong></p>'
```

```ts
sanitizeHTML(`
  <div>
    <!-- secret -->
    <a href=" javascript:alert(1) " onclick="evil()">Click me</a>
    <script>alert(1)</script>
  </div>
`);
// '<div><a>Click me</a></div>'
```

## References

- Web API [HTMLTemplateElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTemplateElement)
- Web API [HTML Sanitizer API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Sanitizer_API)
- Concept [sanitizer](https://web.dev/articles/sanitizer)

## Solutions

```ts
const REMOVED_TAGS = new Set(['script', 'iframe', 'object', 'embed']);
const URL_ATTRIBUTES = new Set(['href', 'src']);

function isDangerousUrl(value: string): boolean {
  return value.trim().toLowerCase().startsWith('javascript:');
}

function stripDangerousAttributes(element: Element): void {
  for (const attribute of Array.from(element.attributes)) {
    const attributeName = attribute.name.toLowerCase();

    if (attributeName.startsWith('on')) {
      element.removeAttribute(attribute.name);
      continue;
    }

    if (URL_ATTRIBUTES.has(attributeName) && isDangerousUrl(attribute.value)) {
      element.removeAttribute(attribute.name);
    }
  }
}

function sanitizeNode(parent: Node): void {
  let currentNode = parent.firstChild;

  while (currentNode != null) {
    // Snapshot the next sibling before mutating the current node so traversal
    // still advances correctly after removals.
    const nextSibling = currentNode.nextSibling;

    if (currentNode.nodeType === Node.COMMENT_NODE) {
      currentNode.remove();
      currentNode = nextSibling;
      continue;
    }

    if (currentNode.nodeType === Node.ELEMENT_NODE) {
      const element = currentNode as Element;
      const tagName = element.tagName.toLowerCase();

      if (REMOVED_TAGS.has(tagName)) {
        currentNode.remove();
        currentNode = nextSibling;
        continue;
      }

      stripDangerousAttributes(element);
      sanitizeNode(element);
    }

    currentNode = nextSibling;
  }
}

export default function sanitizeHTML(input: string): string {
  const template = document.createElement('template');
  template.innerHTML = input;

  sanitizeNode(template.content);

  return template.innerHTML;
}
```
