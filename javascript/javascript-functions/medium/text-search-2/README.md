# Text Search II

Browsers can find specific words or phrases within a webpage by using `Ctrl + F` (Windows, Linux) or `⌘ + F` (Mac) and entering the search term. Matches that appear are highlighted in yellow.

Implement a simple version of browser in-page search that receives a string (as opposed to HTML) and `bolds` search matches.

Given a string and an array of queries, implement a function `textSearch` that finds all case-insensitive matches from the `queries` array within the string, wrapping the matches in `<b>...</b>` tags.

## Examples

```ts
textSearch('The Quick Brown Fox Jumps Over The Lazy Dog', ['fox']);
// 'The Quick Brown <b>Fox</b> Jumps Over The Lazy Dog'
textSearch('The quick brown fox jumps over the lazy dog', ['fox', 'dog']);
// 'The quick brown <b>fox</b> jumps over the lazy <b>dog</b>'
```

If two such queries overlap or are consecutive, they should be wrapped in a single pair of <b> tags.

```ts
textSearch('This is Uncopyrightable!', ['copy', 'right']);
// 'This is Un<b>copyright</b>able!'
textSearch('This is Uncopyrightable!', ['copy', 'right', 'table']);
// 'This is Un<b>copyrightable</b>!'
```

A character will not match the same query more than once, with earlier letters taking priority.

```ts
textSearch('aaa', ['aa']);
// '<b>aa</b>a'
// This is because the second character cannot be used as a match again.
textSearch('aaaa', ['aa']);
// '<b>aaaa</b>'
```

## References

- Javascript [RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)

## Solutions

```ts
export default function textSearch(
  text: string,
  queries: Array<string>,
): string {
  if (text.trim() === '') {
    return text;
  }

  const boldChars = Array.from({ length: text.length }, () => 0);

  for (const query of queries) {
    if (query.trim() === '') continue;
    for (let i = 0; i < text.length; ) {
      const substr = text.slice(i, i + query.length);
      if (substr.toLowerCase() === query.toLowerCase()) {
        boldChars.fill(1, i, i + query.length);
        // Start from next character if there's a match since one
        // character cannot match the same query more than once.
        i = i + query.length;
      } else {
        i++;
      }
    }
  }

  let highlightedString = '';
  for (let i = 0; i < text.length; i++) {
    // When the current character should be bolded
    // and the previous character should not be bolded,
    // append an opening tag to the final string.
    const shouldAddOpeningTag = boldChars[i] === 1 && boldChars[i - 1] !== 1;
    // When the current character should be bolded
    // and the next character should not be bolded,
    // append a closing tag to the final string.
    const shouldAddClosingTag = boldChars[i] === 1 && boldChars[i + 1] !== 1;
    let char = text[i];

    if (shouldAddOpeningTag) {
      char = '<b>' + char;
    }

    if (shouldAddClosingTag) {
      char = char + '</b>';
    }
    highlightedString += char;
  }

  return highlightedString;
}
```
