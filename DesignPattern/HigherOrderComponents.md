# Day 05: Higher Order Components (HOC)

## 1. Introduction: The "Component Factory"
A **Higher Order Component (HOC)** is a pattern used for reusing component logic. It is not a part of the React API, per se, but a pattern that emerges from React’s compositional nature.

*   **Definition:** An HOC is a function that takes a component and returns a new, enhanced component.
*   **Analogy:** Think of it as a "Component Factory." You provide a "bare" component, the HOC adds "features" (like authentication, data fetching, or color changes), and returns a "shiny new version" of that component.

---

## 2. Foundation: Higher Order Functions (HOF)
To understand HOCs, you must understand **Higher Order Functions** in JavaScript.

*   **What is an HOF?** A function that does at least one of the following:
    1.  Takes one or more functions as arguments.
    2.  Returns a function as its result.
*   **Example (`withLogging`):**
    *   A function takes an `add` function.
    *   It returns a new function that logs "Calling function..." before executing the original `add` logic.
    *   **Result:** You get an enhanced version of the original function without modifying the original code.

---

## 3. How HOCs Work in React
The goal is to take **Input Components** and pass them through a **Reusable Logic Layer** to produce **Enhanced Components**.

### The Movie App Scenario (The "Messy" Way)
Imagine an app with a `MovieList` and a `MovieAnalytics` component.
*   **Redundancy:** Both components need to fetch the same data, handle "Loading" states, and handle "Error" states.
*   **Code Smell:** You end up copy-pasting the `useEffect` and `useState` fetching logic into every component that needs movie data.

### The HOC Solution (The "Pattern" Way)
Instead of repeating code, we create an HOC called `withDataFetching`.

1.  **Input:** Pass the `MovieList` component to `withDataFetching`.
2.  **Internal Logic:** The HOC manages `data`, `loading`, and `error` states. It performs the `fetch` call inside a `useEffect`.
3.  **Rendering:**
    *   If loading $\rightarrow$ Return "Loading..."
    *   If error $\rightarrow$ Return the error message.
    *   If success $\rightarrow$ Return the `MovieList` with the fetched `data` passed as a prop.

---

## 4. Implementation Details
### Naming Convention
It is a standard industry practice to prefix HOC names with **"with"** (e.g., `withDataFetching`, `withAuthentication`, `withTheme`). This immediately signals to other developers that the function is an HOC.

### Basic Structure
```javascript
function withDataFetching(WrappedComponent) {
  return function EnhancedComponent(props) {
    // 1. Add shared logic (state, effects)
    // 2. Conditional rendering (Loading/Error)
    // 3. Return the original component with extra props
    return <WrappedComponent data={data} {...props} />;
  };
}
```

---

## 5. Use Cases
HOCs are perfect for "cross-cutting concerns" (logic that many components need):
*   **Authentication/Authorization:** Checking if a user is logged in before showing a page.
*   **Permissions:** Checking if a user is an "Admin" or "Editor."
*   **Data Fetching:** Wrapping components that need to talk to the same API.
*   **Logging/Analytics:** Tracking when components mount or when buttons are clicked.
*   **Theme/Styling:** Injecting global theme properties into a component.

---

## 6. Pitfalls and Modern Alternatives

### Pitfalls
1.  **Composition Hell:** If you need to wrap a component in 5 HOCs (Auth, Data, Theme, Analytics, etc.), your code becomes very hard to debug and follow.
2.  **Prop Collisions:** If two different HOCs try to pass a prop with the same name (e.g., `data`), the second one will overwrite the first.
3.  **Static Methods:** Static methods on the original component are lost unless you manually copy them over to the enhanced component.

### The Alternative: Hooks
While HOCs are still widely found in **legacy codebases** and some libraries (like Redux's `connect`), **Custom Hooks** are the modern replacement. Hooks solve the logic-sharing problem without the "nesting" or "prop collision" issues of HOCs.

---

## 7. Task Assignment (Day 05)
**Objective:** Create a Permission-Based Access HOC.

1.  **The Setup:** Build three basic components:
    *   `ProfilePage` (accessible to all).
    *   `AdminPanel` (accessible only to users with the "Admin" role).
    *   `ReportsPage` (accessible only to users with "Report" permission).
2.  **The HOC:** Create `withUserDataAndPermissions`.
    *   It should simulate fetching a user object (e.g., `{ name: "Tapas", role: "Admin", permissions: ["reports"] }`).
    *   It should check if the wrapped component's requirements match the user's role/permissions.
    *   If permission is denied, render an "Access Denied" message.
3.  **Submission:** Post your solution on the Discord "task-assignments" channel.

---
*Follow the series on GitHub: `github.com/tapascript/15-days-of-react-design-patterns`*
