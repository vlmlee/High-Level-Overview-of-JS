### Notes from http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
#### Written by: Dmitry A. Soshnikov
#### Published on: 2010-09-02

# High Level Overview of JS

An __object__ is a collection of properties and has a __single prototype object__. The prototype can either be an __object__ or __the null value__.

For the code:
```javascript 
var foo = {
    x: 10,
    y: 20
};
```

We have a hidden property __proto__ that points to the object’s prototype, which in this case is Object.prototype. There is a notion of __explicit__ and __implicit__ properties. Explicit properties are named while implicit properties are built into the object either through the __prototype chain__ or through the language itself. 

> Note: A prototype chain is the finite chain of objects which is used to implement inheritance and shared properties.

Simple rule: if a property or method is not found in the object itself, then there is an attempt to find this property or method in the prototype chain. This is called __delegation__ or __dynamic dispatch__.

> Note: The _this_ value used in an inherited method is set to the original object and not to the prototype object.

If a prototype is not specified for an object explicitly, the default value of __proto__ will be Object.prototype. Object.prototype also has a __proto__ which is the final link in the chain and it is set to __null__.

## Constructors

A constructor function does a useful thing: it __automatically sets a prototype object for newly created objects__. 

To define shared and inherited properties or methods to an object, we have to add those properties or methods to the object’s prototype.

> Note: Functions also have prototypes called Function.prototype.

## Execution Context Stack

There are three kinds of ECMAScript code: global code, function code, and eval code. Every code is evaluated in its __execution context__.

There is only one global context but there can be many instances of function and eval contexts. Every call of a function enters the function execution context and evaluates the function code type. Every call of eval function enters the eval execution context and evaluates the code.

An __execution context__ may activate another execution context, that is, functions may call other functions and the global context may call other global functions. This is implemented as a stack which is called the __execution context stack__. A context which activates another context is called a __caller__. A context being activated is called a __callee__.

When a caller activates another context, the caller suspends its execution and parses its control flow to the callee. The callee is pushed onto the stack and becomes the running, or __active__ execution context. After the callee’s context ends, it returns control to the caller and the evaluation of the caller’s context proceeds until the end. It may activate other contexts and __pass control flow__ before it ends.

A callee may simply end by returning an __exception__. A __thrown__ but not __caught__ exception may exit one or more contexts, meaning its context is popped from the execution context stack.

The ECMAScript program runtime can be presented as the execution context stack, where the top of the stack is an active context. When a program begins, it __enters__ the global execution context, which is the bottom and the first element of the stack. Then the global code provides some initialization, __creating needed objects and functions__.

After the initialization is done, the runtime system will __wait for some event__ which will __activate__ some function which will allow the program to enter a new execution context. This is how the runtime system of ECMAScript manages the execution of code.

## Structure of Execution Context

Every execution context can be represented as an object. Each one has a set of properties necessary to track the execution progress of its associated code. These properties are: the __variable object__, the __thisValue__, and the __scope chain__.

### Variable Object

A __variable object__ is a __container of data__ associated with the execution context. It's a special object that stores variables and function declarations defined in the context. Notice that __function expressions are not__ included in the variable object. 

> Note: Variable object is an abstract concept.

In ECMAScript, __only functions create new scopes__. Variables and functions defined within a scope of a function __are not visible__ directly outside of that scope so that the global variable object does not get polluted. There is no block scope in ECMAScript. 

When a function is activated, or called by the caller, a special object called the __activation object__ is created. It’s filled with formal parameters and the special arguments object which is a map of formal parameters but with index properties. The activation object is then __used as the variable object__. function Foo(x, y, []) ...

### Scope Chain

A __scope chain__ is a __list of objects that are searched for identifiers__ that appear in the code of the context. If a variable is not found in its own scope, or in the variable / activation object, its look up proceeds to the parent’s variable object and so on.

> Note: Identifiers are names of variables, function declaration, formal parameters, etc. Free variables are variables that are not local to the current scope.

A scope chain is a list of all those parent variable objects, plus the function’s own variable / activation objects. When __resolving__, or looking up, an identifier, the scope chain is searched starting from the activation object. We may assume the linkage of the scope chain objects via the implicit __parent__ property which refers to the next object in the chain. The scope chain lookup is 2-dimensional (1) __the scope link chain__ (2) __into the depth of the link’s prototype chain__.

Before we go to __parent__, first __proto__ is considered. There is nothing special about getting parent data from inner functions. After a context ends, however, all its state and it itself is destroyed. At the same time, an inner function may be returned from the parent function. 

The __upward funarg problem__ appears when a function returns __up__ to the outside from another function and uses free variables. To be able to access variables of its parent’s contexts __even after__ the parent context ends, __the inner function saves in its scope property its parent’s scope chain at creation__. When the function is activated, the scope chain of its context is formed as a combination of the activation object and this scope property. 

> Note: Scope chain = [[Scope]] + Activation Object

The main point: exactly __at creation__ a function saves its parent’s scope chain because exactly this scope chain will be used for variable lookup in further calls of the function. This style of scope is called __lexical or static__ scope. Static scope is used in the __downward funarg problem__. Dynamic scope is __not__ used in Javascript.

A __closure__ is a combination of a code block and statically / lexically saved parent scopes. Through these saved scopes a function may easily refer to free variables. Since every function saves scopes at creation, theoretically, __all functions are closures__. Several functions may have the same parent scope. In this case, variables stored in the [[Scope]] property are shared between all functions having the same parent scope chain. 

> Note: Double square brackets [[Foo]] means that Foo is an internal property of an object but are not part of the ECMA specification. This is to say that they are not identifiers that are accessible as members and are not to be used in code.

Changes in variables __made by one closure are reflected by reading the variables__ of another closure. 

> Closures are anonymous functions that remember the stack frame which they were instantiated in. They save the parent scope, variable object, and context.

### This Value

A __this__ value is a special object which is related to the execution context. It may be named as a context object, i.e. an object in which context the execution context is activated.

__Any value__ may be used as __this__ value of the context. A this value is the property of the execution context but not the property of the variable object. The this value __never__ participates in the identifier resolution process. The this value is taken __directly from the execution context__ and without any scope chain lookup. The value of __this__ is determined only when entering the context. 

In ECMAScript, it is not possible to change the value of __this__ because it is __not__ a variable.

