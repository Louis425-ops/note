# React + TypeScript

**React 方式：**

```jsx
const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  const [newTodo, setNewTodo] = useState("");

  const handleAddTodo = () => {
    setTodos([...todos, { text: newTodo, completed: false }]);
    setNewTodo("");
  };

  return (
    <div className="todo-app">
      <input 
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="Add a new todo"
      />
      <button onClick={handleAddTodo}>Add</button>
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
};
```

## React 核心语法

### 1. 严格的闭合标签

React 要求所有标签必须闭合，且一个函数只能返回一个根标签：

```jsx
function App() {
  return (
    <>
      <h1>hello</h1>
      <br/>
      <p>next</p>
    </>
  );
}
```

### 2. 组件导出与导入

```jsx
// 导出组件
function MyComponent() {
  return <div>Hello, World!</div>;
}
export default MyComponent;

// 导入并使用
import MyComponent from './MyComponent';
function App() {
  return (
    <div>
      <MyComponent />
    </div>
  );
}
```

**注意：** React 组件名必须以大写字母开头！

### 3. 事件响应

```jsx
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }
  
  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

### 4. 组件状态管理

使用 `useState` Hook 管理组件状态：

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        +1
      </button>
    </div>
  );
}
```

**重要提醒：**

- 永远不要直接修改状态（如 `count = count + 1`）
- 必须使用状态更新函数（如 `setCount`）

### 5. Props 传递参数

**父组件传递 props：**

```jsx
function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

**子组件接收 props：**

```jsx
function Avatar({ person, size }) {
  return (
    <img
      src={person.imageId}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

**使用展开语法简化：**

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

### 6. 列表渲染 - map 方法

```jsx
const users = [
  { id: 1, name: 'Alice', age: 25 },
  { id: 2, name: 'Bob', age: 30 },
  { id: 3, name: 'Charlie', age: 35 }
];

const UserList = () => {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          {user.name} - {user.age}
        </li>
      ))}
    </ul>
  );
};
```

**重要：** 每个列表项必须有唯一的 `key` 属性！

------

## React Hooks

### useState 

```jsx
function CounterWithIssue() {
  const [count, setCount] = useState(0);
  
  function startCountdown() {
    setInterval(() => {
      setCount(count + 1); // 会导致闭包问题
    }, 1000);
  }
  
  // 正确做法：
  // setCount(count => count + 1); // 这个做法是对的
}
```

### useEffect

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(count => count + 1);
    }, 1000);
    
    // 清理函数
    return () => clearInterval(interval);
  }, []); // 依赖数组
  
  return <div>Count: {count}</div>;
}
```

### 路由配置

使用 `react-router-dom`：

```bash
npm install react-router-dom
// main.tsx
import { BrowserRouter } from 'react-router-dom';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
);

// App.tsx
import { Routes, Route } from 'react-router-dom';

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
    </Routes>
  );
}
```

------

## TypeScript 基础

### 基础语法

#### 1. 类型标注

```typescript
let name: string = "Alice";
let age: number = 25;
let isStudent: boolean = true;

// 联合类型
let value: number | string = "hello";

// 函数类型标注
function greet(name: string): string {
  return `Hello, ${name}!`;
}
```

#### 2. 接口定义

```typescript
interface User {
  name: string;
  age: number;
  email?: string; // 可选属性
  readonly id: number; // 只读属性
}

const user: User = {
  id: 1,
  name: "Alice",
  age: 25
};
```

#### 3. 泛型

```typescript
// 泛型函数
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

// 泛型接口
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message: string;
}
```

#### 4. 类型断言

```typescript
let value: any = "Hello TypeScript";
let length: number = (value as string).length;
// 或者
let length2: number = (<string>value).length;
```