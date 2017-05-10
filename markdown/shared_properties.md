# Shared Properties with `__proto__` and `prototype`

## Objectives

After completing this section of the repo you should be able to:

- Describe the `__proto__` property on JavaScript values
- Create objects with their `__proto__` pointing to a programmer defined object
- Describe the `prototype` object on functions
- Describe the use of `__proto__` and `prototype` in built in JavaScript code
- Prove that in JavaScript, "everything is nothing"
- Use constructor functions to create object instances with both properties unique to themselves, and properties shared amongst all instances

## The `__proto__` Property on JavaScript Values

**In JavaScript every value except `null` and `undefined` has a `__proto__` property. The `__proto__` property for any value points to some object.**

:question: What will the following log to the console?

```javascript
let object = {a: 1};
let number = 45;
let string = 'string';
let boolean = true;
let func = () => {};
let array = [1, 2, 3];
let u = undefined;
let n = null;

// `!!` will typecast a value to a boolean
console.log(!!object.__proto__);
console.log(!!number.__proto__);
console.log(!!string.__proto__);
console.log(!!boolean.__proto__);
console.log(!!func.__proto__);
console.log(!!array.__proto__);
console.log(!!u.__proto__);
console.log(!!n.__proto__);
```

:star: In the Chrome browser console, create a value several different types of values, including `null` and `undefined` and look for their `__proto__` properties.

**If there is ever a property lookup on a value and that value does not itself contain that property, the interpreter will go looking for the property on the object that the value's own `__proto__` property points to, until either the property is found, or, it arrives at a value without a `__proto__` property.**

This fact can be used to create properties that are shared amongst object instances in the following way:

- Store properties (including methods) that we wish to share in some object
- For each of the values that we wish to be sharing the properties, make sure its `__proto__` property points to the object with shared properties...
- ...or, points to another object whose `__proto__` property points to the object with the shared properties

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

**`Object.create(someObject)` creates a new object with a `__proto__` property defined on it pointing to whatever was passed in as `someObject`.**

Therefore:

```javascript
let sharedProperties = {};
sharedProperties.one = 1;

let newObject = Object.create(sharedProperties);

console.log(newObject.one); // 1
```

Remembering that property lookup continues until the JavaScript interpreter reaches an object without a `__proto__` property, the following also works.

```javascript
let sharedProperties = {};
sharedProperties.one = 1;

let newObject = Object.create(sharedProperties);

console.log(newObject.one); // 1

let anotherNewObject = Object.create(newObject);

console.log(anotherNewObject.one) // 1
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
  // such that the `console.log`s below log `true`.
}

let objectWithSharedProperties1 = makeObjectWithSharedProperties();
let objectWithSharedProperties2 = makeObjectWithSharedProperties();
console.log(objectWithSharedProperties1.one === 1);                                     // should log `true`
console.log(objectWithSharedProperties1.logger === objectWithSharedProperties2.logger); // should log `true`
```

## The `prototype` Object on Functions

Recall that in JavaScript, functions are perhaps more accurately thought of as *function objects*, upon which we can set and lookup properties. For example:

```javascript
function aFunction() {}

aFunction.one = 1;

console.log(aFunction.hasOwnProperty('one')); // true
console.log(aFunction.one);                   // 1
```

**When functions are defined, they are automatically given a property with the name `prototype`, which is simply an object containing a single property. This single property has the key `constructor`, with the value being a reference to the function that the `prototype` object is a property on.**

:star: Create a named function and log the function's `prototype.constructor` property.

:speak_no_evil: Read the following code block out loud, being sure to explain why each expression logs what it does:

```javascript
function aFunction() {
  console.log('I am a function');
}

console.log(typeof aFunction);                       // 'function'
console.log(typeof aFunction.prototype);             // 'object'
console.log(typeof aFunction.prototype.constructor); // 'function'
console.log(aFunction.name);                         // 'aFunction'
console.log(aFunction.prototype.constructor.name);   // 'aFunction'
console.log(aFunction());                            // logs 'I am a function'
console.log(aFunction.prototype.constructor());      // logs 'I am a function'
```

:star: In the browser's JavaScript console, create a function and then enter it into the console so that you can inspect it. Look at its `prototype` property, and then, at that `prototype` property's `constructor` property, and then, at that `constructor` property's `prototype` property. How far do you think you could continue this?

>This article will avoid a longer discussion of the `constructor` property, focusing instead on the `prototype` object itself.

This `prototype` property on functions, which is an object, can, like any other object, be given additional properties:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

console.log(aFunction.prototype.one); // 1
```

Also like any other object, any value's `__proto__` property can point to one of these `prototype` objects. Here we create an object with its `__proto__` property pointing to a function's `prototype` property by using `Object.create`:

:speak_no_evil: Read the following code block out loud, making a case for why it logs what it does:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

let newObject = Object.create(aFunction.prototype);

console.log(newObject.__proto__ === aFunction.prototype); // true
console.log(newObject.one);                               // 1
```
:star: In the code below, utilize `makeCreature`'s `prototype` property, and, `Object.create` so that the log statements return `true`.

```javascript
function makeBell() {
}

let bell1 = makeBell();
let bell2 = makeBell();

console.log(bell1.ding() === 'ding');                       // Should log `true`
console.log(bell2.ding() === 'ding');                       // Should log `true`
console.log(typeof makeBell.prototype.ding === 'function'); // Should log `true`
console.log(bell1.ding === bell2.ding);                     // Should log `true`
```

## `__proto__` and `prototype` in Native JavaScript

In fact, built in JavaScript code relies heavily on the use of `__proto__` and `prototype` so that developers can do things like use type specific methods (like `Array.forEach` and `String.split`) on every instance of that type, without having to create unique copies of these methods every time a new value is created.

Remember [from above](#the-__proto__-property-on-javascript-values) that all values in JavaScript, except `null` and `undefined`, have `__proto__` properties. With that in mind:

:speak_no_evil: Read the following section of code out loud, making arguments for each of the logs:

```javascript
let array = [1, 2, 3];

console.log(typeof array.__proto__);                 // 'object'
console.log(array.__proto__ === Array.prototype);    // true
console.log(array.hasOwnProperty('pop'));            // false
console.log(array.__proto__.hasOwnProperty('pop'));  // true

console.log(array.pop());                            // 3
console.log(array.length);                           // 2
```

:question: Why does the line of code `'rowan'.toUpperCase()` work in spite of the fact that `'rowan'.hasOwnProperty('toUpperCase')` is `false`? Be sure to use the terms `__proto__` and `prototype` in your answer.

:star: Using the browser console, prove the claim that "in JavaScript everything is nothing". Do this by digging into values of different types and showing that excpet for `undefined` they all, eventually, inherit from `null`. Note the prototype chain for each type of value on its way to eventually inheriting from `null`. Here's an example to get you started:

```javascript
let func = () => {}
console.log(func.__proto__                     === Function.prototype) // true
console.log(func.__proto__.__proto__           === Object.prototype)   // true
console.log(func.__proto__.__proto__.__proto__ === null)               // true
```

:star2: Passing almost any value into `isNothing` below will cause it to return `true`.

- Try it out and explain why it works
- What (legal) value can you pass into it to cause it to throw an error? Once you discover what the value is, explain why it throws an error?
- Why does `isNothing` throw an error when passed no argument?

```javascript
function isNothing(value) {
  return value === null ? true : isNothing(value.__proto__);
}
```

## Objects with Unique Properties and Shared Properties

Recall the `makePerson` function from the [Building Objects with Functions](building_objects_with_functions.md#passing-values-for-dynamic-object-creation) section. Our approach then required that every single property defined on the returned object be made anew each time the constructor function was called:

```javascript
function makePerson(name, age) {
  let person = {
    name: name,
    age: age,
    curious: true,

    sleep: function() {
      return 'zzzzzzzzzz';
    }
  };

  return person;
}
```

:star: Using `Object.create` and the `makePerson.prototype` object, refactor the `makePerson` function so that:

- `name`, and `age` are defined uniquely on the object created by `makePerson` each time it is called
- `curious` and `sleep` are shared properties amongst all objects created and returned by calls to `makePerson`

```javascript
function makePerson(name, age) {
}

let rowan = makePerson('rowan', 0);
let soren = makePerson('soren', 4);

console.log(rowan.age);                     // should be `0`
console.log(soren.age);                     // should be `4`
console.log(rowan.sleep());                 // should be `'zzzzzzzzzz'`
console.log(soren.sleep());                 // should be `'zzzzzzzzzz'`
console.log(rowan.hasOwnProperty('sleep')); // should be `false`
console.log(soren.hasOwnProperty('sleep')); // should be `false`
```

## Conclusion

Constructor functions can return objects that share properties by creating these objects with `Object.create(objectHousingSharedProperties)`, commonly passing in the constructor function's own `prototype` property, which is just an object. This technique utilizes the fact that all values (except `null` and `undefined`) contain a property called `__proto__` which points to an object to use for property lookups when lookups fail on the value containing the `__proto__` property.

Properties intended to be unique amongst instances can simply be defined on the object that the constructor function creates and returns. Thus the following code behaves as expected:

```javascript
function makePerson(name, age) {
  let person = Object.create(makePerson.prototype);
  person.name = name;
  person.age = age;
  return person;
}

makePerson.prototype.sleep = function() {
  return 'zzzzzzzzzz';
}

let rowan = makePerson('rowan', 0);
let soren = makePerson('soren', 4);

console.log(rowan.age);                     // `0`
console.log(soren.age);                     // `4`
console.log(rowan.sleep());                 // `'zzzzzzzzzz'`
console.log(soren.sleep());                 // `'zzzzzzzzzz'`
console.log(rowan.hasOwnProperty('sleep')); // `false`
console.log(soren.hasOwnProperty('sleep')); // `false`
```

By now you know how to:

- Use functions to create and return object instances
- Return instances that contain properties unique unto themselves
- Share properties amongst all instances created by the constructor
- Create instances that contain both unique and shared instances

Something very important that we have not yet learned is how to:

- Use functions to create object instances that have unique properties, and, **shared properties with access to that particular instance's unique properties**

For example what if every instance returned from `makePerson` were to **share** a method called `haveABirthday` that incremented the value of the **unique `age` property for the instance it was called on**?

```javascript
console.log(rowan.age);             // 0
console.log(soren.age);             // 4
console.log(rowan.haveABirthday()); // This method is shared, even though it only affects the `rowan` object
console.log(rowan.age);             // 1 (updated by the call to `rowan.haveABirthday`)
console.log(soren.age);             // 4 (not affected by the call to `rowan.haveABirthday`)
```

In order to do this we must learn how to utilize the `this` mechanism, to which we will now turn our focus.

## Review

Before moving on to the next section of this repo, be sure you are able to:

- Describe the `__proto__` property on JavaScript values
- Create objects with their `__proto__` pointing to a programmer defined object
- Describe the `prototype` object on functions
- Describe the use of `__proto__` and `prototype` in built in JavaScript code
- Prove that in JavaScript, "everything is nothing"
- Use constructor functions to create object instances with both properties unique to themselves, and properties shared amongst all instances

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](building_objects_with_functions.md)
- *Shared Properties with `__proto__` and `prototype`*
- [Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties](using_this.md)
- [Subclassing](subclassing.md)
