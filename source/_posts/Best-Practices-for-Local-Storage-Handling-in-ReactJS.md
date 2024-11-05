---
title: Best Practices for Local Storage Handling in ReactJS
date: 2024-11-04 17:49:26
categories:
- Deep Dive
- ReactJS
- Redux
- Best Practices
tags:
- Deep Dive
- ReactJS
- Redux
- Best Practices
---

- [Introduction](#introduction)
- [1. Local Storage Basics](#1-local-storage-basics)
- [1.1 Project Setup](#11-project-setup)
  - [Yarn Commands](#yarn-commands)
- [1.2 Project Folder Structure](#12-project-folder-structure)
  - [2.1 What is Local Storage?](#21-what-is-local-storage)
  - [2.2 Local Storage vs Session Storage](#22-local-storage-vs-session-storage)
- [3. Local Storage Basics](#3-local-storage-basics)
- [3.1 Basic API](#31-basic-api)
  - [3.2 Type Safety](#32-type-safety)
  - [3.3 Error Handling](#33-error-handling)
  - [3.4 Data Serialization](#34-data-serialization)
  - [3.5 Expiration Handling](#35-expiration-handling)
- [4. Best Practices](#4-best-practices)
  - [4.1 Encapsulating Local Storage Access](#41-encapsulating-local-storage-access)
- [5 More Scenarios](#5-more-scenarios)
  - [5.1 Scenario 1: User Preferences](#51-scenario-1-user-preferences)
  - [5.2 Scenario 2: Shopping Cart](#52-scenario-2-shopping-cart)
- [6-Design Diagrams](#6-design-diagrams)
  - [6.1 Component Diagram](#61-component-diagram)
  - [6.2 Sequence Diagram](#62-sequence-diagram)
- [7. Unit Test Cases](#7-unit-test-cases)
  - [7.1 UserPreferencesComponent.test.tsx](#71-userpreferencescomponenttesttsx)
  - [7.2 ShoppingCartComponent.test.tsx](#72-shoppingcartcomponenttesttsx)
- [8. Commands to Init and Add Dependencies](#8-commands-to-init-and-add-dependencies)
- [9. Conclusion](#9-conclusion)
- [10. More Samples](#10-more-samples)
  - [10.1 Project Setup](#101-project-setup)
- [References](#references)

<a name="introduction"></a>
## Introduction

Local storage is a powerful tool for persisting data in web applications. In ReactJS, local storage can be used to store user preferences, shopping cart items, and other data that needs to persist across sessions. This article will deep dive into best practices for handling local storage in ReactJS with TypeScript, focusing on functional components and the latest version of the tech stack. We will provide real production scenarios with TypeScript code samples, design diagrams, project folder structures, unit test cases, and commands to initialize and add dependencies using Yarn.

<a name="1-local-storage-basics"></a>
## 1. Local Storage Basics

<a name="11-project-setup"></a>
## 1.1 Project Setup
To get started, we need to set up a React project with TypeScript and add the necessary dependencies using Yarn.

### Yarn Commands
```bash
# Create a React project with TypeScript template
yarn create react-app local-storage-app --template typescript

# Navigate into the project directory
cd local-storage-app

# Add any additional dependencies if needed (e.g., testing libraries)
yarn add @testing-library/react @testing-library/jest-dom
```

<a name="12-project-folder-structure"></a>
## 1.2 Project Folder Structure

```
project-root/
├── src/
│   ├── components/
│   │   ├── UserPreferencesComponent.tsx
│   │   ├── ShoppingCartComponent.tsx
│   ├── hooks/
│   │   └── useLocalStorage.ts
│   ├── utils/
│   │   ├── localStorageUtils.ts
│   ├── App.tsx
│   ├── index.tsx
├── tests/
│   ├── components/
│   │   ├── UserPreferencesComponent.test.tsx
│   │   ├── ShoppingCartComponent.test.tsx
│   ├── utils/
│   │   ├── localStorageUtils.test.ts
├── package.json
├── tsconfig.json
├── yarn.lock
```

<a name="21-what-is-local-storage"></a>
### 2.1 What is Local Storage?

Local storage is a web storage API that allows you to store key-value pairs in a web browser. It is a synchronous, persistent storage that allows developers to store data client-side. The data stored in local storage persists even after the browser is closed and reopened. Leveraging local storage in React applications with TypeScript can help maintain user state, save preferences, and reduce API calls.

<a name="22-local-storage-vs-session-storage"></a>
### 2.2 Local Storage vs Session Storage

- **Local Storage**: Data persists across browser sessions.
- **Session Storage**: Data is cleared when the browser session ends (e.g., when the browser is closed).

<a name="3-local-storage-basics"></a>
## 3. Local Storage Basics

<a name="31-basic-api"></a>
## 3.1 Basic API
Local storage provides a simple API: `setItem()`, `getItem()`, and `removeItem()`. Data stored in local storage is accessible across browser sessions.

```typescript
// Setting an item
localStorage.setItem('key', 'value');

// Getting an item
const value = localStorage.getItem('key');

// Removing an item
localStorage.removeItem('key');
```

<a name="32-type-safety"></a>
### 3.2 Type Safety
Use TypeScript to ensure type safety when interacting with local storage. Create utility functions or custom hooks in TypeScript to handle type-safe operations.

```typescript
interface UserPreferences {
  theme: string;
  notifications: boolean;
}

const setUserPreferences = (preferences: UserPreferences): void => {
  setLocalStorageItem('userPreferences', preferences);
};

const getUserPreferences = (): UserPreferences | null => {
  return getLocalStorageItem<UserPreferences>('userPreferences');
};
```

<a name="33-error-handling"></a>
### 3.3 Error Handling

Handle errors gracefully when interacting with local storage. Wrap local storage operations in `try-catch` blocks to handle any potential errors gracefully.

```typescript
export const setLocalStorageItem = <T>(key: string, value: T): void => {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch (error) {
    console.error(`Error setting local storage item: ${error}`);
  }
};

export const getLocalStorageItem = <T>(key: string): T | null => {
  try {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  } catch (error) {
    console.error(`Error getting local storage item: ${error}`);
    return null;
  }
};
```

<a name="34-data-serialization"></a>
### 3.4 Data Serialization
Ensure that data is properly serialized and deserialized when storing and retrieving from local storage. Always serialize complex data structures as strings using `JSON.stringify()`.

```typescript
export const setLocalStorageItem = <T>(key: string, value: T): void => {
  localStorage.setItem(key, JSON.stringify(value));
};

export const getLocalStorageItem = <T>(key: string): T | null => {
  const item = localStorage.getItem(key);
  return item ? JSON.parse(item) : null;
};
```

<a name="35-expiration-handling"></a>
### 3.5 Expiration Handling

Implement expiration handling for local storage items to ensure that stale data is not used.

```typescript
interface ExpirableItem<T> {
  value: T;
  expiration: number;
}

export const setExpirableLocalStorageItem = <T>(key: string, value: T, expirationInMinutes: number): void => {
  const expiration = new Date().getTime() + expirationInMinutes * 60 * 1000;
  const item: ExpirableItem<T> = { value, expiration };
  setLocalStorageItem(key, item);
};

export const getExpirableLocalStorageItem = <T>(key: string): T | null => {
  const item = getLocalStorageItem<ExpirableItem<T>>(key);
  if (item && item.expiration > new Date().getTime()) {
    return item.value;
  }
  removeLocalStorageItem(key);
  return null;
};
```


<a name="4-best-practices"></a>
## 4. Best Practices

<a name="41-encapsulating-local-storage-access"></a>
### 4.1 Encapsulating Local Storage Access
Encapsulate local storage logic inside React hooks or service functions to keep components clean. Encapsulate local storage access within utility functions to avoid code duplication and improve maintainability.

```typescript
export const setLocalStorageItem = <T>(key: string, value: T): void => {
  localStorage.setItem(key, JSON.stringify(value));
};

export const getLocalStorageItem = <T>(key: string): T | null => {
  const item = localStorage.getItem(key);
  return item ? JSON.parse(item) : null;
};

export const removeLocalStorageItem = (key: string): void => {
  localStorage.removeItem(key);
};
```

<a name="5-more-scenarios"></a>
## 5 More Scenarios

<a name="51-scenario-1-user-preferences"></a>
### 5.1 Scenario 1: User Preferences

```typescript
import React, { useState, useEffect } from 'react';
import { setUserPreferences, getUserPreferences } from './localStorageUtils';

interface UserPreferences {
  theme: string;
  notifications: boolean;
}

const UserPreferencesComponent: React.FC = () => {
  const [preferences, setPreferences] = useState<UserPreferences | null>(null);

  useEffect(() => {
    const storedPreferences = getUserPreferences();
    if (storedPreferences) {
      setPreferences(storedPreferences);
    }
  }, []);

  const handleThemeChange = (theme: string) => {
    const updatedPreferences = { ...preferences, theme } as UserPreferences;
    setUserPreferences(updatedPreferences);
    setPreferences(updatedPreferences);
  };

  const handleNotificationsChange = (notifications: boolean) => {
    const updatedPreferences = { ...preferences, notifications } as UserPreferences;
    setUserPreferences(updatedPreferences);
    setPreferences(updatedPreferences);
  };

  return (
    <div>
      <h1>User Preferences</h1>
      <div>
        <label>Theme:</label>
        <select value={preferences?.theme} onChange={(e) => handleThemeChange(e.target.value)}>
          <option value="light">Light</option>
          <option value="dark">Dark</option>
        </select>
      </div>
      <div>
        <label>Notifications:</label>
        <input
          type="checkbox"
          checked={preferences?.notifications}
          onChange={(e) => handleNotificationsChange(e.target.checked)}
        />
      </div>
    </div>
  );
};

export default UserPreferencesComponent;
```

<a name="52-scenario-2-shopping-cart"></a>
### 5.2 Scenario 2: Shopping Cart

```typescript
import React, { useState, useEffect } from 'react';
import { setLocalStorageItem, getLocalStorageItem, removeLocalStorageItem } from './localStorageUtils';

interface CartItem {
  id: number;
  name: string;
  price: number;
}

const ShoppingCartComponent: React.FC = () => {
  const [cartItems, setCartItems] = useState<CartItem[]>([]);

  useEffect(() => {
    const storedCartItems = getLocalStorageItem<CartItem[]>('cartItems');
    if (storedCartItems) {
      setCartItems(storedCartItems);
    }
  }, []);

  const addToCart = (item: CartItem) => {
    const updatedCartItems = [...cartItems, item];
    setLocalStorageItem('cartItems', updatedCartItems);
    setCartItems(updatedCartItems);
  };

  const removeFromCart = (id: number) => {
    const updatedCartItems = cartItems.filter((item) => item.id !== id);
    setLocalStorageItem('cartItems', updatedCartItems);
    setCartItems(updatedCartItems);
  };

  return (
    <div>
      <h1>Shopping Cart</h1>
      <ul>
        {cartItems.map((item) => (
          <li key={item.id}>
            {item.name} - ${item.price}
            <button onClick={() => removeFromCart(item.id)}>Remove</button>
          </li>
        ))}
      </ul>
      <button onClick={() => addToCart({ id: Date.now(), name: 'New Item', price: 10 })}>Add Item</button>
    </div>
  );
};

export default ShoppingCartComponent;
```

<a name="6-design-diagrams"></a>
## 6-Design Diagrams

<a name="61-component-diagram"></a>
### 6.1 Component Diagram

```
+-------------------+       +-------------------+       +-------------------+
| App               |       | UserPreferences   |       | ShoppingCart      |
+-------------------+       +-------------------+       +-------------------+
        |                           |                           |
        | (1) User Preferences      |                           |
        |-------------------------->|                           |
        |                           | (2) Shopping Cart         |
        |                           |-------------------------->|
        |                           |                           |
        | (3) Local Storage         |                           |
        |<--------------------------|                           |
        |                           |                           |
+-------------------+       +-------------------+       +-------------------+
| App               |       | UserPreferences   |       | ShoppingCart      |
+-------------------+       +-------------------+       +-------------------+
```

<a name="62-sequence-diagram"></a>
### 6.2 Sequence Diagram

```
+-------------------+       +-------------------+       +-------------------+
| User              |       | UserPreferences   |       | Local Storage     |
+-------------------+       +-------------------+       +-------------------+
        |                           |                           |
        | (1) Change Theme          |                           |
        |-------------------------->|                           |
        |                           | (2) Update Preferences    |
        |                           |-------------------------->|
        |                           |                           |
        | (3) Store Preferences     |                           |
        |<--------------------------|                           |
        |                           |                           |
+-------------------+       +-------------------+       +-------------------+
| User              |       | UserPreferences   |       | Local Storage     |
+-------------------+       +-------------------+       +-------------------+
```



<a name="7-unit-test-cases"></a>
## 7. Unit Test Cases

<a name="71-user-preferences-component-test"></a>
### 7.1 UserPreferencesComponent.test.tsx

```typescript
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import UserPreferencesComponent from '../../src/components/UserPreferencesComponent';
import { setUserPreferences, getUserPreferences } from '../../src/utils/localStorageUtils';

jest.mock('../../src/utils/localStorageUtils', () => ({
  setUserPreferences: jest.fn(),
  getUserPreferences: jest.fn(),
}));

test('UserPreferencesComponent updates and retrieves preferences', () => {
  (getUserPreferences as jest.Mock).mockReturnValue({ theme: 'light', notifications: true });

  const { getByLabelText, getByText } = render(<UserPreferencesComponent />);

  fireEvent.change(getByLabelText('Theme:'), { target: { value: 'dark' } });
  expect(setUserPreferences).toHaveBeenCalledWith({ theme: 'dark', notifications: true });

  fireEvent.click(getByLabelText('Notifications:'));
  expect(setUserPreferences).toHaveBeenCalledWith({ theme: 'dark', notifications: false });
});
```

<a name="72-shopping-cart-component-test"></a>
### 7.2 ShoppingCartComponent.test.tsx

```typescript
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import ShoppingCartComponent from '../../src/components/ShoppingCartComponent';
import { setLocalStorageItem, getLocalStorageItem, removeLocalStorageItem } from '../../src/utils/localStorageUtils';

jest.mock('../../src/utils/localStorageUtils', () => ({
  setLocalStorageItem: jest.fn(),
  getLocalStorageItem: jest.fn(),
  removeLocalStorageItem: jest.fn(),
}));

test('ShoppingCartComponent adds and removes items', () => {
  (getLocalStorageItem as jest.Mock).mockReturnValue([{ id: 1, name: 'Item 1', price: 10 }]);

  const { getByText } = render(<ShoppingCartComponent />);

  fireEvent.click(getByText('Add Item'));
  expect(setLocalStorageItem).toHaveBeenCalled();

  fireEvent.click(getByText('Remove'));
  expect(removeLocalStorageItem).toHaveBeenCalled();
});
```

<a name="8-commands-to-init-and-add-dependencies"></a>
## 8. Commands to Init and Add Dependencies

```bash
# Initialize a new React project with TypeScript
yarn create react-app my-app --template typescript

# Navigate to the project directory
cd my-app

# Add Testing Library
yarn add --dev @testing-library/react @testing-library/jest-dom
```

<a name="9-conclusion"></a>
## 9. Conclusion

Local storage is a powerful tool for persisting data in web applications. This article provided a deep dive into best practices for handling local storage in ReactJS with TypeScript, focusing on functional components and the latest version of the tech stack. We explored real production scenarios with TypeScript code samples, design diagrams, project folder structures, unit test cases, and commands to initialize and add dependencies using Yarn.

By understanding and applying these best practices, developers can effectively manage local storage in their ReactJS applications, ensuring they remain scalable, maintainable, and performant.


<a name="10-more-samples"></a>
## 10. More Samples

<a name="101-project-setup"></a>
### 10.1 Project Setup
To get started, we need to set up a React project with TypeScript and add the necessary dependencies using Yarn.

>**Yarn Commands**
```bash
# Create a React project with TypeScript template
yarn create react-app local-storage-app --template typescript

# Navigate into the project directory
cd local-storage-app

# Add any additional dependencies if needed (e.g., testing libraries)
yarn add @testing-library/react @testing-library/jest-dom
```

>**Project Folder Structure**
```
local-storage-app/
├── src/
│   ├── components/
│   │   ├── LocalStorageHandler.tsx
│   │   └── UserPreferences.tsx
│   ├── hooks/
│   │   └── useLocalStorage.ts
│   ├── tests/
│   │   └── LocalStorageHandler.test.tsx
│   ├── App.tsx
│   └── index.tsx
├── public/
└── package.json
```

>**useLocalStorage.ts**
```typescript
// src/hooks/useLocalStorage.ts
import { useState } from 'react';

/**
 * Custom hook for local storage handling
 * @param key - The key under which data is stored in local storage
 * @param initialValue - The initial value to use if no value is stored
 * @returns A tuple containing the stored value and a function to set the value
 */
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch (error) {
      console.error('Error reading from local storage', error);
      return initialValue;
    }
  });

  /**
   * Sets a new value in local storage
   * @param value - The new value to store
   */
  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to local storage', error);
    }
  };

  return [storedValue, setValue] as const;
}

export default useLocalStorage;
```

>**UserPreferences**: Detailed Code for Using `useLocalStorage` in a Component
```typescript
// src/components/UserPreferences.tsx
import React from 'react';
import useLocalStorage from '../hooks/useLocalStorage';

const UserPreferences: React.FC = () => {
  // Using the custom hook to manage theme preference
  const [theme, setTheme] = useLocalStorage<string>('theme', 'light');

  // Toggles between 'light' and 'dark' themes
  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return (
    <div>
      <h1>User Preferences</h1>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>
        Toggle Theme
      </button>
    </div>
  );
};

export default UserPreferences;
```

>**Type-Safe** Local Storage Utility Functions
>For larger applications, you might prefer having utility functions to handle local storage logic:
```typescript
// src/utils/localStorageUtils.ts
export const getLocalStorageItem = <T>(key: string, defaultValue: T): T => {
  try {
    const storedItem = localStorage.getItem(key);
    return storedItem ? (JSON.parse(storedItem) as T) : defaultValue;
  } catch (error) {
    console.error(`Error reading key "${key}" from local storage`, error);
    return defaultValue;
  }
};

export const setLocalStorageItem = <T>(key: string, value: T): void => {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch (error) {
    console.error(`Error setting key "${key}" in local storage`, error);
  }
};
```

>Detailed Unit Test Cases
```typescript
// src/tests/useLocalStorage.test.tsx
import { renderHook, act } from '@testing-library/react';
import useLocalStorage from '../hooks/useLocalStorage';

describe('useLocalStorage hook', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('should return initial value if no value is stored', () => {
    const { result } = renderHook(() => useLocalStorage('testKey', 'initialValue'));
    expect(result.current[0]).toBe('initialValue');
  });

  it('should save and retrieve a value', () => {
    const { result } = renderHook(() => useLocalStorage('testKey', 'initialValue'));

    act(() => {
      result.current[1]('newValue');
    });

    expect(result.current[0]).toBe('newValue');
    expect(localStorage.getItem('testKey')).toBe(JSON.stringify('newValue'));
  });

  it('should handle JSON parsing errors gracefully', () => {
    localStorage.setItem('testKey', '{invalid JSON');
    const { result } = renderHook(() => useLocalStorage('testKey', 'defaultValue'));
    expect(result.current[0]).toBe('defaultValue');
  });
});
```

>Commands to Run Unit Tests
```bash
# Run unit tests with Yarn
yarn test
```


<a name="references"></a>
## References

- [React Documentation](https://reactjs.org/docs/getting-started.html)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [Testing Library](https://testing-library.com/docs/react-testing-library/intro)

---

This article provides a comprehensive guide to best practices for local storage handling in ReactJS with TypeScript, including detailed code samples, design diagrams, project folder structures, unit test cases, and commands to initialize and add dependencies using Yarn. By understanding and applying these practices, developers can leverage the full potential of local storage in their ReactJS projects.
