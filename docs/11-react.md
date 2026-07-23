# 11. React Fundamentals - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 30

---

## React Basics

### What is React?
```
JavaScript library for building UIs with reusable components

Virtual DOM: React maintains in-memory representation of UI
            ↓
       Diffing Algorithm: Compares old & new Virtual DOM
            ↓
     Minimal updates to real DOM (efficient)
```

---

## 1️⃣ JSX and Components

### JSX Syntax:

```jsx
// JSX looks like HTML but is JavaScript
const element = <h1>Hello, {name}!</h1>;

// Compiles to:
// const element = React.createElement('h1', null, 'Hello, ' + name + '!');

// JSX expressions
const greeting = (
  <div>
    <h1>Welcome</h1>
    <p>Today is {new Date().toLocaleDateString()}</p>
    <button disabled={isLoading}>Click me</button>
  </div>
);
```

### Functional Components (Modern):

```jsx
// Simple component
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// With props
function Greeting({ name, age }) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Age: {age}</p>
    </div>
  );
}

// Default props
Greeting.defaultProps = {
  name: 'Guest',
  age: 18
};

// Usage
<Greeting name="John" age={30} />
<Greeting />  // Uses defaults

// Arrow function component
const Welcome = ({ name }) => <h1>Hello, {name}!</h1>;

// With fragment (no wrapper div)
function App() {
  return (
    <>
      <Header />
      <Main />
      <Footer />
    </>
  );
}
```

### Class Components (Legacy):

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// With constructor
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

---

## 2️⃣ State & Props

### Props (Read-only):

```jsx
// Parent to child communication
function Parent() {
  return <Child name="John" age={30} greet={handleGreet} />;
}

function Child({ name, age, greet }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>Age: {age}</p>
      <button onClick={() => greet(name)}>Greet</button>
    </div>
  );
}
```

### State (Mutable):

```jsx
// Using useState hook
function Counter() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <input 
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
    </div>
  );
}

// State object
function Form() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setUser(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  return (
    <form>
      <input 
        name="name" 
        value={user.name} 
        onChange={handleChange}
      />
      <input 
        name="email" 
        value={user.email} 
        onChange={handleChange}
      />
    </form>
  );
}
```

---

## 3️⃣ Hooks

### useState:

```jsx
const [state, setState] = useState(initialValue);

// Lazy initialization
const [state, setState] = useState(() => computeExpensiveValue());
```

### useEffect:

```jsx
// Run after every render
useEffect(() => {
  console.log('Component rendered');
});

// Run only on mount
useEffect(() => {
  console.log('Component mounted');
  return () => console.log('Component unmounted');  // Cleanup
}, []);

// Run when dependencies change
useEffect(() => {
  console.log('Count changed:', count);
}, [count]);

// Multiple dependencies
useEffect(() => {
  console.log('Name or age changed');
}, [name, age]);

// Fetch data
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

### useContext:

```jsx
// Create context
const ThemeContext = React.createContext();

// Provider
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main />
    </ThemeContext.Provider>
  );
}

// Consumer
function Header() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <header className={theme}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </header>
  );
}
```

### useReducer:

```jsx
function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  function reducer(state, action) {
    switch(action.type) {
      case 'INCREMENT':
        return { count: state.count + 1 };
      case 'DECREMENT':
        return { count: state.count - 1 };
      case 'RESET':
        return { count: 0 };
      default:
        return state;
    }
  }
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}
```

### useMemo & useCallback:

```jsx
// useMemo: Memoize expensive computation
function ExpensiveComponent({ items }) {
  const sortedItems = useMemo(() => {
    console.log('Sorting...');
    return items.sort((a, b) => a - b);
  }, [items]);
  
  return <div>{sortedItems.join(', ')}</div>;
}

// useCallback: Memoize function reference
function Parent() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);  // Function never changes
  
  return <Child onClick={handleClick} />;
}
```

### Custom Hooks:

```jsx
// Custom hook for form handling
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(values);
  };
  
  return { values, errors, handleChange, handleSubmit };
}

// Usage
function LoginForm() {
  const { values, handleChange, handleSubmit } = useForm(
    { email: '', password: '' },
    (values) => console.log('Form submitted:', values)
  );
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        name="email" 
        value={values.email}
        onChange={handleChange}
      />
      <input 
        name="password" 
        type="password"
        value={values.password}
        onChange={handleChange}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## 4️⃣ Event Handling

```jsx
function EventHandling() {
  const handleClick = () => console.log('Clicked');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted');
  };
  
  const handleChange = (e) => {
    console.log('Input value:', e.target.value);
  };
  
  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed');
    }
  };
  
  return (
    <div>
      <button onClick={handleClick}>Click</button>
      <form onSubmit={handleSubmit}>
        <input onChange={handleChange} onKeyPress={handleKeyPress} />
        <button type="submit">Submit</button>
      </form>
    </div>
  );
}
```

---

## 5️⃣ Conditional Rendering

```jsx
function Dashboard({ isLoggedIn, userRole }) {
  // If-else
  if (!isLoggedIn) {
    return <LoginPage />;
  }
  
  // Ternary operator
  return (
    <div>
      {isLoggedIn ? <Welcome /> : <LoginPage />}
      
      {/* Logical AND */}
      {userRole === 'admin' && <AdminPanel />}
      
      {/* Switch-like pattern */}
      {userRole === 'admin' && <AdminPanel />}
      {userRole === 'user' && <UserPanel />}
      {userRole === 'guest' && <GuestPanel />}
      
      {/* Better switch pattern */}
      {(() => {
        switch(userRole) {
          case 'admin':
            return <AdminPanel />;
          case 'user':
            return <UserPanel />;
          default:
            return <GuestPanel />;
        }
      })()}
    </div>
  );
}
```

---

## 6️⃣ Lists and Keys

```jsx
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>  {/* Always use unique key */}
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
}

// ❌ Bad: Using index as key
function BadList({ users }) {
  return (
    <ul>
      {users.map((user, index) => (
        <li key={index}>{user.name}</li>  // ❌ Can cause issues
      ))}
    </ul>
  );
}

// ✅ Good: Using unique identifier
function GoodList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>  // ✅ Unique ID
      ))}
    </ul>
  );
}
```

---

## 7️⃣ Form Handling

```jsx
function SignUpForm() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    password: '',
    subscribe: false
  });
  
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const validate = () => {
    const newErrors = {};
    if (!formData.firstName) newErrors.firstName = 'First name required';
    if (!formData.email) newErrors.email = 'Email required';
    if (formData.password.length < 8) newErrors.password = 'Min 8 characters';
    return newErrors;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    
    if (Object.keys(newErrors).length === 0) {
      console.log('Form valid, submit:', formData);
      // Submit to API
    } else {
      setErrors(newErrors);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>First Name:</label>
        <input
          name="firstName"
          value={formData.firstName}
          onChange={handleChange}
        />
        {errors.firstName && <span>{errors.firstName}</span>}
      </div>
      
      <div>
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span>{errors.email}</span>}
      </div>
      
      <div>
        <label>Password:</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <span>{errors.password}</span>}
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="subscribe"
            checked={formData.subscribe}
            onChange={handleChange}
          />
          Subscribe to newsletter
        </label>
      </div>
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

---

## Real-World Example: Todo App

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false }
  ]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, {
        id: Date.now(),
        text: input,
        completed: false
      }]);
      setInput('');
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  const completedCount = todos.filter(t => t.completed).length;
  
  return (
    <div>
      <h1>Todo App</h1>
      <p>{completedCount}/{todos.length} completed</p>
      
      <div>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add a todo..."
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Interview Tips:

✅ **Understand Virtual DOM and reconciliation**

✅ **Know difference between state and props**

✅ **Master useState, useEffect, useContext hooks**

✅ **Explain component lifecycle**

✅ **Know when to use useMemo and useCallback**

✅ **Understand keys in lists**

✅ **Know controlled vs uncontrolled components**

---

📖 **Next Topic:** [JavaScript ES6+](./12-javascript-es6.md)
