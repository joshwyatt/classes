# ES6 Classes

## Introduction

Now that you know all the gory details of class constructor functions, including how to create instances with shared methods having access to instance specific properties, and, how to subclass these class constructor functions, it's time to show you the sweet sweet syntactic sugar magic of `this`, `new`, `class` ✨ syntax created in ES6.

## Objectives

By the end of this section you should be able to:

- Refactor legacy JavaScript class and subclass constructors to use ES6 `class` syntax

## The Current Situation

Here is the `Person` constructor function and the `Programmer` constructor function that subclasses it:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

function Programmer(name, age) {
  Person.call(this, name, age);
  this.canCode = true;
}

Programmer.prototype = Object.create(Person.prototype);
Programmer.prototype.constructor = Person;

Programmer.prototype.writeCode = function() {
  return 'code';
}

let jo = new Person('Jo', 35);
let emmaTheProgrammer = new Programmer('Emma', 25);

console.log(jo.name); // 'Jo'
console.log(emmaTheProgrammer.age); // 25

console.log(jo.greet()); // 'Hello, my name is Jo'
console.log(emmaTheProgrammer.greet());    // 'Hello, my name is Emma'
console.log(jo.greet === emmaTheProgrammer.greet); // true
console.log(jo.canCode); // undefined
console.log(emmaTheProgrammer.canCode); // true
console.log(emmaTheProgrammer.writeCode()); // 'code'
```

## A Refactor Recipe

To refactor class constructor functions to utilize the ES6 `class` syntax use the following recipe:

1) Replace the constructor function with a `class` of the same name
2) Inside the `class` create a function called `constructor` that expects the same arguments the previous constructor function did
3) Add the entirety of the old constructor function into the `class`'s new `constructor` function
4) Move every function defined on the old constructor's `prototype` to be standalone functions inside the new `class`

To refactor a class constructor that is a subclass of another class constructor:

1) Use the `extends` keyword after the class name, followed by the class it extends
2) Any time you would do `SuperClass.call(this)` just do `super()` instead


```javascript
class Person {
  constructor(name, age) {
    this.age = age;
    this.name = name;
  }

  greet() {
    return `Hello, my name is ${this.name}`;
  };

}

class Programmer extends Person {
  constructor(age, name) {
    super(age, name);
    this.canCode = true;
  }

  writeCode() {
    return 'code';
  }
}

let jo = new Person('Jo', 35);
let emmaTheProgrammer = new Programmer('Emma', 25);

console.log(jo.name); // 'Jo'
console.log(emmaTheProgrammer.age); // 25

console.log(jo.greet()); // 'Hello, my name is Jo'
console.log(emmaTheProgrammer.greet());    // 'Hello, my name is Emma'
console.log(jo.greet === emmaTheProgrammer.greet); // true
console.log(jo.canCode); // undefined
console.log(emmaTheProgrammer.canCode); // true
console.log(emmaTheProgrammer.writeCode()); // 'code'
```

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](building_objects_with_functions.md)
- [Shared Properties with `__proto__` and `prototype`](shared_properties.md)
- [Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties](using_this.md)
- [Subclassing](subclassing.md)
- *ES6 Classes*
