# The Need for Shared Properties with Access to Unique Properties

To recap, by calling functions with the `new` keyword, we can use them as class constructor functions that invisibly create and return object instances, each which will look for shared properties and methods defined on the class constructor's `prototype` object. Given that the class constructors create and return this object invisibly, how can developers define properties directly on it, for example, when needing to define properties on it **not** intended to be shared?

To make the importance of this question clear, consider the `makePerson` function from the [Building Objects with Functions](building_objects_with_functions.md) section:

```javascript
function makePerson(name, age) {
  let person = {
    name: name,
    age: age,
    curious: true

    sleep: function() {
      return 'zzzzzzzzzz';
    }
  };

  return person;
}
```

Refactoring `makePerson` to be a constructor called with `new` involves:

1) Creating the `person` object using `Object.create(makePerson.prototype)`
2) Defining properties intended to be shared amongst instances on `makePerson.prototype`
3) Defining propeties not intended to be shared on the invisible object created when calling the constructor with `new` (somehow):

```javascript
function makePerson(name, age) {
  let person = Object.create(makePerson.prototype); // 1

  person.name = name;
  person.age = age;
  person.curious = true;

  return person;
}

makePerson.prototype.sleep = function() {           // 2
  return 'zzzzzzzzzz';
};

let ro = makePerson('Ro', 0);
let soren = makePerson('Soren', 4);

ro.name;     // 'Ro'
soren.sleep; // 'zzzzzzzzzz'
```

This code works.

What if we wanted, instead of manually creating the `person` object and returning it, to use the `new` keyword? We would know how to do this if the only property we wanted on each instance was the shared method `sleep`, however, how do we  What if we wanted, instead of manually creating the `person` object and returning it, to use the `new` keyword?

## Contents

- [Introduction](../README.md)
- [Building Objects with Functions](building_objects_with_functions.md)
- [Shared Properties with `__proto__` and `prototype`](shared_properties.md)
- *The Need for Shared Properties with Access to Unique Properties*
