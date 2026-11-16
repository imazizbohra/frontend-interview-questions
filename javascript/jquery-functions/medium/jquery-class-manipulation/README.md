# jQuery Class Manipulation

Note: If you haven't completed the [jQuery.css](../jquery-class-manipulation/) question, you should attempt that first.

Before `Element.classList` and `DOMTokenList` were part of the browser standards, it was a hassle to manipulate classes on a DOM element.

`jQuery` is a library that simplifies DOM manipulation (among other things). jQuery (using the `$` alias function) provided convenient APIs to `toggle`, `add`, and `remove` classes from elements via `.toggleClass()`, `.addClass()`, and `.removeClass()`.

## Example(s)

```ts
// <button class="foo bar">Click me</button>
$('button').toggleClass('bar'); // <button class="foo">Click me</button>
$('button').addClass('baz'); // <button class="foo baz">Click me</button>
$('button').removeClass('foo'); // <button class="baz">Click me</button>
$('button').toggleClass('bar'); // <button class="baz bar">Click me</button>
```

The return value of most jQuery manipulation APIs is the jQuery object itself, so that method calls can be chained. The above can be simplified further:

```ts
// <button class="foo bar">Click me</button>
$('button')
  .toggleClass('bar')
  .addClass('baz')
  .removeClass('foo')
  .toggleClass('bar');
// <button class="baz bar">Click me</button>
```

Implement the `toggleClass()`, `addClass()`, and `removeClass()` methods according to the following specifications. Do not use `Element.classList` or `DOMTokenList` methods; otherwise the problem becomes quite trivial.

Class strings may contain leading, trailing, or repeated whitespace. Treat them as whitespace-separated class tokens, and do not create duplicate classes.

## Reference(s)

- Element [className](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)

## Solution(s)

```ts
interface JQuery {
  toggleClass: (className: string, state?: boolean) => JQuery;
  addClass: (className: string) => JQuery;
  removeClass: (className: string) => JQuery;
}

export default function $(selector: string): JQuery {
  const element = document.querySelector(selector);

  return {
    toggleClass: function (className: string, state?: boolean): JQuery {
      if (!element) return this;

      const existingTokens = new Set(
        element.className.trim().replace(/\s+/g, " ").split(" "),
      );
      const newTokens = className.trim().replace(/\s+/g, " ").split(" ");
      newTokens.forEach((token) => {
        if (state == null) {
          if (existingTokens.has(token)) {
            existingTokens.delete(token);
          } else {
            existingTokens.add(token);
          }
        } else if (state == true) {
          existingTokens.add(token);
        } else {
          existingTokens.delete(token);
        }
      });
      element.className = Array.from(existingTokens).join(" ");

      return this;
    },
    addClass: function (className: string): JQuery {
      if (!element) return this;

      this.toggleClass(className, true);

      return this;
    },
    removeClass: function (className): JQuery {
      if (!element) return this;

      this.toggleClass(className, false);

      return this;
    },
  };
}
```
