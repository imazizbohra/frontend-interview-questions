# Text Search

Browsers can find specific words or phrases within a webpage by using `Ctrl + F` (Windows, Linux) or `⌘ + F` (Mac) and entering the search term. Matches that appear are highlighted in yellow.

Implement a simple version of browser in-page search that receives a string (as opposed to HTML) and `bolds` search matches.

Given a content string and a query string, implement a function `textSearch` that finds all case-insensitive matches with the query string, wrapping the matches in `<b>...</b>` tags.

## Examples

```ts
textSearch('The Quick Brown Fox Jumps Over The Lazy Dog', 'fox');
// 'The Quick Brown <b>Fox</b> Jumps Over The Lazy Dog'
textSearch('The hardworking Dog overtakes the lazy dog', 'dog');
// 'The hardworking <b>Dog</b> overtakes the lazy <b>dog</b>'
```

A character will not match the same query more than once, with letters appearing earlier taking priority.

```ts
textSearch('aaa', 'aa');
// '<b>aa</b>a'
// This is because the second character cannot be used as a match again.
```

Consecutive matches should be combined into a single `<b>` tag.

```ts
textSearch('aaaa', 'aa');
// Correct: '<b>aaaa</b>'
// Wrong: '<b>aa</b><b>aa</b>'
```

## References

- Javascript [RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)

## Solutions

```ts
// Approach 1

export default function textSearch(text: string, query: string): string {
  if (text.trim() === '' || query.trim() === '') {
    return text;
  }

  const boldChars = Array.from({ length: text.length }, () => 0);

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

```ts
// Approach 2

export default function textSearch(text: string, query: string): string {
  if (text.length === 0 || query.length === 0) {
    return text;
  }

  const regex = new RegExp(query, "gi");
  const indexes = text
    .matchAll(regex)
    .map((match) => [match.index, match.index + query.length - 1])
    .reduce((result, [start, end]) => {
      // compaction
      const last = result[result.length - 1];

      if (last && start === last[1] + 1) {
        last[1] = end;
      } else {
        result.push([start, end]);
      }

      return result;
    }, [] as number[][]);

  // process all the segments
  let lastProcessedIdx = 0;
  let result = "";
  indexes.forEach(([startIdx, endIdx]) => {
    const first = text.slice(lastProcessedIdx, startIdx);
    const second = text.slice(startIdx, endIdx + 1);
    lastProcessedIdx = endIdx + 1;

    result += first.concat("<b>").concat(second).concat("</b>");
  });

  // concat remaining unprocessed string to result
  result += text.slice(lastProcessedIdx);

  return result;
}
```
