https://www.promisejs.org/implementing/


Since a promise is just a state machine

```javascript
// Promise states
const PENDING = 0;
const FULFILLED = 1;
const REJECTED = 2;

class MyPromise {
  constructor(executor) {
    this.state = PENDING;
    this.value = null;
    this.handlers = [];
    this.isHandled = false; // Flag to ensure we don't call handlers multiple times

    // Try to resolve/reject the promise
    const resolve = (result) => this.resolve(result);
    const reject = (error) => this.reject(error);

    try {
      executor(resolve, reject); // Call the executor to start promise resolution
    } catch (e) {
      reject(e);
    }
  }

  // Resolving the promise
  resolve(result) {
    if (this.state !== PENDING) return; // Once resolved or rejected, no changes
    if (result && typeof result.then === 'function') {
      // If the result is a thenable, use that promise's resolution
      return result.then(this.resolve.bind(this), this.reject.bind(this));
    }

    this.state = FULFILLED;
    this.value = result;
    this.executeHandlers();
  }

  // Rejecting the promise
  reject(error) {
    if (this.state !== PENDING) return;
    this.state = REJECTED;
    this.value = error;
    this.executeHandlers();
  }

  // Execute all the handlers that were added with .then or .done
  executeHandlers() {
    if (this.state === PENDING || this.isHandled) return;
    this.isHandled = true;
    setTimeout(() => {
      this.handlers.forEach(handler => this.handle(handler));
      this.handlers = [];
    }, 0); // Always execute handlers asynchronously
  }

  // Add handlers for promise fulfillment or rejection
  handle(handler) {
    if (this.state === PENDING) {
      this.handlers.push(handler);
    } else if (this.state === FULFILLED && typeof handler.onFulfilled === 'function') {
      handler.onFulfilled(this.value);
    } else if (this.state === REJECTED && typeof handler.onRejected === 'function') {
      handler.onRejected(this.value);
    }
  }

  // Attach fulfillment and rejection handlers and return a new promise
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      this.handle({
        onFulfilled: (value) => {
          if (typeof onFulfilled === 'function') {
            try {
              return resolve(onFulfilled(value));
            } catch (ex) {
              return reject(ex);
            }
          } else {
            return resolve(value);
          }
        },
        onRejected: (error) => {
          if (typeof onRejected === 'function') {
            try {
              return resolve(onRejected(error));
            } catch (ex) {
              return reject(ex);
            }
          } else {
            return reject(error);
          }
        }
      });
    });
  }

  // Done is similar to .then but doesn't return a new promise
  done(onFulfilled, onRejected) {
    this.handle({
      onFulfilled,
      onRejected
    });
  }
}

// Helper function to easily check for the existence of a `then` method (for handling promises)
function getThen(value) {
  if (value && typeof value === 'object' || typeof value === 'function') {
    const then = value.then;
    if (typeof then === 'function') return then;
  }
  return null;
}

```

```javascript
// Using the custom Promise
const myPromise = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('Hello, world!');
  }, 1000);
});

myPromise.then((result) => {
  console.log(result); // "Hello, world!"
}).catch((error) => {
  console.log('Error:', error);
});

// Handling rejection
const failingPromise = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    reject('Something went wrong!');
  }, 500);
});

failingPromise.then(null, (error) => {
  console.log('Caught error:', error); // "Caught error: Something went wrong!"
});
```
