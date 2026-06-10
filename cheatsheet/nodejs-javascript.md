# Node.js / JavaScript Cheatsheet (candidate items)

## JS basics
- `let`, `const`, `var` (use `let`/`const`, not `var`)
- `const` immutable binding (object contents still mutable)
- Arrow function: `(x) => x*2`
- Template literal: `` `Hello ${name}` ``
- Destructure: `const {a, b} = obj;` `const [x, y] = arr;`
- Spread: `[...arr]`, `{...obj}`, `f(...args)`

## == vs ===
- `==` loose equality (type coercion!) — `"5" == 5` is `true`
- `===` strict equality (no coercion) — `"5" === 5` is `false`
- **Always use `===`** (and `!==`)

## Truthy / Falsy
Falsy: `false`, `0`, `""`, `null`, `undefined`, `NaN`
Everything else truthy (including `[]` and `{}`!)

## null vs undefined
- `undefined` — variable not assigned
- `null` — explicitly empty
- `typeof undefined === "undefined"`
- `typeof null === "object"` (JS bug, historical)

## Functions
```js
function f(x) { return x * 2; }          // declaration (hoisted)
const f = function(x) { return x*2; };    // expression
const f = (x) => x * 2;                   // arrow
const f = x => x * 2;                     // arrow, single arg no parens
const f = () => ({ a: 1 });               // wrap obj literal in parens
```
- Arrow does NOT have its own `this` (inherits from enclosing scope)

## Array methods
```js
arr.push(x) / arr.pop()           // end
arr.shift() / arr.unshift(x)      // start
arr.slice(a, b)                    // returns new
arr.splice(i, n, ...items)         // mutates
arr.indexOf(x)
arr.includes(x)
arr.map(f)
arr.filter(pred)
arr.reduce((acc, x) => ..., init)
arr.forEach(f)
arr.find(pred)
arr.some(pred) / arr.every(pred)
arr.sort((a,b) => a-b)             // default is LEX (string sort)!
```

## Object
```js
const o = { name: "Alice", age: 30 };
o.name; o["name"];
Object.keys(o) / Object.values(o) / Object.entries(o)
delete o.name;
"name" in o;
```

## Async / promises / await
```js
async function f() {
    try {
        const x = await fetchData();
        return x.value;
    } catch (e) { ... }
}

p.then(x => ...).catch(e => ...);

Promise.all([p1, p2, p3])      // wait for all
Promise.race([p1, p2])         // first to finish
Promise.any([p1, p2])          // first to fulfill
```

## Callback / event loop / blocking vs non-blocking
- Node is single-threaded but uses **event loop**
- Blocking: `fs.readFileSync`, `JSON.parse` of huge string
- Non-blocking: `fs.readFile(path, cb)`, async I/O
- I/O happens in background; callback fires on event loop tick

## Modules
**CommonJS** (Node default):
```js
const fs = require("fs");
module.exports = { foo };
```
**ES modules** (`.mjs` or `"type":"module"`):
```js
import fs from "fs";
export const foo = ...;
export default obj;
```

## JSON
```js
JSON.stringify(obj)         // obj → string
JSON.parse(str)             // string → obj
```
- Throws on invalid JSON — wrap in try/catch
- Functions, undefined, symbols are dropped/converted

## npm
- `npm init` create `package.json`
- `npm install <pkg>` add dependency
- `npm install --save-dev <pkg>` dev-only
- `npm install -g <pkg>` global
- `npm install` install from package.json
- `npm run <script>` run script from package.json
- `package-lock.json` locks exact versions

## HTTP server (Node only)
```js
const http = require("http");
const server = http.createServer((req, res) => {
    res.writeHead(200, {"Content-Type": "text/plain"});
    res.end("Hello");
});
server.listen(3000);
```

## Express
```js
const express = require("express");
const app = express();
app.get("/", (req, res) => res.send("hi"));
app.post("/api", (req, res) => res.json({...}));
app.use(middleware);
app.all("*", (req, res) => res.status(404).send("Not found"));
app.listen(3000);
```

## Closures
```js
function makeCounter() {
    let n = 0;
    return () => ++n;
}
const c = makeCounter();
c(); c(); c();   // 1, 2, 3
```

## this gotcha
- `this` is determined by HOW a function is called, not WHERE it's defined
- Arrow functions inherit `this` from enclosing scope (no own `this`)
- `bind(obj)`, `call(obj, ...)`, `apply(obj, [...])` to set `this`

## Common traps
- `==` does type coercion (use `===`)
- `[] == false` is true
- `typeof null === "object"`
- `parseInt("08")` historically returned 0 (no longer, but be explicit: `parseInt(s, 10)`)
- Array sort is string-based by default
- `this` inside callbacks loses context (use arrow or `bind`)

## Additional items (potentially missing)

### process module (Node)
- `process.argv` — command-line args (`[node, script, ...args]`)
- `process.env.NAME` — environment variable
- `process.exit(0)` — exit with code
- `process.cwd()` — current dir
- `process.platform` — 'win32', 'darwin', 'linux'
- `process.version` — Node version

### Reading argv example
```js
const args = process.argv.slice(2);  // skip node + script
console.log(args[0]);
```

### path module
```js
const path = require("path");
path.join("a", "b", "c.txt")        // "a/b/c.txt" or "a\b\c.txt"
path.basename("/a/b/c.txt")         // "c.txt"
path.dirname("/a/b/c.txt")          // "/a/b"
path.extname("/a/b/c.txt")          // ".txt"
path.resolve("a", "b")              // absolute path
```

### fs module (sync vs async)
```js
const fs = require("fs");

// Sync — blocks event loop (only use at startup)
const data = fs.readFileSync("f.txt", "utf-8");

// Async with callback
fs.readFile("f.txt", "utf-8", (err, data) => {
    if (err) throw err;
    console.log(data);
});

// Async with promises
const fsp = require("fs/promises");
const data = await fsp.readFile("f.txt", "utf-8");

// Write
fs.writeFileSync("out.txt", "data");
fs.appendFileSync("log.txt", "line\n");
```

### Common fs operations
- `fs.existsSync(path)`
- `fs.mkdirSync(path, {recursive: true})`
- `fs.readdirSync(dir)` — list files
- `fs.statSync(path)` — file info
- `fs.unlinkSync(path)` — delete file

### Buffer
- Binary data type
- `Buffer.from("hi", "utf-8")`
- `buf.toString("base64")`
- Used in fs (when no encoding given), networking

### Streams
- Readable: `fs.createReadStream(path)`
- Writable: `fs.createWriteStream(path)`
- `.pipe(other)` — chain
- Used for large files / network I/O

### URL parsing
```js
const u = new URL("https://example.com:8080/path?x=1");
u.hostname        // "example.com"
u.pathname        // "/path"
u.searchParams.get("x")    // "1"
```

### Error handling patterns
```js
// Promise rejection
p.then(...).catch(e => ...);

// Async/await
try {
    const x = await p;
} catch (e) {
    handle(e);
}

// Callback (older Node)
fn((err, result) => {
    if (err) return handleError(err);
    use(result);
});
```

### setTimeout / setInterval
```js
setTimeout(fn, 1000);             // run once after 1s
const id = setInterval(fn, 1000); // every 1s
clearInterval(id);                // stop
```

### typeof
```js
typeof undefined      // "undefined"
typeof "hi"           // "string"
typeof 42             // "number"
typeof true           // "boolean"
typeof null           // "object"  (historical bug)
typeof {}             // "object"
typeof []             // "object" (!)
typeof function(){}   // "function"
```
For arrays: `Array.isArray(x)`

### Numeric quirks
- `0.1 + 0.2 === 0.3`  // false (floating point!)
- `Number.MAX_SAFE_INTEGER` = 2^53 - 1
- `NaN !== NaN` (use `Number.isNaN(x)`)
- `Infinity`, `-Infinity`
- `parseInt("10", 2)` — parse as binary

### Spread / rest
```js
const a = [1,2,3];
const b = [...a, 4, 5];           // spread
const {x, y, ...rest} = obj;      // rest
function f(...args) { ... }       // rest in params
f(...[1,2,3]);                    // spread in call
```

### Optional chaining & nullish coalescing
```js
user?.address?.zip                // returns undefined if any null
const port = process.env.PORT ?? 3000;   // ?? = null/undefined fallback (NOT ||)
```

### JSON nuances
```js
JSON.stringify(obj, null, 2);     // pretty-print with 2-space indent
JSON.stringify(obj, ['name', 'age']);  // replacer = whitelist
JSON.parse(str, (key, val) => ...);    // reviver function
```

### Express middleware basics
```js
app.use(express.json());            // parse JSON body → req.body
app.use(express.static('public'));  // serve static files

app.use((req, res, next) => {
    console.log(req.method, req.url);
    next();                         // continue chain
});

// Error-handling middleware (4 args!)
app.use((err, req, res, next) => {
    res.status(500).send(err.message);
});
```

### Common HTTP request bodies
```js
// In Express
app.post("/api", (req, res) => {
    const data = req.body;          // requires express.json() middleware
    res.json({ ok: true });
});
```
