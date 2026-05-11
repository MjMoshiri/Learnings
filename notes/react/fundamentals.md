---
title: "React Fundamentals"
tags: [ "react", "state-management", "frontend", "immutability" ]
prerequisites: []
status: "draft"
last_updated: "2025-12-23"
todos:
  - "Add examples of Context API."
  - "Discuss external state management libraries."
---

## Summary
Effective state management in React relies on understanding how data flows through the component tree, the importance of immutability for change detection, and the lifecycle of state updates during renders.

## Core Concepts

### State Placement & Flow
*   **Lifting State Up:** State should be placed in the closest common parent component that needs access to the data. This allows sharing state between sibling components via props.
*   **Snapshots:** Every render is isolated. The function body (and functions defined inside it) sees a fixed "snapshot" of the state and props for that specific render cycle. Changing state does not change the variable in the current running function; it triggers a *new* render with the new value.

### Updating State
*   **Updater Functions:** When an update depends on the previous state, pass a function to the setter (e.g., `setCount(prev => prev + 1)`). React queues this function to run after the current render, ensuring consistency even with batched updates.
*   **Immutability:** React compares objects by reference. To trigger a re-render, you must provide a *new* object/array reference, not mutate the existing one.

#### Array Operations Cheatsheet

| Operation | Avoid (Mutates) | Prefer (Returns New Reference) |
| :--- | :--- | :--- |
| **Adding** | `push`, `unshift` | `concat`, `[...arr]` (spread syntax) |
| **Removing** | `pop`, `shift`, `splice` | `filter`, `slice` |
| **Replacing** | `splice`, `arr[i] = ...` | `map` |
| **Sorting** | `reverse`, `sort` | Copy the array first (e.g. `[...arr]`), then sort |

*   **Helper Libraries:** Tools like **use-immer** allow you to write "mutable-style" logic (e.g., `draft.push(1)`) on a proxy object, which it then converts into a correct immutable update.

### Structuring State
*   **Group Related State:** If two state variables always update together, consider merging them into a single object.
*   **Avoid Contradictions:** Structure state to prevent "impossible" states.
    *   *Bad:* `const [isSending, setIsSending] = useState(false); const [isSent, setIsSent] = useState(false);` (Both could be true).
    *   *Good:* `const [status, setStatus] = useState('typing');` (Values: 'typing', 'sending', 'sent').
*   **Avoid Redundancy:** Do not store data that can be computed from existing props or state.
    *   *Bad:* `firstName`, `lastName`, `fullName` (Calculate `fullName` during render).
    *   *Bad:* `items`, `selectedItem` (Store `selectedId` instead to avoid sync issues).
*   **No Props in State:** Avoid initializing state with props (e.g., `useState(props.val)`) unless you intentionally want to ignore future prop updates.
*   **Flatten Data:** Deeply nested state objects are hard to update immutably. Flatten them to simplify logic.

### State Preservation
*   **UI Tree Position:** React preserves state as long as the component is rendered at the same position in the UI tree. It is not tied to the JSX tag or function.
*   **Resetting State:** You can force a state reset by:
    *   Rendering a different component at the same position.
    *   Changing the component's `key` prop.
*   **Do Not Nest Definitions:** Never define a component function *inside* another component. This creates a new component reference on every render, forcing a remount and state loss for the child.

### Side Effects
*   **Event Handlers First:** Whenever possible, put side effects (API calls, logging) in event handlers. They run only when the user interacts.
*   **useEffect as Last Resort:** Use `useEffect` only when the side effect is caused by *rendering* itself (e.g., syncing with an external system) and cannot be handled by an event. It runs after the commit phase.

### Refs
*   **Generic Concept:** Refs are a generic concept, but most often you'll use them to hold DOM elements.
*   **Accessing DOM Nodes:** You instruct React to put a DOM node into `myRef.current` by passing `<div ref={myRef}>`.
*   **Non-Destructive Actions:** Usually, you will use refs for non-destructive actions like focusing, scrolling, or measuring DOM elements.
*   **Avoid DOM Mutations:** Avoid changing DOM nodes managed by React. If you do modify DOM nodes managed by React, modify parts that React has no reason to update.
*   **Synchronous Updates:** `flushSync` allows you to force React to synchronously flush updates, bypassing the normal batching behavior. Use sparingly, as it can impact performance.

```tsx
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

## Architecture: Trees
*   **Render Tree:** Represents the hierarchy of components actually rendered to the DOM. This is used to understand the application's UI structure and optimize rendering performance.
*   **Dependency Tree:** Represents the static module dependencies (imports). This is used by bundlers to tree-shake and optimize the build artifact.
