Every value except `null` and `undefined` has a `__proto__` property.

The `__proto__` property points to an object.

If the code tries a property lookup on the value that fails, the interpreter will go looking for the property on the object that the value's own `__proto__` property points to, until either, the property is found, or, it arrives at a value without a `__proto__` property.

This fact can be used to our advantage. We can store properties (often methods) that we wish to share amongst different values in an object, and then, make sure that these different values all, either directly or eventually, attempt property lookup in this object via the objects referenced in their own, and/or in ancestor value's `__proto__` property.

---

`Object.create(someObject)` creates a new value of type `'object'` with a `__proto__` property pointing to `someObject`.

Therefore:

```javascript
const sharedProperties = {};
sharedProperties.one = 1;

let newObject = Object.create(sharedProperties);

console.log(newObject.one); // 1
```

Because property lookup continues until the JavaScript interpreter reaches an object without a `__proto__` property, the following also works:

```javascript
const sharedProperties = {};
sharedProperties.one = 1;

let anotherNewObject = Object.create(sharedProperties);

console.log(anotherNewObject.one); // 1

let yetAnotherNewObject = Object.create(anotherNewObject);

console.log(yetAnotherNewObject.one) // 1
```

---

When functions are defined, they are automatically given a special property called `prototype`, that contains a single property `constructor`, which is a reference to itself.

```javascript
function aFunction() {
  console.log('I am a function');
}

console.log(typeof aFunction.prototype);             // 'object'
console.log(typeof aFunction.prototype.constructor); // 'function'
console.log(aFunction.prototype.constructor.name);   // 'aFunction'
console.log(aFunction.prototype.constructor());      // logs 'I am a function'
```

This `prototype` property on functions, which is an object, can be given additional properties:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

console.log(aFunction.prototype.one); // 1
```

Like any other object, any value's `__proto__` property can point to one of these `prototype` objects:

```javascript
function aFunction() {
}

aFunction.prototype.one = 1;

let newObject = Object.create(aFunction.prototype);

console.log(newObject.one); // 1
```

When a function is called with the `new` keyword prefixed to it, the JavaScript interpreter silently creates a new object whose `__proto__` property points to the function's `prototype` object. We could imagine this invisible line of code looking something like the following:

```javascript
function aFunction() {
  let newObject = Object.create(aFunction.prototype); // this line is only created when calling the function with `new`
}

new aFunction();
```

Additionally, if the function does not already contain a `return` statement, calling it with the `new` keyword will also, invisibly, cause the function to return the object that was invisibly created on the first line of the function call:


```javascript
function aFunction() {
  let newObject = Object.create(aFunction.prototype); // this line is only created when calling the function with `new`
  return newObject
}

new aFunction();
```

For these reasons:

- Functions called with the `new` keyword and no explicit `return` statement return objects
- These objects each have a `__proto__` property that points to the `prototype` object that was automatically created on the function when it was defined
- When using these objects, should they not contain a property that is asked for, the JavaScript interpreter will go looking for that property in the `prototype` property of the function called with `new` that created the object to begin with.

Thus we can use functions, as creators of objects that share properties as follows:

```javascript
function makeObjectWithSharedProperties() {}

makeObjectWithSharedProperties.prototype.one = 1;
makeObjectWithSharedProperties.prototype.two = 2;

let object1 = new makeObjectWithSharedProperties();
let object2 = new makeObjectWithSharedProperties();

console.log(object1.one) // 1
console.log(object1.two) // 2
console.log(object2.one) // 1
console.log(object2.two) // 2
```
