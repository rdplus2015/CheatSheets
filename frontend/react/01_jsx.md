# Notes on React JSX and Components

## JSX Syntax

- JSX is regular JavaScript that allows you to write HTML-like syntax.
- It gets converted to regular JavaScript by Babel.
- Attributes are written in camelCase (exceptions: `aria-*` and `data-*`).
- `class` becomes `className` (to avoid conflict with JS `class` keyword).
- Inline styles are written as an object with camelCase CSS properties:
  ```jsx
  <div style={{width: 50, height: 50, backgroundColor: 'blue'}}/>
  ```
- Self-closing tags are required.
- Components must return a single root element (like a `div`) or use a Fragment (`<></>` or `<Fragment></Fragment>`) for a virtual wrapper.

## Variable Interpolation

- JSX allows JavaScript expressions inside `{ ... }`.

```jsx
const text = 'Hello folks';
const id = 'myId';

export function App() {
    return <h1 id={id}>{text}</h1>
}
```

## Styling in React

- Styles are JS objects.
- CSS properties become camelCase.
- Values are strings (except for unitless numbers).

```jsx
<h1 id={id} style={{color: 'red'}}>{text}</h1>

// Or using a separate object
const style = {color: 'red'}
<h1 id={id} style={style}>{text}</h1>
```

## Event Handling

React uses camelCase for event handlers and expects a function.

```jsx
function App() {
  const handleClick = () => {
    alert("Click detected!");
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

Inline function:
```jsx
<button onClick={() => alert('Hi!')}>Click</button>
```

> Defining the function outside JSX is preferred for clarity and performance.

- Examples: `onClick`, `onChange`, `onSubmit`, `onMouseEnter`
- You pass a **function** or **arrow function**.

## Synthetic Events

React wraps native events with `SyntheticEvent` for consistency across platforms — same API, same behavior regardless of the browser.

```jsx
export function App () {
  const doSomething = (e) => {
    e.preventDefault();
    e.stopPropagation();
  };

  return <form onSubmit={doSomething}>Hello folks</form>;
}
```

```jsx
function App() {
  const handleClick = (event) => {
    console.log('Event:', event);
    console.log('Target:', event.target);
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

### Useful Properties & Methods

**`e.target` vs `e.currentTarget`**

- `e.target` — the actual element the user interacted with; can be a child of the element holding the listener.
- `e.currentTarget` — the element the event handler is actually attached to.

```jsx
function App() {
  const handleClick = (e) => {
    console.log('target:', e.target);             // the exact element clicked
    console.log('currentTarget:', e.currentTarget); // always the <div>
  };

  return (
    <div onClick={handleClick}>
      <button>Click me</button>
      <span>or here</span>
    </div>
  );
}
```

Clicking the `<button>` makes `e.target` the `<button>`, while `e.currentTarget` stays the `<div>` (since that's where `onClick` is defined).

**`e.preventDefault()`**

Stops the browser's default behavior for that event:
- On `<form onSubmit={...}>` — prevents a full page reload (the native HTML form behavior)
- On `<a href="...">` — prevents navigation

This is why it's almost always used on form submissions in React — without it, the page reloads and all component state is lost.

**`e.stopPropagation()`**

Stops the event from bubbling up to parent elements. By default, a click on a child element also triggers `onClick` handlers on all its ancestors (event bubbling).

```jsx
function App() {
  const handleParentClick = () => console.log('Parent clicked');
  const handleChildClick = (e) => {
    e.stopPropagation(); // prevents handleParentClick from firing
    console.log('Child clicked');
  };

  return (
    <div onClick={handleParentClick}>
      <button onClick={handleChildClick}>Click me</button>
    </div>
  );
}
```

Without `stopPropagation()`, clicking the button logs both "Child clicked" and "Parent clicked". With it, only "Child clicked" logs.

**`e.nativeEvent`**

Gives access to the actual underlying browser event, bypassing React's `SyntheticEvent` wrapper. Rarely needed — mostly for browser APIs not exposed through the synthetic event.

```jsx
const handleClick = (e) => {
  console.log(e.nativeEvent); // raw DOM event, not the React wrapper
};
```

### Common React Events

| Event Type             | React Attribute               | JSX Example                        |
|------------------------|-------------------------------|------------------------------------|
| **Click**              | `onClick`                     | `<button onClick={...} />`        |
| **Keyboard**           | `onKeyDown`, `onKeyUp`        | `<input onKeyDown={...} />`       |
| **Form input**         | `onChange`                    | `<input onChange={...} />`        |
| **Form submission**    | `onSubmit`                    | `<form onSubmit={...} />`         |
| **Mouse**              | `onMouseEnter`, etc.          | `<div onMouseMove={...} />`       |
| **Focus / Blur**       | `onFocus`, `onBlur`           | `<input onBlur={...} />`          |
| **Context menu**       | `onContextMenu`               | `<div onContextMenu={...} />`     |
| **Drag & Drop**        | `onDragStart`, `onDrop`       | `<div onDrop={...} />`            |
| **Double click**       | `onDoubleClick`               | `<div onDoubleClick={...} />`     |
| **Clipboard**          | `onCopy`, `onPaste`           | `<input onPaste={...} />`         |
| **Scroll**             | `onScroll`                    | `<div onScroll={...} />`          |
| **Touch (mobile)**     | `onTouchStart`, etc.          | `<div onTouchStart={...} />`      |
| **Animation**          | `onAnimationEnd`, etc.        | `<div onAnimationEnd={...} />`    |

## Conditional Rendering

JSX does not support `if` statements directly.

### Valid methods

```jsx
{isVisible && <Modal/>}
{isLoading ? <Spinner/> : <Data/>}
```

Intermediate variable:
```jsx
let content;
if (isLoading) content = <Spinner/>
else content = <Data/>

return <div>{content}</div>
```

### Invalid

```jsx
return (
  <div>
    { if (isLoading) { return <Spinner /> } } //  ERROR
  </div>
);
```

| Method               | Valid in JSX | Recommended Use         |
|----------------------|--------------|--------------------------|
| `if/else` outside JSX| Yes          | For clarity              |
| Ternary `? :`        | Yes          | For short conditions     |
| Logical `&&`         | Yes          | For single elements      |
| `if` inside `{}`     | No           | Not allowed in JSX       |

## Rendering Lists with `map()`

- Transforms an array into JSX elements
- React requires a unique `key` per element

```jsx
const todos = ['Task 1', 'Task 2', 'Task 3'];

export function App() {
  return (
    <>
      <h1>My todo list</h1>
      <ul>
        {todos.map(todo => (
          <li key={todo}>{todo}</li>
        ))}
      </ul>
    </>
  );
}
```

### Why the `key` prop matters

When React re-renders a list, it needs to decide whether to reuse existing DOM elements or destroy/recreate them. Without a stable `key`, React compares list items **by position only**.

Example: deleting the first item of `['Task 1', 'Task 2', 'Task 3']` leaves `['Task 2', 'Task 3']`. Without a reliable key, React assumes position 0's text changed from "Task 1" to "Task 2", and position 1's text changed from "Task 2" to "Task 3" — then deletes the now-unused third `<li>`. Visually correct, but the underlying DOM elements are now mismatched with their data.

This becomes a real bug when list items hold their own state — an `<input>` mid-typing, a checked checkbox, an open/closed toggle. That state stays attached to the DOM element, not to the data, so it can end up on the wrong item after a reorder or deletion.

With a stable, unique `key` (usually a database id — never the array index if the list can reorder), React matches items **by identity** instead of position: it knows exactly which element was removed, added, or unchanged, and leaves the others untouched.

```jsx
// Prefer a stable unique id over the array index
todos.map(todo => <li key={todo.id}>{todo.text}</li>)
```

**In short:** `key` is an identity card for each list item, so React knows "this is the same element as before" regardless of where it now sits in the list.

## Functional Components

A component is just a JavaScript function that returns JSX. Two rules: the name must be **PascalCase**, and it must return JSX (or `null`). The PascalCase naming is how React distinguishes `<Title>` (a component call) from `<title>` (a native HTML tag).

```jsx
function Title({ color, children }) {
  return <h1 style={{ color }}>{children}</h1>;
}
```

Without destructuring, the same thing:
```jsx
function Title(props) {
  return <h1 style={{ color: props.color }}>{props.children}</h1>;
}
```

### `props`

Writing `<Title color="blue" />` makes React build an object `{ color: "blue" }` and pass it as the function's single argument. Destructuring it in the signature (`{ color }`) is just a shorthand for `props.color`.

### `children`

Anything placed **between** a component's opening and closing tags is automatically passed as the `children` prop.

```jsx
<Title color="red">This is a title</Title>
// equivalent to <Title color="red" children="This is a title" />
```

### Extra / unused props

Props that are passed but not destructured are simply ignored — no error, no warning.

```jsx
function Title({ color }) {
  return <h1 style={{ color }}>Hello +</h1>;
}
<Title color="blue" size="large" className="title" /> // size and className are silently unused
```

Accessing everything at once (no destructuring):
```jsx
function Title(props) {
  console.log(props); // { color, size, className, children }
  return <h1 style={{ color: props.color }}>{props.children}</h1>;
}
```

### Spread operator — capturing "the rest"

```jsx
function Title({ color, children, ...props }) {
  return <h1 style={{ color }} {...props}>{children}</h1>;
}
```

`...props` collects every prop not explicitly destructured (`className`, `id`, `onClick`...) into a new object, which `{...props}` then spreads as real HTML attributes on the `<h1>`. Common pattern for building flexible, reusable components that handle a few specific props while transparently forwarding the rest.

### Why component-based UI

- Less repetition
- Code reuse across the app
- Clearer organization (one responsibility per component)
- Easier to test and maintain in isolation

### Summary

| Concept | Explanation |
|---|---|
| Function = component | Must be PascalCase |
| `props` | Object of passed attributes |
| `children` | Content between `<Title>...</Title>` |
| Destructuring | Pull specific values from `props` |
| `...props` | Capture the rest of `props` |