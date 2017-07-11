# string padding

## 1. Padding strings

Use cases for padding strings include:

- Displaying tabular data in a monospaced font.
- Adding a count or an ID to a file name or a URL: 'file 001.txt'
- Aligning console output: 'Test 001: ✓'
- Printing hexadecimal or binary numbers that have a fixed number of digits: '0x00FF'

## 2. `String.prototype.padStart(maxLength, fillString=' ')`

This method (possibly repeatedly) prefixes the receiver with fillString, until its length is maxLength:

```js
> 'x'.padStart(5, 'ab')
'ababx'
```

If necessary, a fragment of fillString is used so that the result’s length is exactly maxLength:

```js
> 'x'.padStart(4, 'ab')
'abax'
```

If the receiver is as long as, or longer than, maxLength, it is returned unchanged:

```js
> 'abcd'.padStart(2, '#')
'abcd'
```

If maxLength and fillString.length are the same, fillString becomes a mask into which the receiver is inserted, at the end:

```js
> 'abc'.padStart(10, '0123456789')
'0123456abc'
```

If you omit fillString, a string with a single space in it is used (' '):

```js
> 'x'.padStart(3)
'  x'
```

### 2.1 A simple implementation of `padStart()`

The following implementation gives you a rough idea of how padStart() works, but isn’t completely spec-compliant (for a few edge cases).

```js
String.prototype.padStart =
function (maxLength, fillString=' ') {
    let str = String(this);
    if (str.length >= maxLength) {
        return str;
    }

    fillString = String(fillString);
    if (fillString.length === 0) {
        fillString = ' ';
    }

    let fillLen = maxLength - str.length;
    let timesToRepeat = Math.ceil(fillLen / fillString.length);
    let truncatedStringFiller = fillString
        .repeat(timesToRepeat)
        .slice(0, fillLen);
    return truncatedStringFiller + str;
};
```

## 3. `String.prototype.padEnd(maxLength, fillString=' ')`

padEnd() works similarly to padStart(), but instead of inserting the repeated fillString at the start, it inserts it at the end:

```js
> 'x'.padEnd(5, 'ab')
'xabab'
> 'x'.padEnd(4, 'ab')
'xaba'
> 'abcd'.padEnd(2, '#')
'abcd'
> 'abc'.padEnd(10, '0123456789')
'abc0123456'
> 'x'.padEnd(3)
'x  '
```

Only the last line of an implementation of padEnd() is different, compared to the implementation of padStart():

```js
return str + truncatedStringFiller;
```

## 4. FAQ: string padding

### 4.1 Why aren’t the padding methods called `padLeft` and `padRight`?

For bidirectional or right-to-left languages, the terms left and right don’t work well. Therefore, the naming of padStart and padEnd follows the existing names startsWith and endsWith.
