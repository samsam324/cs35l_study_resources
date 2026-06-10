# CS35L Study Guide: JavaScript & Node.js

This guide is a dense, complete reference for the JavaScript / Node.js portion of CS 35L Lecture 5 ("Client Server & Node.js") together with the full Node.js reference page (`node.md`). The pure networking content (TCP/IP layers, HTTP protocol details, P2P, status codes, IP addresses, UDP/TCP handshake, etc.) is covered in a separate guide.

---

## 1. JavaScript: A Familiar Hybrid Language

If Python and C++ had a child that was raised on the internet, it would be JavaScript. It powers most of the interactive web you use daily, runs on servers via Node.js (used at companies such as **LinkedIn, PayPal, Uber, and NASA**), and ships in cross-platform desktop apps like **VS Code and Discord** (via the **Electron** framework, which embeds Node.js).

- **From C++, JS inherits its syntax**: curly braces `{}`, semicolons `;`, `if/else` statements, `for` and `while` loops, and `switch` statements.
- **From Python, JS inherits its dynamic nature**: dynamically typed. You don't declare whether a variable is an `int` or a `string`. You don't manage memory with `malloc` or `new/delete`; there are no explicit pointers, and a garbage collector handles memory for you.
- Modern engines like **V8** don't simply interpret JavaScript — they execute bytecode through a fast interpreter (**Ignition**) and **Just-In-Time-compile** hot code paths to native machine code via **TurboFan/Maglev**.

### JavaScript (lecture characterization)

- **Interpreted**, **dynamically typed** programming language (similar to Python).
- Can be hooked into HTML.

```js
// A function that takes two arguments
function add(num1, num2) {
    let sum = num1 + num2;
    return sum; // Returns the result
}

let result = add(5, 7); // result is now 12
console.log(result); // Output: 12
```

### Variable Declaration

Instead of C++'s `int x = 5;` or Python's `x = 5`, modern JavaScript uses `let` and `const`:

```js
let count = 0;       // A variable that can be reassigned
const name = "UCLA"; // A constant that cannot be reassigned
```

**Never use `var`** — it has function-scoped hoisting rules that violate the block-scope behavior you learned in C++ and Python. Always prefer `let` or `const`.

---

## 2. What is Node.js?

Historically, JavaScript was trapped inside the web browser. It was strictly a front-end language used to make websites interactive.

**Node.js** is a **runtime environment** that takes JavaScript out of the browser and lets it run directly on your computer's operating system. It embeds Google's **V8 engine** to execute code, but also includes a powerful C library called **libuv** to handle the asynchronous event loop and system-level tasks like file I/O and networking. This means you can use JavaScript to write backend servers just like you would with Python or C++.

### Node.js (lecture summary)

- **JavaScript runtime** for asynchronous events.
- Allows you to run JavaScript outside of the browser.
- Is **not** a programming language (you are writing code in JavaScript to be run within the Node.js environment).
- Fundamentally **single-threaded** for its main execution of JavaScript code.
- Works via **callbacks** and **event handlers**.

### Mental Model Comparison (C++ vs Python vs Node.js)

| Aspect    | C++                  | Python                                | JavaScript (Node.js)              |
|-----------|----------------------|---------------------------------------|------------------------------------|
| Typing    | Static               | Dynamic                               | Dynamic                            |
| Memory    | Manual (`new/delete`)| GC (reference counting + cycle collector) | GC (V8: generational, tracing) |
| Run with  | Compile → `./app`    | `python script.py`                    | `node script.js`                   |
| I/O model | Synchronous (blocks) | Synchronous (blocks)                  | Asynchronous (non-blocking)        |

### Running a script

Like Python, there is no compilation step. You run a JavaScript file directly:

```
node script.js
```

And like Python, there is no required `main()` function — Node.js executes scripts top-to-bottom. V8 JIT-compiles the code at runtime.

### Printing output

JavaScript's equivalent of Python's `print()` and C++'s `printf()` is `console.log()`. It writes to stdout with a trailing newline:

```js
// Python equivalent: print("Hello from Node.js!")
// C++ equivalent:    printf("Hello from Node.js!\n");
console.log("Hello from Node.js!");
```

---

## 3. The Paradigm Shift: Asynchronous Programming

Here is the largest "threshold concept" you must cross: **JavaScript is fundamentally asynchronous and single-threaded.**

In C++ or Python, if you make a network request or read a file, your code typically stops and waits (blocks) until that task finishes. In Node.js, blocking the main thread is a cardinal sin. Instead, Node.js uses an **Event Loop**. When you ask Node.js to read a file, it delegates that task to the operating system and immediately moves on to execute the next line of code. When the file is ready, a "callback" function is placed in a queue to be executed.

**Mental Model Adjustment**: You must stop thinking of your code as executing strictly top-to-bottom. You are now setting up "listeners" and "callbacks" that react to events as they finish.

---

## 4. The Event Loop in Detail

The Event Loop is best understood with the **Restaurant Metaphor**:

| Kitchen Role                 | Node.js Equivalent | What It Does                                                                                  |
|------------------------------|--------------------|-----------------------------------------------------------------------------------------------|
| The Chef                     | Call Stack         | Executes one task at a time. If busy, everything else waits.                                  |
| The Appliances (oven, fryer) | libuv / OS         | Handle slow work (file reads, network) in the background.                                     |
| The Waiter                   | Task Queue         | When an appliance finishes, the callback is queued.                                           |
| The Kitchen Manager          | Event Loop         | Only when the Chef's hands are completely empty does the Manager hand over the next callback. |

**The critical insight**: `setTimeout(fn, 0)` does NOT mean "run immediately". It means "run when the call stack is empty". Synchronous code always runs to completion before any callback fires:

```js
setTimeout(() => console.log("B"), 0);   // queued in Task Queue
console.log("A");                        // runs immediately
console.log("C");                        // runs immediately
// Output: A, C, B  (NOT A, B, C!)
```

This is why blocking the main thread with a long synchronous operation is catastrophic in Node.js — it prevents **ALL** other requests, timers, and I/O callbacks from being processed.

---

## 5. Blocking vs. Non-Blocking I/O (Lecture)

`require` imports the `fs` (file system) module from node standard library (similar to `import fs as fs`).

`const` declares a constant (cannot be changed after initialization). Use constants to prevent bugs in your code!

### Blocking Code

```js
const fs = require('node:fs');
const data = fs.readFileSync('/file.md');
// blocks here until file is read
```

### Non-Blocking Code

```js
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
    if (err) {
        // …
    }
});
// following code is executed instantly
```

The function `(err, data) => { ... }` defines an **asynchronous callback** that executes after the data has been completely stored in the `data` variable.

**Node.js philosophy**: use non-blocking code for all non-instant tasks to maintain **high performance** and **scalability**.

### Defining Callback Functions for Long Non-Blocking Code

```js
function handleMarkdownRead(err, data) {
    if (err) {
        // …
    }
}

fs.readFile('/file.md', handleMarkdownRead);
// following code is executed instantly
```

**Callback Functions** allow you to **reuse** non-blocking code, give it a **name**, and make your code **easier to read**.

### Demo (lecture interactive)

```js
const fs = require('fs');

// --- Blocking ---
console.log("=== Blocking readFileSync ===");
const dataSync = fs.readFileSync('message.txt', 'utf8');
console.log(dataSync);
console.log("(This line runs AFTER the file is read)\n");
```

---

## 6. Modern Asynchrony: Promises and async/await

In the earlier example, we mentioned that Node.js uses "callbacks" to handle events. However, nesting multiple callbacks inside one another leads to a notoriously difficult-to-read structure known as **"Callback Hell"**.

To manage cognitive load and make asynchronous code easier to reason about, modern JavaScript introduced **Promises** (conceptually similar to `std::future` in C++) and the **async/await** syntax.

A **Promise** is exactly what it sounds like: an object representing the eventual completion (or failure) of an asynchronous operation. Using `async/await` allows you to write asynchronous code that looks and reads like traditional, synchronous C++ or Python code.

### Creating a Promise

The `new Promise(...)` constructor takes a single function (called the **executor**) that receives two arguments — `resolve` (call when the work succeeds) and `reject` (call when it fails):

```js
// Under the hood, this is how async operations are built:
const promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("data ready!"), 100);
});

// Consuming it with .then():
promise.then(data => console.log(data));   // "data ready!" after 100ms
```

In practice you rarely create Promises from scratch — you mostly consume them using `await` or `.then()`. Libraries like `fs.promises` and `fetch` return Promises for you.

### Three Generations of Async

Node.js async syntax evolved through three generations. You need to recognize all three — and write the third:

**Generation 1: Callbacks** — each async operation nests inside the previous one ("Callback Hell"):

```js
fetchData('a', (err, dataA) => {
    if (err) throw err;
    fetchData('b', (err2, dataB) => {  // "Pyramid of Doom"
        if (err2) throw err2;
    });
});
```

**Generation 2: Promises** — flatten the nesting with `.then()` chains:

```js
fetchData('a')
    .then(dataA => fetchData('b'))
    .then(dataB => console.log(dataB))
    .catch(err  => console.error(err));
```

**Generation 3: async/await** — looks like synchronous code but doesn't block:

```js
async function fetchUserData(userId) {
    try {
        // 'await' suspends THIS function (non-blocking!) and lets other work proceed
        const response = await database.getUser(userId);
        console.log(`User found: ${response.name}`);
    } catch (error) {
        // Error handling looks exactly like C++ or Python
        console.error(`Error fetching user: ${error.message}`);
    }
}
```

When JavaScript hits `await`, it suspends the async function, frees the call stack, and lets the Event Loop process other work. When the Promise resolves, execution resumes. This looks like synchronous C++/Python code — but it does **NOT** block the event loop.

### Sequential vs Parallel

If two operations are independent, use `Promise.all()` for better performance:

```js
// SLOWER: sequential — total time = time(A) + time(B)
const a = await fetchA();
const b = await fetchB();

// FASTER: parallel — total time = max(time(A), time(B))
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

### The `.forEach()` Trap

`.forEach()` does **NOT** await async callbacks — it fires them all and returns immediately:

```js
// BUG: "All done!" prints BEFORE items are processed
items.forEach(async (item) => {
    await processItem(item);
});
console.log("All done!");  // runs immediately!

// FIX (sequential): use for...of
for (const item of items) {
    await processItem(item);
}
console.log("All done!");  // runs after all items

// FIX (parallel): use Promise.all + .map()
await Promise.all(items.map(item => processItem(item)));
console.log("All done!");
```

`.forEach()` ignores the Promises returned by its async callbacks — it has no mechanism to wait for them. This is one of the most common async bugs in JavaScript.

---

## 7. The `===` Trap: Type Coercion

JavaScript has TWO equality operators. **Only ever use `===`**:

```js
// WRONG: == triggers implicit type coercion — a JS-specific danger
console.log(1 == "1");    // true  ← DANGEROUS SURPRISE
console.log(0 == false);  // true  ← DANGEROUS SURPRISE

// RIGHT: === checks value AND type (behaves like == in Python and C++)
console.log(1 === "1");   // false ← correct
console.log(0 === false); // false ← correct
```

This is **negative transfer**: your `==` intuition from C++ and Python is correct — but JavaScript's `==` does something different. Use `===` and it matches your expectation.

Strict equality table:

```js
// ✓ Strict equality — no surprises
1 === "1"     // false
0 === false   // false
"" === false  // false

// ✗ Loose equality — implicit coercion traps
1 == "1"      // true  ← DANGER
0 == false    // true  ← DANGER
"" == false   // true  ← DANGER
```

The same applies to `!==` (use it) vs `!=` (avoid it).

---

## 8. JavaScript's Two "Nothings": `null` vs `undefined`

C++ has `nullptr`. Python has `None`. JavaScript has **two distinct values** meaning "nothing":

```js
let score;                // declared but no value assigned → undefined
console.log(score);       // undefined
console.log(typeof score); // "undefined"

let student = null;       // explicitly set to "no value"
console.log(student);     // null
console.log(typeof student); // "object" (a famous JS bug that can never be fixed)
```

| Concept           | `undefined`                                                      | `null`                          |
|-------------------|------------------------------------------------------------------|---------------------------------|
| Meaning           | "no value was assigned yet"                                      | "intentionally empty"           |
| When you see it   | Uninitialized variables, missing function args, `req.query.missing` | You (or an API) explicitly set it |
| `typeof`          | `"undefined"`                                                    | `"object"` (a historical JS bug) |
| Python equivalent | No direct equivalent (`NameError`)                               | `None`                          |

**Watch out**: `null == undefined` is `true` (coercion!), but `null === undefined` is `false`. One more reason to always use `===`.

---

## 9. Control Flow Syntax

JavaScript's control flow looks like C++ (braces required), not Python (no colons/indentation):

```js
// if/else — braces required (no colons like Python, no elif — use else if)
if (score >= 90) {
    console.log("A");
} else if (score >= 60) {
    console.log("Pass");
} else {
    console.log("Fail");
}

// for loop — same structure as C++
for (let i = 0; i < 5; i++) {
    console.log(i);
}

// for...of — like Python's "for x in list"
const names = ["Alice", "Bob", "Carol"];
for (const name of names) {
    console.log(name);
}
```

---

## 10. Functions as First-Class Values

In C++ you've encountered function pointers. In Python, you've passed functions to `sorted(key=...)`. JavaScript takes this further: **functions are just values, exactly like numbers or strings**.

Arrow functions are the modern preferred syntax:

```js
// C++ equivalent: int add(int a, int b) { return a + b; }
// Python equivalent: lambda a, b: a + b

const add    = (a, b) => a + b;
const greet  = (name) => `Hello, ${name}!`;
const double = n => n * 2;           // Parens optional for single param
```

### `.map()`, `.filter()`, `.reduce()`

These array methods take callback functions — the same "functions as values" concept. They are the JavaScript equivalents of Python's `map()`, `filter()`, and `functools.reduce()`:

```js
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(n => n * 2);              // [2, 4, 6, 8, 10]
const evens   = numbers.filter(n => n % 2 === 0);     // [2, 4]
const sum     = numbers.reduce((acc, n) => acc + n, 0); // 15
```

### `.find()`

`.find()` returns the **first matching element** (or `undefined` if none match) — use it when you need one specific item:

```js
const students = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];
const alice = students.find(s => s.id === 1);   // { id: 1, name: "Alice" }
const missing = students.find(s => s.id === 99); // undefined
```

Understanding callbacks is essential — all of Node.js's async operations notify you they are finished by calling a function you provided.

---

## 11. Destructuring: Unpacking Values

JavaScript has compact syntax for extracting values from arrays and objects:

```js
// Array destructuring (like Python's tuple unpacking: r, g, b = color)
const [red, green, blue] = [255, 128, 0];

// Object destructuring (extract properties by name)
const config = { host: "localhost", port: 3000, debug: true };
const { host, port } = config;   // host = "localhost", port = 3000

// Works in function parameters — you will see this in every Express route and React component:
function startServer({ host, port }) {
    console.log(`Listening on ${host}:${port}`);
}
```

---

## 12. Formatting Output: `.toFixed()` and `.padEnd()`

Two utilities you will use when formatting output:

```js
// .toFixed(n) — format a number to exactly n decimal places (returns a string)
const avg = 87.666;
console.log(avg.toFixed(1));   // "87.7"
console.log(avg.toFixed(2));   // "87.67"

// .padEnd(n) — pad a string with spaces to reach length n (left-aligns text in columns)
console.log("Alice".padEnd(7) + "| 95");   // "Alice  | 95"
console.log("Bob".padEnd(7) + "| 42");     // "Bob    | 42"

// .padStart(n) — pad from the left (right-aligns text)
console.log("42".padStart(5));   // "   42"
```

---

## 13. Data Representation: JavaScript Objects and JSON

If you understand Python dictionaries, you already understand the general structure of JavaScript Objects. Unlike C++, where you must define a `struct` or `class` before instantiating an object, JavaScript allows you to **create objects on the fly using key-value pairs**.

**Wait, what about JSON?** While they look similar, **JSON (JavaScript Object Notation)** is a strict data-interchange format. Unlike JS objects, JSON requires double quotes for all keys and string values, and it cannot store functions or special values like `undefined`. JSON is simply this structure serialized into a string format so it can be sent over a network.

```js
// This is a JavaScript Object (similar to a Python dictionary, but keys are coerced to
// strings/Symbols and objects also have a prototype chain)
const student = {
    name: "Joe Bruin",
    uid: 123456789,
    courses: ["CS31", "CS32", "CS35L"],
    isGraduating: false
};

// Accessing properties is done via dot notation (like C++ objects)
console.log(student.courses[2]); // Outputs: CS35L
```

JSON is simply this exact object structure serialized into a string format so it can be sent over an HTTP network request.

### JSON (lecture characterization)

JSON is a **human-readable, lightweight data-interchange format** used to transfer data between a server and a web page, or between applications. It is the standard format for REST API responses. In JavaScript, objects look almost identical to JSON.

Example JSON document (from lecture):

```json
{
    "address-dict": {
        "city": "New York",
        "residential": true,
        "postal_code": 10023
    },
    "phone_numbers": [
        {
            "type": "home",
            "number": "212 555-1234"
        },
        {
            "type": "office",
            "number": "646 555-4567"
        }
    ],
    "children": [
        "Catherine",
        "Thomas",
        "Trevor"
    ]
}
```

---

## 14. HTML (Hyper Text Markup Language)

HTML is the most common **markup language** used to create web pages. It supports structured content. HTML is **not** a programming language.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Page Title</title>
</head>
<body>
    <h1>This is a Heading</h1>
    <p>This is a paragraph.</p>
</body>
</html>
```

- **Head** contains metadata read by the browser.
- **Body** contains the content to be rendered.
- **h1** indicates the top heading (others: h2 – h6).

---

## 15. HTTP Server from Scratch with Node's `http` Module

Node.js ships with a built-in `http` module. You can create a server without any third-party packages:

```js
const http = require('http');
const PORT = 3000;

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World!\n');
});

server.listen(PORT, 'localhost', () => {
    console.log(`Server running at http://localhost:${PORT}/`);
});
```

Line-by-line explanation (from lecture annotations):

- `const http = require('http');` — Imports http module.
- `const server = http.createServer((req, res) => { ... });` — Creates an HTTP server object by defining a callback to be called whenever the server receives an HTTP request (`req`).
- `res.writeHead(200, { 'Content-Type': 'text/plain' });` — Sets HTTP response header to status code 200 (success) and plain text content.
- `res.end('Hello, World!\n');` — Writes `'Hello, World!'` into the HTTP response body and sends the message.
- `server.listen(PORT, 'localhost', () => { ... });` — Starts the server on localhost to listen on port 3000.
- The backtick template literal `` `Server running at http://localhost:${PORT}/` `` is like f-strings in Python.

---

## 16. Express.js — A Routing Framework

Express is a popular framework (install via `npm install express`) that simplifies routing URLs to handlers.

### Worked Example: Simple Client-Server Setup (from node.md)

```js
// 'require' is JS's version of Python's 'import' or C++'s '#include'
const express = require('express');
const app = express();
const port = 8080;

// Route for a GET request to localhost:8080/users/123
app.get('/users/:userId', (req, res) => {
    // Notice the backticks (`). This allows string interpolation.
    // It is exactly like f-strings in Python: f"GET request to user {userId}"
    res.send(`GET request to user ${req.params.userId}`);
});

// Route for all POST requests to localhost:8080/
app.post('/', (req, res) => {
    res.send('POST request to the homepage');
});

// Start the server
app.listen(port, () => {
    console.log(`Server listening on port ${port}`);
});
```

**Breakdown:**

- **Arrow Functions `(req, res) => { ... }`**: This is a concise way to write an anonymous function. You are passing a function as an argument to `app.get()`. This is how JS handles asynchronous events: "When someone makes a GET request to this URL, run this block of code."
- **`req` and `res`**: These represent the HTTP Request and HTTP Response objects, abstracting away the raw network sockets you would have to manage manually in lower-level C++.

### Larger Express Example (Lecture Slide)

```js
const express = require('express');
const app = express();
const port = 8080;

app.get('/users/:userId', (req, res) => {
    res.send(`GET request to user ${req.params.userId}`);
});

app.post('/', (req, res) => {
    res.send('POST request to the homepage');
});

app.get('/about', (req, res) => {
    res.send('About page');
});

app.all('*', (req, res) => {
    res.status(404).send('404 - Page not found');
});

// app.listen() was missing from the lecture slide — added here!
app.listen(port, () => {
    console.log(`Express server listening on port ${port}`);
});
```

Lecture annotations:

- `const express = require('express');` — Imports the super helpful `express` module that helps you with routing URLs.
- `app.get('/users/:userId', ...)` — Route for all GET requests to `localhost:8080/users/.*`. Retrieves the user id from the GET request URL via `req.params.userId`.
- `app.post('/', ...)` — Route for all POST requests to `localhost:8080/`.
- `app.get('/about', ...)` — Route for all GET requests to `localhost:8080/about`.
- `app.all('*', ...)` — Route for all other requests (catch-all 404 handler — must be last).

Example output when running:

```
> app.js
Express server listening on port 8080
Express server listening on port 8080
```

Hitting `GET http://localhost:8080/users/213` returns `GET request to user 213` (200 OK, 23 bytes).

---

## 17. NPM — The Node Package Manager

If you remember using `#include <vector>` in C++ or `import requests` (via pip) in Python, Node.js has **NPM**. NPM is a massive ecosystem of open-source packages. Whenever you start a new Node.js project, you will run:

- `npm init` (creates a `package.json` file to track your dependencies)
- `npm install <package_name>` (downloads code into a `node_modules` folder)

### From the Lecture

NPM allows you to install node packages and keep the version listed in your `package.json`. Node has a large ecosystem of useful packages.

**Common npm commands:**

```
$ npm init                       (creates a new NPM package from your code)
$ npm install express            (installs the express package)
$ npm install express@4          (installs version 4 of express)
$ npm install --save             (creates a package.json)
$ npm install                    (installs all packages listed in the package.json)
```

---

## 18. Tips for Mastering JS / Node.js

Here is how you should approach mastering this new ecosystem:

- **Utilize Pair Programming**: Don't learn Node.js in isolation. Sit at a single screen with a peer (one "Driver" typing, one "Navigator" reviewing and strategizing). Research shows pair programming significantly increases confidence and code quality while reducing frustration for novices transitioning to a new language paradigm (McDowell et al. 2006; Cockburn and Williams 2000; Williams and Kessler 2000).
- **Embrace Test-Driven Development (TDD)**: In Python, you might have used `pytest`; in C++, `gtest`. In JavaScript, frameworks like **Jest** are the standard. Before you write a complex API endpoint in Express, write a test for what it should do. This acts as a formative assessment, giving you immediate, automated feedback on whether your mental model of the code aligns with reality.
- **Avoid "Vibe Coding" with AI**: While Large Language Models (LLMs) can generate Node.js boilerplate instantly, relying on them before you understand the asynchronous Event Loop will lead to "unsound abstractions". Use AI to explain confusing syntax or error messages, but do not let it rob you of the cognitive struggle required to build your own notional machine of how JavaScript executes.

---

## 19. Top 10 JavaScript & Node.js Best Practices

These are the most important conventions and idioms that experienced JavaScript developers follow. Internalizing them will make your code more predictable, less error-prone, and immediately recognizable as modern JavaScript.

### 1. Default to `const`, Use `let` Only When Reassigning, Never Use `var`

`const` prevents accidental reassignment and signals intent. `let` is for values that genuinely change. `var` has broken scoping rules — never use it.

```js
// ✓ const — value never changes
const MAX_RETRIES = 3;
const students = ["Alice", "Bob"];  // The array can be mutated, but the binding cannot

// ✓ let — value changes
let count = 0;
for (let i = 0; i < 5; i++) {
    count += i;
}

// ✗ Never use var — it leaks out of blocks and hoists unexpectedly
var x = 10;
if (true) { var x = 20; }
console.log(x);  // 20 — surprised?
```

**Note**: `const` prevents reassignment, **not mutation**. A `const` array can still be `.push()`-ed to. To prevent mutation, use `Object.freeze()`.

### 2. Always Use `===` (Strict Equality), Never `==`

JavaScript's `==` performs implicit type coercion, producing dangerous surprises. `===` checks both value AND type — matching the behavior you expect from C++ and Python.

```js
// ✓ Strict equality — no surprises
1 === "1"     // false
0 === false   // false
"" === false  // false

// ✗ Loose equality — implicit coercion traps
1 == "1"      // true  ← DANGER
0 == false    // true  ← DANGER
"" == false   // true  ← DANGER
```

The same applies to `!==` (use it) vs `!=` (avoid it).

### 3. Use `async/await` for Asynchronous Code

Modern JavaScript uses `async/await` for asynchronous operations. It reads like synchronous code while remaining non-blocking. Always wrap `await` in `try/catch`.

```js
// ✓ Modern: async/await with error handling
async function loadData() {
    try {
        const data = await fetchFromAPI();
        return process(data);
    } catch (err) {
        console.error("Failed to load:", err.message);
    }
}

// ✗ Avoid: deeply nested callbacks ("Callback Hell")
fetchA((err, a) => {
    fetchB((err, b) => {
        fetchC((err, c) => { /* pyramid of doom */ });
    });
});
```

### 4. Use `Promise.all()` for Independent Async Operations

When two operations do not depend on each other, run them concurrently. Sequential `await` wastes time.

```js
// ✓ Concurrent — total time = max(time(A), time(B))
const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
]);

// ✗ Sequential — total time = time(A) + time(B)
const users = await fetchUsers();   // waits...
const posts = await fetchPosts();   // then waits again
```

### 5. Use Template Literals for String Formatting

Backtick strings with `${expression}` are JavaScript's equivalent of Python's f-strings. They are more readable and less error-prone than `+` concatenation.

```js
const name = "Alice";
const score = 95;

// ✓ Template literal — clear and concise
const msg = `${name} scored ${score} points`;

// ✗ Concatenation — verbose and easy to break
const msg = name + " scored " + score + " points";
```

Template literals also support multi-line strings and arbitrary expressions inside `${}`.

### 6. Use Arrow Functions for Callbacks

Arrow functions are concise and lexically bind `this` (they inherit `this` from the enclosing scope, avoiding a common class of bugs).

```js
const numbers = [1, 2, 3, 4, 5];

// ✓ Arrow functions — concise
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// ✗ Verbose equivalent
const doubled = numbers.map(function(n) { return n * 2; });
```

**When NOT to use arrow functions**: Object methods that need their own `this`, and constructor functions.

### 7. Use Destructuring to Extract Values

Destructuring makes code more concise and self-documenting by extracting values from objects and arrays in one step.

```js
// ✓ Object destructuring
const { name, grade } = student;

// ✓ In function parameters (common in React)
function printStudent({ name, grade }) {
    console.log(`${name}: ${grade}`);
}

// ✓ Array destructuring with Promise.all
const [roster, grades] = await Promise.all([fetchRoster(), fetchGrades()]);

// ✗ Verbose alternative
const name = student.name;
const grade = student.grade;
```

### 8. Never Block the Event Loop

Node.js is single-threaded. Blocking the main thread prevents **ALL** other requests, timers, and callbacks from executing. Always use asynchronous I/O.

```js
// ✓ Non-blocking — other requests can proceed
const data = await fs.promises.readFile("data.json", "utf8");

// ✗ Blocking — entire server freezes until file is read
const data = fs.readFileSync("data.json", "utf8");
```

For CPU-intensive work, offload to **Worker Threads** instead of running it on the main thread.

### 9. Use Optional Chaining (`?.`) and Nullish Coalescing (`??`)

These modern operators replace verbose null-checking patterns and make code more robust.

```js
// ✓ Optional chaining — safe deep access
const city = user?.address?.city;           // undefined if any link is null
const first = results?.[0];                 // safe array access

// ✓ Nullish coalescing — default only for null/undefined
const port = config.port ?? 3000;           // 0 is preserved as valid
const name = user.name ?? "Anonymous";      // "" is preserved as valid

// ✗ Verbose null checking
const city = user && user.address && user.address.city;

// ✗ || treats 0, "", and false as "missing"
const port = config.port || 3000;           // if port is 0, uses 3000!
```

### 10. Use `.map()`, `.filter()`, `.reduce()` Instead of Manual Loops

These array methods are more declarative, less error-prone, and do not mutate the original array. They are the JavaScript equivalents of Python's `map()`, `filter()`, and `functools.reduce()`.

```js
const students = [
    { name: "Alice", grade: 95 },
    { name: "Bob",   grade: 42 },
    { name: "Carol", grade: 78 },
];

// ✓ Declarative — chain operations fluently
const honors = students
    .filter(s => s.grade >= 90)
    .map(s => s.name);
// ["Alice"]

// ✗ Imperative — more code, mutation, more room for bugs
const honors = [];
for (let i = 0; i < students.length; i++) {
    if (students[i].grade >= 90) {
        honors.push(students[i].name);
    }
}
```

Use regular `for` loops when you need early termination (`break`), when performance on very large arrays matters, or when the logic is too complex for a single chain.

---

## 20. Practice Problems

### Node.js/JavaScript Syntax — What Does This Code Do? (Basic)

You are shown JavaScript/Node.js code. Explain what it does and what it outputs.

```js
let count = 0;
const MAX = 200;
```

### Node.js/JavaScript Syntax — Write the Code (Intermediate)

Check if a variable `userInput` (which might be a string) equals the number `42`, without being tricked by type coercion.

### Node.js Concepts Quiz (Intermediate)

You are building a TikTok-style feed. Match each task to the best array method:

- **Task A**: Remove videos the user has already seen
- **Task B**: Convert each video object into a `<VideoCard>` component
- **Task C**: Calculate the total watch time across all videos

Choices:

- **A**: A: `.filter()`, B: `.map()`, C: `.reduce()`
- **B**: A: `.filter()`, B: `.reduce()`, C: `.map()`
- **C**: A: `.reduce()`, B: `.map()`, C: `.filter()`
- **D**: A: `.map()`, B: `.filter()`, C: `.reduce()`

(Correct answer: **A** — filter to remove seen videos, map to transform each video into a component, reduce to aggregate total watch time.)

---

## 21. NodeJS Tutorials (Lecture References)

- **Course tutorial**: not a fresh start, builds on what you know from Python & C++. Interactive tutorial right in the browser on smaller individual tasks. Also teaches some basics of express, communication via HTTP, and JSON. URL: https://tobiasduerschmid.github.io/SEBook/tools/nodejs-tutorial
- **Video Tutorial**: https://www.youtube.com/watch?v=32M1al-Y6Ag — Practically-minded "bootcamp" on how to build a complete server backend in NodeJS.

---

## 22. References (from `node.md`)

- **Cockburn and Williams 2000**: Alistair Cockburn and Laurie Williams (2000) "The costs and benefits of pair programming," International Conference on Extreme Programming and Flexible Processes in Software Engineering (XP), pp. 223–243.
- **McDowell et al. 2006**: Charlie McDowell, Linda Werner, Heather E. Bullock, and Julian Fernald (2006) "Pair programming improves student retention, confidence, and program quality," Communications of the ACM, 49(8), pp. 90–95.
- **Williams and Kessler 2000**: Laurie A. Williams and Robert R. Kessler (2000) "All I really need to know about pair programming I learned in kindergarten," Communications of the ACM, 43(5), pp. 108–114.


---

# Appendix: Lecture 5 Slides (raw extracted text)

The following is the full extracted text of Lecture 5. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 5 � Client
Server & Node.js
Assistant Teaching Professor
Computer Science Department
ACM ICPC proudly presents:
 CodeSprint LA 2026!
Embark on a quest through the Kingdom of Hyrule! Our Legend of Zelda themed
competitive programming contest features $2,100+ in total prizes, including $750
reserved exclusively for UCLA Beginners.  Team up to solve challenging, out-
of-the-box programming problems and win BIG! The event runs from 9:00 AM to
5:00 PM and includes:
� Morning: Beginner Workshop, Opening Ceremony, and Lunch .
� The Contest: 11:30 AM � 4:30 PM
� Evening: Closing Ceremony and Awards .
When: Saturday, May 9th, 2026 Where: De Neve Plaza Room
Format: Teams of up to 3
Register by May 3rd: https://codesprintla.uclaacm.com/register
For the full schedule and more details, visit https://codesprintla.uclaacm.com/
Now this Course is Starting to Get Hard...
This lecture:
� Will give a very short intro to JavaScript & Node.js
� Will teach you the most important concepts of client-server interactions that
  Node.js implements
� Include some examples code snippets of how this is implemented in Node.js
� Will not replace going through a thorough tutorial of JavaScript & Node.js
  (This is not CS31 with JavaScript / Node.js)
We built a tutorial for this. Please go through the tutorial in enough detail for you to
become familiar with Node.js. I recommend to do this before Thursday since we'll
look into React.js & have a React tutorial & React homework then.
If you struggle with JS & Node, please ask on Piazza and/or LA/TA office hours
               CS 35L Software Construction: Lecture 5 � Client Server & Node.js          3
               Tobias D�rschmid
CS 35L Software Construction: Lecture 5 � Client Server & Node.js  4
Self-Study Tips   to Become a Superhero
� Don't just watch video tutorials. Code yourself!
   � You only learn by going & quizzing yourself afterwards
   � That's why we build the interactive tutorials!
� Feeling the effort in the learning process is what makes it so effective!
� Regularly practice concepts from previous lectures in the SE Gym
We are happy to help you, particularly if you have not had much programming
experience yet. If you need help, please come to us directly
Client Server Architectures Define Two Roles
� Multiple clients can use the same server
� Connections are initiated by the client, not the server
� Centralized architecture (all communication flows via the server)
    Client        Request                                            Server
(consumer of      Response                                         (provider of
                                                                   resources)
 resources)
    Client
Client
Alternative: In Peer-to-Peer Architectures (P2)
Clients Directly Interact with other Clients
� Decentralized Architecture          Peer                         Peer
� Peers are equally privileged        Peer                               Peer
  participants in the network
� Peers are both suppliers and        Peer                         Peer
  consumers of resources
� In practice, there are more hybrid
architectures than pure P2P architectures. Would you implement Zoom via
                                            Client-Server or via P2P?
Hybrid means that communication
happens via client-server, and some   It's hybrid, starting with client-server. For video &
communication happens via P2P         audio of 1-on-1 calls, Zoom attempts peer to peer
                                       commutation and uses client server as fallback
Throughput & Latency are important Quality
Attributes for Client-Server Systems
� Throughput measures the volume of work, data, or messages processed
  by a system, network, or process within a specific period of time
  (e.g., the system processes 100 requests per second)
� Latency / Response time measures the time it takes for a single request
  to receive a reply (e.g., the average response time is 10ms)
� Latency can be improved by making the implementation more efficient
� Throughput can be improved by improving latency and/or duplicating
  servers so that more requests can be processed at the same time
How do Requests reach the Server?
                                    Messages from higher layer protocols are
TCP/IP Stack wrapped in messages from lower layer protocols
 Application Layer              Provides an interface for applications to
                              access network services and handles actual
(e.g., HTTP, HTTPS, SSH,
   DNS, POP,TLS/SSL)                data exchange between applications
  Transport Layer            Provides end-to-end communication between
                                  applications running on different hosts
      (e.g., TCP, UDP)
Internet Layer                  Enables communication between networks
                             through addressing and routing data packets
 (e.g., IPv4, IPv6)
      Link Layer             Handles the physical transmission of data over
                                      underlying local network hardware
(e.g., Ethernet, Wifi, MAC)
Each Message has a Header and a Payload
  Headers contain meta information
  (e.g., destination, origin, content type, index number, checksum, ...)
� The payload portion contains the content of the message
� Different protocols define different headers
                                            Message
Header            Payload
Package Wrapping
� Higher-layer protocols use the protocols     Application Layer
  directly below them to send messages.
                                                (e.g., HTTP, HTTPS)
� Whenever this happens, the higher-layer
                                                Transport Layer
message might get split up.
                                                   (e.g., TCP, UDP)
Then it the individual message
                                                 Internet Layer
get placed in the payload portion
                                                   (e.g., IPv4, IPv6)
of the lower-layer messages
                                                   Link Layer
                                   Message
                                                (e.g., Ethernet, MAC)
Ethernet IP       TCP HTTP
                                            Payload
Header Header Header Header
Your Applications use Protocols in the
Application Layer
� Your application can use:             Application Layer
  � HTTP/HTTPS to request files
    documents a server                   (e.g., HTTP, HTTPS,
  � FTP to request static files from a       SSH, DNS,FTP
    server                                   POP,TLS/SSL)
  � POP/SMTP to send emails
  �...                                    Transport Layer
                                            (e.g., TCP, UDP)
                                           Internet Layer
                                            (e.g., IPv4, IPv6)
                                             Link Layer
                                         (e.g., Ethernet, MAC)
HTTP (Hypertext Transfer Protocol)
� The foundation of data                             www.ucla.edu
  communication on the World
  Wide Web                        HTTP REQUEST
                                    HTTP RESPONSE
� Stateless protocol
   � Each request is independent
   � the server doesn't remember
     any information about
     previous requests from the
     same client
HTTP Requests can use different Verbs
� GET
   � Requests a resource (e.g., a web page, data entry, image, file, ...)
   � Response: The content of the resource & status code
� POST
   � Sends data to the server to create or update a resource (e.g., a direct
     message, file upload, complex request objects, ...)
   � Response: status code
� PUT: Updates an existing resource on the server
� DELETE: Deletes a resource on the server
� HEAD: Retrieves only headers of a resource, not body
A URL (Uniform Resource Locator) is the web
address of a resource on the internet
         http://localhost:8080/users/1
           {protocol}://{domain}(:{port})?(/{resource})?
         https://myapp.com/about.html
Common HTTP Status Codes
� 2xx Successful responses (the request was successfully received,
  understood, and accepted)
   � 200 OK: The request was successful.
   � 201 Created: request fulfilled, and new resource created
� 4xx Client error responses (client's request contains an error)
   � 400 Bad Request: invalid request (e.g., malformed syntax)
   � 404 Not Found: The server cannot find the requested resource.
   � Others: 401 Unauthorized, 403 Forbidden, ...
Read more here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
Common HTTP Status Codes
� 5xx Server error responses (the server failed to fulfill a valid request)
   � 500 Internal Server Error: A generic error message indicating an
     unexpected condition on the server.
   � Others: 502 Bad Gateway, 503 Service Unavailable, ...
Read more here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
Common HTTP Header Fields
� Content-Type (the media type of the resource)
   � "text/html; charset=utf-8" for HTML files in UTF-8 encoding
   � "text/plain" for plain text
   � "application/json" for json files (used for API requests / responses)
Read more here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Type
HTTPS (Hypertext Transfer Protocol Secure)
� Uses Secure Sockets Layer (SSL) / Transport Layer Security (TLS) to
  encrypt communication
� Very important whenever sensitive data is
  transferred (e.g., passwords, personally-identifiable
  information, private data, private messages, ...)
� Has become the default for all public web pages,
  even for non-sensitive data
Application Layer Protocols use Transport Layer
Protocols to Transmit Messages
� Higher-layer protocols use the protocols                         Application Layer
  directly below them to send messages.
                                                                   (e.g., HTTP, HTTPS)
� Whenever this happens, the higher-layer
  message might get split up.                                      Transport Layer
  Then it the individual message
  get placed in the payload portion                                   (e.g., TCP, UDP)
  of the lower-layer messages
                                                                     Internet Layer
                                                                      (e.g., IPv4, IPv6)
                                                                       Link Layer
                                                                   (e.g., Ethernet, MAC)
UDP (User Datagram Protocol)
Just "throws" Messages at the Receiver
                                                          Data
In UDP, the Bob simply delivers
packages to Alice's  Here are some
address               packages for
without waiting
                            you
for a response.
UDP Is a Simple, Unreliable,
but Fast & High Throughput Protocol
UDP provides fast,          The End
connectionless, and
lightweight communication.
It does not guarantee       Did she receive
delivery, order, or error   the packages?
checking.                   We don't know.
                            But it was fast!
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� To initiate the connection, Bob   SYN  Hello!
                                         I have some packages
sends a synchronize message (SYN).       for you. Would you like
                                                 them now?
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
                                                      SYN-ACK
� Alice confirms the connection by
sending a SYN-ACK message to                                       Yes, I'm ready
Bob.                                                               to receive the
                                                                     packages!
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
                                                      ACK    Great. Then I
� Bob confirms that the connection                         will give you the
                                                           packages now.
  has been established by sending an
  ACK (acknowledgment) message to
Alice.
        CS 35L Software Construction: Lecture 5 � Client Server & Node.js     26
        Tobias D�rschmid
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� Bob sends data to Alice in multiple  Data  Here is the
  ordered messages that contain a
                                             first package!
checksum to allow Alice to detect
corrupted data and request
retransmission if needed.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� If Alice has received the message.       ACK
  Since it does not contain errors, she
                                                I received the
                                                first package!
sends an acknowledgement (ACK) to
Bob.
� If Bob does not receive the ACK until a
  given timeout, he attempts to send
  the message again.
      CS 35L Software Construction: Lecture 5 � Client Server & Node.js  28
      Tobias D�rschmid
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� Bob sends data to Alice in multiple  Data  Here is the
  ordered messages that contain a
                                             second package!
checksum to allow Alice to detect
corrupted data and request
retransmission if needed.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� If Alice has received the message.       ACK
  Since it does not contain errors, she
                                                      I received the
                                                   second package!
sends an acknowledgement (ACK) to
Bob.
� If Bob does not receive the ACK until a
  given timeout, he attempts to send
  the message again.
      CS 35L Software Construction: Lecture 5 � Client Server & Node.js  30
      Tobias D�rschmid
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
                                                       FIN   That's all I
� To notify Alice that Bob does not                         have for you
  have any more messages to send,
  he sends a FIN message.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� Alice acknowledges Bob's message   FIN ACK
  (ACK). She also does not have any
  more messages to send.                        Okay.
                                              Goodbye
So, she sends a FIN message too.
TCP (Transmission Control Protocol) is a more
Complicated, but Reliable Protocol
� Bob acknowledges Alice's FIN  ACK
message with an ACK message.         Goodbye
To achieve reliability, TCP sends at least 6+2N
messages for N messages of data
Client TCP Server                                       Client UDP Server
SYN       SYN-ACK        Establishes the                Data1
                                                        Data2
ACK    ACK[Data1]        Connection
Data1  ACK[Data2]
                         (to ensure both parties are
Data2                    ready to send / receive data)
                         (Full Duplex)          UPD completes the transfer
                         Data Transfer          before TCP even sends the first
                                                data package
                         (both can send
                         messages at any time)
FIN    FIN-ACK           Terminates the Connection
ACK
                         (to free up resources on both the client and server systems
                         and to ensure reliable closure of the communication session)
       CS 35L Software Construction: Lecture 5 � Client Server & Node.js               34
       Tobias D�rschmid
TCP and UDP Offer Different Trade-Offs
TCP                    Message Order is preserved
                       Error-Detection included
                       Lost messages get re-sent
                       Protocol adds additional overhead
UDP                     Messages can arrive in Any Order
                        No Error-Detection included
                        Some messages Might Get Lost
                        No Additional Delay
     CS 35L Software Construction: Lecture 5 � Client Server & Node.js  35
     Tobias D�rschmid
TCP and UDP Work Well For Different Use Cases
TCP                    Text Messaging (e.g., Slack, Discord)
                       Web Browsing
                       File Transfer
UDP                    Live Video Streaming (e.g., Webcasts)
                       Real-Time Voice Chat (e.g., Audio Call)
     CS 35L Software Construction: Lecture 5 � Client Server & Node.js  36
     Tobias D�rschmid
What about online gaming? Talk to your Neighbor(s)!
TCP and UDP Work Well For Different Use Cases
TCP                    � Ideal for round-based games (e.g., Chess, Go)
                          Many games use a hybrid approach:
                       � Reliable, non-time-sensitive data where loss
                           is fatal (e.g., chat messages, logging into the
                           server, inventory transactions, scoring)
UDP                    � High-speed real-time events
                          (e.g., player positions, physics)
                       � Game updates should not require previous
                          messages, so they need to include the absolute
                          positions of all movable objects
     CS 35L Software Construction: Lecture 5 � Client Server & Node.js      37
     Tobias D�rschmid
IP Addresses
Challenge: Identification             8.8.4.4 120.28.1.4 10.80.14.4 210.82.4.14
How to identify a particular sender/receiver out of the billions of
computers connected to the internet?
  Solution Idea: IP Addresses
Have a portion of the address that represents the network (like a "city")
and a portion that represents the host (like a "street address")
IPv4 addresses range from 0.0.0.0 to 255.255.255.255
There are some special IP addresses, e.g., 127.0.0.1 is "local host",
which is internet-speak for the literal words "your machine"
After running out of all IP addresses, a new standard (IPv6) was created
                                    Messages from higher layer protocols are
TCP/IP Stack wrapped in messages from lower layer protocols
 Application Layer           � TCP/IP is an instance of a layered
                                architecture in which higher-level
(e.g., HTTP, HTTPS, SSH,        layers use only lower-level layers
   DNS, POP,TLS/SSL)
                             � This increases reusability of lower-
  Transport Layer               layer implementations
      (e.g., TCP, UDP)       � It also allows flexibility as you can
                                simply replace one layer with a
    Internet Layer              different implementation to get
                                different results
      (e.g., IPv4, IPv6)
      Link Layer
(e.g., Ethernet, Wifi, MAC)
TCP-IP was invented by     I went to UCLA!
Robert Kahn & Vinton Cerf     Go Bruins
Read more about TCP here: https://www.geeksforgeeks.org/computer-networks/what-is-transmission-control-protocol-tcp/
HTML (Hyper Text Markup Language)
HTML is the most common markup language used to create web pages.
It supports structured content. HTML is not a programming language
<!DOCTYPE html>  Head contains metadata read by the browser
<html>           Body contains the content to be rendered
<head>
<title>Page Title</title>
</head>                      h1 indicates the top heading (others: h2 � h6)
<body>
<h1>This is a Heading</h1>
<p>This is a paragraph.</p>
</body>
</html>
         CS 35L Software Construction: Lecture 5 � Client Server & Node.js   41
         Tobias D�rschmid
Java Script (JS)
� Interpreted, dynamically typed programming language (similar to Python)
� Can be hooked into HTML
// A function that takes two arguments
function add(num1, num2) {
   let sum = num1 + num2;
   return sum; // Returns the result
}
let result = add(5, 7); // result is now 12
console.log(result); // Output: 12
JSON (JavaScript Object Notation)
human-readable, lightweight data-interchange   {
format used to transfer data between a server     "address-dict": {
and a web page, or between applications.              "city": "New York",
                                                      "residential": true,
                                                      "postal_code": 10023
                                                  },
                                                  "phone_numbers": [
                                                      {
                                                         "type": "home",
                                                         "number": "212 555-1234"
                                                      },
                                                      {
                                                         "type": "office",
                                                         "number": "646 555-4567"
                                                      }
                                                  ],
                                                  "children": [
                                                      "Catherine",
                                                      "Thomas",
                                                      "Trevor"
                                                  ]
                                               }
Node.js
� JavaScript runtime for asynchronous events
� Allows you to run JavaScript outside of the browser
� Is not a programming language
  (you are writing code in Java Script to be run within the Node.js environment)
� Fundamentally single-threaded for its main execution of JavaScript code
� Works via callbacks and event handlers
CS 35L Software Construction: Lecture 5 � Client Server & Node.js  45
CS 35L Software Construction: Lecture 5 � Client Server & Node.js  46
Blocking vs. Non-Blocking Code
                           require imports the fs (file system) module from node
                           standard library (similar to import fs as fs)
const fs = require('node:fs');
const declares a constant (cannot be changed after initialization).
Use constants to prevent bugs in you code!
const data = fs.readFileSync('/file.md');
// blocks here until file is read
Read more here: https://nodejs.org/en/learn/asynchronous-work/overview-of-blocking-vs-non-blocking
Blocking vs. Non-Blocking Code
const fs = require('node:fs');
                         Blocking Code
const data = fs.readFileSync('/file.md');
// blocks here until file is read
            Non-Blocking Code                                                Node.js philosophy:
                                                                             use non-blocking
fs.readFile('/file.md', (err, data) => {                                     code for all non-
                                                                             instant tasks to
if (err) {                                                                   maintain high
                                                                             performance and
     ...    Defines an asynchronous callback                                 scalability
}           that executes after the data has been
});         completely stored in the data variable
// following code is executed instantly
          CS 35L Software Construction: Lecture 5 � Client Server & Node.js  48
          Tobias D�rschmid
Define Callbacks Functions
for Long Non-Blocking Code
function handleMarkdownRead(err, data) {  Callbacks Functions
   if (err) {                             allow you to reuse
        ...                               non-blocking code,
   }                                      give it a name, and
                                          make your code
}                                         easier to read
fs.readFile('/file.md', handleMarkdownRead);
// following code is executed instantly
CS 35L Software Construction: Lecture 5 � Client Server & Node.js  50
                                                                            See: https://www.w3schools.com/nodejs/nodejs_http.asp
What does it do? Talk to your Neighbor(s)!
const http = require('http'); Imports http module
const PORT = 3000;     Creates an HTTP server object by defining an callback to be
                       called whenever the server receives an HTTP requires (req)
const server = http.createServer((req, res) => {
res.writeHead(200, { 'Content-Type': 'text/plain' });
                 Sets HTTP response header to status code 200 (success) and plain text content
res.end('Hello, World!\n');
});  Writes 'Hello, World!' into the HTTP response body and sends the message
                     Starts the server on localhost to listen on port 3000
server.listen(PORT, 'localhost', () => {
console.log(`Server running at http://localhost:${PORT}/`);
});                    Like f-strings in Python
     CS 35L Software Construction: Lecture 5 � Client Server & Node.js         51
     Tobias D�rschmid
CS 35L Software Construction: Lecture 5 � Client Server & Node.js  52
                          See https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Server-
                          side/Express_Nodejs/Introduction
What does it do? Talk to your Neighbor(s)!
const express = require('express');  Imports the super helpful express module
const app = express();
const port = 8080;                   that helps you with routing URLs
app.get('/users/:userId', (req, res) => {           Route for all GET requests to
res.send(`GET request to user ${req.params.userId}`); localhost:8080/users/.*
}); Retrieves the user id from the GET request URL
app.post('/', (req, res) => {
res.send('POST request to the homepage'); Route for all POST requests
});                                                 to localhost:8080/
app.get('/about', (req, res) => {    Route for all GET requests to
   res.send('About page');           localhost:8080/about
});
app.all('*', (req, res) => {
   res.status(404).send('404 - Page not found'); Route for all other requests
});
     CS 35L Software Construction: Lecture 5 � Client Server & Node.js                                           53
     Tobias D�rschmid
CS 35L Software Construction: Lecture 5 � Client Server & Node.js  54
                                               See https://nodesource.com/blog/an-absolute-beginners-guide-to-using-npm
NPM � The Node Package Manager
� NPM allows you to install node packages and keep the version
  listed in your package.json
� Node has a large ecosystem of useful package
Common npm cpmmands
$ npm init (creates a new NPM package from your code)
$ npm install express (installs the express package)
$ npm install express@4 (installs version 4 of express)
$ npm install --save (creates a package.json)
$ npm install (installs all packages listed in the package.json)
NodeJS Tutorials
� Course tutorial
   � not a fresh start, builds on what you know from Python & C++
   � Interactive tutorial right in the browser on smaller individual tasks
   � Also teaches some basics of express, communication via HTTP, and JSON
   � https://tobiasduerschmid.github.io/SEBook/tools/nodejs-tutorial
� Video Tutorial
   � https://www.youtube.com/watch?v=32M1al-Y6Ag
   � Practically-minded "bootcamp" on how to build a complete server backend in
     NodeJS
Please Fill out your Exit Tickets on Bruin Learn!
Please summarize three insights you learned about Client Server, Networking,
and/or Node.js today.
Imagine you should implement the video feature for YouTube. Which Application Layer
protocol and which Transport Layer protocol would you use and why?
Please leave any questions that you have about today's materials and things that
are still unclear or confusing to you (if none, simply write N/A).
Credits: These slide use images from Flaticon.com (Creators: Freepik, syafii5758, kerismaker)

```


---

# Appendix: Full root `node.md` reference (verbatim)

This is a reference page for JavaScript and Node.js, designed to be kept open alongside the Node.js Essentials Tutorial. Use it to look up syntax, concepts, and comparisons while you work through the hands-on exercises.

New to Node.js? Start with the interactive tutorial first — it teaches these concepts through practice with immediate feedback. This page is a reference, not a teaching resource.

The Syntax and Semantics: A Familiar Hybrid
If Python and C++ had a child that was raised on the internet, it would be JavaScript. It powers most of the interactive web you use daily, runs on servers via Node.js (used at companies such as LinkedIn, PayPal, Uber, and NASA), and ships in cross-platform desktop apps like VS Code and Discord (via the Electron framework, which embeds Node.js).

From C++, JS inherits its syntax: You will feel right at home with curly braces {}, semicolons ;, if/else statements, for and while loops, and switch statements.
From Python, JS inherits its dynamic nature: Like Python, JS is dynamically typed. You don’t need to declare whether a variable is an int or a string. You don’t have to manage memory explicitly with malloc or new/delete; there are no explicit pointers, and a garbage collector handles memory for you. Modern engines like V8 don’t simply interpret JavaScript — they execute bytecode through a fast interpreter (Ignition) and Just-In-Time-compile hot code paths to native machine code via TurboFan/Maglev.
Variable Declaration: Instead of C++’s int x = 5; or Python’s x = 5, modern JavaScript uses let and const:

let count = 0;       // A variable that can be reassigned
const name = "UCLA"; // A constant that cannot be reassigned
Never use var — it has function-scoped hoisting rules that violate the block-scope behavior you learned in C++ and Python. Always prefer let or const.

What is Node.js? (Taking off the Training Wheels)
Historically, JavaScript was trapped inside the web browser. It was strictly a front-end language used to make websites interactive.

Node.js is a runtime environment that takes JavaScript out of the browser and lets it run directly on your computer’s operating system. It embeds Google’s V8 engine to execute code, but also includes a powerful C library called libuv to handle the asynchronous event loop and system-level tasks like file I/O and networking. This means you can use JavaScript to write backend servers just like you would with Python or C++.

Here is how JavaScript (via Node.js) fits into your mental model from C++ and Python:

Node.js table
Aspect	C++	Python	JavaScript (Node.js)
Typing	Static	Dynamic	Dynamic
Memory	Manual (new/delete)	GC (reference counting + cycle collector)	GC (V8: generational, tracing)
Run with	Compile → ./app	python script.py	node script.js
I/O model	Synchronous (blocks)	Synchronous (blocks)	Asynchronous (non-blocking)
Running a script: Like Python, there is no compilation step. You run a JavaScript file directly:

node script.js
And like Python, there is no required main() function — Node.js executes scripts top-to-bottom. V8 JIT-compiles the code at runtime.

Printing output: JavaScript’s equivalent of Python’s print() and C++’s printf() is console.log(). It writes to stdout with a trailing newline:

// Python equivalent: print("Hello from Node.js!")
// C++ equivalent:    printf("Hello from Node.js!\n");
console.log("Hello from Node.js!");
The Paradigm Shift: Asynchronous Programming
Here is the largest “threshold concept” you must cross: JavaScript is fundamentally asynchronous and single-threaded.

In C++ or Python, if you make a network request or read a file, your code typically stops and waits (blocks) until that task finishes. In Node.js, blocking the main thread is a cardinal sin. Instead, Node.js uses an Event Loop. When you ask Node.js to read a file, it delegates that task to the operating system and immediately moves on to execute the next line of code. When the file is ready, a “callback” function is placed in a queue to be executed.

Mental Model Adjustment: You must stop thinking of your code as executing strictly top-to-bottom. You are now setting up “listeners” and “callbacks” that react to events as they finish.

NPM: The Node Package Manager
If you remember using #include <vector> in C++ or import requests (via pip) in Python, Node.js has NPM. NPM is a massive ecosystem of open-source packages. Whenever you start a new Node.js project, you will run:

npm init (creates a package.json file to track your dependencies)
npm install <package_name> (downloads code into a node_modules folder)
Worked Example: A Simple Client-Server Setup
Let’s look at how you would set up a basic web server in Node.js using a popular framework called Express (which you would install via npm install express).

Notice the syntax connections to C++ and Python:

// 'require' is JS's version of Python's 'import' or C++'s '#include'
const express = require('express'); 
const app = express(); 
const port = 8080;

// Route for a GET request to localhost:8080/users/123
app.get('/users/:userId', (req, res) => { 
    // Notice the backticks (`). This allows string interpolation.
    // It is exactly like f-strings in Python: f"GET request to user {userId}"
    res.send(`GET request to user ${req.params.userId}`); 
}); 

// Route for all POST requests to localhost:8080/
app.post('/', (req, res) => { 
    res.send('POST request to the homepage'); 
}); 

// Start the server
app.listen(port, () => {
    console.log(`Server listening on port ${port}`);
});
Breakdown of the Example:

Arrow Functions (req, res) => { ... }: This is a concise way to write an anonymous function. You are passing a function as an argument to app.get(). This is how JS handles asynchronous events: “When someone makes a GET request to this URL, run this block of code.”
req and res: These represent the HTTP Request and HTTP Response objects, abstracting away the raw network sockets you would have to manage manually in lower-level C++.
The === Trap: Type Coercion
JavaScript has TWO equality operators. Only ever use ===:

// WRONG: == triggers implicit type coercion — a JS-specific danger
console.log(1 == "1");    // true  ← DANGEROUS SURPRISE
console.log(0 == false);  // true  ← DANGEROUS SURPRISE

// RIGHT: === checks value AND type (behaves like == in Python and C++)
console.log(1 === "1");   // false ← correct
console.log(0 === false); // false ← correct
This is negative transfer: your == intuition from C++ and Python is correct — but JavaScript’s == does something different. Use === and it matches your expectation.

JavaScript’s Two “Nothings”: null vs undefined
C++ has nullptr. Python has None. JavaScript has two distinct values meaning “nothing”:

let score;                // declared but no value assigned → undefined
console.log(score);       // undefined
console.log(typeof score); // "undefined"

let student = null;       // explicitly set to "no value"
console.log(student);     // null
console.log(typeof student); // "object" (a famous JS bug that can never be fixed)
Node.js table
Concept	undefined	null
Meaning	“no value was assigned yet”	“intentionally empty”
When you see it	Uninitialized variables, missing function args, req.query.missing	You (or an API) explicitly set it
typeof	"undefined"	"object" (a historical JS bug)
Python equivalent	No direct equivalent (NameError)	None
Watch out: null == undefined is true (coercion!), but null === undefined is false. One more reason to always use ===.

Control Flow Syntax
JavaScript’s control flow looks like C++ (braces required), not Python (no colons/indentation):

// if/else — braces required (no colons like Python, no elif — use else if)
if (score >= 90) {
    console.log("A");
} else if (score >= 60) {
    console.log("Pass");
} else {
    console.log("Fail");
}

// for loop — same structure as C++
for (let i = 0; i < 5; i++) {
    console.log(i);
}

// for...of — like Python's "for x in list"
const names = ["Alice", "Bob", "Carol"];
for (const name of names) {
    console.log(name);
}
Functions as First-Class Values
In C++ you’ve encountered function pointers. In Python, you’ve passed functions to sorted(key=...). JavaScript takes this further: functions are just values, exactly like numbers or strings.

Arrow functions are the modern preferred syntax:

// C++ equivalent: int add(int a, int b) { return a + b; }
// Python equivalent: lambda a, b: a + b

const add    = (a, b) => a + b;
const greet  = (name) => `Hello, ${name}!`;
const double = n => n * 2;           // Parens optional for single param
.map(), .filter(), .reduce()
These array methods take callback functions — the same “functions as values” concept. They are the JavaScript equivalents of Python’s map(), filter(), and functools.reduce():

const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(n => n * 2);              // [2, 4, 6, 8, 10]
const evens   = numbers.filter(n => n % 2 === 0);     // [2, 4]
const sum     = numbers.reduce((acc, n) => acc + n, 0); // 15
.find() returns the first matching element (or undefined if none match) — use it when you need one specific item:

const students = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];
const alice = students.find(s => s.id === 1);   // { id: 1, name: "Alice" }
const missing = students.find(s => s.id === 99); // undefined
Understanding callbacks is essential — all of Node.js’s async operations notify you they are finished by calling a function you provided.

Destructuring: Unpacking Values
JavaScript has compact syntax for extracting values from arrays and objects:

// Array destructuring (like Python's tuple unpacking: r, g, b = color)
const [red, green, blue] = [255, 128, 0];

// Object destructuring (extract properties by name)
const config = { host: "localhost", port: 3000, debug: true };
const { host, port } = config;   // host = "localhost", port = 3000

// Works in function parameters — you will see this in every Express route and React component:
function startServer({ host, port }) {
    console.log(`Listening on ${host}:${port}`);
}
Formatting Output: .toFixed() and .padEnd()
Two utilities you will use when formatting output:

// .toFixed(n) — format a number to exactly n decimal places (returns a string)
const avg = 87.666;
console.log(avg.toFixed(1));   // "87.7"
console.log(avg.toFixed(2));   // "87.67"

// .padEnd(n) — pad a string with spaces to reach length n (left-aligns text in columns)
console.log("Alice".padEnd(7) + "| 95");   // "Alice  | 95"
console.log("Bob".padEnd(7) + "| 42");     // "Bob    | 42"

// .padStart(n) — pad from the left (right-aligns text)
console.log("42".padStart(5));   // "   42"
Ready to Practice?
Head to the Node.js Essentials Tutorial for hands-on exercises with immediate feedback — no setup required.

The Event Loop in Detail
The Event Loop is best understood with the Restaurant Metaphor:

Node.js table
Kitchen Role	Node.js Equivalent	What It Does
The Chef	Call Stack	Executes one task at a time. If busy, everything else waits.
The Appliances (oven, fryer)	libuv / OS	Handle slow work (file reads, network) in the background.
The Waiter	Task Queue	When an appliance finishes, the callback is queued.
The Kitchen Manager	Event Loop	Only when the Chef’s hands are completely empty does the Manager hand over the next callback.
The critical insight: setTimeout(fn, 0) does NOT mean “run immediately”. It means “run when the call stack is empty”. Synchronous code always runs to completion before any callback fires:

setTimeout(() => console.log("B"), 0);   // queued in Task Queue
console.log("A");                        // runs immediately
console.log("C");                        // runs immediately
// Output: A, C, B  (NOT A, B, C!)
This is why blocking the main thread with a long synchronous operation is catastrophic in Node.js — it prevents ALL other requests, timers, and I/O callbacks from being processed.

Modern Asynchrony: Promises and Async/Await
In the earlier example, we mentioned that Node.js uses “callbacks” to handle events. However, nesting multiple callbacks inside one another leads to a notoriously difficult-to-read structure known as “Callback Hell”.

To manage cognitive load and make asynchronous code easier to reason about, modern JavaScript introduced Promises (conceptually similar to std::future in C++) and the async/await syntax.

A Promise is exactly what it sounds like: an object representing the eventual completion (or failure) of an asynchronous operation. Using async/await allows you to write asynchronous code that looks and reads like traditional, synchronous C++ or Python code.

Creating a Promise: The new Promise(...) constructor takes a single function (called the executor) that receives two arguments — resolve (call when the work succeeds) and reject (call when it fails):

// Under the hood, this is how async operations are built:
const promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("data ready!"), 100);
});

// Consuming it with .then():
promise.then(data => console.log(data));   // "data ready!" after 100ms
In practice you rarely create Promises from scratch — you mostly consume them using await or .then(). Libraries like fs.promises and fetch return Promises for you.

Node.js async syntax evolved through three generations. You need to recognize all three — and write the third:

Generation 1: Callbacks — each async operation nests inside the previous one (“Callback Hell”):

fetchData('a', (err, dataA) => {
    if (err) throw err;
    fetchData('b', (err2, dataB) => {  // "Pyramid of Doom"
        if (err2) throw err2;
    });
});
Generation 2: Promises — flatten the nesting with .then() chains:

fetchData('a')
    .then(dataA => fetchData('b'))
    .then(dataB => console.log(dataB))
    .catch(err  => console.error(err));
Generation 3: async/await — looks like synchronous code but doesn’t block:

async function fetchUserData(userId) {
    try {
        // 'await' suspends THIS function (non-blocking!) and lets other work proceed
        const response = await database.getUser(userId);
        console.log(`User found: ${response.name}`);
    } catch (error) {
        // Error handling looks exactly like C++ or Python
        console.error(`Error fetching user: ${error.message}`);
    }
}
When JavaScript hits await, it suspends the async function, frees the call stack, and lets the Event Loop process other work. When the Promise resolves, execution resumes. This looks like synchronous C++/Python code — but it does NOT block the event loop.

Sequential vs Parallel: If two operations are independent, use Promise.all() for better performance:

// SLOWER: sequential — total time = time(A) + time(B)
const a = await fetchA();
const b = await fetchB();

// FASTER: parallel — total time = max(time(A), time(B))
const [a, b] = await Promise.all([fetchA(), fetchB()]);
⚠️ The .forEach() Trap: .forEach() does NOT await async callbacks — it fires them all and returns immediately:

// BUG: "All done!" prints BEFORE items are processed
items.forEach(async (item) => {
    await processItem(item);
});
console.log("All done!");  // runs immediately!

// FIX (sequential): use for...of
for (const item of items) {
    await processItem(item);
}
console.log("All done!");  // runs after all items

// FIX (parallel): use Promise.all + .map()
await Promise.all(items.map(item => processItem(item)));
console.log("All done!");
.forEach() ignores the Promises returned by its async callbacks — it has no mechanism to wait for them. This is one of the most common async bugs in JavaScript.

Data Representation: JavaScript Objects and JSON
If you understand Python dictionaries, you already understand the general structure of JavaScript Objects. Unlike C++, where you must define a struct or class before instantiating an object, JavaScript allows you to create objects on the fly using key-value pairs.

Wait, what about JSON? While they look similar, JSON (JavaScript Object Notation) is a strict data-interchange format. Unlike JS objects, JSON requires double quotes for all keys and string values, and it cannot store functions or special values like undefined. JSON is simply this structure serialized into a string format so it can be sent over a network.

// This is a JavaScript Object (similar to a Python dictionary, but keys are coerced to strings/Symbols and objects also have a prototype chain)
const student = {
    name: "Joe Bruin",
    uid: 123456789,
    courses: ["CS31", "CS32", "CS35L"],
    isGraduating: false
};

// Accessing properties is done via dot notation (like C++ objects)
console.log(student.courses[2]); // Outputs: CS35L
JSON is simply this exact object structure serialized into a string format so it can be sent over an HTTP network request.

Tips for Mastering JS/Node.js
Here is how you should approach mastering this new ecosystem:

Utilize Pair Programming: Don’t learn Node.js in isolation. Sit at a single screen with a peer (one “Driver” typing, one “Navigator” reviewing and strategizing). Research shows pair programming significantly increases confidence and code quality while reducing frustration for novices transitioning to a new language paradigm (McDowell et al. 2006; Cockburn and Williams 2000; Williams and Kessler 2000).
Embrace Test-Driven Development (TDD): In Python, you might have used pytest; in C++, gtest. In JavaScript, frameworks like Jest are the standard. Before you write a complex API endpoint in Express, write a test for what it should do. This acts as a formative assessment, giving you immediate, automated feedback on whether your mental model of the code aligns with reality.
Avoid “Vibe Coding” with AI: While Large Language Models (LLMs) can generate Node.js boilerplate instantly, relying on them before you understand the asynchronous Event Loop will lead to “unsound abstractions”. Use AI to explain confusing syntax or error messages, but do not let it rob you of the cognitive struggle required to build your own notional machine of how JavaScript executes.
Top 10 JavaScript & Node.js Best Practices
These are the most important conventions and idioms that experienced JavaScript developers follow. Internalizing them will make your code more predictable, less error-prone, and immediately recognizable as modern JavaScript.

1. Default to const, Use let Only When Reassigning, Never Use var
const prevents accidental reassignment and signals intent. let is for values that genuinely change. var has broken scoping rules — never use it.

// ✓ const — value never changes
const MAX_RETRIES = 3;
const students = ["Alice", "Bob"];  // The array can be mutated, but the binding cannot

// ✓ let — value changes
let count = 0;
for (let i = 0; i < 5; i++) {
    count += i;
}

// ✗ Never use var — it leaks out of blocks and hoists unexpectedly
var x = 10;
if (true) { var x = 20; }
console.log(x);  // 20 — surprised?
Note: const prevents reassignment, not mutation. A const array can still be .push()-ed to. To prevent mutation, use Object.freeze().

2. Always Use === (Strict Equality), Never ==
JavaScript’s == performs implicit type coercion, producing dangerous surprises. === checks both value AND type — matching the behavior you expect from C++ and Python.

// ✓ Strict equality — no surprises
1 === "1"     // false
0 === false   // false
"" === false  // false

// ✗ Loose equality — implicit coercion traps
1 == "1"      // true  ← DANGER
0 == false    // true  ← DANGER
"" == false   // true  ← DANGER
The same applies to !== (use it) vs != (avoid it).

3. Use async/await for Asynchronous Code
Modern JavaScript uses async/await for asynchronous operations. It reads like synchronous code while remaining non-blocking. Always wrap await in try/catch.

// ✓ Modern: async/await with error handling
async function loadData() {
    try {
        const data = await fetchFromAPI();
        return process(data);
    } catch (err) {
        console.error("Failed to load:", err.message);
    }
}

// ✗ Avoid: deeply nested callbacks ("Callback Hell")
fetchA((err, a) => {
    fetchB((err, b) => {
        fetchC((err, c) => { /* pyramid of doom */ });
    });
});
4. Use Promise.all() for Independent Async Operations
When two operations do not depend on each other, run them concurrently. Sequential await wastes time.

// ✓ Concurrent — total time = max(time(A), time(B))
const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
]);

// ✗ Sequential — total time = time(A) + time(B)
const users = await fetchUsers();   // waits...
const posts = await fetchPosts();   // then waits again
5. Use Template Literals for String Formatting
Backtick strings with ${expression} are JavaScript’s equivalent of Python’s f-strings. They are more readable and less error-prone than + concatenation.

const name = "Alice";
const score = 95;

// ✓ Template literal — clear and concise
const msg = `${name} scored ${score} points`;

// ✗ Concatenation — verbose and easy to break
const msg = name + " scored " + score + " points";
Template literals also support multi-line strings and arbitrary expressions inside ${}.

6. Use Arrow Functions for Callbacks
Arrow functions are concise and lexically bind this (they inherit this from the enclosing scope, avoiding a common class of bugs).

const numbers = [1, 2, 3, 4, 5];

// ✓ Arrow functions — concise
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// ✗ Verbose equivalent
const doubled = numbers.map(function(n) { return n * 2; });
When NOT to use arrow functions: Object methods that need their own this, and constructor functions.

7. Use Destructuring to Extract Values
Destructuring makes code more concise and self-documenting by extracting values from objects and arrays in one step.

// ✓ Object destructuring
const { name, grade } = student;

// ✓ In function parameters (common in React)
function printStudent({ name, grade }) {
    console.log(`${name}: ${grade}`);
}

// ✓ Array destructuring with Promise.all
const [roster, grades] = await Promise.all([fetchRoster(), fetchGrades()]);

// ✗ Verbose alternative
const name = student.name;
const grade = student.grade;
8. Never Block the Event Loop
Node.js is single-threaded. Blocking the main thread prevents ALL other requests, timers, and callbacks from executing. Always use asynchronous I/O.

// ✓ Non-blocking — other requests can proceed
const data = await fs.promises.readFile("data.json", "utf8");

// ✗ Blocking — entire server freezes until file is read
const data = fs.readFileSync("data.json", "utf8");
For CPU-intensive work, offload to Worker Threads instead of running it on the main thread.

9. Use Optional Chaining (?.) and Nullish Coalescing (??)
These modern operators replace verbose null-checking patterns and make code more robust.

// ✓ Optional chaining — safe deep access
const city = user?.address?.city;           // undefined if any link is null
const first = results?.[0];                 // safe array access

// ✓ Nullish coalescing — default only for null/undefined
const port = config.port ?? 3000;           // 0 is preserved as valid
const name = user.name ?? "Anonymous";      // "" is preserved as valid

// ✗ Verbose null checking
const city = user && user.address && user.address.city;

// ✗ || treats 0, "", and false as "missing"
const port = config.port || 3000;           // if port is 0, uses 3000!
10. Use .map(), .filter(), .reduce() Instead of Manual Loops
These array methods are more declarative, less error-prone, and do not mutate the original array. They are the JavaScript equivalents of Python’s map(), filter(), and functools.reduce().

const students = [
    { name: "Alice", grade: 95 },
    { name: "Bob",   grade: 42 },
    { name: "Carol", grade: 78 },
];

// ✓ Declarative — chain operations fluently
const honors = students
    .filter(s => s.grade >= 90)
    .map(s => s.name);
// ["Alice"]

// ✗ Imperative — more code, mutation, more room for bugs
const honors = [];
for (let i = 0; i < students.length; i++) {
    if (students[i].grade >= 90) {
        honors.push(students[i].name);
    }
}
Use regular for loops when you need early termination (break), when performance on very large arrays matters, or when the logic is too complex for a single chain.

Practice
Node.js/JavaScript Syntax — What Does This Code Do?
You are shown JavaScript/Node.js code. Explain what it does and what it outputs.

Difficulty:
Basic
You are shown JavaScript/Node.js code. Explain what it does and what it outputs.

let count = 0;
const MAX = 200;
Show Answer
Node.js/JavaScript Syntax — Write the Code
You are given a task description. Write the JavaScript code that accomplishes it.

Difficulty:
Intermediate
Check if a variable userInput (which might be a string) equals the number 42, without being tricked by type coercion.

Show Answer
Node.js Concepts Quiz
Test your deeper understanding of JavaScript's async model, type system, and paradigm differences from C++ and Python. Includes Parsons problems, technique-selection questions, and spaced interleaving across all concepts.

Difficulty:
Intermediate
You are building a TikTok-style feed. Match each task to the best array method:

Task A: Remove videos the user has already seen
Task B: Convert each video object into a <VideoCard> component
Task C: Calculate the total watch time across all videos

A
A: .filter(), B: .map(), C: .reduce()


B
A: .filter(), B: .reduce(), C: .map()


C
A: .reduce(), B: .map(), C: .filter()


D
A: .map(), B: .filter(), C: .reduce()

References
(Cockburn and Williams 2000): Alistair Cockburn and Laurie Williams (2000) “The costs and benefits of pair programming,” International Conference on Extreme Programming and Flexible Processes in Software Engineering (XP), pp. 223–243.
(McDowell et al. 2006): Charlie McDowell, Linda Werner, Heather E. Bullock, and Julian Fernald (2006) “Pair programming improves student retention, confidence, and program quality,” Communications of the ACM, 49(8), pp. 90–95.
(Williams and Kessler 2000): Laurie A. Williams and Robert R. Kessler (2000) “All I really need to know about pair programming I learned in kindergarten,” Communications of the ACM, 43(5), pp. 108–114.