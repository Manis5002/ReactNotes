# Day 04: The Render Props Pattern

## 1. Introduction: Why Learn Render Props?
In the modern React era (post-hooks), many developers rely solely on Custom Hooks to share logic. However, the **Render Props Pattern** remains a fundamental concept in React history and architecture.

### Why it matters today:
*   **Legacy Codebases:** Many large-scale applications built before React 16.8 (Hooks) heavily use this pattern.
*   **Migration & Maintenance:** You may be tasked with fixing or refactoring these patterns into modern hooks.
*   **Library Architecture:** Many popular libraries still use Render Props internally to provide maximum rendering flexibility to the user.
*   **Fundamental Logic Sharing:** It helps you understand how to separate **behavior (logic)** from **representation (UI)**.

---

## 2. The Problem: Logic Duplication (The "Messy" Way)
Imagine you need to track the mouse position for two different UI elements: a **Car** and a **Bike**.

### The Messy Approach:
1.  **CarTracker Component:** Contains state for `x` and `y`, a `handleMouseMove` function, and JSX that displays a Car at those coordinates.
2.  **BikeTracker Component:** Contains the **exact same** state and `handleMouseMove` logic, but displays a Bike icon.

### The Result:
*   **Code Smell:** You have duplicated the business logic (event listeners and state management) across multiple files.
*   **Maintenance Nightmare:** If you want to change how mouse tracking works (e.g., adding a throttle), you have to change it in every tracker component.

---

## 3. The Pattern Definition: "How" vs. "What"
The **Render Props Pattern** is a technique for sharing code between React components using a prop whose value is a **function**.

*   **The Logic Component (The "How"):** This component implements the behavior (e.g., tracking mouse movement). It doesn't know what it is rendering; it only knows how to calculate the data.
*   **The Consumer (The "What"):** This is the parent component that tells the Logic component what to display by passing a function that returns JSX.

> **Key Concept:** The Logic component provides the data, and the Consumer provides the UI.

---

## 4. Implementation Walkthrough

### Step 1: Create the Logic Component (`MouseTracker.jsx`)
This component handles the state and the event listeners. Instead of rendering a specific UI, it calls the `render` prop and passes the state to it.

```javascript
const MouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (event) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };

  return (
    <div style={{ height: '300px', border: '1px solid black' }} onMouseMove={handleMouseMove}>
      {/* 
         Calling the 'render' function and passing the state.
         The consumer will decide what the JSX looks like.
      */}
      {render(position)}
    </div>
  );
};
```

### Step 2: Use the Pattern (The Consumer)
The parent component now consumes the logic and decides the UI (Car vs. Bike) dynamically.

```javascript
// Example for a Car
<MouseTracker 
  render={(pos) => (
    <p>Car is at {pos.x}, {pos.y}</p>
  )} 
/>

// Example for a Bike
<MouseTracker 
  render={({ x, y }) => (
    <p>Bike is at {x}, {y}</p>
  )} 
/>
```

---

## 5. Variation: The `children` Prop
You can implement the same pattern using the special `children` prop. This often makes the code more readable and "React-like."

### The Logic Component:
```javascript
const MouseTracker = ({ children }) => {
  // ... state and logic ...
  return (
    <div onMouseMove={handleMouseMove}>
      {children(position)}
    </div>
  );
};
```

### The Consumer:
```javascript
<MouseTracker>
  {({ x, y }) => (
    <p>The pointer is at {x}, {y}</p>
  )}
</MouseTracker>
```

---

## 6. Use Cases & Pitfalls

### Ideal Use Cases:
*   **Logic Sharing:** Tracking mouse position, window resizing, or scroll position.
*   **State Wrappers:** Handling "Show/Hide" toggle logic or "Data Fetching" states.
*   **Reusable Libraries:** Building UI libraries where the user needs to provide their own custom markup for internal states.

### Pitfalls & Anti-Patterns:
1.  **Performance:** Passing an inline function to a render prop creates a new function on every render. While usually fine, it can lead to unnecessary re-renders in deep component trees.
2.  **"Render Prop Hell":** Just like "Callback Hell," nesting multiple render props can make JSX very difficult to read.
3.  **HOC vs. Render Props:** Avoid mixing high-complexity patterns like HOCs (Higher Order Components) and Render Props together, as it complicates debugging.

---

## 7. Modern Alternative: Custom Hooks
Today, the most common way to solve this is a Custom Hook (e.g., `useMousePosition`). 
*   **Render Props:** Better when the logic is tied to a specific **DOM element** (like the `div` wrapper in our tracker).
*   **Hooks:** Better for **global** or **non-visual** logic (like window size).

---

## 8. Task Assignment (Day 04)
**Objective:** Build a Reusable Toggle Component.

1.  **The Component:** Create a `Toggle` component that manages an internal `isOpened` (Boolean) state.
2.  **The Logic:** Implement a function inside `Toggle` to switch the state between true and false.
3.  **The Pattern:** Use the **Render Props** pattern (or `children` prop) to allow the parent to decide:
    *   What the "Open" UI looks like (e.g., a "Close" button and some text).
    *   What the "Closed" UI looks like (e.g., an "Open" button).
4.  **Submission:** Share your GitHub link and thoughts on the readability of this pattern compared to standard props in the Discord community.

---
*Follow the series on GitHub: `github.com/tapascript/15-days-of-react-design-patterns`*
