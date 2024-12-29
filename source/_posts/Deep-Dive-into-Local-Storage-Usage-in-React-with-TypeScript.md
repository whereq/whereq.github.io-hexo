---
title: Deep Dive into Local Storage Usage in React with TypeScript
date: 2024-12-27 16:00:44
categories:
- Deep Dive
- React
- TypeScript
tags:
- Deep Dive
- React 
- TypeScript
---

Local storage is a simple yet powerful tool for persisting data in web applications. In this article, we'll explore every detail of local storage usage in React with TypeScript, including best practices, sample code, and test cases. The illustrations are based on the latest versions of React (18.x) and TypeScript (5.x).

---

## **What is Local Storage?**

Local storage is a web API that allows developers to store key-value pairs in a web browser. The data persists even after the browser is closed, making it ideal for saving user preferences, authentication tokens, and other non-sensitive data.

### Key Characteristics of Local Storage:
- **Synchronous API**: Operations are blocking.
- **Storage Limit**: Approximately 5MB per domain.
- **Data Lifetime**: Persists until explicitly deleted.
- **Data Scope**: Accessible only within the same origin.

---

## **Using Local Storage in React with TypeScript**

### 1. **Basic CRUD Operations**

Local storage provides three core methods:
- `localStorage.setItem(key, value)`
- `localStorage.getItem(key)`
- `localStorage.removeItem(key)`

### Example: Wrapper Utility for Local Storage

We can create a type-safe utility to handle common operations:

```typescript
// utils/localStorage.ts
export class LocalStorageHelper {
    static setItem<T>(key: string, value: T): void {
        localStorage.setItem(key, JSON.stringify(value));
    }

    static getItem<T>(key: string): T | null {
        const value = localStorage.getItem(key);
        return value ? JSON.parse(value) as T : null;
    }

    static removeItem(key: string): void {
        localStorage.removeItem(key);
    }
}
```

### Test Cases

Using Jest, we can write unit tests to validate the utility:

```typescript
// utils/localStorage.test.ts
import { LocalStorageHelper } from "./localStorage";

describe("LocalStorageHelper", () => {
    beforeEach(() => {
        localStorage.clear();
    });

    it("should set and retrieve an item", () => {
        LocalStorageHelper.setItem("key", { name: "John" });
        const item = LocalStorageHelper.getItem<{ name: string }>("key");
        expect(item).toEqual({ name: "John" });
    });

    it("should return null for non-existent keys", () => {
        const item = LocalStorageHelper.getItem("nonexistent");
        expect(item).toBeNull();
    });

    it("should remove an item", () => {
        LocalStorageHelper.setItem("key", "value");
        LocalStorageHelper.removeItem("key");
        const item = LocalStorageHelper.getItem("key");
        expect(item).toBeNull();
    });
});
```

---

### 2. **Integrating Local Storage in React Components**

#### Example: Using Local Storage with `useState`

```typescript
// components/Counter.tsx
import React, { useState, useEffect } from "react";
import { LocalStorageHelper } from "../utils/localStorage";

const Counter: React.FC = () => {
    const [count, setCount] = useState<number>(() => {
        return LocalStorageHelper.getItem<number>("count") || 0;
    });

    useEffect(() => {
        LocalStorageHelper.setItem("count", count);
    }, [count]);

    return (
        <div>
            <h1>Counter: {count}</h1>
            <button onClick={() => setCount((prev) => prev + 1)}>Increment</button>
            <button onClick={() => setCount((prev) => prev - 1)}>Decrement</button>
        </div>
    );
};

export default Counter;
```

#### Test Cases

```typescript
// components/Counter.test.tsx
import React from "react";
import { render, screen, fireEvent } from "@testing-library/react";
import Counter from "./Counter";

describe("Counter Component", () => {
    it("should initialize with value from local storage", () => {
        localStorage.setItem("count", "5");
        render(<Counter />);
        expect(screen.getByText(/Counter: 5/i)).toBeInTheDocument();
    });

    it("should increment the counter", () => {
        render(<Counter />);
        fireEvent.click(screen.getByText(/Increment/i));
        expect(screen.getByText(/Counter: 1/i)).toBeInTheDocument();
    });

    it("should decrement the counter", () => {
        render(<Counter />);
        fireEvent.click(screen.getByText(/Decrement/i));
        expect(screen.getByText(/Counter: -1/i)).toBeInTheDocument();
    });
});
```

---

### 3. **React Hooks for Local Storage**

We can create a custom hook to encapsulate the logic for interacting with local storage.

#### Example: Custom Hook

```typescript
// hooks/useLocalStorage.ts
import { useState } from "react";

function useLocalStorage<T>(key: string, initialValue: T) {
    const [storedValue, setStoredValue] = useState<T>(() => {
        try {
            const item = localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            console.error("Error reading localStorage key", key, error);
            return initialValue;
        }
    });

    const setValue = (value: T) => {
        try {
            setStoredValue(value);
            localStorage.setItem(key, JSON.stringify(value));
        } catch (error) {
            console.error("Error setting localStorage key", key, error);
        }
    };

    return [storedValue, setValue] as const;
}

export default useLocalStorage;
```

#### Usage Example

```typescript
// components/ThemeSwitcher.tsx
import React from "react";
import useLocalStorage from "../hooks/useLocalStorage";

const ThemeSwitcher: React.FC = () => {
    const [theme, setTheme] = useLocalStorage<string>("theme", "light");

    return (
        <div className={`theme-${theme}`}>
            <h1>Current Theme: {theme}</h1>
            <button onClick={() => setTheme("light")}>Light</button>
            <button onClick={() => setTheme("dark")}>Dark</button>
        </div>
    );
};

export default ThemeSwitcher;
```

---

### 4. **Best Practices for Local Storage in React**

1. **Avoid Storing Sensitive Data**: Local storage is not secure; avoid storing sensitive information like passwords or tokens.
2. **Validate Stored Data**: Always validate and sanitize data retrieved from local storage.
3. **Error Handling**: Wrap local storage operations in `try-catch` blocks to handle potential errors.
4. **Abstract Logic**: Use utilities or hooks to encapsulate local storage interactions for reusability and type safety.
5. **Synchronize Across Tabs**: Use the `storage` event to synchronize data across multiple tabs.

#### Example: Tab Synchronization

```typescript
// hooks/useSyncLocalStorage.ts
import { useEffect } from "react";

export function useSyncLocalStorage(key: string, callback: (newValue: string | null) => void) {
    useEffect(() => {
        const handler = (event: StorageEvent) => {
            if (event.key === key) {
                callback(event.newValue);
            }
        };

        window.addEventListener("storage", handler);
        return () => window.removeEventListener("storage", handler);
    }, [key, callback]);
}
```