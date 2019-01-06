---
Author: Vassilis Vatikiotis
Tags: React Javascript
Date: 23 November 2017
Title: On passing state
---

# Handling imperative stuff in React

- Use event handlers to just update component state.
- Move all imperative logic out of lifecycle methods.
- Use `componentDidMount` and `componentDidUpdate` to invoke any imperative
  stuff.

That way `state` tells us out component state, at any point in time.

To move further into this technique, we can abstract all lifecycle methods and
imperative stuff into another, inner, component which renders nothing, `null`, and keep
the state into the driving, outer component. The outer component is then
responsible for rendering the first component.

Very useful when we want to wrap imperative libraries with React. The
controlling component's render method is now very informative. We know its
state, we know what the imperative stuff wrapping component does _and_ we can
now compose the imperative stuff.

**The only imperative thing in React is `setState`.**

:)
