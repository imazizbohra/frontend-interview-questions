# HTML Serializer

Given an object that resembles a DOM tree, implement a function that serializes the object into a formatted string with proper indentation (one tab (`\t` character) per nesting level) and one tag per line.

## Example(s)

```ts
const tree = {
  tag: 'body',
  children: [
    { tag: 'div', children: [{ tag: 'span', children: ['foo', 'bar'] }] },
    { tag: 'div', children: ['baz'] },
  ],
};

serializeHTML(tree);
// Output:
`<body>
  <div>
    <span>
      foo
      bar
    </span>
  </div>
  <div>
    baz
  </div>
</body>`;
```

## Reference(s)

- Glossary [Recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion)

## Solution(s)

```ts
type HTMLNode = string | HTMLElementNode;

type HTMLElementNode = {
  tag: string;
  children: HTMLNode[];
};

export default function serializeHTML(root: HTMLElementNode): string {
  function serialize(node: HTMLNode, level: number): string {
    const indentation = "\t".repeat(level);

    if (typeof node === "string") {
      return "".concat(indentation).concat(node);
    }

    const element = node as HTMLElementNode;
    const tag = element.tag.toLowerCase();

    const sChildren = [];
    for (let child of element.children) {
      const sChild = serialize(child, level + 1);
      sChildren.push(sChild);
    }

    return ""
      .concat(indentation)
      .concat(`<${tag}>`)
      .concat("\n")
      .concat(sChildren.join("\n"))
      .concat("\n")
      .concat(indentation)
      .concat(`</${tag}>`);
  }

  return serialize(root, 0);
}
```
