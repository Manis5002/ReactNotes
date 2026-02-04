# Day 01: The Container-Presenter Pattern
**Also known as:** Smart and Dumb Components Pattern.

## 1. Introduction & The Problem
In early React development, components often grow into "monsters." A single component might handle business logic, API calls, state management, and complex rendering all at once. This leads to **Code Smells**.

### Identified Code Smells (The "Messy" Way)
1.  **Violation of SRP (Single Responsibility Principle):** One component does everything (fetching, state, UI, error handling).
2.  **Low Reusability:** Logic (like loading spinners or error messages) is hard-coded and cannot be shared with other parts of the app.
3.  **Poor Testability:** Testing requires mocking Axios, multiple API endpoints, and complex state transitions simultaneously.
4.  **Difficult Maintenance:** Any change to business logic risks breaking the UI, and vice versa.

---

## 2. The Pattern Definition
The **Container-Presenter Pattern** splits a component into two distinct roles:

### A. The Container Component (The "Smart" One)
*   **Responsibility:** *How things work.*
*   **Duties:**
    *   Fetching data (API calls).
    *   Managing state (using `useState`, `useEffect`).
    *   Handling business logic.
    *   Passing data and callbacks to the Presenter via **props**.
*   **UI:** Usually contains no (or very little) markup/CSS of its own.

### B. The Presenter Component (The "Dumb" One)
*   **Responsibility:** *How things look.*
*   **Duties:**
    *   Receiving data through props.
    *   Rendering JSX/HTML/CSS.
    *   Triggering callbacks passed from the container (e.g., `onClick`).
*   **Logic:** It should not know where the data comes from or how to fetch it.

---

## 3. Implementation Walkthrough
The video demonstrates refactoring a monolithic `UserProfile` component.

### Step 1: Create the Container (`UserContainer.jsx`)
Move all `axios` calls, `useEffect` hooks, and primary states (`user`, `posts`, `loading`, `error`) here.
*   **Key Logic:** It fetches the user ID, gets data, and sets the state.
*   **Rendering:** It returns the Presenter component, passing the state as props.

### Step 2: Create the Presenter (`UserPresenter.jsx`)
Accepts `user`, `posts`, `loading`, and `error` as props.
*   **Refinement:** To avoid a "giant" presenter, break it down further into smaller sub-presenters:
    *   `ProfileHeader`: Shows user bio and photo.
    *   `PostList`: Iterates through the posts array.
    *   `Common Components`: Create reusable `LoadingSpinner` and `ErrorMessage` components.

### Step 3: Handling Actions (Callbacks)
When a user interacts with the UI (e.g., clicking "Save" or "Retry"):
1.  The **Presenter** triggers a prop function (e.g., `onRetry`).
2.  The **Container** defines the actual logic in a handler (e.g., `handleRetry`).
3.  **Naming Convention:**
    *   Props: Start with `on` (e.g., `onSaveProfile`).
    *   Internal Functions: Start with `handle` (e.g., `handleSaveProfile`).

---

## 4. Where to Use & Best Practices

### Use Cases
*   **Data-Heavy Components:** User dashboards, product catalogs, analytics reports.
*   **Form-Heavy Applications:** Multi-step wizards, checkout processes, and complex validations.

### Pitfalls to Avoid (Anti-Patterns)
*   **Overengineering:** Do not use this for tiny, simple components that don't fetch data or manage complex state.
*   **Prop Drilling:** If you find yourself passing props through 4-5 layers, the Container-Presenter pattern might not be enough. In such cases, consider the **Context API** or **State Reducer Pattern**.

---

## 5. Summary Table

| Feature | Container (Smart) | Presenter (Dumb) |
| :--- | :--- | :--- |
| **Logic** | Business logic & Data fetching | UI logic only |
| **State** | Managing data state | Managing UI state (e.g., "isEditing") |
| **Data Source** | API, Redux, Context | Props |
| **Styling** | Little to none | Primary focus (CSS/Styles) |

---

## 6. Practical Task (Homework)
**Objective:** Build a Product List using the pattern.

1.  **ProductListContainer:**
    *   Fetch product data from an API.
    *   Manage cart state.
    *   Handle errors and retries.
2.  **ProductListPresenter:**
    *   Receive products as props.
    *   Render the list.
    *   Contain sub-components like `ProductCard` and `SortControls`.
3.  **Validation:**
    *   Ensure the Container has **no** rendering logic (no UI tags).
    *   Ensure the Presenter has **no** API calls.

---

## 7. Useful Links (Mentioned in Video)
*   **Prerequisites:** Familiarity with JSX, Props, `useState`, and `useEffect`.
*   **Fake API Repo:** `github.com/tapascript/fakey-apis`
*   **Notion Template:** Available in the video description for tracking learning.
*   **Discord Community:** For task submission and peer review.
