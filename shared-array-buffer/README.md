# Shared memory and atomics

## Using a shared lock

In the main thread, we set up shared memory so that it encodes a closed lock and send it to a worker (line A). Once the user clicks, we open the lock (line B).

```js
// main.js

// Set up the shared memory
const sharedBuffer = new SharedArrayBuffer(1 * Int32Array.BYTES_PER_ELEMENT);
const sharedArray = new Int32Array(sharedBuffer);

// Set up the lock
Lock.initialize(sharedArray, 0);
const lock = new Lock(sharedArray, 0);
lock.lock(); // writes to sharedBuffer

worker.postMessage({ sharedBuffer }); // (A)

document.getElementById("unlock").addEventListener("click", event => {
  event.preventDefault();
  lock.unlock(); // (B)
});
```

In the worker, we set up a local version of the lock (whose state is shared with the main thread via a Shared Array Buffer). In line B, we wait until the lock is unlocked. In lines A and C, we send text to the main thread, which displays it on the page for us (how it does that is not shown in the previous code fragment). That is, we are using self.postMessage() much like console.log() in these two lines.

```js
// worker.js

self.addEventListener("message", function(event) {
  const { sharedBuffer } = event.data;
  const lock = new Lock(new Int32Array(sharedBuffer), 0);

  self.postMessage("Waiting for lock..."); // (A)
  lock.lock(); // (B) blocks!
  self.postMessage("Unlocked"); // (C)
});
```

It is noteworthy that waiting for the lock in line B stops the complete worker. That is real blocking, which hasn’t existed in JavaScript until now (await in async functions is an approximation).

## Implementing a shared lock

In this section, we’ll need (among others) the following Atomics function:

- `Atomics.compareExchange(ta : TypedArray<T>, index, expectedValue, replacementValue) : T`

If the current element of `ta` at `index` is `expectedValue`, replace it with `replacementValue`. Return the previous (or unchanged) element at `index`.

The implementation starts with a few constants and the constructor:

```js
const UNLOCKED = 0;
const LOCKED_NO_WAITERS = 1;
const LOCKED_POSSIBLE_WAITERS = 2;

// Number of shared Int32 locations needed by the lock.
const NUMINTS = 1;

class Lock {
  /**
   * @param iab an Int32Array wrapping a SharedArrayBuffer
   * @param ibase an index inside iab, leaving enough room for NUMINTS
   */
  constructor(iab, ibase) {
    // OMITTED: check parameters
    this.iab = iab;
    this.ibase = ibase;
  }

  /**
   * Acquire the lock, or block until we can. Locking is not recursive:
   * you must not hold the lock when calling this.
   */
  lock() {
    const iab = this.iab;
    const stateIdx = this.ibase;
    var c;
    if (
      (c = Atomics.compareExchange(
        iab,
        stateIdx, // (A)
        UNLOCKED,
        LOCKED_NO_WAITERS
      )) !== UNLOCKED
    ) {
      do {
        if (
          c === LOCKED_POSSIBLE_WAITERS || // (B)
          Atomics.compareExchange(
            iab,
            stateIdx,
            LOCKED_NO_WAITERS,
            LOCKED_POSSIBLE_WAITERS
          ) !== UNLOCKED
        ) {
          Atomics.wait(
            iab,
            stateIdx, // (C)
            LOCKED_POSSIBLE_WAITERS,
            Number.POSITIVE_INFINITY
          );
        }
      } while (
        (c = Atomics.compareExchange(
          iab,
          stateIdx,
          UNLOCKED,
          LOCKED_POSSIBLE_WAITERS
        )) !== UNLOCKED
      );
    }
  }

  /**
   * Unlock a lock that is held.  Anyone can unlock a lock that
   * is held; nobody can unlock a lock that is not held.
   */
  unlock() {
    const iab = this.iab;
    const stateIdx = this.ibase;
    var v0 = Atomics.sub(iab, stateIdx, 1); // A

    // Wake up a waiter if there are any
    if (v0 !== LOCKED_NO_WAITERS) {
      Atomics.store(iab, stateIdx, UNLOCKED);
      Atomics.wake(iab, stateIdx, 1);
    }
  }

  // ···
}
```
