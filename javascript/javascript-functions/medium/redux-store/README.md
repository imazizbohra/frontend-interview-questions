# Redux Store

`Redux` stores keep application state in one place and update it by passing actions through a reducer.

Implement a simplified Redux store. The `createStore` function only needs to initialize reducer state, expose `getState()`, and update state through `dispatch()`.

Subscriptions are intentionally out of scope for this question and are added in the follow-up question.

## Argument(s)

`createStore(reducer, preloadedState?)`

- `reducer ((state, action) => nextState)`: Receives the current state and an action object, then returns the next state.
- `preloadedState (unknown, optional)`: When provided, this value should be passed to the reducer during initialization.
- The store should initialize itself by calling `reducer()` once with an internal action whose type is `@@redux-store/INIT`.

## Returns

Returns a store object with the following methods:

`store.getState()`
Returns the current state stored by the reducer.

`store.dispatch(action)`
Passes action to the reducer, stores the returned state, and returns the same action object.

## Example(s)

```ts
function counterReducer(state = 0, action) {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    default:
      return state;
  }
}

const store = createStore(counterReducer);

store.getState(); // 0

store.dispatch({ type: 'increment' });
store.getState(); // 1

store.dispatch({ type: 'decrement' });
store.getState(); // 0
```

```ts
function counterReducer(state = 0, action) {
  if (action.type === 'increment') {
    return state + 1;
  }

  return state;
}

const store = createStore(counterReducer, 10);

store.getState(); // 10

store.dispatch({ type: 'increment' });
store.getState(); // 11
```

## Notes

- This is the first step of a Redux-style store. You do not need to implement `subscribe()` in this question.
- State updates should be synchronous.
- You do not need to support `enhancers`, `middleware`, `combineReducers`, `replaceReducer`, action validation, or nested dispatch rules.

## Solution(s)

```ts
export type Reducer<State, Action = { type: string }> = (
  state: State | undefined,
  action: Action,
) => State;

export interface Store<State, Action = { type: string }> {
  getState(): State;
  dispatch(action: Action): Action;
}

export default function createStore<State, Action = { type: string }>(
  reducer: Reducer<State, Action>,
  preloadedState?: State,
): Store<State, Action> {
  let currentState: State = reducer(preloadedState, {
    type: "@@redux-store/INIT",
  } as Action);

  return {
    getState() {
      return currentState;
    },
    dispatch(action: Action) {
      currentState = reducer(currentState, action);
      return action;
    },
  };
}
```
