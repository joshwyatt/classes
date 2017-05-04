# Classes

## Introduction

**In JavaScript, _Classes_ (or _Class Constructor Functions_) are functions that return objects, which are referred to as _instances_ of that class. Class constructor functions promote well organized code as well as code reuse. Class constructor functions are frequenty called with the `new` keyword which facilitates leveraging the `this` mechanism and  _prototypal inheritance_, both which support code reuse.**

It is worth noting here that in the field of Computer Science, _Class_ has a formal definition, and, there are many computer languages that use classes as they are formally defined. **JavaScript is not such a language and it does not have classes at all as they are formally defined in Computer Science. JavaScript programmers use the term out of familiarity with other languages, and because of similar behavior within their code, in spite of the fact that in JavaScript they are doing something quite different than in other languages.**

## Building Objects with Functions

### Most Simple Object Creation

Object promote well organized code. Developers use them to encapsulate thematically or functionally interrelated data and behavior.

Functions promote code reuse. Developers use them to execute large sections of code with a single function call.

Objects and functions play well together. Consider the following `makeRowan` function which creates and returns an object, representing a person named Rowan:

```javascript
function makeRowan() {
  let rowan = {
    name: 'Rowan',
    age: 0,
    curious: true

    laugh: function() {
      console.log('Ehhhhhhhhhhhhh heh heh heh');
    }
  };

  return rowan;
}
```

The `rowan` object is an encapsulation of interrelated data and behavior, and using the `makeRowan` function, a new `rowan` object can be created with a simple function call:

```javascript
const rowan1 = makeRowan();
const rowan2 = makeRowan();
const rowan3 = makeRowan();
```

### Passing Values for Dynamic Object Creation

Very frequently, when developers wish to reuse code by calling functions, they want to have the ability to provide specifics about the code when calling the function. The ability to define parameters in function definitions and pass in arguments when calling functions greatly increases the flexibility around code reuse, thereby allowing for a much larger range of possible use cases for a given function.

It is easy to imagine wishing to make objects not only for one single person, as `makeRowan` currently does, but for *any* person. In order to do this, the `makeRowan` function should be refectored such that the body of the function is more *general*, with the specifics for any particular object being passed in as arguments.

```javascript
function makePerson(name, age, laughter) {
  let person = {
    name: name,
    age: age,
    curious: true

    laugh: function() {
      console.log(laughter);
    }
  };

  return person;
}
```

Now using the `makePerson` function, a new person object can be created with a simple function call, and, by simply providing different arguments, the same function can be reused to create objects representing many different people:

```javascript
const rowan = makeRowan('Rowan', 0, 'Ehhhhhhhhhhhhh heh heh heh');
const bonnie = makeRowan('Bonnie', 34, 'Hah!');
const soren = makeRowan('Soren', 4, 'AHHHHH!!!!');
```

## Sharing Object Properties and Methods with Prototypal Inheritance

## A Naive Approach to Object Context when Using Shared Methods

## Using the `this` Mechanism to Provide Object Context when Using Shared Methods

## Using `new` to Utilize Prototypal Inheritance and the `this` Mechanism

## The `class` Keyword: Syntactic Sugar
