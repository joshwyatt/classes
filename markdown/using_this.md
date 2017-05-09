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

### Functions as Methods on the `window` and `global` Objects

Regarding the rule that **`this` refers to the object that the function was called as a method on**, there is an important, and invisible, feature of JavaScript that needs pointing out. **Standalone functions, not called as methods on a specified object, are in fact methods on either, the `window` object (if run in a browser) or the `global` object (if run in NodeJS).**

#### In the Browser

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

#### In NodeJS

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

#### A Crucial Gotcha'

Given the following two facts:

- `this` in a function defintion will be equal to the object that the function is called as a method on
- Any standalone function is in fact called on a method on either `window` or `global` (depending on whether the code is run in the browser or NodeJS)

Care must be taken not to define `this` in a helper function, or anonymous callback function within a method defintion. Consider the following:

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
```

There are 2 ways to address this common gotcha', by using **arrow function expressions**, or, by **explicitly binding function defintions**.

#### Arrow Functions Expressions Get `this` from Their Next Outer Scope

**Arrow function expressions are not given a `this` definition when called, but rather, utilize whatever `this` would be were the next outer lexical scope called.** This is incredibly useful when methods use higher order anonymous functions. By using arrow function expressions for their defintions, `this` will refer to the object housing the main method the arrow function expression is a part of, which was likely the intention to begin with. Thus, the above example could be made to behave as expected simply by refactoring the `filter` callback to be an arrow function expression:

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
```

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](markdown/building_objects_with_functions.md)
- [Shared Properties with `__proto__` and `prototype`](markdown/shared_properties.md)
- *Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties*
