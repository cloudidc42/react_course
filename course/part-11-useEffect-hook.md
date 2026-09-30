# Part 11: useEffect Hook

## Step 211-240 | ระดับ: กลาง

---

## สารบัญ

- [Step 211: useEffect คืออะไร?](#step-211-useeffect-คืออะไร)
- [Step 212: Syntax พื้นฐานของ useEffect](#step-212-syntax-พื้นฐานของ-useeffect)
- [Step 213: Dependency Array คืออะไร?](#step-213-dependency-array-คืออะไร)
- [Step 214: useEffect ที่ไม่มี Dependency Array](#step-214-useeffect-ที่ไม่มี-dependency-array)
- [Step 215: useEffect ที่มี Empty Dependency Array](#step-215-useeffect-ที่มี-empty-dependency-array)
- [Step 216: useEffect ที่มี Dependencies](#step-216-useeffect-ที่มี-dependencies)
- [Step 217: Side Effects ประเภทต่างๆ](#step-217-side-effects-ประเภทต่างๆ)
- [Step 218: Cleanup Functions คืออะไร?](#step-218-cleanup-functions-คืออะไร)
- [Step 219: useEffect กับ API Calls](#step-219-useeffect-กับ-api-calls)
- [Step 220: useEffect กับ AbortController](#step-220-useeffect-กับ-abortcontroller)
- [Step 221: useEffect กับ Event Listeners](#step-221-useeffect-กับ-event-listeners)
- [Step 222: useEffect กับ Timers](#step-222-useeffect-กับ-timers)
- [Step 223: useEffect กับ Subscriptions](#step-223-useeffect-กับ-subscriptions)
- [Step 224: Infinite Loop กับดัก](#step-224-infinite-loop-กับดัก)
- [Step 225: การใช้ useEffect หลายตัว](#step-225-การใช้-useeffect-หลายตัว)
- [Step 226: useEffect กับ async/await](#step-226-useeffect-กับ-asyncawait)
- [Step 227: useEffect กับ localStorage](#step-227-useeffect-กับ-localstorage)
- [Step 228: useEffect กับ Document Title](#step-228-useeffect-กับ-document-title)
- [Step 229: useEffect กับ WebSocket](#step-229-useeffect-กับ-websocket)
- [Step 230: Best Practices สำหรับ useEffect](#step-230-best-practices-สำหรับ-useeffect)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 211: useEffect คืออะไร?

`useEffect` เป็น Hook ใน React ที่ใช้สำหรับจัดการ **Side Effects** ในฟังก์ชัน Component

### Side Effect คืออะไร?

Side Effect คือการกระทำที่เกิดขึ้นนอกเหนือจากการ render ของ Component เช่น:

- การเรียก API (fetch data)
- การอัปเดต DOM โดยตรง
- การ subscribe/unsubscribe events
- การตั้ง Timer (setTimeout, setInterval)
- การ log ข้อมูล
- การเขียน localStorage

### ทำไมต้องใช้ useEffect?

ใน React การ render ควรเป็น **pure function** ที่ไม่มี side effects แต่ในความเป็นจริง แอปพลิเคชันต้องการ side effects เช่น การดึงข้อมูลจาก server

```jsx
// ❌ ไม่ควรทำแบบนี้ - side effect ใน render
function UserProfile({ userId }) {
  // การเรียก API ใน render จะทำให้เกิดปัญหา
  fetch(`/api/users/${userId}`).then(/* ... */);
  
  return <div>...</div>;
}

// ✅ ควรใช้ useEffect แทน
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);
  
  return <div>{user?.name}</div>;
}
```

---

## Step 212: Syntax พื้นฐานของ useEffect

```jsx
import { useEffect } from 'react';

useEffect(effectFunction, dependencyArray);
```

- **effectFunction**: ฟังก์ชันที่จะรันเป็น side effect
- **dependencyArray**: array ของ dependencies (optional)

### ตัวอย่าง Syntax ต่างๆ

```jsx
import { useState, useEffect } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // 1. ไม่มี dependency array - รันทุกครั้งที่ render
  useEffect(() => {
    console.log('Component rendered');
  });

  // 2. Empty dependency array - รันครั้งเดียวหลัง mount
  useEffect(() => {
    console.log('Component mounted');
  }, []);

  // 3. มี dependencies - รันเมื่อ count เปลี่ยน
  useEffect(() => {
    console.log('Count changed:', count);
  }, [count]);

  // 4. มี cleanup function
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('tick');
    }, 1000);
    
    // Cleanup function
    return () => {
      clearInterval(timer);
    };
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

---

## Step 213: Dependency Array คืออะไร?

Dependency Array บอก React ว่า effect ควรรันเมื่อใด

```jsx
useEffect(() => {
  // effect code
}, [dep1, dep2, dep3]);
```

### กฎของ Dependency Array

```jsx
function ExampleComponent({ userId, filter }) {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    // ✅ ทุก value ที่ใช้ใน effect ต้องอยู่ใน dependency array
    setLoading(true);
    fetch(`/api/data?userId=${userId}&filter=${filter}`)
      .then(res => res.json())
      .then(result => {
        setData(result);
        setLoading(false);
      });
  }, [userId, filter]); // ✅ userId และ filter อยู่ใน deps

  return (
    <div>
      {loading ? 'Loading...' : data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}
```

> **⚠️ คำเตือน**: ถ้าลืมใส่ dependency ใน array อาจทำให้ effect ใช้ค่าเก่า (stale closure)

---

## Step 214: useEffect ที่ไม่มี Dependency Array

เมื่อไม่มี dependency array, effect จะรันหลังทุกครั้งที่ component re-renders

```jsx
function TrackingComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('React');

  // รันทุกครั้งที่ component render
  useEffect(() => {
    console.log('Component re-rendered');
    document.title = `Count: ${count}, Name: ${name}`;
  }); // ไม่มี dependency array

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <input value={name} onChange={e => setName(e.target.value)} />
    </div>
  );
}
```

> **💡 Tips**: การไม่มี dependency array มักทำให้ effect รันบ่อยเกินจำเป็น ควรระวัง performance

---

## Step 215: useEffect ที่มี Empty Dependency Array

Empty array `[]` หมายความว่า effect รันครั้งเดียวหลัง component mount

```jsx
function DataFetcher() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  // รันครั้งเดียวเมื่อ component mount
  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/posts/1')
      .then(res => {
        if (!res.ok) throw new Error('Network error');
        return res.json();
      })
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []); // [] = รันครั้งเดียว

  if (loading) return <p>กำลังโหลด...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <article>
      <h2>{data.title}</h2>
      <p>{data.body}</p>
    </article>
  );
}
```

---

## Step 216: useEffect ที่มี Dependencies

Effect จะรันใหม่เมื่อค่าใด dependency เปลี่ยนแปลง

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);

  // รันเมื่อ userId เปลี่ยน
  useEffect(() => {
    setUser(null); // reset user ก่อน
    
    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]); // รันใหม่เมื่อ userId เปลี่ยน

  // รันเมื่อ userId เปลี่ยน (effect แยก)
  useEffect(() => {
    setPosts([]); // reset posts ก่อน
    
    fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`)
      .then(res => res.json())
      .then(data => setPosts(data));
  }, [userId]);

  if (!user) return <p>Loading user...</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <h3>Posts ({posts.length})</h3>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

// การใช้งาน
function App() {
  const [selectedUserId, setSelectedUserId] = useState(1);
  
  return (
    <div>
      <div>
        {[1, 2, 3, 4, 5].map(id => (
          <button 
            key={id}
            onClick={() => setSelectedUserId(id)}
            style={{ 
              fontWeight: selectedUserId === id ? 'bold' : 'normal' 
            }}
          >
            User {id}
          </button>
        ))}
      </div>
      <UserProfile userId={selectedUserId} />
    </div>
  );
}
```

---

## Step 217: Side Effects ประเภทต่างๆ

### 1. Data Fetching

```jsx
function PostList() {
  const [posts, setPosts] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/posts?_page=${page}&_limit=10`)
      .then(res => res.json())
      .then(data => setPosts(data));
  }, [page]);

  return (
    <div>
      {posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
        </div>
      ))}
      <button onClick={() => setPage(p => p - 1)} disabled={page === 1}>
        Previous
      </button>
      <span> Page {page} </span>
      <button onClick={() => setPage(p => p + 1)}>Next</button>
    </div>
  );
}
```

### 2. DOM Manipulation

```jsx
function AnimatedBox() {
  const [isVisible, setIsVisible] = useState(false);
  
  useEffect(() => {
    // ใช้ imperative DOM manipulation
    const box = document.getElementById('animated-box');
    if (box) {
      if (isVisible) {
        box.style.opacity = '1';
        box.style.transform = 'translateY(0)';
      } else {
        box.style.opacity = '0';
        box.style.transform = 'translateY(-20px)';
      }
    }
  }, [isVisible]);

  return (
    <div>
      <button onClick={() => setIsVisible(v => !v)}>
        Toggle Box
      </button>
      <div
        id="animated-box"
        style={{
          transition: 'all 0.3s ease',
          background: 'blue',
          padding: '20px',
          color: 'white',
          marginTop: '10px'
        }}
      >
        Animated Box
      </div>
    </div>
  );
}
```

### 3. Browser APIs

```jsx
function GeolocationComponent() {
  const [position, setPosition] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!navigator.geolocation) {
      setError('Geolocation is not supported');
      return;
    }

    const watchId = navigator.geolocation.watchPosition(
      (pos) => {
        setPosition({
          lat: pos.coords.latitude,
          lng: pos.coords.longitude
        });
      },
      (err) => {
        setError(err.message);
      }
    );

    // Cleanup: หยุด watch เมื่อ unmount
    return () => {
      navigator.geolocation.clearWatch(watchId);
    };
  }, []);

  if (error) return <p>Error: {error}</p>;
  if (!position) return <p>Getting location...</p>;

  return (
    <p>
      Lat: {position.lat.toFixed(4)}, 
      Lng: {position.lng.toFixed(4)}
    </p>
  );
}
```

---

## Step 218: Cleanup Functions คืออะไร?

Cleanup function คือฟังก์ชันที่ return ออกมาจาก effect เพื่อ "ทำความสะอาด" เมื่อ:
1. Component unmount
2. Effect รันใหม่ (ก่อนรัน effect ครั้งต่อไป)

```jsx
function CleanupExample() {
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    console.log('Effect started');
    const interval = setInterval(() => {
      console.log('Interval tick');
    }, 1000);

    // Cleanup function
    return () => {
      console.log('Cleanup ran');
      clearInterval(interval);
    };
  }, [isRunning]); // รันใหม่เมื่อ isRunning เปลี่ยน

  return (
    <div>
      <p>Status: {isRunning ? 'Running' : 'Stopped'}</p>
      <button onClick={() => setIsRunning(r => !r)}>
        {isRunning ? 'Stop' : 'Start'}
      </button>
    </div>
  );
}
```

### ลำดับการรัน Cleanup

```
Component Mount:
  → Effect runs

State changes:
  → Cleanup from previous effect runs
  → Effect runs again

Component Unmount:
  → Cleanup from last effect runs
```

---

## Step 219: useEffect กับ API Calls

### Pattern พื้นฐาน

```jsx
function ArticleDetail({ articleId }) {
  const [article, setArticle] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Reset state เมื่อ articleId เปลี่ยน
    setLoading(true);
    setError(null);
    setArticle(null);

    fetch(`https://api.example.com/articles/${articleId}`)
      .then(res => {
        if (!res.ok) {
          throw new Error(`HTTP error! status: ${res.status}`);
        }
        return res.json();
      })
      .then(data => {
        setArticle(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [articleId]);

  if (loading) return (
    <div className="loading-spinner">
      <span>กำลังโหลด...</span>
    </div>
  );
  
  if (error) return (
    <div className="error-message">
      <p>เกิดข้อผิดพลาด: {error}</p>
      <button onClick={() => window.location.reload()}>
        ลองใหม่
      </button>
    </div>
  );
  
  if (!article) return null;

  return (
    <article>
      <h1>{article.title}</h1>
      <p>{article.content}</p>
    </article>
  );
}
```

### Pattern ที่ดีกว่า: Custom Hook

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    setError(null);

    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// การใช้งาน
function ArticleList() {
  const { data: articles, loading, error } = useFetch(
    'https://jsonplaceholder.typicode.com/posts'
  );

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {articles?.map(article => (
        <li key={article.id}>{article.title}</li>
      ))}
    </ul>
  );
}
```

---

## Step 220: useEffect กับ AbortController

การยกเลิก fetch request เมื่อ component unmount หรือ dependency เปลี่ยน

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }

    // สร้าง AbortController
    const abortController = new AbortController();
    const { signal } = abortController;

    setLoading(true);

    fetch(`https://api.example.com/search?q=${query}`, { signal })
      .then(res => res.json())
      .then(data => {
        setResults(data);
        setLoading(false);
      })
      .catch(err => {
        // ไม่ต้อง handle error ถ้าเป็น abort
        if (err.name !== 'AbortError') {
          console.error('Search error:', err);
          setLoading(false);
        }
      });

    // Cleanup: abort request เมื่อ query เปลี่ยนหรือ unmount
    return () => {
      abortController.abort();
    };
  }, [query]);

  return (
    <div>
      {loading && <p>Searching...</p>}
      {results.map(item => (
        <div key={item.id}>{item.title}</div>
      ))}
    </div>
  );
}
```

### ตัวอย่าง Search ที่สมบูรณ์

```jsx
import { useState, useEffect } from 'react';

function LiveSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedTerm, setDebouncedTerm] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  // Debounce search term
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 500);

    return () => clearTimeout(timer);
  }, [searchTerm]);

  // Fetch results เมื่อ debouncedTerm เปลี่ยน
  useEffect(() => {
    if (!debouncedTerm.trim()) {
      setResults([]);
      return;
    }

    const controller = new AbortController();
    setLoading(true);

    fetch(
      `https://jsonplaceholder.typicode.com/posts?q=${debouncedTerm}`,
      { signal: controller.signal }
    )
      .then(res => res.json())
      .then(data => {
        setResults(data.slice(0, 5));
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setLoading(false);
        }
      });

    return () => controller.abort();
  }, [debouncedTerm]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="ค้นหา..."
      />
      {loading && <p>กำลังค้นหา...</p>}
      <ul>
        {results.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Step 221: useEffect กับ Event Listeners

```jsx
function KeyboardHandler() {
  const [lastKey, setLastKey] = useState('');
  const [keys, setKeys] = useState([]);

  useEffect(() => {
    // สร้าง event handler
    function handleKeyDown(event) {
      setLastKey(event.key);
      setKeys(prev => [...prev.slice(-9), event.key]); // เก็บ 10 keys ล่าสุด
    }

    // Add event listener
    window.addEventListener('keydown', handleKeyDown);

    // Cleanup: Remove event listener เมื่อ unmount
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, []); // [] = add listener ครั้งเดียว

  return (
    <div>
      <p>กด key ใดก็ได้...</p>
      <p>Last key: <strong>{lastKey}</strong></p>
      <div>
        Keys: {keys.map((k, i) => (
          <span key={i} style={{ margin: '0 4px', padding: '2px 6px', background: '#eee' }}>
            {k}
          </span>
        ))}
      </div>
    </div>
  );
}
```

### Scroll Detection

```jsx
function ScrollProgress() {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    function handleScroll() {
      const scrollTop = window.scrollY;
      const docHeight = document.documentElement.scrollHeight - window.innerHeight;
      const scrollPercent = docHeight > 0 ? (scrollTop / docHeight) * 100 : 0;
      setProgress(Math.round(scrollPercent));
    }

    window.addEventListener('scroll', handleScroll, { passive: true });
    
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);

  return (
    <div style={{
      position: 'fixed',
      top: 0,
      left: 0,
      width: `${progress}%`,
      height: '4px',
      background: 'blue',
      transition: 'width 0.1s',
      zIndex: 9999
    }} />
  );
}
```

### Resize Observer

```jsx
function ResizeObserverComponent() {
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  useEffect(() => {
    function handleResize() {
      setDimensions({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }

    // รัน 1 ครั้งตอน mount เพื่อตั้งค่าเริ่มต้น
    handleResize();

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return (
    <div>
      <p>Window: {dimensions.width} x {dimensions.height}</p>
    </div>
  );
}
```

---

## Step 222: useEffect กับ Timers

### setTimeout

```jsx
function AutoDismissAlert({ message, duration = 3000 }) {
  const [isVisible, setIsVisible] = useState(true);

  useEffect(() => {
    const timer = setTimeout(() => {
      setIsVisible(false);
    }, duration);

    // Cleanup: ยกเลิก timer เมื่อ unmount หรือ props เปลี่ยน
    return () => clearTimeout(timer);
  }, [duration]); // รันใหม่ถ้า duration เปลี่ยน

  if (!isVisible) return null;

  return (
    <div style={{
      background: '#4CAF50',
      color: 'white',
      padding: '12px 20px',
      borderRadius: '4px',
      marginBottom: '10px'
    }}>
      {message}
    </div>
  );
}
```

### setInterval - Countdown Timer

```jsx
function CountdownTimer({ initialSeconds }) {
  const [seconds, setSeconds] = useState(initialSeconds);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning || seconds <= 0) return;

    const interval = setInterval(() => {
      setSeconds(s => {
        if (s <= 1) {
          setIsRunning(false);
          return 0;
        }
        return s - 1;
      });
    }, 1000);

    return () => clearInterval(interval);
  }, [isRunning, seconds]);

  const formatTime = (secs) => {
    const mins = Math.floor(secs / 60);
    const remaining = secs % 60;
    return `${mins.toString().padStart(2, '0')}:${remaining.toString().padStart(2, '0')}`;
  };

  return (
    <div>
      <div style={{ fontSize: '3rem', fontFamily: 'monospace' }}>
        {formatTime(seconds)}
      </div>
      <button 
        onClick={() => setIsRunning(r => !r)}
        disabled={seconds === 0}
      >
        {isRunning ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => {
        setIsRunning(false);
        setSeconds(initialSeconds);
      }}>
        Reset
      </button>
    </div>
  );
}
```

### Real-time Clock

```jsx
function DigitalClock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const interval = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => clearInterval(interval);
  }, []); // [] = สร้าง interval ครั้งเดียว

  const formatTime = (date) => {
    return date.toLocaleTimeString('th-TH', {
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
      hour12: false
    });
  };

  return (
    <div style={{ fontSize: '2rem', fontFamily: 'monospace' }}>
      🕐 {formatTime(time)}
    </div>
  );
}
```

---

## Step 223: useEffect กับ Subscriptions

```jsx
// จำลอง WebSocket subscription
function useWebSocketSimulation(channelId) {
  const [messages, setMessages] = useState([]);
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    if (!channelId) return;

    console.log(`Subscribing to channel: ${channelId}`);
    setConnected(true);

    // จำลองการรับข้อความ
    const interval = setInterval(() => {
      setMessages(prev => [
        ...prev,
        {
          id: Date.now(),
          channel: channelId,
          text: `Message at ${new Date().toLocaleTimeString()}`
        }
      ].slice(-10)); // เก็บแค่ 10 messages ล่าสุด
    }, 2000);

    // Cleanup: unsubscribe
    return () => {
      console.log(`Unsubscribing from channel: ${channelId}`);
      setConnected(false);
      clearInterval(interval);
    };
  }, [channelId]);

  return { messages, connected };
}

function ChatRoom({ channelId }) {
  const { messages, connected } = useWebSocketSimulation(channelId);

  return (
    <div>
      <div>
        Status: {connected ? '🟢 Connected' : '🔴 Disconnected'}
        {channelId && ` to #${channelId}`}
      </div>
      <div style={{ height: '200px', overflowY: 'auto', border: '1px solid #ccc', padding: '10px' }}>
        {messages.map(msg => (
          <div key={msg.id} style={{ marginBottom: '5px' }}>
            {msg.text}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Step 224: Infinite Loop กับดัก

### กรณีที่ 1: Object/Array ใน Dependencies

```jsx
// ❌ เกิด Infinite Loop!
function BadExample() {
  const [data, setData] = useState([]);

  const options = { method: 'GET' }; // Object ถูกสร้างใหม่ทุก render!

  useEffect(() => {
    fetch('/api/data', options)
      .then(res => res.json())
      .then(setData);
  }, [options]); // ❌ options เปลี่ยนทุก render → infinite loop

  return <div>{data.length} items</div>;
}

// ✅ แก้ไขโดย: ย้าย options ออกไปนอก component
const options = { method: 'GET' }; // ✅ สร้างแค่ครั้งเดียว

function GoodExample() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch('/api/data', options)
      .then(res => res.json())
      .then(setData);
  }, []); // ✅ ไม่ต้องใส่ options ใน deps เพราะมันไม่เปลี่ยน

  return <div>{data.length} items</div>;
}
```

### กรณีที่ 2: setState ใน Effect โดยไม่มี Condition

```jsx
// ❌ เกิด Infinite Loop!
function BadCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(count + 1); // ❌ ทุกครั้งที่ count เปลี่ยน, effect รัน, count เปลี่ยนอีก...
  }, [count]);

  return <div>{count}</div>;
}

// ✅ แก้ไขโดย: ใช้ functional update
function GoodCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // นี่ก็ยังเป็น infinite loop เพราะ count เปลี่ยนทุกครั้ง
    // ถ้าต้องการนับ ควรใช้ event แทน
  }, [count]);

  // ✅ ใช้ event handler แทน
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

### กรณีที่ 3: Function ใน Dependencies

```jsx
// ❌ เกิด Infinite Loop!
function BadFunctionDep() {
  const [data, setData] = useState(null);

  // Function ถูกสร้างใหม่ทุก render
  const fetchData = () => {
    return fetch('/api/data').then(res => res.json());
  };

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ❌ fetchData สร้างใหม่ทุก render

  return <div>{JSON.stringify(data)}</div>;
}

// ✅ แก้ไขโดย: ใช้ useCallback หรือย้ายฟังก์ชันเข้าไปใน effect
function GoodFunctionDep() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // ✅ ย้ายฟังก์ชันเข้ามาใน effect
    const fetchData = () => {
      return fetch('/api/data').then(res => res.json());
    };

    fetchData().then(setData);
  }, []); // ✅ ไม่ต้องใส่ fetchData ใน deps

  return <div>{JSON.stringify(data)}</div>;
}
```

---

## Step 225: การใช้ useEffect หลายตัว

แต่ละ `useEffect` ควรจัดการ concern เดียว (Single Responsibility)

```jsx
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [notifications, setNotifications] = useState([]);
  const [theme, setTheme] = useState('light');

  // Effect 1: โหลดข้อมูล user
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);

  // Effect 2: โหลด notifications
  useEffect(() => {
    fetch(`/api/users/${userId}/notifications`)
      .then(res => res.json())
      .then(setNotifications);
  }, [userId]);

  // Effect 3: อัปเดต document title
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Dashboard`;
    }
    return () => {
      document.title = 'My App';
    };
  }, [user]);

  // Effect 4: Theme management
  useEffect(() => {
    document.body.classList.remove('light', 'dark');
    document.body.classList.add(theme);
  }, [theme]);

  // Effect 5: Cleanup เมื่อ unmount
  useEffect(() => {
    return () => {
      document.title = 'My App';
    };
  }, []);

  return (
    <div className={theme}>
      <h1>Welcome, {user?.name}</h1>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <p>{notifications.length} notifications</p>
    </div>
  );
}
```

---

## Step 226: useEffect กับ async/await

`useEffect` callback ไม่รองรับ `async` โดยตรง

```jsx
// ❌ ผิด - useEffect ไม่ควรเป็น async function
function BadAsync() {
  const [data, setData] = useState(null);

  useEffect(async () => { // ❌ async effect ทำให้ cleanup ทำงานไม่ถูกต้อง
    const res = await fetch('/api/data');
    const data = await res.json();
    setData(data);
  }, []);

  return <div>{JSON.stringify(data)}</div>;
}

// ✅ วิธีที่ 1: ประกาศ async function ภายใน
function GoodAsync1() {
  const [data, setData] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const res = await fetch('/api/data');
        const json = await res.json();
        setData(json);
      } catch (error) {
        console.error('Error:', error);
      }
    }

    fetchData();
  }, []);

  return <div>{JSON.stringify(data)}</div>;
}

// ✅ วิธีที่ 2: IIFE (Immediately Invoked Function Expression)
function GoodAsync2() {
  const [data, setData] = useState(null);

  useEffect(() => {
    (async () => {
      try {
        const res = await fetch('/api/data');
        const json = await res.json();
        setData(json);
      } catch (error) {
        console.error('Error:', error);
      }
    })();
  }, []);

  return <div>{JSON.stringify(data)}</div>;
}
```

### async/await กับ AbortController

```jsx
function AsyncWithAbort({ productId }) {
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchProduct() {
      setLoading(true);
      setError(null);
      
      try {
        const res = await fetch(
          `https://fakestoreapi.com/products/${productId}`,
          { signal: controller.signal }
        );
        
        if (!res.ok) throw new Error('Product not found');
        
        const data = await res.json();
        setProduct(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        if (!controller.signal.aborted) {
          setLoading(false);
        }
      }
    }

    fetchProduct();
    
    return () => controller.abort();
  }, [productId]);

  if (loading) return <p>Loading product...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!product) return null;

  return (
    <div>
      <h2>{product.title}</h2>
      <p>฿{product.price}</p>
      <img src={product.image} alt={product.title} style={{ width: '200px' }} />
    </div>
  );
}
```

---

## Step 227: useEffect กับ localStorage

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Failed to save to localStorage:', error);
    }
  }, [key, value]);

  return [value, setValue];
}

// การใช้งาน
function ThemeSettings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'th');

  return (
    <div>
      <h2>Settings</h2>
      <div>
        <label>Theme: </label>
        <select value={theme} onChange={e => setTheme(e.target.value)}>
          <option value="light">Light</option>
          <option value="dark">Dark</option>
          <option value="auto">Auto</option>
        </select>
      </div>
      <div>
        <label>Language: </label>
        <select value={language} onChange={e => setLanguage(e.target.value)}>
          <option value="th">ไทย</option>
          <option value="en">English</option>
        </select>
      </div>
      <p>Settings saved automatically!</p>
    </div>
  );
}
```

---

## Step 228: useEffect กับ Document Title

```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = title;
    
    // Restore title เมื่อ unmount
    return () => {
      document.title = previousTitle;
    };
  }, [title]);
}

// การใช้งาน
function ProductPage({ product }) {
  useDocumentTitle(product ? `${product.name} - MyShop` : 'Loading... - MyShop');
  
  if (!product) return <p>Loading...</p>;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}

function App() {
  const [currentPage, setCurrentPage] = useState('home');
  
  const pages = {
    home: { title: 'Home - MyApp', content: 'Welcome to MyApp!' },
    about: { title: 'About - MyApp', content: 'About us page' },
    contact: { title: 'Contact - MyApp', content: 'Contact us' }
  };
  
  const page = pages[currentPage];
  useDocumentTitle(page.title);
  
  return (
    <div>
      <nav>
        {Object.keys(pages).map(key => (
          <button key={key} onClick={() => setCurrentPage(key)}>
            {key}
          </button>
        ))}
      </nav>
      <main>{page.content}</main>
    </div>
  );
}
```

---

## Step 229: useEffect กับ WebSocket

```jsx
function useWebSocket(url) {
  const [socket, setSocket] = useState(null);
  const [messages, setMessages] = useState([]);
  const [status, setStatus] = useState('disconnected');

  useEffect(() => {
    if (!url) return;

    const ws = new WebSocket(url);

    ws.onopen = () => {
      setStatus('connected');
      setSocket(ws);
    };

    ws.onmessage = (event) => {
      setMessages(prev => [...prev, {
        id: Date.now(),
        data: event.data,
        timestamp: new Date()
      }]);
    };

    ws.onerror = (error) => {
      setStatus('error');
      console.error('WebSocket error:', error);
    };

    ws.onclose = () => {
      setStatus('disconnected');
      setSocket(null);
    };

    // Cleanup: ปิด connection เมื่อ unmount
    return () => {
      ws.close();
    };
  }, [url]);

  const sendMessage = (message) => {
    if (socket?.readyState === WebSocket.OPEN) {
      socket.send(message);
    }
  };

  return { messages, status, sendMessage };
}

// การใช้งาน (ใช้ echo server สำหรับทดสอบ)
function WebSocketChat() {
  const { messages, status, sendMessage } = useWebSocket('wss://echo.websocket.org');
  const [inputText, setInputText] = useState('');

  const handleSend = () => {
    if (inputText.trim()) {
      sendMessage(inputText);
      setInputText('');
    }
  };

  return (
    <div>
      <p>Status: {status}</p>
      <div style={{ height: '200px', overflowY: 'auto', border: '1px solid #ccc' }}>
        {messages.map(msg => (
          <div key={msg.id}>
            <small>{msg.timestamp.toLocaleTimeString()}</small>: {msg.data}
          </div>
        ))}
      </div>
      <div>
        <input
          value={inputText}
          onChange={e => setInputText(e.target.value)}
          onKeyPress={e => e.key === 'Enter' && handleSend()}
          placeholder="Type a message..."
        />
        <button onClick={handleSend} disabled={status !== 'connected'}>
          Send
        </button>
      </div>
    </div>
  );
}
```

---

## Step 230: Best Practices สำหรับ useEffect

### 1. แยก Effects ตาม Concern

```jsx
// ❌ Effect เดียวทำหลายอย่าง
useEffect(() => {
  fetchUser();
  fetchPosts();
  setupAnalytics();
  subscribeToNotifications();
}, []);

// ✅ แยก Effects
useEffect(() => { fetchUser(); }, []);
useEffect(() => { fetchPosts(); }, []);
useEffect(() => { setupAnalytics(); }, []);
useEffect(() => { subscribeToNotifications(); }, []);
```

### 2. ใส่ Dependencies ให้ครบ

```jsx
// ❌ Missing dependency
function BadExample({ userId, filter }) {
  const [data, setData] = useState([]);
  
  useEffect(() => {
    fetchData(userId, filter); // ใช้ filter แต่ไม่ใส่ใน deps
  }, [userId]); // ❌ ขาด filter
}

// ✅ ครบทุก dependency
function GoodExample({ userId, filter }) {
  const [data, setData] = useState([]);
  
  useEffect(() => {
    fetchData(userId, filter);
  }, [userId, filter]); // ✅
}
```

### 3. Always Clean Up

```jsx
// ✅ Always clean up subscriptions, timers, event listeners
useEffect(() => {
  const subscription = someAPI.subscribe(handleData);
  return () => subscription.unsubscribe();
}, []);
```

### 4. Avoid Race Conditions

```jsx
function SafeFetch({ id }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    let isMounted = true; // flag เพื่อตรวจว่า component ยัง mount อยู่ไหม

    fetch(`/api/data/${id}`)
      .then(res => res.json())
      .then(result => {
        if (isMounted) { // ✅ ตรวจก่อน setState
          setData(result);
        }
      });

    return () => {
      isMounted = false; // เปลี่ยน flag เมื่อ cleanup
    };
  }, [id]);

  return <div>{JSON.stringify(data)}</div>;
}
```

### 5. Extract Custom Hooks

```jsx
// ✅ แยก logic ออกเป็น custom hook
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => setSize({
      width: window.innerWidth,
      height: window.innerHeight
    });

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// การใช้งานสะอาด
function ResponsiveComponent() {
  const { width } = useWindowSize();
  return <div>Screen is {width < 768 ? 'mobile' : 'desktop'}</div>;
}
```

---

## สรุป useEffect

| Scenario | Code |
|---|---|
| รันครั้งเดียวหลัง mount | `useEffect(() => {...}, [])` |
| รันทุกครั้งที่ render | `useEffect(() => {...})` |
| รันเมื่อ state/prop เปลี่ยน | `useEffect(() => {...}, [dep])` |
| รันพร้อม cleanup | `useEffect(() => { ...; return () => {...}; }, [])` |

---

## Quiz และแบบฝึกหัด

### Quiz

1. `useEffect` กับ `[]` dependency รันเมื่อใด?
2. ทำไม async function ไม่ควรเป็น useEffect callback โดยตรง?
3. Cleanup function ทำงานเมื่อใด?
4. ปัญหาของ Object/Array ใน dependency array คืออะไร?

### แบบฝึกหัด

**Exercise 1**: สร้าง `Counter` ที่นับทุก 1 วินาที มีปุ่ม Start/Stop/Reset

```jsx
// TODO: Implement this
function IntervalCounter() {
  // ใช้ useState สำหรับ count และ isRunning
  // ใช้ useEffect กับ setInterval
  // อย่าลืม cleanup!
}
```

**Exercise 2**: สร้าง `OnlineStatus` ที่แสดงว่า user online หรือ offline

```jsx
// TODO: Implement this
function OnlineStatus() {
  // ใช้ navigator.onLine
  // Add event listeners สำหรับ 'online' และ 'offline' events
  // อย่าลืม cleanup!
}
```

**Exercise 3**: สร้าง Search component ที่ debounce 500ms ก่อนค้นหา

```jsx
// TODO: Implement this
function DebouncedSearch() {
  // ใช้ useEffect 2 ตัว:
  // 1. Debounce search term
  // 2. Fetch results
}
```

---

➡️ ไปต่อ: [Part 12: Component Lifecycle](./part-12-component-lifecycle.md)
