---
title: React Best Practices Guide - 2025
date: 2025-12-02 22:41:40
categories:
- ReactJS
tags:
- ReactJS
---

Below is a clean, concise **React Best Practices** document that covers:
✔ Avoiding unnecessary re-renders
✔ Comparing Redux vs React built-in state vs Zustand
✔ When to use `useMemo` and `useCallback`
✔ Real-world examples and patterns

---

# **React Best Practices Guide (Rendering, State Management, and Memoization)**

## **1. Avoiding Unnecessary Re-renders**

Unnecessary re-renders slow down your UI and degrade performance. The core rules to prevent them:

### **1.1 Keep State Local and Minimal**

* Only store what the component *needs* to render.
* Don’t store derived data in state (compute when needed).
* Lift state *only when necessary*.

✔ Good:

```jsx
const [count, setCount] = useState(0);
```

✘ Bad:

```jsx
const [double, setDouble] = useState(count * 2);  // derived → no need
```

---

### **1.2 Break Components into Smaller Memoized Pieces**

* Smaller components re-render less often.
* Apply `React.memo()` to memoize pure components.

```jsx
const UserRow = React.memo(({ user }) => {
  return <div>{user.name}</div>;
});
```

---

### **1.3 Avoid Recreating Objects/Arrays in Props**

Passing a *new object reference* triggers re-renders.

✘ Bad:

```jsx
<UserList filters={{ active: true }} />  // new object every time
```

✔ Good:

```jsx
const filters = useMemo(() => ({ active: true }), []);
<UserList filters={filters} />
```

---

### **1.4 Avoid Inline Functions When Passing to Children**

Inline functions cause new references → re-render.

✔ Solve with `useCallback`
(see section 4)

---

---

# **2. State Management Comparison**

## **2.1 React Built-in State (useState, useReducer, context)**

### ✔ Best for:

* Local UI state
* Small/medium apps
* Component-specific logic

### ✔ Pros:

* Simple
* No external dependency
* Fast, minimal overhead

### ✘ Cons:

* Context re-renders entire subtree
* Harder for large global state
* Not ideal for complex async logic

---

## **2.2 Redux Toolkit (RTK)**

### ✔ Best for:

* Large applications
* Complex global state workflows
* Predictable state transitions
* Time travel debugging
* Strict immutability, stable dev tooling

### ✔ Pros:

* Centralized, predictable state
* Excellent debugging/devtools
* Built-in async support (RTK Query)

### ✘ Cons:

* More boilerplate than alternatives (though RTK solved most)
* Heavy for small apps
* Requires good understanding of reducers, actions

---

## **2.3 Zustand**

A lightweight state manager that avoids context re-rendering.

### ✔ Best for:

* Medium/large apps needing global state
* Performance-sensitive apps
* Real-time or frequently updated data (e.g., WebGL apps, dashboards)

### ✔ Pros:

* Extremely simple API (`useStore`)
* No provider required
* Selectors prevent unnecessary re-renders
* Small footprint

```js
const useStore = create((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
}));
```

✔ Component re-renders only when the selected part changes:

```jsx
const count = useStore((s) => s.count); // selector = optimized re-render
```

### ✘ Cons:

* No built-in time travel/debugging like Redux
* Fewer guardrails for large teams

---

### **Summary Table**

| Feature           | React Built-in    | Redux Toolkit | Zustand     |
| ----------------- | ----------------- | ------------- | ----------- |
| App size          | Small–Medium      | Medium–Large  | Small–Large |
| Global state      | Limited (Context) | Excellent     | Excellent   |
| Performance       | Good              | Good          | Best        |
| Boilerplate       | Low               | Medium        | Very Low    |
| Devtools          | Basic             | Excellent     | Good        |
| Re-render control | Weak              | Medium        | Strong      |

---

---

# **3. `useMemo` and `useCallback` — When and Why**

## **3.1 `useMemo`**

Memoizes a **computed value**.

### ✔ Use when:

* Expensive calculations
* Stable object/array references needed
* Props require referential equality to avoid re-render

Example:

```jsx
const sortedUsers = useMemo(
  () => users.sort((a, b) => a.name.localeCompare(b.name)),
  [users]
);
```

---

## **3.2 `useCallback`**

Memoizes a **function reference**.

### ✔ Use when:

* Passing callbacks to memoized children
* Functions used in dependencies that should not change
* Avoiding new function reference on each render

Example:

```jsx
const handleClick = useCallback(() => {
  doSomething(value);
}, [value]);
```

---

## **3.3 Real-World Example Combining Both**

```jsx
const filters = useMemo(() => ({ active: true }), []);

const onSelect = useCallback((id) => {
  setSelected(id);
}, []);
```

---

### **When NOT to use them**

Overuse causes more harm than good.

❌ Don’t use them on trivial values/functions.
❌ Only optimize where profiling shows need.

---

---

# **4. Real-World Scenarios and Patterns**

## **4.1 Optimizing a Large Table**

**Use memoized rows + stable props**

```jsx
const Row = React.memo(({ item, onSelect }) => { … });

const onSelect = useCallback((id) => { … }, []);
```

---

## **4.2 Using Zustand for Real-Time WebSocket Updates**

```js
const useStore = create((set) => ({
  messages: [],
  addMessage: (m) => set((s) => ({ messages: [...s.messages, m] })),
}));
```

React components subscribe to slices:

```jsx
const messages = useStore((s) => s.messages);
```

Only re-renders on message changes—ideal for chat/dashboards.

---

## **4.3 Redux for Multi-page Enterprise UI**

* Form wizards
* Multi-step state
* Async API flows
* Devtools for debugging complex workflows

---

## **4.4 Heavy Calculation Memoization**

```jsx
const processed = useMemo(() => expensiveWork(data), [data]);
```

---

---

# **5. Final Recommendations**

### **If you want simplicity and speed** → **Zustand**

### **If you have huge global state and need devtools** → **Redux Toolkit**

### **If state is mostly local** → **React built-in hooks**

### **For performance tuning:**

* Split components
* Use memoization selectively
* Stabilize references with `useMemo` and `useCallback`
* Prefer selectors in Zustand or RTK Query

---

That's a great request. As a senior architect, here is a best practices document focusing on performance, state management, and memoization in React, designed for intermediate to expert developers.

-----

# 🚀 React Best Practices for Performance & State Management

This document outlines key strategies and architectural decisions for building highly performant and scalable React applications, focusing on avoiding unnecessary re-renders and selecting the optimal state management tool.

## 1\. Minimizing Unnecessary Re-Renders

The primary goal of React performance optimization is ensuring that components only render when their necessary inputs (props or state) actually change.

### A. The Golden Rule: Use `React.memo` Strategically

  * **Practice:** Always wrap presentational (or "dumb") components that receive props and do not manage their own state in **`React.memo`**.
  * **Mechanism:** `React.memo` is a **Higher-Order Component (HOC)** that performs a shallow comparison of the component's previous props and its new props. If the props are shallowly equal, React skips rendering the component.
  * **Caveat:** If a parent component passes a new **object** or **function reference** on every render, `React.memo` will fail the shallow comparison and the child component will still re-render. This leads directly to the need for memoization hooks (see Section 3).

### B. State Colocation (The "Lift State Down" Rule)

  * **Practice:** Manage state at the **lowest possible level** in the component tree.
  * **Rationale:** When state is defined using `useState` in a parent component, every time that state changes, the parent **and all its children** re-render. By moving state down to the component that actually uses it, you isolate the re-render scope, preventing large sections of the application from updating needlessly.

### C. Destructure Props Carefully

  * **Practice:** When passing large objects or arrays as props, only pass the specific properties the child component needs.
  * **Example:**
    ```javascript
    // ❌ Bad: Causes Re-render if any user property changes
    <UserAvatar user={largeUserObject} />

    // ✅ Good: Only causes Re-render if the name changes
    <UserAvatar userName={largeUserObject.name} />
    ```

-----

## 2\. State Management Comparison

Choosing the right state management tool depends on the **complexity** and **frequency** of your application's state updates.

| Feature | React Context + `useReducer` | Redux / Redux Toolkit (RTK) | Zustand / Jotai / Recoil |
| :--- | :--- | :--- | :--- |
| **Philosophy** | Built-in, simple state sharing for medium-sized apps. | Centralized, predictable state container. | Decentralized, atomic, and minimal global state. |
| **Re-render Scope** | **Propagates broadly.** Any component subscribing to the Context **re-renders** when *any* part of the Context value changes. | **Optimized.** Uses selectors and middleware (`reselect`) to ensure components only re-render when the *specific data* they select changes. | **Optimized.** Components only re-render when the specific **slice of state** they subscribe to changes, even if it's deeply nested. |
| **Boilerplate** | Low (only context provider/consumer needed). | High (actions, reducers, constants, thunks, store config). **RTK reduces this significantly.** | **Very Low.** Minimal boilerplate; functions like a global `useState`. |
| **Ideal Scenario** | Theming, internationalization, or infrequent global state (e.g., current user ID). | Large, complex applications requiring **data normalization**, middleware, complex debugging, and persistent history (time-travel debugging). | Micro-frontends, small/medium applications, or managing **high-frequency** state updates with maximum efficiency. |

### Architectural Recommendation: The Hybrid Approach

  * Use **Zustand** or **Jotai** for **High-Frequency Global State** (e.g., streaming chat data, animation state, or modal control) due to its minimal rendering overhead.
  * Use **React Context** for **Infrequent UI State** (e.g., theme, language settings).
  * Avoid a single massive **Redux** store unless the application's complexity explicitly requires centralized data normalization and persistence.

-----

## 3\. Deep Dive into Memoization Hooks

The following hooks are necessary to prevent the issues created by passing non-primitive values (objects, arrays, or functions) down to components wrapped in `React.memo`.

### A. `useMemo`

| Feature | Description |
| :--- | :--- |
| **Purpose** | **Memoizes a computed value.** It caches the result of a function and only re-runs the function when one of its dependencies changes. |
| **Syntax** | `const memoizedValue = useMemo(() => expensiveFunction(a, b), [a, b]);` |
| **Real-World Scenario** | **Heavy Computation:** Calculating a user's filtered transaction history, complex charting data, or deep filtering/sorting of a large list that takes time. |
| **Anti-Pattern** | Do not use `useMemo` to memoize simple operations (e.g., `a + b`). The overhead of the hook itself often outweighs the cost of the simple calculation. |

### B. `useCallback`

| Feature | Description |
| :--- | :--- |
| **Purpose** | **Memoizes a function instance.** It returns the same function reference across renders unless one of its dependencies changes. |
| **Syntax** | `const memoizedHandler = useCallback((e) => handler(e, a), [a]);` |
| **Real-World Scenario** | **Passing functions to `React.memo` Children:** Use this hook when passing event handlers (like `onClick`, `onChange`) down to child components that are wrapped in `React.memo`. This ensures the child component recognizes the handler as the *same* prop and avoids re-rendering. |
| **Anti-Pattern** | Do not use `useCallback` unless the function is being passed as a prop to a component that uses `React.memo` or is a dependency of another hook (like `useEffect`). |

### C. **The Relationship: The Memoization Cascade**

The three tools work together to manage the rendering cascade :

1.  The **Parent Component** renders.
2.  **`useMemo`** ensures complex objects/values remain the same.
3.  **`useCallback`** ensures function references remain the same.
4.  The child component, wrapped in **`React.memo`**, receives the *same* prop references as the last render.
5.  `React.memo` performs the shallow comparison and successfully **skips the re-render**.




# A fundamental question in optimizing React performance! The difference "before" and "after" using **`React.memo`**, **`useMemo`**, and **`useCallback`** is about **control over the rendering lifecycle** and preventing unnecessary work.

## 💡 The Core Concept: Referential Equality

Before diving into the hooks, it's essential to understand the underlying problem: In JavaScript, objects, arrays, and functions are **reference types**.

* When a parent component re-renders, any object or function created inside it (even if logically identical) is given a **new memory address** (a new reference).
* If this new reference is passed down as a prop to a child component, React perceives it as a **change**, forcing the child to re-render, even if the content hasn't changed.

The three memoization tools work to preserve those references, giving React the signal to skip unnecessary updates.

---

## 1. `React.memo` (Component Optimization)

`React.memo` is a Higher-Order Component (HOC) used to optimize **function components**.

| Feature | Before `React.memo` | After `React.memo` |
| :--- | :--- | :--- |
| **Rendering** | The component renders **every time** its parent renders, regardless of whether its props change. | The component performs a **shallow comparison** of the new props against the previous props. If the props are shallowly equal (same values for primitives, same references for objects/functions), the render is **skipped**. |
| **Use Case** | Component is simple or props change frequently. | Component is **"pure"** (output depends only on props) and the render logic is **expensive** or the component is **deeply nested** in the tree. |
| **Result** | Wasted CPU cycles, slow user interface (especially if there are many instances). | Reduced rendering cycles, faster updates, lower CPU usage. |

### Example

A child component wrapped in `React.memo` will *only* re-render if its props (e.g., `title`, `count`) are different from the last render.

---

## 2. `useMemo` (Value Optimization)

`useMemo` is a Hook used to cache the **result of an expensive computation** or a **reference-type value** (object/array).

| Feature | Before `useMemo` | After `useMemo` |
| :--- | :--- | :--- |
| **Value Creation** | The expensive function runs **on every render**. The array or object it returns is created with a **new reference** on every render. | The function runs **only when its dependency array changes**. The computed value and its reference are cached and reused on subsequent renders. |
| **Dependencies** | Not applicable; computation always runs. | The function is dependent on the values listed in the dependency array (`[a, b]`). |
| **Result** | High CPU usage and, more importantly, causes unnecessary re-renders in child components that receive the new object/array reference. | **Performance gain** (by skipping computation) and **stability** (by providing a consistent reference to children wrapped in `React.memo`). |

### Real-World Scenario

Calculating a complex derivative or filtering a list of 5,000 items. If the filtering criteria haven't changed, the list doesn't need to be re-filtered.

---

## 3. `useCallback` (Function Optimization)

`useCallback` is a Hook used to cache the **function instance itself**.

| Feature | Before `useCallback` | After `useCallback` |
| :--- | :--- | :--- |
| **Function Reference** | The function is redefined, generating a **new reference** on every render. | The same function instance and its memory address are returned on every render unless its dependency array changes. |
| **Dependencies** | Not applicable; function is always new. | The function is dependent on the values listed in the dependency array (`[c, d]`). |
| **Result** | Causes unnecessary re-renders in children wrapped in `React.memo` because the child sees a "new" function prop. | **Stability** by providing a consistent reference, allowing **`React.memo`** in the child component to work correctly. |

### Real-World Scenario

Passing an `onClick` or `onChange` handler to a highly optimized child button component. If the handler is wrapped in `useCallback`, the button component will not re-render unnecessarily when the parent renders.