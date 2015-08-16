- A **closure** is creates when a function references data that is contained outside its scope. (because of JS function scope, an inner function declaration can reference data declared on a parent scope).
- A **closure** stores function state even if the function is out of scope, even if the function has returned.
- A **lambda** is function that is used as data, i.e. as a value, as input, as a return value, as argument to another function.
- Named function expression VS anonymous function expression.
- Don't use *this* if you expect a function to be invoked on its own. *this*, in that case, will refer to the global/window object.
- Function declarations are hoisted.
- Function expression hoisting works exactly like variable declaration hoisting. 

#####Method design
- *Named parameter list*: Use an object (say, options) as single argument and *extend* a new object with the options obj.
- *Function polymorphism*: polymorphic functions behave differently based on their arguments. Use *slice* on the *arguments* array-like object. 
  `var args = [].slice.call(arguments, 0)`
    - Dynamic dispatch: select teh appropriate method by checking the parameters passed on runtime.
    - Generics/Collection polymorphism: in JS, objects and arrays are collections. 
- *Method chaining*: enables *fluent* APIs, i.e. can read like natural language.
- *Lambda expressions*:

#####Functional programming
- Abstract algorithms from datatypes: use higher order functions to abstract list iteration boilerplate from algorithm implementation. Example: *sort* collections based on different criteria. The comparison function can be passed as an argument to `[].sort` .
- Pure functions: stateless functions, they do not modify anything defined outside their **own** scope.
  - can be run in parallel.
  - can be chained.
- Partial application and Currying
  - Partial application wraps a function that takes a number of arguments and returns another function with fewer arguments, by *fixing* (or applying) some arguments.  
    - Currying: transform a function that takes multiple args into a *chain*of functions, each of which doesn't take more than 1 arg. Example: `add(1, 2, 3) -> add(1)(2)(3)`. Since JS functions take multiple args, it's not common to see true currying in JS apps.

#####Asynchronus operations
- Callbacks
- Promises: objects that allow you to add callback functions in success or failure queues.
- Deferred: objects that control the promises, with a few extra methods, very often a `resolve()` and a `reject()`, that trigger the corresponding callbacks.

#####Conclussion
- Function are powerful.
- Closures can act like object, *to the extend of replacing them and use only closures when we need objects*.
