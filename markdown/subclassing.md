# Subclassing

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

---

Here are some observations from comparing and constrasting `Person` to `Programmer` above:

1) The entirety of `Person`'s constructor function is repeated inside `Programmer`
2) The `Programmer` constructor function adds an additional property to `this` not contained in the `Person` constuctor: `canCode`
3) `Programmer.prototype` contains all the methods that `Person.prototype` does
4) `Programmer.prototype` also contains an additional method: `writeCode`
