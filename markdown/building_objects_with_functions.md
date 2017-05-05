# Building Objects with Functions

## Objectives

After completing this section of the repo you should be able to:

- Create objects with functions
- Pass values for dynamic object creation
- Articulate the desire to share object properties and methods

## Most Simple Object Creation

Objects promote well organized code. Developers use them to encapsulate thematically or functionally interrelated data and behavior.

Functions promote code reuse. Developers use them to execute large sections of code with a single function call.

Objects and functions play well together. Consider the following `makeRowan` function which creates and returns an object, representing a person named Rowan:

```javascript
function makeRowan() {
  let rowan = {
    name: 'Rowan',
    age: 0,
    curious: true

    sleep: function() {
      console.log('zzzzzzzzzz');
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

rowan1.name; // 'Rowan'
rowan2.age;  // 0
rowan3.sleep // logs 'zzzzzzzzzz'
```

## Passing Values for Dynamic Object Creation

Very frequently, when developers wish to reuse code by calling functions, they want to have the ability to provide specifics about the code when calling the function. The ability to define parameters in function definitions and pass in arguments when calling functions greatly increases the flexibility around code reuse, thereby allowing for a much larger range of possible use cases for a given function.

For example, it is easy to imagine wishing to make objects not only for one single person, as `makeRowan` currently does, but for *any* person. In order to do this, the `makeRowan` function should be refectored such that the body of the function is more *general*, with the specifics for any particular object being passed in as arguments to the function.

```javascript
function makePerson(name, age) {
  let person = {
    name: name,
    age: age,
    curious: true

    sleep: function() {
      console.log('zzzzzzzzzz');
    }
  };

  return person;
}
```

Now using the `makePerson` function, a new person object can be created with a simple function call, and, by simply providing different arguments, the same function can be reused to create objects representing many different people:

```javascript
let rowan = makePerson('Rowan', 0);
let julian = makePerson('Julian', 2);
let soren = makePerson('Soren', 4);

rowan.name;   // 'Rowan'
julian.age;   // 2
soren.curious // true
```

## The Desire to Share Object Properties and Methods

The object returned by `makePerson` has properties on it that can be divided into two categories:

- Properties that utilize passed in values and therefore, may be different each time `makePerson` is called
- Properties that are exactly the same every time `makePerson` is called

```javascript
function makePerson(name, age) {
  let person = {
    name: name,
    age: age,
    curious: true

    sleep: function() {
      console.log('zzzzzzzzzz');
    }
  };

  return person;
}
```

It would be potentially poor for performance (and rather inelegant) if these properties (and methods) needed to be built from scratch every time a constructor function was run, even thought the constructor is doing the work of creating the definition. Here's a *very* convoluted example to make the point:

```javascript
// This function creates a function
function makeLoggerFunction() {
  let loggerFunction = () {
    console.log('I am logger');
  }
}

// If we want to use `loggerFunction` we can make it with `makeLoggerFunction`
var loggerFunction = makeLoggerFunction();
loggerFunction(); // 'I am logger'

// What if each time we wanted to use `loggerFunction` we had to call `makeLoggerFunction` again!?
var loggerFunction = makeLoggerFunction();
loggerFunction(); // 'I am logger'

var loggerFunction = makeLoggerFunction();
loggerFunction(); // 'I am logger'
```

Obviously, programmers want to define functions as few times as is necessary to promote code reuse, and consequently, save memory. JavaScript provides a mechanism for values (including objects that are returned by functions) to share properties, to which we will now turn our focus.

## Contents

- [Introduction](../README.md)
- *Building Objects with Functions*
- [Shared Properties with `__proto__` and `prototype`](shared_properties.md)
