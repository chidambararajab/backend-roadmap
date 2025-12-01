# React Developer Interview Preparation’s

### React

- **forwardRef (How it used & use cases)**
    
    The `useForwardRef` hook in React allows a functional component to forward its ref to a child component, enabling the parent component to directly access the child's DOM node or instance methods. This is useful for cases where you want to manage focus or other interactions on a child component while maintaining a clean API.
    
    ### Example: Using `React.forwardRef`
    
    1. **Child Component**: A simple input component that forwards its ref.
    
    ```jsx
    import React, { forwardRef } from 'react';
    
    // Child component that forwards its ref
    const TextInput = forwardRef((props, ref) => {
      return <input ref={ref} type="text" placeholder="Type something..." />;
    });
    ```
    
    1. **Parent Component**: The parent component uses the `TextInput` and manages focus on it.
    
    ```jsx
    import React, { useRef } from 'react';
    import TextInput from './TextInput'; // Assuming TextInput is in the same directory
    
    const ParentComponent = () => {
      const inputRef = useRef(null);
    
      const focusInput = () => {
        // Focus the input when the button is clicked
        if (inputRef.current) {
          inputRef.current.focus();
        }
      };
    
      return (
        <div>
          <TextInput ref={inputRef} />
          <button onClick={focusInput}>Focus Input</button>
        </div>
      );
    };
    
    export default ParentComponent;
    ```
    
- **Explain useCallback in react in 2 lines**
    
    The `useCallback` hook in React memoizes a function, only recreating it when its dependencies change, which is useful for passing stable functions to child components to prevent unnecessary renders. It returns the same function reference between renders unless dependencies update.
    
    ### Example
    
    ```jsx
    import React, { useState, useCallback } from 'react';
    
    function Counter() { 
      const [count, setCount] = useState(0);
    
      const increment = useCallback(() => {
        setCount((prevCount) => prevCount + 1);
      }, []); // `increment` is memoized and won't change on re-renders
    
      return (
        <div>
          <p>{count}</p>
          <button onClick={increment}>Increment</button>
        </div>
      );
    }
    
    export default Counter;
    ```
    
    In this example, `increment` is memoized, so it retains the same reference across renders unless dependencies (here, none) change.
    
- **Explain useMemo in react in 2 lines**
    
    The **`useMemo` hook in React memoizes a computed value**, recalculating it only when dependencies change, which helps optimize performance by preventing expensive computations on every render. It returns the memoized result, ensuring that heavy calculations or derived values only update when necessary.
    
    ### Example
    
    ```jsx
    import React, { useState, useMemo } from 'react';
    
    function ExpensiveCalculation({ num }) {
      const factorial = useMemo(() => calculateFactorial(num), [num]); // Memoizes result
    
      return <p>Factorial of {num} is {factorial}</p>;
    }
    
    function calculateFactorial(n) {
      console.log('Calculating factorial...');
      return n <= 1 ? 1 : n * calculateFactorial(n - 1);
    }
    
    export default ExpensiveCalculation;
    ```
    
    Here, `useMemo` avoids recalculating `factorial` unless `num` changes, improving render efficiency.
    
- **Controlled and uncontrolled components**
    
    The concept of controlled and uncontrolled components also applies to functional components in React.
    
    In a controlled functional component, the value of an input element is managed by the state of the component. The value of the input element is passed as a prop to the component, and when the value changes, an event handler is called that updates the state of the component with the new value. Here's an example:
    
    ```jsx
    import React, { useState } from 'react';
    
    function ControlledInput() {
      const [value, setValue] = useState('');
    
      const handleChange = (event) => {
        setValue(event.target.value);
      };
    
      return (
        <input type="text" value={value} onChange={handleChange} />
      );
    }
    ```
    
    In this example, the `useState` hook is used to create a state variable `value` and a function `setValue` to update it. The `value` is used as the `value` prop of the `input` element, and the `handleChange` function is called when the value changes to update the state.
    
    In an uncontrolled functional component, the value of an input element is managed by the DOM itself, rather than by React state. The `ref` attribute is used to get a reference to the DOM element, and its value can be accessed through the `current` property of the ref. Here's an example:
    
    In the below example, we store the data of input in ref instead of storing in state. Its helps the app to not re-render again and again. since ref value will not render the UI like setState.
    
    ```jsx
    import React, { useRef } from 'react';
    
    function UncontrolledInput() {
      const inputRef = useRef(null);
    
      const handleSubmit = (event) => {
        console.log(inputRef.current.value);
        event.preventDefault();
      };
    
      return (
        <form onSubmit={handleSubmit}>
          <input type="text" ref={inputRef} />
          <button type="submit">Submit</button>
        </form>
      );
    }
    ```
    
    In this example, the `useRef` hook is used to create a `ref` variable `inputRef`. This `ref` is then passed as the `ref` prop to the `input` element. When the form is submitted, the value of the input element can be accessed through the `current` property of the `ref` object.
    
    In general, controlled components offer more control and flexibility over user input, and are easier to test. However, uncontrolled components can be useful in certain situations, such as when dealing with large forms or when integrating with third-party libraries that require access to the DOM.
    
- Higher-Order Components in 2min
    
    Higher-Order Components (HOCs) in React are functions that take a component and return a new enhanced component, adding reusable logic or behavior without modifying the original component. They’re commonly used for concerns like authentication, logging, or styling across multiple components.
    
    ### Example
    
    ```jsx
    // HOC that adds extra props to a component
    function withExtraProps(WrappedComponent) {
      return function EnhancedComponent(props) {
        return <WrappedComponent {...props} extraProp="Extra Value" />;
      };
    }
    
    function SimpleComponent({ extraProp }) {
      return <div>{extraProp}</div>;
    }
    
    export default withExtraProps(SimpleComponent); // `SimpleComponent` now has `extraProp` from the HOC
    ```
    
    Here, `withExtraProps` is an HOC that injects `extraProp` into `SimpleComponent`.
    
- **Higher Order Components**
    
    Higher Order Components (HOCs) are functions that take a component as an argument and return a new component with additional functionality. Here's an example of creating an HOC in a functional component:
    
    ```jsx
    import React, { useState } from 'react';
    
    function withToggle(Component) {
      return function WrappedComponent(props) {
        const [isOpen, setIsOpen] = useState(false);
    
        const toggle = () => setIsOpen(!isOpen);
    
        return (
          <Component isOpen={isOpen} toggle={toggle} {...props} />
        );
      }
    }
    
    function Menu(props) {
      const { isOpen, toggle } = props;
    
      return (
        <div>
          <button onClick={toggle}>
            {isOpen ? 'Close' : 'Open'} Menu
          </button>
          {isOpen && <ul>
            <li>Home</li>
            <li>About</li>
            <li>Contact</li>
          </ul>}
        </div>
      );
    }
    
    const MenuWithToggle = withToggle(Menu);
    
    function App() {
      return (
        <div>
          <h1>My App</h1>
          <MenuWithToggle />
        </div>
      );
    }
    ```
    
    In this example, the `withToggle` function is an HOC that takes a component (`Menu` in this case) as an argument and returns a new component (`WrappedComponent`) that has additional functionality. The `WrappedComponent` has a new state variable `isOpen` and a function `toggle` that toggles the value of `isOpen`. The `WrappedComponent` passes the `isOpen` and `toggle` props to the original component (`Menu`) and any additional props that are passed to it.
    
    The `Menu` component is the original component that is being wrapped by the HOC. It receives the `isOpen` and `toggle` props from the HOC and uses them to control the visibility of the menu items.
    
    Finally, the `MenuWithToggle` component is the wrapped component that has the additional functionality provided by the `withToggle` HOC. It is used in the `App` component to render the menu with the toggle functionality.
    
    HOCs can be a powerful way to add functionality to your components, especially when you want to reuse that functionality across multiple components. However, they can also add complexity to your code, so it's important to use them judiciously and keep your code as simple as possible.
    
- **Lazy Loading**
    
    Lazy loading is a technique in React that allows you to load components only when they are needed, instead of loading them all up front. This can help improve the performance of your application, especially if you have a large number of components.
    
    Here's an example of lazy loading a component in a functional component using the `React.lazy()` function:
    
    ```jsx
    import React, { lazy, Suspense } from 'react';
    
    const LazyComponent = lazy(() => import('./LazyComponent'));
    
    function App() {
      return (
        <div>
          <h1>My App</h1>
          <Suspense fallback={<div>Loading...</div>}>
            <LazyComponent />
          </Suspense>
        </div>
      );
    }
    ```
    
    Note that lazy loading is currently only supported for default exports (i.e., components that are exported using `export default`). If you need to lazy load a named export, you can use the `import()` function directly, but you won't be able to use the `React.lazy()` function.
    
- **React Events**
    
    Here's a brief explanation of the listed events:
    
    1. `onClick`: This event is triggered when an element is clicked using the mouse or keyboard.
    2. `onDoubleClick`: This event is triggered when an element is double-clicked using the mouse or keyboard.
    3. `onMouseOver`: This event is triggered when the mouse pointer moves over an element.
    4. `onMouseOut`: This event is triggered when the mouse pointer moves out of an element.
    5. `onMouseDown`: This event is triggered when the mouse button is pressed down on an element.
    6. `onMouseUp`: This event is triggered when the mouse button is released on an element.
    7. `onKeyDown`: This event is triggered when a keyboard key is pressed down.
    8. `onKeyUp`: This event is triggered when a keyboard key is released.
    9. `onSubmit`: This event is triggered when a form is submitted.
    10. `onChange`: This event is triggered when the value of an element is changed.
    11. `onLoad`: This event is triggered when an element (e.g. an image) finishes loading.
    12. `onUnload`: This event is triggered when a page is unloaded (e.g. when the user navigates away from the page).
    13. `onResize`: This event is triggered when the size of an element or the window is changed.
    14. `onScroll`: This event is triggered when an element is scrolled.
    15. `onBlur`: This event is triggered when an element loses focus.
    16. `onFocus`: This event is triggered when an element receives focus.
    17. `onTouchStart`: This event is triggered when a touch is initiated on a touch screen device.
    18. `onTouchMove`: This event is triggered when a touch moves on a touch screen device.
    19. `onTouchEnd`: This event is triggered when a touch ends on a touch screen device.
    20. `onTouchCancel`: This event is triggered when a touch is cancelled on a touch screen device (e.g. due to a system event like an incoming call).
- **Error Boundaries**
    
    Error Boundaries in React are special components that can catch errors thrown by their child components during rendering or lifecycle methods, and display a fallback UI instead of crashing the entire application.
    
    Here's an example of using an Error Boundary in a functional component:
    
    ```jsx
    import React, { useState } from 'react';
    
    function ErrorFallback(props) {
      const { error } = props;
    
      return (
        <div>
          <h1>Something went wrong.</h1>
          <p>{error.message}</p>
        </div>
      );
    }
    
    class ErrorBoundary extends React.Component {
      constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
      }
    
      static getDerivedStateFromError(error) {
        return { hasError: true, error };
      }
    
      componentDidCatch(error, errorInfo) {
        console.error(error, errorInfo);
      }
    
      render() {
        const { hasError, error } = this.state;
        const { children } = this.props;
    
        if (hasError) {
          return <ErrorFallback error={error} />;
        }
    
        return children;
      }
    }
    
    function App() {
      const [count, setCount] = useState(0);
    
      const handleClick = () => {
        if (count === 2) {
          throw new Error('I crashed!');
        }
        setCount(count + 1);
      };
    
      return (
        <div>
          <h1>My App</h1>
          <p>Count: {count}</p>
          <button onClick={handleClick}>Increment Count</button>
        </div>
      );
    }
    
    function AppWithErrorBoundary() {
      return (
        <ErrorBoundary>
          <App />
        </ErrorBoundary>
      );
    }
    
    export default AppWithErrorBoundary;
    ```
    
    In this example, we have an `ErrorBoundary` component that wraps the `App` component. The `App` component contains a button that increments a counter, and throws an error when the counter reaches a value of 2.
    
    The `ErrorBoundary` component uses the `static getDerivedStateFromError()` method to catch any errors thrown by its child components, and update the state accordingly. If an error occurs, the `ErrorBoundary` component renders an `ErrorFallback` component that displays a custom error message.
    
    To use the `ErrorBoundary` component, we create a new component called `AppWithErrorBoundary` that wraps the `App` component in an `ErrorBoundary`. This ensures that any errors thrown by the `App` component are caught by the `ErrorBoundary` and don't crash the entire application.
    
    Note that Error Boundaries only catch errors that occur during rendering or lifecycle methods. They won't catch errors that occur during event handling, asynchronous code, or server-side rendering.
    
- **Portals**
    
    Portals in React allow you to render a component's subtree in a different location in the DOM tree, outside of the parent component. This can be useful for scenarios such as modal dialogs or dropdown menus, where you need to render a component outside of its parent's DOM node.
    
    Here's an example of using a portal in a functional component:
    
    ```jsx
    import React, { useState } from 'react';
    import ReactDOM from 'react-dom';
    
    function Modal(props) {
      const { isOpen, onClose } = props;
    
      if (!isOpen) {
        return null;
      }
    
      return ReactDOM.createPortal(
        <div className="modal">
          <div className="modal-content">
            <button className="close-button" onClick={onClose}>
              Close
            </button>
            {props.children}
          </div>
        </div>,
        document.getElementById('modal-root')
      );
    }
    
    function App() {
      const [isOpen, setIsOpen] = useState(false);
    
      const handleOpen = () => setIsOpen(true);
      const handleClose = () => setIsOpen(false);
    
      return (
        <div>
          <h1>My App</h1>
          <button onClick={handleOpen}>Open Modal</button>
          <Modal isOpen={isOpen} onClose={handleClose}>
            <h2>Modal Content</h2>
            <p>This is the content of the modal dialog.</p>
          </Modal>
        </div>
      );
    }
    
    ReactDOM.render(<App />, document.getElementById('root'));
    ```
    
    In this example, we have a simple `Modal` component that renders its children inside a modal dialog. The `Modal` component uses `ReactDOM.createPortal()` to render its content inside a DOM node with the `modal-root` ID, which is outside of the parent component.
    
    The `App` component contains a button that toggles the `isOpen` state, and passes it down to the `Modal` component as a prop. When the `isOpen` state is true, the `Modal` component is rendered using `ReactDOM.createPortal()`.
    
    Note that the `Modal` component is rendered outside of the `App` component's DOM node, but it still behaves as if it were inside it. For example, clicking the `Close` button in the `Modal` component will trigger the `onClose` callback, which will update the `isOpen` state in the `App` component.
    
- **Pure Components**
    
    In React, a pure component is a component that only re-renders when its state or props change. It helps optimize performance by reducing the number of re-renders that occur unnecessarily.
    
    In this example, the `MyComponent` functional component uses the `React.memo` higher-order component to make it a pure component. The `useState` hook is used to manage a count state, and the `handleClick` function is used to increment the count when the button is clicked.
    
    Here's an example of a pure functional component:
    
    ```jsx
    import React, { useState } from "react";
    
    const MyComponent = React.memo(props => {
      const [count, setCount] = useState(0);
    
      const handleClick = () => {
        setCount(count + 1);
      };
    
      console.log("Rendering MyComponent");
    
      return (
        <div>
          <h1>Count: {count}</h1>
          <button onClick={handleClick}>Increment Count</button>
        </div>
      );
    });
    
    export default MyComponent;
    ```
    
    In this example, the `MyComponent` functional component uses the `React.memo` higher-order component to make it a pure component. The `useState` hook is used to manage a count state, and the `handleClick` function is used to increment the count when the button is clicked.
    
    The `console.log` statement is added to show when the component is re-rendered.
    
    By wrapping the component with `React.memo`, it will only re-render when its props or state changes, and not when its parent component re-renders unnecessarily. This helps optimize performance and reduce unnecessary re-renders.
    
- **Stateless and Stateful Component**
    
    In React, components can be divided into two categories: Stateless and Stateful components.
    
    Stateless components are also known as functional components. These components are defined as functions and do not have any state or lifecycle methods. They receive props as input and return JSX elements. Stateless components are usually used for presentational purposes, such as displaying data or rendering a button.
    
    Here's an example of a stateless component:
    
    ```jsx
    function Greeting(props) {
      return <h1>Hello, {props.name}!</h1>;
    }
    ```
    
    Stateful components are also known as class components. These components are defined as classes that extend the `React.Component` class. They have their own internal state and can have lifecycle methods. Stateful components are usually used for more complex functionality, such as handling user input or managing data.
    
    Here's an example of a stateful component:
    
    ```jsx
    class Counter extends React.Component {
      constructor(props) {
        super(props);
        this.state = { count: 0 };
      }
    
      handleClick = () => {
        this.setState({ count: this.state.count + 1 });
      };
    
      render() {
        return (
          <div>
            <p>Count: {this.state.count}</p>
            <button onClick={this.handleClick}>Increment</button>
          </div>
        );
      }
    }
    ```
    
    In summary, stateless components are simple functions that take props as input and return JSX elements, while stateful components are classes that have their own internal state and can have lifecycle methods.
    
- **Flux with example**
    
    
- **Event loop**
    
    In JavaScript, the event loop is a mechanism used to handle asynchronous operations. It is responsible for managing the execution of code and processing of events in a single-threaded environment.
    
    The event loop works by continuously checking the call stack and the task queue. When the call stack is empty, the event loop looks at the task queue for the next task to be processed. Tasks can be added to the task queue using various APIs such as `setTimeout`, `setInterval`, `requestAnimationFrame`, `Promise`, and many others.
    
    When a task is pulled from the task queue, it is pushed onto the call stack to be executed. Once the task is completed, the call stack is cleared, and the event loop looks for the next task in the queue.
    
    The event loop ensures that the JavaScript runtime can handle I/O operations, such as fetching data from a server or reading from a file, without blocking the execution of the rest of the program.
    
    It is important to note that the event loop is a complex and sometimes misunderstood topic in JavaScript. Understanding how it works is crucial for writing efficient and reliable code.
    
    Here is an example of the event loop in JavaScript:
    
    ```jsx
    console.log('start');
    
    setTimeout(function() {
      console.log('setTimeout');
    }, 0);
    
    Promise.resolve().then(function() {
      console.log('Promise');
    });
    
    console.log('end');
    ```
    
    In this example, the console logs "start" and "end" synchronously first. Then, a `setTimeout` function with a delay of 0 milliseconds is called, which means it will be added to the event queue and executed in the next iteration of the event loop. Next, a `Promise` is resolved and a `then` function is called, which will also be added to the event queue and executed in the next iteration of the event loop.
    
    The event loop will check the event queue and execute the functions that are waiting to be executed in the order they were added. So in this example, the event loop will first execute the `setTimeout` function and log "setTimeout" to the console, and then execute the `then` function and log "Promise" to the console. The output of this code will be:
    
    ```jsx
    start
    end
    Promise
    setTimeout
    ```
    
- **Understanding the React Lifecycle with Functional Components and Effects**
    
    In React, the lifecycle of a component consists of three main phases: **mounting**, **updating**, and **unmounting**. However, when dealing with effects in functional components using `useEffect`, it's important to think of the effect's lifecycle independently from the component's lifecycle. Let’s explore this concept in detail.
    
    ### Component Lifecycle Overview
    
    1. **Mounting**: When a component is first added to the DOM.
    2. **Updating**: When the component re-renders due to changes in props or state.
    3. **Unmounting**: When the component is removed from the DOM.
    
    ### Effect Lifecycle
    
    Unlike components, effects in React have a simpler lifecycle. An effect does two things:
    
    - **Start Synchronization (ComponenetDidMount)**: This occurs when the effect runs, usually to sync with an external system or perform side effects like data fetching or subscriptions.
    - **Stop Synchronization (ComponenetDidUnMount)**: This occurs when the effect's dependencies change or the component unmounts, usually to clean up resources or cancel subscriptions.
    
    These two actions can happen multiple times throughout the component's lifecycle depending on changes in the component’s props or state.
    
    ### Example: Chat Room Component
    
    Consider a chat application where you connect to different chat rooms based on a `roomId` prop.
    
    ```jsx
    import { useEffect } from 'react';
    
    const serverUrl = 'https://localhost:1234';
    
    function ChatRoom({ roomId }) {
      useEffect(() => {
        const connection = createConnection(serverUrl, roomId);
        connection.connect();
    
        return () => {
          connection.disconnect();
        };
      }, [roomId]);
    
      return <h1>Welcome to the {roomId} room!</h1>;
    }
    ```
    
    ### How the Effect's Lifecycle Works
    
    1. **Initial Mount (Connecting to a Chat Room)**:
        - When the `ChatRoom` component mounts with `roomId = "general"`, the effect runs:
            - A connection to the "general" chat room is established.
        - The cleanup function is returned but not immediately executed.
    2. **Updating (Switching Chat Rooms)**:
        - Suppose the user selects a new chat room, changing `roomId` to `"travel"`.
        - The effect re-runs because the `roomId` dependency has changed.
        - Before establishing the new connection, React calls the cleanup function from the previous effect, disconnecting from the "general" room.
        - A new connection is established with the "travel" room.
    3. **Unmounting (Disconnecting)**:
        - When the `ChatRoom` component unmounts, the effect's cleanup function is called one last time, disconnecting from the "travel" room.
    
    ### Key Concepts in the Effect Lifecycle
    
    1. **Dependencies**:
        - The `useEffect` hook runs whenever the values in its dependency array change. In this example, the effect depends on `roomId`.
        - If `roomId` changes, the effect will re-run, and the previous effect’s cleanup function will be executed.
    2. **Synchronization**:
        - The effect is responsible for keeping the external system (like a chat server) in sync with the current `roomId` prop.
        - React will ensure that the effect is synchronized correctly by re-running it when dependencies change.
    3. **Reactive Values**:
        - Only values that can change during re-renders, like `props` and `state`, are considered reactive.
        - In the `ChatRoom` example, `roomId` is reactive, so it's included in the dependencies array.
    4. **Empty Dependency Array**:
        - If you provide an empty dependency array `[]`, the effect will run only once, when the component mounts, and the cleanup will run when the component unmounts.
        - This is useful for effects that don't rely on any reactive values.
    
    ### Practical Example
    
    Let’s look at a more practical scenario, where we also include the server URL as a state, making it reactive:
    
    ```jsx
    import { useState, useEffect } from 'react';
    
    function ChatRoom({ roomId }) {
      const [serverUrl, setServerUrl] = useState('https://localhost:1234');
    
      useEffect(() => {
        const connection = createConnection(serverUrl, roomId);
        connection.connect();
    
        return () => {
          connection.disconnect();
        };
      }, [roomId, serverUrl]);
    
      return (
        <>
          <label>
            Server URL:
            <input
              value={serverUrl}
              onChange={e => setServerUrl(e.target.value)}
            />
          </label>
          <h1>Welcome to the {roomId} room!</h1>
        </>
      );
    }
    ```
    
    In this example:
    
    - The effect will re-run whenever either `roomId` or `serverUrl` changes.
    - If the server URL is updated while the chat is open, the connection will reset, ensuring the user is connected to the correct server and room.
    
    ### Conclusion
    
    When working with effects in React, it’s crucial to think of each effect independently from the component lifecycle. Focus on starting and stopping synchronization processes based on the effect's dependencies. This mindset helps you write resilient, maintainable code that behaves predictably as your component's props and state change.
    
- **Custom Hooks**
    
    ### Understanding Custom Hooks in React
    
    React provides several built-in hooks like `useState`, `useEffect`, and `useContext` that help manage state and side effects in functional components. However, as your application grows, you might find yourself writing the same logic across multiple components. To avoid code duplication and make your code more reusable, you can create **Custom Hooks**.
    
    ### What is a Custom Hook?
    
    A **Custom Hook** is a JavaScript function that uses one or more of React’s built-in hooks and encapsulates some reusable logic that can be shared between components. Custom Hooks follow the naming convention of starting with the word `use` (e.g., `useOnlineStatus`) and can return any type of value (state, functions, objects, etc.).
    
    ### Why Use Custom Hooks?
    
    1. **Code Reusability**: Extract reusable logic into a Custom Hook to avoid duplicating code across multiple components.
    2. **Separation of Concerns**: Custom Hooks allow you to separate logic (like fetching data or handling events) from the component’s UI, making your components more focused and easier to maintain.
    3. **Simplifying Components**: By moving logic into a Custom Hook, you can keep your components simpler and more readable.
    
    **Conclusion**
    
    Custom Hooks are a powerful way to share logic between components in React without repeating code. By encapsulating logic in Custom Hooks, you make your components more focused, maintainable, and reusable, leading to cleaner and more efficient code.
    
- **Custom Hook Example `useFetchData`**
    
    Let's consider a scenario where you need to fetch data from an API and manage the loading state, errors, and the fetched data. This logic is common in many components, so it's ideal to encapsulate it in a custom hook.
    
    **Example:**
    
    ```jsx
    import { useState, useEffect } from 'react';
    
    // Custom Hook for Fetching Data
    function useFetchData(url) {
      const [data, setData] = useState(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);
    
      useEffect(() => {
        let isMounted = true; //To avoid setting state on unmounted component
    
        fetch(url)
          .then((response) => {
            if (!response.ok) {
              throw new Error('Network response was not ok');
            }
            return response.json();
          })
          .then((data) => {
            if (isMounted) {
              setData(data);
              setLoading(false);
            }
          })
          .catch((error) => {
            if (isMounted) {
              setError(error);
              setLoading(false);
            }
          });
    
        return () => {
          isMounted = false;
        };
      }, [url]);
    
      return { data, loading, error };
    }
    
    // Component using the useFetchData Hook
    function UserList() {
      const { data, loading, error } = useFetchData('https://api.example.com/users');
    
      if (loading) return <p>Loading...</p>;
      if (error) return <p>Error: {error.message}</p>;
    
      return (
        <ul>
          {data.map((user) => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      );
    }
    
    export default UserList;
    ```
    
    **Explanation:**
    
    - The `useFetchData` hook encapsulates the logic for fetching data from a given URL.
    - It handles three states: `data`, `loading`, and `error`, which are crucial for robust data fetching.
    - The `UserList` component can then use this hook without worrying about the intricacies of data fetching.
- **Why use custom hook instead of normal javascript function**
    
    Using a custom hook like `useFetchData` in React has several advantages over a regular JavaScript function. Here are the key reasons why you would choose a custom hook:
    
    ### 1. **State Management**
    
    - **Custom Hook**: A custom hook allows you to encapsulate and manage state (using `useState`) related to the fetch operation, such as `data`, `loading`, and `error`.
    - **Normal Function**: A regular function cannot maintain internal state in the context of a React component, making it less suitable for managing dynamic data.
    
    ### 2. **Lifecycle Management**
    
    - **Custom Hook**: You can leverage React’s lifecycle methods with hooks like `useEffect` to handle side effects, such as making an API call when the component mounts or when the URL changes.
    - **Normal Function**: A standard function does not have access to the component lifecycle, which makes it difficult to handle asynchronous operations properly.
    
    ### 3. **Reusability**
    
    - **Custom Hook**: Custom hooks can be reused across multiple components, making your code DRY (Don't Repeat Yourself) and easier to maintain. You can use the same logic in different components without duplicating code.
    - **Normal Function**: While you can reuse a normal function, it won’t be able to manage state or lifecycle in the way a custom hook does, which limits its usefulness in a React context.
    
    ### 4. **Integration with React's Rendering**
    
    - **Custom Hook**: Custom hooks integrate seamlessly with React's rendering flow. When a hook's state changes, it triggers a re-render of the component that uses it.
    - **Normal Function**: A regular function cannot cause a re-render when its internal state changes, as it doesn't operate within React's context.
    
    ### 5. **Encapsulation of Logic**
    
    - **Custom Hook**: A custom hook encapsulates specific logic related to fetching data. This makes it easier to reason about the component's behavior and separate concerns.
    - **Normal Function**: A regular function might not provide the same level of encapsulation since it does not have the same hooks and state management capabilities.
    
    ### Example Scenario
    
    Suppose you have multiple components that need to fetch user data from an API. With a custom hook, you can easily share the fetching logic across those components:
    
    ### Using Custom Hook
    
    ```jsx
    // Custom Hook
    function useFetchData(url) {
      // State and effect logic here...
    }
    
    // Component 1
    function UserList() {
      const { data, loading, error } = useFetchData('https://api.example.com/users');
      // Render logic here...
    }
    
    // Component 2
    function AdminList() {
      const { data, loading, error } = useFetchData('https://api.example.com/admins');
      // Render logic here...
    }
    
    ```
    
    ### Using Normal Function
    
    If you were to use a normal function, you might end up duplicating the fetching logic across different components, which would violate the DRY principle:
    
    ```jsx
    function fetchData(url) {
      // Fetch logic here...
    }
    
    // Component 1
    function UserList() {
      const data = fetchData('https://api.example.com/users');
      // Render logic here...
    }
    
    // Component 2
    function AdminList() {
      const data = fetchData('https://api.example.com/admins');
      // Render logic here...
    }
    
    ```
    
    ### Conclusion
    
    In summary, custom hooks provide a powerful way to manage state and side effects in a reusable, encapsulated manner that aligns with React's declarative nature. This makes them far superior to standard JavaScript functions for managing complex logic in React applications.
    
- **Custom Hook for UI Manipulation: `useFadeIn`**
    
    Imagine you want to create a fade-in animation for a component when it mounts. This is a common UI effect that can be reused in multiple components. To implement this, you can create a `useFadeIn` custom hook.
    
    **Example:**
    
    ```jsx
    import { useState, useEffect, useRef } from 'react';
    
    // Custom Hook for Fade-In Animation
    function useFadeIn(duration = 1000) {
      const elementRef = useRef(null);
      const [opacity, setOpacity] = useState(0);
    
      useEffect(() => {
        const element = elementRef.current;
        let start = null;
    
        function fadeIn(timestamp) {
          if (!start) start = timestamp;
          const progress = timestamp - start;
          const newOpacity = Math.min(progress / duration, 1);
          setOpacity(newOpacity);
          element.style.opacity = newOpacity;
    
          if (progress < duration) {
            requestAnimationFrame(fadeIn);
          }
        }
    
        requestAnimationFrame(fadeIn);
      }, [duration]);
    
      return elementRef;
    }
    
    // Component using the useFadeIn Hook
    function Welcome() {
      const fadeInRef = useFadeIn(2000);
    
      return (
        <h1 ref={fadeInRef} style={{ opacity: 0 }}>
          Welcome to the App!
        </h1>
      );
    }
    
    export default Welcome;
    
    ```
    
    **Explanation:**
    
    - The `useFadeIn` hook handles the logic for animating the opacity of a DOM element.
    - It uses `useRef` to reference the DOM node and `useEffect` to manage the animation with `requestAnimationFrame`.
    - The component that uses this hook (`Welcome`) simply applies it, and the hook takes care of the rest.
- **State Managers Are Making Your Code Worse in React Applications**
    
    In the world of React development, there are numerous state management libraries available, like Redux, Zustand, and Jotai. While these tools were essential in the early days of React, recent advancements in React and related technologies have made many of these libraries less necessary, especially for most applications.
    
    ### **The Problem of Prop Drilling**
    
    When React was first introduced, state was managed locally within components. However, as applications grew in complexity, passing state from parent components to deeply nested child components became problematic—a process known as "prop drilling." This led to messy and hard-to-maintain code, as state had to be passed through components that didn't even need it, just to reach a deeply nested component that did.
    
    ### **Introduction of Global State Management Libraries**
    
    To solve prop drilling, developers started using global state management libraries like Redux. These libraries allowed state to be stored in a centralized "store," accessible from any component in the application, eliminating the need for prop drilling. Redux, in particular, became popular because it provided a structured way to manage complex, interdependent state through the use of reducers and actions.
    
    However, Redux and similar libraries introduced their own complexities, particularly in the form of boilerplate code, which made them cumbersome for simple applications.
    
    ### **React's Built-in Solutions: Context API and Hooks**
    
    React introduced the Context API and hooks like `useContext` and `useReducer` to address state management issues without needing external libraries. The Context API allows developers to create global state that can be accessed by any component within a specific tree, effectively solving the prop drilling problem. The `useReducer` hook brings in the concept of reducers from Redux, allowing for complex state logic within a React component.
    
    ### **Shift to Meta Frameworks and Server-side Rendering**
    
    With the rise of meta frameworks like Next.js, which combine backend and frontend logic, much of the state management complexity has been offloaded to the server. Server components in Next.js, for example, allow data to be fetched and rendered server-side, reducing the need for client-side state management. This shift has made it easier to manage state, as developers can now rely on server-side logic for tasks that previously required complex client-side state management.
    
    ### **Modern Best Practices**
    
    For most applications, the built-in tools React offers, such as `useState`, `useReducer`, and `useContext`, combined with modern practices like using URLs to manage state (e.g., storing filter options in query parameters), are sufficient. These practices simplify state management and reduce the need for heavyweight libraries like Redux.
    
    For example, an e-commerce site with features like coupon application and product filtering can be built using minimal state management, primarily relying on the URL for state management and server-side rendering for data fetching.
    
    ### **When to Use State Management Libraries**
    
    State management libraries are still useful in highly complex applications, like social media platforms or large-scale web apps, where state is shared across many components and pages. However, for most projects, starting with React's built-in tools and only introducing a state management library if absolutely necessary is a more efficient approach.
    
    In summary, while state management libraries like Redux were once essential, modern React features and meta frameworks have made them less critical, allowing for simpler and more maintainable code in most applications.
    
- **Get Api Call in React using fetch & axios**
    
    ```jsx
    import React, { useState, useEffect } from 'react';
    
    const DataFetchingComponent = () => {
      const [data, setData] = useState(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);
    
      useEffect(() => {
        // Function to fetch data
        const fetchData = async () => {
          try {
    	      // using fetch
            const response = await fetch('https://api.example.com/data');
            if (!response.ok) {
              throw new Error('Network response was not ok');
            }
            const result = await response.json();
            setData(result);
            
            
            // Using Axios
            // const response = await axios.get('https://api.example.com/data');
            // setData(response.data);
            
          } catch (error) {
            setError(error.message);
          } finally {
            setLoading(false);
          }
        };
    
        // Call the fetch function
        fetchData();
      }, []); // Empty dependency array means this effect runs once when the component mounts
    
      // Conditional rendering based on loading and error states
      if (loading) return <p>Loading...</p>;
      if (error) return <p>Error: {error}</p>;
    
      // Render the fetched data
      return (
        <div>
          <h1>Data</h1>
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      );
    };
    
    export default DataFetchingComponent;
    
    ```
    
- **Why useContext is more then enough for medium size project and why its bad to use redux on medium size project. provide real time scenario with example**
    
    Using `useContext` for state management in a medium-sized project can often be more than enough because it offers simplicity and ease of use, without the overhead that comes with a more complex state management library like Redux. However, Redux can be overkill for medium-sized projects due to its boilerplate code, additional setup, and complexity. Let’s dive deeper into the reasons and provide real-world scenarios.
    
    ### **Why `useContext` is Enough for Medium-Sized Projects**
    
    1. **Simplicity and Readability**:
        - `useContext` is built into React and requires no additional libraries. It provides a straightforward way to share state across multiple components.
        - It makes your code more readable because you don’t need to define actions, reducers, and dispatchers as you would in Redux.
    2. **Reduced Boilerplate**:
        - With `useContext`, you avoid the boilerplate code associated with Redux (like creating action types, action creators, reducers, and setting up a store).
        - This makes your codebase cleaner and easier to maintain.
    3. **Performance**:
        - For medium-sized projects, the performance overhead introduced by using Redux may not be justified. `useContext` can efficiently manage the state without the extra complexity.
    4. **Learning Curve**:
        - `useContext` has a much lower learning curve than Redux. Developers familiar with React can quickly grasp `useContext` and `useReducer` (if more complex state logic is needed).
        - This is beneficial when onboarding new developers to the project.
    
    ### **Real-World Scenario: Using `useContext`**
    
    Imagine a medium-sized project, such as a blog platform where users can view, create, and manage blog posts. The application has the following components:
    
    - `Header` (for navigation)
    - `PostList` (for displaying a list of posts)
    - `PostDetails` (for viewing a single post)
    - `UserProfile` (for displaying and editing the user’s profile)
    
    In this scenario, global state might include:
    
    - The current authenticated user
    - A list of posts
    - The selected post
    
    ### **Implementing with `useContext`**
    
    1. **Create a Context for the User State**:
        
        ```jsx
        jsxCopy code
        import React, { createContext, useContext, useState } from 'react';
        
        const UserContext = createContext();
        
        export const UserProvider = ({ children }) => {
          const [user, setUser] = useState(null);
        
          return (
            <UserContext.Provider value={{ user, setUser }}>
              {children}
            </UserContext.Provider>
          );
        };
        
        export const useUser = () => useContext(UserContext);
        
        ```
        
    2. **Using the Context in Components**:
        
        ```jsx
        jsxCopy code
        import React from 'react';
        import { useUser } from './UserContext';
        
        const UserProfile = () => {
          const { user, setUser } = useUser();
        
          const handleLogout = () => {
            setUser(null);
          };
        
          return (
            <div>
              <h1>{user.name}'s Profile</h1>
              <button onClick={handleLogout}>Logout</button>
            </div>
          );
        };
        
        export default UserProfile;
        
        ```
        
    
    This setup with `useContext` is straightforward and efficient for a medium-sized project where the state management requirements are not overly complex.
    
    ### **Why Redux Can Be Overkill for Medium-Sized Projects**
    
    1. **Complexity**:
        - Redux introduces a higher level of complexity with its strict patterns (actions, reducers, middleware).
        - It requires developers to follow specific conventions, making the codebase more complex than it might need to be for a medium-sized project.
    2. **Boilerplate Code**:
        - The boilerplate code in Redux can be overwhelming. For every state change, you need to define action types, create action creators, write reducers, and dispatch actions.
        - This increases the amount of code you need to write and maintain, which might not be necessary for a medium-sized project.
    3. **Performance Considerations**:
        - While Redux is efficient for managing large and complex state trees, it may introduce unnecessary overhead in medium-sized projects where `useContext` and `useReducer` could handle the same tasks with less complexity.
    
    ### **Real-World Scenario: Overhead with Redux**
    
    Consider the same blog platform example. If you were to use Redux, you would need:
    
    - Action types for fetching, adding, and deleting posts.
    - Action creators for each action.
    - A reducer to handle the different actions and update the state accordingly.
    - A store to manage the global state.
    - Potential middleware to handle side effects like API calls.
    
    This setup, while powerful, is excessive for a project where the state is relatively simple and does not require the advanced features that Redux provides.
    
    ### **When Redux Might Be Necessary**
    
    That said, there are scenarios where Redux might be necessary, such as:
    
    - Large-scale applications with complex state management requirements.
    - Applications where you need to persist state across sessions, synchronize state with local storage, or share state between deeply nested components that are not directly related.
    - Projects requiring sophisticated debugging tools (e.g., Redux DevTools) or where state changes need to be predictable and traceable.
    
    ### **Conclusion**
    
    For a medium-sized project, `useContext` is usually more than enough due to its simplicity, ease of use, and minimal setup. On the other hand, Redux might be overkill, introducing unnecessary complexity and boilerplate. By choosing the right tool for the job, you can keep your codebase clean, maintainable, and performant.
    
- **Why redux is required for big react project why useContext is not a good fit in big react projects. provide real time scenario with example**
    
    Redux is often preferred in large React projects over `useContext` due to its ability to handle complex state management scenarios, provide scalability, and offer tools for debugging and maintaining consistency across the application. While `useContext` works well for smaller and medium-sized projects, it has limitations that can make it less suitable for larger applications. Let's explore why Redux is required for big projects and when `useContext` falls short.
    
    ### **Limitations of `useContext` in Large Projects**
    
    1. **Scalability Issues**:
        - In large applications, state management can become complex, with multiple components needing access to different parts of the state. `useContext` can lead to performance issues because every component that consumes the context will re-render whenever the context value changes, even if the component doesn’t need to update.
        - Handling deeply nested components can be cumbersome with `useContext`, as prop drilling can become a problem.
    2. **Lack of Middleware Support**:
        - `useContext` doesn’t offer built-in support for middleware, which is often necessary in large projects to handle side effects like logging, asynchronous operations, or routing.
        - Implementing complex features like optimistic updates, caching, and undo functionality can be challenging without middleware.
    3. **Debugging and DevTools**:
        - Redux provides powerful DevTools for tracking state changes, time-travel debugging, and inspecting dispatched actions. `useContext` lacks these out-of-the-box capabilities, making debugging more difficult in large applications.
    4. **Single Context Limitation**:
        - Managing multiple contexts with `useContext` can become unmanageable. As the number of contexts increases, the code can become difficult to follow and maintain.
        - In contrast, Redux allows for a more centralized state management approach, which can be easier to scale and maintain.
    
    ### **Why Redux is Better for Large Projects**
    
    1. **Centralized State Management**:
        - Redux stores the entire application state in a single, centralized store. This makes it easier to manage and reason about the state, especially in large applications where state changes occur frequently across different parts of the app.
    2. **Predictability and Immutability**:
        - Redux enforces a predictable state management pattern through actions and reducers. This predictability is crucial in large applications where multiple developers are working on the same codebase.
        - The immutability of the state in Redux ensures that state changes are traceable and controlled, reducing the likelihood of bugs and unintended side effects.
    3. **Middleware and Side Effects**:
        - Redux supports middleware like `redux-thunk` or `redux-saga`, which are essential for handling side effects, such as API calls, logging, and asynchronous actions. This makes it easier to manage complex logic that spans multiple parts of the application.
        - Middleware also allows for extending Redux with custom behaviors, making it more flexible and powerful.
    4. **Debugging and Tooling**:
        - Redux DevTools provide extensive capabilities for inspecting the state, viewing action history, and even rolling back to previous states. This is particularly valuable in large projects where tracking down bugs can be time-consuming.
        - Time-travel debugging allows developers to step through state changes and understand how a particular state was reached.
    5. **Modular and Scalable Architecture**:
        - Redux promotes a modular approach to state management through reducers and actions, which can be split into smaller, manageable pieces. This makes it easier to scale the application as it grows.
        - The architecture encourages separation of concerns, with clear boundaries between state, actions, and views.
    
    ### **Real-World Scenario: Why Redux is Necessary**
    
    **Scenario: E-Commerce Platform**
    
    Consider a large e-commerce platform with the following features:
    
    - **Product Listing and Search**: Users can browse through thousands of products, filter by categories, and search for specific items.
    - **Shopping Cart**: Users can add, remove, and update items in their cart. The cart state needs to be synchronized across different parts of the app (e.g., product pages, checkout page, header).
    - **User Authentication**: The app needs to manage user authentication state, handle login/logout, and store user information securely.
    - **Order Management**: The platform needs to manage order history, track order status, and handle cancellations and returns.
    - **Real-Time Notifications**: Users receive real-time notifications for price drops, order status updates, and promotional offers.
    
    ### **Implementing with Redux**
    
    1. **Centralized State Management**:
        - The global state might include the user’s authentication status, cart items, product list, filters, and search criteria. All of this can be managed efficiently in a single Redux store.
    2. **Predictability and Immutability**:
        - Actions like `ADD_TO_CART`, `REMOVE_FROM_CART`, `LOGIN_SUCCESS`, and `LOGOUT` are handled by reducers that ensure the state transitions are predictable and immutable.
    3. **Middleware for Side Effects**:
        - Middleware like `redux-thunk` can be used to handle asynchronous operations, such as fetching product data from an API or processing payment during checkout.
        - Redux’s middleware makes it easy to implement complex logic, like optimistic updates (e.g., showing an item as added to the cart before the API confirms the addition).
    4. **Debugging with DevTools**:
        - Redux DevTools allow developers to monitor the state of the cart, user authentication, and product listings in real-time. They can also roll back state changes if a bug occurs, making it easier to debug issues in a large codebase.
    5. **Modular Architecture**:
        - The state is divided into smaller slices (e.g., `cartReducer`, `userReducer`, `productReducer`), each responsible for a specific part of the state. This modular approach makes it easier to maintain and scale the application.
    
    ### **Limitations of `useContext` in this Scenario**
    
    If you were to implement this e-commerce platform with `useContext`, you would face several challenges:
    
    - **Performance Issues**: The context provider for the cart, user, or products would need to be accessed by many components. Any change in these contexts could trigger unnecessary re-renders, leading to performance bottlenecks.
    - **Complex State Management**: Managing complex asynchronous operations (e.g., API calls) without middleware would require custom solutions, making the codebase more difficult to maintain.
    - **Debugging Challenges**: Without Redux DevTools, tracking state changes and debugging issues would be much harder, especially in a large application with many interconnected components.
    
    ### **Conclusion**
    
    In large React projects, Redux is often necessary due to its centralized state management, predictability, middleware support, and powerful debugging tools. `useContext`, while useful in smaller projects, can become unwieldy and inefficient in complex applications. By choosing Redux for large projects, developers can maintain a scalable, maintainable, and performant codebase.
    
- **React context api vs use context**
    
    In React, the **Context API** and the **useContext** hook are closely related but serve different roles in managing shared state across the component tree. To understand their relationship and differences, let’s break them down:
    
    ### **1. React Context API**
    
    The **Context API** is a system in React that enables you to share values (or state) across components without having to pass props down manually at every level of the component tree. It is especially useful for managing global state or theme, authentication, or language settings that need to be accessed by deeply nested components.
    
    ### Key Components of Context API:
    
    - **Context Provider**: Provides the value to the subtree of components that need access to it.
    - **Context Consumer**: Any component that needs to access the context can consume the value provided by the Provider.
    
    ### **How Context API Works:**
    
    You create a context using `React.createContext()`, and the Provider component is used to "provide" the context to all its descendant components.
    
    ### Example:
    
    ```jsx
    jsxCopy code
    import React, { createContext, useState } from 'react';
    
    // Create a Context
    const ThemeContext = createContext();
    
    function App() {
      const [theme, setTheme] = useState('light');
    
      return (
        // Use the Provider to supply the context value
        <ThemeContext.Provider value={{ theme, setTheme }}>
          <Toolbar />
        </ThemeContext.Provider>
      );
    }
    
    function Toolbar() {
      return (
        <div>
          <ThemeButton />
        </div>
      );
    }
    
    function ThemeButton() {
      return (
        <ThemeContext.Consumer>
          {({ theme, setTheme }) => (
            <button
              onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
            >
              {theme === 'light' ? 'Switch to Dark' : 'Switch to Light'}
            </button>
          )}
        </ThemeContext.Consumer>
      );
    }
    
    ```
    
    Here, `ThemeContext.Provider` supplies the `theme` state and `setTheme` function to any component that consumes the context. `ThemeContext.Consumer` is used to consume the context in `ThemeButton`.
    
    ### **2. useContext Hook**
    
    The **useContext** hook is a more convenient and modern way to consume context in functional components. It simplifies the process of consuming context by eliminating the need for a `Context.Consumer` component. It allows you to access the context value directly inside any function component.
    
    ### Example with `useContext` Hook:
    
    ```jsx
    jsxCopy code
    import React, { createContext, useState, useContext } from 'react';
    
    // Create a Context
    const ThemeContext = createContext();
    
    function App() {
      const [theme, setTheme] = useState('light');
    
      return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
          <Toolbar />
        </ThemeContext.Provider>
      );
    }
    
    function Toolbar() {
      return (
        <div>
          <ThemeButton />
        </div>
      );
    }
    
    function ThemeButton() {
      // useContext hook to consume the context
      const { theme, setTheme } = useContext(ThemeContext);
    
      return (
        <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
          {theme === 'light' ? 'Switch to Dark' : 'Switch to Light'}
        </button>
      );
    }
    
    ```
    
    In this example, instead of using `ThemeContext.Consumer`, the `useContext` hook is used to access `theme` and `setTheme` inside the `ThemeButton` function component. This approach is more concise and readable.
    
    ### **Comparison:**
    
    | **Aspect** | **React Context API (Context.Consumer)** | **useContext Hook** |
    | --- | --- | --- |
    | **Usage** | Uses `Context.Consumer` to access the context value. | Uses `useContext` to access the context value directly. |
    | **Component Type** | Works with both class and functional components. | Only works with functional components. |
    | **Readability** | Less readable as it uses a function as a child in the consumer. | More concise and easier to read, no need for function-as-child. |
    | **Simplicity** | Requires extra boilerplate (Consumer component). | Simpler and cleaner, directly consumes the context. |
    | **Updates** | Both update whenever the context value changes. | Both update whenever the context value changes. |
    | **Code Complexity** | More verbose, especially with nested components. | Easier and more declarative, perfect for function components. |
    | **Class Components** | Can be used in class components with `Context.Consumer`. | Cannot be used in class components (functional only). |
    
    ### **When to Use the Context API vs. useContext:**
    
    - **Context API (`Context.Consumer`)**: Useful if you're still working with **class components** or need to access context in a class-based architecture. It is also used when you need to customize context behavior, for instance, when working with multiple consumers or handling context nesting.
    - **useContext Hook**: If you're working in **functional components** (as is common in modern React), `useContext` is the preferred way. It provides cleaner and more readable code by allowing direct access to context values without the need for an extra wrapper component.
    
    ### **Real-World Scenario:**
    
    Imagine you're building a theme-switching application where you want to toggle between light and dark modes. With the Context API, you'd typically wrap your application in a `ThemeProvider` that provides the current theme and a function to switch it. With `useContext`, you'd access that theme data directly in any functional component.
    
    - **Context API**: May still use `Context.Consumer` if you're in a project that uses class components.
    - **useContext Hook**: In a modern functional component-based React project, you'd likely use `useContext` for this task, as it's cleaner and more in line with React's hooks philosophy.
    
    ### **Conclusion:**
    
    - **Context API** provides a way to pass down data without prop drilling, and `Context.Consumer` can be used in both class and function components.
    - **useContext** is a simpler and more direct approach to consuming context, but it only works with functional components.
    
    In modern React development, **useContext** is often the better choice due to its simplicity and the popularity of functional components. However, both are important parts of React's context system.
    

---

### JavaScript

- **Event Delegation**
    
    Event delegation is a technique where you attach a single event listener to a parent element that handles events for all its children through event bubbling, rather than attaching individual listeners to each child element.
    
    ```jsx
    import React from "react";
    
    const App = () => {
    	// This handleClick will trigger the added event in div elemet
    	// In our case its onMouseOver, but we can specify
      const handleClick = (event) => {
        if (event.target.tagName === "BUTTON") {
          console.log(`Button clicked: ${event.target.textContent}`);
        }
      };
    
      return (
        <div onMouseOver={handleClick}>
          <button>Button 1</button>
          <button>Button 2</button>
          <button>Button 3</button>
        </div>
      );
    };
    export default App;
    ```
    
- **Event Bubbling**
    
    Event bubbling is a DOM event propagation mechanism where events triggered on a nested element automatically "bubble up" through the hierarchy of parent elements, executing handlers on each ancestor element in sequence.
    
    ```html
    <div id="parent">
      <div id="child">
        <button id="btn">Click me</button>
      </div>
    </div>
    ```
    
    If a user clicks on the button element, a "click" event will be triggered on the button. However, because of event bubbling, the "click" event will also trigger any event listeners attached to the child and parent elements.
    
    Here's an example of how event bubbling can be used in JavaScript:
    
    ```jsx
    document.getElementById("parent").addEventListener("click", function() {
      console.log("Parent clicked");
    });
    
    document.getElementById("child").addEventListener("click", function() {
      console.log("Child clicked");
    });
    
    document.getElementById("btn").addEventListener("click", function() {
      console.log("Button clicked");
    });
    ```
    
    In this example, if the user clicks on the button, the following output will be logged to the console:
    
    ```css
    Button clicked
    Child clicked
    Parent clicked
    ```
    
    This is because the "click" event bubbles up from the button to the child and parent elements, triggering each of their event listeners in turn.
    
- **Object Copy In Javascript**
    
    
    | **Method** | **Pros** | **Cons** |
    | --- | --- | --- |
    | [**shallow copy with `=`**](https://code.tutsplus.com/articles/the-best-way-to-deep-copy-an-object-in-javascript--cms-39655#shallow) | clear and direct, the default | only shallow copies objects |
    | [**`JSON.stringify()` and `JSON.parse()`**](https://code.tutsplus.com/articles/the-best-way-to-deep-copy-an-object-in-javascript--cms-39655#json) | deep copies nested objects | doesn't copy functions |
    | `const user = { name: "Kingsley", age: 28, job: "Web Developer", incrementAge: function() { this.age++ } }` 
    
    `let clone = Object.assign({}, user)` | copies the immediate members of an object—including functions | doesn't deep copy nested objects |
    | [**the `...` spread operator**](https://code.tutsplus.com/articles/the-best-way-to-deep-copy-an-object-in-javascript--cms-39655#spread) | simple syntax, the preferred way to copy an object | doesn't deep copy nested objects |
    | [**Lodash `cloneDeep()`**](https://code.tutsplus.com/articles/the-best-way-to-deep-copy-an-object-in-javascript--cms-39655#lodash) | clones nested objects including functions | adds an external dependency to your project |
- **Attribute & Property**
    
    In HTML, attributes and properties are used to define and manipulate the characteristics of an element. However, they differ in how they are accessed and their underlying functionality.
    
    An attribute is a string value that is defined in HTML and sets a value for an element's characteristic. It is used to define an element's initial state, and can be changed by the user, the browser, or a script. Attribute values are case-insensitive, and multiple attributes can be used to define an element.
    
    A property, on the other hand, is an object property that represents the current state of an element's attribute. It is a direct reference to the attribute value, and changes to the property update the attribute's value. Property values are case-sensitive, and there can only be one property for each element.
    
    Here is an example to illustrate the difference between attributes and properties in HTML and JavaScript:
    
    HTML code:
    
    ```html
    <div id="example" class="blue"></div>
    ```
    
    JavaScript code:
    
    ```jsx
    var el = document.getElementById('example');
    
    // Get attribute value
    var attrVal = el.getAttribute('class');
    console.log(attrVal); // Output: 'blue'
    
    // Get property value
    var propVal = el.className;
    console.log(propVal); // Output: 'blue'
    
    // Update attribute value
    el.setAttribute('class', 'red');
    console.log(el.getAttribute('class')); // Output: 'red'
    
    // Update property value
    el.className = 'green';
    console.log(el.className); // Output: 'green'
    
    // Note that the attribute value remains unchanged
    console.log(el.getAttribute('class')); // Output: 'red'
    ```
    
    In this example, the `div` element has an `id` attribute and a `class` attribute with the value of 'blue'. The JavaScript code retrieves the values of these attributes using the `getAttribute()` method and the `className` property, respectively.
    
    The example also shows how to update the attribute and property values. The `setAttribute()` method changes the attribute value, while changing the `className` property updates the property value. Note that changing the property value does not change the attribute value.
    
    ```jsx
    // <input type="text" value="Hello">
    // console.log(input.getAttribute('value')) // Output: Hello
    ```
    
- **DOMContentLoad vs Load**
    
    In JavaScript, `DOMContentLoaded` and `load` are two events that are fired during the loading of a webpage.
    
    `DOMContentLoaded` event is triggered when the HTML document has been completely loaded and parsed, without waiting for images, scripts, and other resources to be loaded. This means that the DOM tree is ready and can be manipulated using JavaScript.
    
    `load` event is triggered when all the resources on the page have been loaded, including images, scripts, and stylesheets. This means that the entire page, including all its dependencies, has been loaded and is ready to be used.
    
    Here is an example of how to use `DOMContentLoaded` and `load` events:
    
    ```html
    <!DOCTYPE html>
    <html>
    <head>
    	<title>DOMContentLoaded vs Load</title>
    </head>
    <body>
    	<h1 id="heading">JavaScript DOMContentLoaded vs Load</h1>
    
    	<script>
    		document.addEventListener("DOMContentLoaded", function() {
    		    console.log("DOM is loaded");
    		    var heading = document.getElementById("heading");
    		    heading.style.color = "red";
    		});
    
    		window.addEventListener("load", function() {
    		    console.log("All resources finished loading");
    		});
    	</script>
    </body>
    </html>
    ```
    
    In the above example, the `DOMContentLoaded` event is used to change the color of the heading element to red once the DOM has been loaded. The `load` event is used to log a message to the console once all the resources on the page have been loaded
    
- **Mono-repository**
    
    Not Sure the below answer is correct or not
    
    A Mono-repository, also known as a Monorepo, is a software development strategy where multiple projects are managed within a single repository, rather than using a separate repository for each project. In the context of HTML and JavaScript, a Monorepo could contain multiple web applications or components, along with shared libraries, tools, and configuration files.
    
    The advantages of using a Monorepo include:
    
    1. Simplified development workflows: Developers can work on multiple projects within the same repository, making it easier to manage dependencies and share code between projects.
    2. Consistent versioning: When multiple projects are managed in separate repositories, it can be challenging to ensure that all projects are using the same version of a shared library or tool. With a Monorepo, it is easier to ensure that all projects are using the same version of a dependency.
    3. Easier code sharing: With a Monorepo, it is easier to share code between projects since everything is managed within the same repository.
    4. Better code organization: Monorepos make it easier to organize code across multiple projects since all projects are managed within the same repository.
    
    Some popular tools for managing Monorepos in HTML and JavaScript include Yarn Workspaces, Lerna, and Nx. These tools provide features such as dependency management, versioning, and building and testing multiple projects within the same repository.
    
- **Polyfills**
    
    Polyfill is a piece of code (usually JavaScript) that provides modern functionality on older browsers that don't natively support it. Essentially, a polyfill is a piece of code that "fills in" the gaps in older browsers to make them compatible with newer web standards. Here's an example of using a polyfill to add support for the `Array.prototype.includes()` method, which was introduced in ECMAScript 2016:
    
    ```jsx
    // Check if Array.prototype.includes is already supported by the browser
    if (!Array.prototype.includes) {
      // Define a polyfill for Array.prototype.includes
      Object.defineProperty(Array.prototype, 'includes', {
        value: function(valueToFind, fromIndex) {
          if (this == null) {
            throw new TypeError('"this" is null or undefined');
          }
          var o = Object(this);
          var len = o.length >>> 0;
          if (len === 0) {
            return false;
          }
          var n = fromIndex | 0;
          var k = Math.max(n >= 0 ? n : len - Math.abs(n), 0);
          while (k < len) {
            if (o[k] === valueToFind) {
              return true;
            }
            k++;
          }
          return false;
        }
      });
    }
    
    // Now we can use Array.prototype.includes() in our code
    var fruits = ['apple', 'banana', 'orange'];
    console.log(fruits.includes('banana')); // Output: true
    console.log(fruits.includes('grape')); // Output: false
    ```
    
    In this example, the code first checks whether the browser supports `Array.prototype.includes()` by checking if it exists. If it doesn't exist, a polyfill is defined using `Object.defineProperty()` to add the method to the `Array.prototype` object. This polyfill implementation checks if the array includes a given value and returns a boolean value accordingly.
    
- **Shadow Dom**
    
    Shadow DOM is a web standard that allows encapsulation of DOM and CSS within a single component, making it easier to build and maintain complex web applications. It enables the creation of reusable web components that can be used across multiple projects without any conflicts in styling and functionality.
    
    - Shadow DOM is a web API that provides a way to attach a hidden DOM to a DOM element
    - Shadow DOM is encapsulation, you can not link css to Shadow DOM tree.
    - `querySelectorAll(’h1’)` will not access the Shadow DOM elements
    
    Here is an example of using Shadow DOM in HTML and JavaScript:
    
    ```html
    <!DOCTYPE html>
    <html>
    
    <head>
        <title>Browser</title>
        <style>
            h1 {
                color: blue;
            }
        </style>
    </head>
    
    <body>
        <h1>Light DOM</h1>
    
        <script>
            // Create a div element and shadow root with mode open
            const el = document.createElement('div');
            const shadowRoot = el.attachShadow({
                mode: 'open'
            });
            shadowRoot.innerHTML = `<h1>From Shadow DOM</h1>`
    
            // Get the body and appendClild
            const container = document.querySelector('body');
    
            // Attach the cloned template to the shadow root
            container.appendChild(el);
    
            // querySelectorAll will not get shadow DOM element
            // if used document
           	console.log(document.querySelectorAll('h1').length)
            console.log(document.querySelectorAll('h1')[0].textContent)
    
            // querySelectorAll will get shadow DOM element 
          	// if used el.shadowRoot
            console.log(el.shadowRoot.querySelectorAll('h1').length);
            console.log(el.shadowRoot.querySelectorAll('h1')[0].textContent);
        </script>
    </body>
    
    </html>
    ```
    
    In this example, we define a custom component using a `template` tag. The `template` tag contains the HTML and CSS for our component. We then create a shadow root using the `attachShadow` method and attach it to a container element. Next, we clone the `template` tag and attach it to the shadow root using the `appendChild` method.
    
    The end result is a custom component with encapsulated styling that can be used across multiple projects without any conflicts.
    
    ![Screenshot 2023-03-30 at 12.46.48 AM.png](React%20Developer%20Interview%20Preparation%E2%80%99s%20268585597ad780dc8662ce69647dda2f/Screenshot_2023-03-30_at_12.46.48_AM.png)
    
    ![Untitled](React%20Developer%20Interview%20Preparation%E2%80%99s%20268585597ad780dc8662ce69647dda2f/Untitled.png)
    
    There are some bits of shadow DOM terminology to be aware of:
    
    - **Shadow host**: The regular DOM node that the shadow DOM is attached to.
    - **Shadow tree**: The DOM tree inside the shadow DOM.
    - **Shadow boundary**: the place where the shadow DOM ends, and the regular DOM begins.
    - **Shadow root**: The root node of the shadow tree.
- **Arrow function vs normal function**
    
    In JavaScript, **arrow functions (`=>`)** and **normal functions (`function`)** are used to define functions, but they have key differences in behavior.
    
    ---
    
    ## **1. `this` Binding**
    
    ### 🔹 **Arrow Function** - **Does NOT have its own `this`**
    
    - `this` is **lexically bound** (it takes `this` from the surrounding scope).
    
    ```jsx
    const obj = {
      name: "Alice",
      arrowFunc: () => {
        console.log(this.name); // ❌ undefined (inherits `this` from parent scope)
      },
      normalFunc() {
        console.log(this.name); // ✅ "Alice" (Own `this`)
      }
    };
    
    obj.arrowFunc();
    obj.normalFunc();
    ```
    
    ### 🔹 **Normal Function** - **Has its own `this`**
    
    - `this` depends on **how the function is called**.
    
    ---
    
    ## **2. `arguments` Object**
    
    ### 🔹 **Arrow Function** - **Does NOT have `arguments`**
    
    ```jsx
    javascript
    CopyEdit
    const arrowFunc = () => console.log(arguments);
    arrowFunc(1, 2, 3); // ❌ Error: arguments is not defined
    
    ```
    
    ### 🔹 **Normal Function** - **Has `arguments`**
    
    ```jsx
    javascript
    CopyEdit
    function normalFunc() {
      console.log(arguments); // ✅ [1, 2, 3]
    }
    normalFunc(1, 2, 3);
    
    ```
    
    ---
    
    ## **3. Usage in Methods**
    
    ### 🔹 **Arrow Functions Should Not Be Used as Object Methods**
    
    ```jsx
    const obj = {
      value: 42,
      arrowMethod: () => {
        console.log(this.value); // ❌ undefined
      },
      normalMethod() {
        console.log(this.value); // ✅ 42
      }
    };
    obj.arrowMethod();
    obj.normalMethod();
    ```
    
    **🔹 Why?**
    
    - Arrow functions **don't have their own `this`**, so they use `this` from the outer scope (which is likely `window` in browsers).
    
    ---
    
    ## **4. Constructor Function (`new` Keyword)**
    
    ### 🔹 **Arrow Functions Can't Be Used as Constructors**
    
    ```jsx
    javascript
    CopyEdit
    const Person = (name) => {
      this.name = name; // ❌ Error: Cannot use 'this' in arrow function
    };
    const p = new Person("Alice"); // ❌ TypeError
    
    ```
    
    ### 🔹 **Normal Functions Can Be Used as Constructors**
    
    ```jsx
    javascript
    CopyEdit
    function Person(name) {
      this.name = name; // ✅ Works correctly
    }
    const p = new Person("Alice");
    console.log(p.name); // ✅ "Alice"
    
    ```
    
    ---
    
    ## **5. Implicit Return (`return` Keyword)**
    
    ### 🔹 **Arrow Functions Can Have an Implicit Return**
    
    ```jsx
    javascript
    CopyEdit
    const add = (a, b) => a + b; // ✅ No `{}` needed
    console.log(add(2, 3)); // 5
    
    ```
    
    ### 🔹 **Normal Functions Require `return`**
    
    ```jsx
    javascript
    CopyEdit
    function add(a, b) {
      return a + b; // ✅ Must explicitly return
    }
    console.log(add(2, 3)); // 5
    
    ```
    
    ---
    
    ## **6. When to Use Which?**
    
    | Feature | Arrow Function (`=>`) | Normal Function (`function`) |
    | --- | --- | --- |
    | `this` Binding | Inherits from the surrounding scope (lexical `this`) | Has its own `this` |
    | `arguments` Object | ❌ Not available | ✅ Available |
    | `new` Constructor | ❌ Not allowed | ✅ Allowed |
    | Object Methods | ❌ Avoid (loses `this`) | ✅ Works well |
    | Short Syntax | ✅ Yes (implicit return) | ❌ More verbose |
    | Callbacks & Functional Programming | ✅ Preferred | ✅ Also works |
    
    ---
    
    ## **Final Thoughts**
    
    ✅ **Use Arrow Functions (`=>`)** for:
    
    - **Short functions & one-liners** (e.g., array methods: `map`, `filter`, `reduce`).
    - **Callbacks** (e.g., event handlers, async functions).
    - **Preserving `this` from the surrounding scope** (e.g., inside a class method).
    
    ✅ **Use Normal Functions (`function`)** for:
    
    - **Object methods** (to avoid `this` issues).
    - **Functions needing `arguments`**.
    - **Constructor functions** (`new` keyword usage).
    
    ---
    
    ### **🎯 Freshworks Interview Tip**
    
    - If asked **"Which one should we use?"**, say:
        
        > "It depends on the use case. Arrow functions are great for short, functional-style code, but normal functions are needed for objects, constructors, and when we need arguments or this binding."
        > 
    
    ### **Key Differences:**
    
    | Feature | Normal Function | Arrow Function |
    | --- | --- | --- |
    | **`this` Binding** | Own `this` (depends on how it's called) | Inherits `this` from the surrounding scope |
    | **`arguments` Object** | Has its own `arguments` | Inherits from the surrounding function |
    | **Can be a Constructor?** | ✅ Yes, can be used with `new` | ❌ No, cannot be used as a constructor |
    | **Implicit Return** | Requires `return` keyword (unless one-liner) | Can implicitly return values in single-line expressions |
    | **Usage in Methods** | Suitable for object methods | Not suitable for object methods (because `this` is not bound to the object) |
- **Dom**
    1. What is the Document Object Model (DOM)?
    The Document Object Model (DOM) is a programming interface for web documents. It represents the page so that programs can change the document structure, style, and content.
    2. What is a node in the DOM?
    A node is an object in the DOM that represents an element, attribute, or piece of text in an HTML or XML document.
    3. What is the difference between the HTML DOM and XML DOM?
    The HTML DOM is designed to work with HTML documents, while the XML DOM is designed to work with XML documents. However, both are part of the same DOM standard.
    4. What is a parent node and a child node?
    A parent node is an element that contains one or more child nodes. Child nodes are elements or other nodes that are contained within a parent node.
    5. What is an attribute in the DOM?
    An attribute is a characteristic of an HTML or XML element that is specified using the element's attributes.
    6. How do you create a new element in the DOM?
    You can create a new element in the DOM using the **`createElement()`** method. For example, to create a new paragraph element:
        
        ```jsx
        var paragraph = document.createElement("p");
        ```
        
    7. How do you add a new element to the DOM?
    You can add a new element to the DOM using the **`appendChild()`** method. For example, to add a new paragraph element to the body of an HTML document:
        
        ```jsx
        document.body.appendChild(paragraph);
        ```
        
    8. How do you remove an element from the DOM?
    You can remove an element from the DOM using the **`removeChild()`** method. For example, to remove a paragraph element from the body of an HTML document:
        
        ```
        document.body.removeChild(paragraph);
        ```
        
    9. What is the difference between innerHTML and outerHTML?
    **`innerHTML`** is a property that sets or returns the content of an element, including any HTML tags inside it. **`outerHTML`** is a property that sets or returns the entire HTML content of an element, including the element itself.
    10. What is the difference between textContent and innerText?
    **`textContent`** is a property that sets or returns the text content of an element, without any HTML tags. **`innerText`** is a property that sets or returns the visible text content of an element, with any HTML tags removed.
    11. How do you get the value of an element in the DOM?
    You can get the value of an element in the DOM using the **`value`** property. For example, to get the value of an input element:
        
        ```jsx
        var input = document.getElementById("myInput");
        var value = input.value;
        ```
        
    12. How do you set the value of an element in the DOM?
    You can set the value of an element in the DOM using the **`value`** property. For example, to set the value of an input element:
        
        ```jsx
        var input = document.getElementById("myInput");
        input.value = "Hello, World!";
        ```
        
    13. How do you get the style of an element in the DOM?
    You can get the style of an element in the DOM using the **`style`** property. For example, to get the color of a paragraph element:
        
        ```jsx
        var paragraph = document.getElementById("myParagraph");
        var color = paragraph.style.color;
        ```
        
    14. How do you set the style of an element in the DOM?
    You can set the style of an element in the DOM using the **`style`** property. For example, to set the color of a paragraph element:
        
        ```jsx
        var paragraph = document.getElementById("myParagraph");
        paragraph.style.color = "red";
        ```
        
    15. What are the different levels of the DOM?
    16. What is the difference between the DOM and HTML?
    17. How does the browser create the DOM tree?
    18. What are the different types of nodes in the DOM tree?
    19. How can you access elements in the DOM?
    20. What is the difference between getElementsByTagName and getElementsByName?
    21. How do you create new nodes in the DOM tree?
    22. What is the difference between appendChild and insertBefore?
    23. How can you remove a node from the DOM tree?
    24. What is the difference between innerHTML and outerHTML?
    25. What is event propagation in the DOM?
    26. What is the difference between event bubbling and event capturing?
    27. How can you prevent event propagation?
    28. What is the target of an event in the DOM?
    29. How do you create an event in JavaScript?
    30. How can you add an event listener to an element in the DOM?
    31. What is the difference between onclick and addEventListener?
    32. What is the event loop in JavaScript?
- **map() Vs forEach()**
    
    In JavaScript, `Map` and `forEach` are both functions that are used for iterating over an array or an iterable object, but they have some differences.
    
    `Map` function: The `Map` function is used to create a new array by applying a callback function to each element of the array. It returns a new array with the same length as the original array. The `Map` function does not modify the original array. The callback function passed to the `Map` function can take three arguments - the current value of the element, the index of the element, and the original array.
    
    Example:
    
    ```jsx
    const arr = [1, 2, 3, 4];
    
    const mapArr = arr.map(function(num) {
      return num * 2;
    });
    
    console.log(mapArr); // Output: [2, 4, 6, 8]
    console.log(arr); // Output: [1, 2, 3, 4]
    ```
    
    `forEach` function: The `forEach` function is used to iterate over the array and execute a callback function for each element. It does not return a new array. The `forEach` function modifies the original array. The callback function passed to the `forEach` function can take three arguments - the current value of the element, the index of the element, and the original array.
    
    Example:
    
    ```jsx
    const arr = [1, 2, 3, 4];
    
    arr.forEach(function(num, index, array) {
      console.log(num, index, array);
    });
    
    // Output:
    // 1 0 [1, 2, 3, 4]
    // 2 1 [1, 2, 3, 4]
    // 3 2 [1, 2, 3, 4]
    // 4 3 [1, 2, 3, 4]
    console.log(arr); // Output: [1, 2, 3, 4]
    ```
    
    Difference:
    
    1. `Map` function returns a new array while `forEach` function does not return any value.
    2. `Map` function does not modify the original array, while `forEach` function modifies the original array.
    3. `Map` function creates a new array by applying a callback function to each element of the array, while `forEach` function iterates over the array and executes a callback function for each element.
- **How to Key and Value in Object**
    
    In JavaScript, you can add a key-value pair to an object using the following syntax:
    
    ```jsx
    objectName[key] = value;
    ```
    
    Here, `objectName` is the name of the object to which you want to add the key-value pair, `key` is the name of the key, and `value` is the value associated with that key.
    
    For example, consider the following object:
    
    ```jsx
    let person = {
      name: "John",
      age: 30
    };
    ```
    
    To add a new key-value pair to this object, you can use the following syntax:
    
    ```jsx
    person["email"] = "john@example.com";
    ```
    
    Now, the `person` object will have a new key-value pair `email: "john@example.com"`. Alternatively, you can also use the dot notation to add a new key-value pair to an object, like this:
    
    ```jsx
    person.gender = "male";
    ```
    
    This will add a new key-value pair `gender: "male"` to the `person` object.
    
    Note that you can also use a variable as a key in an object by wrapping it in square brackets `[]`, like this:
    
    ```jsx
    let propertyName = "email";
    person[propertyName] = "john@example.com";
    ```
    
    This will add a new key-value pair `email: "john@example.com"` to the `person` object, where the key name is stored in the `propertyName` variable.
    
- **Invert Binary Tree JavaScript Recursive Solution**
    
    Inverting a binary tree means that for every node in the tree, its left and right subtrees are swapped. This can be achieved recursively by traversing the tree and swapping the left and right children of each node. Here's a JavaScript solution to invert a binary tree:
    
    ```jsx
    function invertTree(root) {
      if (root === null) {
        return null;
      }
      // Swap left and right subtrees recursively
      const left = invertTree(root.left);
      const right = invertTree(root.right);
      root.left = right;
      root.right = left;
      return root;
    }
    ```
    
    In this solution, the function `invertTree` takes the root of the binary tree as input and returns the inverted binary tree.
    
    The function starts by checking if the input `root` is `null`. If it is, the function simply returns `null`.
    
    If the `root` is not `null`, the function recursively calls itself to invert the left and right subtrees of the `root`. The result of these recursive calls are stored in `left` and `right` variables.
    
    Then, the left and right subtrees of the `root` are swapped by assigning the `right` subtree to the `root.left` property and the `left` subtree to the `root.right` property.
    
    Finally, the function returns the inverted `root`.
    
- **Chaining**
    
    Chaining is a technique used in JavaScript that allows calling multiple functions on an object or a value in a single line of code. This technique is used to make code more readable and concise. In order to chain functions, each function should return the object or value it operates on.
    
    Here's an example of chaining in JavaScript:
    
    ```jsx
    const numbers = [1, 2, 3, 4, 5];
    
    const result = numbers
      .filter(num => num % 2 === 0)
      .map(num => num * 2)
      .reduce((acc, num) => acc + num, 0);
    
    console.log(result); // Output: 12
    ```
    
    In this example, the `filter()` method filters out the odd numbers from the array, the `map()` method doubles the filtered values and the `reduce()` method returns the sum of the doubled values.
    
    As you can see, each function is chained to the previous function by using the dot (**`.`**) operator, and the result of each function is passed to the next function. This makes the code more concise and easier to read.
    
- **Currying**
    
    Currying is a technique in functional programming where a function with multiple arguments is transformed into a sequence of functions, each taking one argument. The curried function returns a new function with the first argument already set, waiting for the remaining arguments.
    
    Here's an example of currying in JavaScript:
    
    ```jsx
    // Function to add three numbers
    function add(a, b, c) {
      return a + b + c;
    }
    
    // Curried version of add function
    function curriedAdd(a) {
      return function(b) {
        return function(c) {
          return a + b + c;
        }
      }
    }
    
    // Usage of curriedAdd function
    const result = curriedAdd(1)(2)(3);
    console.log(result); // Output: 6
    ```
    
    In the above example, `add` function takes three arguments `a`, `b`, and `c` and returns their sum. The `curriedAdd` function is the curried version of `add` function. It returns a new function that takes one argument at a time and returns the sum of all three arguments when all the arguments are provided.
    
    The `curriedAdd` function takes the first argument `a` and returns a new function that takes the second argument `b`. When this new function is called with `b`, it returns another function that takes the third argument `c`. Finally, when the last function is called with `c`, it returns the sum of all three arguments `a`, `b`, and `c`.
    
    Currying is a powerful technique that enables the creation of specialized functions that are easy to reuse and compose with other functions.
    
    **Below are some scenarios where currying can be used in JavaScript:**
    
    1. Function Composition: In function composition, we can use currying to build complex functions from simple ones. For example, we can create a function that multiplies the input by a constant value and then adds another value to the result. This can be achieved by creating a curried function that takes the constant value as its first argument and returns a function that takes the input and returns the result.
    2. Event Handlers: In event-driven programming, currying can be used to create event handlers that take additional parameters. For example, we can create a curried function that takes the event object as its first argument and returns a function that takes the target element as its second argument. This can help in reducing code duplication and improve code readability.
    3. Data Transformation: Currying can also be used in data transformation scenarios, where we need to transform data from one format to another. For example, we can create a curried function that takes a string and returns an array of words. This function can be used in various scenarios where we need to transform string data.
    4. Memoization: Memoization is a technique in which the results of expensive function calls are cached and returned when the same inputs occur again. Currying can help in creating memoized functions that return cached results for the same inputs. This can help in improving the performance of the application.
    
    Overall, currying is a powerful technique in JavaScript that can be used in a wide range of scenarios to improve code organisation, readability, and performance.
    
- **Lexical scope**
    
    In JavaScript, every function creates a new scope. The scope determines the visibility and accessibility of variables and functions in the code. There are two types of scope in JavaScript: lexical scope and dynamic scope.
    
    Lexical scope refers to the way that variable names are resolved in nested functions: inner functions contain the scope of parent functions even if the parent function has already returned.
    
    Here's an example of lexical scope in JavaScript:
    
    ```jsx
    function outerFunction() {
      const outerVar = 'I am in the outer function';
    
      function innerFunction() {
        const innerVar = 'I am in the inner function';
        console.log(innerVar); // logs 'I am in the inner function'
        console.log(outerVar); // logs 'I am in the outer function'
      }
    
      return innerFunction;
    }
    
    const inner = outerFunction();
    inner(); // logs 'I am in the inner function' and 'I am in the outer function'
    ```
    
    In this example, `outerFunction` creates a new variable `outerVar` and a nested function `innerFunction`. `innerFunction` creates a new variable `innerVar` and logs the values of both `innerVar` and `outerVar`.
    
    When `outerFunction` is called and `innerFunction` is returned, `inner` becomes a reference to `innerFunction`, preserving its lexical scope. When `inner` is called, it logs the values of `innerVar` and `outerVar` from the outer scope of `outerFunction`.
    
    This demonstrates that inner functions in JavaScript have access to the variables and functions defined in the outer functions in which they are nested, even after those outer functions have returned.
    
- **Dynamic scope**
    
    JavaScript uses lexical scoping, which means that the scope of a variable is determined by its location in the code. However, some programming languages like Perl and Bash use dynamic scoping.
    
    In dynamic scoping, the scope of a variable is determined by the current execution context rather than its location in the code. This means that the value of a variable depends on the call stack rather than the location in the code.
    
    Here's an example in JavaScript to illustrate the difference between lexical and dynamic scoping:
    
    ```jsx
    var x = 1;
    
    function foo() {
      console.log(x);
    }
    
    function bar() {
      var x = 2;
      foo();
    }
    
    bar(); // output: 1
    ```
    
    In this example, we have a global variable `x` with a value of 1. The function `foo()` accesses the value of `x` in the global scope and logs it to the console. The function `bar()` declares a local variable `x` with a value of 2 and then calls `foo()`. When `foo()` logs the value of `x`, it looks for the value in the global scope, since that's where `foo()` is defined. Therefore, the output is 1.
    
    If JavaScript were using dynamic scoping, the output would be 2, since `foo()` would look for the value of `x` in the local scope of `bar()` instead of the global scope.
    
    Since JavaScript uses lexical scoping, the output is determined by the location of the variable in the code, rather than the current execution context.
    
- **Normal function & Arrow function in javascript**
    
    n JavaScript, both normal (traditional) functions and arrow functions are used to define functions, but they differ in several key ways. Let's break down these differences with examples.
    
    ### **Normal Function (Traditional Function)**
    
    **Syntax:**
    
    ```jsx
    function functionName(parameters) {
      // function body
    }
    ```
    
    **Characteristics:**
    
    1. **`this` Binding**:
        - Traditional functions have their own `this` context. The value of `this` depends on how the function is called.
    2. **`arguments` Object**:
        - Traditional functions have access to the `arguments` object, which contains the list of arguments passed to the function.
    3. **Constructors**:
        - Traditional functions can be used as constructors to create instances of objects.
    4. **Method Binding**:
        - When used as object methods, `this` refers to the object that owns the method.
    
    **Example:**
    
    ```jsx
    // Traditional Function
    function greet(name) {
      console.log(`Hello, ${name}`);
    }
    
    greet('Alice'); // Output: Hello, Alice
    
    // `this` context in traditional functions
    const obj = {
      value: 42,
      getValue: function() {
        return this.value; // `this` refers to `obj`
      }
    };
    
    console.log(obj.getValue()); // Output: 42
    ```
    
    ### **Arrow Function**
    
    **Syntax:**
    
    ```jsx
    const functionName = (parameters) => {
      // function body
    }
    ```
    
    **Characteristics:**
    
    1. **`this` Binding**:
        - Arrow functions do not have their own `this` context. Instead, `this` is lexically inherited from the surrounding context (i.e., the `this` value of the surrounding function or global context).
    2. **`arguments` Object**:
        - Arrow functions do not have their own `arguments` object. You need to use rest parameters (`...args`) if you need access to arguments.
    3. **Constructors**:
        - Arrow functions cannot be used as constructors. They will throw an error if used with `new`.
    4. **Method Binding**:
        - When used as object methods, arrow functions do not bind their own `this` and instead inherit `this` from the outer scope.
    
    **Example:**
    
    ```jsx
    // Arrow Function
    const greet = (name) => {
      console.log(`Hello, ${name}`);
    };
    
    greet('Bob'); // Output: Hello, Bob
    
    // `this` context in arrow functions
    const obj = {
      value: 42,
      getValue: () => {
        return this.value; // `this` refers to the global context, not `obj`
      }
    };
    
    console.log(obj.getValue()); // Output: undefined
    
    // Arrow functions and `arguments`
    const sum = (...args) => {
      return args.reduce((acc, curr) => acc + curr, 0);
    };
    
    console.log(sum(1, 2, 3, 4)); // Output: 10
    ```
    
    ### **Key Differences**
    
    1. **`this` Binding**:
        - **Normal Function**: `this` is dynamically bound depending on the call context.
        - **Arrow Function**: `this` is lexically bound based on the surrounding code.
    2. **`arguments` Object**:
        - **Normal Function**: Available and contains all arguments passed to the function.
        - **Arrow Function**: Not available; use rest parameters instead.
    3. **Constructor Behavior**:
        - **Normal Function**: Can be used as a constructor (with `new`).
        - **Arrow Function**: Cannot be used as a constructor.
    4. **Method Binding**:
        - **Normal Function**: `this` refers to the object owning the method.
        - **Arrow Function**: `this` is inherited from the outer scope and does not bind to the object.
    
    ### **Usage Recommendations**
    
    - **Use Normal Functions**: When you need `this` to refer to the object calling the method, or when you need access to the `arguments` object, or when you want to use the function as a constructor.
    - **Use Arrow Functions**: For short functions or callbacks where you want to preserve the `this` context from the surrounding scope, or when you don't need to use `arguments`.
    
    Understanding these differences helps in writing more predictable and maintainable JavaScript code.
    
- **Debounce and Throttling in JavaScript**
    
    ### Debounce and Throttling in JavaScript
    
    **Debounce** and **throttling** are two techniques used to control how frequently a function executes, particularly in response to events that fire repeatedly, such as scrolls, clicks, or window resizes.
    
    ### **Debounce**
    
    **Debounce** ensures that a function is only executed once after a certain period of time has passed since the last time it was invoked. It essentially "waits" for a specified period of inactivity before triggering the function.
    
    **Use Case:** Debounce is useful for situations where you want to limit the rate at which a function executes, especially when the function is triggered by a high-frequency event. For example, form validation as a user types, window resizing, or search input field.
    
    **Example of Debounce:**
    
    ```jsx
    function debounce(func, delay) {
        let timeoutId;
        return function(...args) {
            clearTimeout(timeoutId);  // Clear the previous timeout
            timeoutId = setTimeout(() => {
                func.apply(this, args);
            }, delay);
        };
    }
    
    // Example usage:
    const logMessage = () => console.log('Debounced function executed!');
    const debouncedLogMessage = debounce(logMessage, 2000);
    
    // If you call the following in quick succession, the logMessage will only be printed once, 2 seconds after the last call.
    debouncedLogMessage();
    debouncedLogMessage();
    debouncedLogMessage();
    ```
    
    In this example, `logMessage` will only be logged once, even if `debouncedLogMessage()` is called multiple times in quick succession. It waits 2000ms after the last call to actually invoke `logMessage`.
    
    Let's break down the provided `debounce` function and its usage step by step.
    
    The `debounce` Function
    
    ```jsx
    function debounce(func, delay) {
        let timeoutId;
        return function(...args) {
            clearTimeout(timeoutId);  // Clear the previous timeout
            timeoutId = setTimeout(() => {
                func.apply(this, args);
            }, delay);
        };
    }
    ```
    
    1. **Parameters:**
        - `func`: The function you want to debounce (delay its execution).
        - `delay`: The amount of time (in milliseconds) to wait before executing the function after the last call.
    2. **Inside the Function:**
        - `timeoutId`: This variable will store the identifier of the timeout that is set by `setTimeout`. It's used to keep track of the last timer that was set.
    3. **Return Value:**
        - The `debounce` function returns a new function that wraps the original `func` with debounce logic. This returned function has the ability to manage the timing of when `func` gets called.
    4. **The Inner Function:**
        - `clearTimeout(timeoutId)`: Whenever the returned function is called, it first clears any existing timeout (using `clearTimeout`). This means that if the function is called again within the delay period, the previous timer is canceled.
        - `setTimeout(() => { func.apply(this, args); }, delay)`: After clearing the previous timeout, a new one is set. This means the `func` will only be executed after the delay period has passed without any further calls.
    
    Example Usage:
    
    ```jsx
    const logMessage = () => console.log('Debounced function executed!');
    const debouncedLogMessage = debounce(logMessage, 2000);
    ```
    
    1. **`logMessage`:**
        - A simple function that logs a message to the console.
    2. **`debouncedLogMessage`:**
        - This is the debounced version of `logMessage`, created by passing `logMessage` and a delay of 2000 milliseconds (2 seconds) to the `debounce` function.
    
    How It Works in Action:
    
    ```jsx
    debouncedLogMessage();
    debouncedLogMessage();
    debouncedLogMessage();
    ```
    
    - When you call `debouncedLogMessage()` the first time, it sets a timer for 2 seconds.
    - If you call `debouncedLogMessage()` again within those 2 seconds, the timer is reset. The previous timer is cleared, and a new one is set for another 2 seconds.
    - This process repeats with each subsequent call to `debouncedLogMessage()`.
    
    **Outcome:**
    
    - **"Debounced function executed!"** will only be logged once, **2 seconds after the last call** to `debouncedLogMessage()`. This is because the function only executes if no further calls are made within the specified delay period.
    
    **Real-World Use Case:**
    
    Consider a scenario where you're typing in a search input field that fetches search suggestions from a server. Without debouncing, each keystroke would trigger a server request, which can be expensive and inefficient. By debouncing the input handler, the search request is only sent once after the user stops typing for a specified amount of time, reducing unnecessary server requests.
    
    This makes debounce an essential tool in optimizing performance, especially when dealing with frequently occurring events.
    
    ### **Throttling**
    
    **Throttling** ensures that a function is only executed at most once in a specified period of time, regardless of how many times the triggering event occurs. Throttling ensures that a function is called at a regular interval.
    
    **Use Case:** Throttling is particularly useful when you want to ensure that a function is called at consistent intervals, regardless of how often the event is fired. Examples include scrolling, resizing, or controlling a continuous action like button mashing.
    
    **Example of Throttling:**
    
    ```jsx
    function throttle(func, limit) {
        let inThrottle;
        return function(...args) {
            if (!inThrottle) {
                func.apply(this, args);
                inThrottle = true;
                setTimeout(() => inThrottle = false, limit);
            }
        };
    }
    
    // Example usage:
    const logMessage = () => console.log('Throttled function executed!');
    const throttledLogMessage = throttle(logMessage, 2000);
    
    // If you call the following in quick succession, logMessage will be logged at most once every 2 seconds.
    throttledLogMessage();
    throttledLogMessage();
    throttledLogMessage();
    ```
    
    In this example, `logMessage` will be logged no more than once every 2 seconds, even if `throttledLogMessage()` is called multiple times.
    
    ### **Key Differences Between Debounce and Throttling**
    
    - **Debounce:** Waits for a specified period of inactivity before executing the function. Useful when you want to ensure a function only executes after events have stopped firing for a specified time (e.g., form input validation).
    - **Throttling:** Ensures a function is called at most once per specified interval. Useful for limiting the rate of function execution during continuous events (e.g., handling scroll events).
    
    Both techniques are crucial for improving the performance of applications by reducing the frequency of expensive operations in response to events.
    
- **Different promises and when to use what**
    
    In JavaScript, promises are a powerful tool for managing asynchronous operations. They provide a way to handle the eventual completion or failure of an asynchronous operation, allowing for more readable and maintainable code compared to traditional callback-based approaches. Different types of promises and promise patterns are used based on specific scenarios. Here’s a detailed explanation of different promise types and when to use them, along with real-time scenarios.
    
    ### 1. **Basic Promise**
    
    **Definition:**
    
    - A basic promise represents a single asynchronous operation. It can be in one of three states: pending, fulfilled, or rejected.
    
    **Usage Scenario:**
    
    - Use a basic promise when you need to handle a single asynchronous operation, such as fetching data from an API.
    
    **Example:**
    
    ```jsx
    const fetchData = () => {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                const data = "Sample data";
                resolve(data); // Operation successful
                // reject("Error fetching data"); // Operation failed
            }, 1000);
        });
    };
    
    fetchData()
        .then(data => console.log(data))
        .catch(error => console.error(error));
    ```
    
    **Real-Time Scenario:**
    
    - Fetching user details from a server when a user logs into an application.
    
    ### 2. **Promise.all**
    
    **Definition:**
    
    - `Promise.all` takes an array of promises and returns a single promise that resolves when all the promises in the array have resolved, or rejects if any of the promises reject.
    
    **Usage Scenario:**
    
    - Use `Promise.all` when you have multiple asynchronous operations that need to be completed before proceeding, and you need the results of all operations.
    
    **Example:**
    
    ```jsx
    const fetchUser = () => new Promise(resolve => setTimeout(() => resolve("User"), 1000));
    const fetchPosts = () => new Promise(resolve => setTimeout(() => resolve("Posts"), 2000));
    
    Promise.all([fetchUser(), fetchPosts()])
        .then(([user, posts]) => {
            console.log("User:", user);
            console.log("Posts:", posts);
        })
        .catch(error => console.error(error));
    ```
    
    **Real-Time Scenario:**
    
    - Fetching user profile data and their recent posts in a social media application, where both pieces of data are needed together.
    
    ### 3. **Promise.race**
    
    **Definition:**
    
    - `Promise.race` takes an array of promises and returns a single promise that resolves or rejects as soon as the first promise in the array resolves or rejects.
    
    **Usage Scenario:**
    
    - Use `Promise.race` when you need to proceed with the result of the first promise that completes, regardless of the results of other promises.
    
    **Example:**
    
    ```jsx
    const fetchData1 = () => new Promise(resolve => setTimeout(() => resolve("Data 1"), 1000));
    const fetchData2 = () => new Promise(resolve => setTimeout(() => resolve("Data 2"), 500));
    
    Promise.race([fetchData1(), fetchData2()])
        .then(data => console.log("First completed:", data))
        .catch(error => console.error(error));
    ```
    
    **Real-Time Scenario:**
    
    - Implementing a timeout for an API request where you want to proceed with the result of the quickest response.
    
    ### 4. **Promise.allSettled**
    
    **Definition:**
    
    - `Promise.allSettled` takes an array of promises and returns a promise that resolves after all the given promises have either resolved or rejected. It provides an array of objects describing the outcome of each promise.
    
    **Usage Scenario:**
    
    - Use `Promise.allSettled` when you need to handle multiple asynchronous operations and you want to know the outcome of each operation, regardless of whether they succeeded or failed.
    
    **Example:**
    
    ```jsx
    const fetchData1 = () => new Promise((resolve, reject) => setTimeout(() => resolve("Data 1"), 1000));
    const fetchData2 = () => new Promise((resolve, reject) => setTimeout(() => reject("Error"), 500));
    
    Promise.allSettled([fetchData1(), fetchData2()])
        .then(results => {
            results.forEach((result, index) => {
                if (result.status === 'fulfilled') {
                    console.log(`Promise ${index} succeeded with value: ${result.value}`);
                } else {
                    console.error(`Promise ${index} failed with reason: ${result.reason}`);
                }
            });
        });
    ```
    
    **Real-Time Scenario:**
    
    - Submitting multiple independent requests (e.g., updating user profile, logging activity, and notifying other services) where you need to know the result of each request, whether successful or not.
    
    ### 5. **Promise.any**
    
    **Definition:**
    
    - `Promise.any` takes an array of promises and returns a single promise that resolves as soon as any of the promises in the array resolves. It rejects if no promises in the array resolve (i.e., all promises reject).
    
    **Usage Scenario:**
    
    - Use `Promise.any` when you want to proceed with the result of the first successful promise, ignoring the others.
    
    **Example:**
    
    ```jsx
    const fetchData1 = () => new Promise((_, reject) => setTimeout(() => reject("Error 1"), 1000));
    const fetchData2 = () => new Promise((_, reject) => setTimeout(() => reject("Error 2"), 500));
    const fetchData3 = () => new Promise(resolve => setTimeout(() => resolve("Data 3"), 1500));
    
    Promise.any([fetchData1(), fetchData2(), fetchData3()])
        .then(data => console.log("First successful data:", data))
        .catch(error => console.error("All promises rejected:", error));
    ```
    
    **Real-Time Scenario:**
    
    - Attempting to connect to multiple servers or services, where you want to use the first available one that responds successfully.
    
    ### **Summary**
    
    - **Basic Promise:** For single asynchronous operations.
    - **Promise.all:** When multiple promises need to be completed before proceeding, and you need results from all.
    - **Promise.race:** When you need the result of the first promise that completes, regardless of the results of other promises.
    - **Promise.allSettled:** When you need the outcome of all promises, whether they succeed or fail.
    - **Promise.any:** When you want the result of the first successful promise, ignoring rejections.
    
    Understanding when to use each type of promise helps manage asynchronous operations effectively, ensuring your applications are both performant and maintainable.
    
- **Hoisting**
    
    Hoisting is a JavaScript behavior where variable and function declarations are moved ("hoisted") to the top of their containing scope during the compilation phase, before the code is executed. This means that you can use variables and functions before they are declared in your code, but there are important distinctions between how different types of declarations are hoisted.
    
    ### **How Hoisting Works**
    
    1. **Variable Declarations with `var`:**
        - When you declare a variable using `var`, JavaScript hoists the declaration to the top of its function or global scope, but not its initialization.
        - If you try to access a `var` variable before its declaration, it will be `undefined`, not a `ReferenceError`.
    2. **Function Declarations:**
        - Function declarations are fully hoisted. This means that the entire function is moved to the top of its scope, so you can call the function before it is declared in the code.
    3. **Variable Declarations with `let` and `const`:**
        - Unlike `var`, variables declared with `let` and `const` are hoisted but are not initialized. They are in a "temporal dead zone" from the start of the block until the declaration is encountered. Trying to access them before their declaration will result in a `ReferenceError`.
    
    ### **Examples to Illustrate Hoisting**
    
    ### 1. **Hoisting with `var`:**
    
    ```jsx
    console.log(myVar); // Output: undefined
    var myVar = 5;
    console.log(myVar); // Output: 5
    ```
    
    - **Explanation**:
        - The first `console.log` prints `undefined` because the declaration of `myVar` is hoisted to the top, but the initialization (`myVar = 5`) is not.
        - The code behaves as if it were written like this:
            
            ```jsx
            var myVar;
            console.log(myVar); // undefined
            myVar = 5;
            console.log(myVar); // 5
            ```
            
    
    ### 2. **Hoisting with Function Declarations:**
    
    ```jsx
    greet(); // Output: "Hello!"
    function greet() {
        console.log("Hello!");
    }
    ```
    
    - **Explanation**:
        - The function `greet` is hoisted to the top, so you can call it before its actual declaration. The entire function definition is moved to the top of its scope.
    
    ### 3. **Hoisting with `let` and `const`:**
    
    ```jsx
    console.log(myLet); // ReferenceError: Cannot access 'myLet' before initialization
    let myLet = 10;
    console.log(myLet); // Output: 10
    ```
    
    - **Explanation**:
        - `myLet` is hoisted, but it remains in the "temporal dead zone" until the actual declaration. Trying to access it before the declaration results in a `ReferenceError`.
    
    ### 4. **Hoisting with Function Expressions:**
    
    ```jsx
    console.log(myFunc); // Output: undefined
    var myFunc = function() {
        console.log("Hi there!");
    };
    myFunc(); // Output: "Hi there!"
    ```
    
    - **Explanation**:
        - Here, the variable `myFunc` is hoisted, but since it’s a `var` declaration, it’s initialized as `undefined`. The function expression itself is not hoisted, so calling `myFunc` before it’s assigned will result in `undefined` rather than a function.
    
    ### **Real-World Example of Hoisting**
    
    Consider a situation where you’re writing a function that needs to calculate the area of a rectangle. You might want to define some helper functions inside your main function.
    
    ```jsx
    function calculateArea(length, width) {
        console.log(area(length, width)); // Output: 20
    
        function area(a, b) {
            return a * b;
        }
    }
    
    calculateArea(4, 5);
    ```
    
    - **Explanation**:
        - The `area` function is hoisted within the `calculateArea` function, so it can be called before it’s defined in the code.
    
    However, if you used a function expression instead:
    
    ```jsx
    function calculateArea(length, width) {
        console.log(area(length, width)); // Error: area is not a function
    
        var area = function(a, b) {
            return a * b;
        };
    }
    
    calculateArea(4, 5);
    ```
    
    - **Explanation**:
        - Here, `area` is hoisted as a variable and initialized as `undefined`, so trying to call it before the assignment will throw an error.
    
    ### **Key Points to Remember**
    
    - **Variables declared with `var`** are hoisted and initialized with `undefined`.
    - **Functions** declared using function declarations are fully hoisted, so they can be called before their declaration.
    - **Variables declared with `let` and `const`** are hoisted but not initialized, leading to a `ReferenceError` if accessed before declaration.
    - **Function expressions and arrow functions** assigned to variables behave like variables—they are hoisted as undefined if using `var`, and are not accessible if using `let` or `const` until they are defined.
    
    ### **Conclusion**
    
    Understanding hoisting helps prevent unexpected behavior in your JavaScript code. By knowing how declarations are moved and initialized during the compilation phase, you can write more predictable and bug-free code.
    
- **Closures**
    
    ### **What is a Closure?**
    
    A **closure** is a feature in JavaScript where an inner function has access to the outer (enclosing) function's variables—even after the outer function has finished executing. Specifically, a closure allows a function to access:
    
    1. **Variables defined in its own scope.**
    2. **Variables of its outer (enclosing) function.**
    3. **Global variables.**
    
    The closure "closes over" the environment in which it was created, meaning it remembers the variables and functions from its lexical scope even when it's executed outside that scope.
    
    ### **Basic Example of a Closure**
    
    ```jsx
    function outerFunction() {
      let outerVariable = 'I am outside!';
    
      function innerFunction() {
        console.log(outerVariable);
      }
    
      return innerFunction;
    }
    
    const myClosure = outerFunction();
    myClosure(); // Output: 'I am outside!'
    ```
    
    In this example, `innerFunction` forms a closure. It retains access to `outerVariable`, even though `outerFunction` has finished executing when `myClosure` is called.
    
    ### **How Closures Work**
    
    When `outerFunction` is called, it creates a new execution context with its own variables and functions. When `innerFunction` is defined, it "captures" the `outerVariable` from its lexical environment. This captured environment is retained in memory when `innerFunction` is returned, forming a closure. Therefore, when `myClosure` is called, it still has access to `outerVariable`.
    
    ### **Real-World Examples of Closures**
    
    ### **1. Data Encapsulation**
    
    Closures are often used to encapsulate data and restrict access to it, similar to private variables in object-oriented programming.
    
    **Example:**
    
    ```jsx
    function createCounter() {
      let count = 0;
    
      return {
        increment: function() {
          count += 1;
          console.log(count);
        },
        decrement: function() {
          count -= 1;
          console.log(count);
        },
        getCount: function() {
          return count;
        }
      };
    }
    
    const counter = createCounter();
    counter.increment(); // Output: 1
    counter.increment(); // Output: 2
    console.log(counter.getCount()); // Output: 2
    counter.decrement(); // Output: 1
    ```
    
    Here, `count` is a private variable that can only be accessed and modified through the methods `increment`, `decrement`, and `getCount`. This is a common pattern for creating modules or objects with private state in JavaScript.
    
    ### **2. Function Factories**
    
    Closures are useful for creating function factories, where you generate specialized functions based on the input parameters.
    
    **Example:**
    
    ```jsx
    function createGreeting(greeting) {
      return function(name) {
        console.log(`${greeting}, ${name}!`);
      };
    }
    
    const sayHello = createGreeting('Hello');
    const sayHi = createGreeting('Hi');
    
    sayHello('Alice'); // Output: 'Hello, Alice!'
    sayHi('Bob'); // Output: 'Hi, Bob!'
    ```
    
    In this example, `createGreeting` returns a new function that "remembers" the greeting that was passed to it. This is a closure in action, allowing you to create customized greeting functions.
    
    ### **3. Maintaining State in Asynchronous Code**
    
    Closures are particularly useful in asynchronous programming, where you need to maintain state across callbacks.
    
    **Example:**
    
    ```jsx
    function createTimer() {
      let start = Date.now();
    
      return function() {
        let elapsed = Date.now() - start;
        console.log(`Elapsed time: ${elapsed}ms`);
      };
    }
    
    const timer = createTimer();
    
    setTimeout(timer, 1000); // Output: 'Elapsed time: 1000ms'
    setTimeout(timer, 2000); // Output: 'Elapsed time: 2000ms'
    ```
    
    Here, the `createTimer` function returns a closure that retains the `start` time. Even though the function is executed after a delay, it still correctly calculates the elapsed time because it has access to the `start` variable.
    
    ### **4. Iterators and Generators**
    
    Closures can be used to create iterators, which are functions that keep track of their progress.
    
    **Example:**
    
    ```jsx
    javascriptCopy code
    function createIterator(array) {
      let index = 0;
    
      return function() {
        if (index < array.length) {
          return array[index++];
        } else {
          return null;
        }
      };
    }
    
    const nextItem = createIterator([1, 2, 3]);
    
    console.log(nextItem()); // Output: 1
    console.log(nextItem()); // Output: 2
    console.log(nextItem()); // Output: 3
    console.log(nextItem()); // Output: null
    
    ```
    
    In this example, the `createIterator` function returns a closure that maintains the `index` across multiple invocations, allowing you to iterate over the array.
    
    ### **Why Closures Are Important**
    
    - **Encapsulation:** Closures allow you to encapsulate data, making it accessible only through specific functions, thus protecting it from the global scope or unintended modifications.
    - **State Management:** Closures help in managing state across function calls, especially in asynchronous operations like event handlers, timers, or promises.
    - **Functional Programming:** Closures are a key feature in functional programming, enabling techniques like partial application, currying, and function composition.
    
    ### **Conclusion**
    
    Closures are a powerful and flexible feature in JavaScript, allowing functions to retain access to their lexical scope even after the outer function has completed. This capability is crucial for data encapsulation, maintaining state, and writing modular, reusable code. Understanding closures helps you take full advantage of JavaScript's functional programming capabilities and write more efficient and maintainable code.
    
- **Explain reduce function in javascript with example**
    
    ### **Syntax of `reduce`**
    
    ```jsx
    array.reduce(callback(accumulator, currentValue, currentIndex, array), initialValue);
    ```
    
    - **`callback`:** A function that is executed on each element of the array. It takes up to four arguments:
        - **`accumulator`**: The accumulator accumulates the callback's return values. It is the accumulated value previously returned in the last invocation of the callback, or `initialValue` if provided.
        - **`currentValue`**: The current element being processed in the array.
        - **`currentIndex`** (optional): The index of the current element being processed in the array. Starts from 0 if `initialValue` is provided, otherwise from 1.
        - **`array`** (optional): The array `reduce` was called upon.
    - **`initialValue`** (optional): A value to use as the first argument to the first call of the `callback`. If no `initialValue` is supplied, the first element in the array will be used as the initial accumulator value, and the `callback` function will start from the second element.
    
    ### **Example 1: Sum of an Array**
    
    A common use case for `reduce` is to sum all the numbers in an array.
    
    ```jsx
    const numbers = [1, 2, 3, 4, 5];
    
    const sum = numbers.reduce((accumulator, currentValue) => {
        return accumulator + currentValue;
    }, 0);
    
    console.log(sum); // Output: 15
    ```
    
    - **Explanation**:
        - The `reduce` function iterates over the array, adding each `currentValue` to the `accumulator`.
        - The `initialValue` is set to `0`, so the `accumulator` starts at `0`.
        - The result is the sum of all numbers in the array, which is `15`.
    
    ### **Example 2: Flattening an Array of Arrays**
    
    You can use `reduce` to flatten a nested array into a single array.
    
    ```jsx
    javascriptCopy code
    const nestedArray = [[1, 2], [3, 4], [5, 6]];
    
    const flattenedArray = nestedArray.reduce((accumulator, currentValue) => {
        return accumulator.concat(currentValue);
    }, []);
    
    console.log(flattenedArray); // Output: [1, 2, 3, 4, 5, 6]
    ```
    
    - **Explanation**:
        - The `reduce` function is used to concatenate each sub-array (`currentValue`) to the `accumulator`, which starts as an empty array `[]`.
        - The result is a single, flattened array `[1, 2, 3, 4, 5, 6]`.
    
    ### **Example 3: Counting Occurrences of Items in an Array**
    
    You can also use `reduce` to count how many times each item appears in an array.
    
    ```jsx
    javascriptCopy code
    const fruits = ['apple', 'banana', 'orange', 'apple', 'orange', 'banana', 'apple'];
    
    const fruitCount = fruits.reduce((accumulator, currentValue) => {
        accumulator[currentValue] = (accumulator[currentValue] || 0) + 1;
        return accumulator;
    }, {});
    
    console.log(fruitCount);
    // Output: { apple: 3, banana: 2, orange: 2 }
    
    ```
    
    - **Explanation**:
        - The `reduce` function iterates over the `fruits` array.
        - For each `currentValue`, it checks if it already exists in the `accumulator` object. If it does, it increments the count; otherwise, it initializes it with `1`.
        - The result is an object that counts the occurrences of each fruit.
    
    ### **Example 4: Calculating Average Age**
    
    You can use `reduce` to calculate the average age of a group of people.
    
    ```jsx
    javascriptCopy code
    const people = [
        { name: 'Alice', age: 25 },
        { name: 'Bob', age: 30 },
        { name: 'Charlie', age: 35 }
    ];
    
    const totalAge = people.reduce((accumulator, currentValue) => {
        return accumulator + currentValue.age;
    }, 0);
    
    const averageAge = totalAge / people.length;
    
    console.log(averageAge); // Output: 30
    
    ```
    
    - **Explanation**:
        - The `reduce` function sums the ages of all people in the array.
        - The total age is then divided by the number of people to get the average age.
    
    ### **Conclusion**
    
    The `reduce` function is a versatile and powerful tool in JavaScript for reducing an array to a single value. Whether you're summing numbers, flattening arrays, counting occurrences, or calculating averages, `reduce` provides a concise and readable way to perform these operations. Understanding how `reduce` works and when to use it can greatly enhance your ability to manipulate and process arrays in JavaScript.
    
    4o
    

---

### HTML & CSS

- **HTML5**
    
    HTML5 introduced several new features and APIs to improve web development. Here are some HTML5 features with examples:
    
    1. Semantic tags: HTML5 introduced several semantic tags such as **`<header>`**, **`<footer>`**, **`<nav>`**, and **`<article>`**, which help in creating a well-structured web page.
        
        Example:
        
        ```html
        <header>
          <h1>My Website</h1>
          <nav>
            <ul>
              <li><a href="#">Home</a></li>
              <li><a href="#">About</a></li>
              <li><a href="#">Contact</a></li>
            </ul>
          </nav>
        </header>
        ```
        
    2. Canvas: The **`<canvas>`** tag provides a way to draw graphics and animations using JavaScript.
        
        Example:
        
        ```html
        <canvas id="myCanvas" width="200" height="100"></canvas>
        ```
        
    3. Video and audio: HTML5 introduced native support for playing video and audio files without the need for third-party plugins.
        
        Example:
        
        ```html
        <video controls>
          <source src="myVideo.mp4" type="video/mp4">
        </video>
        
        <audio controls>
          <source src="myAudio.mp3" type="audio/mp3">
        </audio>
        ```
        
    4. Local Storage: HTML5 introduced a new **`localStorage`** object that allows web developers to store data in the browser and retrieve it later.
        
        Example:
        
        ```jsx
        localStorage.setItem("username", "JohnDoe");
        var username = localStorage.getItem("username");
        ```
        
    5. Geolocation: HTML5 introduced the **`Geolocation`** API, which allows web developers to retrieve the user's location information.
        
        Example:
        
        ```jsx
        navigator.geolocation.getCurrentPosition(function(position) {
          var latitude = position.coords.latitude;
          var longitude = position.coords.longitude;
        });
        ```
        
    6. Web Workers: HTML5 introduced the **`Web Workers`** API, which allows web developers to run JavaScript code in the background without affecting the performance of the main page.
        
        Example:
        
        ```jsx
        // Create a new web worker
        var myWorker = new Worker("worker.js");
        
        // Send a message to the worker
        myWorker.postMessage("Hello");
        
        // Receive a message from the worker
        myWorker.onmessage = function(event) {
          console.log(event.data);
        };
        ```
        
    7. WebSockets: This allows two-way communication between client and server. Example: chat application, real-time notifications.
    8. Web Storage API: This allows client-side storage of key-value pairs without the need for cookies. Example: storing user preferences, shopping cart items.
    9. Drag and Drop API: This allows dragging and dropping of HTML elements on a web page. Example: moving items on a to-do list, dragging and dropping images into a photo editor.
    10. WebRTC: This enables real-time communication between browsers without the need for plugins. Example: video conferencing, peer-to-peer file sharing.
    11. Server-Sent Events: This allows the server to send events to the client without the need for client-side polling. Example: real-time updates of sports scores, stock prices.
    12. ContentEditable API: This allows web page content to be edited by users. Example: a rich-text editor, a collaborative document editor.
    13. Web Notifications API: This allows websites to display desktop notifications to users. Example: browser-based chat application, email notifications.
    14. Web Components: This allows creation of custom, reusable UI components. Example: a custom video player, a social media sharing button.
    15. Web Speech API: This allows the browser to recognize and interpret speech. Example: voice commands for a website, transcribing audio recordings.
    16. Web Animations API: This allows creation of complex animations and transitions on web pages. Example: animating a menu item on hover, creating a loading animation.
- **HTML Events**
    
    1. `onclick`: The `onclick` event occurs when an element is clicked by the user. This event can be used to trigger some action when a button or a link is clicked.
    2. `ondblclick`: The `ondblclick` event occurs when an element is double-clicked by the user. This event can be used to trigger some action when a user double-clicks on an element.
    3. `onmouseover`: The `onmouseover` event occurs when the user moves the mouse pointer over an element. This event can be used to trigger some action when the mouse pointer enters an element.
    4. `onmouseout`: The `onmouseout` event occurs when the user moves the mouse pointer out of an element. This event can be used to trigger some action when the mouse pointer leaves an element.
    5. `onmousedown`: The `onmousedown` event occurs when the user presses a mouse button over an element. This event can be used to trigger some action when the user clicks down on an element.
    6. `onmouseup`: The `onmouseup` event occurs when the user releases a mouse button over an element. This event can be used to trigger some action when the user clicks up on an element.
    7. `onkeydown`: The `onkeydown` event occurs when the user presses a keyboard key. This event can be used to trigger some action when the user types a key.
    8. `onkeyup`: The `onkeyup` event occurs when the user releases a keyboard key. This event can be used to trigger some action when the user stops typing a key.
    9. `onsubmit`: The `onsubmit` event occurs when a form is submitted. This event can be used to validate the user input or to trigger some action when the user submits a form.
    10. `onchange`: The `onchange` event occurs when the value of an element is changed. This event can be used to validate the user input or to trigger some action when the user changes the value of an element.
    11. `onload`: The `onload` event occurs when an element is loaded. This event can be used to trigger some action when an image or a web page is loaded.
    12. `onunload`: The `onunload` event occurs when a web page is unloaded. This event can be used to trigger some action when the user leaves a web page.
    13. `onresize`: The `onresize` event occurs when the size of a window or an element is changed. This event can be used to trigger some action when the user resizes a window.
    14. `onscroll`: The `onscroll` event occurs when the user scrolls an element. This event can be used to trigger some action when the user scrolls a web page.
    15. `onblur`: The `onblur` event occurs when an element loses focus. This event can be used to trigger some action when the user leaves an input field.
    16. `onfocus`: The `onfocus` event occurs when an element receives focus. This event can be used to trigger some action when the user clicks on an input field.
    17. `oncontextmenu`: The `oncontextmenu` event occurs when the user right-clicks on an element. This event can be used to trigger some action when the user right-clicks on an element.
    
- **Common HTML Tags**
    1. **`div`** - used to group and organize HTML elements
    2. **`span`** - used to style and manipulate specific parts of text within a larger element
    3. **`p`** - used to display paragraphs of text
    4. **`h1-h6`** - used to display different levels of headings
    5. **`ul`** - used to create an unordered list
    6. **`ol`** - used to create an ordered list
    7. **`li`** - used to create a list item within an ordered or unordered list
    8. **`a`** - used to create a hyperlink to another webpage or resource
    9. **`img`** - used to display an image on the webpage
    10. **`form`** - used to create a form for user input
    11. **`input`** - used to create various types of input fields in a form
    12. **`button`** - used to create a button that performs an action when clicked
    13. **`label`** - used to label an input field in a form
    14. **`select`** - used to create a dropdown list in a form
    15. **`option`** - used to define an option within a dropdown list
    16. **`textarea`** - used to create a larger input field for text in a form
    17. **`table`** - used to create a table
    18. **`tr`** - used to create a table row
    19. **`td`** - used to create a table data cell
    20. **`th`** - used to create a table header cell
- **CSS Box Model**
    
    The CSS Box Model is a fundamental concept in front-end web development that defines how content is displayed and organized on a web page. It consists of four main components:
    
    1. Content: The actual content of an HTML element, such as text, images, or videos.
    2. Padding: The space between the content and the border of an element. Padding can be used to create visual separation between content and its surrounding elements.
    3. Border: A line that surrounds an element, separating it from other elements on the page.
    4. Margin: The space between an element and its neighboring elements. Margins can be used to create whitespace and provide visual separation between elements.
    
    By understanding and utilizing the CSS Box Model, front-end developers can create clean and organized layouts that are visually appealing and easy to navigate. Proper use of the Box Model can also ensure that elements are displayed consistently across different devices and screen sizes.
    
    ![Screenshot 2023-03-30 at 11.44.44 PM.png](React%20Developer%20Interview%20Preparation%E2%80%99s%20268585597ad780dc8662ce69647dda2f/Screenshot_2023-03-30_at_11.44.44_PM.png)
    
- **display: none vs visibility: hidden**
    
    `display: none:` This value completely removes the element from the document flow, meaning that it takes up no space and is not rendered on the page. This is often used to hide elements that are not needed on a particular page or that should only be displayed under certain conditions.
    
    `visibility: hidden:` This value hides the element, but it still takes up space and is rendered on the page. This is often used to temporarily hide an element without affecting the layout of the page, such as when an element is being animated or loaded dynamically.
    
    In general, it's best to use "display: none" when you want to completely remove an element from the page, and "visibility: hidden" when you want to hide an element without affecting the layout. However, the choice of which to use will depend on the specific use case and design requirements.
    
- display block vs display inline
    1. **Line Behaviour**:
        - **Block**: Starts on a new line and takes up the full width available.
        - **Inline**: Does not start on a new line and only takes up as much width as its content.
    2. **Width and Height**:
        - **Block**: Width and height can be set explicitly.
        - **Inline**: Width and height are ignored; size is determined by content.
    3. **Margin and Padding**:
        - **Block**: Margin and padding affect all sides.
        - **Inline**: Margin and padding only affect left and right.
    4. **Content Flow**:
        - **Block**: Forces a new line before and after the element.
        - **Inline**: Flows within the surrounding text without disrupting the line flow.
    
    ### **Use Cases**
    
    - **Block-Level Elements**: Use for structural elements that need to be stacked vertically, such as containers, headings, and sections.
    - **Inline Elements**: Use for styling parts of text or small elements within a line, such as links, spans, and emphasis.
- **Input**
    
    Here are the different input types you can use in HTML:
    
    - `<input type="button">`
    - `<input type="checkbox">`
    - `<input type="color">`
    - `<input type="date">`
    - `<input type="datetime-local">`
    - `<input type="email">`
    - `<input type="file">`
    - `<input type="hidden">`
    - `<input type="image">`
    - `<input type="month">`
    - `<input type="number">`
    - `<input type="password">`
    - `<input type="radio">`
    - `<input type="range">`
    - `<input type="reset">`
    - `<input type="search">`
    - `<input type="submit">`
    - `<input type="tel">`
    - `<input type="text">`
    - `<input type="time">`
    - `<input type="url">`
    - `<input type="week">`

---

### Program

- To find the highest number in an array, which Math method to be used in javascript ?
    
    ```jsx
    const numbers = [3, 9, 1, 6, 4, 8, 2, 5, 7];
    const highestNumber = Math.max(...numbers);
    
    console.log(highestNumber); // Output: 9
    ```
    
- To find the lowest number in an array, which Math method to be used in javascript ?
    
    ```jsx
    const arr = [4, 2, 8, 1, 5, 9];
    const lowestNum = Math.min(...arr);
    
    console.log(lowestNum); // Output: 1
    ```
    
- Object To Array
    
    ```jsx
    // Object To Array
    const myObject = {
      name: "John",
      age: 30,
      city: "New York"
    };
    
    const myArray = Object.keys(myObject).map(key => [key, myObject[key]]);
    
    console.log(myArray);
    // Output: [["name", "John"], ["age", 30], ["city", "New York"]]
    ```
    
- flat an array of nested arrays to single array
    
    ```jsx
    var arr=[[1,2],[3,[2]],[[2,5,[4]]]];
    // return result in single array
    
    // Infinity is a keyword used in flat to spereaad all 
    // the array elemts inside the array
    console.log(arr.flat(Infinity)); 
    // Output: [1, 2, 3, 2, 2, 5, 4]
    ```
    
- Console prints order
    
    ```jsx
    // Set Time Out is a Async Function
    setTimeout(()=>{console.log('1')},0)
    console.log('2')
    setTimeout(()=>{console.log('3')},10)
    console.log('4')
    setTimeout(()=>{console.log('5')},0)
    console.log('6')
    
    // Output: 2 4 6 1 5 3
    ```
    
- Get all the name of the matched filter data in entire object
    
    ```jsx
    // Get all the name of the matched filter data in entire object
    var merchants = [
      {
        name: 'Joes\'s Couch World',
        state: 'UT',
        products: ['furniture', 'appliance']
      },
      {
        name: 'The Couch Store',
        state: 'UT',
        products: ['furniture']
      },
      {
        name: 'Frisco Tire, Wheel and Mini-Fridge',
        state: 'CA',
        products: ['auto', 'appliance']
      },
      {
        name: 'Take a Seat',
        state: 'UT',
        products: ['couch']
      }
    ];
     
    function filterMerchants(filter, merchants){
      let filtered = merchants.filter(_ => {
         const productStatus = _.products.filter(_ => _.toLowerCase() === filter.toLowerCase());
         return _.state === filter || productStatus.length > 0
      });
      filtered = mapedData(filtered,filter);
     return filtered;
    }
    
    const mapedData = (data,filter) => {
        if(!data) return
        return data.map(_ => {
            return _.name
        });
    }
     
    console.log( filterMerchants( 'UT', merchants ) );
    // Should return ['Joes\'s Couch World', 'The Couch Store', 'Take a Seat']
     
    console.log( filterMerchants( 'appliance', merchants ) );
    // Should return ['Joes\'s Couch World', 'Frisco Tire, Wheel and Mini-Fridge']
     
    console.log( filterMerchants( 'Auto', merchants ) );
    // Should return ['Frisco Tire, Wheel and Mini-Fridge']
    
    // Optimized Version
    const filterMerchants = (filter, merchants) => {
      return merchants.reduce((acc, curr) => {
        const productStatus = curr.products.filter(p => p.toLowerCase() === filter.toLowerCase());
        if (curr.state === filter || productStatus.length > 0) {
          acc.push(curr.name);
        }
        return acc;
      }, []);
    };
    ```
    
- Find the maximum repeating number from given array
    
    ```jsx
    // Find the maximum repeating number from given array
    let arr = [1, 2, 2, 2, 0, 2, 0, 2, 3, 8, 0, 9, 2, 3]
    // Result: [2, 6]
    
    function findMaxRepeating(arr) {
      let freq = {};
      let maxNum = null;
      let maxCount = 0;
    
      for (let num of arr) {
        freq[num] = (freq[num] || 0) + 1;
    
        if (freq[num] > maxCount) {
          maxCount = freq[num];
          maxNum = num;
        }
      }
    
      return [Number(maxNum), maxCount];
    }
    
    console.log(findMaxRepeating(arr)); // Output: [2, 6]
    ```
    
- Invert Binary Tree JavaScript Recursive Solution
    
    Here's an example of a recursive solution to invert a binary tree in JavaScript:
    
    ```jsx
    function invertTree(root) {
      if (!root) {
        return null;
      }
    
      const left = invertTree(root.left);
      const right = invertTree(root.right);
    
      root.left = right;
      root.right = left;
    
      return root;
    }
    ```
    
    In this solution, we use recursion to traverse the binary tree and invert its left and right nodes. We first check if the current node is null, and if so, we return null. Otherwise, we recursively call the `invertTree` function on the left and right nodes, and swap their positions. Finally, we return the root node, which has now been inverted.
    
    Note that this solution assumes that the binary tree is implemented as an object with `left` and `right` properties pointing to the left and right nodes, respectively. If your binary tree is implemented differently, you may need to modify the solution accordingly.
    
- Take the count of each word
    
    ```jsx
    let str = "Hello my name is John Hello John how are you doing today";
    
    const keyAndCount = (str) => {
      const temp = str.split(" ");
      let obj = {};
      for(let i=0; temp.length > i; i++){
        if(Object(obj).hasOwnProperty(temp[i])){
          obj[temp[i]] = obj[temp[i]] + 1;
        }else {
          obj[temp[i]] = 1;
        }
      }
      return obj
    }
    
    console.log(keyAndCount(str));
    ```
    
- Palindrome
    
    ```jsx
    // palindrome
    const str = [0,1,2,3];
    
    const checkPalindrome = (data) => {
        let reversed = "";
        for(let i = 0; str.length > i ; i++){
            reversed = str[i] + reversed;
        }
        return reversed === str
    }
    
    console.log(checkPalindrome(str));
    ```
    
- Only get uniques values in array
    
    ```jsx
    // Remove duplicates in array like
    // Input: [1,1,2,2,5,6,7,7]
    // output: [5,6];
    
    const arr = [1, 1, 2, 2, 5, 6, 7, 7];
    
    function removeDuplicates(arr) {
      // Step 1: Create a map to count the occurrences of each element
      const countMap = {};
    
      arr.forEach(num => {
        countMap[num] = (countMap[num] || 0) + 1;
      });
    
      // Step 2: Filter the array to only include elements that appear exactly once
      return arr.filter(num => countMap[num] === 1);
    }
    
    const result = removeDuplicates(arr);
    console.log(result); // Output: [5, 6]
    
    ```
    

---

### String Based Problems

### **1️⃣ Longest Substring Without Repeating Characters**

**Problem:** Determine the length of the longest substring in a given string that contains all unique characters.

**Solution:**

```jsx
function lengthOfLongestSubstring(s) {
  let seen = new Map();
  let start = 0;
  let maxLength = 0;

  for (let end = 0; end < s.length; end++) {
    if (seen.has(s[end])) {
      start = Math.max(seen.get(s[end]) + 1, start);
    }
    seen.set(s[end], end);
    maxLength = Math.max(maxLength, end - start + 1);
  }

  return maxLength;
}

// Example usage:
console.log(lengthOfLongestSubstring("abcabcbb")); // Output: 3 ("abc")
console.log(lengthOfLongestSubstring("bbbbb"));    // Output: 1 ("b")
```

**Explanation:** This approach utilizes a sliding window technique with two pointers (`start` and `end`) to track the current substring. A `Map` stores the last seen index of each character. As we iterate through the string, if a character is repeated, we adjust the `start` pointer to ensure all characters in the window are unique.

---

### **2️⃣ Longest Palindromic Substring**

**Problem:** Find the longest substring in a given string that reads the same forward and backward.

**Solution:**

```jsx
function longestPalindrome(s) {
  if (s.length < 2) return s;

  let start = 0, maxLength = 1;

  function expandAroundCenter(left, right) {
    while (left >= 0 && right < s.length && s[left] === s[right]) {
      left--;
      right++;
    }
    return right - left - 1;
  }

  for (let i = 0; i < s.length; i++) {
    let len1 = expandAroundCenter(i, i);   // Odd length palindromes
    let len2 = expandAroundCenter(i, i + 1); // Even length palindromes
    let len = Math.max(len1, len2);

    if (len > maxLength) {
      start = i - Math.floor((len - 1) / 2);
      maxLength = len;
    }
  }

  return s.substring(start, start + maxLength);
}

// Example usage:
console.log(longestPalindrome("babad")); // Output: "bab" or "aba"
console.log(longestPalindrome("cbbd"));  // Output: "bb"
```

**Explanation:** This solution employs the "expand around center" technique. For each character (and each pair of characters for even-length palindromes), it expands outward while the characters on both sides are equal, thus identifying the longest palindromic substring centered at that position.

---

### **3️⃣ String Concatenation**

**Problem:** Demonstrate how to concatenate two or more strings in JavaScript.

**Solution:**

```jsx
let str1 = "Hello";
let str2 = "World";

// Using the + operator
let result1 = str1 + " " + str2;
console.log(result1); // Output: "Hello World"

// Using the concat() method
let result2 = str1.concat(" ", str2);
console.log(result2); // Output: "Hello World"
```

**Explanation:** In JavaScript, strings can be concatenated using the `+` operator or the `concat()` method. Both methods combine multiple strings into one.

### **1️⃣ Check if Two Strings are Anagrams**

👉 Two words are **anagrams** if they have the same characters in the same frequency but in a different order.

**Example:** `"listen"` and `"silent"` are anagrams.

### **Solution**

```jsx
const isAnagram = (str1, str2) => {
  if (str1.length !== str2.length) return false;

  const sortedStr1 = str1.toLowerCase().split("").sort().join("");
  const sortedStr2 = str2.toLowerCase().split("").sort().join("");

  return sortedStr1 === sortedStr2;
};

console.log(isAnagram("listen", "silent")); // true
console.log(isAnagram("hello", "world"));   // false
```

✅ **Time Complexity:** `O(n log n)` (because of sorting)

✅ **Space Complexity:** `O(n)`

---

## **2️⃣ Reverse a String**

👉 Reverse the given string **without using built-in reverse()**.

### **Solution**

```jsx
javascript
CopyEdit
const reverseString = (str) => {
  let reversed = "";
  for (let i = str.length - 1; i >= 0; i--) {
    reversed += str[i];
  }
  return reversed;
};

console.log(reverseString("hello")); // "olleh"

```

✅ **Time Complexity:** `O(n)`

✅ **Space Complexity:** `O(n)`

---

## **3️⃣ Check if a String is a Palindrome**

👉 A **palindrome** is a word that reads the same forward and backward.

**Example:** `"racecar"`, `"madam"`

### **Solution**

```jsx
javascript
CopyEdit
const isPalindrome = (str) => {
  return str === str.split("").reverse().join("");
};

console.log(isPalindrome("racecar")); // true
console.log(isPalindrome("hello"));   // false

```

✅ **Time Complexity:** `O(n)`

✅ **Space Complexity:** `O(n)`

---

## **4️⃣ Find the First Non-Repeating Character**

👉 Find the **first character that appears only once** in the string.

**Example:** `"swiss"` → **First non-repeating character = `'w'`**

### **Solution**

```jsx
javascript
CopyEdit
const firstNonRepeatingChar = (str) => {
  const charMap = {};

  for (let char of str) {
    charMap[char] = (charMap[char] || 0) + 1;
  }

  for (let char of str) {
    if (charMap[char] === 1) return char;
  }

  return null;
};

console.log(firstNonRepeatingChar("swiss")); // "w"
console.log(firstNonRepeatingChar("aabbcc")); // null

```

✅ **Time Complexity:** `O(n)`

✅ **Space Complexity:** `O(n)`

---

## **5️⃣ Count Occurrences of Each Character**

👉 Count how many times each character appears in a string.

### **Solution**

```jsx
javascript
CopyEdit
const countCharacterFrequency = (str) => {
  const frequencyMap = {};

  for (let char of str) {
    frequencyMap[char] = (frequencyMap[char] || 0) + 1;
  }

  return frequencyMap;
};

console.log(countCharacterFrequency("hello"));
// { h: 1, e: 1, l: 2, o: 1 }

```

✅ **Time Complexity:** `O(n)`

✅ **Space Complexity:** `O(n)`

---

## **6️⃣ Find All Substrings of a String**

👉 Generate **all possible substrings** of a given string.

### **Solution**

```jsx
javascript
CopyEdit
const getAllSubstrings = (str) => {
  let substrings = [];

  for (let i = 0; i < str.length; i++) {
    for (let j = i + 1; j <= str.length; j++) {
      substrings.push(str.substring(i, j));
    }
  }

  return substrings;
};

console.log(getAllSubstrings("abc"));
// [ 'a', 'ab', 'abc', 'b', 'bc', 'c' ]

```

✅ **Time Complexity:** `O(n²)`

✅ **Space Complexity:** `O(n²)`

---

## **7️⃣ Find the Longest Common Prefix**

👉 Find the **longest prefix** that is common in an array of strings.

**Example:** `["flower", "flow", "flight"]` → **Output: `"fl"`**

### **Solution**

```jsx
javascript
CopyEdit
const longestCommonPrefix = (strs) => {
  if (!strs.length) return "";

  let prefix = strs[0];

  for (let i = 1; i < strs.length; i++) {
    while (strs[i].indexOf(prefix) !== 0) {
      prefix = prefix.slice(0, -1);
      if (!prefix) return "";
    }
  }

  return prefix;
};

console.log(longestCommonPrefix(["flower", "flow", "flight"])); // "fl"
console.log(longestCommonPrefix(["dog", "racecar", "car"])); // ""

```

✅ **Time Complexity:** `O(n * m)` (where `n` = number of strings, `m` = length of the shortest string)

✅ **Space Complexity:** `O(1)`

---

## **8️⃣ Find if One String is a Rotation of Another**

👉 Check if one string is a **rotation** of another string.

**Example:** `"waterbottle"` is a rotation of `"erbottlewat"`

### **Solution**

```jsx
javascript
CopyEdit
const isRotation = (s1, s2) => {
  if (s1.length !== s2.length) return false;
  return (s1 + s1).includes(s2);
};

console.log(isRotation("waterbottle", "erbottlewat")); // true
console.log(isRotation("hello", "llohe")); // true
console.log(isRotation("hello", "olelh")); // false

```

✅ **Time Complexity:** `O(n)`

✅ **Space Complexity:** `O(n)`

---

## **9️⃣ Find the Most Frequent Character**

👉 Find the character that appears **the most** in a string.

### **Solution**

```jsx
javascript
CopyEdit
const mostFrequentChar = (str) => {
  const freqMap = {};
  let maxChar = "", maxCount = 0;

  for (let char of str) {
    freqMap[char] = (freqMap[char] || 0) + 1;
    if (freqMap[char] > maxCount) {
      maxCount = freqMap[char];
      maxChar = char;
    }
  }

  return maxChar;
};

console.log(mostFrequentChar("aabbbccccd")); // "c"

```

✅ **Time Complexity:** `O(n)`

✅ **Space Complexity:** `O(n)`

---

### Array Based Problems

## **1️⃣ Product of Array Except Self**

**Problem:** Given an array of numbers, create a new array where each element at index `i` is the product of all the numbers in the original array except the one at `i`.

**Solution:**

```jsx
function productExceptSelf(nums) {
  const length = nums.length;
  const result = new Array(length).fill(1);
  let leftProduct = 1;
  let rightProduct = 1;

  for (let i = 0; i < length; i++) {
    result[i] *= leftProduct;
    leftProduct *= nums[i];
  }

  for (let i = length - 1; i >= 0; i--) {
    result[i] *= rightProduct;
    rightProduct *= nums[i];
  }

  return result;
}

// Example usage:
console.log(productExceptSelf([1, 2, 3, 4])); // Output: [24, 12, 8, 6]
```

**Explanation:** This approach calculates the product of all elements to the left and right of each index without using division, ensuring an efficient solution.

---

## **2️⃣ Find Unique Pairs with Given Sum**

**Problem:** Find the number of unique pairs in an array whose sum equals a given value `k`.

**Solution:**

```jsx
javascript
CopyEdit
function countUniquePairs(nums, k) {
  const seen = new Set();
  const pairs = new Set();

  for (const num of nums) {
    const complement = k - num;
    if (seen.has(complement)) {
      const pair = [num, complement].sort((a, b) => a - b);
      pairs.add(pair.toString());
    }
    seen.add(num);
  }

  return pairs.size;
}

// Example usage:
console.log(countUniquePairs([1, 5, 7, -1, 5], 6)); // Output: 2 (Pairs: [1, 5] and [7, -1])

```

**Explanation:** Using a set to track seen numbers and another set to store unique pairs ensures that each pair is counted only once.

---

## **3️⃣ Remove Duplicates from Sorted Array**

**Problem:** Given a sorted array, remove the duplicates in-place such that each element appears only once and return the new length.

**Solution:**

```jsx
javascript
CopyEdit
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let uniqueIndex = 0;

  for (let i = 1; i < nums.length; i++) {
    if (nums[i] !== nums[uniqueIndex]) {
      uniqueIndex++;
      nums[uniqueIndex] = nums[i];
    }
  }

  return uniqueIndex + 1;
}

// Example usage:
const nums = [1, 1, 2, 2, 3];
const length = removeDuplicates(nums);
console.log(length); // Output: 3
console.log(nums.slice(0, length)); // Output: [1, 2, 3]

```

**Explanation:** By maintaining a `uniqueIndex`, we can overwrite duplicates and keep only unique elements in the array.

---

## **4️⃣ Merge Intervals**

**Problem:** Given an array of intervals where intervals[i] = [starti, endi], merge all overlapping intervals.

**Solution:**

```jsx
javascript
CopyEdit
function mergeIntervals(intervals) {
  if (intervals.length <= 1) return intervals;

  intervals.sort((a, b) => a[0] - b[0]);
  const merged = [intervals[0]];

  for (let i = 1; i < intervals.length; i++) {
    const last = merged[merged.length - 1];
    const current = intervals[i];

    if (current[0] <= last[1]) {
      last[1] = Math.max(last[1], current[1]);
    } else {
      merged.push(current);
    }
  }

  return merged;
}

// Example usage:
console.log(mergeIntervals([[1, 3], [2, 6], [8, 10], [15, 18]]));
// Output: [[1, 6], [8, 10], [15, 18]]

```

**Explanation:** After sorting the intervals by their start times, we iterate through and merge overlapping intervals by comparing the current interval's start with the last merged interval's end.

---

## **5️⃣ Rotate Array**

**Problem:** Rotate an array to the right by `k` steps, where `k` is non-negative.

**Solution:**

```jsx
javascript
CopyEdit
function rotateArray(nums, k) {
  k = k % nums.length;
  reverse(nums, 0, nums.length - 1);
  reverse(nums, 0, k - 1);
  reverse(nums, k, nums.length - 1);
}

function reverse(nums, start, end) {
  while (start < end) {
    [nums[start], nums[end]] = [nums[end], nums[start]];
    start++;
    end--;
  }
}

// Example usage:
const nums = [1, 2, 3, 4, 5, 6, 7];
rotateArray(nums, 3);
console.log(nums); // Output: [5, 6, 7, 1, 2, 3, 4]

```

**Explanation:** This solution involves reversing the entire array, then reversing the first `k` elements, and finally reversing the remaining elements to achieve the desired rotation.

---

### Find Duplicates in array

```jsx
function findDuplicates(array) {
  const seen = new Set();
  const duplicates = [];
  
  for (const item of array) {
    if (seen.has(item)) {
      duplicates.push(item);
    } else {
      seen.add(item);
    }
  }
  
  return duplicates;
}

findDuplicates([1, 2, 3, 1, 2, 4, 5]); // [1, 2]
```

---