# Diffing Algorithm

A small, dependency-free demonstration of a virtual DOM and a recursive diffing algorithm. The entire application is in [`diffingAlgorithm.html`](diffingAlgorithm.html); open that file in a modern browser to run it.

## What the app does

The page displays a counter with **+** and **−** buttons. Clicking a button changes the counter and renders a new virtual tree. On the first render, the tree is turned into real DOM nodes. On later renders, the diffing function compares the old and new trees and updates the existing DOM where it can.

## Run it

No build tools, server, or dependencies are required.

1. Open `diffingAlgorithm.html` in a browser (or serve the project directory with any static file server).
2. Click **+** to increment the counter.
3. Click **−** to decrement it. The value is clamped at zero.

## Code walkthrough

### 1. Virtual node factory: `h`

```js
function h(type, props, ...children) {
  return { type, props: { ...props, children: children.flat() } };
}
```

`h` creates a plain JavaScript object describing an element rather than creating a browser DOM node immediately. For example, `h("h1", {}, "Counter: 0")` describes an `h1` with one text child. The `...children` parameter accepts any number of children, and `flat()` flattens nested child arrays by one level.

The resulting virtual node shape is `{ type, props }`. Its `props.children` property is reserved for the child nodes; other properties represent HTML attributes or event handlers.

### 2. Initial DOM creation: `createDOMElement`

`createDOMElement(vNode)` converts a virtual node into a real DOM node recursively:

- If the node is a string, it creates a text node.
- Otherwise, it creates an element using `document.createElement(vNode.type)`.
- It walks through the node's properties, skipping `children`.
- Properties whose names start with `on` are registered as event listeners. For example, `onClick` becomes a `click` listener.
- Other properties are applied as HTML attributes using `setAttribute`.
- Each child is converted recursively and appended to its parent.

This function is used for the first mount and whenever diffing must replace or add a node.

### 3. Updating the DOM: `diff`

`diff(oldVNode, newVNode, parentEl, index)` compares two virtual nodes and patches the real DOM to match the new node.

#### Find the existing DOM node

For the root call, `index` is omitted and the existing element is read from `oldVNode.el`. For a child call, the existing node is taken from `parentEl.childNodes[index]`.

The root DOM element is stored on the root virtual node during initial rendering. Child nodes are located by their position in the parent's `childNodes` list.

#### Replace incompatible nodes

If one node is text and the other is an element, or if both are elements with different tag names, the old DOM node is replaced with a newly created one. A changed root node also gets its `.el` reference updated.

#### Update text nodes

When both nodes are text, their values are compared. If the text changed, `textContent` is updated; otherwise the DOM is left alone.

#### Reuse elements and update properties

When both nodes are elements with the same type, the existing DOM element is reused and assigned to `newVNode.el`.

The algorithm removes old properties that are absent from the new node. Removed event handlers are detached with `removeEventListener`; removed attributes are cleared with `removeAttribute`. It then applies the new properties. For event handlers, it removes a prior handler for that event before adding the new one.

#### Reconcile children

Children at matching positions are diffed recursively. If the new list is shorter, extra old DOM children are removed from the end. If it is longer, the remaining new children are converted and appended.

This is a position-based reconciliation strategy: it does not use keys to track children that move or are reordered.

### 4. Counter application: `count` and `render`

`count` holds the app's state. Each call to `render()` builds a fresh virtual tree using `h`:

- A `div.app` wraps the counter heading and buttons.
- The heading displays the current count.
- The **+** button increments `count` and calls `render()`.
- The **−** button decrements `count`, clamps it to zero, and calls `render()`.

On the first call, `render()` creates the full DOM tree, stores it in `render.prev`, and appends it to `document.body`. On subsequent calls, it diffs the previous tree against the newly built one, then saves the new tree as `render.prev` for the next update.

The final `render()` call starts the app by performing its initial mount.

## Limitations

- Child matching is based on position; there are no keys for preserving identity through reorderings.
- Event properties are treated as event listeners only when the property name starts with `on`.
- Non-event properties are applied as attributes, so DOM properties such as `value`, boolean attributes, and styles do not receive framework-specific handling.
- This is an educational example, not a production-ready rendering library.

## Project files

| File | Purpose |
| --- | --- |
| `diffingAlgorithm.html` | HTML page, virtual DOM implementation, diffing logic, and counter app |
| `README.md` | Project overview and code explanation |

## React's Virtual DOM + Diffing algorithm written using plain JavaScript

Absolutely. This code is a **small version of React's Virtual DOM + Diffing algorithm** written using plain JavaScript.

The easiest way to understand it is:

> **Virtual DOM = JavaScript object describing the UI**  
> **Real DOM = Actual HTML elements in the browser**  
> **Diffing = Compare old Virtual DOM with new Virtual DOM and update only what changed**

Let's go step by step.

---

# 1. The overall flow

When you click `+`, this happens:

```text
Click +
   ↓
count++
   ↓
render()
   ↓
Create NEW Virtual DOM tree
   ↓
Compare OLD Virtual DOM vs NEW Virtual DOM
   ↓
Find what changed
   ↓
Update only that DOM element
```

For example:

Initially:

```text
Counter: 0
```

After clicking `+`:

```text
Counter: 1
```

The algorithm doesn't recreate the whole page.

It finds:

```text
"Counter: 0"
        ↓
"Counter: 1"
```

and updates only that text.

---

# 2. `h()` — Creating a Virtual DOM node

```js
function h(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.flat()
    }
  };
}
```

This function creates a **JavaScript object representing an HTML element**.

For example:

```js
h("h1", {}, "Hello")
```

creates:

```js
{
  type: "h1",
  props: {
    children: ["Hello"]
  }
}
```

So instead of immediately creating:

```html
<h1>Hello</h1>
```

we first create a JavaScript object describing it.

That's our **Virtual DOM node**.

---

## Example

This:

```js
h(
  "button",
  {
    onClick: () => console.log("clicked")
  },
  "+"
)
```

becomes approximately:

```js
{
  type: "button",

  props: {
    onClick: function() {...},
    children: ["+"]
  }
}
```

So:

```text
h()
 ↓
JavaScript object
 ↓
Virtual DOM
```

---

# 3. `createDOMElement()` — Virtual DOM → Real DOM

Now we need to convert our JavaScript object into an actual browser element.

```js
function createDOMElement(vNode) {
```

`vNode` means **Virtual DOM Node**.

---

## Step 1: Check if it is text

```js
if (typeof vNode === "string") {
  return document.createTextNode(vNode);
}
```

For example:

```js
"Counter: 0"
```

is just text.

So:

```js
document.createTextNode("Counter: 0")
```

creates a real DOM text node.

---

# 4. Create the HTML element

If it isn't text:

```js
const el = document.createElement(vNode.type);
```

Suppose:

```js
vNode.type === "button"
```

Then:

```js
document.createElement("button")
```

creates:

```html
<button></button>
```

---

# 5. Add attributes

This part:

```js
if (vNode.props) {
  for (const [key, value] of Object.entries(vNode.props)) {
```

loops through the properties.

For example:

```js
{
  class: "app"
}
```

Then:

```js
el.setAttribute(key, value);
```

does:

```html
<div class="app"></div>
```

---

# 6. Special handling for events

There is this:

```js
if (key.startsWith("on")) {
```

Why?

Because:

```js
onClick
```

is an event handler, not a normal HTML attribute.

So:

```js
el.addEventListener(
  key.slice(2).toLowerCase(),
  value
);
```

For:

```js
onClick
```

this:

```js
key.slice(2)
```

removes `"on"`:

```text
onClick
  ↓
Click
  ↓
click
```

So it becomes:

```js
el.addEventListener("click", value);
```

---

# 7. Create the children

This part:

```js
if (vNode.props?.children) {
  vNode.props.children.forEach((child) =>
    el.appendChild(createDOMElement(child))
  );
}
```

means:

> For every child in the Virtual DOM, create a real DOM element and add it.

For example:

```js
h("h1", {}, "Counter: 0")
```

becomes:

```text
h1
 └── "Counter: 0"
```

Then:

```js
createDOMElement(...)
```

creates:

```html
<h1>Counter: 0</h1>
```

---

# 8. Now let's look at the actual application

You have:

```js
let count = 0;
```

So initially:

```text
count = 0
```

Then:

```js
render();
```

runs.

---

# 9. `render()` creates the Virtual DOM

Inside `render()`:

```js
const tree = h(
  "div",
  { class: "app" },
```

This creates:

```html
<div class="app">
```

Then it has:

```js
h("h1", {}, "Counter: " + count)
```

Since:

```js
count = 0
```

this becomes:

```text
Counter: 0
```

So:

```html
<h1>Counter: 0</h1>
```

---

# 10. The first button

```js
h(
  "button",
  {
    onClick: () => {
      count++;
      render();
    },
  },
  "+",
)
```

This means:

```html
<button>+</button>
```

When the button is clicked:

```js
count++;
```

So:

```text
0 → 1
```

Then:

```js
render();
```

runs again.

---

# 11. The second button

```js
h(
  "button",
  {
    onClick: () => {
      count--;
      if (count < 0) count = 0;
      render();
    },
  },
  "−",
)
```

This creates:

```html
<button>−</button>
```

Clicking it does:

```text
count--
```

But this prevents negative values:

```js
if (count < 0) count = 0;
```

---

# 12. What does the Virtual DOM tree look like?

Initially, roughly:

```text
div.app
│
├── h1
│   └── "Counter: 0"
│
├── button
│   └── "+"
│
└── button
    └── "−"
```

This is just JavaScript objects.

The browser eventually gets:

```html
<div class="app">
  <h1>Counter: 0</h1>
  <button>+</button>
  <button>−</button>
</div>
```

---

# 13. First render

This part handles the first render:

```js
if (!render.prev) {
```

At the beginning:

```js
render.prev
```

doesn't exist.

So we enter here:

```js
render.prev = tree;
```

We save the current Virtual DOM tree.

Think:

```text
render.prev = OLD Virtual DOM
```

Then:

```js
render.prev.el = createDOMElement(tree);
```

We convert the Virtual DOM into real DOM.

Then:

```js
document.body.appendChild(render.prev.el);
```

puts it into the webpage.

So:

```text
Virtual DOM
     ↓
createDOMElement()
     ↓
Real DOM
     ↓
document.body
```

---

# 14. What happens when you click `+`?

Initially:

```text
count = 0
```

Virtual DOM:

```text
<h1>Counter: 0</h1>
```

Click `+`.

This runs:

```js
count++;
```

Now:

```text
count = 1
```

Then:

```js
render();
```

creates a **new Virtual DOM**.

Now it contains:

```text
<h1>Counter: 1</h1>
```

So now we have:

### Old Virtual DOM

```text
<h1>Counter: 0</h1>
```

### New Virtual DOM

```text
<h1>Counter: 1</h1>
```

Now we need to compare them.

That's what `diff()` does.

---

# 15. `diff()` — The most important part

```js
function diff(oldVNode, newVNode, parentEl, index)
```

Think of it as:

> **Compare the old UI with the new UI and make the real DOM match the new UI.**

---

# 16. Find the existing DOM element

```js
const oldEl =
  index === undefined
    ? oldVNode.el
    : parentEl.childNodes[index];
```

Basically:

> Find the real DOM element corresponding to the old Virtual DOM node.

For example:

```text
Old Virtual DOM
      ↓
<h1>Counter: 0</h1>
      ↓
Real DOM
      ↓
<h1>Counter: 0</h1>
```

---

# 17. Check whether nodes are text

```js
const oldIsText = typeof oldVNode === "string";
const newIsText = typeof newVNode === "string";
```

For example:

```js
"Counter: 0"
```

is text.

So:

```js
oldIsText = true
```

---

# 18. Check if the element type changed

This is very important:

```js
if (
  oldIsText !== newIsText ||
  (!oldIsText && oldVNode.type !== newVNode.type)
)
```

Suppose old node:

```js
h("div", {}, ...)
```

and new node:

```js
h("section", {}, ...)
```

The types are different:

```text
div
 ↓
section
```

So instead of trying to modify the `<div>` into `<section>`, we simply replace it.

```js
const newEl = createDOMElement(newVNode);

parentEl.replaceChild(newEl, oldEl);
```

---

# 19. Text node update

Now imagine:

```text
Old:
"Counter: 0"

New:
"Counter: 1"
```

Both are text.

So this condition is false:

```js
if (oldIsText !== newIsText ...)
```

Then:

```js
if (oldIsText) {
  if (oldVNode !== newVNode)
    oldEl.textContent = newVNode;

  return;
}
```

It sees:

```text
"Counter: 0" !== "Counter: 1"
```

So it does:

```js
oldEl.textContent = "Counter: 1";
```

That's the actual update in the browser.

**It doesn't recreate the `<h1>`.**

Only the text changes.

---

# 20. This is the core idea of diffing

Before:

```html
<h1>Counter: 0</h1>
```

After:

```html
<h1>Counter: 1</h1>
```

Instead of:

```text
Delete entire h1
Create new h1
```

we do:

```text
Find h1
   ↓
Find text
   ↓
Change "0" → "1"
```

That's the basic idea behind Virtual DOM diffing.

---

# 21. What is `newVNode.el = oldEl`?

```js
newVNode.el = oldEl;
```

This connects the new Virtual DOM node to the existing real DOM element.

Think:

```text
New Virtual DOM
      │
      │ .el
      ↓
Existing DOM element
```

So the algorithm knows:

> "This new Virtual DOM node corresponds to this existing browser element."

---

# 22. Updating attributes

Next:

```js
const oldProps = oldVNode.props || {};
const newProps = newVNode.props || {};
```

Suppose:

```text
OLD:
class="red"

NEW:
class="blue"
```

The algorithm compares them.

First it looks for removed properties:

```js
for (const key of Object.keys(oldProps)) {
```

If something existed before but doesn't exist now:

```js
if (!(key in newProps))
```

remove it:

```js
oldEl.removeAttribute(key);
```

---

# 23. Updating new properties

Then:

```js
for (const [key, value] of Object.entries(newProps)) {
```

It loops through the new properties.

For normal attributes:

```js
oldEl.setAttribute(key, value);
```

For events:

```js
oldEl.removeEventListener(...)
oldEl.addEventListener(...)
```

This is important because every `render()` creates new arrow functions:

```js
onClick: () => {
  count++;
  render();
}
```

So the old event listener needs to be removed and the new one attached.

---

# 24. Now comes children reconciliation

This is another important section:

```js
const oldChildren = oldProps.children || [];
const newChildren = newProps.children || [];
```

For example:

### Old

```text
div
├── h1 "Counter: 0"
├── button "+"
└── button "−"
```

### New

```text
div
├── h1 "Counter: 1"
├── button "+"
└── button "−"
```

The algorithm compares these children one by one.

---

# 25. Compare children at the same position

```js
const sharedLength =
  Math.min(oldChildren.length, newChildren.length);
```

Both have 3 children:

```text
old = 3
new = 3

sharedLength = 3
```

Then:

```js
for (let i = 0; i < sharedLength; i++) {
  diff(oldChildren[i], newChildren[i], oldEl, i);
}
```

So:

```text
Child 0 → compare
Child 1 → compare
Child 2 → compare
```

---

# 26. What happens to child 0?

Old:

```text
<h1>Counter: 0</h1>
```

New:

```text
<h1>Counter: 1</h1>
```

`diff()` is called again.

It sees:

```text
h1 → h1
```

Same type.

So it doesn't replace the `<h1>`.

Then it compares the children:

```text
"Counter: 0"
        ↓
"Counter: 1"
```

Then:

```js
oldEl.textContent = newVNode;
```

Done.

---

# 27. What about the buttons?

Old:

```text
<button>+</button>
```

New:

```text
<button>+</button>
```

Same type.

The button itself stays.

Its event handler gets updated.

Same thing for:

```text
<button>−</button>
```

---

# 28. Removing children

This part:

```js
for (
  let i = oldChildren.length - 1;
  i >= newChildren.length;
  i--
) {
  oldEl.removeChild(oldEl.childNodes[i]);
}
```

handles when the new Virtual DOM has **fewer children**.

Example:

```text
Old:
div
├── A
├── B
└── C

New:
div
├── A
└── B
```

The algorithm removes:

```text
C
```

from the real DOM.

---

# 29. Adding children

This part:

```js
for (
  let i = sharedLength;
  i < newChildren.length;
  i++
) {
  oldEl.appendChild(
    createDOMElement(newChildren[i])
  );
}
```

handles new children.

Example:

```text
Old:
div
├── A
└── B

New:
div
├── A
├── B
└── C
```

It creates:

```html
C
```

and appends it.

---

# 30. Finally update `render.prev`

At the end:

```js
render.prev = tree;
```

This is very important.

It means:

> The new Virtual DOM becomes the old Virtual DOM for the next render.

So:

```text
Before click:

render.prev
    ↓
Counter: 0
```

After click:

```text
render.prev
    ↓
Counter: 1
```

Then if you click again:

```text
Counter: 1
       ↓
Counter: 2
```

The algorithm always compares:

```text
previous render
      VS
current render
```

---

# 31. Complete example: clicking `+`

Let's follow the entire process.

### Initial state

```js
count = 0
```

`render()`:

```text
Virtual DOM:

div
├── h1 "Counter: 0"
├── button "+"
└── button "−"
```

Real DOM:

```html
<div class="app">
  <h1>Counter: 0</h1>
  <button>+</button>
  <button>−</button>
</div>
```

---

### User clicks `+`

Event handler:

```js
count++;
```

Now:

```text
count = 1
```

Then:

```js
render();
```

New Virtual DOM:

```text
div
├── h1 "Counter: 1"
├── button "+"
└── button "−"
```

---

### `diff()` compares

```text
OLD                         NEW

div                         div
├── h1                      ├── h1
│   └── Counter: 0          │   └── Counter: 1
├── button +                ├── button +
└── button −                └── button −
```

It sees:

```text
div = div       → keep
h1 = h1         → keep
text changed    → update
button = button → keep
button = button → keep
```

So the browser only changes:

```text
Counter: 0
     ↓
Counter: 1
```

---

# 32. Why is this called "Diffing"?

Because we're calculating the **difference** between:

```text
OLD UI
```

and:

```text
NEW UI
```

For example:

```text
OLD:
Counter: 0

NEW:
Counter: 1
```

Difference:

```text
0 → 1
```

Then we apply only that change to the real DOM.

---

# 33. One important limitation of this implementation

This is a **simplified diffing algorithm**, not React's full implementation.

For example, it compares children by their **position**:

```text
old child 0 ↔ new child 0
old child 1 ↔ new child 1
old child 2 ↔ new child 2
```

It doesn't have React's `key` mechanism.

Suppose:

```text
Old:
A
B
C
```

New:

```text
C
A
B
```

This implementation doesn't intelligently understand that the items moved.

It mostly sees:

```text
A → C
B → A
C → B
```

and updates them.

Real frameworks use more sophisticated reconciliation strategies.

---

# 34. The entire code in one simple picture

```text
                    render()
                       │
                       ▼
              Create Virtual DOM
                       │
              ┌────────┴────────┐
              │                 │
         First render       Later render
              │                 │
              ▼                 ▼
      createDOMElement()       diff()
              │                 │
              ▼                 ▼
          Real DOM       Compare old/new
                                │
                       ┌────────┼────────┐
                       │        │        │
                    Replace   Update   Add/remove
                    element   props    children
                       │        │        │
                       └────────┼────────┘
                                ▼
                           Updated DOM
```

---

# 35. The 5 functions you should remember

If you're explaining this in an interview, remember these five concepts:

| Function | Simple meaning |
|---|---|
| `h()` | Creates a Virtual DOM object |
| `createDOMElement()` | Converts Virtual DOM → Real DOM |
| `render()` | Creates the UI and starts the update process |
| `diff()` | Compares old and new Virtual DOM |
| `createDOMElement()` during diff | Creates new DOM when needed |

And the most important concept:

> **We don't blindly rebuild the entire DOM on every state change. We create a new Virtual DOM, compare it with the previous Virtual DOM, and update only the parts that changed.**

### Interview-friendly answer

> "This code implements a simplified Virtual DOM. The `h()` function creates JavaScript objects representing HTML elements. On the first render, `createDOMElement()` converts those objects into real DOM nodes. When state changes, `render()` creates a new Virtual DOM tree. The `diff()` function compares the old and new trees, checks element types, attributes, text, and children, and updates only the changed parts of the real DOM. After that, the new tree becomes the previous tree for the next comparison."

