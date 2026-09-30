# Part 17: useReducer Hook

## Step 376-405 | ระดับ: กลาง-สูง

---

## สารบัญ

- [Step 376: useReducer คืออะไร?](#step-376-usereducer-คืออะไร)
- [Step 377: Reducer Pattern](#step-377-reducer-pattern)
- [Step 378: Actions และ Dispatch](#step-378-actions-และ-dispatch)
- [Step 379: useReducer Syntax](#step-379-usereducer-syntax)
- [Step 380: useReducer vs useState](#step-380-usereducer-vs-usestate)
- [Step 381: Complex State Management](#step-381-complex-state-management)
- [Step 382: Immer กับ useReducer](#step-382-immer-กับ-usereducer)
- [Step 383: ตัวอย่าง Todo App](#step-383-ตัวอย่าง-todo-app)
- [Step 384: ตัวอย่าง Form Reducer](#step-384-ตัวอย่าง-form-reducer)
- [Step 385: ตัวอย่าง Shopping Cart Reducer](#step-385-ตัวอย่าง-shopping-cart-reducer)
- [Step 386: Middleware-like Patterns](#step-386-middleware-like-patterns)
- [Step 387: Testing Reducers](#step-387-testing-reducers)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 376: useReducer คืออะไร?

`useReducer` เป็น Hook ที่จัดการ state ที่ซับซ้อนโดยใช้ **Reducer Pattern** (เหมือนกับ Redux)

### แนวคิดหลัก

```
State + Action → Reducer Function → New State

(currentState, action) => newState
```

### เมื่อไหร่ใช้ useReducer?

- State มีหลาย sub-values ที่เกี่ยวข้องกัน
- Next state ขึ้นอยู่กับ previous state ซับซ้อน
- Logic การ update state ยุ่งยาก
- ต้องการ centralized state updates

---

## Step 377: Reducer Pattern

```jsx
// Reducer เป็น pure function
// รับ state ปัจจุบัน + action → return state ใหม่
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
      
    case 'DECREMENT':
      return { count: state.count - 1 };
      
    case 'RESET':
      return { count: 0 };
      
    case 'SET_VALUE':
      return { count: action.payload };
      
    default:
      // ✅ ควร throw error สำหรับ unknown actions
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

// Reducer ต้องเป็น pure function:
// ✅ ไม่มี side effects
// ✅ ไม่แก้ไข state โดยตรง (immutable)
// ✅ Same input → Same output เสมอ

// ❌ ผิด - mutate state โดยตรง
function badReducer(state, action) {
  state.count += 1; // ❌ Mutation!
  return state;
}

// ✅ ถูก - return new object
function goodReducer(state, action) {
  return { ...state, count: state.count + 1 }; // ✅ New object
}
```

---

## Step 378: Actions และ Dispatch

```jsx
// Action คือ object ที่บอกว่าจะเกิดอะไรขึ้น
// ต้องมี 'type' property
const action1 = { type: 'INCREMENT' };
const action2 = { type: 'SET_VALUE', payload: 42 };
const action3 = { type: 'UPDATE_USER', payload: { name: 'Alice', age: 25 } };

// Action Creators - functions ที่สร้าง action objects
const increment = () => ({ type: 'INCREMENT' });
const setValue = (value) => ({ type: 'SET_VALUE', payload: value });
const updateUser = (data) => ({ type: 'UPDATE_USER', payload: data });

// Dispatch ส่ง action ไปยัง reducer
dispatch(increment());
dispatch(setValue(42));
dispatch(updateUser({ name: 'Bob' }));
```

---

## Step 379: useReducer Syntax

```jsx
import { useReducer } from 'react';

// Basic syntax
const [state, dispatch] = useReducer(reducer, initialState);

// With initializer function (lazy initialization)
const [state, dispatch] = useReducer(reducer, initialArg, initFunction);

// ตัวอย่าง: Counter
function counter(state, action) {
  switch (action.type) {
    case 'INCREMENT': return { count: state.count + (action.payload || 1) };
    case 'DECREMENT': return { count: state.count - (action.payload || 1) };
    case 'RESET': return { count: 0 };
    default: throw new Error(`Unknown: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counter, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT', payload: 5 })}>-5</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'INCREMENT', payload: 5 })}>+5</button>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
    </div>
  );
}
```

### Lazy Initialization

```jsx
function initializeState(initialCount) {
  // ทำ expensive computation ครั้งเดียว
  return { 
    count: initialCount,
    history: [],
    lastUpdated: new Date()
  };
}

function Counter({ initialCount = 0 }) {
  // initializeState จะรันครั้งเดียวตอน mount
  const [state, dispatch] = useReducer(counter, initialCount, initializeState);
  
  return <div>Count: {state.count}</div>;
}
```

---

## Step 380: useReducer vs useState

### useState - ใช้สำหรับ:

```jsx
// Simple, independent state values
const [isOpen, setIsOpen] = useState(false);
const [count, setCount] = useState(0);
const [name, setName] = useState('');

// Simple object state
const [user, setUser] = useState({ name: '', email: '' });
function updateName(name) {
  setUser(prev => ({ ...prev, name }));
}
```

### useReducer - ใช้สำหรับ:

```jsx
// Complex state with multiple sub-values
const [state, dispatch] = useReducer(formReducer, {
  name: '',
  email: '',
  password: '',
  isValid: false,
  errors: {},
  isSubmitting: false,
  submitCount: 0
});

// Next state depends on previous state in complex ways
dispatch({ type: 'SUBMIT_START' }); // sets isSubmitting: true
dispatch({ type: 'SUBMIT_SUCCESS', payload: data }); // resets form
dispatch({ type: 'SUBMIT_ERROR', payload: error }); // sets errors
```

### เปรียบเทียบ Side by Side

```jsx
// useState version (กระจัดกระจาย)
function TodoApp_useState() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const addTodo = (text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
  };
  
  const toggleTodo = (id) => {
    setTodos(prev => prev.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  // ... more handlers
}

// useReducer version (centralized)
const todoReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return { 
        ...state, 
        todos: [...state.todos, { id: Date.now(), text: action.payload, completed: false }]
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo => 
          todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
        )
      };
    case 'DELETE_TODO':
      return { ...state, todos: state.todos.filter(t => t.id !== action.payload) };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    case 'FETCH_START':
      return { ...state, isLoading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, todos: action.payload, isLoading: false };
    case 'FETCH_ERROR':
      return { ...state, error: action.payload, isLoading: false };
    default:
      return state;
  }
};

function TodoApp_useReducer() {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all',
    isLoading: false,
    error: null
  });
  
  const addTodo = (text) => dispatch({ type: 'ADD_TODO', payload: text });
  const toggleTodo = (id) => dispatch({ type: 'TOGGLE_TODO', payload: id });
  const deleteTodo = (id) => dispatch({ type: 'DELETE_TODO', payload: id });
}
```

---

## Step 381: Complex State Management

### Multi-step Form

```jsx
const formReducer = (state, action) => {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        fields: {
          ...state.fields,
          [action.payload.field]: action.payload.value
        },
        errors: {
          ...state.errors,
          [action.payload.field]: '' // clear error on change
        }
      };
      
    case 'SET_ERRORS':
      return { ...state, errors: action.payload };
      
    case 'NEXT_STEP':
      return { ...state, currentStep: state.currentStep + 1 };
      
    case 'PREV_STEP':
      return { ...state, currentStep: Math.max(0, state.currentStep - 1) };
      
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true, submitError: null };
      
    case 'SUBMIT_SUCCESS':
      return { ...state, isSubmitting: false, isComplete: true };
      
    case 'SUBMIT_ERROR':
      return { ...state, isSubmitting: false, submitError: action.payload };
      
    case 'RESET':
      return initialFormState;
      
    default:
      return state;
  }
};

const initialFormState = {
  currentStep: 0,
  isSubmitting: false,
  isComplete: false,
  submitError: null,
  fields: {
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    address: '',
    city: '',
    country: '',
    cardNumber: '',
    expiry: '',
    cvv: ''
  },
  errors: {}
};

function MultiStepForm() {
  const [state, dispatch] = useReducer(formReducer, initialFormState);
  
  const setField = (field, value) => dispatch({ 
    type: 'SET_FIELD', 
    payload: { field, value } 
  });
  
  const steps = [
    {
      title: 'ข้อมูลส่วนตัว',
      fields: ['firstName', 'lastName', 'email', 'phone'],
      render: () => (
        <div>
          <input
            placeholder="ชื่อ"
            value={state.fields.firstName}
            onChange={e => setField('firstName', e.target.value)}
          />
          <input
            placeholder="นามสกุล"
            value={state.fields.lastName}
            onChange={e => setField('lastName', e.target.value)}
          />
          <input
            placeholder="Email"
            value={state.fields.email}
            onChange={e => setField('email', e.target.value)}
          />
          <input
            placeholder="เบอร์โทร"
            value={state.fields.phone}
            onChange={e => setField('phone', e.target.value)}
          />
        </div>
      )
    },
    {
      title: 'ที่อยู่จัดส่ง',
      render: () => (
        <div>
          <input
            placeholder="ที่อยู่"
            value={state.fields.address}
            onChange={e => setField('address', e.target.value)}
          />
          <input
            placeholder="เมือง"
            value={state.fields.city}
            onChange={e => setField('city', e.target.value)}
          />
          <input
            placeholder="ประเทศ"
            value={state.fields.country}
            onChange={e => setField('country', e.target.value)}
          />
        </div>
      )
    },
    {
      title: 'ชำระเงิน',
      render: () => (
        <div>
          <input
            placeholder="หมายเลขบัตร"
            value={state.fields.cardNumber}
            onChange={e => setField('cardNumber', e.target.value)}
          />
          <input
            placeholder="MM/YY"
            value={state.fields.expiry}
            onChange={e => setField('expiry', e.target.value)}
          />
          <input
            placeholder="CVV"
            value={state.fields.cvv}
            onChange={e => setField('cvv', e.target.value)}
          />
        </div>
      )
    }
  ];
  
  const currentStepData = steps[state.currentStep];
  
  const handleNext = () => {
    dispatch({ type: 'NEXT_STEP' });
  };
  
  const handlePrev = () => {
    dispatch({ type: 'PREV_STEP' });
  };
  
  const handleSubmit = async () => {
    dispatch({ type: 'SUBMIT_START' });
    try {
      await submitForm(state.fields);
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', payload: error.message });
    }
  };
  
  if (state.isComplete) {
    return <div>✅ สั่งซื้อสำเร็จ! ขอบคุณที่ใช้บริการ</div>;
  }
  
  return (
    <div style={{ maxWidth: '500px', margin: '0 auto' }}>
      {/* Progress */}
      <div style={{ display: 'flex', marginBottom: '24px' }}>
        {steps.map((step, i) => (
          <div key={i} style={{ 
            flex: 1, 
            textAlign: 'center',
            color: i === state.currentStep ? '#007bff' : i < state.currentStep ? '#28a745' : '#ccc'
          }}>
            <div style={{ 
              width: '30px', height: '30px', 
              borderRadius: '50%',
              border: `2px solid currentColor`,
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              margin: '0 auto 4px'
            }}>
              {i < state.currentStep ? '✓' : i + 1}
            </div>
            <small>{step.title}</small>
          </div>
        ))}
      </div>
      
      <h2>{currentStepData.title}</h2>
      {currentStepData.render()}
      
      {state.submitError && (
        <p style={{ color: 'red' }}>{state.submitError}</p>
      )}
      
      <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: '20px' }}>
        <button onClick={handlePrev} disabled={state.currentStep === 0}>
          ← ย้อนกลับ
        </button>
        {state.currentStep < steps.length - 1 ? (
          <button onClick={handleNext}>
            ถัดไป →
          </button>
        ) : (
          <button onClick={handleSubmit} disabled={state.isSubmitting}>
            {state.isSubmitting ? 'กำลังส่ง...' : 'ยืนยันคำสั่งซื้อ'}
          </button>
        )}
      </div>
    </div>
  );
}
```

---

## Step 382: Immer กับ useReducer

Immer ทำให้เขียน reducer ได้ง่ายขึ้นโดยใช้ mutable syntax

```bash
npm install immer
```

```jsx
import { produce } from 'immer';

// ❌ ไม่ใช้ Immer - verbose immutable updates
const todoReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return { ...state, todos: [...state.todos, action.payload] };
    case 'UPDATE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, ...action.payload.updates }
            : todo
        )
      };
    case 'DELETE_NESTED':
      return {
        ...state,
        data: {
          ...state.data,
          users: {
            ...state.data.users,
            [action.payload.userId]: {
              ...state.data.users[action.payload.userId],
              posts: state.data.users[action.payload.userId].posts.filter(
                p => p.id !== action.payload.postId
              )
            }
          }
        }
      };
  }
};

// ✅ ใช้ Immer - เขียนง่ายกว่ามาก
const todoReducerWithImmer = produce((draft, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      draft.todos.push(action.payload); // ✅ mutation โดยตรง (Immer จัดการ immutability ให้)
      break;
    case 'UPDATE_TODO': {
      const todo = draft.todos.find(t => t.id === action.payload.id);
      if (todo) Object.assign(todo, action.payload.updates); // ✅
      break;
    }
    case 'DELETE_NESTED':
      const { userId, postId } = action.payload;
      draft.data.users[userId].posts = draft.data.users[userId].posts.filter(
        p => p.id !== postId
      ); // ✅ กว่าจะเขียนแบบ immutable ยาวมาก
      break;
  }
});

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducerWithImmer, { todos: [] });
  
  return <div>...</div>;
}
```

---

## Step 383: ตัวอย่าง Todo App

```jsx
// todo-reducer.js
const TODO_ACTIONS = {
  ADD: 'ADD_TODO',
  TOGGLE: 'TOGGLE_TODO',
  DELETE: 'DELETE_TODO',
  EDIT: 'EDIT_TODO',
  SET_FILTER: 'SET_FILTER',
  CLEAR_COMPLETED: 'CLEAR_COMPLETED',
  REORDER: 'REORDER_TODOS'
};

const todoReducer = (state, action) => {
  switch (action.type) {
    case TODO_ACTIONS.ADD:
      if (!action.payload.text.trim()) return state;
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload.text.trim(),
            completed: false,
            createdAt: new Date().toISOString(),
            priority: action.payload.priority || 'medium'
          }
        ]
      };
      
    case TODO_ACTIONS.TOGGLE:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed, updatedAt: new Date().toISOString() }
            : todo
        )
      };
      
    case TODO_ACTIONS.DELETE:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case TODO_ACTIONS.EDIT:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text, updatedAt: new Date().toISOString() }
            : todo
        )
      };
      
    case TODO_ACTIONS.SET_FILTER:
      return { ...state, filter: action.payload };
      
    case TODO_ACTIONS.CLEAR_COMPLETED:
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
      
    default:
      return state;
  }
};

// Todo App Component
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all'
  });
  
  const [inputText, setInputText] = useState('');
  const [editingId, setEditingId] = useState(null);
  const [editText, setEditText] = useState('');

  const filteredTodos = useMemo(() => {
    switch (state.filter) {
      case 'active': return state.todos.filter(t => !t.completed);
      case 'completed': return state.todos.filter(t => t.completed);
      default: return state.todos;
    }
  }, [state.todos, state.filter]);
  
  const stats = useMemo(() => ({
    total: state.todos.length,
    active: state.todos.filter(t => !t.completed).length,
    completed: state.todos.filter(t => t.completed).length
  }), [state.todos]);

  const handleAdd = (e) => {
    e.preventDefault();
    dispatch({ type: TODO_ACTIONS.ADD, payload: { text: inputText } });
    setInputText('');
  };

  const handleEdit = (id) => {
    const todo = state.todos.find(t => t.id === id);
    setEditingId(id);
    setEditText(todo.text);
  };

  const handleSaveEdit = (id) => {
    dispatch({ type: TODO_ACTIONS.EDIT, payload: { id, text: editText } });
    setEditingId(null);
  };

  return (
    <div style={{ maxWidth: '600px', margin: '40px auto', padding: '0 16px' }}>
      <h1 style={{ textAlign: 'center', color: '#333' }}>📝 Todo App</h1>
      
      {/* Stats */}
      <div style={{ display: 'flex', gap: '16px', marginBottom: '20px', padding: '12px', background: '#f8f9fa', borderRadius: '8px' }}>
        <span>ทั้งหมด: <strong>{stats.total}</strong></span>
        <span>กำลังทำ: <strong>{stats.active}</strong></span>
        <span>เสร็จแล้ว: <strong>{stats.completed}</strong></span>
      </div>
      
      {/* Add Todo Form */}
      <form onSubmit={handleAdd} style={{ display: 'flex', gap: '8px', marginBottom: '20px' }}>
        <input
          type="text"
          value={inputText}
          onChange={e => setInputText(e.target.value)}
          placeholder="เพิ่มงานใหม่..."
          style={{ flex: 1, padding: '10px 14px', border: '2px solid #e0e0e0', borderRadius: '8px', fontSize: '16px' }}
        />
        <button 
          type="submit"
          disabled={!inputText.trim()}
          style={{ padding: '10px 20px', background: '#007bff', color: 'white', border: 'none', borderRadius: '8px', cursor: 'pointer', fontSize: '16px' }}
        >
          เพิ่ม
        </button>
      </form>
      
      {/* Filter Tabs */}
      <div style={{ display: 'flex', gap: '8px', marginBottom: '16px' }}>
        {['all', 'active', 'completed'].map(f => (
          <button
            key={f}
            onClick={() => dispatch({ type: TODO_ACTIONS.SET_FILTER, payload: f })}
            style={{
              padding: '6px 12px',
              borderRadius: '4px',
              border: '1px solid #ccc',
              background: state.filter === f ? '#007bff' : 'white',
              color: state.filter === f ? 'white' : '#333',
              cursor: 'pointer'
            }}
          >
            {f === 'all' ? 'ทั้งหมด' : f === 'active' ? 'กำลังทำ' : 'เสร็จแล้ว'}
          </button>
        ))}
        {stats.completed > 0 && (
          <button
            onClick={() => dispatch({ type: TODO_ACTIONS.CLEAR_COMPLETED })}
            style={{ marginLeft: 'auto', padding: '6px 12px', background: 'none', border: '1px solid #dc3545', color: '#dc3545', borderRadius: '4px', cursor: 'pointer' }}
          >
            ลบที่เสร็จแล้ว
          </button>
        )}
      </div>
      
      {/* Todo List */}
      <ul style={{ listStyle: 'none', padding: 0, margin: 0 }}>
        {filteredTodos.length === 0 ? (
          <li style={{ textAlign: 'center', color: '#999', padding: '40px' }}>
            ไม่มีงานใน filter นี้
          </li>
        ) : (
          filteredTodos.map(todo => (
            <li
              key={todo.id}
              style={{
                display: 'flex', alignItems: 'center', gap: '12px',
                padding: '12px 16px',
                marginBottom: '8px',
                background: 'white',
                border: '1px solid #e0e0e0',
                borderRadius: '8px',
                opacity: todo.completed ? 0.6 : 1
              }}
            >
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => dispatch({ type: TODO_ACTIONS.TOGGLE, payload: todo.id })}
                style={{ width: '18px', height: '18px', cursor: 'pointer' }}
              />
              
              {editingId === todo.id ? (
                <input
                  value={editText}
                  onChange={e => setEditText(e.target.value)}
                  onBlur={() => handleSaveEdit(todo.id)}
                  onKeyDown={e => e.key === 'Enter' && handleSaveEdit(todo.id)}
                  autoFocus
                  style={{ flex: 1, padding: '4px 8px', border: '1px solid #007bff', borderRadius: '4px' }}
                />
              ) : (
                <span
                  style={{
                    flex: 1,
                    textDecoration: todo.completed ? 'line-through' : 'none',
                    color: todo.completed ? '#999' : '#333'
                  }}
                  onDoubleClick={() => handleEdit(todo.id)}
                >
                  {todo.text}
                </span>
              )}
              
              <div style={{ display: 'flex', gap: '4px' }}>
                <button
                  onClick={() => handleEdit(todo.id)}
                  style={{ background: 'none', border: 'none', cursor: 'pointer', fontSize: '16px' }}
                  title="แก้ไข"
                >
                  ✏️
                </button>
                <button
                  onClick={() => dispatch({ type: TODO_ACTIONS.DELETE, payload: todo.id })}
                  style={{ background: 'none', border: 'none', cursor: 'pointer', fontSize: '16px' }}
                  title="ลบ"
                >
                  🗑️
                </button>
              </div>
            </li>
          ))
        )}
      </ul>
    </div>
  );
}
```

---

## Step 384: ตัวอย่าง Form Reducer

```jsx
const formReducer = (state, action) => {
  switch (action.type) {
    case 'CHANGE':
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
        errors: { ...state.errors, [action.field]: '' },
        isDirty: true
      };
      
    case 'BLUR':
      return {
        ...state,
        touched: { ...state.touched, [action.field]: true }
      };
      
    case 'SET_ERRORS':
      return { ...state, errors: action.errors };
      
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true, submitError: null };
      
    case 'SUBMIT_SUCCESS':
      return { 
        ...state, 
        isSubmitting: false, 
        isSuccess: true,
        isDirty: false
      };
      
    case 'SUBMIT_ERROR':
      return { 
        ...state, 
        isSubmitting: false, 
        submitError: action.error 
      };
      
    case 'RESET':
      return action.initialState;
      
    default:
      return state;
  }
};

function useForm({ initialValues, validate, onSubmit }) {
  const initialState = {
    values: initialValues,
    errors: {},
    touched: {},
    isSubmitting: false,
    isSuccess: false,
    isDirty: false,
    submitError: null
  };
  
  const [state, dispatch] = useReducer(formReducer, initialState);
  
  const handleChange = useCallback((field, value) => {
    dispatch({ type: 'CHANGE', field, value });
  }, []);
  
  const handleBlur = useCallback((field) => {
    dispatch({ type: 'BLUR', field });
  }, []);
  
  const handleSubmit = useCallback(async (e) => {
    e?.preventDefault();
    
    // Validate
    const errors = validate ? validate(state.values) : {};
    if (Object.keys(errors).length > 0) {
      dispatch({ type: 'SET_ERRORS', errors });
      return;
    }
    
    dispatch({ type: 'SUBMIT_START' });
    try {
      await onSubmit(state.values);
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', error: error.message });
    }
  }, [state.values, validate, onSubmit]);
  
  const reset = useCallback(() => {
    dispatch({ type: 'RESET', initialState });
  }, []);
  
  const getFieldProps = useCallback((field) => ({
    value: state.values[field] || '',
    onChange: (e) => handleChange(field, e.target.value),
    onBlur: () => handleBlur(field)
  }), [state.values, handleChange, handleBlur]);
  
  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isSubmitting: state.isSubmitting,
    isSuccess: state.isSuccess,
    isDirty: state.isDirty,
    submitError: state.submitError,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    getFieldProps
  };
}

// การใช้งาน
function RegisterForm() {
  const { values, errors, touched, isSubmitting, isSuccess, handleSubmit, getFieldProps } = useForm({
    initialValues: { name: '', email: '', password: '' },
    validate: (values) => {
      const errors = {};
      if (!values.name) errors.name = 'กรุณาใส่ชื่อ';
      if (!values.email) errors.email = 'กรุณาใส่ email';
      if (!values.email.includes('@')) errors.email = 'Email ไม่ถูกต้อง';
      if (values.password.length < 8) errors.password = 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
      return errors;
    },
    onSubmit: async (values) => {
      await fetch('/api/register', {
        method: 'POST',
        body: JSON.stringify(values)
      });
    }
  });

  if (isSuccess) return <div>✅ สมัครสมาชิกสำเร็จ!</div>;

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input placeholder="ชื่อ" {...getFieldProps('name')} />
        {touched.name && errors.name && <p style={{ color: 'red' }}>{errors.name}</p>}
      </div>
      <div>
        <input placeholder="Email" type="email" {...getFieldProps('email')} />
        {touched.email && errors.email && <p style={{ color: 'red' }}>{errors.email}</p>}
      </div>
      <div>
        <input placeholder="รหัสผ่าน" type="password" {...getFieldProps('password')} />
        {touched.password && errors.password && <p style={{ color: 'red' }}>{errors.password}</p>}
      </div>
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'กำลังสมัคร...' : 'สมัครสมาชิก'}
      </button>
    </form>
  );
}
```

---

## Step 385: ตัวอย่าง Shopping Cart Reducer

```jsx
const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find(i => i.id === action.payload.id);
      if (existing) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    }
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter(i => i.id !== action.payload) };
    case 'UPDATE_QTY':
      if (action.payload.qty <= 0) {
        return { ...state, items: state.items.filter(i => i.id !== action.payload.id) };
      }
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.qty }
            : item
        )
      };
    case 'CLEAR':
      return { ...state, items: [] };
    default:
      return state;
  }
};
```

---

## Step 386: Middleware-like Patterns

```jsx
// Logger middleware
function withLogger(reducer) {
  return function(state, action) {
    console.group(`Action: ${action.type}`);
    console.log('Previous state:', state);
    const nextState = reducer(state, action);
    console.log('Next state:', nextState);
    console.groupEnd();
    return nextState;
  };
}

// LocalStorage persistence middleware
function withPersistence(reducer, storageKey) {
  return function(state, action) {
    const nextState = reducer(state, action);
    localStorage.setItem(storageKey, JSON.stringify(nextState));
    return nextState;
  };
}

// การใช้งาน
const enhancedReducer = withLogger(withPersistence(todoReducer, 'todos'));

function App() {
  const [state, dispatch] = useReducer(enhancedReducer, { todos: [] });
  // ...
}
```

---

## Step 387: Testing Reducers

Reducers เป็น pure functions ทำให้ test ง่ายมาก

```jsx
// __tests__/todoReducer.test.js
import { todoReducer } from '../reducers/todoReducer';

describe('todoReducer', () => {
  const initialState = { todos: [], filter: 'all' };
  
  it('should add a todo', () => {
    const action = { type: 'ADD_TODO', payload: { text: 'Test todo' } };
    const newState = todoReducer(initialState, action);
    
    expect(newState.todos).toHaveLength(1);
    expect(newState.todos[0].text).toBe('Test todo');
    expect(newState.todos[0].completed).toBe(false);
  });
  
  it('should toggle a todo', () => {
    const stateWithTodo = {
      todos: [{ id: 1, text: 'Test', completed: false }],
      filter: 'all'
    };
    
    const action = { type: 'TOGGLE_TODO', payload: 1 };
    const newState = todoReducer(stateWithTodo, action);
    
    expect(newState.todos[0].completed).toBe(true);
  });
  
  it('should delete a todo', () => {
    const stateWithTodo = {
      todos: [{ id: 1, text: 'Test', completed: false }],
      filter: 'all'
    };
    
    const action = { type: 'DELETE_TODO', payload: 1 };
    const newState = todoReducer(stateWithTodo, action);
    
    expect(newState.todos).toHaveLength(0);
  });
  
  it('should not mutate state', () => {
    const action = { type: 'ADD_TODO', payload: { text: 'New' } };
    const newState = todoReducer(initialState, action);
    
    expect(newState).not.toBe(initialState);
    expect(newState.todos).not.toBe(initialState.todos);
  });
});
```

---

## สรุป useReducer

```jsx
// Template
const ACTIONS = { /* constants */ };

function myReducer(state, action) {
  switch (action.type) {
    case ACTIONS.SOMETHING:
      return { ...state, /* changes */ };
    default:
      return state; // or throw error
  }
}

function MyComponent() {
  const [state, dispatch] = useReducer(myReducer, initialState);
  
  const doSomething = () => dispatch({ type: ACTIONS.SOMETHING, payload: data });
  
  return <div>...</div>;
}
```

---

## Quiz และแบบฝึกหัด

### Quiz

1. Reducer function ต้องเป็น pure function คืออะไร?
2. เมื่อไหร่ควรใช้ useReducer แทน useState?
3. ทำไม state ใน reducer ต้อง immutable?
4. Immer ช่วยอะไรกับ useReducer?

### แบบฝึกหัด

**Exercise 1**: สร้าง Bank Account Reducer

```jsx
// TODO: สร้าง reducer สำหรับ bank account ที่มี:
// - balance: number
// - transactions: Transaction[]
// - Actions: DEPOSIT, WITHDRAW, TRANSFER
// - Validation: ไม่สามารถถอนเกิน balance

const bankReducer = (state, action) => {
  // implement here...
};
```

**Exercise 2**: สร้าง Board Game State

```jsx
// TODO: สร้าง reducer สำหรับ Tic-Tac-Toe ที่มี:
// - board: (string|null)[] ขนาด 9
// - currentPlayer: 'X' | 'O'
// - winner: string|null
// - gameStatus: 'playing' | 'won' | 'draw'
// - Actions: MAKE_MOVE, RESET_GAME

const ticTacToeReducer = (state, action) => {
  // implement here...
};
```

---

➡️ ไปต่อ: [Part 18: useMemo และ useCallback](./part-18-useMemo-useCallback.md)
