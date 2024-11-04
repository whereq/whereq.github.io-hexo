---
title: Deep Dive into State Management in ReactJS
date: 2024-11-04 15:47:50
categories:
- Deep Dive
- ReactJS
- Redux
tags:
- Deep Dive
- ReactJS
- Redux
---

- [Introduction](#introduction)
- [1. State Management Basics](#1-state-management-basics)
  - [1.1 What is State?](#11-what-is-state)
  - [1.2 Local State vs Global State](#12-local-state-vs-global-state)
- [2. Setting Up the Project](#2-setting-up-the-project)
  - [Initialize the React Project with TypeScript](#initialize-the-react-project-with-typescript)
- [3. State Management Options](#3-state-management-options)
  - [3.1 React Context API](#31-react-context-api)
- [4. Local State Management with `useState` and `useReducer`](#4-local-state-management-with-usestate-and-usereducer)
  - [4.1 Using `useState`](#41-using-usestate)
  - [4.2 Using `useReducer`](#42-using-usereducer)
- [5. Global State Management with Redux](#5-global-state-management-with-redux)
  - [5.1 Architecture of Redux](#51-architecture-of-redux)
  - [5.2 Redux state machine](#52-redux-state-machine)
  - [Explanation:](#explanation)
  - [5.2 Redux Samples](#52-redux-samples)
- [6. Alternative Global State Management with Zustand](#6-alternative-global-state-management-with-zustand)
  - [6.1 Architecture of Zustand](#61-architecture-of-zustand)
  - [6.2 TypeScript Code Example](#62-typescript-code-example)
- [More Scenarios](#more-scenarios)
  - [7.1 Scenario 1: User Authentication](#71-scenario-1-user-authentication)
    - [React Context API](#react-context-api)
    - [Redux](#redux)
  - [7.2 Scenario 2: Shopping Cart](#72-scenario-2-shopping-cart)
    - [React Context API](#react-context-api-1)
    - [Redux](#redux-1)
- [8. Some Diagrams](#8-some-diagrams)
- [Project Folder Structure](#project-folder-structure)
- [Unit Test Cases](#unit-test-cases)
  - [LoginPage.test.tsx](#loginpagetesttsx)
  - [Dashboard.test.tsx](#dashboardtesttsx)
- [Commands to Init and Add Dependencies](#commands-to-init-and-add-dependencies)
- [Conclusion](#conclusion)
- [References](#references)

<a name="introduction"></a>
## Introduction

State management is a critical aspect of building complex ReactJS applications. Proper state management ensures that your application remains scalable, maintainable, and performant. This article will deep dive into state management in ReactJS, focusing on functional components and the latest version of the tech stack. We will explore two main state management options: React Context API and Redux. Additionally, we will provide real production scenarios with TypeScript code samples, design diagrams, project folder structures, unit test cases, and commands to initialize and add dependencies using Yarn.

<a name="1-state-management-basics"></a>
## 1. State Management Basics

<a name="11-what-is-state"></a>
### 1.1 What is State?

State in ReactJS is an object that represents the dynamic data of a component. It determines how the component renders and behaves. State can be local to a single component or shared across multiple components.

State management refers to managing data that changes over time, ensuring that the user interface updates appropriately. Choosing the right state management solution is crucial for developing scalable and maintainable applications. We'll explore `useState` and `useReducer` for local state management and Redux and Zustand for global state management, with detailed TypeScript examples.

<a name="12-local-state-vs-global-state"></a>
### 1.2 Local State vs Global State

- **Local State**: State that is confined to a single component. It is managed using the `useState` hook.
- **Global State**: State that is shared across multiple components. It is managed using state management libraries like React Context API or Redux.


<a name="2-setting-up-the-project"></a>
## 2. Setting Up the Project

### Initialize the React Project with TypeScript

Run the following commands to create a new React project with TypeScript and install the necessary dependencies:

```bash
# Create a new React app with TypeScript
yarn create react-app react-state-management --template typescript

# Navigate to the project folder
cd react-state-management

# Install Redux and Zustand for state management
yarn add redux react-redux @reduxjs/toolkit zustand

# Install TypeScript types for React and Redux
yarn add -D @types/react-redux
```

<a name="3-state-management-options"></a>
## 3. State Management Options

<a name="31-react-context-api"></a>
### 3.1 React Context API

The React Context API provides a way to pass data through the component tree without having to pass props down manually at every level. It is ideal for small to medium-sized applications.

```typescript
import React, { createContext, useContext, useState } from 'react';

interface AuthContextType {
  isAuthenticated: boolean;
  login: () => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType>({
  isAuthenticated: false,
  login: () => {},
  logout: () => {},
});

export const AuthProvider: React.FC = ({ children }) => {
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const login = () => setIsAuthenticated(true);
  const logout = () => setIsAuthenticated(false);

  return (
    <AuthContext.Provider value={{ isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

<a name="4-local-state-management"></a>
## 4. Local State Management with `useState` and `useReducer`

<a name="41-using-usestate"></a>
### 4.1 Using `useState`

`useState` is simple to use and ideal for managing local state.

**TypeScript Code Sample**:
```typescript
import React, { useState } from 'react';

const Counter: React.FC = () => {
  const [count, setCount] = useState<number>(0);

  const increment = (): void => {
    setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>Current Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};

export default Counter;
```

**Best Practices**:
- Use `useState` for simple and straightforward state updates.
- Type your state variables for better type safety.

<a name="42-using-usereducer"></a>
### 4.2 Using `useReducer`

`useReducer` is suitable for managing complex state logic.

**TypeScript Code Sample**:
```typescript
import React, { useReducer } from 'react';

interface State {
  count: number;
}

interface Action {
  type: 'increment' | 'decrement';
}

const initialState: State = { count: 0 };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unknown action type');
  }
}

const Counter: React.FC = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Current Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
    </div>
  );
};

export default Counter;
```

**Best Practices**:
- Use `useReducer` when state logic is complex or involves multiple sub-values.
- Type both state and actions for enhanced type safety.

---


<a name="5-global-state-management-redux"></a>
## 5. Global State Management with Redux

Redux is a predictable state container for JavaScript apps. It is widely used for managing global state in large-scale applications.

<a name="51-architecture-of-redux"></a>
### 5.1 Architecture of Redux

**Redux Architecture**:
```
+------------+      +---------------+      +-------------+
|   Action   | --> |    Reducer     | --> |    Store     |
+------------+      +---------------+      +-------------+
       ^                                          |
       |                                          |
       +------------------------------------------+
```

<a name="52-redux-state-machine"></a>
### 5.2 Redux state machine
Here's a textual representation of a state machine diagram for a Redux state management flow:

```
+--------------------+       Action       +----------------------+
|                    |   (dispatch)       |                      |
|   React Component  | -----------------> |       Action         |
|                    |                    | (type & payload)     |
+--------------------+                    +----------------------+
                                               |
                                               |
                                       Dispatches action
                                               |
                                               v
                                       +----------------+
                                       |                |
                                       |    Reducer     |
                                       | (pure function)|
                                       +----------------+
                                               |
                       Updates state           |
                       based on action         |
                                               v
                                       +----------------+
                                       |                |
                                       |    Store       |
                                       | (holds state)  |
                                       +----------------+
                                               |
                      State subscription       |
                      triggers re-render       |
                      of component             |
                                               v
+--------------------+      Updated       +----------------------+
|                    |      State         |                      |
|   React Component  | <----------------- |      Store           |
|                    |                    |                      |
+--------------------+                    +----------------------+
```

### Explanation:
- **React Component**: The UI component where user interactions or events trigger actions.
- **Action**: An object describing what type of event occurred (e.g., `{ type: 'INCREMENT' }`).
- **Reducer**: A pure function that takes the current state and an action, then returns a new state.
- **Store**: The central place where the state is kept, updated, and managed.
- **State Flow**: Components subscribe to the store to get updates when the state changes, causing a re-render to reflect new data.


<a name="52-redux-samples"></a>
### 5.2 Redux Samples 

>**The Simple One**
```typescript
import { createStore } from 'redux';

interface State {
  count: number;
}

interface Action {
  type: string;
}

const initialState: State = { count: 0 };

const reducer = (state = initialState, action: Action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const store = createStore(reducer);

export default store;
```


>**The Sample of Details**
**`actions.ts`**:
```typescript
export const increment = () => ({ type: 'INCREMENT' } as const);
export const decrement = () => ({ type: 'DECREMENT' } as const);

export type ActionType = ReturnType<typeof increment | typeof decrement>;
```

**`reducers.ts`**:
```typescript
import { ActionType } from './actions';

interface State {
  count: number;
}

const initialState: State = { count: 0 };

export const counterReducer = (state: State = initialState, action: ActionType): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
};
```

**`store.ts`**:
```typescript
import { createStore } from 'redux';
import { counterReducer } from './reducers';

const store = createStore(counterReducer);

export default store;
```

**`App.tsx`**:
```typescript
import React from 'react';
import { Provider, useDispatch, useSelector } from 'react-redux';
import store from './state/redux/store';
import { increment, decrement } from './state/redux/actions';

interface RootState {
  count: number;
}

const App: React.FC = () => {
  const count = useSelector((state: RootState) => state.count);
  const dispatch = useDispatch();

  return (
    <Provider store={store}>
      <div>
        <p>Current Count: {count}</p>
        <button onClick={() => dispatch(increment())}>Increment</button>
        <button onClick={() => dispatch(decrement())}>Decrement</button>
      </div>
    </Provider>
  );
};

export default App;
```

---

<a name="6-global-state-management-zustand"></a>
## 6. Alternative Global State Management with Zustand

Zustand offers a simple and flexible state management solution without the boilerplate of Redux.

<a name="61-architecture-of-zustand"></a>
### 6.1 Architecture of Zustand

**Simple Architecture**:
```
Component --> useStore --> State & Actions
```

<a name="62-zustand-typescript-example"></a>
### 6.2 TypeScript Code Example

**`useStore.ts`**:
```typescript
import create from 'zustand';

interface StoreState {
  count: number;
  increment: () => void;
  decrement: () => void;
}

const useStore = create<StoreState>(set => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 }))
}));

export default useStore;
```

**`App.tsx`**:
```typescript
import React from 'react';
import useStore from './state/zustand/useStore';

const App

: React.FC = () => {
  const { count, increment, decrement } = useStore();

  return (
    <div>
      <p>Current Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
};

export default App;
```

---

<a name="7-more-scenarios"></a>
## More Scenarios

<a name="71-scenario-1-user-authentication"></a>
### 7.1 Scenario 1: User Authentication

#### React Context API

```typescript
import React from 'react';
import { AuthProvider, useAuth } from './AuthContext';

const LoginPage: React.FC = () => {
  const { login } = useAuth();

  return (
    <div>
      <h1>Login Page</h1>
      <button onClick={login}>Login</button>
    </div>
  );
};

const Dashboard: React.FC = () => {
  const { isAuthenticated, logout } = useAuth();

  return (
    <div>
      <h1>Dashboard</h1>
      {isAuthenticated ? (
        <button onClick={logout}>Logout</button>
      ) : (
        <p>Please login to access the dashboard.</p>
      )}
    </div>
  );
};

const App: React.FC = () => (
  <AuthProvider>
    <LoginPage />
    <Dashboard />
  </AuthProvider>
);

export default App;
```

#### Redux

```typescript
import React from 'react';
import { Provider, useSelector, useDispatch } from 'react-redux';
import store from './store';

const LoginPage: React.FC = () => {
  const dispatch = useDispatch();

  const login = () => dispatch({ type: 'LOGIN' });

  return (
    <div>
      <h1>Login Page</h1>
      <button onClick={login}>Login</button>
    </div>
  );
};

const Dashboard: React.FC = () => {
  const isAuthenticated = useSelector((state: any) => state.isAuthenticated);
  const dispatch = useDispatch();

  const logout = () => dispatch({ type: 'LOGOUT' });

  return (
    <div>
      <h1>Dashboard</h1>
      {isAuthenticated ? (
        <button onClick={logout}>Logout</button>
      ) : (
        <p>Please login to access the dashboard.</p>
      )}
    </div>
  );
};

const App: React.FC = () => (
  <Provider store={store}>
    <LoginPage />
    <Dashboard />
  </Provider>
);

export default App;
```

<a name="72-scenario-2-shopping-cart"></a>
### 7.2 Scenario 2: Shopping Cart

#### React Context API

```typescript
import React, { createContext, useContext, useState } from 'react';

interface CartItem {
  id: number;
  name: string;
  price: number;
}

interface CartContextType {
  cartItems: CartItem[];
  addToCart: (item: CartItem) => void;
  removeFromCart: (id: number) => void;
}

const CartContext = createContext<CartContextType>({
  cartItems: [],
  addToCart: () => {},
  removeFromCart: () => {},
});

export const CartProvider: React.FC = ({ children }) => {
  const [cartItems, setCartItems] = useState<CartItem[]>([]);

  const addToCart = (item: CartItem) => setCartItems([...cartItems, item]);
  const removeFromCart = (id: number) =>
    setCartItems(cartItems.filter((item) => item.id !== id));

  return (
    <CartContext.Provider value={{ cartItems, addToCart, removeFromCart }}>
      {children}
    </CartContext.Provider>
  );
};

export const useCart = () => useContext(CartContext);
```

#### Redux

```typescript
import { createStore } from 'redux';

interface CartItem {
  id: number;
  name: string;
  price: number;
}

interface State {
  cartItems: CartItem[];
}

interface Action {
  type: string;
  payload?: any;
}

const initialState: State = { cartItems: [] };

const reducer = (state = initialState, action: Action) => {
  switch (action.type) {
    case 'ADD_TO_CART':
      return { cartItems: [...state.cartItems, action.payload] };
    case 'REMOVE_FROM_CART':
      return {
        cartItems: state.cartItems.filter((item) => item.id !== action.payload),
      };
    default:
      return state;
  }
};

const store = createStore(reducer);

export default store;
```

<a name="8-some-diagrams"></a>
## 8. Some Diagrams

```
+-------------------+       +-------------------+       +-------------------+
| App               |       | AuthProvider      |       | CartProvider      |
+-------------------+       +-------------------+       +-------------------+
        |                            |                           |
        | (1) Auth Context           |                           |
        |--------------------------->|                           |
        |                            | (2) Cart Context          |
        |                            |-------------------------->|
        |                            |                           |
        | (3) LoginPage              |                           |
        |<------------ --------------|                           |
        |                            |                           |
        | (4) Dashboard              |                           |
        |<------------ --------------|                           |
        |                            |                           |
+-------------------+       +-------------------+       +-------------------+
| App               |       | AuthProvider      |       | CartProvider      |
+-------------------+       +-------------------+       +-------------------+
```

```
+-------------------+       +-------------------+       +-------------------+
| User              |       | LoginPage         |       | AuthProvider      |
+-------------------+       +-------------------+       +-------------------+
        |                           |                           |
        | (1) Click Login Button    |                           |
        |-------------------------->|                           |
        |                           | (2) Dispatch Login Action |
        |                           |-------------------------->|
        |                           |                           |
        | (3) Update Auth State     |                           |
        |<--------------------------|                           |
        |                           |                           |
+-------------------+       +-------------------+       +-------------------+
| User              |       | LoginPage         |       | AuthProvider      |
+-------------------+       +-------------------+       +-------------------+
```

<a name="project-folder-structure"></a>
## Project Folder Structure

```
project-root/
├── src/
│   ├── components/
│   │   ├── LoginPage.tsx
│   │   ├── Dashboard.tsx
│   ├── contexts/
│   │   ├── AuthContext.tsx
│   │   ├── CartContext.tsx
│   ├── redux/
│   │   ├── store.ts
│   ├── App.tsx
│   ├── index.tsx
├── tests/
│   ├── components/
│   │   ├── LoginPage.test.tsx
│   │   ├── Dashboard.test.tsx
│   ├── contexts/
│   │   ├── AuthContext.test.tsx
│   │   ├── CartContext.test.tsx
│   ├── redux/
│   │   ├── store.test.ts
├── package.json
├── tsconfig.json
├── yarn.lock
```

<a name="unit-test-cases"></a>
## Unit Test Cases

<a name="login-page-test"></a>
### LoginPage.test.tsx

```typescript
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import { AuthProvider, useAuth } from '../../src/contexts/AuthContext';
import LoginPage from '../../src/components/LoginPage';

const TestComponent: React.FC = () => {
  const { isAuthenticated } = useAuth();
  return <div>{isAuthenticated ? 'Logged In' : 'Logged Out'}</div>;
};

test('LoginPage logs in user', () => {
  const { getByText } = render(
    <AuthProvider>
      <LoginPage />
      <TestComponent />
    </AuthProvider>
  );

  fireEvent.click(getByText('Login'));
  expect(getByText('Logged In')).toBeInTheDocument();
});
```

<a name="dashboard-test"></a>
### Dashboard.test.tsx

```typescript
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import { AuthProvider, useAuth } from '../../src/contexts/AuthContext';
import Dashboard from '../../src/components/Dashboard';

const TestComponent: React.FC = () => {
  const { isAuthenticated } = useAuth();
  return <div>{isAuthenticated ? 'Logged In' : 'Logged Out'}</div>;
};

test('Dashboard logs out user', () => {
  const { getByText } = render(
    <AuthProvider>
      <Dashboard />
      <TestComponent />
    </AuthProvider>
  );

  fireEvent.click(getByText('Logout'));
  expect(getByText('Logged Out')).toBeInTheDocument();
});
```

<a name="commands-to-init-and-add-dependencies"></a>
## Commands to Init and Add Dependencies

```bash
# Initialize a new React project with TypeScript
yarn create react-app my-app --template typescript

# Navigate to the project directory
cd my-app

# Add Redux and React-Redux
yarn add redux react-redux

# Add React Context API (no additional packages needed)

# Add Testing Library
yarn add --dev @testing-library/react @testing-library/jest-dom
```

<a name="conclusion"></a>
## Conclusion

State management is a crucial aspect of building scalable and maintainable ReactJS applications. This article provided a deep dive into state management in ReactJS, focusing on functional components and the latest version of the tech stack. We explored two main state management options: React Context API and Redux. Additionally, we provided real production scenarios with TypeScript code samples, design diagrams, project folder structures, unit test cases, and commands to initialize and add dependencies using Yarn.

By understanding and applying these concepts, developers can effectively manage state in their ReactJS applications, ensuring they remain scalable, maintainable, and performant.

<a name="references"></a>
## References

- [React Documentation](https://reactjs.org/docs/getting-started.html)
- [Redux Documentation](https://redux.js.org/introduction/getting-started)
- [React Context API](https://reactjs.org/docs/context.html)
- [Testing Library](https://testing-library.com/docs/react-testing-library/intro)

---

This article provides a comprehensive guide to state management in ReactJS, including detailed code samples, design diagrams, project folder structures, unit test cases, and commands to initialize and add dependencies using Yarn. By understanding and applying these practices, developers can leverage the full potential of state management in their ReactJS projects.