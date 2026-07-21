# React | Django Mock Interview — Deduplicated Q&A Reference

All duplicate questions from the interview sessions (React/Django set + Shadow/MERN+Angular set) merged into one list, organized by topic, with short beginner-friendly answers.

## Table of Contents

1. [OOP Fundamentals](#1-oop-fundamentals)
2. [SOLID Principles](#2-solid-principles)
3. [Design Patterns](#3-design-patterns)
4. [Python](#4-python)
5. [JavaScript](#5-javascript)
6. [Java](#6-java)
7. [TypeScript](#7-typescript)
8. [React](#8-react)
9. [Angular](#9-angular)
10. [Node.js](#10-nodejs)
11. [Django & Backend](#11-django--backend)
12. [Databases & SQL](#12-databases--sql)
13. [MongoDB & NoSQL](#13-mongodb--nosql)
14. [Data Structures](#14-data-structures)
15. [Git](#15-git)
16. [HTML & CSS](#16-html--css)
17. [Auth & Security](#17-auth--security)
18. [System Design, Performance & Deployment](#18-system-design-performance--deployment)
19. [DevOps, Docker & Testing](#19-devops-docker--testing)
20. [AI/ML Basics](#20-aiml-basics-bonus-if-asked)
21. [CS Theory](#21-cs-theory)
22. [Behavioral](#22-behavioral)
23. [Coding Problems Mentioned](#23-coding-problems-mentioned-practice-separately-on-leetcode)

---

## 1. OOP Fundamentals

**4 Pillars of OOP**
- Encapsulation: hide internal details, expose only what's needed.
- Abstraction: show *what* an object does, hide *how*.
- Inheritance: a class reuses code from a parent class.
- Polymorphism: same method name, different behavior per class.

**Why Do We Use OOP / What Was OOP Trying to Solve**
- Organizes code around real-world entities (objects) instead of raw functions, making large codebases more reusable, maintainable, and easier to reason about.

**Class vs Instance / Class vs Object**
- Class = the blueprint (e.g. `Car`). Instance/Object = the actual thing built from it (e.g. `my_car`).

**Object vs Struct**
- Object: bundles data + behavior (methods) together, usually supports inheritance/encapsulation.
- Struct (e.g. in C): usually just bundles data fields, no methods or inheritance.

**Composition vs Aggregation vs Association vs Inheritance**
- Composition: strong ownership — part dies with the whole (Car has Engine).
- Aggregation: weak ownership — part can exist without the whole (Team has Players).
- Association: two classes just know about each other, no ownership (Teacher–Student).
- Inheritance: "is-a" relationship (Dog is an Animal).

**Overloading vs Overriding**
- Overloading: same method name, different parameters (not really supported in Python/JS, common in Java).
- Overriding: child class redefines a parent's method with the same signature.

**Types of Inheritance**
- Single, Multiple, Multilevel, Hierarchical, Hybrid.

**Types of Polymorphism**
- Compile-time (method overloading) and Runtime (method overriding).

**Interface vs Abstract Class**
- Abstract class: can have some implemented methods plus state; a class can usually extend only one.
- Interface: only declares method signatures (no implementation in most languages); a class can implement many interfaces.

**Diamond Problem (Multiple Inheritance)**
- Happens when a class inherits from two parents that both define the same method — it's unclear which version to use. This ambiguity is why many languages block multiple inheritance for classes and use interfaces instead.

**Constructor vs Destructor**
- Constructor: runs when an object is created, to set it up.
- Destructor: runs when an object is destroyed, to clean up resources (not commonly used directly in Python/JS since they use garbage collection).

**Constructor Types**
- Default (no args, sets defaults), Parameterized (takes args to initialize fields), Copy (creates a new object from an existing one).

**Data Hiding vs Data Binding**
- Data hiding: restricting direct access to internal data (part of encapsulation, e.g. private fields).
- Data binding: keeping UI and underlying data in sync (common term in frameworks like Angular).

**Private Constructors**
- A constructor made inaccessible from outside the class, so you can't create instances directly — used in the Singleton pattern to fully control object creation.

**Access Modifiers**
- `public`: accessible from anywhere. `private`: accessible only inside the class. `protected`: accessible inside the class and its subclasses.

**Final vs Static**
- `final`: a variable/method/class can't be changed, overridden, or extended after it's defined.
- `static`: belongs to the class itself, not to any individual instance (shared across all objects).

**Aim of `this`**
- Refers to the current instance of the object the code is running on.

**Is Everything in Python an Object?**
- Yes — numbers, functions, classes, even modules are objects in Python.

---

## 2. SOLID Principles

**S** — Single Responsibility: a class should do one job only.
**O** — Open/Closed: open for extension, closed for modification (add new code, don't edit old working code).
**L** — Liskov Substitution: a subclass should be usable anywhere its parent is, without breaking things.
**I** — Interface Segregation: don't force a class to implement methods it doesn't need — split big interfaces into smaller ones.
**D** — Dependency Inversion: depend on abstractions (interfaces), not on concrete classes.

**Dependency Inversion vs Dependency Injection**
- Dependency Inversion = the *principle* (depend on abstractions, not concrete implementations).
- Dependency Injection = the *technique* used to achieve it (passing dependencies into a class instead of creating them inside it).

---

## 3. Design Patterns

**Types of Design Patterns**
- Creational (object creation): Singleton, Builder, Factory.
- Structural (object composition): Adapter, Decorator.
- Behavioral (object communication): Observer, Strategy.

**Singleton**
- Ensures only one instance of a class exists (e.g. a single DB connection object). Usually implemented with a private constructor.

**Singleton — Disadvantages (Especially with Multithreading)**
- Introduces global state, which makes code harder to test and can hide dependencies between parts of the system.
- In a multithreaded environment, naive Singleton implementations can create a race condition: two threads could both check "no instance exists yet" at the same time and each create their own instance, breaking the "only one instance" guarantee — needs a thread-safe creation approach (e.g. locking, or creating the instance eagerly at startup) to fix.

**Types of Abstraction / When to Use Each**
- Data abstraction: hiding internal data representation, exposing only relevant properties (e.g. an object exposing `getBalance()` instead of raw internal fields).
- Process/control abstraction: hiding the steps of *how* something is done, exposing only *what* it does (e.g. an interface with a `save()` method — the caller doesn't know or care about the underlying implementation).
- Use data abstraction when protecting internal state; use process abstraction when you want implementations to be swappable behind a consistent contract.

**Builder**
- Builds a complex object step by step instead of one huge constructor.

**Observer**
- One object (subject) notifies many other objects (observers) when its state changes — like event listeners. Node's `EventEmitter` is a real-world example (also called Pub/Sub).

**Adapter**
- Converts one interface into another so incompatible classes can work together.

---

## 4. Python

**Python Data Types**
- Numeric (int, float, complex), Sequence (list, tuple, str), Mapping (dict), Set, Boolean, NoneType.

**List vs Tuple**
- List: mutable, slower, more memory. Tuple: immutable, faster, less memory. Use tuple for fixed data.

**Mutable vs Immutable**
- Mutable: can change after creation (list, dict). Immutable: can't (tuple, str, int).

**Shallow Copy vs Deep Copy**
- Shallow copy: copies outer object, inner objects still shared/linked.
- Deep copy: copies everything fully, no shared references.

***args vs **kwargs**
- `*args`: collects extra positional arguments into a tuple.
- `**kwargs`: collects extra keyword arguments into a dictionary.

**Closures**
- A function that "remembers" variables from the scope it was created in, even after that scope has finished running.

**Decorators**
- A function that wraps another function to add extra behavior without changing its code.
- Runs **both** before and after: the wrapper code before calling the original function runs first, code after it runs last.

**Exception Handling / `pass` in except**
- `try/except` catches errors so the program doesn't crash.
- Using `pass` in an except block means "ignore this specific error and keep running" — use carefully, don't silence everything blindly.

**== vs is**
- `==` compares values. `is` compares identity (same object in memory).

**Multithreading vs Multiprocessing vs Async**
- Multithreading: multiple threads, same process, share memory — good for I/O-bound tasks, limited by the GIL for CPU tasks.
- Multiprocessing: multiple separate processes, each with own memory — good for CPU-bound tasks (true parallelism).
- Async: single thread, switches between tasks while waiting (e.g. waiting on network) — good for many I/O tasks at once.

**What is the GIL?**
- Global Interpreter Lock — only one thread executes Python bytecode at a time, even on multi-core CPUs. This is why multithreading doesn't speed up CPU-heavy Python code.

**Memory Allocation in Python**
- Python manages memory automatically via a private heap; objects are reference-counted and a garbage collector cleans up unused objects.

**Pass by Value vs Reference vs Object**
- Python passes by "object reference": you get a reference to the same object. Mutating a mutable object (list) affects the original; reassigning a variable doesn't.

**Object Chaining**
- Calling multiple methods in sequence on the same object, e.g. `obj.method1().method2()` — each method returns `self` (or another object) to allow chaining.

**Reference Sharing**
- Two variables can point to the same object in memory, so changing it through one variable affects the other.

**Object Caching**
- Python reuses (caches) small immutable objects like small integers and short strings for performance.

**Does Python have `===`?**
- No. Python only has `==` (value) and `is` (identity) — no separate strict-equality operator like JS.

**Recursion vs Loop**
- Recursion: function calls itself, good for naturally recursive problems (trees, divide & conquer), but uses more memory (call stack).
- Loop: usually faster and uses less memory — prefer loops unless the problem is naturally recursive.

**Static vs Dynamic Typed Language**
- Static (e.g. Java, TypeScript): variable types are fixed and checked at compile time.
- Dynamic (e.g. Python, JavaScript): types are determined at runtime and a variable can hold different types over time.

**Class Methods vs Static Methods vs Instance Methods**
- Instance method: the default — takes `self`, works on a specific object's data.
- Class method (`@classmethod`): takes `cls` instead of `self`, works on the class itself (e.g. alternative constructors) — shared across all instances.
- Static method (`@staticmethod`): takes neither `self` nor `cls` — just a regular function grouped inside the class for organization, no access to instance/class state.

**Reference Counting**
- Python's core memory management mechanism: every object tracks how many references point to it. When that count hits zero, the object is immediately deallocated. A separate garbage collector handles the one thing reference counting can't: reference cycles (objects referencing each other).

**List Comprehensions — Syntactic Sugar or Different Memory Behavior?**
- Mostly syntactic sugar for readability, but they're not purely cosmetic: they run slightly faster than an equivalent `for` loop because the looping happens in optimized C code internally, with fewer Python-level function calls (like repeated `.append()` lookups).

**List Comprehension vs For-Loop Append — Which Is Faster?**
- List comprehensions are generally faster, since they avoid the repeated overhead of looking up and calling `.append()` on every iteration — the loop and append logic run in C under the hood instead of interpreted Python bytecode.

**Mutability vs Immutability — Why It Exists / What It's Used For**
- It exists to give you a choice between safety and flexibility: immutable objects (tuples, strings) are safe to share/cache/use as dict keys because they can't change unexpectedly, while mutable objects (lists, dicts) are useful when you need to efficiently update data in place without creating new objects each time.

**If Python Has the GIL, When Do We Still Use Multithreading?**
- For I/O-bound tasks (network calls, file/database reads/writes) — the GIL is released while a thread waits on I/O, so other threads can run in that gap. Multithreading doesn't help CPU-bound tasks (like heavy computation), where multiprocessing is needed instead.

**Multithreading (with GIL) vs Async**
- Multithreading: the OS switches between threads, useful for I/O-bound work, but has real overhead (context switching, and only one thread executes Python bytecode at a time due to the GIL).
- Async (`asyncio`): a single thread cooperatively switches between tasks at `await` points — no OS-level context-switching overhead, generally more efficient for large numbers of concurrent I/O tasks, but requires async-aware libraries throughout.

**OOP in Python vs Java vs C++**
- Python: everything is an object (even functions/classes), no strict access modifiers (just naming conventions like `_protected`/`__private`), duck typing, single inheritance is simple but supports multiple inheritance too.
- Java: strict access modifiers enforced by the compiler, no multiple inheritance for classes (only via interfaces), everything must live inside a class.
- C++: gives manual control (pointers, manual memory management), supports multiple inheritance directly (with the diamond problem to manage), faster since it compiles to machine code rather than running on a VM/interpreter.

**When to Use Python vs Java**
- Python: faster to write/prototype, huge ecosystem for scripting/data/AI, dynamic typing — good when development speed matters most.
- Java: stricter typing catches more bugs at compile time, generally faster raw execution, true native multithreading (no GIL) — good for large enterprise systems and CPU-heavy concurrent workloads.

**Is Java Single or Multi-threaded?**
- Multi-threaded natively — Java doesn't have anything like Python's GIL, so multiple threads can truly run in parallel on multiple CPU cores.

**Custom Decorator Example (e.g. DRF-Style Permission Check)**
- A decorator can wrap a view function to add a check before it runs, e.g. a decorator that verifies a user's role and blocks the request (returns a 403) if they don't have permission — similar to how Django REST Framework's permission classes gate access to specific views without repeating the check in every view's code.

---

## 5. JavaScript

**var vs let vs const**
- `var`: function-scoped, can be redeclared, hoisted as `undefined`.
- `let`: block-scoped, can be reassigned but not redeclared in the same scope.
- `const`: block-scoped, can't be reassigned (the value itself can still be mutated if it's an object/array).

**Hoisting**
- JS moves variable and function *declarations* to the top of their scope before running the code. `var` is hoisted as `undefined`; `let`/`const` are hoisted but not accessible before declaration (temporal dead zone).

**Temporal Dead Zone (TDZ)**
- The gap between entering a scope and the actual line where a `let`/`const` variable is declared — accessing the variable during this gap throws an error.

**ES6 Features**
- `let`/`const`, arrow functions, template literals, destructuring, spread/rest operators, Promises, classes, modules (`import`/`export`).

**Spread vs Rest**
- Spread (`...`): expands an array/object into individual elements, e.g. `[...arr]`.
- Rest (`...`): collects multiple arguments/elements into one array, e.g. `function f(...args)`.

**=== vs ==**
- `===`: strict equality, compares value **and** type.
- `==`: loose equality, converts types before comparing — prefer `===` to avoid surprises.

**Coercion vs Casting**
- Coercion: JS automatically converts types for you (e.g. `"5" + 1` → `"51"`).
- Casting: you explicitly convert types yourself (e.g. `Number("5")`).

**Null vs Undefined**
- `undefined`: a variable was declared but never assigned a value.
- `null`: intentionally set to "no value" by the developer.

**Undefined vs Not Defined vs ReferenceError vs Null**
- `undefined`: variable exists but has no value yet.
- "Not defined" / `ReferenceError`: variable was never declared at all.
- `null`: an intentional "empty" value set by the developer.

**Promises + States**
- Represents a value that will be available later (from an async operation).
- States: pending (not finished yet), fulfilled (succeeded), rejected (failed).

**Promise.all() vs Promise.allSettled()**
- `all()`: fails fast — rejects immediately if any single promise fails.
- `allSettled()`: waits for every promise and returns all results, whether they succeeded or failed.

**Async/Await vs Promise**
- `async/await` is syntax sugar over Promises — lets async code read top-to-bottom like synchronous code instead of chaining `.then()`.

**Callback Hell**
- Deeply nested callbacks that become hard to read/maintain. Solved with Promises, and made even cleaner with `async/await`.

**for...of vs for...in**
- `for...of`: loops over values (arrays, strings, iterables).
- `for...in`: loops over keys/property names (objects, or array indexes).

**JS Single-Threaded / How JS Runs (Execution Context)**
- JS runs on one thread. It creates an execution context (global first, then one per function call) that sets up memory before running code line by line.
- For async work, JS uses the Event Loop: it sets async tasks (timers, network calls) aside, keeps running other code, then processes their callbacks once the main stack is empty.

**Filter vs Map vs Reduce (vs forEach)**
- `filter`: keeps items that pass a condition, returns a smaller/equal array.
- `map`: transforms each item, returns a new array of the same length.
- `reduce`: combines all items into a single value (sum, object, etc.).
- `forEach`: just loops and runs a function per item, returns nothing (`undefined`).

**for Loop vs forEach**
- `for` loop: can use `break`/`continue`, generally faster for large datasets.
- `forEach`: cleaner syntax but can't `break` out early.

**Higher Order Functions**
- Functions that take other functions as arguments or return a function (e.g. `map`, `filter`, event listeners).

**Call vs Apply vs Bind**
- All three set what `this` refers to inside a function.
- `call()`: invokes immediately, arguments listed individually.
- `apply()`: invokes immediately, arguments passed as an array.
- `bind()`: returns a new function with `this` set, without calling it right away.

**Object.create() vs Object.assign()**
- `Object.create()`: makes a new object with a given object as its prototype.
- `Object.assign()`: copies properties from one or more source objects into a target object.

**Object.freeze() vs Object.seal()**
- `freeze()`: locks the object completely — no adding, removing, or changing properties.
- `seal()`: blocks adding/removing properties, but existing ones can still be changed.

**How to Clone an Object**
- Shallow clone: spread syntax `{...obj}` or `Object.assign({}, obj)` — nested objects are still shared references.
- Deep clone: `structuredClone(obj)` (modern browsers/Node), or `JSON.parse(JSON.stringify(obj))` (loses functions/special types), or a library like Lodash's `cloneDeep`.

**Primitive Data Types (JavaScript)**
- `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint` — all immutable and compared by value (unlike objects, which are compared by reference).

**Prototype & Prototype Chain**
- Every JS object has a hidden link to another object (its prototype) that it can borrow methods/properties from.
- The chain of these links is the prototype chain — JS walks up it until it finds the property or runs out of links.

**Prototype vs `__proto__`**
- `prototype`: a property on constructor functions, used as the template for objects created from them.
- `__proto__`: the actual link on an instance pointing to its prototype object.

**Scope Chain**
- How JS looks up a variable: checks the current scope first, then keeps checking outer scopes until it finds it (or throws a `ReferenceError`).

**Lexical Scope**
- A function's scope is decided by *where it's written* in the code, not where it's called — inner functions can access variables of their outer (enclosing) functions.

**Function Scope vs Block Scope**
- Function scope: variable is visible anywhere inside the function it's declared in (`var`).
- Block scope: variable is visible only inside the `{ }` block it's declared in (`let`, `const`).

**First-Class Functions / First-Class Objects**
- In JS, functions are treated like any other value — they can be stored in variables, passed as arguments, and returned from other functions.

**Currying**
- Transforming a function that takes multiple arguments into a chain of functions that each take one argument, e.g. `add(2)(3)` instead of `add(2, 3)`.

**Self-Invoked Function (IIFE)**
- An "Immediately Invoked Function Expression" — a function that runs the moment it's defined: `(function(){ ... })();`. Used to create a private scope.

**Arrow Functions**
- Shorter syntax for writing functions; they don't have their own `this` (they inherit it from the surrounding scope) — useful in callbacks.

**How JS Handles Inheritance**
- Through the prototype chain — objects inherit properties and methods from their prototype object rather than from classes (classes are just syntax sugar over prototypes).

**Which OOP Principle Aligns with Closures?**
- Encapsulation — closures let you keep variables private to a function while still allowing controlled access through returned functions.

**Types of Functions**
- Function declarations, function expressions, arrow functions, anonymous functions, and IIFEs (Immediately Invoked Function Expressions).

**Event Bubbling**
- When an event on a child element also triggers on its parent elements as it "bubbles up" the DOM — can be stopped with `stopPropagation()`.

**Axios vs Fetch**
- `fetch`: built into browsers, needs manual JSON parsing and manual error handling for bad status codes.
- `axios`: a library with automatic JSON parsing, better error handling, and extras like interceptors and timeouts.

**Local Storage vs Session Storage**
- Local Storage: persists even after closing the browser.
- Session Storage: cleared once the tab is closed.

**npm vs npx & package.json**
- `npm`: installs and manages packages.
- `npx`: runs a package's binary without installing it globally.
- `package.json`: lists a project's dependencies, scripts, and metadata.

**WeakMap vs WeakSet**
- Both hold object references "weakly," meaning if there's no other reference to an object, it can be garbage collected even while still a key/member here — unlike `Map`/`Set`, which hold strong references and prevent garbage collection.
- `WeakMap`: key-value pairs where keys must be objects. `WeakSet`: a collection of unique objects.
- Used for attaching metadata to objects (e.g. caching, tracking) without causing memory leaks.

**Function vs Method**
- Function: a standalone block of reusable code, not tied to any object.
- Method: a function that's a property of an object — called on that object (e.g. `obj.method()`), and can access the object via `this`.

**Ways to Create a Function in JavaScript**
- Function declaration (`function foo() {}`), function expression (`const foo = function() {}`), arrow function (`const foo = () => {}`), and the rarely-used `Function` constructor (`new Function(...)`).

**Ways to Create an Object in JavaScript**
- Object literal (`{ key: value }`), constructor function (`new Foo()`), `Object.create()`, ES6 `class`, and factory functions (a regular function that returns a new object).

**Ways to Create a Class in JavaScript**
- ES6 `class` syntax (syntactic sugar over prototypes), a constructor function + manually adding methods to its `.prototype`, `Object.create()` with a prototype object, and factory functions that return objects mimicking class instances (no `new` or `this` needed).

**Event Loop: Microtask vs Macrotask Queue**
- Macrotasks: `setTimeout`, `setInterval`, I/O callbacks — run one at a time, each after a full render cycle.
- Microtasks: Promise callbacks (`.then`), `queueMicrotask` — have **higher priority**: after each single task/callback finishes, JS drains the *entire* microtask queue before moving to the next macrotask (e.g. before the next `setTimeout` callback), so promises consistently run before timeouts even with a `0ms` delay.

**`this` in JavaScript**
- Refers to the object a function is currently being called on — its value depends on *how* a function is called (as a method → the object; alone → `undefined`/global in strict/non-strict mode; with `call`/`apply`/`bind` → whatever you set; in an arrow function → inherited from the surrounding scope, not rebound).

**Call by Value vs Call by Reference (JavaScript)**
- Primitives (string, number, boolean, etc.): passed by value — a copy is made, changes inside a function don't affect the original.
- Objects/arrays: the reference is passed by value — you can mutate the object's contents through that reference, but reassigning the parameter to a brand-new object won't affect the original variable outside.

**Coercion Example: `"1" + 1`**
- Results in `"11"` — the `+` operator sees a string on one side, so it coerces the number `1` into `"1"` and concatenates them, instead of adding numerically.

**Real-World Use Case for a Closure**
- A common one: creating a private counter/cache. E.g. a function `createCounter()` returns an `increment()` function that keeps access to a private `count` variable — the outside world can call `increment()` but can never directly read or reset `count`, giving true data privacy without classes.

**Generator Functions (JavaScript)**
- Declared with `function*`, they can pause execution with `yield` and resume later, producing a sequence of values lazily (one at a time) instead of computing them all upfront — useful for large/infinite sequences or custom iteration logic.

**IIFE — Use Case**
- Used to run setup code once immediately while keeping its variables out of the global scope (avoiding naming collisions) — common in older JS module patterns before ES6 modules existed, e.g. wrapping a whole library/script in one IIFE.

**Garbage Collection (General Concept)**
- The automatic process of finding and freeing memory used by objects that are no longer reachable/referenced by the running program, so developers don't have to manually free memory — used by JS, Python, Java, etc. (with different underlying algorithms like reference counting or mark-and-sweep).

**Compiled vs Interpreted Languages**
- Compiled (e.g. C, C++): source code is translated into machine code ahead of time by a compiler — faster execution, but needs a separate compile step per platform.
- Interpreted (e.g. JavaScript, Python): code is read and executed line-by-line at runtime by an interpreter (though most modern interpreters also JIT-compile hot code for speed) — more flexible, no separate build step, but generally slower for raw computation.

**Hash Collision — Why It Can Sometimes Be a Good Thing**
- A collision happens when two different keys hash to the same slot. It's usually a performance problem (handled via chaining or open addressing), but it can be intentionally useful in things like similarity hashing (e.g. locality-sensitive hashing), where you *want* similar inputs to collide into the same bucket to group them together.

---

## 6. Java

**Stack vs Heap (Memory)**
- Stack: stores method calls and local primitive variables, fast, automatically cleaned up when a method returns.
- Heap: stores objects, shared across the program, managed by the garbage collector.

**How to Handle Exceptions (2 Ways)**
- `try/catch/finally` blocks to catch and handle an exception where it occurs.
- Declaring `throws` on a method to pass the exception up to the caller to handle instead.

**throw vs throws**
- `throw`: actually raises an exception at a specific point in the code.
- `throws`: a keyword in a method signature declaring that the method might throw that exception, so callers must handle it.

**try-with-resources**
- A `try` block that automatically closes resources (like files or DB connections) once it finishes, without needing a manual `finally` block to close them.

**StringBuffer vs StringBuilder**
- Both are mutable string classes (unlike `String`).
- `StringBuffer`: thread-safe (synchronized), slightly slower.
- `StringBuilder`: not thread-safe, faster — use it unless multiple threads modify the same string.

**Handling Multiple Threads**
- Use synchronization (`synchronized` keyword), locks, or thread-safe collections to prevent race conditions when multiple threads access shared data.

**Abstract Class vs Interface (Java)**
- Abstract class: can have both implemented and unimplemented methods, plus fields/state; a class can extend only one.
- Interface: traditionally only method signatures; since Java 8, interfaces can also have `default` methods with a body. A class can implement many interfaces.

**Can an Interface Have a Method with a Body?**
- Yes — since Java 8, interfaces can define `default` (and `static`) methods that include an implementation.

**Functional Interface**
- An interface with exactly one abstract method — used as the target for lambda expressions (e.g. `Runnable`, `Comparator`).

**What is ORM / How Do We Use It**
- Object-Relational Mapping — lets you interact with a database using objects/classes instead of writing raw SQL. You define models, and the ORM generates the SQL behind the scenes (e.g. Hibernate in Java, Django ORM in Python).

**Method Overriding vs Method Overloading**
- Overriding: subclass redefines a parent method with the same signature (runtime polymorphism).
- Overloading: same method name, different parameter lists in the same class (compile-time polymorphism).

**Memory Leak & How to Handle It**
- Happens when objects are no longer needed but still referenced somewhere, so the garbage collector can't free them.
- Fix by removing unnecessary references (e.g. clearing listeners/caches), using weak references where appropriate, and profiling with tools to find leaks.

**SQL ACID Transaction**
- A transaction that follows Atomicity, Consistency, Isolation, and Durability — ensuring reliable database operations even with failures or concurrent access.

**Isolation Levels**
- Control how much one transaction can see of another's uncommitted changes. From loosest to strictest: Read Uncommitted, Read Committed, Repeatable Read, Serializable — stricter levels prevent more anomalies but reduce concurrency.

**Interface Segregation (with Example)**
- Don't force a class to implement methods it doesn't need. Example: instead of one big `Worker` interface with `code()` and `cook()`, split into `Coder` and `Cook` interfaces so a `Coder` class isn't forced to implement `cook()`.

**Checked vs Unchecked Exceptions**
- Checked: must be either caught or declared with `throws` (e.g. `IOException`) — checked at compile time.
- Unchecked: don't need to be declared, usually programming errors (e.g. `NullPointerException`, `ArithmeticException`) — checked at runtime.

**DDL vs DML**
- DDL (Data Definition Language): defines/changes structure — `CREATE`, `ALTER`, `DROP`.
- DML (Data Manipulation Language): works with the data itself — `SELECT`, `INSERT`, `UPDATE`, `DELETE`.

**Cookies vs Session**
- Cookie: small data stored on the client's browser, sent with every request.
- Session: data stored on the server, identified by a session ID (often stored in a cookie) — better for sensitive/larger data.

**Stateless vs Stateful**
- Stateless: server doesn't remember anything about previous requests (e.g. typical REST APIs).
- Stateful: server keeps track of a client's state across requests (e.g. traditional sessions).

**Life Before OOP**
- Programming was mostly procedural: a sequence of functions operating on shared/global data, which became hard to maintain and reuse as programs grew — OOP organized code around objects to fix this.

**Other Ways to Make Something Immutable in Java (Besides `final`)**
- Make all fields `private` and `final`, don't provide setters, and if a field is a mutable object (e.g. a `List`), return a defensive copy from getters instead of the original reference.

**`final` vs Immutable**
- `final`: means a variable's reference can't be reassigned (the object it points to can still be mutated if it's not designed to be immutable).
- Immutable: means the object's actual state can never change after creation — achieved by combining `final` fields with no setters and defensive copying.

**Can You Override a Static Method?**
- No — static methods belong to the class, not an instance, so a subclass can only *hide* it (define a new one with the same signature), not truly override it polymorphically.

**Polymorphism, Overloading & How They're Achieved**
- Polymorphism: same interface, different underlying behavior. Achieved via method overriding (runtime) and method overloading (compile-time).

---

## 7. TypeScript

**What is TypeScript**
- A superset of JavaScript that adds static typing, catching type errors at compile time instead of at runtime.

**Any vs Unknown**
- `any`: turns off type checking completely — unsafe.
- `unknown`: also accepts any value, but forces you to check/narrow the type before using it — safer.

**Type vs Interface**
- Both describe the shape of an object.
- `interface`: can be extended and re-opened/merged later — preferred for objects.
- `type`: more flexible (unions, primitives, tuples), but can't be reopened once defined.

**Union vs Intersection**
- Union (`A | B`): a value can be type A **or** type B.
- Intersection (`A & B`): a value must satisfy type A **and** type B combined.

**Optional Operator (`?`)**
- Marks a property or parameter as optional — it may be `undefined`.

**OR (`||`) vs Nullish Coalescing (`??`)**
- `||`: falls back for any falsy value (`0`, `""`, `false`, `null`, `undefined`).
- `??`: falls back only for `null` or `undefined` — safer when `0` or `""` are valid values.

**How TypeScript is Compiled**
- The TypeScript compiler (`tsc`) checks types, then strips them out and converts `.ts` files into plain `.js`, configured via `tsconfig.json`.

**How to Make an Attribute Required**
- Just don't mark it with `?` — properties are required by default unless explicitly made optional.

---

## 8. React

**useState vs useRef**
- `useState`: stores data that, when changed, re-renders the component.
- `useRef`: stores data that persists across renders but does **not** trigger a re-render when changed. Also used to directly access DOM elements.

**useMemo vs useCallback vs React.memo**
- `useMemo`: caches a computed **value** so it's not recalculated on every render.
- `useCallback`: caches a **function** so it's not recreated on every render.
- `React.memo`: wraps a whole component to skip re-rendering if its props haven't changed.

**useContext vs Redux**
- `useContext`: simple way to share data across components without prop drilling, built into React.
- Redux: a full state management library with a single global store, useful for large/complex apps.
- The problem with `useContext` alone: every consumer re-renders on any context change, and there's no built-in tooling for debugging/organizing complex state like Redux has.

**State vs Props**
- State: data owned and managed *inside* a component, can change over time.
- Props: data passed *into* a component from its parent, read-only inside the child.

**Props Drilling**
- Passing props down through many nested components just to reach a deeply nested child — solved with Context or state management libraries.

**Component Lifecycle (React)**
- Mounting (component created and added to DOM), Updating (re-renders on state/props change), Unmounting (component removed). In function components, handled with `useEffect`.

**Virtual DOM**
- A lightweight JS copy of the real DOM. React updates the Virtual DOM first, compares it to the previous version (diffing), then updates only the changed parts of the real DOM — faster than updating the whole page.

**DOM vs BOM**
- DOM: represents the HTML page structure (elements, tags).
- BOM: represents the browser itself (window, navigator, location, history).

**What is React / Why Use It**
- A JavaScript library for building UIs from reusable components. Benefits: component reusability, the Virtual DOM for fast updates, one-way data flow, and a huge ecosystem.

**React: Library or Framework?**
- A library — it only handles the view/UI layer, and you add your own tools for routing, state management, etc. (Angular is a full framework by contrast).

**Hooks**
- Functions that let function components use React features like state and lifecycle (e.g. `useState`, `useEffect`, `useRef`, `useContext`, `useMemo`) without writing class components.

**useEffect**
- Runs side effects (data fetching, subscriptions, DOM updates) after render. It's how function components replicate class lifecycle methods (mount, update, unmount).

**Dependency Array (in useEffect)**
- Controls when the effect re-runs:
  - `[]` empty: runs once after the first render (like mount).
  - `[value]`: runs whenever `value` changes.
  - no array: runs after every render.

**Replicating Class Lifecycle with useEffect**
- Mount → `useEffect(fn, [])`; Update → `useEffect(fn, [deps])`; Unmount → return a cleanup function inside `useEffect`.

**JSX**
- A syntax that lets you write HTML-like markup inside JavaScript; it compiles down to `React.createElement()` calls.

**Why Use `key` in Lists**
- Keys give each list item a stable identity so React can efficiently tell which items changed, were added, or removed — avoiding unnecessary re-renders and bugs.

**How React Enhances Performance**
- Virtual DOM diffing (updates only what changed), memoization (`React.memo`, `useMemo`, `useCallback`), lazy loading/code splitting, and reconciliation with keys.

**How to Make Global State (Redux vs useContext — Which Is Better?)**
- `useContext`: simplest option, built-in, fine for small-to-medium apps or state that doesn't change often.
- Redux: better for large apps with complex/frequent state updates — gives predictable state updates, dev tools, and avoids the "every consumer re-renders" issue of Context.
- Neither is universally "better" — pick based on app size and how often/complex the shared state changes.

**Can You Return a Function Inside `useEffect`?**
- Yes — that returned function is a cleanup function, automatically run when the component unmounts (or before the effect re-runs), used to cancel subscriptions, clear timers, or remove event listeners.

**Optimizing React App Performance**
- `useMemo`/`useCallback` to avoid unnecessary recalculations/re-creations, `React.memo` to skip re-rendering unchanged components, code-splitting/lazy loading, and using stable `key`s in lists.

**Memoization (General Concept)**
- Caching the result of an expensive function call so that if it's called again with the same inputs, the cached result is returned instantly instead of recomputing — `useMemo`/`useCallback`/`React.memo` are all just specific applications of this general idea in React.

**What is Redux**
- A predictable global state management library: all app state lives in a single store, components read from it, and the only way to change it is by dispatching an "action" that a "reducer" function processes into a new state — makes state changes traceable and centralized instead of scattered across components.

**React Component Lifecycle Methods (Class Components)**
- Mounting: `constructor()` → `render()` → `componentDidMount()`.
- Updating: `render()` → `componentDidUpdate()` (runs after props/state change).
- Unmounting: `componentWillUnmount()` (cleanup, like `useEffect`'s cleanup function).
- Error handling: `componentDidCatch()` (used to build error boundaries).

**Class Component — Full Anatomy**
- `constructor(props)`: sets up initial `this.state` and binds methods.
- `this.state`: the component's local data.
- `this.setState({...})`: updates state and triggers a re-render (batched, asynchronous).
- `this.props`: read-only data passed from the parent.
- `render()`: returns the JSX to display — the only required method.
- Lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`): handle side effects at different points, replaced by hooks in function components.

**Hook → Class Component Equivalents**
- `useState` → `this.state` + `this.setState()`.
- `useEffect` (mount) → `componentDidMount()`.
- `useEffect` (with deps, on update) → `componentDidUpdate()` (with manual prop/state comparison).
- `useEffect` (cleanup) → `componentWillUnmount()`.
- `useContext` → `static contextType = MyContext` or `<MyContext.Consumer>`.
- `useRef` → `React.createRef()` (set up in the constructor).
- `useReducer` → `this.state` managed through a reducer-style function you call manually inside `setState`.
- `useMemo` / `useCallback` → no direct class equivalent — you'd manually cache values/functions (e.g. in the constructor or with instance variables).

---

## 9. Angular

**What is Angular**
- A full frontend framework (by Google) for building single-page applications, using TypeScript, components, and data binding out of the box.

**Angular vs React**
- Angular: a full framework with built-in routing, forms, HTTP client, and dependency injection, using TypeScript by default.
- React: a lighter UI library (just the view layer) — you choose your own tools for routing/state management.

**Component**
- A self-contained building block of an Angular app — combines a template (HTML), logic (TypeScript class), and styles.

**Component vs Module**
- Component: a single UI building block.
- Module (`NgModule`): groups related components, services, and other code together into one cohesive unit of the app.

**Data Binding & Types**
- Keeps the UI and underlying data in sync. Types:
  - Interpolation (`{{ }}`): one-way, data → view.
  - Property binding (`[ ]`): one-way, data → view.
  - Event binding (`( )`): one-way, view → data.
  - Two-way binding (`[( )]`): both directions at once.

**ngModules**
- Angular's way of organizing an app into cohesive blocks of functionality (declares which components/services belong together).

**Services**
- Reusable classes that hold logic/data (e.g. API calls) shared across components, provided through Angular's dependency injection.

**Can You Build a Single Page App with Angular?**
- Yes — that's Angular's core design goal, handling routing/navigation without full page reloads.

**JIT vs AOT (Angular Compilation)**
- JIT (Just-In-Time): compiles the app in the browser at runtime — slower startup.
- AOT (Ahead-Of-Time): compiles the app during the build step, before it ships — faster startup, smaller bundle, used by default in production.

**Directives**
- Instructions that tell Angular to change/manipulate the DOM. Types:
  - Structural (`*ngIf`, `*ngFor`): add/remove elements.
  - Attribute (`ngClass`, `ngStyle`): change an element's appearance/behavior.

**Pipes**
- Transform data directly in the template for display (e.g. `{{ price | currency }}`, `{{ date | date }}`) without changing the underlying value.

**Interpolation**
- One-way binding that inserts a component's data into the template using `{{ }}`.

**Lifecycle Hooks**
- Methods Angular calls at key moments of a component's life, e.g. `ngOnInit` (after init), `ngOnChanges` (on input change), `ngOnDestroy` (before removal).

**FormModule Types (Template-driven vs Reactive Forms)**
- Template-driven: forms built mostly in the HTML template, simpler, good for basic forms.
- Reactive: forms defined and controlled in the TypeScript code, more scalable and testable, better for complex/dynamic forms.

**Observables vs Promises**
- Promise: represents a single future value, executes immediately once created, can't be cancelled.
- Observable (RxJS, used heavily in Angular): represents a stream of multiple values over time, is lazy (doesn't run until subscribed to), can be cancelled (unsubscribed), and comes with powerful operators (`map`, `filter`, `switchMap`) for transforming the stream.

**Interceptors**
- Functions that sit between your app and every outgoing HTTP request / incoming response, letting you globally modify requests (e.g. attach an auth token to every call) or handle responses (e.g. catch 401 errors everywhere) without repeating that logic in every API call. Available in Angular's `HttpClient` and in libraries like Axios.

---

## 10. Node.js

**What is Node.js**
- A runtime that lets you run JavaScript outside the browser (e.g. on a server), built on Chrome's V8 engine.

**Node.js Event Loop**
- Lets Node handle many operations (file/network I/O) without blocking, by delegating them and running their callbacks once they're done.
- Has phases (timers, pending callbacks, poll, check, close callbacks) and prioritizes microtasks (Promises, `process.nextTick`) between phases.

**V8 & Libuv**
- V8: executes the JavaScript code itself.
- Libuv: handles async I/O and manages the thread pool and event loop under the hood.

**Is Node.js Single or Multi-threaded?**
- JavaScript itself runs on a single thread, but Node uses libuv's thread pool (4 threads by default, configurable) behind the scenes for things like file system operations.

**Parallelism vs Concurrency**
- Concurrency: handling multiple tasks by switching between them (not necessarily at the exact same instant).
- Parallelism: truly running multiple tasks at the same time (needs multiple cores/threads).

**Module.exports vs exports**
- `module.exports`: the actual object returned by `require()`.
- `exports`: just a shortcut reference to it — reassigning `exports` directly breaks the link, so use `module.exports` to be safe.

**Middleware**
- A function that runs between receiving a request and sending a response — used for things like auth checks, logging, or parsing request data.
- Calling `next()` passes control to the next middleware; without it, the request hangs and never completes.

**EventEmitter**
- A Node.js class that lets objects emit named events and other code "listen" and react to them — the basis of the Observer/Pub-Sub pattern in Node.

**RESTful API**
- An API design style using standard HTTP methods and URLs to represent and manipulate resources (e.g. `GET /users/1`).

**HTTP Methods**
- GET (read), POST (create), PUT (replace), PATCH (partial update), DELETE (remove).

**PUT vs PATCH**
- PUT: replaces the entire resource.
- PATCH: updates only the specified fields.

**Common Status Codes**
- 201: Created successfully.
- 403: Forbidden (authenticated but not allowed).
- 500: Internal Server Error (something broke on the server).

**Stateful vs Stateless**
- Stateless (e.g. REST APIs): server doesn't remember previous requests — each request carries all the info it needs.
- Stateful: server keeps track of client state between requests (e.g. sessions).

**spawn vs fork (Child Processes)**
- `spawn`: launches a new process to run any command, streams its output — good for large data/long-running processes.
- `fork`: a special case of spawn that creates a new Node process with a built-in communication channel — used to run separate Node scripts.

**Stream vs Buffer**
- Buffer: a temporary chunk of binary data held fully in memory.
- Stream: processes data piece by piece as it arrives (read/write in chunks) — memory-efficient for large files.

**Blocking vs Non-Blocking**
- Blocking: code waits for an operation to finish before moving on (synchronous).
- Non-blocking: code moves on and handles the result later via a callback/promise (asynchronous) — Node prefers this to stay responsive.

**Error-Handling Middleware**
- A special Express middleware with four arguments `(err, req, res, next)`. Express routes errors to it when `next(err)` is called or an error is thrown, centralizing error responses.

**Query vs Params**
- Route params (`/users/:id`): part of the URL path, identify a specific resource.
- Query params (`/users?role=admin`): come after `?`, used for filtering, sorting, pagination.

**Request & Response Objects**
- Request (`req`): carries incoming data — URL, headers, body, params, query, cookies.
- Response (`res`): used to send data back — status code, headers, and the response body.

**I/O-bound vs CPU-intensive Tasks in Node**
- Node excels at I/O-bound work (many concurrent network/file operations) thanks to its non-blocking event loop.
- It's poor at CPU-intensive work (heavy computation), since that blocks the single main thread — offload those to worker threads or child processes.

**Can I Make Node.js Multi-threaded?**
- Yes — using the `worker_threads` module for CPU-heavy tasks, or the `cluster` module to run multiple Node processes (one per CPU core) that share the same server port.

**How to Use `.env` (Environment Variables)**
- Store secrets/config (API keys, DB URLs) in a `.env` file, then load them with a package like `dotenv` (`require('dotenv').config()`) so they're available via `process.env.VAR_NAME` without hardcoding them in the code.

**Node Code Flow / How Node Runs Code**
- V8 executes the JS; async operations are handed to libuv, which uses the event loop and thread pool, then queues their callbacks to run once the main stack is clear.


## 11. Django & Backend

**Request Lifecycle in Django**
- Request hits `urls.py` → matched to a view → view uses `models.py`/serializers to get data → returns a `response` → Django middleware processes it on the way in and out → `settings.py` configures the whole project (apps, database, middleware, etc.).

**select_related vs prefetch_related**
- `select_related`: for Foreign Key / One-to-One — does a SQL JOIN in a single query. Use when the related object is "one".
- `prefetch_related`: for Many-to-Many / reverse Foreign Key — runs a separate query and joins in Python. Use when the related objects are "many".

**N+1 Problem**
- Happens when you fetch a list of objects, then run a separate query for each one's related data (1 + N queries). Fixed with `select_related`/`prefetch_related` to combine queries.

**Serialization (in/out)**
- Serialization: converting Python/Django objects into JSON to send to the client (output).
- Deserialization: converting incoming JSON back into Python objects/validated data (input).

**Types of Views**
- Function-Based Views (FBV): plain Python functions.
- Class-Based Views (CBV): classes with methods per HTTP verb (get, post, etc.).
- ViewSets: group related views (list, create, update, delete) into one class, often paired with routers.
- `.as_view()`: converts a class-based view into a callable function Django's URL router can use.

**makemigrations vs migrate**
- `makemigrations`: generates migration files based on model changes (doesn't touch the DB).
- `migrate`: actually applies those migration files to the database.

**Handling Migration Conflicts**
- Happens when two branches create conflicting migration files. Fix by merging with `makemigrations --merge`, or manually reordering/renaming migration dependencies.

**CSRF Tokens / What is CORS**
- CSRF token: a unique token sent with forms/requests to prove the request came from your own site, protecting against Cross-Site Request Forgery.
- CORS (Cross-Origin Resource Sharing): a browser security rule that blocks a webpage from calling an API on a different origin unless that API explicitly allows it via response headers.

**Signals in Django**
- Let one part of the app "listen" and react automatically when something happens elsewhere (e.g. send an email after a `User` is saved), without directly coupling the code together.

**ASGI vs WSGI**
- WSGI: synchronous, handles one request at a time — standard for traditional Django.
- ASGI: supports async and WebSockets, handles many requests concurrently — used for real-time features.

**Django vs Express**
- Django: Python, "batteries-included" (ORM, admin, auth built in), more opinionated/structured.
- Express: Node.js, minimal/unopinionated, you add libraries yourself for ORM, auth, etc.

**Multi-session Django**
- Handling multiple simultaneous logged-in sessions per user, usually via session/token management and storing session data per device.

**python manage.py CLI commands**
- Common ones: `runserver`, `makemigrations`, `migrate`, `createsuperuser`, `shell`, `startapp`, `test`.

**Django vs FastAPI vs Flask — When to Use Each**
- Django: "batteries-included" — built-in ORM, admin panel, auth, and structure. Best for full-featured apps where you want a lot of functionality out of the box (e.g. content-heavy or admin-driven apps).
- Flask: minimal microframework — you add only what you need. Best for small apps or APIs where you want full control over structure and dependencies.
- FastAPI: modern, async-first, type-hint-driven (auto validation via Pydantic + auto-generated docs). Best for high-performance APIs, especially ones needing async I/O or heavy data validation.

**REST Resource Naming (Why `/users/1`, Not `/clients/1`)**
- REST convention is to name endpoints after resources using plural nouns (`/users`, `/orders`), not domain-specific or singular/ambiguous terms — this keeps the API predictable and consistent as more resource types are added, rather than mixing naming styles per feature.

---

## 12. Databases & SQL

**What is a Database / Types of Databases**
- A database is an organized, persistent store of data you can query and manage.
- Main types: Relational (SQL — tables with fixed schema, e.g. PostgreSQL/MySQL) and Non-relational (NoSQL — flexible documents/key-value/graph, e.g. MongoDB/Redis).

**Relational vs Non-Relational**
- Relational: structured tables with defined relationships and a fixed schema, uses SQL.
- Non-relational: flexible/schema-less, scales horizontally, better for unstructured or rapidly changing data.

**Common SQL Queries**
- `SELECT` (read), `INSERT` (create), `UPDATE` (modify), `DELETE` (remove), plus `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY` clauses.

**SQL vs NoSQL**
- SQL: structured tables, fixed schema, relationships, strong consistency (e.g. PostgreSQL).
- NoSQL: flexible schema, documents/key-value/graph, scales easily, better for unstructured/fast-changing data (e.g. MongoDB).

**ACID Principles**
- Atomicity: a transaction fully happens or not at all.
- Consistency: DB moves from one valid state to another.
- Isolation: concurrent transactions don't interfere with each other.
- Durability: once committed, data survives even a crash.

**Normalization vs Denormalization**
- Normalization: splitting data into related tables to reduce duplication — good for data integrity.
- Denormalization: combining data back for faster reads — good when read speed matters more than storage/duplication (e.g. reporting/dashboards).

**Indexing**
- Advantage: much faster lookups/searches on indexed columns.
- Disadvantage: slower writes (insert/update/delete) since indexes must update too, and it uses extra storage.
- Use case: index columns you filter/search/sort by often (e.g. `email`, `user_id`).

**Primary vs Clustered vs Non-Clustered Index**
- Primary index: on the primary key, uniquely identifies rows.
- Clustered index: physically sorts and stores table data in index order — only one per table.
- Non-clustered index: a separate lookup structure pointing to the actual rows — a table can have many.

**Composite/Compound Index Order**
- Yes, order matters — the index is only used efficiently if the query filters on its fields starting from the leftmost one in the defined order.

**Aggregate Functions vs Window Functions**
- Aggregate (`SUM`, `COUNT`, `AVG`): collapses multiple rows into one result.
- Window functions: perform calculations across rows *without* collapsing them — you still see every row, plus the calculated value (e.g. running total).

**LIKE vs ILIKE**
- `LIKE`: case-sensitive pattern matching. `ILIKE`: case-insensitive version (PostgreSQL).

**Data Warehousing**
- A large central database optimized for analysis/reporting (not daily transactions), usually combining data from multiple sources.

**Solutions to Speed Up DB Retrieval / Optimizing a Slow Query**
- Add indexes, use caching (Redis), optimize/avoid N+1 queries, use pagination, denormalize where needed, use read replicas.

**Validations vs Constraints**
- Validation: checked in application code (e.g. Django form/serializer) before saving.
- Constraint: enforced by the database itself (e.g. `NOT NULL`, `UNIQUE`, foreign keys) — the last line of defense even if app code is bypassed.

**DML vs DDL vs DCL**
- DML (Data Manipulation Language): works with data — `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- DDL (Data Definition Language): defines/changes structure — `CREATE`, `ALTER`, `DROP`.
- DCL (Data Control Language): manages access/permissions — `GRANT`, `REVOKE`.

**How to Get Duplicates from a Table**
- Group rows by the column(s) you're checking, then filter groups with a count greater than 1: `GROUP BY column HAVING COUNT(*) > 1`.

**HAVING vs WHERE (and Which Runs First)**
- `WHERE`: filters individual rows *before* grouping.
- `HAVING`: filters groups *after* `GROUP BY`/aggregation.
- Execution order: `WHERE` runs first, then `GROUP BY`, then `HAVING`.

**UNION vs UNION ALL**
- `UNION`: combines results from two queries and removes duplicate rows.
- `UNION ALL`: combines results and keeps all rows, including duplicates — faster since it skips the dedup step.

**DELETE vs TRUNCATE (vs DROP)**
- `DELETE`: removes rows one by one (can use `WHERE`), logged, can be rolled back, doesn't reset auto-increment/primary key sequence.
- `TRUNCATE`: removes all rows at once, faster, minimally logged, usually resets the auto-increment sequence, harder/impossible to roll back in some databases.
- `DROP`: removes the entire table structure, not just the data.

**SQL Views & Types**
- A view is a saved, reusable query that behaves like a virtual table.
- Types: simple views (single table, updatable) and complex views (joins/aggregations, often read-only).

**Example: 1-to-Many Query (Contacts → Numbers)**
- To get 10 users with a distinct phone number: join `contacts` to `numbers`, then use `SELECT DISTINCT` on the phone number column (or `GROUP BY` phone number) with `LIMIT 10` to cap the results.

**Types of Relationships in SQL**
- One-to-One: each row in Table A relates to exactly one row in Table B (e.g. a `User` and their `Profile`).
- One-to-Many: one row in Table A relates to many rows in Table B (e.g. one `Author` has many `Books`).
- Many-to-Many: rows in both tables can relate to multiple rows in the other, implemented via a junction/join table (e.g. `Students` and `Courses`, linked through an `Enrollments` table).

**Why Use Joins Instead of Nested (Subquery) Queries**
- Joins let the database's query optimizer plan and execute the combination of tables in one efficient pass (often using indexes well), while deeply nested subqueries can force the database to run a query repeatedly for each outer row — generally slower and harder for the optimizer to reason about, though modern optimizers can sometimes rewrite simple subqueries into joins automatically.

**Database Engines (e.g. InnoDB vs MyISAM in MySQL)**
- A storage engine determines how a database physically stores, indexes, and retrieves data, and what features it supports.
- InnoDB: supports transactions, foreign keys, and row-level locking — the modern default for MySQL.
- MyISAM: faster for simple read-heavy workloads, but no transaction support and only table-level locking — largely legacy now.

---

## 13. MongoDB & NoSQL

**.explain() Method**
- Shows how MongoDB actually executed a query (e.g. whether it used an index or scanned the whole collection) — the go-to tool for diagnosing slow queries.

**Types of Indexes in MongoDB**
- Single field, compound (multiple fields), multikey (for array fields), text (for text search), and geospatial (for location data).

**Aggregation Methods**
- MongoDB's aggregation pipeline (`$match`, `$group`, `$sort`, `$project`, etc.) transforms and summarizes data across multiple stages.

**Pagination in MongoDB**
- Typically `.skip()` and `.limit()` for simple cases, or cursor-based pagination (using a field like `_id`) for better performance on large collections.

**Find / Insert / Delete Documents**
- `find()` / `findOne()` to read, `insertOne()` / `insertMany()` to create, `deleteOne()` / `deleteMany()` to remove documents.

**Upsert**
- "Update or insert" — updates a document if it matches the query, or creates a new one if no match is found.

**updateOne() vs findOneAndUpdate()**
- `updateOne()`: updates and returns a status/count only.
- `findOneAndUpdate()`: updates and also returns the actual document (before or after the update, your choice).

**$set vs $unset**
- `$set`: adds or changes a field's value.
- `$unset`: removes a field entirely from the document.

**Embedding vs Referencing**
- Embedding: stores related data inside the same document — fast reads, good for data always used together.
- Referencing: stores an ID pointing to another document — better for large, shared, or frequently-changing data.

**Primary vs Secondary Node**
- In a MongoDB replica set, the Primary handles all writes; Secondaries replicate the Primary's data and can serve read traffic.

**Replication vs Sharding**
- Replication: copies the same data across multiple servers for reliability/availability.
- Sharding: splits (partitions) data across multiple servers for scalability.

**CAP Theorem**
- A distributed database can only fully guarantee 2 of 3 at once: Consistency, Availability, Partition tolerance — you choose trade-offs based on your needs.

**BASE vs ACID**
- ACID (SQL): strict consistency guarantees.
- BASE (NoSQL, incl. MongoDB): Basically Available, Soft state, Eventually consistent — trades strict consistency for availability/scalability.

**Structure of ObjectID**
- A 12-byte value encoding a timestamp, a random/machine identifier, and a counter — making it unique and roughly sortable by creation time.

**Vertical vs Horizontal Scaling**
- Vertical: making one server bigger/stronger.
- Horizontal: adding more servers to share the load. MongoDB is built to scale well horizontally (sharding + load balancing).

**Types of Query Selectors**
- Comparison (`$gt`, `$lt`), logical (`$and`, `$or`), and element (`$exists`, `$type`) — used to filter documents.

**Projection**
- Choosing which fields to return from a query instead of the whole document, to reduce data transferred.

**Cursor**
- A pointer to a query's result set — lets you iterate through results (often in batches) instead of loading everything into memory at once.

**Cursor Count vs Document Count**
- Cursor count reflects how many documents the cursor would return for the query (respecting limits/skips), while a plain document count can reflect the total matching documents in the collection.

**Mongoose .populate() ("Joins" via References)**
- Replaces a referenced ObjectId field with the actual related document — Mongoose's way of simulating a SQL JOIN across collections.

**Virtuals & Populate (Mongoose)**
- Virtuals: computed fields that aren't stored in the DB but derived from existing data (e.g. `fullName` from first + last).
- Populate: fills in referenced documents, as above.

**Can You Create a View in MongoDB?**
- Yes — MongoDB supports read-only views defined by an aggregation pipeline on a source collection.

**Primary Key in MongoDB**
- Every document has a unique `_id` field as its primary key. MongoDB auto-generates an ObjectId, but you can define your own `_id` value.

**BSON (Physical Storage)**
- MongoDB stores documents physically as BSON (Binary JSON) — a binary-encoded format that's more compact and supports more data types than plain JSON.

---

## 14. Data Structures

**Stack vs Queue**
- Stack: Last-In-First-Out (like a stack of plates).
- Queue: First-In-First-Out (like a line of people).

**Array vs Linked List**
- Array: fixed/contiguous memory, fast random access (O(1)), slow insert/delete in the middle.
- Linked List: nodes linked via pointers, slow access (O(n)), fast insert/delete once you're at the right node.

**Types of Linked Lists**
- Singly (one direction), Doubly (both directions), Circular (last node points back to the first).

**Hash Map — How It Works**
- Stores key-value pairs; a hash function converts the key into an index, so lookups, inserts, and deletes are close to O(1) on average.

**Binary Search Tree (BST)**
- A tree where each node's left child is smaller and right child is larger — enables fast search/insert/delete, O(log n) if balanced.
- Removing the root: usually replace it with its in-order successor (smallest value in the right subtree) or predecessor, then fix up the tree structure.

---

## 15. Git

**What Are Branches in Git**
- Independent lines of development that let you work on features/fixes without affecting the main codebase until you're ready to merge.

**When to Create a New Branch**
- Typically for each new feature, bug fix, or experiment — keeps the main branch stable and makes changes easy to review/merge or discard.

**Git Revert — Can You Revert to Any Commit?**
- Yes, `git revert <commit>` can target any past commit — it creates a new commit that undoes that commit's changes without deleting history.

- `reset`: moves the branch pointer back, can delete commit history — rewrites history, risky on shared branches.
- `revert`: creates a *new* commit that undoes a previous commit — safe for shared/public branches.

**Fetch vs Pull**
- `fetch`: downloads new commits from remote but doesn't merge them into your branch.
- `pull`: does `fetch` + automatically merges into your current branch.

**Git Stash**
- Temporarily saves uncommitted changes so you can switch branches with a clean working directory, then bring them back later.

**Git Stash Apply vs Pop**
- `stash apply`: re-applies the stashed changes but keeps them saved in the stash list.
- `stash pop`: re-applies the changes and removes them from the stash list.

**Git Cherry-pick**
- Applies one specific commit from another branch onto your current branch, without merging the whole branch.

**Rebase vs Merge**
- `merge`: combines branches and keeps full history (adds a merge commit).
- `rebase`: replays your commits on top of another branch — creates a cleaner, linear history but rewrites commit hashes.

**2 Ways to Undo Changes**
- `git checkout -- <file>` (or `git clean`) to discard uncommitted changes permanently, or `git stash` to save them aside temporarily without losing them.

**.gitignore File**
- A file listing paths/patterns Git should ignore (not track), e.g. `node_modules/`, `.env`, build folders — keeps secrets and generated files out of the repo.

**Handling Merge Conflicts**
- Git marks conflicting lines in the file; manually edit to keep the correct code, remove the conflict markers, then `git add` and commit (or continue the rebase/merge).

**Branch Not Found Locally / Can't Checkout to Main**
- Run `git fetch` to update your local knowledge of remote branches, then `git checkout <branch>`. If checkout to main fails, check for uncommitted changes first — stash or commit them.

---

## 16. HTML & CSS

**Semantic Tags**
- HTML tags that describe their meaning, not just appearance (e.g. `<header>`, `<article>`, `<nav>`) — better for accessibility and SEO than generic `<div>`s.

**SEO (Search Engine Optimization)**
- Practices that help a page rank higher in search results — semantic HTML, meaningful titles/meta tags, fast load times, alt text on images, and mobile responsiveness.

**HTML4 vs HTML5**
- HTML5 added semantic elements (`<header>`, `<nav>`, `<article>`), native audio/video, canvas, local storage, and form improvements — HTML4 relied on generic `<div>`s and plugins (like Flash) for media.
- HTML5's semantic structure directly helps SEO by letting search engines understand page content better.

**CSS Specificity**
- The rule that decides which conflicting CSS style wins. Rough order (low → high): element selectors < classes/attributes < IDs < inline styles (`!important` overrides all).

**Flex: justify-content vs align-items**
- `justify-content`: aligns items along the **main axis** (horizontal by default).
- `align-items`: aligns items along the **cross axis** (vertical by default).

**Flex vs Grid**
- Flexbox: one-dimensional layout (row OR column) — great for aligning items in a line.
- Grid: two-dimensional layout (rows AND columns) — great for full page/section layouts.

**px vs rem vs em**
- `px`: fixed size, doesn't scale.
- `em`: relative to the parent element's font size.
- `rem`: relative to the root (`html`) font size — more predictable than `em`.

**Inline vs Block**
- Block: takes full width, starts on a new line (e.g. `<div>`).
- Inline: takes only as much width as needed, stays in line with text (e.g. `<span>`).

**alt Attribute**
- Provides alternative text for images — used by screen readers (accessibility) and shown if the image fails to load.

**Script Tag Placement**
- Put `<script>` at the bottom of `<body>` (or use `defer`) so the page's HTML loads and renders first before JS blocks it.

**Transition vs Animation**
- Transition: smoothly changes a property between two states, needs a trigger (e.g. hover).
- Animation: can have multiple steps/keyframes and run automatically/repeatedly without a trigger.

**Inline Styles vs `<style>` Tag vs External CSS (performance)**
- External CSS files are generally best: they're cached by the browser across pages, keep HTML clean.
- Inline styles are fastest to apply for a single element but not reusable/cacheable and hurt maintainability.

**Lazy Loading**
- Delays loading of resources (images, components) until they're actually needed (e.g. when scrolled into view) — improves initial page load speed.

## 17. Auth & Security

**Authentication vs Authorization**
- Authentication: verifying **who** you are (login — checking credentials).
- Authorization: verifying **what** you're allowed to do (permissions/roles after logging in).

**How to Authenticate a User**
- User submits credentials → server verifies them → server issues a token (e.g. JWT) → client sends that token with future requests → server verifies the token to confirm identity on each request.

**Handling Users with Different Authorization**
- Assign roles/permissions to users (e.g. admin, editor, viewer), store them (often inside the JWT payload), and check the role on each protected endpoint before allowing the action.

**Token vs JWT**
- Token: any string used to authenticate a request (could be random/opaque).
- JWT (JSON Web Token): a specific type of token that encodes data (claims) directly inside it, signed so it can be verified without a database lookup.

**JWT Contents & Decoding**
- Contains 3 parts: Header (algorithm info), Payload (claims/data), Signature (verifies it wasn't tampered with).
- Yes, the payload **can** be decoded by anyone (it's just Base64, not encrypted) — never put secrets in it. Only the signature prevents tampering, not reading.

**JWT Advantages/Disadvantages**
- Advantage: stateless (no DB lookup needed to verify), works well across services.
- Disadvantage: hard to revoke/invalidate before expiry, payload is readable by anyone who has it.

**JWT vs Session Tokens**
- Session token: a random opaque ID; the server looks it up in a session store (DB/Redis) on every request to find the associated user data — stateful, easy to revoke instantly (just delete the session).
- JWT: self-contained and signed; the server verifies it without a lookup — stateless, scales better across servers, but harder to revoke before it naturally expires (since the server doesn't "remember" issued tokens).

**Full JWT Auth Flow (Token Creation, Storage, Comparison)**
- Signup: user's password is hashed (e.g. bcrypt) and stored — the plain password is never saved.
- Login: user submits email/password → server hashes the submitted password and compares it to the stored hash (never compares plain text) → if it matches, the server creates a JWT (signs the header + payload with a secret key) and returns it to the client.
- Storage: client stores the token (ideally in an `httpOnly` cookie, or in memory for SPAs).
- Subsequent requests: client sends the token (usually in the `Authorization` header) → server verifies the signature using its secret key to confirm the token wasn't tampered with and hasn't expired, then trusts the payload's claims (e.g. user ID, role) without hitting the database again.

**Login Page — Full Flow (Backend + Frontend)**
- Frontend: user submits the login form → an API call sends credentials to the backend → on success, the frontend stores the returned token and redirects/updates UI state (e.g. into a global auth context) → on failure, shows an error message.
- Backend: receives credentials → looks up the user by email → hashes the submitted password and compares to the stored hash → if valid, issues a JWT (and optionally a refresh token) → returns them in the response; every future protected request from the frontend includes the token for the backend to verify.

**Access Token vs Refresh Token**
- Access token: short-lived, used to access protected resources.
- Refresh token: long-lived, used only to get a new access token when it expires, without forcing the user to log in again.
- Use case for multiple access tokens: different scopes/permissions per device or per API integration, or limiting blast radius if one token leaks.

**JWT vs OAuth**
- JWT: a token *format*.
- OAuth: an *authorization protocol/flow* for granting access (e.g. "Login with Google") — OAuth can use JWTs internally, but they solve different problems.

**Hash vs Encryption / Encoding vs Decoding**
- Hashing: one-way, can't be reversed — used for passwords.
- Encryption: two-way, can be decrypted with the right key — used for data you need to read back later.
- Encoding/Decoding: just reformats data (e.g. Base64) for safe transport — not meant for security at all, easily reversible by anyone.

**Types of Hashing Methods**
- MD5, SHA-1, SHA-256, and password-specific ones like bcrypt/Argon2 (which add salting and are slow on purpose to resist brute-force).

**Local Storage vs Session Storage vs Cookies**
- Local Storage: persists even after closing the browser, ~5-10MB, accessible via JS only.
- Session Storage: cleared when the tab closes, same size limits.
- Cookies: small (~4KB), sent automatically with every HTTP request, can be set to `httpOnly` (safer, JS can't read them).

**Login Flow — Where to Save Tokens**
- Typical flow: user submits credentials → server verifies → server returns access + refresh tokens.
- Store access token in memory (safest) or `httpOnly` cookie; avoid `localStorage` for sensitive tokens since it's exposed to XSS attacks.

**Webhook**
- A way for one system to automatically notify another system by sending an HTTP request when an event happens (instead of the other system having to keep asking/polling).

**Controlling Permissions (Backend)**
- Use role-based or permission-based checks (e.g. Django's permission classes) on each endpoint to confirm the authenticated user is allowed to perform that action.

**Preventing Duplicate Payment on Double-Click**
- Use a short-lived lock/flag in Redis keyed by user+action when the request starts; reject any duplicate request while that lock exists, release it once processed.

**Securing a Login-Based Application**
- Use JWTs for stateless auth, store the token in an `httpOnly` cookie (so JS/XSS can't read it), send it automatically via the browser with each request header, and validate/verify it server-side on every protected route.

**API Versioning (Adding a New Feature Without Breaking Existing Clients)**
- Introduce a new version (e.g. `/api/v2/...` or a version header) instead of changing the existing endpoint, so old clients keep working against `v1` while new clients use `v2`.

**SQL Injection & How to Prevent It**
- SQL Injection: an attacker inserts malicious SQL through user input (e.g. a form field) to manipulate or steal data.
- Prevention: always use parameterized queries/prepared statements (never string-concatenate user input into SQL), validate/sanitize input, and use an ORM which handles this by default.

---

## 18. System Design, Performance & Deployment

**Optimizing a Slow Endpoint (6s → 2s)**
- Profile first to find the real bottleneck (don't guess), then typically: add caching, fix N+1 queries, add DB indexes, use async/background jobs for slow parts, paginate large responses.

**Microservices**
- Splitting an app into small, independent services (each with its own responsibility and often its own database), communicating over APIs — improves scalability but adds complexity (networking, data consistency).

**Load Balancing & Deployment Strategies**
- Load balancer distributes incoming traffic across multiple servers so no single server is overwhelmed.
- Deployment strategies: rolling deployment (gradually replace old instances), blue-green (switch traffic between two full environments), canary (release to a small % of users first).

**Dev vs Prod Environment Differences**
- Dev: debug mode on, verbose errors, local/test data, relaxed security.
- Prod: debug off, real user data, stricter security, performance-optimized, monitoring/logging enabled.

**Clustering**
- Grouping multiple servers/instances together so they act as one system for scalability and failover (if one node fails, others keep serving).

**Debugging Cross-Browser/Cross-Device Issues**
- Check browser console for errors first, verify CSS/JS feature support per browser (caniuse.com), test responsive breakpoints, check for vendor-prefix or polyfill needs.

**Handling DB/API Bottlenecks**
- Identify with profiling/monitoring, then: add caching, add indexes, optimize queries, scale horizontally (more servers) or vertically (bigger server), use a queue for heavy tasks.

**JIT vs AOT (General)**
- JIT (Just-In-Time): compiles code to machine code while the program runs — faster startup optimization over time.
- AOT (Ahead-Of-Time): compiles fully before running — faster startup, but less runtime optimization.

**Drawing an ERD (Entity-Relationship Diagram)**
- A diagram showing entities (tables), their attributes, and how they relate (one-to-one, one-to-many, many-to-many) — used to design a database schema before building queries/relations on top of it.

**React vs Angular — When to Use Each**
- React: when you want flexibility and a lighter setup, picking your own tools, good for small-to-medium apps or teams that want more control.
- Angular: when you want a full, opinionated framework with everything built in (routing, forms, DI), good for large enterprise apps needing consistency.

**Node vs Laravel — When to Use Each**
- Node: JavaScript end-to-end, great for real-time apps, high concurrency, I/O-heavy workloads, and teams already using JS on the frontend.
- Laravel (PHP): strong built-in tooling (ORM, auth, templating) out of the box, often faster to build traditional CRUD/admin-style apps.

**SQL vs NoSQL — When to Use Each (and Which Is Faster)**
- SQL: relationships matter, data structure is stable, need strong consistency (e.g. financial systems).
- NoSQL: schema changes often, need to scale horizontally, data is naturally document/key-value shaped (e.g. logging, catalogs).
- Neither is universally "faster" — SQL is typically faster for complex relational queries with proper indexing; NoSQL is typically faster for simple, high-volume read/write on flexible data.

---

## 19. DevOps, Docker & Testing

**Can I Write SQL Statements Inside MongoDB?**
- No — MongoDB uses its own query language (based on JSON-like documents and operators like `$match`, `$set`), not SQL syntax. You'd use MongoDB's own methods instead.

**If the Server Is Not Running, What Should You Do?**
- Check the log files first to see the actual error/reason before restarting or making changes blindly.

**chown vs chmod**
- `chown`: changes who **owns** a file (user/group).
- `chmod`: changes what **permissions** (read/write/execute) a file has.

**What Does 405 Mean After chmod?**
- 405 = "Method Not Allowed" — usually means the server doesn't have the right execute/access permissions on the requested resource after the permission change, so it rejects that HTTP method.

**3-Tier Architecture**
- Splits an app into three layers: Presentation (UI), Application/Logic (business logic/API), and Data (database) — each can be scaled/maintained independently.

**Thin Client vs Thick Client**
- Thin client: does minimal processing, relies heavily on the server (e.g. a browser-based app).
- Thick client: handles more logic/processing locally on the user's machine, needs less from the server.

**What is Docker / Dockerfile**
- Docker packages an app and everything it needs (dependencies, runtime, config) into a portable "container" that runs the same way anywhere.
- A Dockerfile is a script of instructions (base image, copy files, install dependencies, set the start command) used to build that container image.

**CMD vs ENTRYPOINT (Docker)**
- `CMD`: provides default arguments/command for the container, easily overridden when running the container.
- `ENTRYPOINT`: sets the main command that always runs — harder to override, often combined with `CMD` for default arguments.

**Testing & Unit Testing Tools**
- Unit tests check small pieces of code (functions/components) in isolation. Common tools: Jest, Mocha/Chai (JS/Node), Pytest/unittest (Python), JUnit (Java).

---

## 20. AI/ML Basics (bonus, if asked)

**RAG (Retrieval-Augmented Generation)**
- Combines a search step (retrieve relevant documents) with an LLM generation step, so the model answers using real, up-to-date data instead of relying only on what it memorized.

**Vector DB**
- A database optimized for storing and searching "embeddings" (numeric representations of meaning) — used to find semantically similar content, core to RAG systems.

**Open-Source Models (STT/TTS/Image Classification)**
- STT (Speech-to-Text): e.g. Whisper. TTS (Text-to-Speech): e.g. Coqui TTS. Image classification: e.g. ResNet, YOLO for detection.

**Prompt Engineering**
- Crafting inputs (instructions/examples) to get better, more reliable outputs from an LLM without changing the model itself.

---

## 21. CS Theory

**Big O vs Theta (Θ) vs Big Omega (Ω)**
- Big O: worst-case (upper bound) — most commonly used.
- Big Omega: best-case (lower bound).
- Theta: tight bound — when best and worst case grow the same way, describes the average/exact growth rate.

---

## 22. Behavioral

**"Introduce yourself, a project you worked on, and a problem you faced"**
- Keep it short and structured: who you are + your background → one relevant project (what it does, your role, tech used) → one real problem you hit and how you solved it (shows problem-solving, not just what you built).

**Explaining Where You Put Business Logic (Backend vs Frontend)**
- Be ready to justify why a piece of logic (e.g. a view/validation) lives in the backend (Django) vs the frontend (Angular/React) — generally: business rules, data validation, and security-sensitive logic belong in the backend; UI state and presentation logic belong in the frontend.

**Explaining Atomicity in a Feature (e.g. Payments)**
- Be ready to describe how you ensured a multi-step operation (like a payment) either fully completes or fully rolls back — usually via database transactions wrapping all the steps.

**Handling a Conflict with a Colleague**
- Focus on communication: describe listening to their view, explaining your reasoning with data/examples, and reaching a compromise or escalating calmly and professionally if needed.

**Wanting to Implement an Idea the Lead Isn't Convinced About**
- Show you can advocate for an idea with evidence/reasoning, but also respect the final decision and commit to it once made ("disagree and commit").

**Being Asked to Work During Vacation**
- Show flexibility balanced with boundaries — willing to help for something urgent/important, while being honest about availability and not treating it as the norm.

**Practical Interview Tip: Demo Credentials**
- Keep a ready login (username/password) for each of your past projects so you can log in and demo them live during interviews without fumbling to remember credentials.

---

## 23. Coding Problems Mentioned (practice separately on LeetCode)

- Best Time to Buy and Sell Stock
- Max Altitude / running sum type problem
- Two Sum
- Reverse a String
- Convert an Array to an Object
- Remove Duplicates from an Array
- Count the Occurrence of Each Character in a String
- Get All Unique Elements of an Array (approach only)
- Find Prime Numbers from a Large Array with a Custom Selection Pattern (e.g. first batch of 10, then a scaled-up batch from what remains)

*(These need hands-on practice, not a written answer — good candidates for your brute-force → optimal LeetCode workflow.)*
