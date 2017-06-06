<!-- TOC -->

- [On Redux](#on-redux)
  - [Foundation](#foundation)
    - [Three Principles](#three-principles)
    - ["Language" and "Meta Language"](#language-and-meta-language)
  - [How Redux works](#how-redux-works)
    - [The core of Redux](#the-core-of-redux)
    - [Selling Point: Redux Dev Tools](#selling-point-redux-dev-tools)
    - [react-redux and `connect`](#react-redux-and-connect)
    - [react and reselect](#react-and-reselect)
    - [Summary of Redux tech reqs](#summary-of-redux-tech-reqs)
  - [Intent and design of Redux](#intent-and-design-of-redux)
    - [Influences and goals](#influences-and-goals)
    - [Design Principles and Intent](#design-principles-and-intent)
  - [Redux in Practice](#redux-in-practice)
    - [Action, Action Constants and Action Creators](#action-action-constants-and-action-creators)
    - [Actions, Action Creators and Reducers file structure](#actions-action-creators-and-reducers-file-structure)
    - [Slice reducers and reducer composition](#slice-reducers-and-reducer-composition)
    - [Switch statements](#switch-statements)
    - [Middleware for Async Logic](#middleware-for-async-logic)
      - [Thunks, Sagas, Observables, etc](#thunks-sagas-observables-etc)
    - [Dispatching 3-phase async actions](#dispatching-3-phase-async-actions)
    - [Normalized Data](#normalized-data)
    - [Selector functions](#selector-functions)
    - [react-redux: `connect`, `mapState` and `mapDispatch`](#react-redux-connect-mapstate-and-mapdispatch)
  - [Philosophy and Variations in Usage](#philosophy-and-variations-in-usage)

<!-- /TOC -->

# On Redux

Based on reading and thoughts on the series [Idiomatic Redux: The Tao of Redux](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/).
I quote and use much of the above material. Copyright and all rights go to the
author(s) of the original materials, wherever these apply.

Redux is such a small library, it exposes only 3 functions. So the complexity of
using it is not inherent (within Redux), but since it's a lego brick
it has to fit within a larger picture/mindset.

Thought: react redux forces you to think in the way that framework designers think,
except that you are the one designing a framework. Good, if you are interested in
this sort of thing.

> It's important to distinguish between how Redux actually works, the ways that Redux
> is intended to be used conceptually, and the nearly infinite number of ways that it's
> possible to use Redux.

Writing a mini Redux would be trivial:

```javascript
function createStore(reducer) {
    var state;
    var listeners = []

    function getState() {
        return state
    }

    function subscribe(listener) {
        listeners.push(listener)
        return unsubscribe() {
            var index = listeners.indexOf(listener)
            listeners.splice(index, 1)
        }
    }

    function dispatch(action) {
        state = reducer(state, action)
        listeners.forEach(listener => listener())
    }

    dispatch({})

    return { dispatch, subscribe, getState }
}
```



## Foundation

### Three Principles

1. Single source of truth. No need to. Component local state is fine, as per 
redux docs. Even multiple stores are possible.

1. State is read only. Nothing prevents you to modify (mutate) the current state

1. Changes are made with pure functions. Reducer functions could alse mutate the
state, or kick off side effects (The Flux pattern describes having multiple stores).
However, single store enables Redux Dev Tools, persisting and rehydrating state is
easier.

    - For performance issues.
    - App-ise a redux app as a component in larger app. However, we could achieve
    this by dynamically importing and merging a piece of a single store.

### "Language" and "Meta Language"

Cheng Lou talks about code (language) and comments/documentation/tests/blog
posts/books/conferences/talks (meta language). Whatever cannot be conveyed with
code is expresed in terms of meta-language. Redux is minimal, so all discussions 
about Redux cannot happen at code level, but rather at the meta-language level.






## How Redux works

### The core of Redux

All the above being said, the implementation of `createStore` does care about 2 things,
a) actions that reach the reducer must be objects, and b) actions must have a `type`
property. These two requirements originate from the Flux Architecture.

`combineReducers` is another case where more constraints are added. Each slice reducer
should return the state itself when it consumes an action of unknown type. It also
expects the current state value to be an object and that there's a correspondence
between reducer functions and state property keys.




### Selling Point: Redux Dev Tools

- time travelling: needs immutability and pure functions
- the Redux Dev Tools extension lives in a separate process (at least in Chrome),
so all actions and state have to be serializable.




### react-redux and `connect`

Immutability becomes an issue. Connect first does a reference equality check on the
root state object. If the root object hasn't changed, then `connect` assumes that
nothing else in the state has changed and skips any render work. If the root object
has changed, the `connect` calls the supplied `mapStateToProps` function, which does
a shallow equality check between the previous returned result and the current. Again,
if the contents are the same then `connect` skips any rendering.

### react and reselect

reselect creates memoizes selector functions (memoization typically relies on reference
equality checks of the input parameters). Mutating the state wouldn't work well in
this scenario either.

### Summary of Redux tech reqs

You are not supposed to disrespect the above, but it's *possible*.



## Intent and design of Redux

### Influences and goals

- Redux wants all actual state update logic to be synchronous and async behavior 
ept separate from the act of updating the state.

- While Flux proposed multiple stores, redux combines these multiple stores in a
single state tree, for easier debugging, time travelling, undo/redo, etc.

- Dev features like time travelling are intended as use-cases. So immutability and
serializability are needed to enable this use case.

- Single root reducer function = explicit dependency ordering, rather than relying
on mechanisms like Flux's `store.waitFor` event emitter to setup dependency chains.

Interesting link: [an early version of the Redux README](https://github.com/reactjs/redux/commit/07cf1424cb5e7bb7284acb282c996690cfc6d8c5#diff-04c6e90faac2675aa89e2176d2eec7d8).
Notable findings in the above link:

> - An experiment in hot-reloadable Redux.
> - Future dev tools with replay actions during hot reloading.
> - Easy to test things in isolation without mocks
> - No cursors or observables in core. Keep Flux.
> - Store can be hydrated, works for isomorphic apps.
> - All data in a tree
> - Extendible.
> - Flat stores, compose and reuse stores.
> - Minimal API surface


### Design Principles and Intent

- Redux was intended to be another Flux implementation. Many concepts of Flux present
like, dispatching actions, actions are objects with a `type` field, use of action
creators, 'update logic' should be decoupled from the rest of the application.

- Make it easier for developers to understand when, why and how a given piece of state changed.
If data is wrong in Redux store then trace back which dispatched action caused that
state and work backwards from there.

- Redux doesn't care about the actual value of the action's `type`, so the clear
intent is that action type should have some kind of meaning and information. This means
that strings are more useful than Symbols/numbers in terms of conveying information.
Additionally, having more distinct action types is better than  having one or two, cause
having just, say, one action type of, again say, `SET_DATA` would be more difficult to
track down where particular action was dispatched from; also, dispatch log will be
less readable and useful.

- Redux is *explicitly* build with FP concepts and intended to introduce FP concepts
 such as immutability, pure functions and function composition. At the same time,
 it aims at productivity, without overwhelming the user with too many abstract FP
 concepts or terminology.

- Redux promotes testable code. Reducers are pure functions which means that it
 can be tested easily in isolation.

- Since Redux takes the concept of Flux's multiple stores and merges them into a
single combined store, the most straightforward mapping between Flux and Redux would be,
e.g. a `UsersStore`, a `PostsStore` and a `CommentsStore` would map to a root state
tree that looks like `{ users, posts, comments }`. It would be possible to have a
single updater function (reducer) to update all state slices, but any meaningful app
would split the logic based on state slice. Every reducer is responsible for its
own state slice. The `combineReducers` is included to help with reducer composition.

- Having a single action can result in multiple reducers firing, thus multiple updates
in different state slices.

- No magic, everything is intended to be clear, specific, traceable, with minimal
abstraction.

- Unlike Flux's use of `store.waitFor` to set up dependency chains, with Redux this
sequencing can be accomplished by calling specific reducer functions in sequence.

- Minimal API. "Often the best API is no API".

- Extendible. Minimal core. No opinions. Examples: no built in async functionality,
instead, provide middleware API.




## Redux in Practice

### Action, Action Constants and Action Creators

- **Actions**: Using object -> serializable -> time-travel debugging and hot reloading.
- **Action constants**: Consistent naming, easy to see action types, static analysis
in IDEs.
- **Action creators**: Encapsulate logic around creating actions, moving logic out
of components and **act as an API**.

More specific on action creators [Why use action creators?](http://blog.isquaredsoftware.com/2016/10/idiomatic-redux-why-use-action-creators/):

- **Basic abstraction**: move logic out of components and abstract it, in one place
- **Documentation**: Function parameters act as documentation.
- **DRY**
- **Encapsulation/consistency**: move knowledge out of components. Separation of
concerns.
- **Testability**: components are easier to test cause there's no need to reference
`dispatch`. Reuse with, even, something other then React.

For me (also), the easiest approach is to use pre-bound action creators in components.
No need to pass `dispatch`. Passing an object literal for binding actions in `connect`
does this trick. `bindActionCreators` does something similar, however the former
approach is more agreeable to me.

I like manageable and flexible:

> "I've also seen concerns that prolific use of Redux reduces React to nothing more
than a "fancy templating language", or a reinvention of MVC under another name.
But, to me, consistent use of action creators promotes readable code with a reasonable
level of abstraction, is in keeping with software engineering principles of encapsulation
and DRY, and provides just enough decoupling between "display logic" and "business logic"
to keep a codebase manageable and flexible."

*Think of action creators as the reducer API. Reducer and action are separate, with
creators standing in the middle. Bonus: no logic in components.*

Plain objects actions is a core deign principle of redux. Action constants and action
creators are up to the developer, but both are derived from good software engineering
practices.




### Actions, Action Creators and Reducers file structure

I use the community-defined ["ducks" pattern](https://github.com/erikras/ducks-modular-redux).
I feel that using this pattern doesn't guide me away from the idea of multiple slice
reducers. On the contrary, I feel that it complies with and work very well with the
[Fractal Project Structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure).

Takeaways from the Fractal structure:

- Routes can be bundled into 'chunks' using webpack's code-splitting. Each chunk
can be programmed as a 'self-contained' mini app, with its own components and reducer.
This fits well with the idea of slice reducers.
- Global app state can be stored in a slice, very close to the main trunk.

### Slice reducers and reducer composition

The basic concept is that there is only a single reducer function, which is the one
passed to `createStore` and the only one required to have the signature `(state, action) => newState`. It's possible to do everything inside this function but then again
this goes against best software engineering practices: split up large sections to
smaller ones for better, well, everything.

So, split the redux state tree to slices and have each slice updated by a slice
reducer function with the same signature as the root reducer, `(state, action) => newState`. As far as each reducer is concerned, the `state` parameter *is* the entire
state. That's a nice composable approach.

An action can be handled by any one or more or all slices. This keeps components
decoupled from the actual data changes. The duck pattern groups actions to a 
slice reducer but this is definitely not the only way and you should break out
of this paradigm if and when you need to handle an action in more than one reducers.

### Switch statements

My opinion? I use them. No problem. We can use if/else statements, switch statements,
lookup tables. Everything is fine.

### Middleware for Async Logic

> Because middleware form a pipeline around `dispatch` and can modify/intercept/interact
with anything coming through that pipeline, and are given references to the store's
`dispatch` and `getState` methods, they form a loophole where async behavior can occur
but still interact with the store during the process.

#### Thunks, Sagas, Observables, etc

You don't *need* middleware to use async logic in Redux but it's the idiomatic
approach.

Depending on the use case there are 5ish side effect approaches:

- Functions: use of plain functions. `redux-thunk` allows passing functions to
dispatch. Useful for simple async logic like fire-and-forger AJAX calls.
- Promises: use promise as arguments to dispatch, usually dispatching actions as
promises resolve or reject. `redux-promises`; `redux-pack` adds more convention
and guidance to async transactions.
- Generators: use of ES6 generator functions. `redux-saga` enables complex async
workflows via background-thread-like functions, called sagas.
- Observables: use of Observable / FRP libraries like RxJS. `redux-observable`;
`redux-logic`.
- Other: popular is `redux-loop`, similar to how the Elm language works.

Goog comparison at [ What is the right way to do asynchronous operations in Redux?.](https://decembersoft.com/posts/what-is-the-right-way-to-do-asynchronous-operations-in-redux/)

### Dispatching 3-phase async actions

It's common to dispatch multiple actions like `PENDING`, `SUCCESS` or `FAILURE`
when you are making AJAX calls.

> ...you're definitely not required to dispatch actions like this when you make
AJAX requests, but having those actions and the corresponding state values can be 
useful for updating the UI or other aspects of the application.

### Normalized Data

[Normalizing State Shape](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html)
in Redux docs.

- Each item is kept in only one place, so when we need to update we do it in one
place only.
- Reducer logic is simpler when it doesn't have to deal with deep levels of nesting.
- Logic for retrieving/updating an item is simpler .
- No unnecessary rerenders when we update unrelated parts of a state tree. 

> Note that a normalized state structure generally implies that more components are
connected and each component is responsible for looking up its own data, as opposed
to a few connected components looking up large amounts of data and passing all that
data downwards. *As it turns out, having connected parent components simply pass
item IDs to connected children is a good pattern for optimizing UI performance in
a React Redux application, so keeping state normalized plays a key role in improving
performance.*

Performance wise, use it if and when you need it, not before.


### Selector functions

Writing code to access certain nested portions of the state tree is perfectly legal,
however as the lookup becomes more complicated it's a good strategy to abstract it
away to functions. Thus we have the notion of selectors.

Reselect was created from that concept. 2 notable points:

1. It encourages composability as multiple selectors can be composed.
1. It permits memoization (aka caching) which means that expensive lookup will only
be performed when the concerned portion of the state tree truly updates (new value).
This is good for performance and plays well with swallow equality checks in `connect`.


### react-redux: `connect`, `mapState` and `mapDispatch`

Perfectly possible to import the store to every component file and operate using it.
However, as good engineering practice dictates we should encapsulate this logic so
the users won't have to repeatedly deal with it.

Enter `connect`, which:
- handles the store subscription process
- implements the logic of extracting pieces of the state the component needs
- ensures the component rerenders when necessary, and not more
- abstracts away the state shape so the component doesn't need to know state shape
details

`mapState` is a selector function which receives the state and *may* receive
the component's own props as inputs and returns an object whose contents (i.e. keys)
are the component's props.

`mapDispatch` injects `dispatch` allowing components to dispatch actions. If no
`mapDispatch` is provided then the `dispatch` function itself is provided as a prop
to the component, otherwise the the keys of the object returned from `mapDispatch`
are turned into props. `connect` allows for an "object shorthand" syntax where an
object of action creators can be passed instead of an actual `mapDispatch` function.

Using `connect` can be thought of as a way of lightweight dependency injection, where
using React's `context` we are making the store reference available to the component.
Easier testing, opens up the possibility of using separate stores or even overriding/wrapping
the store functions that are expose to nested components.



## Philosophy and Variations in Usage

