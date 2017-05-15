# Subclassing

## Objectives

By the time you complete this section you should be able to:

- Create a subclass constructor function with access to superclass methods
- Create subclass instance specific properties and methods

## Code Reuse with Superclasses and Subclasses

Continuing on the topic of code reuse, a common scenario is to want to create a class constructor function that will return instances very much like instances returned by an already existing class constructor with but with some additional or modified functionality. Here's a simplified of the `Person` constructor function from the last section:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};
```

If we wanted to build a class constructor to return programmers, it would look something like this:

```javascript
function Programmer(name, age) {
  this.age = age;
  this.name = name;
  this.canProgram = true;
}

Programmer.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

Programmer.prototype.writeCode = function() {
  return 'code';
};
```

Even at a glance it's easy to see that `Programmer` repeats very much of what `Person` already does, adding a couple properties and some additional functionality. Although these class constructors are small, with larger functions with many more methods, its easy to imagine wanting to not repeat ourselves. Furthermore, in addition to `Programmer` it may very well be the case that we would wish to create other class constructor functions largely based on `Person`, like `Friend`, `Chef`, `CivicPlanner` and many many more. Also, it's easy to imagine that class constructors like `Programmer`, already similar to `Person` might themselves be likely candidates for building on top of for class constructors like `WebDeveloper`, `DataEngineer`, `ProductManager` and more.

With these thoughts in mind the motivation is high to discover a way to reuse class constructor functions while adding some additional functionality. This technique is called **Subclassing** and we will now turn our attention to it.

## The Desire for Subclassing

Here are some observations from comparing and constrasting `Person` to `Programmer` above:

1) The entirety of `Person`'s constructor function is repeated inside `Programmer`
2) The `Programmer` constructor function adds an additional property to `this` not contained in the `Person` constuctor: `canCode`
3) `Programmer.prototype` contains all the methods that `Person.prototype` does
4) `Programmer.prototype` also contains an additional method: `writeCode`

Let's address each of them in turn.

## Reusing the Entirety of Another Constructor Function

Although the following won't quite work, it is a good place to start figuring how to reuse the entire `Person` constructor function inside of the `Programmer` constructor function. Because `Programmer` will be called with `new`, the "invisible" lines of code created by `new` are added here as comments, for ease of understanding:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

function Programmer(name, age) {
  // let this = Object.create(Programmer.prototype);

  Person(name, age);

  // return this;
}
```

The idea in the snippet above is that by calling `Person` **without** `new`, it will, rather than create and return a new object, simply assign the values passed into it onto the `this` that `Programmer` will create and returned when called with `new`.

The reason why the above will not work is subtle, and requires an understand of knowing what `this` will actually be set equal to `Person` is called. In order to know we must ask the question: what object is `Person` being called as a method on? In the example above, it *appears* as though it is not being called as a method on any object, which means, it is in fact being called as a method on either the `window` or `global` object (depending on whether or not we are running this code in the browser or in NodeJS).

Therefore, assuming the code is run in NodeJS:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

function Programmer(name, age) {
  // let this = Object.create(Programmer.prototype);

  Person(name, age);

  // return this;
}

let emmaTheProgrammer = new Programmer('Emma', 25);

console.log(emmaTheProgrammer.age); // undefined
console.log(global.age);            // 25
```

Fortunately, we know how to call the `Person` constructor function and force any references inside of it `this` to be equal to an object of our choosing by using `call` or `apply`. In this particular scenario, we would wish for `this` references inside of `Person` to point to the `this` inside of `Programmer` that will be invisibly created when it is called with `new`. Therefore:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

function Programmer(name, age) {
  // let this = Object.create(Programmer.prototype);

  Person.call(this, name, age);

  // return this;
}

let emmaTheProgrammer = new Programmer('Emma', 25);

console.log(emmaTheProgrammer.name); // 'Emma'
console.log(emmaTheProgrammer.age); // 25
console.log(global.age);            // undefined
```

## Adding Subclass Specific Properties

After reusing the entirety of another class constructor function, adding additional properties to the `this` of the subclass's constructor function is easy:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

function Programmer(name, age) {
  // let this = Object.create(Programmer.prototype);

  Person.call(this, name, age);
  this.canCode = true;

  // return this;
}

let emmaTheProgrammer = new Programmer('Emma', 25);

console.log(emmaTheProgrammer.name);    // 'Emma'
console.log(emmaTheProgrammer.age);     // 25
console.log(emmaTheProgrammer.canCode); // true
```

## Giving Subclass Instances Access to Superclass Shared Methods

As you well know, `Object.create` can be used to provide an object pointing to methods intended to be shared among instances. Assuming that:

- `Programmer` instances will "go looking" for shared methods on the `Programmer.prototype` object, and
- We would like `Programmer` instances to also go looking for shared methods on `Person.prototype`:

```javascript
function Person(name, age) {
  this.age = age;
  this.name = name;
}

Person.prototype.greet = function() {
  return `Hello, my name is ${this.name}`;
};

function Programmer(name, age) {
  // let this = Object.create(Programmer.prototype);

  Person.call(this, name, age);
  this.canCode = true;

  // return this;
}

Programmer.prototype = Object.create(Person.prototype);

let emmaTheProgrammer = new Programmer('Emma', 25);

console.log(emmaTheProgrammer.greet);    // 'Hello, my name is Emma'
```

Recall that a function's `prototype` property is created with a `constructor` property. Because the approach we are taking here overwrites the already existing `prototype` property on `Programmer.prototype`, we should manually recreate it:

```javascript
Programmer.prototype = Object.create(Person.prototype);
Programmer.prototype.constructor = Programmer;
```

## Adding Subclass Specific Methods

To add subclass specific methods, simply add them the the subclass's constructor function's prototype as you would with any other method intended to be shared amongst instances:

```javascript
Programmer.prototype.writeCode = function() {
  return 'code';
}
```

Thus an entire example of creating a class constructor `Person` and then, using it as a superclass to create the subclass `Programmer`:

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

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](building_objects_with_functions.md)
- [Shared Properties with `__proto__` and `prototype`](shared_properties.md)
- [Using the `this` Mechanism to Interact with Instance Specific Properties from Shared Properties](using_this.md)
- *Subclassing*
- [ES6 Classes](es6_classes.md)
