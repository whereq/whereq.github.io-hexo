---
title: Deep Dive into the  Hook in React
date: 2025-01-24 12:22:32
categories:
- ReactJS
- TypeScript
tags:
- ReactJS
- TypeScript
---

## What is `useRef`?

`useRef` is a hook provided by React that allows you to:

1. **Persist a mutable value across renders without triggering re-renders.**
2. **Access a DOM element directly.**
3. **Store values similar to an instance variable in class components.**

### Syntax

```javascript
const refContainer = useRef(initialValue);
```

- `refContainer` is an object with a `current` property.
- `initialValue` is the initial value assigned to `current`.

Textual diagram of a `useRef` object:

```
useRef
   |
   |--- current: <initialValue>
```

---

## Scenarios and Use Cases

### 1. **Accessing DOM Elements**

`useRef` is commonly used to directly access DOM elements. In class components, this was done using `React.createRef()`.

#### Example:

```javascript
import React, { useRef } from 'react';

const FocusInput = () => {
  const inputRef = useRef(null);

  const handleFocus = () => {
    if (inputRef.current) {
      inputRef.current.focus(); // Directly focus the input element
    }
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Click the button to focus" />
      <button onClick={handleFocus}>Focus Input</button>
    </div>
  );
};

export default FocusInput;
```

**Sequence Diagram:**

```
User clicks the "Focus Input" button
|
|--- `handleFocus` is invoked
     |
     |--- `inputRef.current.focus()` focuses the input element
```

### 2. **Persisting Values Without Re-Renders**

Unlike `useState`, updating a `useRef` value does not trigger a re-render.

#### Example:

```javascript
import React, { useRef, useState } from 'react';

const Counter = () => {
  const renderCount = useRef(0);
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(count + 1);
  };

  renderCount.current += 1;

  return (
    <div>
      <p>Count: {count}</p>
      <p>Component rendered {renderCount.current} times</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};

export default Counter;
```

**Textual Diagram of State vs Ref:**

```
useState (Triggers Re-render):
   state = count: 0 -> render -> state = count: 1 -> render

useRef (No Re-render):
   ref = renderCount: 1 -> render -> ref = renderCount: 2 -> no re-render
```

### 3. **Storing Mutable Values**

When working with mutable values, `useRef` can store data that changes over time without impacting the component lifecycle.

#### Example:

```javascript
import React, { useRef } from 'react';

const Stopwatch = () => {
  const timerId = useRef(null);
  const startTime = useRef(null);

  const startTimer = () => {
    startTime.current = Date.now();
    timerId.current = setInterval(() => {
      console.log("Elapsed time:", Date.now() - startTime.current);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerId.current);
    timerId.current = null;
  };

  return (
    <div>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
};

export default Stopwatch;
```

**Sequence Diagram:**

```
User clicks "Start"
|
|--- `startTimer` invoked
     |
     |--- `startTime.current` set to current timestamp
     |--- `timerId.current` stores interval ID

User clicks "Stop"
|
|--- `stopTimer` invoked
     |
     |--- `clearInterval(timerId.current)` stops the timer
```

---

### 4. **Avoiding Closure Pitfalls**

In functional components, closures can lead to stale values being captured. `useRef` can prevent this by providing a mutable reference.

#### Example:

```javascript
import React, { useState, useRef } from 'react';

const CounterWithRef = () => {
  const countRef = useRef(0);
  const [count, setCount] = useState(0);

  const increment = () => {
    countRef.current += 1; // Updates ref value
    setCount(countRef.current); // Updates state
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};

export default CounterWithRef;
```

---

## When to Use `useRef`

- Accessing DOM elements.
- Storing mutable values that persist across renders.
- Avoiding re-renders caused by state updates.
- Managing timers, intervals, or external libraries.

---

## Common Pitfalls

### 1. **Overusing `useRef`**
Avoid using `useRef` for state that should trigger a re-render. Use `useState` or `useReducer` instead.

### 2. **Directly Manipulating DOM**
While `useRef` can access DOM elements, it’s generally better to rely on React’s declarative approach.

---

## Conclusion

The `useRef` hook is versatile and indispensable for managing references and mutable values. By understanding its use cases and limitations, you can write cleaner, more efficient React components.

---

Here's an updated article snippet that includes the scenario where `useRef` is used with `useEffect` to ensure that changes to the referenced value do not trigger `useEffect`, but the effect can still access the latest value of the reference.

---

### Using `useRef` with `useEffect` to Access the Latest Value Without Re-triggering

One of the powerful features of `useRef` is that it provides a mutable object that does **not trigger re-renders** when its value changes. This characteristic makes it an excellent tool for scenarios where you need to access the latest value inside a `useEffect` but do not want the `useEffect` to re-run whenever the value changes.

#### Why Use `useRef` in This Scenario?
When a state variable is updated, it triggers a re-render, and the `useEffect` with that state as a dependency is re-executed. However, in some cases, you may want to:

1. Access the latest value inside the `useEffect`.
2. Avoid re-triggering the `useEffect` when the value changes.

This is where `useRef` comes in handy.

---

#### Code Example: Accessing the Latest Value Without Re-triggering `useEffect`

```tsx
import React, { useState, useEffect, useRef } from "react";

const TimerComponent = () => {
  const [isRunning, setIsRunning] = useState(false); // Controls if the timer is running
  const countRef = useRef(0); // Tracks the count value
  const intervalRef = useRef<NodeJS.Timer | null>(null); // Tracks the interval ID

  // Start the timer
  const startTimer = () => {
    if (!intervalRef.current) {
      intervalRef.current = setInterval(() => {
        countRef.current += 1; // Update the countRef without causing a re-render
        console.log("Current count:", countRef.current); // Always prints the latest count
      }, 1000);
      setIsRunning(true);
    }
  };

  // Stop the timer
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
      setIsRunning(false);
    }
  };

  // Example: Access the latest value inside useEffect without adding dependencies
  useEffect(() => {
    const logLatestCount = () => {
      console.log("Timer stopped. Latest count value:", countRef.current);
    };

    if (!isRunning) {
      logLatestCount(); // Logs the latest count value without needing countRef as a dependency
    }

    return () => {
      if (intervalRef.current) clearInterval(intervalRef.current); // Cleanup
    };
  }, [isRunning]); // Dependency only on isRunning, but we can still access countRef.current

  return (
    <div>
      <h1>Count: {countRef.current}</h1> {/* Ref does not trigger re-renders */}
      <button onClick={startTimer} disabled={isRunning}>
        Start
      </button>
      <button onClick={stopTimer} disabled={!isRunning}>
        Stop
      </button>
    </div>
  );
};

export default TimerComponent;
```

---

#### Sequence Breakdown

1. **Initial Render:**
   - The `countRef` is initialized to `0`, and the `intervalRef` is `null`.

2. **Start Timer (`startTimer`):**
   - Sets up a timer with `setInterval`.
   - Updates `countRef.current` every second without causing re-renders.
   - `isRunning` is updated, which triggers a re-render to disable the "Start" button.

3. **Stop Timer (`stopTimer`):**
   - Clears the interval and resets `intervalRef` to `null`.
   - Logs the latest value of `countRef.current` using `useEffect` because `isRunning` changes.

4. **Access Latest Value in `useEffect`:**
   - Even though `countRef` is not a dependency, `useEffect` can access its latest value because `useRef` does not reinitialize between renders.

---

#### Advantages of Using `useRef` in This Scenario

- **Avoiding Re-renders:** Unlike `useState`, updates to `useRef` do not trigger re-renders, making it perfect for frequently updated values like a timer count.
- **Accessing Latest Value in Effects:** The latest value in `useRef` is always accessible, even in effects, without adding it as a dependency.

---

#### Textual Diagram: How `useRef` and `useEffect` Work Together

```
[Initial Render]
  |
  |--> useRef initialized: countRef = { current: 0 }
  |
[Timer Starts (setInterval)]
  |
  |--> countRef.current updated every second
  |    (NO re-renders triggered)
  |
[useEffect Triggered on isRunning Change]
  |
  |--> Access countRef.current to log the latest value
  |    (No dependency needed for countRef)
  |
[Timer Stops]
  |
  |--> useEffect logs latest countRef.current value
```

