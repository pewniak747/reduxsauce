<img src='./_art/redux_sauce.png' align=left/>
# Reduxsauce

[![npm module](https://badge.fury.io/js/reduxsauce.svg)](https://www.npmjs.org/package/reduxsauce)

# What's The Story?

Provides a few tools for working with Redux-based codebases.

Currently includes:

1. createReducer - declutter some reducers and make them easier to read & test
1. createTypes - quickly define your types object from a string

More coming...

# createReducer

There's a few patterns I've learned (and was taught), but let's break down the parts of a reducer first:

1. Determining the initial state.
1. Running
1. Knowing when to run.
1. Injecting into the global state tree

#### Initial State

Every reducer I've written has a known and expected state.  And it's always an object.

```js
const INITIAL_STATE = { name: null, age: null }
```

If you're using seamless-immutable, this just get's wrapped. This is optional.

```js
const INITIAL_STATE = Immutable({ name: null, age: null })
```

#### Running

A reducer is a function.  It has 2 inbound parameters and returns the new state.

```js
export const sayHello = (state = INITIAL_STATE, action) => {
  const { age, name } = action
  return { ...state, age, name }
}
```

Notice the `export`?  That's only needed if you would like to write some tests for your reducer.

#### Knowing When To Run

In Redux, all reducers fire in response to *any* action. It's up to the reducer to determine if it should run in response.  This is usually driven by a `switch` on `action.type`.

This works great until you start adding a bunch of code, so, I like to break out "routing" from "running" by registering reducers.

We can use a simple object registry to map action types to our reducer functions.

```js
import Types from './actionTypes'

export const HANDLERS = {
  [Types.SAY_HELLO]: sayHello,
  [Types.SAY_GOODBYE]: sayGoodbye
}
```

The `export` is only needed for testing.  It's optional.


#### Injecting Into The Global State Tree

I like to keep this in the root reducer.  Since reducers can't access other reducers (lies -- it can, but it's complicated), my preference is to not have the reducer file have an opinion.

I like to move that decision upstream.  Up to the root reducer where you use Redux's `combineReducers()`.

So, that brings us back to reduxsauce.  Here's how we handle exporting the reducer from our file:

```js
export default createReducer(INITIAL_STATE, HANDLERS)
```

That's it.


#### Complete Example

Here's a quick full example in action.

```js
// sampleReducer.js
import { createReducer } from 'reduxsauce'
import Types from './actionTypes'

// the initial state of this reducer
export const INITIAL_STATE = { error: false, goodies: null }

// the eagle has landed
export const success = (state = INITIAL_STATE, action) => {
  return { ...state, error: false, goodies: action.goodies }
}

// uh oh
export const failure = (state = INITIAL_STATE, action) => {
  return { ...state, error: true, goodies: null }
}

// map our action types to our reducer functions
export const HANDLERS = [
  [Types.GOODS_SUCCESS]: success,
  [Types.GOODS_FAILURE]: failure
]

export default createReducer(INITIAL_STATE, HANDLERS)
```

This becomes much more readable, testable, and manageable when you reducers start to grow in complexity or volume.

#  createTypes

Use `createTypes()` to create the object representing your action types.  It's whitespace friendly.

```js
// Types.js
import { createTypes } from 'reduxsauce'

export default createTypes(`
  LOGIN_REQUEST
  LOGIN_SUCCESS
  LOGIN_FAILURE

  CHANGE_PASSWORD_REQUEST
  CHANGE_PASSWORD_SUCCESS
  CHANGE_PASSWORD_FAILURE

  LOGOUT
`)

```

# Changes

### May 17, 2016 - 0.1.0

* `NEW` adds createTypes for clean type object creation - [@skellock](https://github.com/skellock)

### May 17, 2016 - 0.0.3

* `DEL` removes the useless createAction function - [@skellock](https://github.com/skellock)

### May 17, 2016 - 0.0.2

* `FIX` removes the babel node from package.json as it was breaking stuff upstream - [@skellock](https://github.com/skellock)

### May 17, 2016 - 0.0.1

* `NEW` initial release - [@skellock](https://github.com/skellock)

