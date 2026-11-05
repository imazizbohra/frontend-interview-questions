# Reading Order

Canvas-based tools often store elements in creation order or some other arbitrary internal order. However, features like keyboard navigation, batch actions on a selection, or exporting a canvas into a human-friendly outline usually need a deterministic order that matches how a person scans the screen.

On a 2D canvas, that usually means reading from top to bottom and from left to right within each row.

In this question, implement readingOrder(elements) for flat rectangle elements.

Each element has this shape:

```json
{
  id: string,
  x: number,
  y: number,
  width: number,
  height: number,
}
```

For this question:

- Elements in the same row always have the same y.
- Different rows do not overlap vertically.

That means the reading order is determined by sorting elements by y first, then by x.

## Example(s)

```ts
readingOrder([
  { id: 'd', x: 50, y: 40, width: 40, height: 20 },
  { id: 'b', x: 60, y: 0, width: 40, height: 20 },
  { id: 'c', x: 10, y: 40, width: 30, height: 20 },
  { id: 'a', x: 0, y: 0, width: 40, height: 20 },
]);
// ['a', 'b', 'c', 'd']
```

```ts
y = 0   +-----a-----+  +-----b-----+

y = 40     +---c---+  +----d----+

reading order: a -> b -> c -> d
```

```ts
readingOrder([
  { id: 'hero', x: 0, y: 0, width: 200, height: 80 },
  { id: 'cta', x: 220, y: 0, width: 80, height: 80 },
  { id: 'footer', x: 0, y: 120, width: 300, height: 40 },
]);
// ['hero', 'cta', 'footer']
```

```ts
y = 0   +------------hero------------+  +--cta--+

y = 120 +------------------footer------------------+

reading order: hero -> cta -> footer
```

## Solution(s)

```ts
export interface CanvasElement {
  id: string;
  x: number;
  y: number;
  width: number;
  height: number;
}

export default function readingOrder(
  elements: Array<CanvasElement>,
): Array<string> {
  return elements
    .slice()
    .sort((a, b) => {
      if (a.y !== b.y) {
        return a.y - b.y;
      }

      return a.x - b.x;
    })
    .map((element) => element.id);
}
```
