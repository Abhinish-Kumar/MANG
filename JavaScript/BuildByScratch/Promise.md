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

### What is a Promise in General?

A **Promise** in programming is a design pattern that is used to handle asynchronous operations in a more manageable way. It represents the eventual completion (or failure) of an asynchronous operation and its resulting value. Essentially, a Promise is an object that can be in one of three states:

1. **Pending**: The initial state, indicating that the operation is still in progress.
2. **Fulfilled**: The operation completed successfully, and the Promise has a resulting value.
3. **Rejected**: The operation failed, and the Promise has a reason for the failure (usually an error).

Promises allow for cleaner handling of asynchronous code by using methods like `.then()` for success and `.catch()` for failure. This avoids callback hell and makes the code more readable and maintainable.

#### Basic Example in JavaScript:
```javascript
let myPromise = new Promise((resolve, reject) => {
  let success = true;  // Simulating success or failure
  if (success) {
    resolve("Operation was successful");
  } else {
    reject("Operation failed");
  }
});

myPromise.then((message) => {
  console.log(message); // "Operation was successful"
}).catch((error) => {
  console.log(error); // "Operation failed"
});
```

### Advantages of Promises:
- **Avoid Callback Hell**: Promises allow for chaining multiple operations in a readable way.
- **Improved Error Handling**: Errors in asynchronous code can be caught in a structured manner using `.catch()` or `.finally()`.
- **Asynchronous Code Management**: Promises provide a cleaner approach to managing asynchronous actions like network requests, file I/O, or timers.

---

### Languages that Support Promises:

1. **JavaScript**:  
   - **Promises** are native to JavaScript. The language introduced Promises in ES6, allowing developers to write asynchronous code more cleanly.
   - Example: `fetch()`, `setTimeout()`, and other asynchronous functions return Promises in JavaScript.

2. **TypeScript**:  
   - TypeScript, which is a superset of JavaScript, supports Promises in the same way as JavaScript. Since it compiles down to JavaScript, it can also use Promises natively.

3. **Python**:  
   - Python introduced **asyncio** in Python 3.5, which allows for asynchronous programming and includes a `Promise`-like concept called **coroutines** (using `async`/`await` syntax).
   - Python's `asyncio.Future()` is the equivalent of a Promise in other languages.

   Example:
   ```python
   import asyncio

   async def my_async_function():
       return "Hello, World!"

   async def main():
       result = await my_async_function()
       print(result)

   asyncio.run(main())
   ```

4. **Ruby**:  
   - Ruby has a similar concept through the **Promise** object (via libraries like `concurrent-ruby`). The `Promise` class allows managing asynchronous operations and handling success or failure.
   
   Example:
   ```ruby
   require 'concurrent-ruby'

   promise = Concurrent::Promise.new do
     "Hello, Ruby!"
   end

   promise.fulfill do |value|
     puts value # "Hello, Ruby!"
   end
   ```

5. **Java**:  
   - Java has **CompletableFuture** in Java 8, which serves as the Promise equivalent. It's used for asynchronous programming and is part of the `java.util.concurrent` package.
   
   Example:
   ```java
   import java.util.concurrent.CompletableFuture;

   public class Main {
       public static void main(String[] args) {
           CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
               return "Hello, Java!";
           });

           future.thenAccept(System.out::println);
       }
   }
   ```

6. **C#**:  
   - C# supports async programming with **Task** and **Task<T>** types, which are similar to Promises. The `async` and `await` keywords are used to handle asynchronous methods.
   
   Example:
   ```csharp
   using System;
   using System.Threading.Tasks;

   class Program {
       static async Task Main(string[] args) {
           string result = await MyAsyncFunction();
           Console.WriteLine(result);
       }

       static async Task<string> MyAsyncFunction() {
           return "Hello, C#!";
       }
   }
   ```

7. **Swift**:  
   - Swift has **async/await** syntax since Swift 5.5, which simplifies asynchronous programming. The `Task` class in Swift works similarly to Promises in JavaScript.
   
   Example:
   ```swift
   import Foundation

   func fetchData() async -> String {
       return "Hello, Swift!"
   }

   Task {
       let result = await fetchData()
       print(result)
   }
   ```

8. **Go**:  
   - Go doesn't have native Promises, but it has the concept of **goroutines** (lightweight threads) and **channels** to manage asynchronous behavior, which is somewhat similar to Promises in functionality.
   
9. **PHP**:  
   - PHP can implement Promises via third-party libraries such as **ReactPHP** or **Guzzle**, which provide Promise-like objects for asynchronous handling.
   
   Example (ReactPHP):
   ```php
   $loop = React\EventLoop\Factory::create();
   $promise = new React\Promise\Promise(function ($resolve, $reject) {
       $resolve("Hello, PHP!");
   });

   $promise->then(function ($value) {
       echo $value;
   });

   $loop->run();
   ```

10. **Kotlin**:  
    - Kotlin has **Coroutines**, which allow for asynchronous programming with a similar concept to Promises. The `suspend` and `async` keywords are used to handle async tasks.
    
    Example:
    ```kotlin
    import kotlinx.coroutines.*

    fun main() = runBlocking {
        val result = async { "Hello, Kotlin!" }
        println(result.await())
    }
    ```

---

### Conclusion:
Promises are a very useful abstraction for handling asynchronous operations in programming, and many modern programming languages have adopted similar patterns. JavaScript is where the concept of Promises first gained wide recognition, but today, many languages have adopted their own Promise-like mechanisms for managing concurrency and asynchronous tasks. Some languages like Python (via `asyncio`), Java, and Kotlin provide native support, while others like Ruby, Go, and PHP provide Promise-like abstractions or tools.
