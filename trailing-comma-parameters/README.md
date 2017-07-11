# Trailing commas in function parameter lists and calls

## 1. Trailing commas in object literals and Array literals

Trailing commas are ignored in object literals:

```js
let obj = {
    first: 'Jane',
    last: 'Doe',
};
```

And they are also ignored in Array literals:

```js
let arr = [
    'red',
    'green',
    'blue',
];
console.log(arr.length); // 3
```

Why is that useful? There are two benefits.

First, rearranging items is simpler, because you don’t have to add and remove commas if the last item changes its position.

Second, it helps version control systems with tracking what actually changed. For example, going from:

```
[
    'foo'
]
```

to:

```
[
    'foo',
    'bar'
]
```

leads to both the line with 'foo' and the line with 'bar' being marked as changed, even though the only real change is the latter line being added.

## 2. Proposal: allow trailing commas in parameter definitions and function calls

Given the benefits of optional and ignored trailing commas, the proposed feature is to bring them to parameter definitions and function calls.

For example, the following function declaration causes a SyntaxError in ECMAScript 6, but would be legal with the proposal:

```js
function foo(
    param1,
    param2,
) {}
```

Similarly, the proposal would make this invocation of foo() syntactically legal:

```js
foo(
    'abc',
    'def',
);
```
