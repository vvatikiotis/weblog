- Everything is an object, **even primitives**!
  - Primitives get wrapped in an object whenever you use the property access notation. You can't assign properties to them though, cause this wrapping is done **temporarily** and then it gets thrown away.
  - Common OO patterns aren't needed. e.g. the *singleton* pattern is just an object literal in JS.

#####Classical inheritance is obsolete
GoF say:

- Programming to an interface, *not* an implementation.
- Prefer composition over inheritance.
    - inheritance exposes parent class implementation to children. 
   - Children parent tight coupling.

Inheritance has several issues:

- Tight coupling of parent children. Children have intimate knowledge of parent. Breaks encapsulation, cause it links children to ancestors' implementation.
- Inflexible hierarchies.
- Multiple inheritance is complicated!
- Brittle architecture. Tight coupling means it's difficult to refactor a class with a 'wrong' design.
- Gorilla/banana problem: Sometimes you want parts of the parent/ancestor, instead you get the whole ancestor features. You want banana, you get gorilla.

Many of the GoF patterns were designed to address the above problems.

**If you have to work with 'classical' inheritance in JS, remember, keep the hierarchy short**

#####Fluent style JS

JS some of the best power features:

- Lambdas and closures.
- Object literal notation.
- Dynamic object extension.
- Prototypes.
- Factories.
- Fluent APIs
  - Read like natural language
  - Usually chainable. 

JS best parts of Scheme (lambdas), Self (prototypal inheritance), Java (syntax).

#####Prototypes

2 usages of prototypes:

- delegate access to a shared prototype.
- make clones of a prototype.

######Delegate prototypes

Prototypal chains stop at Object.prototype.

- An Object literal's prototype is Object.prototype.
- Using Object.create(aProtoObject)

Properties on prototypes **act** like defaults. You can read them, but as soon as you try to set them, a property with the same name is added to the object. So:

` var proto ={ defaultState: true};
  var obj = Object.create(proto);
  console.log(obj.defaultState); // true;
  obj.defaultState = true; // a defaultState property is set on obj` 

Mutating an object or an array inside a prototype is shared. Changing the value of a property sets the property on the instance, ie instance.

**Sharing state on a prototype property is considered an antipattern** cause of accidental mutations.

######Prototype cloning

Use `extend` provided by Underscore or jQuery. JS6 provides `Object.assign`

When cloning you copy properties from `source` obj to `dest` obj.

######Flyweight pattern

This pattern **is** prototypal delegation. Shared methods are stored in the prototype. Default states can be stored in the prototype as well. We replace member objects and arrays if we want them stored in our object instances.

#####Object creation

Using constructors with: 

- the pure functional style, where methods are stored within each instance object.
- the pseudo classical style, using c'tors and prototype to store shared methods and properties (as defaults)(careful, this is anti pattern).

#####Factories

Factory = method to create objects.

#####Prototypal inheritance with Stamps

Needed features of objects:

- Delegate prototype
- Instance state
- Encapsulation

`Stampit` library. Exports a single function, its signature:
`stampit(methods, state, enclose)`. 

Example usage:

```javascript
var testObj = stampit()
.methods({
	delegateMethod: function delegateMethod() {
        return 'shared property'; 
    }
})
.state({
	instanceProp: 'instance property'
})
.enclose(function () {
	var privateProp = 'private property';
	this.getPrivate = function getPrivate() {
		return privateProp;
	}
})
.create();
```