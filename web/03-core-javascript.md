# Part 3: Core JavaScript Deep Dive 🧬

## Week 3 | 10 hours | Master the language, not just the framework

---

## Why This Matters

You've used JavaScript through Vue for 7 years. But FAANG interviews test
RAW JavaScript — no frameworks, no libraries. They want to know:
- Do you truly understand closures, or do you just use them?
- Can you explain prototypal inheritance, or do you just use `class`?
- Can you implement `Promise.all` from scratch?
- Can you build a debounce/throttle without lodash?

**This is the difference between "I use JavaScript" and "I KNOW JavaScript."**

---

## 📖 Reading Plan (3 hours)

### Must Read:
1. **javascript.info** — Read these chapters in full:
   - Closures: https://javascript.info/closure
   - Prototypes: https://javascript.info/prototypes
   - Promises, async/await: https://javascript.info/async
   - Generators: https://javascript.info/generators
   - Proxy and Reflect: https://javascript.info/proxy
   - Iterators: https://javascript.info/iterable

2. **"You Don't Know JS Yet" (YDKJSY) — Kyle Simpson** (FREE online)
   - Book 2: "Scope & Closures" — chapters on lexical scope, closure
   - Book 3: "Objects & Classes" — chapters on `this`, prototypes
   - https://github.com/getify/You-Dont-Know-JS

3. **MDN References** (use as lookup, not sequential reading):
   - `this` keyword: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
   - Equality comparisons: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness

---

## 🧠 Deep Dive: Core Concepts

### 1. Execution Context & Scope Chain

```javascript
// Every function execution creates an EXECUTION CONTEXT:
// {
//   VariableEnvironment: { local variables },
//   LexicalEnvironment: { let/const bindings },
//   ThisBinding: { what `this` refers to },
//   OuterReference: { parent scope — this creates the scope chain }
// }

var x = 'global';

function outer() {
  var y = 'outer';

  function inner() {
    var z = 'inner';
    console.log(x, y, z);
    // scope chain: inner → outer → global
    // finds x in global, y in outer, z in inner
  }

  inner();
}

outer(); // 'global' 'outer' 'inner'
```

**Scope Chain Resolution:**
```
inner() scope:   { z: 'inner' }  → not found? go to outer reference ↓
outer() scope:   { y: 'outer' }  → not found? go to outer reference ↓
global scope:    { x: 'global' } → found!
```

---

### 2. Closures — Deep Understanding

```javascript
// A closure is a function + its lexical environment
// The function "closes over" variables from its outer scope

function createCounter() {
  let count = 0; // This variable is "enclosed" by the returned function

  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount()  { return count; }
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.getCount();  // 2

// `count` is NOT accessible from outside
// But all three methods share the SAME `count` variable
// This is the closure — the methods "remember" their creation scope
```

**Classic Interview Gotcha:**
```javascript
// What does this print?
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Answer: 3, 3, 3
// Why: `var` is function-scoped. All callbacks share the same `i`.
// By the time setTimeout fires, loop is done and i = 3.

// Fix 1: Use `let` (block-scoped — each iteration gets its own `i`)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Answer: 0, 1, 2

// Fix 2: IIFE (creates a new scope per iteration)
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// Answer: 0, 1, 2

// Fix 3: Closure explicitly
for (var i = 0; i < 3; i++) {
  setTimeout(console.log.bind(null, i), 100);
}
// Answer: 0, 1, 2
```

**Closure Use Cases (real-world):**
```javascript
// 1. Data privacy / encapsulation
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private

  return {
    deposit(amount) {
      if (amount <= 0) throw new Error('Invalid amount');
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) throw new Error('Insufficient funds');
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    }
  };
}

// 2. Function factories
function multiply(factor) {
  return (number) => number * factor; // closes over `factor`
}
const double = multiply(2);
const triple = multiply(3);
double(5); // 10
triple(5); // 15

// 3. Memoization
function memoize(fn) {
  const cache = new Map(); // closed over by returned function

  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

---

### 3. `this` Keyword — Complete Rules

```javascript
// `this` is determined by HOW a function is CALLED, not where it's defined.
// There are exactly 5 rules (in priority order):

// Rule 1: `new` binding — `this` = newly created object
function Person(name) {
  this.name = name; // `this` = new object
}
const p = new Person('Alice'); // this = {}

// Rule 2: Explicit binding — call, apply, bind
function greet() {
  console.log(this.name);
}
greet.call({ name: 'Bob' });    // this = { name: 'Bob' }
greet.apply({ name: 'Bob' });   // same, but args as array
const bound = greet.bind({ name: 'Bob' }); // returns new function
bound(); // this = { name: 'Bob' }

// Rule 3: Implicit binding — called as method of object
const obj = {
  name: 'Charlie',
  greet() {
    console.log(this.name); // this = obj
  }
};
obj.greet(); // 'Charlie'

// BUT: Method extraction loses `this`
const fn = obj.greet;
fn(); // undefined! (this = window in non-strict, undefined in strict)

// Rule 4: Default binding — standalone function call
function standalone() {
  console.log(this); // window (non-strict) or undefined (strict mode)
}
standalone();

// Rule 5: Arrow functions — NO own `this`, inherits from enclosing scope
const obj2 = {
  name: 'Diana',
  greet: () => {
    console.log(this.name); // `this` is NOT obj2!
    // Arrow functions take `this` from where they're DEFINED
    // Here it's the module/global scope
  },
  delayedGreet() {
    setTimeout(() => {
      console.log(this.name); // 'Diana' — arrow inherits from delayedGreet's `this`
    }, 100);
  }
};
```

**Priority: new > explicit (call/apply/bind) > implicit (method) > default > arrow (lexical)**

---

### 4. Prototypal Inheritance

```javascript
// JavaScript does NOT have classical inheritance.
// It has PROTOTYPAL inheritance — objects delegate to other objects.

// Every object has an internal [[Prototype]] link
const animal = {
  eat() { console.log('eating'); }
};

const dog = Object.create(animal); // dog's [[Prototype]] = animal
dog.bark = function() { console.log('woof'); };

dog.bark(); // 'woof' — found on dog
dog.eat();  // 'eating' — NOT on dog, found on dog's prototype (animal)

// Prototype chain:
// dog → animal → Object.prototype → null

// How `class` syntax works under the hood:
class Animal {
  constructor(name) {
    this.name = name;
  }
  eat() {
    console.log(`${this.name} eats`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`${this.name} barks`);
  }
}

// Is equivalent to:
function AnimalFn(name) {
  this.name = name;
}
AnimalFn.prototype.eat = function() {
  console.log(`${this.name} eats`);
};

function DogFn(name) {
  AnimalFn.call(this, name); // super() equivalent
}
DogFn.prototype = Object.create(AnimalFn.prototype);
DogFn.prototype.constructor = DogFn;
DogFn.prototype.bark = function() {
  console.log(`${this.name} barks`);
};

// Key insight: methods are on the PROTOTYPE, not on instances
// 1000 Dog instances share ONE eat() function — memory efficient
```

**Prototype chain lookup:**
```javascript
const d = new Dog('Rex');
d.bark();     // found on Dog.prototype ✓
d.eat();      // not on Dog.prototype → found on Animal.prototype ✓
d.toString(); // not on Dog.prototype → not on Animal.prototype → found on Object.prototype ✓
d.foo();      // not found anywhere → TypeError: d.foo is not a function
```

---

### 5. Promises — Complete Understanding

```javascript
// A Promise is a state machine:
// PENDING → FULFILLED (resolved with a value)
//         → REJECTED (rejected with a reason)
// Once settled (fulfilled or rejected), state NEVER changes.

// Creating a Promise:
const myPromise = new Promise((resolve, reject) => {
  // Do async work...
  if (success) resolve(value);
  else reject(error);
});

// Chaining:
myPromise
  .then(value => {
    // runs if fulfilled
    return newValue; // next .then receives newValue
  })
  .catch(error => {
    // runs if rejected (at any point in the chain)
    return recoveredValue; // chain continues as fulfilled!
  })
  .finally(() => {
    // runs regardless of fulfilled or rejected
    // return value is ignored
  });
```

**Implement Promise.all from scratch (FAANG interview question):**
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }

    const results = [];
    let completed = 0;
    const total = promises.length;

    if (total === 0) return resolve([]);

    promises.forEach((promise, index) => {
      // Wrap in Promise.resolve to handle non-promise values
      Promise.resolve(promise)
        .then(value => {
          results[index] = value; // maintain order!
          completed++;
          if (completed === total) {
            resolve(results);
          }
        })
        .catch(reject); // first rejection rejects the whole thing
    });
  });
}
```

**Implement Promise.allSettled:**
```javascript
function promiseAllSettled(promises) {
  return new Promise((resolve) => {
    const results = [];
    let completed = 0;
    const total = promises.length;

    if (total === 0) return resolve([]);

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
        })
        .finally(() => {
          completed++;
          if (completed === total) resolve(results);
        });
    });
  });
}
```

**Implement Promise.race:**
```javascript
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(promise => {
      Promise.resolve(promise).then(resolve).catch(reject);
    });
  });
}
```

---

### 6. Generators & Iterators

```javascript
// Generators are functions that can PAUSE and RESUME execution
function* numberGenerator() {
  console.log('Start');
  yield 1;          // Pause here, return { value: 1, done: false }
  console.log('After 1');
  yield 2;          // Pause here, return { value: 2, done: false }
  console.log('After 2');
  return 3;         // Done, return { value: 3, done: true }
}

const gen = numberGenerator();
gen.next(); // 'Start'    → { value: 1, done: false }
gen.next(); // 'After 1'  → { value: 2, done: false }
gen.next(); // 'After 2'  → { value: 3, done: true }

// Generators implement the Iterator protocol
// They can be used with for...of
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

for (const num of range(1, 5)) {
  console.log(num); // 1, 2, 3, 4, 5
}

// Practical use: Lazy evaluation (process infinite sequences)
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Take first 10 fibonacci numbers (doesn't compute all infinity)
const fib = fibonacci();
const first10 = Array.from({ length: 10 }, () => fib.next().value);
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Async generators (used in AI streaming!):
async function* streamResponse(url) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    yield decoder.decode(value, { stream: true });
  }
}

// Consume:
for await (const chunk of streamResponse('/api/stream')) {
  appendToUI(chunk); // Process each chunk as it arrives
}
```

---

### 7. Proxy & Reflect

```javascript
// Proxy wraps an object and intercepts operations on it
// This is how Vue 3's reactivity system works!

const handler = {
  get(target, property, receiver) {
    console.log(`Reading ${property}`);
    return Reflect.get(target, property, receiver);
  },
  set(target, property, value, receiver) {
    console.log(`Writing ${property} = ${value}`);
    // Vue 3: This is where reactivity tracking happens!
    // trigger re-render when data changes
    return Reflect.set(target, property, value, receiver);
  },
  deleteProperty(target, property) {
    console.log(`Deleting ${property}`);
    return Reflect.deleteProperty(target, property);
  }
};

const data = new Proxy({ name: 'Ganesh', age: 30 }, handler);
data.name;          // "Reading name"
data.age = 31;      // "Writing age = 31"
delete data.name;   // "Deleting name"

// Vue 3's reactive() is essentially this:
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key, receiver) {
      track(target, key);    // Track which component uses this property
      return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
      const result = Reflect.set(target, key, value, receiver);
      trigger(target, key);  // Re-render components that depend on this property
      return result;
    }
  });
}

// Other traps: has, ownKeys, apply, construct, getPrototypeOf, etc.
// Full list: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy
```

---

## 🔨 Build Project (4 hours)

### Project: Implement Utility Library from Scratch

**No libraries. No frameworks. Pure JavaScript.**

```
js-utils/
├── src/
│   ├── debounce.js
│   ├── throttle.js
│   ├── deepClone.js
│   ├── EventEmitter.js
│   ├── promiseAll.js
│   ├── flatten.js
│   ├── curry.js
│   └── observable.js
├── tests/
│   └── (test each utility)
└── index.html (test runner)
```

#### Must Implement:

```javascript
// 1. debounce
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

// 2. throttle
function throttle(fn, limit) {
  let inThrottle = false;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// 3. deepClone (handle objects, arrays, Date, RegExp, Map, Set)
function deepClone(obj, seen = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (seen.has(obj)) return seen.get(obj); // handle circular references

  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);
  if (obj instanceof Map) {
    const map = new Map();
    seen.set(obj, map);
    obj.forEach((v, k) => map.set(deepClone(k, seen), deepClone(v, seen)));
    return map;
  }
  if (obj instanceof Set) {
    const set = new Set();
    seen.set(obj, set);
    obj.forEach(v => set.add(deepClone(v, seen)));
    return set;
  }

  const clone = Array.isArray(obj) ? [] : {};
  seen.set(obj, clone);

  for (const key of Reflect.ownKeys(obj)) {
    clone[key] = deepClone(obj[key], seen);
  }
  return clone;
}

// 4. EventEmitter (pub/sub pattern)
class EventEmitter {
  constructor() {
    this.events = new Map();
  }

  on(event, callback) {
    if (!this.events.has(event)) this.events.set(event, []);
    this.events.get(event).push(callback);
    return this; // for chaining
  }

  off(event, callback) {
    if (!this.events.has(event)) return this;
    this.events.set(event, this.events.get(event).filter(cb => cb !== callback));
    return this;
  }

  emit(event, ...args) {
    if (!this.events.has(event)) return false;
    this.events.get(event).forEach(cb => cb.apply(this, args));
    return true;
  }

  once(event, callback) {
    const wrapper = (...args) => {
      callback.apply(this, args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
}

// 5. curry
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...moreArgs) {
      return curried.apply(this, [...args, ...moreArgs]);
    };
  };
}

// 6. flatten (array, with depth)
function flatten(arr, depth = 1) {
  if (depth <= 0) return arr.slice();
  return arr.reduce((result, item) => {
    if (Array.isArray(item)) {
      result.push(...flatten(item, depth - 1));
    } else {
      result.push(item);
    }
    return result;
  }, []);
}

// 7. Simple Observable (reactive value — Vue's ref() from scratch)
function observable(initialValue) {
  let value = initialValue;
  const subscribers = new Set();

  return {
    get() {
      return value;
    },
    set(newValue) {
      if (Object.is(value, newValue)) return;
      value = newValue;
      subscribers.forEach(fn => fn(value));
    },
    subscribe(fn) {
      subscribers.add(fn);
      return () => subscribers.delete(fn); // unsubscribe function
    }
  };
}
```

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **What is a closure? Give a practical example.**
2. **Explain `this` in JavaScript. What are the 5 binding rules?**
3. **What is the prototype chain? How does property lookup work?**
4. **What's the difference between `==` and `===`?**
   - `==` does type coercion, `===` does not
   - `null == undefined` is true, `null === undefined` is false
   - Always use `===` in production code
5. **What's the difference between `var`, `let`, and `const`?**
   - `var`: function-scoped, hoisted (initialized as undefined)
   - `let`: block-scoped, hoisted (but in TDZ — Temporal Dead Zone)
   - `const`: block-scoped, hoisted (TDZ), must be initialized, reference is constant

### Intermediate:
6. **Implement `debounce` from scratch. Explain when to use it vs `throttle`.**
7. **Implement `Promise.all` from scratch.**
8. **What is the Temporal Dead Zone?**
   ```javascript
   console.log(x); // undefined (var is hoisted with value undefined)
   var x = 5;

   console.log(y); // ReferenceError! (let is hoisted but in TDZ)
   let y = 5;
   ```
9. **Explain event delegation. Why is it useful?**
   ```javascript
   // Instead of adding listener to each <li>:
   document.querySelector('ul').addEventListener('click', (e) => {
     if (e.target.tagName === 'LI') {
       handleItemClick(e.target);
     }
   });
   // Benefits: fewer listeners, works for dynamically added elements
   ```
10. **What is the difference between shallow copy and deep copy?**
    - Shallow: `Object.assign()`, spread `{...obj}` — nested objects share references
    - Deep: `structuredClone()`, custom recursive clone — everything is independent

### Advanced:
11. **Implement `Function.prototype.bind` from scratch.**
    ```javascript
    Function.prototype.myBind = function(context, ...boundArgs) {
      const fn = this;
      return function(...callArgs) {
        return fn.apply(context, [...boundArgs, ...callArgs]);
      };
    };
    ```

12. **Explain how `async/await` is syntactic sugar over generators + promises.**
    ```javascript
    // async/await:
    async function fetchData() {
      const res = await fetch(url);
      const data = await res.json();
      return data;
    }

    // Desugared to generator + auto-runner:
    function fetchData() {
      return spawn(function*() {
        const res = yield fetch(url);
        const data = yield res.json();
        return data;
      });
    }

    function spawn(generatorFn) {
      return new Promise((resolve, reject) => {
        const gen = generatorFn();
        function step(nextFn) {
          let result;
          try { result = nextFn(); }
          catch(e) { return reject(e); }
          if (result.done) return resolve(result.value);
          Promise.resolve(result.value).then(
            v => step(() => gen.next(v)),
            e => step(() => gen.throw(e))
          );
        }
        step(() => gen.next());
      });
    }
    ```

13. **How does Vue 3's reactivity system use Proxy? How is it different from Vue 2's Object.defineProperty?**
    - Vue 2: `Object.defineProperty` — can't detect property addition/deletion
    - Vue 3: `Proxy` — intercepts all operations including add/delete/has
    - You should know this deeply since you use Vue

14. **Implement a simple reactive system (like Vue's `ref` + `watchEffect`).**

15. **What are WeakMap and WeakSet? When would you use them?**
    - Keys must be objects (not primitives)
    - Keys are weakly held — if no other reference exists, GC can collect them
    - Not iterable (can't enumerate entries)
    - Use case: storing metadata about objects without preventing GC

---

## ✅ Week 3 Completion Checklist

- [ ] Can explain closures with 3 different real-world examples
- [ ] Can explain `this` with all 5 binding rules + priority order
- [ ] Can explain prototypal inheritance and how `class` works underneath
- [ ] Can implement: debounce, throttle, deepClone, EventEmitter, curry from scratch
- [ ] Can implement: Promise.all, Promise.race, Promise.allSettled from scratch
- [ ] Understand generators and async generators
- [ ] Understand Proxy/Reflect and how Vue 3 uses them
- [ ] Can answer all 15 interview questions out loud
- [ ] Built the utility library with tests

---

Next → `04-networking.md`
