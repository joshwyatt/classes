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

You can imagine any other number of situations where being required to always define and then pass in an argument to use for instance specific property lookup becomes nightmarish.

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](markdown/building_objects_with_functions.md)
- [Shared Properties with `__proto__` and `prototype`](markdown/shared_properties.md)
- *Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties*
