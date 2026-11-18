# useMediaQuery

Implement a `useMediaQuery` hook that subscribes to and responds to media query changes (e.g. screen size, resolution, orientation, etc.).

## Argument(s)

`query: string`: The media query to match. It must be a [valid CSS media query string](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Media_queries/Using)

## Returns

The hook returns a `boolean` value that indicates whether the media query matches.

## Notes

- The initial return value should come from `window.matchMedia(query).matches`.
- Subscribe to the current query's `change` event and update state when it changes.
- If `query` changes, unsubscribe from the old `MediaQueryList` and subscribe to the new one.
- You do not need a separate `resize` listener.

## Example(s)

```tsx
export default function Component() {
  const isSmallDevice = useMediaQuery('only screen and (max-width: 768px)');

  return <div>{isSmallDevice && <a href="#">Menu</a>}</div>;
}
```

## Reference(s)

- Web API [matchMedia](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia)
- Web API [MediaQueryList](https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList)
- React [useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)
- React [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## Solution(s)

```ts
// Approach 1

import { useState, useEffect } from "react";

export default function useMediaQuery(query: string): boolean {
  const [isMatched, setIsMatched] = useState<boolean>(
    () => window.matchMedia(query).matches,
  );

  useEffect(() => {
    const mql = window.matchMedia(query);

    const listener = () => {
      setIsMatched(mql.matches);
    };

    mql.addEventListener("change", listener);

    return () => mql.removeEventListener("change", listener);
  }, [query]);

  return isMatched;
}
```

```ts
// Approach 2

import { useCallback, useSyncExternalStore } from 'react';

// useSyncExternalStore connects React components to external data sources and ensures they stay synchronized when that data changes.
export default function useMediaQuery(query: string): boolean {
  // useSyncExternalStore expects a subscribe function that removes the exact listener it adds.
  const subscribe = useCallback(
    (callback: () => void) => {
      const mediaQueryList = window.matchMedia(query);

      mediaQueryList.addEventListener('change', callback);

      return () => {
        mediaQueryList.removeEventListener('change', callback);
      };
    },
    [query],
  );

  // matchMedia is the external store; `.matches` is the current snapshot.
  return useSyncExternalStore(
    subscribe,
    () => window.matchMedia(query).matches,
  );
}
```
