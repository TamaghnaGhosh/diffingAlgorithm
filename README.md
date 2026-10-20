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
