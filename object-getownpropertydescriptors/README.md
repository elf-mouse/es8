# `Object.getOwnPropertyDescriptors()`

## 1. Overview

`Object.getOwnPropertyDescriptors(obj)` accepts an object obj and returns an object result:

- For each own (non-inherited) property of obj, it adds a property to result whose key is the same and whose value is the the former property’s descriptor.

Property descriptors describe the attributes of a property (its value, whether it is writable, etc.). For more information, consult Sect. “[Property Attributes and Property Descriptors](http://speakingjs.com/es5/ch17.html#property_attributes)” in “Speaking JavaScript”.

This is an example of using Object.getOwnPropertyDescriptors():

```js
const obj = {
    [Symbol('foo')]: 123,
    get bar() { return 'abc' },
};
console.log(Object.getOwnPropertyDescriptors(obj));

// Output:
// { [Symbol('foo')]:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }
```

This is how you would implement Object.getOwnPropertyDescriptors():

```js
function getOwnPropertyDescriptors(obj) {
    const result = {};
    for (let key of Reflect.ownKeys(obj)) {
        result[key] = Object.getOwnPropertyDescriptor(obj, key);
    }
    return result;
}
```

## 2. Use cases for `Object.getOwnPropertyDescriptors()`

### 2.1 Use case: copying properties into an object

Since ES6, JavaScript already has a tool method for copying properties: Object.assign(). However, this method uses simple get and set operations to copy a property whose key is key:

```js
const value = source[key]; // get
target[key] = value; // set
```

That means that it doesn’t properly copy properties with non-default attributes (getters, setters, non-writable properties, etc.). The following example illustrates this limitation. The object source has a getter whose key is foo:

```js
const source = {
    set foo(value) {
        console.log(value);
    }
};
console.log(Object.getOwnPropertyDescriptor(source, 'foo'));
// { get: undefined,
//   set: [Function: foo],
//   enumerable: true,
//   configurable: true }
```

Using Object.assign() to copy property foo to object target fails:

```js
const target1 = {};
Object.assign(target1, source);
console.log(Object.getOwnPropertyDescriptor(target1, 'foo'));
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }
```

Fortunately, using Object.getOwnPropertyDescriptors() together with [Object.defineProperties()](http://speakingjs.com/es5/ch17.html#Object.defineProperties) works:

```js
const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
console.log(Object.getOwnPropertyDescriptor(target2, 'foo'));
// { get: undefined,
//   set: [Function: foo],
//   enumerable: true,
//   configurable: true }
```

### 2.2 Use case: cloning objects

Shallow cloning is similar to copying properties, which is why Object.getOwnPropertyDescriptors() is a good choice here, too.

This time, we use [Object.create()](http://speakingjs.com/es5/ch17.html#Object.create) that has two parameters:

- The first parameter specifies the prototype of the object it returns.
- The optional second parameter is a property descriptor collection like the ones returned by Object.getOwnPropertyDescriptors().

```js
const clone = Object.create(Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj));
```

### 2.3 Use case: cross-platform object literals with arbitrary prototypes

The syntactically nicest way of using an object literal to create an object with an arbitrary prototype prot is to use the special property `__proto__`:

```js
const obj = {
    __proto__: prot,
    foo: 123,
};
```

Alas, that feature is only guaranteed to be there in browsers. The common work-around is Object.create() and assignment:

```js
const obj = Object.create(prot);
obj.foo = 123;
```

But you can also use Object.getOwnPropertyDescriptors():

```js
const obj = Object.create(
    prot,
    Object.getOwnPropertyDescriptors({
        foo: 123,
    })
);
```

Another alternative is Object.assign():

```js
const obj = Object.assign(
    Object.create(prot),
    {
        foo: 123,
    }
);
```

## 3. copying methods that use `super`

A method that uses super is firmly connected with its home object (the object it is stored in). There is currently no way to copy or move such a method to a different object.
