# Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties

## Objectives

By the time you complete this section, you should be able to:

- Articulate the need for shared methods to interact with instance specific properties

## The Need for Shared Methods to Interact with Instance Specific Properties

So far you are able to write class constructors that return objects with instance specific properties:

```javascript
function makeRowan() {
  return {
    name: 'Rowan'
  };
}

let rowan = makeRowan();
console.log(rowan.name);   // 'Rowan'
```

And you are able to write class constructors that return objects with instance specific properties based on passed in values:

```javascript
function makePerson(name) {
  return {
    name: name
  };
}

let rowan = makePerson('Rowan');
let soren = makePerson('Soren');

console.log(rowan.name); // 'Rowan'
console.log(soren.name); // 'Soren'
```

Additionally you know how to store properties and methods that are shared among all instances with `Object.create`:

```javascript
let sharedMethods = {
  sleep: function() {
    return 'zzz';
  }
};

function makePerson(name) {
  let person = Object.create(sharedMethods);
  person.name = name;
  return person;
}

let rowan = makePerson('Rowan');
let soren = makePerson('Soren');

console.log(rowan.sleep());               // 'zzz'
console.log(soren.sleep());               // 'zzz'
console.log(rowan.sleep === soren.sleep); // true
```

Additionally you know how to utilize the constructor function's `prototype` object for housing shared properties:

```javascript
function makePerson(name) {
  let person = Object.create(makePerson.prototype);
  person.name = name;
  return person;
}

makePerson.prototype.sleep = function() {
  return 'zzz';
}

let rowan = makePerson('Rowan');
let soren = makePerson('Soren');

console.log(rowan.sleep());               // 'zzz'
console.log(soren.sleep());               // 'zzz'
console.log(rowan.sleep === soren.sleep); // true
```

What you do not yet know how to do is to make shared properties that are able to interact with instance specific properties. Consider the following `makeGreeting` function that intends to utilize the **shared** `makeGreeting` function but return a value that utilizes the **instance's specific** `name` property:

```javascript
function makePerson(name) {
  let person = Object.create(makePerson.prototype);
  person.name = name;
  return person;
}

makePerson.prototype.sleep = function() {
  return 'zzz';
}

makePerson.prototype.makeGreeting = function() {
  return 'Hello my name is ???';
}

let rowan = makePerson('Rowan');
let soren = makePerson('Soren');

console.log(rowan.makeGreeting === soren.makeGreeting); // true
console.log(rowan.makeGreeting()); // should equal 'Hello my name is Rowan'
console.log(soren.makeGreeting()); // should equal 'Hello my name is Soren'
```

## Explicitly Passing in a Context Object Argument

One way to solve this issue would be to define a parameter on the `makeGreeting` method so that we could pass in a specific instance with each call:

```javascript
function makePerson(name) {
  let person = Object.create(makePerson.prototype);
  person.name = name;
  return person;
}

makePerson.prototype.sleep = function() {
  return 'zzz';
}

makePerson.prototype.makeGreeting = function(instanceWithNamePropertyToUse) {
  const name = instanceWithNamePropertyToUse.name;
  return `Hello my name is ${name}`;
}

let rowan = makePerson('Rowan');
let soren = makePerson('Soren');

console.log(rowan.makeGreeting === soren.makeGreeting); // true
console.log(rowan.makeGreeting(rowan)); // 'Hello my name is Rowan'
console.log(soren.makeGreeting(soren)); // 'Hello my name is Soren'
```

As a matter of fact, this technique works perfectly fine. The main issue with it is aesthetic. To use this technique would require:

- Defining a parameter akin to `instanceWithNamePropertyToUse` above on every shared method ever created that wished to interact with instance specific properties
- Passing in the instance name (along with any other arguments) explicitly, even though the name of the instance is already visibly present as the object calling the method in the first place

This doesn't seem so bad in the example above, but what if in built in JavaScript code, where shared methods are able to operate on specific instances, we had to do the same thing?

```javascript
// We would have to do...
'rowan'.toUpperCase('rowan');
// ...instead of
'rowan'.toUpperCase();
```

```javascript
// We would have to do...
let a = [1, 2, 3];
a.map(a, (element => element * 2));

// ...instead of
[1, 2, 3].map(element => element * 2);
```

```javascript
// We would have to do...
let a = [1, 2, 3];
let b = a.map(a, (element => element * 2));
b.reduce(b, (a, b) => a + b);

// ...instead of
[1, 2, 3]
  .map(element => element * 2)
  .reduce((a, b) => a + b)
```

```javascript
// We would have to do...
$('#submit').on($('#submit'), 'click', (e) => {console.log('clicked')});
// ...instead of
$('#submit').on('click', (e) => {console.log('clicked')});
```

You can imagine any other number of situations where being required to **explicitly** pass in an instance for property lookup within the shared method becomes nightmarish. Fortunately, JavaScript provides us a much more elegant way: the `this` mechanism.

## The `this` Mechanism

**`this` is used inside of function definitions to refer to some object. Just like with parameters in function definitions, we cannot know the actual value of `this` until the function is called.**

Consider the function defintion `add` below which defines 2 parameters:

```javascript
function add(num1, num2) {
  return num1 + num2;
}
```

Although we can tell what will be done with the values represented by `num1` and `num2` by reading the function definition, they will not be assigned **actual** values until `add` is called and passed arguments. In order to determine what a parameter's actual value is, we simply look for the argument passed in at the same position that the parameter was defined:

```javascript
function add(num1, num2) {
  return num1 + num2;
}

add(1, 2); // `num1` is `1` and `num2` is `2`
add(10, 20); // `num1` is `10` and `num2` is `20`
```

Similarly, function definitions can refer to an object called `this` and we cannot know what its **actual** value is until the function is called. The rules by which we understand the **actual** value that `this` refers to are more complicated than for function parameters. Here is a summary of how to know what object `this` refers to. We will deal with each of them in order:

- `this` refers to the object that the function was called as a method on...
- ...unless the function is **bound** with either `call(object)`, `apply(object)`, or `bind(object)`, in which case `this` refers to whatever was passed in as `object`.
- Superseding the other rules, if the function is called with the keyword `new`, `this` refers to the "invisible" object created, and potentially returned by calling the function with `new`

## `this` Referring to the Object that a Function was Called as a Method On

By far the most common use for the `this` mechanism is to use it in shared methods to refer to a specific object instance. In the following example, the shared method `sayName` uses `this` so that any instance can call the method with the method having access to that instances `name` property:

```javascript
function createPerson(name) {
  let newPerson = Object.create(createPerson.prototype);
  newPerson.name = name;
  return newPerson;
}

createPerson.prototype.sayName = function() {
  let name = this.name;
  return `Hello, my name is ${name}`;
}

let rowan = createPerson('Rowan');
let soren = createPerson('Soren');

console.log(rowan.sayName()); // 'Rowan'
console.log(soren.sayName()); // 'Soren'
```

Because `this` stands in for an instance object, it can be used to add properties on the instance, update properties on the instance, and delete properties on the instance:

```javascript
function createPerson(name) {
  let newPerson = Object.create(createPerson.prototype);
  newPerson.name = name;
  return newPerson;
}

createPerson.prototype.addCharacteristic = function(characteristic, initialValue) {
  this[characteristic] = initialValue;
}

createPerson.prototype.updateCharacteristic = function(characteristic, newValue) {
  this[characteristic] = newValue;
}

createPerson.prototype.removeCharacteristic = function(characteristic) {
  delete this[characteristic];
}

let rowan = createPerson('Rowan');
let soren = createPerson('Soren');

console.log(rowan.diaperSize);               // undefined
rowan.addCharacteristic('diaperSize', 3);
console.log(rowan.diaperSize);               // 3
rowan.updateCharacteristic('diaperSize', 4);
console.log(rowan.diaperSize);               // 4
rowan.removeCharacteristic('diaperSize');
console.log(rowan.diaperSize);               // undefined
```

## Functions as Methods on the `window` and `global` Objects

Regarding the rule that **`this` refers to the object that the function was called as a method on**, there is an important, and invisible, feature of JavaScript that needs pointing out. **Standalone functions, not called as methods on a specified object, are in fact called on  methods on either, the `window` object (if run in a browser) or the `global` object (if run in NodeJS).**

### In the Browser

Consider the following code, assumed to be running in a browser:

```javascript
console.log(typeof window);  // 'object'

window.num = 1;
console.log(window.num)      // 1

function getWindowNum() {
  return this.num;
}
console.log(getWindowNum()); // 1
```

### In NodeJS

Consider the following code, assumed to be running in NodeJS:

```javascript
console.log(typeof global); // 'object'

global.num = 1;
console.log(global.num) // 1

function getGlobalNum() {
  return this.num;
}
console.log(getGlobalNum()); // 1
```

### A Crucial Gotcha'

Given the following two facts:

- `this` in a function defintion will be equal to the object that the function is called as a method on
- Any standalone function is in fact called as a method on either `window` or `global` (depending on whether the code is run in the browser or NodeJS)

care must be taken not to define `this` in a standalon helper function, or callback function within a method defintion. Consider the following:

```javascript
function makePerson(friends, favoriteLetter) {
  let newPerson = Object.create(makePerson.prototype);
  newPerson.friends = friends;
  newPerson.favoriteLetter = favoriteLetter;
  return newPerson;
}

makePerson.prototype.getFriendsStartingWithFavoriteLetter = function() {
  return this.friends.filter(function(friend) {
    return friend.startsWith(this.favoriteLetter);
  });
};

let friendsOfBonnie = ['Josh', 'Aron', 'Svea', 'Joe', 'Amrit'];
let bonnie = makePerson(friendsOfBonnie, 'A');

let favoriteLetterFriends = bonnie.getFriendsStartingWithFavoriteLetter();

console.log(favoriteLetterFriends); // `[]` and NOT `['Aron', 'Amrit']`
```

There are 2 ways to address this common gotcha', by using **arrow function expressions**, or, by **explicitly binding function defintions**.

## Arrow Functions Expressions Don't Get their own `this` When Called

**Unlike all other JavaScript functions, Arrow function expressions are not given a `this` definition when called. Therefore, should their defintions contain reference to `this`, the interpreter will go looking in the next out lexical scope, as it would with any undefined variable.**

Here's a simple example of utilizing a variable defined in an outer lexical scope:

```javascript
let name = 'rowan';

function printName() {
  console.log(name);
}

printName();
```

The fact that arrow function expressions don't create their own `this` is incredibly useful when instance methods use higher order anonymous functions. By using arrow function expressions for their defintions, `this` will refer to the object housing the main method that the arrow function expression is a part of, which was likely the intention to begin with. Thus, the above `favoriteLetterFriends` example could be made to behave as expected simply by refactoring the anonymous `filter` callback function to be an arrow function expression:

```javascript
function makePerson(friends, favoriteLetter) {
  let newPerson = Object.create(makePerson.prototype);
  newPerson.friends = friends;
  newPerson.favoriteLetter = favoriteLetter;
  return newPerson;
}

makePerson.prototype.getFriendsStartingWithFavoriteLetter = function() {
  return this.friends.filter(friend => friend.startsWith(this.favoriteLetter))
};

let friendsOfBonnie = ['Josh', 'Aron', 'Svea', 'Joe', 'Amrit'];
let bonnie = makePerson(friendsOfBonnie, 'A');

let favoriteLetterFriends = bonnie.getFriendsStartingWithFavoriteLetter();
console.log(favoriteLetterFriends); // ['Aron', 'Amrit']
```

## Using `bind` to Control What `this` Will Be When a Function is Called

**All JavaScript function defintions have available on them a `bind` method that takes an object as its only argument. Calling the `bind` method on a JavaScript function definition will return a new function with the same defintion where any references to `this` will point to the object passed into `bind` when the function is called**.

```javascript
let rowan = {
  name: 'Rowan'
};

let soren = {
  name: 'Soren'
};

function sayName() {
  console.log(this.name);
};

let sayRowan = sayName.bind(rowan);
sayRowan(); // 'Rowan'

let saySoren = sayName.bind(soren);
saySoren(); // 'Soren'
```

Therefore an alternative solution to the `favoriteLetterFriends` function is to call `bind` on the anonymous callback function definition, thereby returning a bound function to be passed in as the argument to `filter`:

```javascript
function makePerson(friends, favoriteLetter) {
  let newPerson = Object.create(makePerson.prototype);
  newPerson.friends = friends;
  newPerson.favoriteLetter = favoriteLetter;
  return newPerson;
}

makePerson.prototype.getFriendsStartingWithFavoriteLetter = function() {
  return this.friends.filter(function(friend) {
    return friend.startsWith(this.favoriteLetter);
  }.bind(this));
};

let friendsOfBonnie = ['Josh', 'Aron', 'Svea', 'Joe', 'Amrit'];
let bonnie = makePerson(friendsOfBonnie, 'A');

let favoriteLetterFriends = bonnie.getFriendsStartingWithFavoriteLetter();

console.log(favoriteLetterFriends); // ['Aron', 'Amrit']`
```

## Explicitly Stating a `this` Binding with `call` or `apply`

**Rather than calling a function definition directly by adding `()` to it, it can be called with either its `call` method or `apply` method, each which take a first argument specifying what any reference to `this` in the function definition should point to.**

```javascript
let rowan = {
  name: 'Rowan'
};

let soren = {
  name: 'Soren'
};

function sayName() {
  console.log(this.name);
};

sayName.call(rowan); // 'Rowan'

sayName.call(soren); // 'Soren'
```

If the function expects arguments, these are passed in after the first `this` binding argument. When using `call`, the additional arguments are passed in as comma separated arguments, when using `apply` they are passed in as a single array:

```javascript
function greet(salutation, punctuation) {
  console.log(`${salutation}, my name is ${this.name} and I am ${this.age} years old${punctuation}`);
}

let rowan = {
  name: 'Rowan',
  age: 0
};

let soren = {
  name: 'Soren',
  age: 4
};

greet.call(rowan, 'Howdy', '?'); // 'Howdy, my name is Rowan and I am 0 years old?'
greet.apply(soren, ['Hi', '!']); // 'Hi, my name is Soren and I am 4 years old!'
```

`call` and `apply` are useful when you need to create helper or utility functions to operate on class instances that you would not wish to define on within the class constructor itself, or are unable to do so. For example, writing a throw away debugging function to inspect the behavior of a class instance, or, writing a function to interact with an instance of a class that you do not have access to because it was defined in a library you are using.

## The Keyword `new`

**Calling a function with the `new` keyword will cause the function to "invisibly":**

- **Create a new object whose `__proto__` points to the function's `prototype` property**
- **Bind all uses of `this` in the function definition to this newly created "invisible" object**
- **Return this newly created object if the function does not already have a `return` statement**

Creating class constructor functions intended to be called with the keyword `new` make writing class constructor's more terse and elegant than otherwise. At the same time, understanding the implications of using it require a lot of knowledge about a lot of the inner workings of JavaScript. Because calling a function with `new` is so much different than calling it without, function definitions intended to be called with the `new` keyword are, by convention, capitalized:

```javascript
function Person(name, age) {
  // Imagine the next line being inserted "invisibly" when and if this function is called with `new`
  // this = Object.create(Person.prototype);

  this.name = name;
  this.age = age;

  // Imagine the next line being inserted "invisibly" when and if this function is called with `new`
  // return this;
}

Person.prototype.greet = function(salutation, punctuation) {
  console.log(`${salutation}, my name is ${this.name} and I am ${this.age} years old${punctuation}`);
}

let rowan = new Person('Rowan', 0);
let soren = new Person('Soren', 4);

rowan.greet('Howdy', '?'); // 'Howdy, my name is Rowan and I am 0 years old?'
soren.greet('Hi', '!');    // 'Hi, my name is Soren and I am 4 years old!'
```

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](markdown/building_objects_with_functions.md)
- [Shared Properties with `__proto__` and `prototype`](markdown/shared_properties.md)
- *Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties*
