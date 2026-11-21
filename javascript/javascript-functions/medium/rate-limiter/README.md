# Rate Limiter

Rate limiters decide whether each incoming task should be accepted based on how many recent accepted tasks are still inside a fixed time window.

In this question, implement `rateLimit(timestamps, maxAllowed, windowLen)` for a simplified batch setting. The input is a sorted list of task timestamps, and you should return a boolean array where each entry says whether that task is accepted.

This question is intentionally interview-scoped:

- The timestamps are already sorted in non-decreasing order.
- Only previously accepted tasks count against the limit.
- Rejected tasks do not consume slots in the window.
- You do not need real-time behavior, retries, token buckets, or per-user limits.

## Returns

Returns a `boolean[]` of the same length as timestamps.

Each result is:

`true` if the task is accepted.
`false` if the task is rejected because the sliding window already contains `maxAllowed` previously accepted tasks.

## Examples

```ts
rateLimit([1, 3, 5, 6, 8, 9, 10], 2, 5);
// [true, true, false, true, true, false, false]

rateLimit([1, 6, 7], 1, 5);
// [true, true, false]
```

In the second example, the task at time `6` is accepted because the accepted task at time `1` is exactly `5` units earlier, so it is outside the window.

```ts
rateLimit([2, 2, 2], 2, 3);
// [true, true, false]
```

## Notes

- A task at time `t` should count previously accepted tasks whose timestamps satisfy `t - previous < windowLen`.
- A task exactly `windowLen` units earlier does not count.
- Inputs are guaranteed valid.
- Do not mutate the input array.

## Solutions

```ts
export default function rateLimit(
  timestamps: number[],
  maxAllowed: number,
  windowLen: number,
): boolean[] {
  const acceptedTimestamps: number[] = [];
  const result: boolean[] = [];
  let head = 0;

  for (const timestamp of timestamps) {
    // Move the head past accepted timestamps that are no longer inside the window.
    while (
      head < acceptedTimestamps.length &&
      timestamp - acceptedTimestamps[head] >= windowLen
    ) {
      head += 1;
    }

    const isAccepted = acceptedTimestamps.length - head < maxAllowed;
    result.push(isAccepted);

    // Only accepted tasks consume capacity in future windows.
    if (isAccepted) {
      acceptedTimestamps.push(timestamp);
    }
  }

  return result;
}
```
