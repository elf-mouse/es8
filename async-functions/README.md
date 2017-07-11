# async functions

## 1. Writing async code via Promises and generators

For functions that compute their one-off results asynchronously, Promises, which are part of ES6, are becoming increasingly popular. One example is the client-side fetch API, which is an alternative to XMLHttpRequest for retrieving files. Using it looks as follows:

```js
function fetchJson(url) {
    return fetch(url)
    .then(request => request.text())
    .then(text => {
        return JSON.parse(text);
    })
    .catch(error => {
        console.log(`ERROR: ${error.stack}`);
    });
}
fetchJson('http://example.com/some_file.json')
.then(obj => console.log(obj));
```

co is a library that uses Promises and generators to enable a coding style that looks more synchronous, but works the same as the style used in the previous example:

```js
const fetchJson = co.wrap(function* (url) {
    try {
        let request = yield fetch(url);
        let text = yield request.text();
        return JSON.parse(text);
    }
    catch (error) {
        console.log(`ERROR: ${error.stack}`);
    }
});
```

Every time the callback (a generator function!) yields a Promise to co, the callback gets suspended. Once the Promise is settled, co resumes the callback: if the Promise was fulfilled, yield returns the fulfillment value, if it was rejected, yield throws the rejection error. Additionally, co promisifies the result returned by the callback (similarly to how then() does it).

## 2. Async functions

Async functions are basically dedicated syntax for what co does:

```js
async function fetchJson(url) {
    try {
        let request = await fetch(url);
        let text = await request.text();
        return JSON.parse(text);
    }
    catch (error) {
        console.log(`ERROR: ${error.stack}`);
    }
}
```

Internally, async functions work much like generators, but they are not translated to generator functions.

## 3. Variants

The following variants of async functions exist:

- Async function declarations: `async function foo() {}`
- Async function expressions: `const foo = async function () {};`
- Async method definitions: `let obj = { async foo() {} }`
- Async arrow functions: `const foo = async () => {};`
