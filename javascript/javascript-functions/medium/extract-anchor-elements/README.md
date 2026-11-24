# Extract Anchor Elements

Browsers parse raw HTML text into DOM structures. An HTML scanner must walk the string and identify tag boundaries while treating characters such as `>` inside quoted attributes as data.

In this interview-sized version, implement `extractAnchors(html)`, which returns every complete anchor element from an HTML string in source order.

Each returned item should be the exact substring from the original input, including attributes, whitespace, and nested non-anchor markup inside the anchor.

## Arguments

`extractAnchors(html)`

- html (string): A valid HTML-like string.

## Returns

Returns an array of anchor element substrings in source order.

## Examples

```ts
extractAnchors(
  '<div> <a   href="/home" >Home</a   > <span>Later</span> <a >Docs</a> </div>',
);
// ['<a   href="/home" >Home</a   >', '<a >Docs</a>']
```

```ts
extractAnchors(
  `<section>
    <p>Read <a
      href="https://example.com/docs"
      data-track="docs link"
    >the docs</a> first.</p>
  </section>`,
);
// [
//   `<a
//       href="https://example.com/docs"
//       data-track="docs link"
//     >the docs</a>`
// ]
```

```ts
extractAnchors(
  `<div>
    <a   href="https://example.com/guides"   class="link" >
      <abbr>API</abbr>
      <span>guide</span>
    </a>
  </div>`,
);
// [
//   `<a   href="https://example.com/guides"   class="link" >
//       <abbr>API</abbr>
//       <span>guide</span>
//     </a>`
// ]
```

## Notes

- Inputs are guaranteed valid for this question.
- Only lowercase `<a ...>` and `</a>` tags need to be handled.
- Anchors are never nested inside other anchors.
- Nested non-anchor tags can appear inside an anchor.
- The HTML can contain extra spaces, indentation, and line breaks inside the markup. Preserve them in the returned substrings.
- Preserve the original substring exactly as it appeared in the input. Do not normalize whitespace or rebuild the HTML.
- You do not need to handle malformed markup, uppercase tags, comments, scripts, or entity decoding.

## Off limits

- Do not use `DOMParser`, `innerHTML`, `querySelectorAll()`, `getElementsByTagName()`, or similar DOM helpers.
- Build the result by scanning the string yourself.

## References

- JavaScript [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/String)

## Solutions

```ts
function isWhitespace(char: string | undefined): boolean {
  return char != null && /\s/.test(char);
}

function isAnchorOpenStart(html: string, index: number): boolean {
  if (html[index] !== '<' || html[index + 1] !== 'a') {
    return false;
  }

  const nextChar = html[index + 2];
  return nextChar === '>' || isWhitespace(nextChar);
}

function isAnchorCloseStart(html: string, index: number): boolean {
  if (
    html[index] !== '<' ||
    html[index + 1] !== '/' ||
    html[index + 2] !== 'a'
  ) {
    return false;
  }

  const nextChar = html[index + 3];
  return nextChar === '>' || isWhitespace(nextChar);
}

function findTagEnd(html: string, startIndex: number): number {
  let quote: '"' | "'" | null = null;

  for (let i = startIndex + 1; i < html.length; i++) {
    const char = html[i];

    if (quote !== null) {
      // Ignore `>` characters until the quoted attribute value closes.
      if (char === quote) {
        quote = null;
      }

      continue;
    }

    if (char === '"' || char === "'") {
      quote = char;
      continue;
    }

    if (char === '>') {
      return i;
    }
  }

  return -1;
}

function findClosingAnchorEnd(html: string, startIndex: number): number {
  for (let i = startIndex; i < html.length; i++) {
    if (html[i] !== '<') {
      continue;
    }

    if (isAnchorCloseStart(html, i)) {
      return findTagEnd(html, i);
    }

    // Skip over other tags so nested markup inside the anchor is preserved.
    const tagEnd = findTagEnd(html, i);

    if (tagEnd === -1) {
      return -1;
    }

    i = tagEnd;
  }

  return -1;
}

export default function extractAnchors(html: string): Array<string> {
  const anchors: Array<string> = [];

  for (let i = 0; i < html.length; i++) {
    if (html[i] !== '<') {
      continue;
    }

    if (isAnchorOpenStart(html, i)) {
      const openingTagEnd = findTagEnd(html, i);

      if (openingTagEnd === -1) {
        break;
      }

      const closingTagEnd = findClosingAnchorEnd(html, openingTagEnd + 1);

      if (closingTagEnd === -1) {
        break;
      }

      anchors.push(html.slice(i, closingTagEnd + 1));
      i = closingTagEnd;
      continue;
    }

    const tagEnd = findTagEnd(html, i);

    if (tagEnd === -1) {
      break;
    }

    i = tagEnd;
  }

  return anchors;
}
```
