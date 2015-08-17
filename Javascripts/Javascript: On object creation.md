####General
- An object's  *prototype attribute* specifies the object to which it delegates unknown methods (and properties, but that is considered an anti-pattern).
- The prototype attribute is implemented as `__proto__` in some browsers. **Do not rely on it!**
- In bibliography, it is also denoted as `[[Prototype]]`. 
- The prototype attribute is set when object is created.
- `Object.prototype`  *is* an object. A rare one! 
      - It does not inherit any properties.
      - It does have certain properties and methods of its own, like `toString`, `isPrototypeOf`, etc.
 
####Functions
- The prototype attribute of a function is `function()` and the prototype attribute of it is `Object.prototype`.
- Every function has a `prototype` property. It's an `Object`, hence its `[[Prototype]]` is `Object.prototype`.

#### Object literals
- The prototype attribute of the new object  is `Object.prototype`.

####Using new
- The prototype attribute of the new object is the value of the `prototype` property of their constructor function., i.e.
 
 When `var o = new SomeConstructor();`, `o` delegates to `SomeConstructor.prototype`.
- `new` uses the prototype's `constructor` function.
- It is *problematic* when *subclassing* because you have to set `Subclass.prototype.constructor = Subclass` manually.

####Using Object.create
- The prototype attribute of the new object  is the first argument. 
- `Object.create({})` and `obj={}` have the same prototype attribute `Object.prototype`.
