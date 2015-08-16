####Modules

- CommonJS - node.js module
- AMD (Asynchronous Module Definition), Require.js follows this, for async loading.

#####Principals of Modularity
Modules should be:

- Specialised
- Independent
- Recomposable: arrange modules together in different ways
- Substitutable

*Open Closed Principal*: A module interface is open to extension, but closed to modification.

#####Interfaces

Interfaces define a contract a module will fullfil. Check https://jsbin.com/didejov.

Brief summary of the jsbin code:

- Define interface
- Define `localStorageProvider` and `cookieStoragePRovider`.
- Depending on which on is supported, create the object and assign it `storage`. Note that the providers and the `post` API *close* over this variable `storage`, so they manipulate it accordingly. `post.save` use it to set (`storage[this.id] = this.data;`) and the providers have access to it to `JSON.stringify()` it.

#####The module pattern

The module pattern uses an IIFE. The downside is that global namespace pollution cause for *each* module a global variable is exposed.

Instead, you can pass a specific namespace variable so at the end of the IIFE the module API is exposed under this specific namespace.

```javascript
var app = {}; 

(function (exports) {

	(function (exports) { 
		var api = {
			moduleExists: function test() { 
				return true;
			} 
		};

		$.extend(exports, api);
	}((typeof exports === 'undefined') ?
          window : exports));

}(app));
```

#####AMD

Wraps the module in a `define` function with the following signature:
`define([moduleId,] dependencies, definitionFunction);`
Leave the `moduleId` out (out of favour) to define an anonymous module.

*Check the section for more info, if you need to use AMD*

#####ES6 Modules

`module.exports` and `require` are replaced with the `export` and `import` keywords. Read about the ES6 module spec in http://exploringjs.com/es6/ch_modules.html

#####Building Client-Side Code with CommonJS, npm, Grunt and Browserify

(I dont think Im going to use this workflow, Gulp and Webpack are my favs for the moment). Still this merits a read if and when it's needed.