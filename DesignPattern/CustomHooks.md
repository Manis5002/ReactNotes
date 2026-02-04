# Day 06: The Custom Hooks Pattern

## 1. Introduction: Saying Goodbye to "Wrapper Hell"
In previous sessions, we explored Render Props and Higher Order Components (HOCs). While effective, these patterns often lead to "Wrapper Hell"—a complex hierarchy of nested components that makes debugging difficult. The **Custom Hooks Pattern** is the modern React solution to this problem.

*   **The Goal:** Extract stateful logic from components into reusable functions.
*   **The Result:** Cleaner code, easier testing, and a better developer experience.

---

## 2. Why Hooks? (The Evolution)
Before React 16.8 (the "Pre-Hook Era"), sharing logic was limited to HOCs and Render Props. These caused several issues:
*   **Prop Drilling:** Passing data through layers of components that don't need it.
*   **Complex Hierarchies:** Too many wrappers making the DOM tree hard to read.
*   **Maintenance:** Difficulty in tracking where a specific piece of logic or prop originated.

**Hooks** changed this by allowing functional components to "hook into" React features (state, effects, context) without the need for complex nesting.

---

## 3. What exactly is a Hook?
A hook is a special JavaScript function that is **"React-Aware."**

### Hook vs. Utility Function
*   **Utility Function:** A plain JS function that performs a task (e.g., calculating a date). It doesn't know about React's lifecycle or state.
*   **Hook:** A function that can manage React state, trigger side effects, and access context. It "hooks" into the React Fiber architecture.

---

## 4. The Rules of Hooks
To ensure React correctly matches hooks with their internal state, you must follow two non-negotiable rules:

1.  **Call Hooks at the Top Level Only:** Never call hooks inside loops, conditions, or nested functions. React relies on the **order** in which hooks are called during every render.
2.  **Only Call Hooks from React Functions:** Call them from React functional components or other custom hooks. Never call them from regular JavaScript utility functions.

---

## 5. Built-in Hooks Overview
React provides several hooks out of the box. Key ones include:
*   **`useState`:** Manages local state.
*   **`useEffect`:** Handles side effects (API calls, event listeners).
*   **`useRef`:** Directly accesses DOM elements.
*   **`useContext`:** Accesses shared data (Global state).
*   **`useReducer`:** Handles complex state transitions.
*   **React 19 Additions:** `useActionState`, `useFormStatus`, `useOptimistic`.

---

## 6. The Custom Hook Pattern
A custom hook is a function whose name **must start with "use"** (e.g., `useFetch`, `useAuth`).

### Implementation Example: `useToggle`
Instead of writing toggle logic (true/false) in every component, we extract it:

**The Hook (`useToggle.js`):**
```javascript
export const useToggle = (initialValue = false) => {
  const [value, setValue] = useState(initialValue);
  const toggle = () => setValue(prev => !prev);
  return [value, toggle]; // Returning an array for easy destructuring
};
```

**The Consumer (`ThemeSwitcher.jsx`):**
```javascript
const [isDark, toggleTheme] = useToggle(false);
return (
  <button onClick={toggleTheme}>
    {isDark ? "Dark Mode" : "Light Mode"}
  </button>
);
```

---

## 7. Practical Use Case Ideas
*   **`useLocalStorage`:** Syncs state with the browser’s storage automatically.
*   **`useDebounce`:** Delays execution of logic (useful for search bars).
*   **`useOnlineStatus`:** Detects if the user's internet is active or offline.
*   **`useClipboard`:** Handles "Copy to Clipboard" functionality.
*   **`useMediaQuery`:** Detects screen width for responsive JS logic.
*   **`useClickOutside`:** Detects clicks outside a specific element (ideal for closing modals).

---

## 8. Pitfalls & Anti-Patterns
1.  **Over-Engineering:** Don't create a hook for logic that is used in only one place. Hooks are for **reusability**.
2.  **Unnecessary Re-renders:** Be careful with the values your hook returns. Use `useMemo` or `useCallback` if the returned functions are causing downstream components to re-render.
3.  **Missing Cleanups:** If your hook adds an event listener (like `scroll` or `resize`), always remove it in the `useEffect` cleanup function to prevent memory leaks.
4.  **Dependency Management:** Always keep your `useEffect` dependency arrays honest and clean to avoid stale closures or infinite loops.

---

## 9. Task Assignment (Day 06)
**Objective:** Build four real-world custom hooks.

1.  **`useLocalStorage`:** A hook that works like `useState` but persists the value in `localStorage`.
2.  **`useClipboard`:** A hook with a `copy` function that takes text and saves it to the user's clipboard.
3.  **`useFetch`:** A generic data fetching hook that handles `data`, `loading`, and `error` states.
4.  **`useClickOutside`:** A hook that takes a `ref` and a `callback`, triggering the callback when the user clicks anywhere else on the page.

---
*Follow the series on GitHub: `github.com/tapascript/15-days-of-react-design-patterns`*
