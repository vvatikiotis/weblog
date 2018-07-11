---
Author: Vassilis Vatikiotis
Tags: React Javascript
Date: 14 December 2017
Title: Context
Series: Advanced React Training
Series URL: https://courses.reacttraining.com/
Series Author: Ryan Florence
---

# Context in React

We use `React.cloneElement` in order to clone immediate children in order to pass
them props. This restricts the app developer ability to structure element the way
he/she wants since, as we said, `cloneElement` clones immediate children, thus
becomes brittle as the element structure changes.

Context provides us a way to do away with this. We no longer have to clone elements
in order to pass them any props we need. Instead, we put any props we need in a
parent context and retrieve it from _any_ child, in any depth we need, from `context`
prop.

* Context can include anything, i.e. props, state, computed values, even handlers.
