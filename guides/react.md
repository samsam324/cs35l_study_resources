# CS 35L Study Guide — React, UI Development & Design Patterns

This guide combines the React reference page with Lecture 6 (UI Development, Design Patterns, & React) by Tobias Durschmid. Use it as a dense exam reference: every concept from both sources is preserved.

---

## 1. Motivating Case Study: Digital Monopoly Game

- There are dozens of implementations of "Monopoly" with vastly different User Interfaces — from pixel boards, to live-action TV-show style, to **plain text UI** built from three boxes:
  - **Next Turn (Mohamed)** — "It is Mohamed's turn." [Roll Dice]
  - **Dice Rolled (Mohamed)** — "The first die shows 6. The second die shows 2." [Okay]
  - **Player Decision Requested (Mohamed)** — "You landed on the unpurchased property 'Royce Hall'. Do you want to buy 'Royce Hall' for Bruin$800? Your current inventory is Bruin$4250." [Yes (-Bruin$800)] [No]
- **What should they all have in common?** All of them should implement **the exact same game logic**.
- So at least in theory they could all run the exact same code, just with a different UI — ideally even with multiple UIs simultaneously.

This motivates the central design principle: **separate UI from logic**.

---

## 2. Separation of Concerns (SoC)

- Systems should be **divided** into **distinct sections**, or **concerns**, where each section addresses a separate, specific goal, purpose, or responsibility.
- The goal is to make the system **easier to develop, maintain, and evolve**.
- This is an **important general design principle**. It applies to **all** software development. Not just UI.
- Classic split: **Front-End** vs **Back-End**.

### Separate the UI Implementation from the Domain Logic

Two layers:

- **Presentation Layer** — Displays information to the user and collects input (e.g., positions of players on the board, style of the boards, buttons, ...).
- **Application Layer** — Implements the **domain logic and behavior** (e.g., effects of community chest cards, player turns, interactions between players). The application layer **doesn't have a clue that there even is a UI**.

The Application Layer's API to the Presentation Layer:

- Allows the presentation layer to register a **callback** for state changes (e.g., `onBalanceChanged`).
- Allows the presentation layer to retrieve information using **getter functions** (e.g., `getCurrentBalance()`).
- Allows the presentation layer to forward user events (e.g., `buyProperty(name, user)`).

---

## 3. The Observer Design Pattern (briefly — full coverage in design-patterns.md)

The lecture introduces the Observer pattern as the mechanism that lets the UI listen to the domain layer without the domain layer knowing about the UI. Full details belong in `design-patterns.md`; here is the lecture summary.

- **Context** — Scenarios requiring distributed event-handling systems or highly decoupled architectures like GUI and signal processing.
- **Problem** — How can a one-to-many dependency between objects be maintained efficiently without making the objects tightly coupled?
- **Solution** — Two main roles:
  - **Subject** — the object sending updates after it has changed (e.g., your domain object with a lot of business logic).
  - **Observer** — the object listening to the updates of Subjects (e.g., the presentation-layer UI element that displays the data of the subject whenever it changes).

Pattern descriptions have many different formats. At minimum each should describe **problem, context, and solution**.

### Class roles

- `Subject` — `register(Observer)`, `unregister(Observer)`, `notify(Event)`, `Data getData()`, `doSomething()`.
- `AbstractObserver` — abstract `update(Event)`.
- `Concrete Observer` — concrete `update(Event)`.
- Sequence: Observer calls `register(self)` on Subject (adds observer to list). When Subject's `doSomething()` runs, it calls `notify(someEvent)` which iterates over the observer list and calls each `update()`. Each observer then calls `getData()` on the subject to pull data.

### Observer pattern in Python (lecture code)

```python
# OBSERVER INTERFACE
class Subscriber(ABC):
    """The Observer interface."""
    @abstractmethod
    def update(self):
        pass

# SUBJECT
class NewsChannel:
    """The Subject that maintains a list of subscribers."""
    def __init__(self):
        self._subscribers: list[Subscriber] = []
        self._latest_post: str = ""

    def follow(self, subscriber: Subscriber):
        if subscriber not in self._subscribers:
            self._subscribers.append(subscriber)

    def unfollow(self, subscriber: Subscriber):
        self._subscribers.remove(subscriber)

    def publish_post(self, text: str):
        self._latest_post = text
        self._notify_subscribers()

    def get_latest_post(self) -> str:
        return self._latest_post

    def _notify_subscribers(self):
        for subscriber in self._subscribers:
            subscriber.update()

# CONCRETE OBSERVERS
class MobileApp(Subscriber):
    """A concrete observer that pulls state from the channel."""
    def __init__(self, channel: NewsChannel):
        self._channel = channel
    def update(self):
        post = self._channel.get_latest_post()
        print(f"[MobileApp] Push notification: {post}")

class EmailDigest(Subscriber):
    """Another concrete observer with different behavior."""
    def __init__(self, channel: NewsChannel):
        self._channel = channel
    def update(self):
        post = self._channel.get_latest_post()
        print(f"[EmailDigest] New email queued: {post}")

# CLIENT CODE
channel = NewsChannel()
app = MobileApp(channel)
email = EmailDigest(channel)
channel.follow(app)
channel.follow(email)
channel.publish_post("New video uploaded!")
# [MobileApp] Push notification: New video uploaded!
# [EmailDigest] New email queued: New video uploaded!
channel.unfollow(email)
channel.publish_post("Live stream starting!")
# [MobileApp] Push notification: Live stream starting!
```

### Design patterns — definition

> "A *pattern* is a **common, acceptable solution** to a reoccurring **problem** that arises in a specific **context**."

- "common" — found in **many instances**.
- "acceptable" — it is a **good** solution, but **not always the best** one.
- "context" — patterns always refer to a specific **situation, goal, or trade-off**. Patterns are **not universally** good.
- "problem" — the problem is generic enough that it **generalizes** beyond a few concrete cases.
- Design Patterns **add roles to classes / objects**. The `NewsChannel` / `MobileApp` / `EmailDigest` example is an instance of the Observer pattern.

### Lesson Learned: Start by Considering Existing Solutions

- Most problems have been **solved already** and described in a **well-documented** way.
- Knowing existing solutions and patterns in your field can save you a lot of time and effort.

---

## 4. Graphics: SVG vs Bitmap

- **Scalable Vector Graphics (SVG)** are **vector-based** graphics defined by **mathematical equations for shapes**. They **scale to any size** and often have **smaller file sizes**.
- **Bitmaps** (e.g., **JPEG, PNG, BMP, GIF**, ...) are **pixel-based** graphics composed of a **fixed grid of colored squares**. Due to their general-purpose representation, they are supported by more editing software.
- Rule of thumb: **Use SVGs whenever you can.**

---

## 5. Color Representations

- **RGB (Red, Green, Blue)** — the most common format to represent digital colors. The resulting color is a **mix of red, green, and blue light**.
  - **Hex-Color** representation uses two hexadecimal digits for each color (`#RRGGBB`, e.g., `#2774AE`, `#FFD100`).
- **HSL / BSH (Hue, Saturation, Luminance/Brightness)** — lets you represent colors based on their **color tone (hue)**, **color intensity (saturation)**, and **brightness (luminance)**.
- **CMYK (Cyan, Magenta, Yellow, Keycolor (Black))** — represents colors based on the **subtractive color model used in printing** and is **less common for screen-based rendering**.

---

## 6. Important HTML Tags

### Content Tags

- **Headings** (`<h1>` to `<h6>`), e.g., `<h1>Top Heading</h1>`.
- **Paragraphs** (`<p>`), e.g., `<p>Example</p>`.
- **Lists** — `<ul>` unordered list and `<ol>` ordered list.

```html
<ol>
  <li>first item</li>
  <li>second item</li>
</ol>
```

### Text Formatting Tags

- **Bold** (`<strong>`, previously `<b>` or `<B>`) — e.g., `<strong>Important Text</strong>`.
- **Italics** (`<em>`, previously `<i>` or `<I>`) — e.g., `<em>emphasis</em>`.
- **Underline** (`<ins>`, previously `<U>`) — e.g., `<ins>inserted</ins>`.
- **Strikethrough** (`<del>`) — e.g., `<del>deleted</del>`.

### Interactive / Media Tags

- **Button** (`<button>`) — e.g., `<button>Click me</button>`.
- **Links** (`<a>`) — e.g., `<a href="ucl.edu">anchor</a>`.
- **Images** (`<img>`) — e.g., `<img src="https://u.edu/l">`.
- **Inputs** (`<input>` with various types) — e.g., `<input type="checkbox">check me</input>` (also `text`, `file`, `email`, `radio`, ...).

Reference: https://www.w3schools.com/tags/default.asp

---

## 7. HTML DOM (Document Object Model)

- The HTML DOM is a **programming interface for HTML documents**.
- It represents the structure of an HTML page as a **tree of objects**, where each object corresponds to a part of the document, such as an **element, attribute, or text**.

Given:

```html
<html>
  <head>
    <title>My Title</title>
  </head>
  <body>
    <h1>Hello</h1>
    <a href="[...]">Link</a>
  </body>
</html>
```

DOM tree:

- Document -> Root Element `<html>`
  - Element `<head>` -> Element `<title>` -> Text `"Title"`
  - Element `<body>`
    - Element `<h1>` -> Text `"Hello"`
    - Element `<a>` -> Attribute `"href"` -> Value `"[...]"`; Text `"Link"`

---

## 8. CSS (Cascading Style Sheets)

- **Declarative-style** computer programming language used to **define the visual presentation and layout** of web pages.
- Works in conjunction with markup languages like **HTML** (which provides the **structure and content** of a webpage).
- HTML = Content & Structure. CSS = Visual presentation & Layout. This is an instance of **Separation of Concerns**.

### Example: CSS defines style classes for HTML `div`s (divisions, used as containers)

`example.css`:

```css
h1 { /* Heading 1 will be navy colored and underlined */
    color: navy;
    text-decoration: underline;
}
.rounded-box { /* Will become a class for divs */
    width: 200px;
    background-color: #FFD100;
    border: 5px solid #2774AE;
    border-radius: 10px; /* corners */
    text-align: center;
    font-family: sans-serif;
}
```

`example.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>CSS Example</title>
  <link rel="stylesheet" href="example.css">
</head>
<body>
  <h1>CSS Example</h1>
  <div class="rounded-box">
    <p>This will be styled with CSS.</p>
  </div>
</body>
</html>
```

Reference: https://www.w3schools.com/css/

---

## 9. React.js — Overview

- **React** (a.k.a. React, ReactJS) is a **free and open-source front-end JavaScript library** for building user interfaces.
- Maintained by **Meta** (formerly Facebook).
- React has a **declarative nature**:
  - You describe **what** you want the UI to look like for a given state.
  - React handles the steps to achieve that UI and **keeps it updated efficiently when the state changes**.
- In contrast, **imperative** programming focuses on describing **how** to achieve a result by manually manipulating the UI elements.

### Welcome — bridging from Node.js to the frontend

- With Node.js experience you already know how to build the **brain** of an application — the server that crunches data, talks to a database, and serves APIs.
- An Express server only speaks in raw data (like JSON). **UI (User Interface) development** is about building the **face** of your application — how users interact with the data your Node.js server provides.

### The Core Paradigm Shift: Declarative vs. Imperative

In C++ or Python, you write **imperative** code — step-by-step instructions:

1. Find the button in the window.
2. Listen for a click.
3. When clicked, find the text box.
4. Change the text to "Clicked!"

React uses a **declarative** approach. Instead of writing steps to change the screen, you **declare what the screen should look like at any given moment, based on your data**.

Think of it like an Express route. In Express, you take a Request, process it, and return a Response. In React, you take Data, process it, and return UI:

\[UI = f(Data)\]

When the data changes, React automatically re-runs your function and efficiently updates the screen for you. You never manually touch the screen; you only update the data.

---

## 10. Components — The Building Blocks

In Python or C++ you don't write your entire program in one massive `main()` function. You break it down into smaller, reusable functions or classes. **React does the exact same thing for user interfaces using Components.** A component is just a JavaScript function that returns a piece of the UI.

- Components are **self-contained, reusable pieces of UI** that manage their own rendering logic.
- Modularity promotes code **reusability**, simplifies development, and improves **maintainability**.
- React components are **defined as functions with a single return element**.
- **Inputs to the functions are called props**.

First example:

```jsx
// A simple React Component
function UserProfile() {
  const username = "CPlusPlusFan99";
  const role = "Admin";

  return (
    <div className="profile-card">
      <h1>{username}</h1>
      <p>System Role: {role}</p>
    </div>
  );
}
```

Lecture example with props and export:

```jsx
function WelcomeMessage(props) {
  return <h1>Hello, {props.name}!</h1>;
}
export default WelcomeMessage;
```

- `props` — Component parameters.
- `props.name` — Component parameter reference.
- `export default WelcomeMessage;` — Makes the component visible to outside modules.
- Referenced via: `<WelcomeMessage name="CS35L"/>`.

---

## 11. JSX (JavaScript XML)

- **JSX is a syntax extension** that allows developers to write **HTML-like code within JavaScript**. Makes it easier to define the structure and content of UI components.
- Under the hood, a compiler (**Babel, SWC, or esbuild**) transforms those HTML-like tags into plain JavaScript function calls.

### Compilation forms

```jsx
// JSX (what you write):
<button className="btn-primary" disabled={false}>Save</button>

// Modern (React 17+) "automatic" JSX transform output:
import { jsx as _jsx } from 'react/jsx-runtime';
_jsx('button', { className: 'btn-primary', disabled: false, children: 'Save' });

// Older "classic" transform output (still produced by some toolchains):
React.createElement('button', { className: 'btn-primary', disabled: false }, 'Save');
```

Either form returns a lightweight JavaScript object — the **Virtual DOM node**. React then compares these object trees to determine the minimal set of real DOM changes needed.

### Embedding JavaScript

Just like f-strings in Python (`f"Hello {username}"`), JSX allows you to seamlessly inject JavaScript variables directly into your UI using **curly braces `{}`**.

Lecture example:

```jsx
const name = "CS 35L"
[...]
return <h1>Hello, {name}!</h1>;
// Compiles to: React.createElement('h1', null, 'Hello, ', name, '!');
// Renders:     Hello, CS 35L
```

### JSX Rules — Where HTML Instincts Break

JSX looks like HTML but is actually JavaScript. These rules catch most beginners:

| Rule                     | Wrong (HTML instinct)      | Correct (JSX)                                            |
|--------------------------|----------------------------|----------------------------------------------------------|
| CSS class                | `class="..."`              | `className="..."` (`class` is a JS keyword)              |
| Self-closing tags        | `<img src={u}>`            | `<img src={u} />`                                        |
| Inline style             | `style="color:red"`        | `style={{color: 'red'}}` (JS object, not CSS string)     |
| Multiple root elements   | `return <h1/><p/>`         | `return <><h1/><p/></>` (fragment wrapper)               |
| Component names          | `<card />`                 | `<Card />` (must be capitalized)                         |
| Event handlers           | `onclick`                  | `onClick` (camelCase)                                    |

---

## 12. State — Component Memory

A UI isn't very useful if it can't change. In a C++ class, you use member variables to keep track of an object's current status. In React, we use **State**.

**State is simply a component's memory.** When a component's state changes, React says, "Ah! The data changed. I need to re-run this function to see what the new UI should look like."

### LikeButton example

```jsx
import { useState } from 'react';

function LikeButton() {
  // 1. Define state: [currentValue, setterFunction] = useState(initialValue)
  const [likes, setLikes] = useState(0);

  // 2. Define an event handler
  function handleLike() {
    setLikes(likes + 1); // Tell React the data changed!
  }

  // 3. Return the UI
  return (
    <div className="like-container">
      <p>This post has {likes} likes.</p>
      <button onClick={handleLike}>
        Like this post
      </button>
    </div>
  );
}
```

### Breaking down `useState`

`useState` is a special React function (called a **Hook**). It returns an array with two things:

- `likes` — the current value (like a standard variable).
- `setLikes` — a setter function. **Crucial rule:** you cannot just do `likes++` like you would in C++. You **must use the setter function** (`setLikes`). Calling the setter is what alerts React to re-render the UI with the new data.

### Lecture Counter example (props.initialCount)

```jsx
import React, { useState } from 'react';

function Counter(props) {
  const [count, setCount] = useState(props.initialCount);

  const increment = () => {
    setCount(count + 1);
  };

  return (
    <>
      <p>Current Count: {count}</p>
      <button onClick={increment}>
        Increment
      </button>
    </>
  );
}
export default Counter;
```

Annotated:

- `count` — **State variable**.
- `setCount` — **State change function name**.
- `useState(props.initialCount)` — **Initial value of the state**.
- `{count}` — **Reference to the state variable**. Changes whenever the state variable is changed via the state change function.
- `onClick={increment}` — **Click event handler** (the name of the function to be called when the user clicks on the button).
- `export default Counter;` — Makes the component visible to outside modules.

### Destructuring props

```jsx
function Counter({initialCount}) {                // Unwrap props
  const [count, setCount] = useState(initialCount);
  ...
}
```

You can add more props:

```jsx
function Counter({initialCount, countName}) {
  const [count, setCount] = useState(initialCount);
  const increment = () => { setCount(count + 1); };
  return (
    <>
      <p>{countName}: {count}</p>
      <button onClick={increment}>Increment</button>
    </>
  );
}
export default Counter;
```

### Functional updates — the `prev` pattern

When new state depends on the old state, always pass a function to the setter instead of the current value. This avoids **stale closure** bugs, where a callback captures an outdated snapshot of the variable:

```jsx
// Risky — `likes` captured at render time; concurrent updates can drop clicks
setLikes(likes + 1);

// Safe — React passes the guaranteed latest value as `prev`
setLikes(prev => prev + 1);
```

A **stale closure** occurs when an event handler closes over a value that was current when the component rendered but has since been superseded by newer state. The `prev =>` pattern sidesteps this because React resolves the function at the moment the update is applied, not at the moment the handler was created.

### State batching

React 18 and later use **automatic batching**: multiple `setState` calls that happen in the same synchronous tick — whether inside event handlers, promises, `setTimeout` callbacks, or `async` functions — are merged into a **single re-render**. This is an optimisation; you will not see intermediate states. If you call `setA(1); setB(2);` in one click handler, the component re-renders once with both changes applied.

---

## 13. `useEffect` — Connecting Frontend to Backend

How does React connect to what you already know? Your Express server might have a route like this:

```javascript
// Express Backend
app.get('/api/users/1', (req, res) => {
  res.json({ name: "Alice", status: "Online" });
});
```

In React, you would write a component that fetches that data and displays it. We use another hook called `useEffect` to run code when the component first appears on the screen:

```jsx
import { useState, useEffect } from 'react';

function Dashboard() {
  const [userData, setUserData] = useState(null);

  // This runs after the component mounts. (In development with React's
  // StrictMode, you'll see it run twice — that's intentional and goes away
  // in production. Real fetch effects should also return a cleanup function
  // — e.g., aborting via AbortController — but it's omitted here for brevity.)
  useEffect(() => {
    // Fetch data from your Express server!
    fetch('http://localhost:3000/api/users/1')
      .then(response => response.json())
      .then(data => setUserData(data));
  }, []);

  // If the data hasn't arrived from the server yet, show a loading message
  if (userData === null) {
    return <p>Loading data from Express...</p>;
  }

  // Once the data arrives, render the actual UI
  return (
    <div>
      <h1>Welcome back, {userData.name}!</h1>
      <p>Status: {userData.status}</p>
    </div>
  );
}
```

Key behaviour notes:

- Runs after the component mounts.
- In development with **StrictMode**, you will see it run **twice** — intentional, goes away in production.
- Real fetch effects should also return a **cleanup function** — e.g., aborting via `AbortController`.

---

## 14. Props — Passing Data Into Components

Components without data are static. **Props** let you pass data into a component, exactly like function arguments:

```jsx
// C++:    void printCard(string name, double price) { ... }
// Python: def render_card(name, price): ...

// React — defining the component:
function ProductCard({ name, price }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>${price.toFixed(2)}</p>
    </div>
  );
}

// React — using the component (like calling a function with named args):
<ProductCard name="Laptop" price={999.99} />
```

### Key props rules

- **One-way flow** — props flow from parent to child, never the reverse.
- **Read-only** — props are immutable inside the component (like `const` parameters).
- **Any JS value** — strings, numbers, booleans, objects, arrays, functions can all be props.

String props can use quotes (`title="Hello"`); all other types need braces (`price={99.99}`, `active={true}`).

---

## 15. Lists, Keys, Conditional Rendering, and the Map Function

In C++ you render lists with `for` loops. In React, you use `.map()` to transform data arrays into JSX.

### Lecture map example

```jsx
const names = ["Royce Hall", "Powell Library", "Kerckhoff Hall"]
[...]
return (
  <ul>
    {names.map((name, index) => (
      <li key={index}>{name}</li>
    ))}
  </ul>
);
// Renders:
//   Royce Hall
//   Powell Library
//   Kerckhoff Hall
```

### Reference example

```jsx
const tasks = [{id: 1, text: 'Learn React', done: true}, ...];

// .map() transforms data into JSX; key identifies each item for React's diffing
const taskList = tasks.map(task =>
  <li key={task.id}>{task.done ? 'done' : 'todo'} {task.text}</li>
);
return <ul>{taskList}</ul>;
```

### Keys

**Keys** tell React which items are stable across re-renders. Without stable keys, React compares by position — causing bugs when items are reordered or deleted. **Never use array index as a key for dynamic lists; use a stable ID from your data.**

### Other array methods you will see constantly

Beyond `.map()`, two other array methods appear constantly in React:

```jsx
// .filter() — keep only items that match a condition
const doneTasks = tasks.filter(task => task.done);

// .reduce() — fold a list into a single value (e.g., a cart total)
const total = cartItems.reduce((sum, item) => sum + item.price, 0);
```

These are plain JavaScript — React adds nothing special — but they are the idiomatic way to derive display data from state without storing redundant copies.

### Conditional rendering

Conditional rendering uses plain JavaScript inside JSX:

```jsx
// Short-circuit: only renders when condition is true
{unreadCount > 0 && <Badge count={unreadCount} />}

// Ternary: choose between two alternatives
{isLoggedIn ? <Dashboard /> : <LoginForm />}
```

**Watch out:** `{count && <Badge />}` renders the number `0` when `count` is `0`, because `0` is a valid React node. Use `{count > 0 && <Badge />}` instead.

---

## 16. Composition Over Inheritance

In C++ and Java, you reuse code via inheritance (`class Dog : Animal`). React uses **composition** — building complex UIs by combining small, generic components:

```jsx
// Generic container — accepts anything as children
function Card({ children, className }) {
  return <div className={'card ' + (className || '')}>{children}</div>;
}

// Specific use — compose with the children prop
function ProfileCard({ user }) {
  return (
    <Card className="profile">
      <Avatar src={user.avatar} />
      <h3>{user.name}</h3>
    </Card>
  );
}
```

The **`children` prop** lets any content be nested inside a component, making it a composable container — analogous to C++ templates or Python's `*args`.

### Prop drilling

When a value must pass through several intermediate components that don't use it themselves — only to reach a deeply nested child — the pattern is called **prop drilling**. It works, but it couples every layer in between to data it doesn't care about, making refactoring painful. For small trees, prop drilling is fine. When it becomes unwieldy, the typical solutions are **lifting state to a closer ancestor** or **using a context/state-management library**.

---

## 17. Thinking in React

React's official methodology for building a new UI:

1. **Break the UI into a component hierarchy** — each component does one job (single-responsibility).
2. **Build a static version first** — props only, no state.
3. **Identify the minimal state** — don't duplicate data that can be derived.
4. **Determine where state lives** — the lowest common ancestor that needs it.
5. **Add inverse data flow** — children call callback functions passed as props.

---

## 18. Lifting State Up

When two sibling components need the same data, move the state to their **lowest common ancestor** and pass it down as props:

```jsx
function Parent() {
  const [text, setText] = useState('');
  return (
    <>
      <SearchBar value={text} onChange={setText} />
      <ResultsList filter={text} />
    </>
  );
}
```

`SearchBar` calls `onChange(e.target.value)` to notify the parent. The parent updates state, which triggers a re-render of both components. This is **"inverse data flow"** — data flows down via props, notifications flow up via callbacks.

---

## 19. The Virtual DOM

- The **Virtual DOM** is a **lightweight representation** (a lightweight JavaScript object tree) of the actual DOM.
- When data changes, React **first updates the Virtual DOM**, then **efficiently calculates the differences** with the real DOM and **applies only the necessary changes** to the real DOM.
- This **minimises direct DOM manipulation** and **improves performance**.

Process:

1. First updates the Virtual DOM (fast — it is just a JavaScript object tree).
2. Calculates the differences between the new and previous Virtual DOM trees.
3. Applies only the minimal patches to the real DOM.

This algorithm is called **Reconciliation**.

---

## 20. Single Page Application (SPA) vs Server-Side Rendering (SSR)

### Single Page Application (SPA) — most common in React

- SPAs are structured as a **single HTML page** that has **no preloaded content**.
- **Content is loaded dynamically via JavaScript** for the entire application and housed within a single HTML page.
- The JavaScript code houses all the data relating to the application logic, UI, and communication with the server.
- Flow: Client `GET ('/')` -> Server returns Single Page App (contains all pages) -> in-page navigation to `/current_news`, `/about`, etc. happens client-side.

### Server-Side Rendering (SSR) — used by frameworks like Next.js (which use React)

- Clients receive a **fully rendered page** (rendered on the server).
- Requires more **server compute**.
- **Faster initial load time** (since only the current page is sent).
- **Longer response time for user interaction**.
- **Better search engine optimisation**.
- Flow: Client `GET ('/')` -> Server returns Main page; Client `GET ('/current_news')` -> Server returns Current news page; each navigation is a fresh server request.

More details: https://hygraph.com/blog/difference-spa-ssg-ssr

---

## 21. Top 10 React Best Practices

These are the most important habits to build early. Every one of them prevents real bugs that trip up beginners — and professionals.

**1. Use `useState` for component memory — never bare variables.**
A `let` variable inside a component resets to its initial value on every render. Only `useState` persists data and triggers re-renders when it changes.

**2. Keep state minimal — derive what you can.**
If a value can be computed from existing state or props, compute it during render instead of storing a second copy. Two copies can drift out of sync.

```jsx
// Good — filter is the only state; visibleTasks is derived
const [filter, setFilter] = useState('all');
const visibleTasks = tasks.filter(t => filter === 'all' || t.status === filter);
```

**3. Never mutate state — always create new arrays and objects.**
React detects changes by reference. `array.push()` returns the same reference, so React skips the re-render. Spread into a new array instead.

```jsx
// Bad — mutates in place, React sees no change
items.push(newItem);
setItems(items);

// Good — new array, React re-renders
setItems([...items, newItem]);
```

**4. Use stable, unique keys for lists — never the array index.**
Keys tell React which element is which across re-renders. If items are reordered or deleted, index-based keys cause state to attach to the wrong element (e.g., checked checkboxes shifting). Use a unique ID from your data.

**5. Destructure props in the function signature.**
It makes the component's API visible at a glance and avoids repetitive `props.` prefixes throughout the body.

```jsx
// Good
function ProductCard({ name, price, onSale }) { ... }

// Avoid
function ProductCard(props) { return <h3>{props.name}</h3>; }
```

**6. Lift state to the lowest common ancestor.**
When two sibling components need the same data, move the state up to their nearest shared parent and pass it down as props. The child notifies the parent through a callback prop — never by reaching into siblings directly.

**7. One component, one job.**
If a component handles product display and cart management and filtering, it is doing too much. Split it into focused pieces (`ProductCard`, `CartSummary`, `FilterBar`). Small components are easier to read, test, and reuse.

**8. Name event handlers `handle*`, callback props `on*`.**
Inside a component, the function that handles a click is `handleClick`. When you pass it to a child as a prop, call the prop `onClick`. This convention makes it immediately clear which end owns the logic and which end fires the event.

```jsx
function App() {
  const handleDelete = (id) => { /* ... */ };
  return <TodoItem onDelete={handleDelete} />;
}
```

**9. Guard `&&` rendering against falsy numbers.**
`{count && <Badge />}` renders the literal `0` when `count` is `0`, because `0` is a valid React node. Use an explicit boolean: `{count > 0 && <Badge />}`.

**10. Follow the two Rules of Hooks.**
React tracks hooks by their call order. Two rules are non-negotiable:

- **Only call hooks at the top level** — never inside `if`, loops, or nested functions. If a `useState` call is skipped on one render, every hook after it shifts position, causing crashes or silent data corruption.
- **Only call hooks inside React function components** (or custom hooks) — never in plain JavaScript utility functions, class methods, or event listeners outside of a component.

---

## 22. Glossary

| Term | Definition |
|---|---|
| **Component** | A JavaScript function that returns JSX. The building block of React UIs. |
| **JSX** | A syntax extension that lets you write HTML-like markup inside JavaScript. A compiler (Babel, SWC, or esbuild) transforms it into JavaScript function calls — historically `React.createElement()`, and since React 17 the automatic transform calls `jsx()` from `react/jsx-runtime`. |
| **Props** | Read-only data passed from a parent component to a child, like function arguments. |
| **State** | Data managed inside a component via `useState`. Changing state triggers a re-render. |
| **Hook** | A special function (prefixed with `use`) that lets components use React features. Must be called at the top level. |
| **Re-render** | When React re-calls your component function because state or props changed, producing a new JSX tree. |
| **Virtual DOM** | A lightweight JavaScript object tree that React builds from your JSX. React diffs the old and new trees and patches only the changed real DOM nodes. |
| **Reconciliation** | The algorithm React uses to compare the old and new Virtual DOM trees and determine the minimal set of DOM updates. |
| **Key** | A special prop on list items that helps React identify which items changed, were added, or were removed during reconciliation. |
| **Fragment** | A wrapper (`<>...</>`) that groups multiple JSX elements without adding an extra DOM node. |
| **Derived state** | A value computed from existing state or props during render, rather than stored in its own `useState`. |
| **Lifting state up** | Moving state to the lowest common ancestor of the components that need it, then passing it down as props. |
| **Stale closure** | A bug where an event handler or callback captures an outdated state value from a previous render. Fixed by using the functional `setState(prev => ...)` pattern. |
| **Functional update** | Passing a function to a state setter (`setState(prev => prev + 1)`) so React provides the latest state value at update time, avoiding stale closure bugs. |
| **State batching** | React 18's optimisation of merging multiple `setState` calls that happen in the same synchronous tick (event handlers, promises, timeouts, async callbacks) into a single re-render. |
| **Prop drilling** | Passing a prop through several intermediate components that don't use it, just to reach a deeply nested child that does. |

---

## 23. Summary

- **Components:** UI is broken down into reusable JavaScript functions.
- **JSX:** We write HTML-like syntax inside JS to describe UI; a compiler turns it into `jsx()` (modern) or `React.createElement` (classic) calls.
- **Props:** Data flows one-way from parent to child. Props are read-only.
- **State:** We use `useState` to give components memory. Updating state triggers re-renders.
- **Lists & Keys:** Use `.map()` with stable `key` props for dynamic lists.
- **Conditional Rendering:** Use `&&` and ternary operators inside JSX.
- **Composition:** Build complex UIs by combining small components via the `children` prop.
- **Integration:** React runs in the user's browser, acting as the client that makes HTTP requests to your Node.js/Express server.
- **Virtual DOM** keeps re-renders cheap by diffing in JavaScript before touching the real DOM.
- **SPAs vs SSR** — React's natural fit is SPAs; frameworks like Next.js add SSR for better SEO and initial load.
- **Separation of Concerns** drives the whole stack: HTML (structure) / CSS (presentation) / JS (behaviour); presentation layer / application layer; Observer pattern lets domain code stay UI-agnostic.

---

## 24. Practice Question Recap (from reference)

### React Syntax — What Does This Code Do? (Basic)

```jsx
{isLoggedIn ? <Dashboard /> : <LoginForm />}
```

This is a ternary inside JSX: if `isLoggedIn` is truthy, render `<Dashboard />`; otherwise render `<LoginForm />`.

### React Syntax — Write the Code (Advanced)

Pass a callback function from a parent to a child component so the child can update the parent's state — see the **Lifting State Up** section (`<SearchBar value={text} onChange={setText} />`).

### React Concepts Quiz (Advanced)

A student stores the full filtered list in state alongside the unfiltered list:

```jsx
const [allTasks, setAllTasks] = useState(tasks);
const [filteredTasks, setFilteredTasks] = useState(tasks);
```

Correct answer: **C — Storing derived data violates minimal state — the filter string is the only real state.** Two copies can drift out of sync; the filtered list should be derived during render from the filter plus the source list.

Questions 1-7 cover tutorial material. Questions 8-10 test advanced concepts from the reference page. Questions 11-15 cover event handlers, `useEffect`, and state immutability.


---

# Appendix: Lecture 6 Slides (raw extracted text)

The following is the full extracted text of Lecture 6. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 5 � React, UI
Development & Design
Pattern Introduction
Assistant Teaching Professor
Computer Science Department
Motivating Case Study: Digital Monopoly Game
There are dozens of implementations of the game "Monopoly"
with vastly different User Interfaces
Motivating Case Study: Digital Monopoly Game
There are dozens of implementations of the game "Monopoly"
with vastly different User Interfaces. Even with just plain text UI!
Next Turn (Mohamed)                          Dice Rolled (Mohamed)
It is Mohamed`s turn.                    The first die shows 6. The second die shows 2.
       Roll Dice                                                    Okay
                       Player Decision Requested (Mohamed)
                       You landed on the unpurchased property "Royce Hall".
                           Do you want to buy "Royce Hall" for Bruin$800?
                                  Your current inventory is Bruin$4250.
                       Yes (-Bruin$800)  No
What Should They All Have In Common?
                     All of them should implement
                       the exact same game logic!
               So at least in theory they could all run
           the exact same code just with a different UI
         (ideally even with multiple UIs simultaneously)
Next Turn (Mohamed)     Dice Rolled (Mohamed)     Player Decision Requested (Mohamed)
 It is Mohamed`s turn
                         The first dice shows 6.  You landed on the unpurchased property "Royce Hall".
        Roll Dice      The second dice shows 2.       Do you want to buy "Royce Hall" for Bruin$800?
                                                             Your current inventory is Bruin$4250.
                                    Okay
                                                  Yes (-Bruin$800)  No
Separation of Concerns                                   Front-
                                                          End
Systems should be divided into distinct
sections, or concerns, where each section
addresses a separate, specific goal,
purpose, or responsibility.
The goal is to make the system easier to
develop, maintain, and evolve.
Applies to all                                           Back-
  software                                                End
development.
  Not just UI
Read more here: https://www.geeksforgeeks.org/software-
engineering/separation-of-concerns-soc/
                CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  5
                Tobias D�rschmid
Separate the UI Implementation
from the Domain Logic of your Application
              Presentation Layer                                Doesn't have a
                                                                clue that there
     Displays information to the user and collects input
(e.g., positions of players on the board, style of the boards,   even is a UI.
                                                                  I mean who
                               buttons, ...)                     really needs a
                                                                  UI? Can't we
               Application Layer
                                                                    just have
         Implements the domain logic and behavior                command line
    (e.g., effects of community chest cards, player turns,       interfaces with
                                                                  the shell???
                   interactions between players)
Separate the UI Implementation
from the Domain Logic of your Application
              Presentation Layer                           Doesn't have a
                                                           clue that there
               Application Layer
                                                            even is a UI.
- Allows presentation later to register a                    I mean who
    callback for state changes (e.g., onBalanceChanged)     really needs a
                                                             UI? Can't we
- Allows presentation layer to retrieve information using
    getter functions (e.g., getCurrentBalance())               just have
                                                            command line
- Allows presentation layer to forward user events          interfaces with
    (e.g., buyProperty(name, user))                          the shell???
The Observer      Pattern descriptions have many different
Design Pattern     formats. At minimum it should describe
                       problem, context, and solution.
  Context
Scenarios requiring distributed event handling systems or highly
decoupled architectures like GUI, and signal processing
  Problem
How can a one-to-many dependency between objects be maintained
efficiently without making the objects tightly coupled?
  Solution
The pattern introduces two main roles: the Subject (the object
sending updates after it has changed) and the Observer (the object
listening to the updates of Subjects).
The Observer Design Pattern                             Read more here:
                                                        https://tobiasduerschmid.github.io/SE
                                                        Book/designpatterns/observer.html
                       *                                Subject                     Observer
Subject                    AbstractObserver                      register(self)
                          update(Event)                             Adds observer
register(Observer)
unregister(Observer)      doSomething()                          to list
notify(Event)
Data getData()            Concrete Observer                      notify(someEvent)
doSomething()             update(Event)
                                                                     Iterates over
                                                                      observer list
                                                                           update()
  e.g., your Domain       e.g., the presentation-later           getData()
 Object with a lot of      UI element that displays                         data
useful business logic       the data of the subject
                             whenever it changes
Design Patterns
                  found in many      It is a good solution,
                    instances        but not always the
                                            best one
   "A pattern is a common, acceptable solution to a
reoccurring problem that arises in a specific context"
  The problem is generic enough      Patterns always refer to a
so that the it generalizes beyond a  specific situation, goal, or
                                     trade-off. Patterns are not
          few concrete cases
                                           universally good
Design Patterns Add Roles To Classes / Objects
This is an Example of the Observer Design Pattern
_channel           NewsChannel                             for subscriber in self._subscribers:
                                                            subscriber.update()
          -_subscribers: list[Subscriber]
          -_latest_post: str                                                      _channel
          +follow(subscriber: Subscriber)
          +unfollow(subscriber: Subscriber)
          +publish_post(text: str)
          +get_latest_post(): str
          -_notify_subscribers()
                          0..* 1_subscribers
                            �ABC�
                      Subscriber
                       +update()
                MobileApp              EmailDigest
          -_channel: NewsChannel  -_channel: NewsChannel
          +update()               +update()
          post = self._channel.get_latest_post()
          print(f"[MobileApp] Push notification: {post}")
          CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React     11
          Tobias D�rschmid
Observer Pattern � Subscriber Interface
  # ==================================
  # OBSERVER INTERFACE
  # ==================================
  class Subscriber(ABC):
         """The Observer interface."""
         @abstractmethod
         def update(self):
                pass
Observer Pattern � Subject (NewsChannel)
# SUBJECT
class NewsChannel:
       """The Subject that maintains a list of subscribers."""
       def __init__(self):
              self._subscribers: list[Subscriber] = []
              self._latest_post: str = ""
def follow(self, subscriber: Subscriber):
       if subscriber not in self._subscribers:
              self._subscribers.append(subscriber)
def unfollow(self, subscriber: Subscriber):
       self._subscribers.remove(subscriber)
Observer Pattern � Subject (NewsChannel)
# SUBJECT
class NewsChannel:
[...]
def publish_post(self, text: str):
       self._latest_post = text
       self._notify_subscribers()
def get_latest_post(self) -> str:
       return self._latest_post
def _notify_subscribers(self):
       for subscriber in self._subscribers:
              subscriber.update()
Observer Pattern � Concrete Observers
  # CONCRETE OBSERVERS
  class MobileApp(Subscriber):
         """A concrete observer that pulls state from the channel."""
         def __init__(self, channel: NewsChannel):
                self._channel = channel
         def update(self):
                post = self._channel.get_latest_post()
                print(f"[MobileApp] Push notification: {post}")
  class EmailDigest(Subscriber):
         """Another concrete observer with different behavior."""
         def __init__(self, channel: NewsChannel):
                self._channel = channel
         def update(self):
                post = self._channel.get_latest_post()
                print(f"[EmailDigest] New email queued: {post}")
Observer Pattern � Client Code
  # CLIENT CODE
  channel = NewsChannel()
  app = MobileApp(channel)
  email = EmailDigest(channel)
  channel.follow(app)
  channel.follow(email)
  channel.publish_post("New video uploaded!")
  # [MobileApp] Push notification: New video uploaded!
  # [EmailDigest] New email queued: New video uploaded!
  channel.unfollow(email)
  channel.publish_post("Live stream starting!")
  # [MobileApp] Push notification: Live stream starting!
Lesson Learned:
Start By Considering Existing Solutions
� Most problems have been solved already and described in a
 well-documented way
� Knowing existing solutions and patterns in your field can
 save you a lot of time and effort
Use SVGs Whenever You Can
Scalable Vector Graphics (SVG) are     Bitmaps (e.g., JPEG, PNG, BMP, GIF,
vector-based graphics defined by       ...) are pixel-based graphics composed of
mathematical equations for shapes.     a fixed grid of colored squares. Due to
They scale to any size and often have  their general-purpose representation, they
smaller file sizes.                    are supported by more editing software.
There are many Different Color Representations
� RGB (Red, Green, Blue) is the most common format to represent digital colors.
  The resulting color is a mix of red, green, and blue light.
   � The Hex-Color representation uses two hexadecimal digits
     for each color (#RRGGBB e.g., #2774AE, #FFD100)
� HSL/BSH (Hue, Saturation, Luminance/Brightness)
  lets you represent colors based on their
  color tone (hue), and color intensity (saturation)
  and brightness (luminance)
� CMYK (Cyan, Magenta, Yellow, Keycolor (Black))
  represents colors based on the subtractive
  color model used in printing and is less common for screen-based rendering
                    See https://www.w3schools.com/tags/default.asp
Important HTML Tag
  Content Tags                           Top Heading
� Headings (<h1> to <h6>)                Example
    e.g., <h1>Top Heading</h1>
                                         1. first item
� Paragraphs (<p>) e.g., <p>Example</p>  2. second item
� Lists (<ul> unordered list
                                         � First item
    and <ol> ordered lists)              � Second item
    e.g., <ol>
                <li>first item</li>
                <li>second item</li>
         </ol>
                    See https://www.w3schools.com/tags/default.asp
Important HTML Tag
  Text Formatting Tags                     Important Text
                                           emphasis
� Bold (<strong> previously: <b> or <B>)   inserted
    e.g., <strong>Important Text</strong>  deleted
� Italics (<em> previously: <i> or <I>)
    e.g., <em>emphasis</em>
� Underline (<ins> previously: <U>)
    e.g., <ins>inserted</ins>
� Strikethrough (<del>)
    e.g., <del>deleted</del>
                    See https://www.w3schools.com/tags/default.asp
Important HTML Tag
  Text Formatting Tags                              Click me
                                                   anchor
� Button (<button>)
    e.g., <button>Click me</button>                    Check me
� Links (<a>)
    e.g., <a href="ucl.edu">anchor</a>
� Images(<img>)
    e.g., <img src="https://u.edu/l">
� Inputs (<input> with various types)
    e.g., <input type="checkbox">check
    me</input> (or text, file, email, radio, ...)
HTML DOM (Document Object Model) is a
programming interface for HTML documents.
It represents the structure of an HTML page as a tree of Document
objects, where each object corresponds to a part of the Root Element
document, such as an element, attribute, or text.
                                                         <html>
<html>                       Element                                  Element
<head>                       <head>                                   <body>
   <title>My Title</title>   Element Element                          Element
</head>                      <title> <h1>                               <a>
<body>
                                                   Text  Text         Attribute       Text
   <h1>Hello</h1>                                                     "href"        "Link"
   <a href="[...]">Link</a>  "Title" "Hello"
</body>                                                                Value
</html>                                                                "[...]"
CSS (Cascading Style Sheets)
� Declarative-style computer programming language used to define the visual
  presentation and layout of web pages.
� Works in conjunction with markup languages like HTML (which provide the
  structure and content of a webpage)
               CSS                      HTML
Visual presentation & Layout    Content & Structure
More details here:
https://www.w3schools.com/css/
CSS can define style classes for HTML divs
(divisions, used as containers)
  example.html                       example.css
<!DOCTYPE html>                    h1 { /* Heading 1 will be navy colored
<html>                             and underlined */
<head>
                                          color: navy;
   <title>CSS Example</title>             text-decoration: underline;
   <link rel="stylesheet"          }
                                   .rounded-box {/* Will become a class
              href="example.css">
</head>                                                           for divs*/
<body>                                    width: 200px;
                                          background-color: #FFD100;
   <h1>CSS Example</h1>                   border: 5px solid #2774AE;
   <div class="rounded-box">              border-radius: 10px; /* corners */
                                          text-align: center;
       <p>This will be styled             font-family: sans-serif;
            with CSS.</p>          }
   </div>
</body>
</html>
CSS can define style classes for HTML divs
(divisions, used as containers)
  example.html
<!DOCTYPE html>
<html>
<head>
   <title>CSS Example</title>
   <link rel="stylesheet"
              href="example.css">
</head>
<body>
   <h1>CSS Example</h1>
   <div class="rounded-box">
       <p>This will be styled
            with CSS.</p>
   </div>
</body>
</html>
React.js (aka. React, ReactJS)
� React is a free and open-source front-end JavaScript library for building user
  interfaces
� maintained by Meta (formerly Facebook)
� React has a declarative nature
   � you describe what you want the UI to look like for a given state
   � React handles the steps to achieve that UI and keep it updated efficiently
     when the state changes.
   � In contrast, imperative programming focuses on describing how to achieve a
     result by manually manipulating the UI elements
React uses JSX (JavaScript XML)
� JSX is a syntax extension that allows developers to write HTML-like code
within JavaScript. This makes it easier to define the structure and content of UI
components.  React.createElement('h1',null,'Hello, ',name,'!');
const name = "CS 35L"
[...]
JavaScript constant (or variable)
defined somewhere
return <h1>Hello, {name}!</h1>;    Hello, CS 35L
Use JavaScript within HTML-like
       tags via curly braces.
CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  35
Map function
 const names = ["Royce Hall", "Powell Library",
 "Kerckhoff Hall"]
 [...]
return (<ul>                        � Royce Hall
   {names.map((name, index) => (    � Powell Library
       <li key={index}>{name}</li>  � Kerckhoff Hall
   ))}
</ul>);
CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  37
React Applications use Components
� Components are self-contained, reusable pieces of UI   Can be referenced via:
  that manage their own rendering logic.                 <WelcomeMessage
                                                         name="CS35L"/>
� This modularity promotes code reusability, simplifies
  development, and improves maintainability.
� React Component are Defined as Functions with a Single Return Element
� Inputs to the functions are called proCposmponent Parameters
function WelcomeMessage(props) {                         Component Parameter
                                                         Reference
   return <h1>Hello, {props.name}!</h1>;
}                    Makes the component visible to outside modules
export default WelcomeMessage;
   CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  38
   Tobias D�rschmid
CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  39
React Components manage their own State
import React, { useState } from 'react';       Initial value of the state
function Counter(props) {
const [count, setCount] = useState(props.initialCount);
State variable     State change function name
const increment = () => {         Reference to the state variable. Changes
   setCount(count + 1);           whenever the state variable is changed via
                                  the state change function
};
return (
   <>
<p>Current Count: {count}</p>
<button onClick={increment}>
        Increment
</button> Click event handler (the name of the function to
   </>             be called when the user clicks on the button)
);
} export default Counter; Makes the component visible to outside modules
                CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  40
                Tobias D�rschmid
React Components manage their own State
import React, { useStatUen}wrfarpompr'orpesact';  Initial value of the state
function Counter({initialCount}) {
const [count, setCount] = useState(initialCount);
State variable  State change function name
const increment = () => {         Reference to the state variable. Changes
   setCount(count + 1);           whenever the state variable is changed via
                                  the state change function
};
return (
   <>
    <p>Current onClick={increment}>
     Increment
    </button> Click event handler (the name of the function to
</>                 be called when the user clicks on the button)
);
Count: {count}</p>                Makes the component visible to outside modules
    <button} export default Counter;
                CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  41
                Tobias D�rschmid
React Components manage their own State
import React, { useState Y}ofurocman'raedadctp'r;ops  Initial value of the state
function Counter({initialCount, countName}) {
const [count, setCount] = useState(initialCount);
State variable     State change function name
const increment = () => {         Reference to the state variable. Changes
   setCount(count + 1);           whenever the state variable is changed via
                                  the state change function
};
return (
   <>
<p>{countName}: {count}</p>
<button onClick={increment}>
        Increment
</button> Click event handler (the name of the function to
   </>             be called when the user clicks on the button)
);
} export default Counter; Makes the component visible to outside modules
                CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  42
                Tobias D�rschmid
CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  43
React employs a Virtual DOM
� Virtual DOM is a lightweight representation of the actual DOM.
� When data changes, React first updates the Virtual DOM, then efficiently
  calculates the differences with the real DOM and applies only the necessary
  changes to the real DOM.
� Minimizes direct DOM manipulation and improves performance
CS 35L Software Construction: Lecture 6 � UI Development, Design Patterns, & React  45
Single Page Application (SPA)                                   Most common in React
� SPAs are structured as a single HTML page that has no preloaded content.
� Content is loaded dynamically via JavaScript for the entire application and
  housed within a single HTML page.
� The JavaScript code houses all the data relating to the application logic, UI, and
  communication with the server
Client                                                          Server
                                                     GET (`/')
        Single Page App (contains all pages)
More details here:             /current_news
https://hygraph.com/blog/diff  /about
erence-spa-ssg-ssr
Server-Side Rendering (SSR)                       Frameworks like Next.js
                                                  (which use React) do this
� Clients receive a fully rendered page (rendered on the server)
� Requires more server compute
� Faster initial load time (since only the current page is sent)
� Longer response time for user interaction
� Better search engine optimization
Client                                                      Server
         Main page                                GET (`/')
More details here:                                GET (`/current_news')
https://hygraph.com/blog/diff
erence-spa-ssg-ssr             Current news page
Please Fill out your Exit Tickets on Bruin Learn!
Please summarize three insights you learned about UI Development, Design Patterns,
or React today.
Please explain how using the Observer Design Pattern for UI elements supports the design
principle of Separation of Concerns
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images generated with Gemini, from Flaticon.com (Creators: Freepik)
and www.svgrepo.com (Creators: Solar Icons, Iconsax, Giovana, Esri)

```


---

# Appendix: Full root `react.md` reference (verbatim)

This is a reference page for React, designed to be kept open alongside the React Tutorial. Use it to look up syntax, concepts, and comparisons while you work through the hands-on exercises.

New to React? Start with the interactive tutorial first — it teaches these concepts through practice with immediate feedback. This page is a reference, not a teaching resource.

Welcome to the world of Frontend Development! Since you already have experience with Node.js, you actually have a massive head start.

You already know how to build the “brain” of an application—the server that crunches data, talks to a database, and serves APIs. But right now, your Express server only speaks in raw data (like JSON). UI (User Interface) development is about building the “face” of your application. It’s how your users will interact with the data your Node.js server provides.

To help you learn React, we are going to bridge what you already know (functions, state, and servers) to how React thinks about the screen.

The Core Paradigm Shift: Declarative vs. Imperative
In C++ or Python, you are used to writing imperative code. You write step-by-step instructions:

Find the button in the window.
Listen for a click.
When clicked, find the text box.
Change the text to “Clicked!”
React uses a declarative approach. Instead of writing steps to change the screen, you declare what the screen should look like at any given moment, based on your data.

Think of it like an Express route. In Express, you take a Request, process it, and return a Response. In React, you take Data, process it, and return UI.

\[UI = f(Data)\]
When the data changes, React automatically re-runs your function and efficiently updates the screen for you. You never manually touch the screen; you only update the data.

The Building Blocks: Components
In Python or C++, you don’t write your entire program in one massive main() function. You break it down into smaller, reusable functions or classes.

React does the exact same thing for user interfaces using Components. A component is just a JavaScript function that returns a piece of the UI.

Let’s look at your very first React component. Don’t worry if the syntax looks a little strange at first:

// A simple React Component
function UserProfile() {
  const username = "CPlusPlusFan99";
  const role = "Admin";

  return (
    <div className="profile-card">
      <h1>{username}</h1>
      <p>System Role: {role}</p>
    </div>
  );
}
What is that HTML doing inside JavaScript?!
You are looking at JSX (JavaScript XML). It is a special syntax extension for React. Under the hood, a compiler (Babel, SWC, or esbuild) transforms those HTML-like tags into plain JavaScript function calls:

// JSX (what you write):
<button className="btn-primary" disabled={false}>Save</button>

// Modern (React 17+) "automatic" JSX transform output:
import { jsx as _jsx } from 'react/jsx-runtime';
_jsx('button', { className: 'btn-primary', disabled: false, children: 'Save' });

// Older "classic" transform output (still produced by some toolchains):
React.createElement('button', { className: 'btn-primary', disabled: false }, 'Save');
Either form returns a lightweight JavaScript object — the Virtual DOM node. React then compares these object trees to determine the minimal set of real DOM changes needed.

Notice the {username} syntax? Just like f-strings in Python (f"Hello {username}"), JSX allows you to seamlessly inject JavaScript variables directly into your UI using curly braces {}.

Adding Memory: State
A UI isn’t very useful if it can’t change. In a C++ class, you use member variables to keep track of an object’s current status. In React, we use State.

State is simply a component’s memory. When a component’s state changes, React says, “Ah! The data changed. I need to re-run this function to see what the new UI should look like.”

Let’s build a component that tracks how many times a user clicked a “Like” button—something you might eventually connect to an Express backend.

import { useState } from 'react';

function LikeButton() {
  // 1. Define state: [currentValue, setterFunction] = useState(initialValue)
  const [likes, setLikes] = useState(0);

  // 2. Define an event handler
  function handleLike() {
    setLikes(likes + 1); // Tell React the data changed!
  }

  // 3. Return the UI
  return (
    <div className="like-container">
      <p>This post has {likes} likes.</p>
      <button onClick={handleLike}>
        👍 Like this post
      </button>
    </div>
  );
}
Breaking down useState:
useState is a special React function (called a “Hook”). It returns an array with two things:

likes: The current value (like a standard variable).
setLikes: A setter function. Crucial rule: You cannot just do likes++ like you would in C++. You must use the setter function (setLikes). Calling the setter is what alerts React to re-render the UI with the new data.
Functional updates — the prev pattern
When new state depends on the old state, always pass a function to the setter instead of the current value. This avoids stale closure bugs, where a callback captures an outdated snapshot of the variable:

// Risky — `likes` captured at render time; concurrent updates can drop clicks
setLikes(likes + 1);

// Safe — React passes the guaranteed latest value as `prev`
setLikes(prev => prev + 1);
A stale closure occurs when an event handler closes over a value that was current when the component rendered but has since been superseded by newer state. The prev => pattern sidesteps this because React resolves the function at the moment the update is applied, not at the moment the handler was created.

State batching
React 18 and later use automatic batching: multiple setState calls that happen in the same synchronous tick — whether inside event handlers, promises, setTimeout callbacks, or async functions — are merged into a single re-render. This is an optimisation; you will not see intermediate states. If you call setA(1); setB(2); in one click handler, the component re-renders once with both changes applied.

Putting it Together: Connecting Frontend to Backend
How does this connect to what you already know?

Right now, your Express server might have a route like this:

// Express Backend
app.get('/api/users/1', (req, res) => {
  res.json({ name: "Alice", status: "Online" });
});
In React, you would write a component that fetches that data and displays it. We use another hook called useEffect to run code when the component first appears on the screen:

import { useState, useEffect } from 'react';

function Dashboard() {
  const [userData, setUserData] = useState(null);

  // This runs after the component mounts. (In development with React's
  // StrictMode, you'll see it run twice — that's intentional and goes away
  // in production. Real fetch effects should also return a cleanup function
  // — e.g., aborting via AbortController — but it's omitted here for brevity.)
  useEffect(() => {
    // Fetch data from your Express server!
    fetch('http://localhost:3000/api/users/1')
      .then(response => response.json())
      .then(data => setUserData(data)); 
  }, []);

  // If the data hasn't arrived from the server yet, show a loading message
  if (userData === null) {
    return <p>Loading data from Express...</p>;
  }

  // Once the data arrives, render the actual UI
  return (
    <div>
      <h1>Welcome back, {userData.name}!</h1>
      <p>Status: {userData.status}</p>
    </div>
  );
}
Props: Passing Data Into Components
Components without data are static. Props let you pass data into a component, exactly like function arguments:

// C++:    void printCard(string name, double price) { ... }
// Python: def render_card(name, price): ...

// React — defining the component:
function ProductCard({ name, price }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>${price.toFixed(2)}</p>
    </div>
  );
}

// React — using the component (like calling a function with named args):
<ProductCard name="Laptop" price={999.99} />
Key props rules:

One-way flow — props flow from parent to child, never the reverse
Read-only — props are immutable inside the component (like const parameters)
Any JS value — strings, numbers, booleans, objects, arrays, functions can all be props
String props can use quotes (title="Hello"); all other types need braces (price={99.99}, active={true}).

JSX Rules — Where HTML Instincts Break
JSX looks like HTML but is actually JavaScript. These rules catch most beginners:

React table
Rule	Wrong (HTML instinct)	Correct (JSX)
CSS class	class="..."	className="..." (class is a JS keyword)
Self-closing tags	<img src={u}>	<img src={u} />
Inline style	style="color:red"	style={{color: 'red'}} (JS object, not CSS string)
Multiple root elements	return <h1/><p/>	return <><h1/><p/></> (fragment wrapper)
Component names	<card />	<Card /> (must be capitalized)
Event handlers	onclick	onClick (camelCase)
Lists, Keys, and Conditional Rendering
In C++ you render lists with for loops. In React, you use .map() to transform data arrays into JSX:

const tasks = [{id: 1, text: 'Learn React', done: true}, ...];

// .map() transforms data → JSX; key identifies each item for React's diffing
const taskList = tasks.map(task =>
  <li key={task.id}>{task.done ? '✓' : '✗'} {task.text}</li>
);
return <ul>{taskList}</ul>;
Keys tell React which items are stable across re-renders. Without stable keys, React compares by position — causing bugs when items are reordered or deleted. Never use array index as a key for dynamic lists; use a stable ID from your data.

Beyond .map(), two other array methods appear constantly in React:

// .filter() — keep only items that match a condition
const doneTasks = tasks.filter(task => task.done);

// .reduce() — fold a list into a single value (e.g., a cart total)
const total = cartItems.reduce((sum, item) => sum + item.price, 0);
These are plain JavaScript — React adds nothing special — but they are the idiomatic way to derive display data from state without storing redundant copies.

Conditional rendering uses plain JavaScript inside JSX:

// Short-circuit: only renders when condition is true
{unreadCount > 0 && <Badge count={unreadCount} />}

// Ternary: choose between two alternatives
{isLoggedIn ? <Dashboard /> : <LoginForm />}
Watch out: {count && <Badge />} renders the number 0 when count is 0, because 0 is a valid React node. Use {count > 0 && <Badge />} instead.

Composition Over Inheritance
In C++ and Java, you reuse code via inheritance (class Dog : Animal). React uses composition — building complex UIs by combining small, generic components:

// Generic container — accepts anything as children
function Card({ children, className }) {
  return <div className={'card ' + (className || '')}>{children}</div>;
}

// Specific use — compose with the children prop
function ProfileCard({ user }) {
  return (
    <Card className="profile">
      <Avatar src={user.avatar} />
      <h3>{user.name}</h3>
    </Card>
  );
}
The children prop lets any content be nested inside a component, making it a composable container — analogous to C++ templates or Python’s *args.

Prop drilling
When a value must pass through several intermediate components that don’t use it themselves — only to reach a deeply nested child — the pattern is called prop drilling. It works, but it couples every layer in between to data it doesn’t care about, making refactoring painful. For small trees, prop drilling is fine. When it becomes unwieldy, the typical solutions are lifting state to a closer ancestor or using a context/state-management library.

Thinking in React
React’s official methodology for building a new UI:

Break the UI into a component hierarchy — each component does one job (single-responsibility)
Build a static version first — props only, no state
Identify the minimal state — don’t duplicate data that can be derived
Determine where state lives — the lowest common ancestor that needs it
Add inverse data flow — children call callback functions passed as props
Lifting State Up
When two sibling components need the same data, move the state to their lowest common ancestor and pass it down as props:

function Parent() {
  const [text, setText] = useState('');
  return (
    <>
      <SearchBar value={text} onChange={setText} />
      <ResultsList filter={text} />
    </>
  );
}
SearchBar calls onChange(e.target.value) to notify the parent. The parent updates state, which triggers a re-render of both components. This is “inverse data flow” — data flows down via props, notifications flow up via callbacks.

Top 10 React Best Practices
These are the most important habits to build early. Every one of them prevents real bugs that trip up beginners — and professionals.

1. Use useState for component memory — never bare variables. A let variable inside a component resets to its initial value on every render. Only useState persists data and triggers re-renders when it changes.

2. Keep state minimal — derive what you can. If a value can be computed from existing state or props, compute it during render instead of storing a second copy. Two copies can drift out of sync.

// Good — filter is the only state; visibleTasks is derived
const [filter, setFilter] = useState('all');
const visibleTasks = tasks.filter(t => filter === 'all' || t.status === filter);
3. Never mutate state — always create new arrays and objects. React detects changes by reference. array.push() returns the same reference, so React skips the re-render. Spread into a new array instead.

// Bad — mutates in place, React sees no change
items.push(newItem);
setItems(items);

// Good — new array, React re-renders
setItems([...items, newItem]);
4. Use stable, unique keys for lists — never the array index. Keys tell React which element is which across re-renders. If items are reordered or deleted, index-based keys cause state to attach to the wrong element (e.g., checked checkboxes shifting). Use a unique ID from your data.

5. Destructure props in the function signature. It makes the component’s API visible at a glance and avoids repetitive props. prefixes throughout the body.

// Good
function ProductCard({ name, price, onSale }) { ... }

// Avoid
function ProductCard(props) { return <h3>{props.name}</h3>; }
6. Lift state to the lowest common ancestor. When two sibling components need the same data, move the state up to their nearest shared parent and pass it down as props. The child notifies the parent through a callback prop — never by reaching into siblings directly.

7. One component, one job. If a component handles product display and cart management and filtering, it is doing too much. Split it into focused pieces (ProductCard, CartSummary, FilterBar). Small components are easier to read, test, and reuse.

8. Name event handlers handle*, callback props on*. Inside a component, the function that handles a click is handleClick. When you pass it to a child as a prop, call the prop onClick. This convention makes it immediately clear which end owns the logic and which end fires the event.

function App() {
  const handleDelete = (id) => { /* ... */ };
  return <TodoItem onDelete={handleDelete} />;
}
9. Guard && rendering against falsy numbers. {count && <Badge />} renders the literal 0 when count is 0, because 0 is a valid React node. Use an explicit boolean: {count > 0 && <Badge />}.

10. Follow the two Rules of Hooks. React tracks hooks by their call order. Two rules are non-negotiable:

Only call hooks at the top level — never inside if, loops, or nested functions. If a useState call is skipped on one render, every hook after it shifts position, causing crashes or silent data corruption.
Only call hooks inside React function components (or custom hooks) — never in plain JavaScript utility functions, class methods, or event listeners outside of a component.
Glossary
Glossary table
Term	Definition
Component	A JavaScript function that returns JSX. The building block of React UIs.
JSX	A syntax extension that lets you write HTML-like markup inside JavaScript. A compiler (Babel, SWC, or esbuild) transforms it into JavaScript function calls — historically React.createElement(), and since React 17 the automatic transform calls jsx() from react/jsx-runtime.
Props	Read-only data passed from a parent component to a child, like function arguments.
State	Data managed inside a component via useState. Changing state triggers a re-render.
Hook	A special function (prefixed with use) that lets components use React features. Must be called at the top level.
Re-render	When React re-calls your component function because state or props changed, producing a new JSX tree.
Virtual DOM	A lightweight JavaScript object tree that React builds from your JSX. React diffs the old and new trees and patches only the changed real DOM nodes.
Reconciliation	The algorithm React uses to compare the old and new Virtual DOM trees and determine the minimal set of DOM updates.
Key	A special prop on list items that helps React identify which items changed, were added, or were removed during reconciliation.
Fragment	A wrapper (<>...</>) that groups multiple JSX elements without adding an extra DOM node.
Derived state	A value computed from existing state or props during render, rather than stored in its own useState.
Lifting state up	Moving state to the lowest common ancestor of the components that need it, then passing it down as props.
Stale closure	A bug where an event handler or callback captures an outdated state value from a previous render. Fixed by using the functional setState(prev => ...) pattern.
Functional update	Passing a function to a state setter (setState(prev => prev + 1)) so React provides the latest state value at update time, avoiding stale closure bugs.
State batching	React 18’s optimisation of merging multiple setState calls that happen in the same synchronous tick (event handlers, promises, timeouts, async callbacks) into a single re-render.
Prop drilling	Passing a prop through several intermediate components that don’t use it, just to reach a deeply nested child that does.
Summary
Components: UI is broken down into reusable JavaScript functions.
JSX: We write HTML-like syntax inside JS to describe UI; a compiler turns it into jsx() (modern) or React.createElement (classic) calls.
Props: Data flows one-way from parent to child. Props are read-only.
State: We use useState to give components memory. Updating state triggers re-renders.
Lists & Keys: Use .map() with stable key props for dynamic lists.
Conditional Rendering: Use && and ternary operators inside JSX.
Composition: Build complex UIs by combining small components via the children prop.
Integration: React runs in the user’s browser, acting as the client that makes HTTP requests to your Node.js/Express server.
Ready to Practice?
Head to the React Tutorial for hands-on exercises with immediate feedback — no setup required.

Practice
React Syntax — What Does This Code Do?
You are shown React/JSX code. Explain what it does and what it renders.

Difficulty:
Basic
You are shown React/JSX code. Explain what it does and what it renders.

{isLoggedIn ? <Dashboard /> : <LoginForm />}
Show Answer
React Syntax — Write the Code
You are given a task description. Write the React/JSX code that accomplishes it.

Difficulty:
Advanced
Pass a callback function from a parent to a child component so the child can update the parent’s state.

Show Answer
React Concepts Quiz
Test your deeper understanding of React's design philosophy, state management, and component architecture. Questions 1–7 cover tutorial material. Questions 8–10 test advanced concepts from the reference page. Questions 11–15 cover event handlers, useEffect, and state immutability.

Difficulty:
Advanced
A student stores the full filtered list in state alongside the unfiltered list: const [allTasks, setAllTasks] = useState(tasks) and const [filteredTasks, setFilteredTasks] = useState(tasks). What design problem does this create?


A
This is correct — having both lists in state makes the component faster


B
The only problem is naming — filteredTasks should be called visibleTasks


C
Storing derived data violates minimal state — the filter string is the only real state.


D
This is correct — React requires all displayed data to be in state