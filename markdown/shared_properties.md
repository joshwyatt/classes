# Shared Properties with `__proto__` and `prototype`

## Objectives

After completing this section of the repo you should be able to:

- Descibe the `__proto__` property on JavaScript values
- Create objects with their `__proto__` pointing to a programmer defined object
- Describe the `prototype` object on functions
- Call functions to return objects with `new`, and understand the implications for the object's `__proto__` property
- Understand `__proto__` and `prototype` in native JavaScript code
- Prove that in JavaScript, everything is nothing

## The `__proto__` Property on JavaScript Values

**In JavaScript every value except `null` and `undefined` has a `__proto__` property. The `__proto__` property for any value points to some object.**

:question: What will the following log to the console?

```javascript
let object = {a: 1};
let number = 45;
let string = 'string';
let boolean = true;
let func = () => {};
let u = undefined;
let n = null;

// `!!` will typecast a value to a boolean
console.log(!!object.__proto__);
console.log(!!number.__proto__);
console.log(!!string.__proto__);
console.log(!!boolean.__proto__);
console.log(!!func.__proto__);
console.log(!!u.__proto__);
console.log(!!n.__proto__);
```

**If there is ever a property lookup on a value and that value does not itself contain that property, the interpreter will go looking for the property on the object that the value's own `__proto__` property points to, until either the property is found, or, it arrives at a value without a `__proto__` property.**

This fact can be used to our advantage in the following way:

- Store properties (including methods) that we wish to share amongst different values in some object
- For each of the values that we wish to share the properties, make sure its `__proto__` property points to the object with shared properties...
- ...or, points to another objects whose `__proto__` property points to the object with the shared properties

:speak_no_evil: Read the following block of code out loud, indicating what you believe will be logged to the console and why:

```javascript
let z = {
  one: 1
};

let a = {};
console.log(a.one);

// Don't ever actually overwrite `__proto__` as we are doing here for instructional purposes.
// We will look at a better way to accomplish what we are doing here immediately below.
a.__proto__ = z;
console.log(a.one);

let b = {};
b.__proto__ = a;
console.log(b.one);
```

**In JavaScript, _inheritence_ means that an object utilizes properties stored on an object available via `__proto__` lookups, which we call _the prototype chain._**.

## Creating Objects with their `__proto__` Pointing to a Programmer Defined Object

**`Object.create(someObject)` creates a new value of type `'object'` with a `__proto__` property pointing to `someObject`.**

Therefore:

```javascript
const sharedProperties = {};
sharedProperties.one = 1;

let newObject = Object.create(sharedProperties);

console.log(newObject.one); // 1
```

Remembering that property lookup continues until the JavaScript interpreter reaches an object without a `__proto__` property, the following also works.

```javascript
const sharedProperties = {};
sharedProperties.one = 1;

let anotherNewObject = Object.create(sharedProperties);

console.log(anotherNewObject.one); // 1

let yetAnotherNewObject = Object.create(anotherNewObject);

console.log(yetAnotherNewObject.one) // 1
```

:star: Complete the `makeObjectWithSharedProperties` function below so that it creates and returns an object that itself contains no properties, but, will be able to access the properties defined on `sharedProperties`.

```javascript
let sharedProperties = {
  one: 1,
  logger: function() {
    console.log('this is logger');
  }
};

function makeObjectWithSharedProperties() {
  // Create and return an object that will have access to the
  // properties defined on `sharedProperties` above
  // such that the `console.log`s below log true.
}

let objectWithSharedProperties1 = makeObjectWithSharedProperties();
let objectWithSharedProperties2 = makeObjectWithSharedProperties();
console.log(objectWithSharedProperties1.one === 1);                                     // should log `true`
console.log(objectWithSharedProperties1.logger === objectWithSharedProperties2.logger); // should log `true`
```

## The `prototype` Object on Functions

Recall that in JavaScript, functions are perhaps more accurately thought of as *function objects*, upon which we can set and lookup properties:

```javascript
function aFunction() {}

aFunction.one = 1

console.log(aFunction.hasOwnProperty('one')); // true
console.log(aFunction.one); // 1
```

**When functions are defined, they are automatically given a property with the name `prototype`, which is simply an object containing a single property `constructor`, which is a reference to the function that the `prototype` object is a property on.**

:speak_no_evil: Read the following code block out loud, being sure to explain why each expression logs what it does:

```javascript
function aFunction() {
  console.log('I am a function');
}

console.log(typeof aFunction.prototype);             // 'object'
console.log(typeof aFunction.prototype.constructor); // 'function'
console.log(aFunction.prototype.constructor.name);   // 'aFunction'
console.log(aFunction.prototype.constructor());      // logs 'I am a function'
```

:star: In the browser's JavaScript console, create a function and then enter it into the console so that you can inspect it. Look at its `prototype` property, and then, at that `prototype` property's `constructor` property, and then, at that `constructor` property's `prototype` property. How far do you think you could continue this?

>This article will avoid a longer discussion of the `constructor` property, focusing instead on the `prototype` object itself.

This `prototype` property on functions, which is an object, can be given additional properties:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

console.log(aFunction.prototype.one); // 1
```

Like any other object, any value's `__proto__` property can point to one of these `prototype` objects. Here we create an object with its `__proto__` property pointing to a function's `prototype` property by using `Object.create`:

:speak_no_evil: Read the following code block out loud, making a case for why it logs what it does:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

let newObject = Object.create(aFunction.prototype);

console.log(newObject.__proto__ === aFunction.prototype); // true
console.log(newObject.one);                               // 1
```

## Calling Functions with `new`

**When a function is called with the `new` keyword prefixed to it, the JavaScript interpreter silently creates a new object whose `__proto__` property points to the function's `prototype` object.**

We could imagine this invisible line of code looking something like the following:

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

console.log(object1.one); // 1
console.log(object1.two); // 2
console.log(object2.one); // 1
console.log(object2.two); // 2
```

:question: What will the code below log to the console and why?

```javascript
function makeObjectWithSharedProperties() {}

makeObjectWithSharedProperties.prototype.one = 1;
makeObjectWithSharedProperties.prototype.two = 2;

let object1 = new makeObjectWithSharedProperties();
let object2 = makeObjectWithSharedProperties();

console.log(object1.one);
console.log(object1.two);
console.log(object2.one);
console.log(object2.two);
```

## `__proto__` and `prototype` in Native JavaScript

:speak_no_evil: Read the following section of code out loud, making arguments for each of the logs:

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

:question: Why does `'rowan'.toUpperCase()` work? Be sure to use the terms `__proto__` and `prototype` in your answer.

:star: Using the browser console, prove the claim that "in JavaScript everything is nothing". Do this by digging into values of different types and showing that excpet for `undefined` they all, eventually, inherit from `null`. Note the prototype chain for each type of value on its way to eventually inheriting from `null`.

## Conclusion

By calling functions with the `new` keyword, we can use them as class constructor functions that return object instances, each which will look for shared properties and methods defined on the class constructor's `prototype` object.

Recall in the [Building Objects with Functions](building_objects_with_functions.md#the-desire-to-share-object-properties-and-methods) section the distinction made between object properties that are identical amongst instances, and can therefore be shared, and those that are potentially unique on account of using values passed in to the constructor function, which ought not to be shared amongst instances. This section largely focused on properties intended to be shared amongst instances, and did not discuss at all properties **not** intended to be shared.

Given that the `new` keyword invisibly creates an object and returns it, how can properties be assigned to it? Furthermore, what if methods that are shred amongst instances wish to reference instance specific properties, how would this be possible? The next section makes clear the importance of these questions before moving on to answer them.

## Review

Before moving on to the next section of this repo, be sure you are able to:

- Descibe the `__proto__` property on JavaScript values
- Create objects with their `__proto__` pointing to a programmer defined object
- Describe the `prototype` object on functions
- Call functions to return objects with `new`, and understand the implications for the object's `__proto__` property
- Understand `__proto__` and `prototype` in native JavaScript code
- Prove that in JavaScript, "everything is nothing"

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](building_objects_with_functions.md)
- *Shared Properties with `__proto__` and `prototype`*
- [The Need for Shared Properties with Access to Unique Properties](shared_accessing_unique.md)
