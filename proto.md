In JavaScript every value except `null` and `undefined` has a `__proto__` property.

The `__proto__` property for any value points to some object.

If there is ever a property lookup on a value that fails, the interpreter will go looking for the property on the object that the value's own `__proto__` property points to, until either the property is found, or, it arrives at a value without a `__proto__` property.

This fact can be used to our advantage. We can store properties (often methods), that we wish to share amongst different values, in some object, and then, make sure that these different values all, either directly or eventually, attempt property lookup in this object via the objects referenced in their own, and/or in ancestor value's `__proto__` property.

This fact can be used to our advantage in the following way:

- Store properties (including methods) that we wish to share amongst different values in some object
- For each of the values that we wish to share the properties, make sure its `__proto__` property points to the object with shared properties...
- ...or, points to another objects whose `__proto__` property points to the object with the shared properties

## Creating Objects with their `__proto__` Pointing to a Programmer Defined Object

`Object.create(someObject)` creates a new value of type `'object'` with a `__proto__` property pointing to `someObject`.

Therefore:

```javascript
const sharedProperties = {};
sharedProperties.one = 1;

let newObject = Object.create(sharedProperties);

console.log(newObject.one); // 1
```

Because property lookup continues until the JavaScript interpreter reaches an object without a `__proto__` property, the following also works:

```javascript
const sharedProperties = {};
sharedProperties.one = 1;

let anotherNewObject = Object.create(sharedProperties);

console.log(anotherNewObject.one); // 1

let yetAnotherNewObject = Object.create(anotherNewObject);

console.log(yetAnotherNewObject.one) // 1
```

## The `prototype` Object on Functions

Recall that in JavaScript, functions are perhaps more accurately thought of as *function objects*, upon which we can set and lookup properties:

```javascript
function aFunction() {}

aFunction.one = 1

console.log(aFunction.hasOwnProperty('one')); // true
console.log(aFunction.one); // 1
```

When functions are defined, they are automatically given a property called `prototype`, which is simply an object containing a single property `constructor`, which is a reference to the function that the `prototype` object belongs to.

```javascript
function aFunction() {
  console.log('I am a function');
}

console.log(typeof aFunction.prototype);             // 'object'
console.log(typeof aFunction.prototype.constructor); // 'function'
console.log(aFunction.prototype.constructor.name);   // 'aFunction'
console.log(aFunction.prototype.constructor());      // logs 'I am a function'
```

*(This article will avoid a long discussion of the `constructor` property, focusing instead on the `prototype` object itself.)*

This `prototype` property on functions, which is an object, can be given additional properties:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

console.log(aFunction.prototype.one); // 1
```

Like any other object, any value's `__proto__` property can point to one of these `prototype` objects. Here we create an object with its `__proto__` property pointing to a function's `prototype` property by using `Object.create`:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

let newObject = Object.create(aFunction.prototype);

console.log(newObject.__proto__ === aFunction.prototype); // true
console.log(newObject.one);                               // 1
```

## Calling Functions with `new`

When a function is called with the `new` keyword prefixed to it, the JavaScript interpreter silently creates a new object whose `__proto__` property points to the function's `prototype` object. We could imagine this invisible line of code looking something like the following:

```javascript
function aFunction() {
  // This line is only created when calling the function with `new`
  let newObject = Object.create(aFunction.prototype);
}

new aFunction();
```

Additionally, if the function does not already contain a `return` statement, calling it with the `new` keyword will also, invisibly, cause the function to return the object that was invisibly created on the first line of the function call:


```javascript
function aFunction() {
  // These lines are only created when calling the function with `new`
  // and are invisible to programmers
  let newObject = Object.create(aFunction.prototype);
  return newObject
}

new aFunction();
```

For these reasons:

- Functions called with the `new` keyword and no explicit `return` statement return objects
- These objects each have a `__proto__` property that points to the `prototype` object that was automatically created on the function when the function was defined
- When using the objects returned by functions called with the keyword `new`, should they not contain a property that is asked for, the JavaScript interpreter will go looking for that property in the `prototype` property of the function called with `new` that created the object to begin with.

Thus we can use functions as creators of objects that share properties as follows:

```javascript
function makeObjectWithSharedProperties() {}

makeObjectWithSharedProperties.prototype.one = 1;
makeObjectWithSharedProperties.prototype.two = 2;

let object1 = new makeObjectWithSharedProperties();
let object2 = new makeObjectWithSharedProperties();

console.log(object1.one) // 1
console.log(object1.two) // 2
console.log(object2.one) // 1
console.log(object2.two) // 2
```

## `__proto__` and `prototype` in the Wild

Consider the following:

```javascript
console.log(typeof Array);                          // 'function'
console.log(typeof Array.prototype);                // 'object'

let array = new Array(1, 2, 3);                     // <- equivalent to `let array = [1, 2, 3]`

console.log(typeof array.__proto__);                // 'object'
console.log(array.__proto__ === Array.prototype);   // true
console.log(array.hasOwnProperty('pop'));           // false
console.log(Array.prototype.hasOwnProperty('pop')); // true

array.pop();                                        // works just fine
```
