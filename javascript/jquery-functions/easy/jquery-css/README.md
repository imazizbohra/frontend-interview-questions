# jQuery.css

In vanilla JavaScript, styles can be set on an element as follows:

```js
const buttonEl = document.querySelector('button');
buttonEl.style.color = 'red';
buttonEl.style.backgroundColor = 'tomato';
buttonEl.style.fontSize = '16px';
```

`jQuery` is a library that simplifies DOM manipulation (among other things). With jQuery (using the $ alias function), the above can be simplified into:

```js
const buttonEl = $('button');
buttonEl.css('color', 'red');
buttonEl.css('backgroundColor', 'tomato');
buttonEl.css('fontSize', '16px');
```

The return value of most jQuery manipulation APIs is the jQuery object itself, so that method calls can be chained. The above can be further simplified again:

```js
$('button')
  .css('color', 'red')
  .css('backgroundColor', 'tomato')
  .css('fontSize', '16px');
```

Additionally, if the second parameter is omitted, the value of that style property is returned.

```js
// <button style="color: red">Submit</button>
$('button').css('color'); // 'red'
```

## Notes

There are some differences from the official jQuery.css() implementation:

- The official library selects all matched elements and modifies all matched elements. However, for this question, assume there will be at most one element matching the selector.
- jQuery understands both .css('background-color') and .css('backgroundColor') and returns the correct value, but only the latter syntax has to be supported.
- Getting an inline style property that has not been set should return undefined.
- If no element matches the selector, getters should return undefined and setters should remain chainable no-ops.

## Reference(s)

- jQuery [css](https://api.jquery.com/css/)
- Javascript [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)

## Techniques

- Closures
- Object chaining

## Solution(s)

```ts
interface JQuery {
  css: (
    prop: string,
    value?: boolean | string | number,
  ) => JQuery | string | undefined;
}

export default function $(selector: string): JQuery {
  const element = document.querySelector(selector) as HTMLElement | null;

  const obj: JQuery = {
    css: function (
      prop: string,
      value?: boolean | string | number,
    ): JQuery | string | undefined {
      // getter
      if (value == null) {
        if (element == null) return undefined;

        const propValue = element.style[prop as any];
        if (propValue === "") return undefined;

        return propValue;
      }

      // setter
      if (element != null) {
        element.style[prop as any] = String(value);
      }

      return this;
    },
  };

  return obj;
}
```
