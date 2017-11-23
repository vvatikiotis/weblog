---
Author: Vassilis Vatikiotis
Tags: React Javascript
Date: 23 November 2017
Title: On passing state
---

# On passing state

I want some lifecycle methods and some state that I want to share across the app

* Compound components with `cloneElement`, usually when the app developer
  doesn't care about state cause it is handled by the compound component.
  Usually used when the library/shared component author wants to isolate/hide
  the state _and_ the app developer doesn't care about that specific state. Good
  practice when you write reusable components to be used by other developers.
  Implicit state.

* Use context, implicit state.

* Use Higher Order Components to compose behavior across components. Explicit
  state.

* Render prop callback, aka Function as children. This pattern enables me to
  compose the behavior of a reusable component _and_ its state. in other words,
  it enables us to move behavior out of one component into another _reusable_
  component and compose state (state handled by the reusable component) and use
  it throughout the app. Explicit state.
