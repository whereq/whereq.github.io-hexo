---
title: Arrow Function Continues...
date: 2025-02-18 19:24:58
categories:
- TypeScript
- JavaScript 
tags:
- TypeScript
- JavaScript 
---

The difference between `onClick={() => handleClick('button clicked')}` and `onClick={handleClick}` lies in **how the function is called** and **what arguments are passed to it**. Let’s break it down:

---

### 1. **`onClick={handleClick}`**
- This directly assigns the `handleClick` function to the `onClick` event handler.
- **Behavior**: When the button is clicked, the `handleClick` function is called **without any arguments**.
- **Issue**: If `handleClick` expects an argument (e.g., `sectionType`), this will result in an error because no argument is passed.

#### Example:
```tsx
<button onClick={handleClick}>Click Me</button>
```
- If `handleClick` is defined as:
  ```tsx
  const handleClick = (sectionType: SectionType) => {
      console.log(sectionType);
  };
  ```
- **Result**: When the button is clicked, `sectionType` will be `undefined`, leading to unexpected behavior or errors.

---

### 2. **`onClick={() => handleClick('button clicked')}`**
- This uses an **arrow function** to create a new function that calls `handleClick` with the argument `'button clicked'`.
- **Behavior**: When the button is clicked, the arrow function is executed, which in turn calls `handleClick` with the specific argument (`'button clicked'`).
- **Why `() =>` is needed**: The arrow function acts as a **wrapper** that ensures `handleClick` is called with the correct argument when the button is clicked.

#### Example:
```tsx
<button onClick={() => handleClick('button clicked')}>Click Me</button>
```
- If `handleClick` is defined as:
  ```tsx
  const handleClick = (sectionType: SectionType) => {
      console.log(sectionType);
  };
  ```
- **Result**: When the button is clicked, `'button clicked'` is passed to `handleClick`, and the function works as expected.

---

### Why Do We Need `() =>` Here?
The `() =>` syntax is necessary because:
1. **Passing Arguments**:
   - If you want to pass an argument to `handleClick` (e.g., `'button clicked'`), you need to wrap it in an arrow function. This ensures the function is called with the correct argument when the event occurs.

2. **Avoiding Immediate Invocation**:
   - Without `() =>`, writing `onClick={handleClick('button clicked')}` would **immediately invoke** the function when the component renders, rather than when the button is clicked. This is not the desired behavior.

3. **Flexibility**:
   - The arrow function allows you to customize the behavior of the `onClick` handler. For example, you can pass different arguments or call multiple functions.

---

### Key Differences:
| **Syntax**                              | **Behavior**                                                                 | **When to Use**                                                                 |
|-----------------------------------------|-----------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| `onClick={handleClick}`         | Calls `handleClick` without arguments when the button is clicked.   | Use when the function does not require any arguments.                           |
| `onClick={() => handleClick('button clicked')}` | Calls `handleClick` with `'button clicked'` when the button is clicked. | Use when the function requires arguments or when you need to customize the call. |

---

### Practical Example:
#### Without Arguments:
If `handleClick` does not require any arguments:
```tsx
const handleClick = () => {
    console.log("Button clicked!");
};

<button onClick={handleClick}>Click Me</button>
```

#### With Arguments:
If `handleClick` requires an argument (e.g., `'button clicked'`):
```tsx
const handleClick = (sectionType: SectionType) => {
    console.log(sectionType);
};

<button onClick={() => handleClick('button clicked')}>Click Me</button>
```

---

### Summary:
- Use `onClick={handleClick}` when the function does not require any arguments.
- Use `onClick={() => handleClick('button clicked')}` when the function requires arguments or when you need to customize the function call.
- The `() =>` syntax ensures the function is called with the correct arguments **only when the button is clicked**, not during rendering.