# TypeScript FSA [![npm version][npm-image]][npm-url] [![Build Status][travis-image]][travis-url]

A simple Action Creator library for TypeScript. Its goal is to provide simple
yet type-safe experience with Flux actions.
Created actions are FSA-compliant:
 
```ts
interface Action<P> {
  type: string;
  payload: P;
  error?: boolean;
  meta?: Object;
}
``` 

## Installation

```
npm install --save typescript-fsa
```

## Usage

### Basic

```ts
import actionCreatorFactory from 'typescript-fsa';

const actionCreator = actionCreatorFactory();

// Specify payload shape as generic type argument. 
const somethingHappened = actionCreator<{foo: string}>('SOMETHING_HAPPENED');

// Get action creator type.
console.log(somethingHappened.type);  // SOMETHING_HAPPENED

// Create action.
const action = somethingHappened({foo: 'bar'});
console.log(action);  // {type: 'SOMETHING_HAPPENED', payload: {foo: 'bar'}}  
```

### Async Action Creators

Async Action Creators are objects with properties `started`, `done` and 
`failed` whose values are action creators. 

```ts
import actionCreatorFactory from 'typescript-fsa';

const actionCreator = actionCreatorFactory();

// specify parameters and result shapes as generic type arguments
const doSomething = 
  actionCreator.async<{foo: string},   // parameter type
                      {bar: number},   // success type
                      {code: number}   // error type
                     >('DO_SOMETHING');

console.log(doSomething.started({foo: 'lol'}));
// {type: 'DO_SOMETHING_STARTED', payload: {foo: 'lol'}}

console.log(doSomething.done({
  params: {foo: 'lol'},
  result: {bar: 42},
});
// {type: 'DO_SOMETHING_DONE', payload: {
//   params: {foo: 'lol'},
//   result: {bar: 42},
// }}

console.log(doSomething.failed({
  params: {foo: 'lol'},
  error: {code: 42},    
});
// {type: 'DO_SOMETHING_FAILED', payload: {
//   params: {foo: 'lol'},
//   error: {code: 42},
// }, error: true}
```
  
### Actions With Type Prefix

You can specify a prefix that will be prepended to all action types. This is 
useful to namespace library actions as well as for large projects where it's 
convenient to keep actions near the component that dispatches them. 

```ts
// MyComponent.actions.ts
import actionCreatorFactory from 'typescript-fsa';

const actionCreator = actionCreatorFactory('MyComponent');

const somethingHappened = actionCreator<{foo: string}>('SOMETHING_HAPPENED');

const action = somethingHappened({foo: 'bar'});
console.log(action);  
// {type: 'MyComponent/SOMETHING_HAPPENED', payload: {foo: 'bar'}}  
```

### Redux

```ts
// actions.ts
import actionCreatorFactory from 'typescript-fsa';

const actionCreator = actionCreatorFactory();

export const somethingHappened = 
  actionCreator<{foo: string}>('SOMETHING_HAPPENED');


// reducer.ts
import {Action} from 'redux';
import {isType} from 'typescript-fsa';
import {somethingHappened} from './actions';

type State = {bar: string};

const reducer = (state: State, action: Action): State => {
  if (isType(action, somethingHappened)) {
    // action.payload is inferred as {foo: string};
    
    action.payload.bar;  // error
    
    return {bar: action.payload.foo};
  }

  return state; 
};
```

## Companion packages

* [typescript-fsa-redux-saga](https://github.com/aikoven/typescript-fsa-redux-saga)
* [typescript-fsa-reducers](https://github.com/dphilipson/typescript-fsa-reducers)

## Resources

* [Type-safe Flux Standard Actions (FSA) in React Using TypeScript FSA](https://www.triplet.fi/blog/type-safe-flux-standard-actions-fsa-in-react-using-typescript-fsa/)

## API

### `actionCreatorFactory(prefix?: string, defaultIsError?: Predicate): ActionCreatorFactory`

Creates Action Creator factory with optional prefix for action types.

* `prefix?: string`: Prefix to be prepended to action types.
* `defaultIsError?: Predicate`: Function that detects whether action is error
 given the payload. Default is `payload => payload instanceof Error`.

### `isType(action: Action, actionCreator: ActionCreator): boolean`

Returns `true` if action has the same type as action creator. Defines 
[Type Guard](https://www.typescriptlang.org/docs/handbook/advanced-types.html#user-defined-type-guards)
that lets TypeScript know `payload` type inside blocks where `isType` returned
`true`:

```ts
const somethingHappened = actionCreator<{foo: string}>('SOMETHING_HAPPENED');

if (isType(action, somethingHappened)) {
  // action.payload has type {foo: string};
}
```

[npm-image]: https://badge.fury.io/js/typescript-fsa.svg
[npm-url]: https://badge.fury.io/js/typescript-fsa
[travis-image]: https://travis-ci.org/aikoven/typescript-fsa.svg?branch=master
[travis-url]: https://travis-ci.org/aikoven/typescript-fsa
