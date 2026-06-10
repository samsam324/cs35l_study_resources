# React Cheatsheet (candidate items)

## Component
```jsx
function Welcome({ name }) {
    return <h1>Hello {name}</h1>;
}

// usage
<Welcome name="Alice" />
```

## JSX rules
- Must return ONE root element (use `<>...</>` fragment for multiple)
- `className` not `class`
- `htmlFor` not `for`
- `{expression}` for JS in JSX
- `style={{color: 'red'}}` (object, not string)
- Self-close tags: `<img />`, `<input />`

## Props
```jsx
function Btn({ label, onClick, disabled = false }) {
    return <button onClick={onClick} disabled={disabled}>{label}</button>;
}
```
- Props are **read-only** in the child
- Destructure in the parameter signature

## useState
```jsx
import { useState } from "react";

function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```
- `setCount(newValue)` — direct
- `setCount(prev => prev + 1)` — functional (safer for concurrent updates)

## useEffect
```jsx
useEffect(() => {
    // runs after render
    return () => { /* cleanup on unmount or dep change */ };
}, [dep1, dep2]);
```
Dependency array semantics:
- `[]` — run ONCE on mount
- `[dep]` — re-run when dep changes
- omitted — run EVERY render (rarely what you want)

## useEffect cleanup
```jsx
useEffect(() => {
    const id = setInterval(tick, 1000);
    return () => clearInterval(id);
}, []);
```

## Lists & keys
```jsx
{items.map(item => (
    <li key={item.id}>{item.name}</li>
))}
```
- Each child in a list MUST have a unique `key`
- Use stable IDs, not array indexes (unless list never reorders)

## Conditional rendering
```jsx
{isLoggedIn && <Welcome />}
{count > 0 ? <p>Yes</p> : <p>No</p>}
{!loaded && <Spinner />}
```

## Event handlers
```jsx
<button onClick={() => doThing()}>Click</button>
<input onChange={e => setText(e.target.value)} />
<form onSubmit={e => { e.preventDefault(); ... }}>...</form>
```

## Lifting state up
- If two siblings share data → move state to their COMMON PARENT
- Pass down via props; pass setter as callback prop

## Controlled vs uncontrolled inputs
```jsx
// Controlled (React owns the value)
<input value={text} onChange={e => setText(e.target.value)} />

// Uncontrolled (DOM owns it; React uses ref to read)
<input ref={inputRef} />
```

## Other hooks (quick mention)
- `useRef` — mutable ref that persists across renders
- `useMemo` — memoize expensive computation
- `useCallback` — memoize a function reference
- `useContext` — consume context

## Virtual DOM
- React maintains a virtual DOM (JS object tree)
- On state change, computes new virtual DOM
- "Diffs" against previous; only modifies real DOM where needed
- Why React feels fast

## Component lifecycle (function-component model)
1. Mount: function runs first time
2. Render: returns JSX
3. useEffect callbacks fire
4. Re-render on state/prop change
5. Unmount: cleanup functions run

## Patterns
- **Lift state up** — siblings share parent state
- **Render props / children as function** — share logic via props
- **Custom hooks** — reusable stateful logic (prefix with `use`)

## SPA vs SSR
- **SPA** (Single Page App) — one HTML page, JS renders everything client-side
- **SSR** (Server-Side Rendering) — HTML generated on server, hydrated by JS (Next.js)
- SSR better for SEO and initial-load perf

## File structure idiom
```
src/
  components/
    Counter.jsx
    Button.jsx
  App.jsx
  index.jsx
```

## Common traps
- Mutating state directly (`arr.push(x)`) — React won't re-render. Always create a NEW value: `setArr([...arr, x])`
- Forgetting `key` in lists → React warning + bad performance
- Stale closures: useEffect callback captures old state — list it in deps, or use functional setter
- Calling hooks conditionally — hooks MUST be at top level of component
- `useEffect` infinite loops from missing/incorrect deps

## Observer Pattern (used by React internally)
- React's whole model is Observer: components subscribe to state, re-render on changes
- Lecture used Subject/Observer with NewsChannel/MobileApp/EmailDigest example

## Additional items (potentially missing)

### useMemo — cache a computed value
```jsx
const expensiveResult = useMemo(() => {
    return heavyCalc(data);
}, [data]);   // only recomputes when data changes
```

### useCallback — cache a function reference
```jsx
const handleClick = useCallback((id) => {
    setSelected(id);
}, [setSelected]);
```
Useful when passing callbacks to memoized children (prevents needless re-renders).

### useRef — mutable value that persists
```jsx
const inputRef = useRef(null);
useEffect(() => { inputRef.current.focus(); }, []);
return <input ref={inputRef} />;
```
Also useful for storing values that DON'T need to trigger re-render.

### useContext — share state across tree
```jsx
const ThemeContext = createContext("light");

function App() {
    return (
        <ThemeContext.Provider value="dark">
            <Toolbar />
        </ThemeContext.Provider>
    );
}

function Button() {
    const theme = useContext(ThemeContext);
    return <button className={theme}>Click</button>;
}
```

### State immutability
- Never mutate state directly — React won't detect the change
```jsx
// BAD
items.push(newItem);
setItems(items);

// GOOD
setItems([...items, newItem]);
setItems(prev => [...prev, newItem]);
```

For objects:
```jsx
// BAD
user.name = "Bob";
setUser(user);

// GOOD
setUser({...user, name: "Bob"});
```

### Rules of Hooks
1. **Only call hooks at the top level** — not inside loops, conditions, or nested functions
2. **Only call hooks from React functions** — not regular JS functions
3. **Hook order must be stable** across renders

### Custom hook
```jsx
function useFetch(url) {
    const [data, setData] = useState(null);
    useEffect(() => {
        fetch(url).then(r => r.json()).then(setData);
    }, [url]);
    return data;
}

// Usage in component
function MyComp() {
    const data = useFetch("/api/items");
    return data ? <List items={data} /> : <Spinner />;
}
```
Convention: name MUST start with `use`.

### Forwarding refs
```jsx
const FancyButton = forwardRef((props, ref) => (
    <button ref={ref}>{props.children}</button>
));
```

### Children prop
```jsx
function Card({ children }) {
    return <div className="card">{children}</div>;
}

// Usage
<Card>
    <h1>Title</h1>
    <p>Body</p>
</Card>
```

### Form handling pattern
```jsx
function Form() {
    const [name, setName] = useState("");
    const [email, setEmail] = useState("");

    const handleSubmit = (e) => {
        e.preventDefault();             // prevent page reload
        submit({name, email});
    };

    return (
        <form onSubmit={handleSubmit}>
            <input value={name} onChange={e => setName(e.target.value)} />
            <input value={email} onChange={e => setEmail(e.target.value)} />
            <button type="submit">Send</button>
        </form>
    );
}
```

### Fragment shorthand
```jsx
return (
    <>
        <h1>Hello</h1>
        <p>World</p>
    </>
);
```

### Conditional class names
```jsx
<div className={`base ${isActive ? "active" : ""}`}>...</div>
```

### Pass children as function (render prop)
```jsx
<DataLoader>{(data) => <List items={data}/>}</DataLoader>
```

### Anti-patterns
- Mutating state directly
- Using index as `key` in dynamic/reorderable lists
- Calling hooks conditionally
- Massive useEffect with many concerns (split into multiple)
- Using state for derived values (just compute on render)

### Performance tips
- Use `React.memo(Component)` for expensive children
- Use `useMemo` for expensive computations
- Use `useCallback` for callback identity stability
- Lazy load: `React.lazy(() => import('./MyComponent'))`

### Lifecycle (class components, legacy)
- `constructor` → `render` → `componentDidMount`
- On update: `render` → `componentDidUpdate`
- On unmount: `componentWillUnmount`
- Function components use hooks instead of these.

### CSS-in-JSX styling options
- Inline: `style={{color: 'red'}}`
- CSS classes: `className="btn primary"`
- CSS modules: `import styles from './x.module.css'`
- Styled components: `styled.div\`...\``
