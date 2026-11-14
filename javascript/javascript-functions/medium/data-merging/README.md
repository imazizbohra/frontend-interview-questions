# Data Merging

A data set of gym sessions looks like this:

```ts
[
  { user: 8, duration: 50, equipment: ['bench'] },
  { user: 7, duration: 150, equipment: ['dumbbell'] },
  { user: 1, duration: 10, equipment: ['barbell'] },
  { user: 7, duration: 100, equipment: ['bike', 'kettlebell'] },
  { user: 7, duration: 200, equipment: ['bike'] },
  { user: 2, duration: 200, equipment: ['treadmill'] },
  { user: 2, duration: 200, equipment: ['bike'] },
];
```

Each session has the following fields:

- `user`: User ID of the session's user.
- `duration`: Duration of the session, in minutes.
- `equipment`: Array of equipment used during the session, in alphabetical order. There are only five different types of equipment.

Implement `mergeData`, a function that returns a unified view of each user's activities by merging each user's sessions. It has the interface `mergeData(sessions)`. Sessions from the same `user` should be merged into one object. When merging:

- Sum the `duration` fields.
- Combine all the `equipment` used, de-duplicating the values and sorting alphabetically.

The order of the results should always remain unchanged from the original set, and in the case of merging sessions with the same user, the row should take the place of the earliest occurrence of that `user`. The input objects should not be modified.

## Example(s)

The following example uses the data set above:

```ts
mergeData(sessions);
// [
//   { user: 8, duration: 50, equipment: ['bench'] },
//   { user: 7, duration: 450, equipment: ['bike', 'dumbbell', 'kettlebell'] },
//   { user: 1, duration: 10, equipment: ['barbell'] },
//   { user: 2, duration: 400, equipment: ['bike', 'treadmill'] },
// ];
```

```ts
function multiplyThree(a, b, c) {
  return a * b * c;
}
const curriedMultiplyThree = curry(multiplyThree);
curriedMultiplyThree(4)(5)(6); // 120
curriedMultiplyThree(4)(5, 6); // 120
curriedMultiplyThree(4, 5)(6); // 120
curriedMultiplyThree(4, 5, 6); // 120

const containsFour = curriedMultiplyThree(4);
const containsFourMulFive = containsFour(5);
containsFourMulFive(6); // 120
```

## Reference(s)

- JavaScript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- JavaScript [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

## Solution(s)

```ts
// Approach 1

type Session = { user: number; duration: number; equipment: Array<string> };

export default function mergeData(sessions: Array<Session>): Array<Session> {
  const map = new Map<number, Session>();

  for (const session of sessions) {
    const { user, duration, equipment } = session;

    if (!map.has(user)) {
      map.set(user, session);
      continue;
    }

    const existingSession = map.get(user);
    const {
      user: _,
      duration: existingDuration,
      equipment: existingEquipment,
    } = existingSession as Session;

    map.set(user, {
      user,
      duration: existingDuration + duration,
      equipment: [...existingEquipment, ...equipment],
    });
  }

  return Array.from(map.values()).map((session) => {
    session.equipment = Array.from(new Set(session.equipment)).sort();
    return session;
  });
}
```

```ts
// Approach 2

type Session = { user: number; duration: number; equipment: Array<string> };

export default function mergeData(sessions: Array<Session>): Array<Session> {
  const results: Array<{
    user: number;
    duration: number;
    equipment: Set<string>;
  }> = [];
  // Point each user id at the cloned session already stored in `results`.
  const sessionsForUser = new Map();

  sessions.forEach((session) => {
    if (sessionsForUser.has(session.user)) {
      const userSession = sessionsForUser.get(session.user);
      userSession.duration += session.duration;
      session.equipment.forEach((equipment) => {
        userSession.equipment.add(equipment);
      });
    } else {
      const clonedSession = {
        ...session,
        // Use a Set internally so repeated equipment is deduplicated while merging.
        equipment: new Set(session.equipment),
      };
      sessionsForUser.set(session.user, clonedSession);
      results.push(clonedSession);
    }
  });

  // Convert the internal Set back to the sorted array shape expected by callers.
  return results.map((session) => ({
    ...session,
    equipment: Array.from(session.equipment).sort(),
  }));
}
```
