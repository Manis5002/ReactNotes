# Day 03: The Compound Components Pattern

## 1. The Core Problem: "Prop Soup"
When building complex UI components (like Modals, Tabs, or Accordions), developers often create a single "smart" component that receives a massive amount of props to handle every possible variation.

### Characteristics of the "Messy" Way:
*   **Rigid Structure:** The component dictates the layout (e.g., Title must be an `H2`, Body must be a `P`).
*   **Lack of Flexibility:** Adding a third button or an image instead of text requires adding even more props (e.g., `hasImage`, `thirdActionText`).
*   **Mixed Responsibilities:** The component tries to handle layout, content, and logic all at once, violating the **Separation of Concerns**.
*   **Poor Scalability:** As new requirements emerge, the component becomes a "monster" with 15-20 props, making it hard to maintain and test.

---

## 2. The Solution: Compound Components Pattern
The **Compound Components Pattern** allows you to build a component as a set of "Lego blocks." Instead of one giant component, you create several smaller, specialized components that work together to form a larger unit.

### The "Lego" Analogy:
*   You have a parent "shell" (the Modal).
*   You have specialized bricks (Header, Body, Footer).
*   The user of the component decides how to assemble these bricks.

### Key Benefits:
1.  **Flexibility:** The user can swap an `H2` for an `H1` or add a Table inside a Modal Body without changing the Modal's source code.
2.  **Explicit Composition:** The structure of the UI is clear in the JSX (e.g., you see `<Modal.Header>` followed by `<Modal.Body>`).
3.  **Reduced Props:** Most configuration is handled by children elements, not top-level props.

---

## 3. Implementation Guide
The pattern relies on attaching sub-components to a parent component using object properties.

### Step 1: Create the Sub-components
These are simple components that accept `children` and apply specific styles.
```javascript
const ModalHeader = ({ children }) => <div className="modal-header">{children}</div>;
const ModalBody = ({ children }) => <div className="modal-body">{children}</div>;
const ModalFooter = ({ children }) => <div className="modal-footer">{children}</div>;
```

### Step 2: Create the Parent Shell
The parent manages the main wrapper (like a backdrop) and simply renders `children`.
```javascript
const Modal = ({ children, isOpen, onClose }) => {
  if (!isOpen) return null;
  return (
    <div className="modal-backdrop">
      <div className="modal-container">
        <button onClick={onClose}>&times;</button>
        {children}
      </div>
    </div>
  );
};
```

### Step 3: The "Magic" Link
Attach the sub-components to the parent so they can be accessed via dot notation.
```javascript
Modal.Header = ModalHeader;
Modal.Body = ModalBody;
Modal.Footer = ModalFooter;

export default Modal;
```

### Step 4: Using the Pattern
The user now has full control over the composition:
```javascript
<Modal isOpen={true} onClose={handleClose}>
  <Modal.Header>
    <h1>Custom Title</h1>
  </Modal.Header>
  <Modal.Body>
    <p>This can be any element!</p>
    <img src="path/to/img" />
  </Modal.Body>
  <Modal.Footer>
    <button>Cancel</button>
    <button>Save Changes</button>
  </Modal.Footer>
</Modal>
```

---

## 4. Architectural Decisions: One File or Many?
*   **The "Stay Together" Rule:** If sub-components are tiny and only make sense inside the parent (e.g., `Modal.Header`), keep them in the same file as the parent.
*   **The "Split" Rule:** If a sub-component grows complex, has its own state, or could be reused elsewhere, move it to its own file.
*   **Discovery:** Keeping them together makes the components easier for other developers to find and use.

---

## 5. Use Cases
Use this pattern whenever **layout and nesting matter**:
*   **Tabs:** `<Tabs>`, `<Tabs.List>`, `<Tabs.Trigger>`, `<Tabs.Content>`.
*   **Accordions:** `<Accordion>`, `<Accordion.Item>`, `<Accordion.Title>`, `<Accordion.Content>`.
*   **Tables:** `<Table>`, `<Table.Header>`, `<Table.Row>`, `<Table.Cell>`.
*   **Menus/Dropdowns:** `<Select>`, `<Select.Option>`.

---

## 6. Pitfalls and Anti-Patterns
1.  **Semantic Mismatch:** Don't attach components that aren't related (e.g., don't attach a `Sidebar` to a `Modal`).
2.  **Re-exporting Separately:** Do not export `ModalHeader` as a standalone component. Export only the `Modal`. This prevents users from using the header without the modal shell, which could break styles or logic.
3.  **Over-Engineering:** If a component truly only ever needs a title and a description, a simple prop-based component is fine. Use this pattern for components that require high customization.

---

## 7. Task Assignment (Day 03)
**Objective:** Build a flexible Card component and a Tab component.

1.  **The Card Task:**
    *   Create a `<Card>` component.
    *   Implement sub-components: `Card.Header`, `Card.Body`, `Card.Footer`, and `Card.Image`.
    *   Demonstrate that you can move the image from the top to the bottom of the card without changing the internal code of the Card component.
2.  **The Tabs Task:**
    *   Implement a simple Tabs component using this pattern to manage switching between content sections.

---
*Follow the series on GitHub: `github.com/tapascript/15-days-of-react-design-patterns`*
