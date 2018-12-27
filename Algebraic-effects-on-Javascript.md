---
Author: Vassilis Vatikiotis
Tags: Javascript 'Functional Programming'
Date: 27 December 2018
Title: Algebraic Effects in JavaScript
Series: Algebraic Effects in JavaScript
Series URL: https://gist.github.com/yelouafi/57825fdd223e5337ba0cd2b6ed757f53
Series OP: Yasin Yelouafi
---

# Part 1 - Control transfer

A program is said to be written in _direct style_ when functions communicate their
results via `return` statement (in JS). In _CPS_ (Continuation Passing Style), a
function takes a callback as an additional argument and communicates the result
using this callback.

So direct style is

```js
function add(arg1, arg2) {
  return arg1 + arg2;
}
```

and continuation passing style is

```js
function add(arg1, arg2, nextFn) {
  return next(arg1 + arg2);
}
```

`nextFn` is called the _continuation_. See that the result is fed into the provided
callback, so the continuation _continues_.

**Objervation**: Instead of getting a returned value from a function just by
calling it, this returned value becomes available as an argument of the provided
callback (or in other words, the provided continuation)

**Observation**: After converting the direct-style interpreter code to CPS style,
the code became harder to read and reason. A few things to note:

1. Every `return` statement either calls the continuation function, or another,
   CPS style function
2. When we need to evaluate multiple expressions, we chain those evaluations by
   providing intermediary continuations which capture the intermediate results and
   when the chaining is finished we throw the result onto the original, main
   continuation.

## Some thoughts, conclusions, pros/cons of using CPS

Local control transfer or local jump within the same function.

1. Flow control: In direct style the caller controls what to do next. In CPS, on
   the other hand, the callee is responsible of what to do next, i.e. how and
   when to invoke the passed continuation.
2. Tail call elimination, since the continuation is positioned in tail call position
   and there is nothing to do after the tail call.
3. Provides asynchronicity.
4. Difficult to be used by human but a suitable target for compilers

> Currently, the idiomatic solution in JavaScript is using async/await, this
> effectively gives us 3 and 4 but not 1. We don't have enough power over control
> flow.

## Exceptions

Non-local control transfer or remote jumps, across stack frames, and in JS, in a
context higher in the call hierarchy. `throw` bypasses intermediate function calls
until it reaches a `catch` and then it discards all those intermediate stack frames.

We can use CPS to implement other control flow strategies, like error handling.
The trick is to pass another error/abort continuation, _before_ the normal completion
continuation, which aborts the computation.

## Conclusions (taken verbatim from OP)

1. Direct style relies on the call stack for control transfer

2. In direct style, control transfer between functions is implicit and hidden
   from us. A function must always return to its direct caller

3. You can use Exceptions for making non local control transfer

4. CPS functions never return their results. They take additional callback
   argument(s) representing the continuation(s) of the current code

5. In CPS, control transfer doesn't rely on the call stack. It's made explicit
   via the provided continuation(s)

6. CPS can emulate both local and non local control transfers but...

7. **CPS is not something meant to be used by humans, hand written CPS code
   becomes quickly unreadable**

8. Make sure to read the previous sentence

# Part 2 - Capturing continuations with Generators
