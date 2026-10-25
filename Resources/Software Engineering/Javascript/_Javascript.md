# Javascript

Noting down some concepts from Javascript that operate a bit differently in this language (as compared to Java and C++)

# Implicit Closures

Functions in Javascript can “persist” some values in their scope, capturing the state of the environment from the time that they are initialised.

For example, you can do this in Javascript

```jsx
function multiplier(factor) {
  return multiplierFunction = function (number) {
    return number * factor;
  };
}

let twice = multiplier(2);
let thrice = multiplier(3);
console.log(twice(5));
console.log(thrice(5));
```

Here is something similar done in Java. AFAIR there is no way of doing closures in Java without using functional interfaces.

```jsx
@FunctionalInterface
interface Multiplier {
    int call(int number);
}

public class MyClass {

    /**
     * The Factory method, now returning a Lambda Expression.
     * The lambda closes over (captures) the 'factor' variable.
     */
    public static Multiplier createMultiplier(int factor) {
        // This Lambda Expression implements the Multiplier interface
        return (number) -> factor * number;
    }

    public static void main(String[] args) {
        // Creating Concrete implementation of Functional Interface using anonymous inner class
        Multiplier twice = new Multiplier() {
            public int call(int number) {
                return number*1;
            }
        };
        
        // Creating Concrete implementation of Functional Interface using lambda
        Multiplier thrice = createMultiplier(3);
			
        System.out.println(twice.call(5));
        System.out.println(thrice.call(5));
    }
}
```

# Bind

**Closure** is about *variables* captured from the surrounding lexical scope at definition time. It's static -- determined by where the function is written in the code:

**this** is a separate binding that is determined *dynamically* at call time, not at definition time. It's not part of the closure -- it's more like a hidden parameter that gets passed to the function based on how you invoke it:

.bind() can be used to set the this object of any function.

```jsx
function sayHi() {
    console.log(this.name);
}

const obj = { name: 'Alice' };
const bound = sayHi.bind(obj);
bound(); // "Alice"
```

# Arrow Functions

 where they are called. This is different from typical lambda functions, which only have access to the scope where they are declared.

For example, this will work

```jsx
let finder = {
  find(array) {
    return array.some(v => v == this.value);
  },
  value: 5
};
console.log(finder.find([4, 5]));

// OUTPUT
// → true
```

But this will not work, since findLambda will not have access to this (it was declared outside the scope where this was available).

```jsx
let findLambda = function(v) {
  return v == this.value;
}

let finder = {
  find(array) {
    return array.some(findLambda);
  },
  value: 5
};
console.log(finder.find([4, 5]));

// OUTPUT
// → false
// if you console.log(this.value) in findLambda you'll get undefined
```

# Prototypes

This used to be the only way to make classes in javascript. Now it’s used as a replacement for abstract classes.

```jsx
let protoRabbit = {
  speak(line) {
    console.log(`The ${this.toString()} says '${line}'`);
  },
  toString() {
    return this.type + " rabbit";
  }
};
let blackRabbit = Object.create(protoRabbit);
blackRabbit.type = "hungry black";
blackRabbit.speak("I am fear and darkness");
console.log(blackRabbit.toString());
```

## Prototype vs. Inheritance

In modern JavaScript, these two concepts are two sides of the same coin. **Inheritance** is the syntax we write, while **Prototypes** are the mechanism the browser actually uses.

- **The Bottom Line:** In 99% of modern development, you will use **Inheritance**. You only need to understand **Prototypes** to understand how JavaScript manages to share that code without duplicating it in memory for every single whale you create. One is not more efficient than the other
    
    ### 1. Inheritance (The "What")
    
    Inheritance is a high-level design pattern. It defines a formal **"is-a"** relationship using the `class` and `extends` keywords.
    
    - **Syntax:** Uses `class Whale extends Mammal`.
    - **Focus:** Organization and readability.
    - **Analogy:** You are looking at the **blueprints**. The Whale blueprint specifically says "start with the Mammal blueprint, then add fins."
    
    ```
    class Mammal {
      breathe() { console.log("Taking a breath..."); }
    }
    
    class Whale extends Mammal {
      swim() { console.log("Splashing in the ocean!"); }
    }
    
    const myWhale = new Whale();
    myWhale.breathe(); // Works because of inheritance
    
    ```
    
    ### 2. Prototypes (The "How")
    
    Prototypes are the physical "chain" that connects objects in memory. When you call a method on an object, JavaScript searches the object itself. If it’s not there, it follows a link (the `__proto__` property) to the prototype object.
    
    - **Syntax:** Uses `Mammal.prototype.breathe = ...`.
    - **Focus:** Memory efficiency and engine behavior.
    - **Analogy:** You are looking at the **ancestry**. The Whale doesn't actually have a "breathe" lung inside it; instead, it has a "phone line" to its ancestor (Mammal). When it needs to breathe, it calls the ancestor and asks how to do it.
    
    ```
    function Mammal() {}
    Mammal.prototype.breathe = function() { console.log("Taking a breath..."); };
    
    function Whale() {}
    // Linking the ancestor chain manually
    Object.setPrototypeOf(Whale.prototype, Mammal.prototype);
    
    const myWhale = new Whale();
    myWhale.breathe(); // Works because it followed the "phone line" to Mammal
    
    ```
    
    ### Comparison Summary
    
    | Feature | Inheritance (Class Syntax) | Prototypes (Underlying Mechanism) |
    | --- | --- | --- |
    | **Visibility** | Explicit and easy to read. | Usually hidden "under the hood." |
    | **Logic** | Defines a structure *before* creation. | Connects objects *during* runtime. |
    | **Modernity** | Standard for ES6+ and LWC. | The "legacy" way to link objects. |
    | **Key Keyword** | `extends` | `prototype` / `__proto__` |

Inheritance works much the same as other languages

```jsx
class Animal {
    constructor(name) {
        this.name = name;
    }

    makeSound() {
        console.log(`${this.name} makes a generic animal sound.`);
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        // super() MUST be called before using 'this' in a child constructor.
        super(name); 
        this.breed = breed;
    }

    makeSound() {
        console.log(`${this.name}, the ${this.breed}, barks!`);
    }
}

const genericAnimal = new Animal('Unknown Creature');
genericAnimal.makeSound(); 

const myDog = new Dog('Rex', 'Golden Retriever');
myDog.makeSound(); 
```

# Variable Naming

| **Prefix** | **Meaning** | **Type** |
| --- | --- | --- |
| `_name` | **Internal/Private** | Convention (Soft privacy) |
| `#name` | **Private** | Language Feature (Hard privacy) |
| `name` | **Public** | Default |

# Access Modifiers

Private properties in objects are defined by beginning their names with a `#` sign

```jsx
class RandomSource {
  #max;
  constructor(max) {
    this.#max = max;
  }
  getNumber() {
    return Math.floor(Math.random() * this.#max);
  }
}
```

# Getters

Getters appear to be static properties (of their native type) of objects, but are actually values that are fetched dynamically at runtime by calling a function. Here is an example

```jsx
let varyingSize = {
  get size() {
    return Math.floor(Math.random() * 100);
  }
};

console.log(varyingSize.size);
// → 73
console.log(varyingSize.size);
// → 49
console.log(typeof varyingSize.size);
// → number
```

As typeof revealed, size actually appears to be just a number.

# Setters

The same concept as getters, but applied to functions that are used to set values

```jsx
class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }
  get fahrenheit() {
    return this.celsius * 1.8 + 32;
  }
  set fahrenheit(value) {
    this.celsius = (value - 32) / 1.8;
  }

  static fromFahrenheit(value) {
    return new Temperature((value - 32) / 1.8);
  }
}

let temp = new Temperature(22);
console.log(temp.fahrenheit);
// → 71.6
temp.fahrenheit = 86;
console.log(temp.celsius);
// → 30
```

# for / of Loops

There is a simpler way to write such loops in modern JavaScript:

```jsx
for (let entry of JOURNAL) {
  console.log(`${entry.events.length} events.`);
}
```

When a `for` loop uses the word `of` after its variable definition, it will loop over the elements of the value given after `of`. This works not only for arrays but also for strings and some other data structures.

# Mixin

A *mixin* provides methods that implement a certain behaviour, and we use it to add the behaviour to other classes.

```jsx
// mixin
let sayHiMixin = {
  sayHi() {
    alert(`Hello ${this.name}`);
  },
  sayBye() {
    alert(`Bye ${this.name}`);
  }
};

// usage:
class User {
  constructor(name) {
    this.name = name;
  }
}

// THIS IS HOW YOU ADD A MIZIN
Object.assign(User.prototype, sayHiMixin);

// now User can say hi
new User("Dude").sayHi(); // Hello Dude!
```

# Strict Equality Comparison

In JavaScript, the **`===` operator is the strict equality comparison operator**, which **checks if two values are equal in both value and data type without performing any type conversion (type coercion)**. It is generally the recommended operator for most use cases to avoid unexpected behaviour. 

# Symbols

Most properties are named with strings. But it is also possible to use symbols as property names. Symbols are values created with the `Symbol` function. 

Unlike strings, newly created symbols are unique—you cannot create the same symbol twice.

```jsx
let sym = Symbol("name");
console.log(sym == Symbol("name"));
// → false
Rabbit.prototype[sym] = 55;
console.log(killerRabbit[sym]);
// → 55
```

The string you pass to `Symbol` is included when you convert it to a string and can make it easier to recognize a symbol when, for example, showing it in the console. But it has no meaning beyond that—multiple symbols may have the same name.

The point of symbols is to live alongside existing properties of the same label without any conflicts. Being both unique and usable as property names makes symbols suitable for defining interfaces that can peacefully live alongside other properties, no matter what their names are.

```jsx
const length = Symbol("length");
Array.prototype[length] = 0;

console.log([1, 2].length);
// → 2
console.log([1, 2][length]);
// → 0
```

# Iterator

The object given to a `for`/`of` loop is expected to be *iterable*. This means it has a method named with the `Symbol.iterator` symbol (a symbol value defined by the language, stored as a property of the `Symbol` function).

(The `Symbol` function is also where I assume all other language standard symbols reside)

When called, that method should return an object that provides a second interface, *iterator*. This is the actual thing that iterates. It has a `next` method that returns the next result. That result should be an object with a `value` property that provides the next value, if there is one, and a `done` property, which should be true when there are no more results and false otherwise.

Note that the `next`, `value`, and `done` property names are plain strings, not symbols. Only `Symbol.iterator`, which is likely to be added to a *lot* of different objects, is an actual symbol.

```jsx
let okIterator = "OK"[Symbol.iterator]();
console.log(okIterator.next());
// → {value: "O", done: false}
console.log(okIterator.next());
// → {value: "K", done: false}
console.log(okIterator.next());
// → {value: undefined, done: true}
```

Note the `()` after `"OK"[Symbol.iterator]` which has to be there because the String `“OK”` has a method named `Symbol.iterator` that fetches the actual iterator object for us.

# Asynchronous Programming

![image.png](Javascript/image.png)

The best way to do async programming in javascript is using Promises. 

## Promise

Promise has a somewhat odd interface: the constructor expects a function as its argument, which it immediately calls, passing it a function that it can use to resolve the promise.

Watch these videos about

Promises

https://www.youtube.com/watch?v=Xs1EMmBLpn4

Event Loop

https://www.youtube.com/watch?v=eiC58R16hb8

For example, this is how you could create a promise-based interface for the `readTextFile` function:

```jsx
function textFile(filename) {
  return new Promise((reject, resolve) => {
    readTextFile(filename, text => resolveFunction(text));
  });
}

function mergeLines(sentences) {
  let result = "";
  for(let sentence of sentences)
    result += sentence;
  return result;
}

textFile("plans.txt")
.then(content => content.trim().split("\n"))
.then(sentences => mergeLines(sentences))
.then(text => console.log(text));
```

Promises can be either resolved (the action finished successfully) or rejected (it failed). Resolve handlers (as registered with `then`) are called only when the action is successful, and rejections are propagated to the new promise returned by `then`. When a handler throws an exception, this automatically causes the promise produced by its `then` call to be rejected. If any element in a chain of asynchronous actions fails, the outcome of the whole chain is marked as rejected, and no success handlers are called beyond the point where it failed.

### **Promise Constructor**

- A new `Promise` object is created in memory when the constructor is executed. This object contains internal slots like:
    - `PromiseState` : `fulfilled` or `rejected`
    - `PromiseResult` : Object that promise returned
    - `PromiseFulfillReactions` : This is the handler that gets set when we do a .then(lambda) call. Gets executed when promise `fulfulls`.
    - `PromiseRejectReactions` : This is the handler that gets set when we do a .catch(lambda) call. Gets executed when promise `rejects`.
    - `PromiseIsHandled` : True or False
- You can `resolve` a promise by calling `resolve`, which sets `PromiseState` to `fulfilled` and `PromiseResult` to the provided value.
javascript
- You can `reject` a promise by calling `reject`, which sets `PromiseState` to `rejected` and `PromiseResult` to the provided value.

```jsx
// Note that this snippet doesn't really make sense per se
// because ideally whatever async operation you're calling will
// rely in callbacks to be asynchronous. 
// These callbacks will register handlers for resolve and reject
// This snippet is purely to illustrate how resolve and reject behave
// and how then anc catch are used.

// Two better, working, examples are given below

const myPromise = new Promise((resolve, reject) => {
	// Asynchronous operation (e.g., fetching data) that set `success`
	if (success) {
		resolve("Data successfully fetched!"); // PromiseState: fulfilled, PromiseResult: "Data successfully fetched!"
	} else {
		reject("Failed to fetch data."); // PromiseState: rejected, PromiseResult: "Failed to fetch data."
	}
});

myPromise
.then(message => {
	console.log(message); // Output: Data successfully fetched!
})
.catch(error => {
	console.error(error); // Output: Failed to fetch data.
});
```

### **Promise Reaction Records and Asynchronous Behaviour**

- The `PromiseFulfillReactions` and `PromiseRejectReactions` fields contain **Promise Reaction Records**.
- These records are created by chaining a `.then()` or `.catch()` method to the promise.
- A reaction record contains a `Handler`, which is the callback function passed to `.then()` or `.catch()` .
- When a promise resolves, the `Handler` is added to the **microtask queue**.
- Promises enable **non-blocking asynchronous operations**, keeping your script interactive while background tasks are handled.

### **Chaining `.then()` Methods**

- The `.then()` method itself returns a new promise, allowing you to chain multiple `.then()` calls.

### Test Your Undestanding

```jsx
	
new Promise(resolve => {
	console.log(1); // Synchronous, executes first
	resolve(2); // Schedules the .then() handler to the microtask queue
}).then(result => {
	console.log(result); // This is in the microtask queue, executes last
});

console.log(3); // Synchronous, executes after console.log(1) but before the microtask queue handler

// Expected output:
// 1
// 3
// 2
```

This order is due to the promise resolving and its handler being scheduled to the microtask queue, while the console.log(3) statement is executed synchronously before the microtask queue is checked.

## Using Promises

### When Provided

Most often, Web API functions will provide you promises that you can choose to either await or chain with then() handlers.

```jsx
console.log("1. Script starts - UI is responsive.");

// We call the 'fetch' Web API. 
// JS executes this synchronously, but the BROWSER takes over the actual network request.
const userPromise = fetch('https://jsonplaceholder.typicode.com/users/1');

// We set up our reactions for WHEN the browser finishes the background work.
userPromise
    .then(response => response.json()) // Parse the JSON when it arrives
    .then(data => {
        // This runs much later, after the network I/O is completely done.
        console.log("4. Network I/O Finished! Data received for:", data.name);
    })
    .catch(error => console.error("Something went wrong:", error));

console.log("2. Fetch request was handed off to the browser.");
console.log("3. Script is moving on! I can handle button clicks or animations now.");
```

### Creating Your Own

We refer to this practice as **"Promisifying."** You typically do this when you need to bridge the gap between modern `async/await` syntax and older, callback-based systems, or when you are dealing with specific timing/UI events.

This can be useful for 

1. Wrapping older APIs that function on callbacks

```jsx
function readUploadedFile(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        
        // When the browser finishes reading, we resolve the promise
        reader.onload = () => resolve(reader.result);
        
        // If the file is corrupt or fails, we reject it
        reader.onerror = () => reject(reader.error);
        
        reader.readAsDataURL(file); // Starts the actual work
    });
}

// Usage:
console.log("1. Starting upload process...");

// Notice: NO 'await' keyword here! 
// We just call the function. It returns a pending Promise immediately.
readUploadedFile(userFile)
    .then(imagePreviewUrl => {
        // This block sits in the background and only runs whenever the file finishes.
        console.log("4. File is finally ready! URL:", imagePreviewUrl);
    })
    .catch(error => {
        console.error("Oops, file read failed:", error);
    });

// BECAUSE THERE IS NO AWAIT, EXECUTION IMMEDIATELY JUMPS HERE
console.log("2. Moving on! The file is reading in the background.");
console.log("3. User can keep typing in the form...");
```

1. Waiting on user action

```jsx
function askUserConfirmation(message) {
    return new Promise((resolve) => {
        // 1. Show your custom UI modal
        myModalElement.innerText = message;
        myModalElement.show();

        // 2. Wait for the button clicks
        document.getElementById('btn-yes').onclick = () => {
            myModalElement.hide();
            resolve(true); 
        };
        
        document.getElementById('btn-no').onclick = () => {
            myModalElement.hide();
            resolve(false); 
        };
    });
}

// Usage:
const isSure = await askUserConfirmation("Delete this record?");
```

# Modules

An **ES6 module** is **the official, standardized system for organizing JavaScript code into separate, reusable files or "chunks," introduced in ECMAScript 2015 (ES6)**. This system uses `import` and `export` statements to share functions, classes, and variables between different files, which helps keep code organized, maintainable, and scalable. 

# Barrel Files

### **What’s a “barrel file” in JS/TS?**

A **barrel file** is a small module whose **only job** is to **re-export** symbols from other modules so consumers can write:

import { X, Y } from 'some-package/some-folder';

instead of many deep imports:

import { X } from 'some-package/some-folder/x';

import { Y } from 'some-package/some-folder/y';

It’s usually named **`index.ts` / `index.js`** at the folder root, but **any file** that only `export { ... } from './other'` is acting as a barrel.