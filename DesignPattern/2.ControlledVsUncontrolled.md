# Day 02: Controlled vs. Uncontrolled Components

## 1. The Foundation: State vs. Refs
To master React forms, one must first understand the two primary ways React stores and manages data within a component.

### A. React State (`useState`)
*   **Purpose:** Manages dynamic data that drives the UI.
*   **Source of Truth:** It is the single source of truth for the component.
*   **Behavior:** **Reactive.** When state changes, React triggers a **re-render** to reflect the new data in the UI.
*   **Usage:** Best for values that need to be synced with the UI (e.g., input values, counters).

### B. React Refs (`useRef`)
*   **Purpose:** Accesses/manipulates DOM elements directly or persists values.
*   **Behavior:** **Non-Reactive.** Updating a ref’s `.current` property **does NOT trigger a re-render.**
*   **Persistence:** The value inside a ref stays the same (persists) even when the component re-renders due to other state changes.
*   **Usage:** Best for DOM interactions (focusing an input, measuring a div) or storing data that shouldn't affect the UI layout.

---

## 2. The "Messy" Pattern: Mixing State & Refs Incorrectly
A common beginner mistake is managing some form values with `state` and others with `refs`. 
*   **The Problem:** Inconsistent behavior. Validating, resetting, or testing the form becomes a nightmare because the data is stored in two different "brains" (one that re-renders and one that doesn't).
*   **Example:** Using `useState` for Name/Email but `useRef` for the Message value.

---

## 3. The Controlled Component Pattern
In a controlled component, the form element's value is driven by React state.

### Key Characteristics:
1.  **State Logic:** The value is held in `useState`.
2.  **Syncing:** The `value` attribute of the input is set to the state variable.
3.  **Updating:** An `onChange` handler updates the state as the user types.

### Best Practice: The "Object State" Pattern
Instead of creating separate state variables for every field (Name, Email, Message), manage them in a single object:
```javascript
const [form, setForm] = useState({ name: '', email: '', message: '' });

const handleChange = (e) => {
  const { name, value } = e.target;
  setForm({ ...form, [name]: value }); // Dynamic key updating
};
```
*Note: Ensure your JSX inputs have a `name` attribute that matches the object keys.*

### When to Mix State and Refs (The Right Way):
You may use **Refs alongside State** when you need to handle **DOM behavior** while state handles **data**.
*   **State:** Check if `form.name` is empty.
*   **Ref:** If empty, use `nameRef.current.focus()` to put the cursor in the box.

---

## 4. The Uncontrolled Component Pattern
In an uncontrolled component, the DOM itself handles the form data. React simply "pulls" the value when needed.

### Two Approaches:
1.  **The Ref Way:** Use `useRef` to grab the value from the DOM node only at the time of submission (e.g., `nameRef.current.value`).
2.  **The Native Way (FormData):** Use the standard HTML/JavaScript `FormData` API inside the `onSubmit` handler.
    *   *Benefit:* No `useState` or `useRef` hooks are needed. It’s pure HTML/JS logic.

---

## 5. Comparison: When to Use Which?

| Feature | Controlled Component | Uncontrolled Component |
| :--- | :--- | :--- |
| **Validation** | Real-time (instant feedback) | On-submit only |
| **Data Flow** | Predictable & Synchronized | Direct from DOM |
| **Complexity** | More boilerplate (handlers) | Simple/Clean (standard HTML) |
| **Performance** | Re-renders on every keystroke | No re-renders until submit |
| **Best For** | Complex forms, dynamic inputs, Redux/Global state | Simple logins, newsletter signups, quick MVPs |

---

## 6. Pitfalls and Anti-Patterns
1.  **Unnecessary Mixing:** Avoid managing "Value A" with state and "Value B" with refs. This causes "split-brain" bugs.
2.  **Over-engineering:** Don't use controlled components for a tiny, static one-field newsletter form. It adds unnecessary boilerplate.
3.  **Maintenance Nightmare:** Don't use uncontrolled components for large forms (10+ fields). It makes testing and complex validation very difficult.

---

## 7. Looking Ahead: React 19 Improvements
React 19 introduces new hooks that simplify form handling even further:
*   **`useFormStatus`:** Allows sub-components to know if a form is currently pending/submitting.
*   **`useActionState`:** (formerly `useFormState`) Manages state based on the result of a form action (perfect for server-side interactions).

---

## 8. Task Assignment (Day 02)
**Objective:** Build a Contact Form (First Name, Last Name, Email, Phone, Hobbies).

1.  **Implement as Controlled:** Use a single state object and implement real-time validation.
2.  **Implement as Uncontrolled:**
    *   Create one version using `useRef`.
    *   Create another version using the native `FormData` API (no hooks).
3.  **Comparison Table:** Write a `task.md` file comparing the readability, reset logic, and complexity of all three approaches.

---
*Follow the series on GitHub: `github.com/tapascript/15-days-of-react-design-patterns`*
