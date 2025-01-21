---
title: 'Component Communication in React: A Comprehensive Guide'
date: 2025-01-21 11:05:53
categories:
- ReactJS
tags:
- ReactJS
---

---

## Table of Contents
1. [Props (Parent to Child)](#1-props-parent-to-child)
2. [Callback Functions (Child to Parent)](#2-callback-functions-child-to-parent)
3. [React Context API (Global State)](#3-react-context-api-global-state)
4. [State Management Libraries (Redux, Zustand, etc.)](#4-state-management-libraries-redux-zustand-etc)
5. [Refs (Direct DOM Manipulation or Between Components)](#5-refs-direct-dom-manipulation-or-between-components)
6. [Event Emitters (Custom Events)](#6-event-emitters-custom-events)
7. [URL Parameters (React Router)](#7-url-parameters-react-router)
8. [Third-Party Pub/Sub Libraries (e.g., PubSubJS, RxJS)](#8-third-party-pubsub-libraries-eg-pubsubjs-rxjs)
9. [API/Server Communication](#9-apiserver-communication)

---

## 1. Props (Parent to Child)
Props allow data to flow from a parent component to a child component.

### Example Code:
```jsx
function Parent() {
  const message = "Hello from Parent!";
  return <Child message={message} />;
}

function Child({ message }) {
  return <p>{message}</p>;
}
```

### Textual Diagram:
```
Parent -> Props -> Child
```

---

## 2. Callback Functions (Child to Parent)
A child component can communicate with its parent by invoking a callback function passed as a prop.

### Example Code:
```jsx
function Parent() {
  const handleChildEvent = (data) => {
    console.log("Received from child:", data);
  };

  return <Child onSend={handleChildEvent} />;
}

function Child({ onSend }) {
  return <button onClick={() => onSend("Hello Parent!")}>Send Message</button>;
}
```

### Textual Diagram:
```
Child -> Callback Function -> Parent
```

---

## 3. React Context API (Global State)
The Context API allows you to share data across the component tree without passing props explicitly.

### Example Code:
```jsx
import React, { createContext, useContext } from 'react';

const MyContext = createContext();

function Parent() {
  return (
    <MyContext.Provider value="Shared Data">
      <Child />
    </MyContext.Provider>
  );
}

function Child() {
  const data = useContext(MyContext);
  return <p>{data}</p>;
}
```

### Textual Diagram:
```
Parent -> Context Provider -> Child
         Context Consumer <- Child
```

---

## 4. State Management Libraries (Redux, Zustand, etc.)
State management libraries allow you to manage the global state of an application efficiently.

### Example Code (Redux):
```jsx
import { createStore } from "redux";
import { Provider, useDispatch, useSelector } from "react-redux";

const reducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    default:
      return state;
  }
};

const store = createStore(reducer);

function Counter() {
  const count = useSelector((state) => state.count);
  const dispatch = useDispatch();

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>Increment</button>
    </div>
  );
}

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

### Textual Diagram:
```
State -> Redux Store -> Component
        Dispatch Action <- Component
```

---

## 5. Refs (Direct DOM Manipulation or Between Components)
Refs allow direct interaction with DOM elements or instances of other components.

### Example Code:
```jsx
function Parent() {
  const childRef = useRef();

  const handleClick = () => {
    childRef.current.sayHello();
  };

  return (
    <>
      <Child ref={childRef} />
      <button onClick={handleClick}>Call Child Method</button>
    </>
  );
}

const Child = React.forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({
    sayHello: () => {
      alert("Hello from Child!");
    },
  }));
  return <p>Child Component</p>;
});
```

### Textual Diagram:
```
Parent -> Ref -> Child Method
```

---

## 6. Event Emitters (Custom Events)
Custom events can be used to communicate between loosely coupled components.

### Example Code:
```jsx
import { EventEmitter } from "events";

const eventEmitter = new EventEmitter();

function ComponentA() {
  const sendMessage = () => {
    eventEmitter.emit("message", "Hello from ComponentA!");
  };

  return <button onClick={sendMessage}>Send Message</button>;
}

function ComponentB() {
  useEffect(() => {
    eventEmitter.on("message", (data) => {
      console.log("Received:", data);
    });

    return () => {
      eventEmitter.removeAllListeners("message");
    };
  }, []);

  return <p>Listening for messages...</p>;
}
```

### Textual Diagram:
```
ComponentA -> Event Emitter -> ComponentB
```

### Comparison with Callback-Based Approach

| **Aspect**              | **EventEmitter**                          | **Callback-Based**                     |
|--------------------------|-------------------------------------------|----------------------------------------|
| **Coupling**             | Decoupled (uses events)                  | Tightly coupled (passes callbacks)     |
| **Scalability**          | Highly scalable for multiple events      | Less scalable for many events          |
| **Complexity**           | Slightly more complex (requires setup)   | Simpler (direct function calls)        |
| **Reusability**          | Reusable across the application          | Limited to specific components         |
| **Memory Management**    | Requires cleanup (off method)            | Automatically cleaned up by React      |

---

## 7. URL Parameters (React Router)
React Router enables communication via route parameters or query strings.

### Example Code:
```jsx
import { BrowserRouter as Router, Route, useParams } from "react-router-dom";

function Parent() {
  return (
    <Router>
      <Route path="/user/:id" component={Child} />
    </Router>
  );
}

function Child() {
  const { id } = useParams();
  return <p>User ID: {id}</p>;
}
```

### Textual Diagram:
```
URL -> Route Params -> Child
```

---

## 8. Third-Party Pub/Sub Libraries (e.g., PubSubJS, RxJS)
Libraries like RxJS or PubSubJS enable event-based communication.

### Example Code (RxJS):
```jsx
import { Subject } from "rxjs";

const subject = new Subject();

function ComponentA() {
  const sendMessage = () => {
    subject.next("Hello from ComponentA!");
  };

  return <button onClick={sendMessage}>Send Message</button>;
}

function ComponentB() {
  useEffect(() => {
    const subscription = subject.subscribe((data) => {
      console.log("Received:", data);
    });

    return () => subscription.unsubscribe();
  }, []);

  return <p>Listening for messages...</p>;
}
```

### Textual Diagram:
```
ComponentA -> Subject -> ComponentB
```

---

## 9. API/Server Communication
Components can exchange data through a server or API.

### Example Code:
```jsx
function Parent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/api/data")
      .then((response) => response.json())
      .then((data) => setData(data));
  }, []);

  return <Child data={data} />;
}

function Child({ data }) {
  return <p>{data ? `Fetched Data: ${data}` : "Loading..."}</p>;
}
```

### Textual Diagram:
```
Server <- Fetch API -> Parent -> Props -> Child
```

---


