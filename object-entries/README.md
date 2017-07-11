# `Object.entries()` and `Object.values()`

## 1. `Object.entries()`

This method has the following signature:

```js
Object.entries(value : any) : Array<[string,any]>
```

If a JavaScript data structure has keys and values then an entry is a key-value pair, encoded as a 2-element Array. Object.entries(x) coerces x to an Object and returns the entries of its enumerable own string-keyed properties, in an Array:

```js
> Object.entries({ one: 1, two: 2 })
[ [ 'one', 1 ], [ 'two', 2 ] ]
```

Properties, whose keys are symbols, are ignored:

```js
> Object.entries({ [Symbol()]: 123, foo: 'abc' });
[ [ 'foo', 'abc' ] ]
```

Object.entries() finally gives us a way to iterate over the properties of an object ([read here why objects aren’t iterable by default](http://exploringjs.com/es6/ch_iteration.html#sec_plain-objects-not-iterable)):

```js
let obj = { one: 1, two: 2 };
for (let [k,v] of Object.entries(obj)) {
    console.log(`${JSON.stringify(k)}: ${JSON.stringify(v)}`);
}
// Output:
// "one": 1
// "two": 2
```

### 1.1 Setting up Maps via `Object.entries()`

Object.entries() also lets you set up a Map via an object. This is more concise than using an Array of 2-element Arrays, but keys can only be strings.

```js
let map = new Map(Object.entries({
    one: 1,
    two: 2,
}));
console.log(JSON.stringify([...map]));
    // [["one",1],["two",2]]
```

### 1.2 FAQ: `Object.entries()`

- Why is the return value of `Object.entries()` an Array and not an iterator?
The relevant precedent in this case is `Object.keys()`, not, e.g., `Map.prototype.entries()`.

- Why does `Object.entries()` only return the enumerable own string-keyed properties?
Again, this is done to be consistent with `Object.keys()`. That method also ignores properties whose keys are symbols. Eventually, there may be a method `Reflect.ownEntries()` that returns all own properties.

## 2. Object.values()

Object.values() has the following signature:

```js
Object.values(value : any) : Array<any>
```

It works much like Object.entries(), but, as its name suggests, it only returns the values of the own enumerable string-keyed properties:

```js
> Object.values({ one: 1, two: 2 })
[ 1, 2 ]
```
