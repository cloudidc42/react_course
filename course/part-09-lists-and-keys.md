# Part 09: Lists และ Keys
## Step 166-185 | ระดับ: พื้นฐาน-กลาง

---

## สารบัญ (Table of Contents)

- [Step 166: Lists ใน React คืออะไร?](#step-166)
- [Step 167: Array.map() ใน JSX](#step-167)
- [Step 168: Key Prop คืออะไร?](#step-168)
- [Step 169: ทำไม Key ต้องไม่ซ้ำ?](#step-169)
- [Step 170: Key ที่ดีและไม่ดี](#step-170)
- [Step 171: Rendering Nested Lists](#step-171)
- [Step 172: Filter ก่อน Render](#step-172)
- [Step 173: Sort ก่อน Render](#step-173)
- [Step 174: Filter และ Sort รวมกัน](#step-174)
- [Step 175: Dynamic List CRUD - Add](#step-175)
- [Step 176: Dynamic List CRUD - Delete](#step-176)
- [Step 177: Dynamic List CRUD - Update](#step-177)
- [Step 178: Paginated List](#step-178)
- [Step 179: Infinite Scroll Pattern](#step-179)
- [Step 180: Virtualized List](#step-180)
- [Step 181: Grouped Lists](#step-181)
- [Step 182: Drag and Drop List](#step-182)
- [Step 183: Search and Filter List](#step-183)
- [Step 184: List Performance Optimization](#step-184)
- [Step 185: Best Practices](#step-185)
- [Quiz และแบบฝึกหัด](#quiz)

---

## Step 166: Lists ใน React คืออะไร? {#step-166}

Lists ใน React คือการ render ชุดของ elements จาก array ข้อมูล

### ปัญหาพื้นฐาน

```jsx
// ❌ ไม่ dynamic - ต้องเขียนซ้ำ
function StaticList() {
  return (
    <ul>
      <li>รายการที่ 1</li>
      <li>รายการที่ 2</li>
      <li>รายการที่ 3</li>
      {/* ถ้าข้อมูลเปลี่ยน ต้องมาแก้ JSX */}
    </ul>
  );
}

// ✅ Dynamic - ใช้ array.map()
function DynamicList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### List Rendering ทำงานอย่างไร?

```jsx
// React render array ของ elements ได้โดยตรง
const elements = [
  <li key="1">Apple</li>,
  <li key="2">Banana</li>,
  <li key="3">Cherry</li>
];

// React render ได้เลย
function FruitList() {
  return <ul>{elements}</ul>;
}

// Map สร้าง array ของ elements
const fruits = ['Apple', 'Banana', 'Cherry'];
const fruitElements = fruits.map((fruit, index) => (
  <li key={index}>{fruit}</li>
));
```

---

## Step 167: Array.map() ใน JSX {#step-167}

`map()` เป็น method หลักที่ใช้ render lists ใน React

### Syntax พื้นฐาน

```jsx
function ProductList({ products }) {
  return (
    <div className="product-grid">
      {products.map(product => (
        <div key={product.id} className="product-card">
          <img src={product.image} alt={product.name} />
          <h3>{product.name}</h3>
          <p>฿{product.price.toLocaleString()}</p>
          <button>เพิ่มในตะกร้า</button>
        </div>
      ))}
    </div>
  );
}
```

### Map กับ Component Extraction

```jsx
// ✅ แยก Component เพื่อ clean code
function ProductCard({ product }) {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>฿{product.price.toLocaleString()}</p>
      <p>⭐ {product.rating}</p>
      <button>เพิ่มในตะกร้า</button>
    </div>
  );
}

function ProductList({ products }) {
  return (
    <div className="product-grid">
      {products.map(product => (
        // Key อยู่ที่ component ที่ rendered ใน map
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Map กับ Conditional Rendering

```jsx
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          <span>{user.name}</span>
          {user.isActive && <span className="active-badge">✅</span>}
          {user.isAdmin && <span className="admin-badge">👑</span>}
        </li>
      ))}
    </ul>
  );
}
```

### หลาย Maps ใน Component เดียว

```jsx
function Dashboard({ categories, featuredItems, recentOrders }) {
  return (
    <div>
      {/* Map ที่ 1: Categories */}
      <section>
        <h2>หมวดหมู่</h2>
        <div className="categories">
          {categories.map(cat => (
            <a key={cat.id} href={`/category/${cat.slug}`}>
              {cat.icon} {cat.name}
            </a>
          ))}
        </div>
      </section>
      
      {/* Map ที่ 2: Featured Items */}
      <section>
        <h2>สินค้าแนะนำ</h2>
        <div className="featured-grid">
          {featuredItems.map(item => (
            <FeaturedCard key={item.id} item={item} />
          ))}
        </div>
      </section>
      
      {/* Map ที่ 3: Recent Orders */}
      <section>
        <h2>คำสั่งซื้อล่าสุด</h2>
        <table>
          <tbody>
            {recentOrders.map(order => (
              <tr key={order.id}>
                <td>{order.id}</td>
                <td>{order.date}</td>
                <td>{order.total}</td>
                <td>{order.status}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </section>
    </div>
  );
}
```

---

## Step 168: Key Prop คืออะไร? {#step-168}

`key` เป็น special prop ที่ React ใช้ระบุ element แต่ละตัวใน list

### ทำไมต้องมี Key?

React ใช้ key เพื่อ:
1. **ระบุ element** ที่เปลี่ยนแปลง เพิ่ม หรือลบ
2. **ประสิทธิภาพ** - re-render เฉพาะ element ที่เปลี่ยน
3. **State preservation** - คง state ของ component ที่ไม่เปลี่ยน

```jsx
// ❌ ไม่มี Key - Warning ใน Console
function BadList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li>{item.name}</li>  {/* Warning: Each child should have unique key */}
      ))}
    </ul>
  );
}

// ✅ มี Key - ไม่มี Warning
function GoodList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### Key ต้องอยู่ที่ไหน?

Key ต้องอยู่ที่ **outermost element** ใน map callback

```jsx
// ✅ Key อยู่ที่ outermost element
{items.map(item => (
  <div key={item.id}>
    <h3>{item.title}</h3>
    <p>{item.description}</p>
  </div>
))}

// ✅ Key อยู่ที่ Component
{items.map(item => (
  <ItemCard key={item.id} item={item} />
))}

// ✅ Key กับ Fragment
{items.map(item => (
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.definition}</dd>
  </React.Fragment>
))}

// ❌ Key อยู่ผิดที่
{items.map(item => (
  <div>
    <h3 key={item.id}>{item.title}</h3>  {/* ผิด! */}
  </div>
))}
```

---

## Step 169: ทำไม Key ต้องไม่ซ้ำ? {#step-169}

### การทำงานของ Reconciliation Algorithm

```
ก่อน update:        หลัง update:
<ul>                <ul>
  <li key="1">A       <li key="1">A  ← เหมือนเดิม, ไม่ทำอะไร
  <li key="2">B       <li key="3">C  ← key ใหม่, สร้างใหม่
  <li key="3">C       <li key="2">B  ← key เดิม, แค่ move
</ul>               </ul>
```

### ตัวอย่างที่แสดงความสำคัญของ Key

```jsx
function KeyDemo() {
  const [items, setItems] = useState([
    { id: 1, text: "Item 1" },
    { id: 2, text: "Item 2" },
    { id: 3, text: "Item 3" },
  ]);
  
  const prependItem = () => {
    setItems(prev => [
      { id: Date.now(), text: `New Item ${Date.now()}` },
      ...prev
    ]);
  };
  
  return (
    <div>
      <button onClick={prependItem}>เพิ่มด้านบน</button>
      
      <h3>ด้วย index เป็น key (ปัญหา)</h3>
      <ul>
        {items.map((item, index) => (
          <li key={index}>
            {item.text}
            <input placeholder="พิมพ์บางอย่าง..." />
          </li>
          // ถ้าเพิ่ม item ด้านบน input state จะไม่ตรงกับ item!
        ))}
      </ul>
      
      <h3>ด้วย id เป็น key (ถูกต้อง)</h3>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.text}
            <input placeholder="พิมพ์บางอย่าง..." />
          </li>
          // input state อยู่กับ item ที่ถูกต้องเสมอ
        ))}
      </ul>
    </div>
  );
}
```

---

## Step 170: Key ที่ดีและไม่ดี {#step-170}

### Key ที่ดี

```jsx
// ✅ ID จาก database (ดีที่สุด)
{users.map(user => (
  <UserCard key={user.id} user={user} />
))}

// ✅ Unique identifier (UUID, slug)
{posts.map(post => (
  <PostCard key={post.slug} post={post} />
))}

// ✅ Composite key (ถ้า single key ไม่ unique)
{items.map(item => (
  <Item key={`${item.userId}-${item.productId}`} item={item} />
))}
```

### Key ที่ไม่ดี

```jsx
// ❌ Index (ปัญหาเมื่อ list เปลี่ยน)
{items.map((item, index) => (
  <Item key={index} item={item} />
))}

// ❌ Random key (re-create ทุก render!)
{items.map(item => (
  <Item key={Math.random()} item={item} />
))}

// ❌ Timestamp (ถ้าสร้างใน render function)
{items.map(item => (
  <Item key={Date.now()} item={item} />
))}
```

### เมื่อไหร index เป็น key ได้?

```jsx
// ✅ ใช้ index ได้ เมื่อ:
// 1. List ไม่เปลี่ยนแปลง (static list)
// 2. Items ไม่มี unique ID
// 3. List ไม่มีการ reorder, add ตรงกลาง, หรือ delete

const staticMenuItems = ['หน้าหลัก', 'เกี่ยวกับ', 'ติดต่อ'];

{staticMenuItems.map((item, index) => (
  <NavItem key={index} label={item} />
))}
```

---

## Step 171: Rendering Nested Lists {#step-171}

```jsx
// Nested list - หมวดหมู่และรายการย่อย
function CategoryMenu({ categories }) {
  return (
    <nav>
      <ul>
        {categories.map(category => (
          <li key={category.id}>
            <h3>{category.name}</h3>
            {/* Nested map */}
            <ul>
              {category.items.map(item => (
                <li key={item.id}>
                  <a href={item.url}>{item.name}</a>
                </li>
              ))}
            </ul>
          </li>
        ))}
      </ul>
    </nav>
  );
}

const categories = [
  {
    id: 1,
    name: "เสื้อผ้า",
    items: [
      { id: 11, name: "เสื้อยืด", url: "/tshirt" },
      { id: 12, name: "กางเกง", url: "/pants" },
    ]
  },
  {
    id: 2,
    name: "อิเล็กทรอนิกส์",
    items: [
      { id: 21, name: "โทรศัพท์", url: "/phone" },
      { id: 22, name: "แล็ปท็อป", url: "/laptop" },
    ]
  }
];
```

### Recursive List Rendering

```jsx
// Tree structure - recursive rendering
function TreeNode({ node, depth = 0 }) {
  const [isExpanded, setIsExpanded] = useState(true);
  const hasChildren = node.children && node.children.length > 0;
  
  return (
    <div style={{ paddingLeft: depth * 20 }}>
      <div 
        style={{ 
          display: 'flex', 
          alignItems: 'center',
          cursor: hasChildren ? 'pointer' : 'default',
          padding: '4px 8px',
          borderRadius: 4,
          hover: { background: '#f5f5f5' }
        }}
        onClick={() => hasChildren && setIsExpanded(!isExpanded)}
      >
        {hasChildren && (
          <span style={{ marginRight: 4 }}>
            {isExpanded ? '▼' : '▶'}
          </span>
        )}
        {!hasChildren && <span style={{ marginRight: 4, opacity: 0 }}>•</span>}
        <span>{node.icon && `${node.icon} `}{node.name}</span>
      </div>
      
      {/* Recursive rendering */}
      {hasChildren && isExpanded && (
        <div>
          {node.children.map(child => (
            <TreeNode key={child.id} node={child} depth={depth + 1} />
          ))}
        </div>
      )}
    </div>
  );
}

function FileTree({ data }) {
  return (
    <div>
      {data.map(node => (
        <TreeNode key={node.id} node={node} />
      ))}
    </div>
  );
}

const fileStructure = [
  {
    id: 1, name: "src", icon: "📁",
    children: [
      { id: 11, name: "components", icon: "📁",
        children: [
          { id: 111, name: "Button.jsx", icon: "📄", children: [] },
          { id: 112, name: "Input.jsx", icon: "📄", children: [] },
        ]
      },
      { id: 12, name: "App.jsx", icon: "📄", children: [] },
    ]
  },
  {
    id: 2, name: "public", icon: "📁",
    children: [
      { id: 21, name: "index.html", icon: "📄", children: [] },
    ]
  }
];
```

---

## Step 172: Filter ก่อน Render {#step-172}

```jsx
function FilterableProductList({ products }) {
  const [category, setCategory] = useState('all');
  const [minPrice, setMinPrice] = useState(0);
  const [maxPrice, setMaxPrice] = useState(Infinity);
  const [inStockOnly, setInStockOnly] = useState(false);
  
  // Filter logic แยกออกมา
  const filteredProducts = products.filter(product => {
    if (category !== 'all' && product.category !== category) return false;
    if (product.price < minPrice || product.price > maxPrice) return false;
    if (inStockOnly && !product.inStock) return false;
    return true;
  });
  
  const categories = ['all', ...new Set(products.map(p => p.category))];
  
  return (
    <div>
      {/* Filter Controls */}
      <div className="filters">
        <select value={category} onChange={e => setCategory(e.target.value)}>
          {categories.map(cat => (
            <option key={cat} value={cat}>
              {cat === 'all' ? 'ทุกหมวดหมู่' : cat}
            </option>
          ))}
        </select>
        
        <div>
          <label>ราคา: ฿{minPrice} - ฿{maxPrice === Infinity ? '∞' : maxPrice}</label>
          <input 
            type="range" 
            min={0} max={5000} 
            value={minPrice}
            onChange={e => setMinPrice(Number(e.target.value))}
          />
        </div>
        
        <label>
          <input 
            type="checkbox"
            checked={inStockOnly}
            onChange={e => setInStockOnly(e.target.checked)}
          />
          มีสินค้าเท่านั้น
        </label>
      </div>
      
      {/* Results */}
      <p>แสดง {filteredProducts.length} จาก {products.length} สินค้า</p>
      
      {filteredProducts.length === 0 ? (
        <p>ไม่พบสินค้าที่ตรงกับเงื่อนไข</p>
      ) : (
        <div className="product-grid">
          {filteredProducts.map(product => (
            <ProductCard key={product.id} product={product} />
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Step 173: Sort ก่อน Render {#step-173}

```jsx
function SortableTable({ data }) {
  const [sortConfig, setSortConfig] = useState({
    key: null,
    direction: 'asc'
  });
  
  const handleSort = (key) => {
    setSortConfig(prev => ({
      key,
      direction: prev.key === key && prev.direction === 'asc' ? 'desc' : 'asc'
    }));
  };
  
  // Sort logic
  const sortedData = [...data].sort((a, b) => {
    if (!sortConfig.key) return 0;
    
    const aVal = a[sortConfig.key];
    const bVal = b[sortConfig.key];
    
    let comparison = 0;
    
    if (typeof aVal === 'string') {
      comparison = aVal.localeCompare(bVal, 'th');
    } else if (typeof aVal === 'number') {
      comparison = aVal - bVal;
    } else if (aVal instanceof Date) {
      comparison = aVal.getTime() - bVal.getTime();
    }
    
    return sortConfig.direction === 'asc' ? comparison : -comparison;
  });
  
  const columns = [
    { key: 'name', label: 'ชื่อ' },
    { key: 'age', label: 'อายุ' },
    { key: 'city', label: 'เมือง' },
    { key: 'salary', label: 'เงินเดือน' },
  ];
  
  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th 
              key={col.key}
              onClick={() => handleSort(col.key)}
              style={{ cursor: 'pointer' }}
            >
              {col.label}
              {sortConfig.key === col.key && (
                <span>{sortConfig.direction === 'asc' ? ' ↑' : ' ↓'}</span>
              )}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sortedData.map(row => (
          <tr key={row.id}>
            {columns.map(col => (
              <td key={col.key}>{row[col.key]}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

---

## Step 174: Filter และ Sort รวมกัน {#step-174}

```jsx
function AdvancedList({ items }) {
  const [searchQuery, setSearchQuery] = useState('');
  const [filterStatus, setFilterStatus] = useState('all');
  const [sortBy, setSortBy] = useState('name');
  const [sortOrder, setSortOrder] = useState('asc');
  
  // Pipeline: filter → sort → slice (for pagination)
  const processedItems = useMemo(() => {
    let result = [...items];
    
    // 1. Filter by search
    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter(item => 
        item.name.toLowerCase().includes(query) ||
        item.description?.toLowerCase().includes(query)
      );
    }
    
    // 2. Filter by status
    if (filterStatus !== 'all') {
      result = result.filter(item => item.status === filterStatus);
    }
    
    // 3. Sort
    result.sort((a, b) => {
      const aVal = a[sortBy];
      const bVal = b[sortBy];
      const multiplier = sortOrder === 'asc' ? 1 : -1;
      
      if (typeof aVal === 'string') return aVal.localeCompare(bVal, 'th') * multiplier;
      return (aVal - bVal) * multiplier;
    });
    
    return result;
  }, [items, searchQuery, filterStatus, sortBy, sortOrder]);
  
  return (
    <div>
      {/* Controls */}
      <div style={{ display: 'flex', gap: 8, marginBottom: 16, flexWrap: 'wrap' }}>
        <input
          type="search"
          value={searchQuery}
          onChange={e => setSearchQuery(e.target.value)}
          placeholder="ค้นหา..."
        />
        
        <select value={filterStatus} onChange={e => setFilterStatus(e.target.value)}>
          <option value="all">ทั้งหมด</option>
          <option value="active">ใช้งาน</option>
          <option value="inactive">ไม่ใช้งาน</option>
        </select>
        
        <select value={sortBy} onChange={e => setSortBy(e.target.value)}>
          <option value="name">เรียงตามชื่อ</option>
          <option value="createdAt">เรียงตามวันที่สร้าง</option>
          <option value="updatedAt">เรียงตามวันที่แก้ไข</option>
        </select>
        
        <button onClick={() => setSortOrder(prev => prev === 'asc' ? 'desc' : 'asc')}>
          {sortOrder === 'asc' ? '↑ น้อย→มาก' : '↓ มาก→น้อย'}
        </button>
      </div>
      
      <p style={{ color: '#666', fontSize: 14 }}>
        แสดง {processedItems.length} จาก {items.length} รายการ
      </p>
      
      {processedItems.length === 0 ? (
        <p>ไม่พบรายการที่ตรงกับเงื่อนไข</p>
      ) : (
        <ul>
          {processedItems.map(item => (
            <li key={item.id}>{item.name} - {item.status}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## Step 175: Dynamic List CRUD - Add {#step-175}

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: "เรียน React", done: false, priority: 'high', createdAt: new Date() },
    { id: 2, text: "ทำแบบฝึกหัด", done: false, priority: 'medium', createdAt: new Date() },
  ]);
  const [newTodo, setNewTodo] = useState('');
  const [priority, setPriority] = useState('medium');
  const [nextId, setNextId] = useState(3);
  
  const addTodo = () => {
    if (!newTodo.trim()) return;
    
    const todo = {
      id: nextId,
      text: newTodo.trim(),
      done: false,
      priority,
      createdAt: new Date()
    };
    
    setTodos(prev => [...prev, todo]);
    setNextId(prev => prev + 1);
    setNewTodo('');
    setPriority('medium');
  };
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter') addTodo();
  };
  
  return (
    <div>
      <h2>Todo List ({todos.filter(t => !t.done).length} รายการที่รอ)</h2>
      
      {/* Add Form */}
      <div style={{ display: 'flex', gap: 8, marginBottom: 16 }}>
        <input
          value={newTodo}
          onChange={e => setNewTodo(e.target.value)}
          onKeyDown={handleKeyDown}
          placeholder="เพิ่มรายการใหม่..."
          style={{ flex: 1, padding: '8px 12px' }}
        />
        <select 
          value={priority} 
          onChange={e => setPriority(e.target.value)}
        >
          <option value="high">🔴 สูง</option>
          <option value="medium">🟡 ปานกลาง</option>
          <option value="low">🟢 ต่ำ</option>
        </select>
        <button onClick={addTodo}>+ เพิ่ม</button>
      </div>
      
      {/* List */}
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {todos.map(todo => (
          <TodoItem 
            key={todo.id} 
            todo={todo}
            onToggle={() => {}}
            onDelete={() => {}}
            onEdit={() => {}}
          />
        ))}
      </ul>
    </div>
  );
}
```

---

## Step 176: Dynamic List CRUD - Delete {#step-176}

```jsx
function TodoItem({ todo, onToggle, onDelete }) {
  const PRIORITY_COLORS = {
    high: '#ffcdd2',
    medium: '#fff9c4',
    low: '#c8e6c9'
  };
  
  return (
    <li
      style={{
        display: 'flex',
        alignItems: 'center',
        gap: 8,
        padding: '8px 12px',
        marginBottom: 8,
        backgroundColor: PRIORITY_COLORS[todo.priority],
        borderRadius: 8,
        opacity: todo.done ? 0.6 : 1
      }}
    >
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{
        flex: 1,
        textDecoration: todo.done ? 'line-through' : 'none'
      }}>
        {todo.text}
      </span>
      <span style={{ fontSize: 12, color: '#666' }}>
        {todo.createdAt.toLocaleDateString('th-TH')}
      </span>
      <button
        onClick={() => onDelete(todo.id)}
        style={{
          background: 'none',
          border: 'none',
          cursor: 'pointer',
          color: '#f44336',
          fontSize: 16
        }}
        title="ลบ"
      >
        🗑️
      </button>
    </li>
  );
}

function TodoManager() {
  const [todos, setTodos] = useState([
    { id: 1, text: "เรียน React", done: false, priority: 'high', createdAt: new Date() },
    { id: 2, text: "ทำโปรเจค", done: true, priority: 'medium', createdAt: new Date() },
    { id: 3, text: "Test Code", done: false, priority: 'low', createdAt: new Date() },
  ]);
  
  const deleteTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  const deleteCompleted = () => {
    setTodos(prev => prev.filter(todo => !todo.done));
  };
  
  const deleteAll = () => {
    if (window.confirm('ต้องการลบทั้งหมดหรือไม่?')) {
      setTodos([]);
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
  
  const completedCount = todos.filter(t => t.done).length;
  
  return (
    <div>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
        <h2>Todo ({todos.length})</h2>
        <div>
          {completedCount > 0 && (
            <button onClick={deleteCompleted}>ลบที่เสร็จแล้ว ({completedCount})</button>
          )}
          {todos.length > 0 && (
            <button onClick={deleteAll} style={{ marginLeft: 8, color: 'red' }}>
              ลบทั้งหมด
            </button>
          )}
        </div>
      </div>
      
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {todos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={toggleTodo}
            onDelete={deleteTodo}
          />
        ))}
      </ul>
      
      {todos.length === 0 && (
        <p style={{ textAlign: 'center', color: '#888' }}>ไม่มีรายการ</p>
      )}
    </div>
  );
}
```

---

## Step 177: Dynamic List CRUD - Update {#step-177}

```jsx
function EditableTodoItem({ todo, onSave, onCancel, onDelete, onToggle }) {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);
  
  const handleSave = () => {
    if (!editText.trim()) return;
    onSave(todo.id, editText.trim());
    setIsEditing(false);
  };
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter') handleSave();
    if (e.key === 'Escape') {
      setEditText(todo.text);
      setIsEditing(false);
    }
  };
  
  if (isEditing) {
    return (
      <li style={{ display: 'flex', gap: 8, alignItems: 'center', padding: '8px 0' }}>
        <input
          autoFocus
          value={editText}
          onChange={e => setEditText(e.target.value)}
          onKeyDown={handleKeyDown}
          style={{ flex: 1, padding: '4px 8px' }}
        />
        <button onClick={handleSave}>✅ บันทึก</button>
        <button onClick={() => {
          setEditText(todo.text);
          setIsEditing(false);
        }}>❌ ยกเลิก</button>
      </li>
    );
  }
  
  return (
    <li style={{ display: 'flex', gap: 8, alignItems: 'center', padding: '8px 0' }}>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => onToggle(todo.id)}
      />
      <span 
        style={{ flex: 1, textDecoration: todo.done ? 'line-through' : 'none' }}
        onDoubleClick={() => setIsEditing(true)}
      >
        {todo.text}
      </span>
      <button onClick={() => setIsEditing(true)}>✏️</button>
      <button onClick={() => onDelete(todo.id)}>🗑️</button>
    </li>
  );
}

function FullCrudTodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: "เรียน React Hooks", done: false },
    { id: 2, text: "ทำ Todo App", done: false },
    { id: 3, text: "ทดสอบ Code", done: false },
  ]);
  const [newText, setNewText] = useState('');
  
  const addTodo = () => {
    if (!newText.trim()) return;
    setTodos(prev => [...prev, {
      id: Date.now(),
      text: newText.trim(),
      done: false
    }]);
    setNewText('');
  };
  
  const updateTodo = (id, newText) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, text: newText } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  const toggleTodo = (id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
  
  return (
    <div style={{ maxWidth: 500, margin: '0 auto' }}>
      <h2>Full CRUD Todo</h2>
      
      <div style={{ display: 'flex', gap: 8, marginBottom: 16 }}>
        <input
          value={newText}
          onChange={e => setNewText(e.target.value)}
          onKeyDown={e => e.key === 'Enter' && addTodo()}
          placeholder="เพิ่ม todo..."
          style={{ flex: 1 }}
        />
        <button onClick={addTodo}>เพิ่ม</button>
      </div>
      
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {todos.map(todo => (
          <EditableTodoItem
            key={todo.id}
            todo={todo}
            onSave={updateTodo}
            onDelete={deleteTodo}
            onToggle={toggleTodo}
          />
        ))}
      </ul>
      
      <p style={{ fontSize: 14, color: '#888' }}>
        เสร็จแล้ว: {todos.filter(t => t.done).length}/{todos.length}
        (ดับเบิ้ลคลิกเพื่อแก้ไข)
      </p>
    </div>
  );
}
```

---

## Step 178: Paginated List {#step-178}

```jsx
function usePagination(items, itemsPerPage = 10) {
  const [currentPage, setCurrentPage] = useState(1);
  
  const totalPages = Math.ceil(items.length / itemsPerPage);
  
  const paginatedItems = items.slice(
    (currentPage - 1) * itemsPerPage,
    currentPage * itemsPerPage
  );
  
  const goToPage = (page) => {
    setCurrentPage(Math.max(1, Math.min(page, totalPages)));
  };
  
  const nextPage = () => goToPage(currentPage + 1);
  const prevPage = () => goToPage(currentPage - 1);
  
  return {
    currentPage,
    totalPages,
    paginatedItems,
    goToPage,
    nextPage,
    prevPage,
    hasNextPage: currentPage < totalPages,
    hasPrevPage: currentPage > 1
  };
}

function Pagination({ currentPage, totalPages, onPageChange }) {
  const pages = Array.from({ length: totalPages }, (_, i) => i + 1);
  
  // แสดงแค่ pages ที่อยู่ใกล้กับ current
  const visiblePages = pages.filter(page => 
    page === 1 || 
    page === totalPages || 
    Math.abs(page - currentPage) <= 2
  );
  
  return (
    <div style={{ display: 'flex', gap: 4, justifyContent: 'center', marginTop: 16 }}>
      <button 
        onClick={() => onPageChange(currentPage - 1)}
        disabled={currentPage === 1}
      >
        ←
      </button>
      
      {visiblePages.map((page, index, arr) => (
        <React.Fragment key={page}>
          {index > 0 && arr[index - 1] !== page - 1 && (
            <span style={{ padding: '4px 8px' }}>...</span>
          )}
          <button
            onClick={() => onPageChange(page)}
            style={{
              padding: '4px 12px',
              backgroundColor: page === currentPage ? '#4CAF50' : 'white',
              color: page === currentPage ? 'white' : 'black',
              border: '1px solid #ddd',
              borderRadius: 4,
              cursor: 'pointer'
            }}
          >
            {page}
          </button>
        </React.Fragment>
      ))}
      
      <button
        onClick={() => onPageChange(currentPage + 1)}
        disabled={currentPage === totalPages}
      >
        →
      </button>
    </div>
  );
}

function PaginatedUserList({ users }) {
  const {
    currentPage, totalPages, paginatedItems,
    goToPage, hasNextPage, hasPrevPage
  } = usePagination(users, 10);
  
  return (
    <div>
      <p>{users.length} ผู้ใช้ทั้งหมด (หน้า {currentPage}/{totalPages})</p>
      
      <ul>
        {paginatedItems.map(user => (
          <li key={user.id}>{user.name} - {user.email}</li>
        ))}
      </ul>
      
      <Pagination
        currentPage={currentPage}
        totalPages={totalPages}
        onPageChange={goToPage}
      />
    </div>
  );
}
```

---

## Step 179: Infinite Scroll Pattern {#step-179}

```jsx
function useInfiniteScroll(fetchMore, hasMore) {
  const observerRef = useRef(null);
  const sentinelRef = useRef(null);
  
  useEffect(() => {
    observerRef.current = new IntersectionObserver(
      entries => {
        if (entries[0].isIntersecting && hasMore) {
          fetchMore();
        }
      },
      { threshold: 0.1 }
    );
    
    if (sentinelRef.current) {
      observerRef.current.observe(sentinelRef.current);
    }
    
    return () => observerRef.current?.disconnect();
  }, [fetchMore, hasMore]);
  
  return sentinelRef;
}

function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [isLoading, setIsLoading] = useState(false);
  
  const loadMore = useCallback(async () => {
    if (isLoading || !hasMore) return;
    
    setIsLoading(true);
    
    // Mock fetch
    await new Promise(resolve => setTimeout(resolve, 800));
    
    const newItems = Array.from({ length: 10 }, (_, i) => ({
      id: (page - 1) * 10 + i + 1,
      title: `รายการที่ ${(page - 1) * 10 + i + 1}`,
      description: 'คำอธิบายสั้นๆ ของรายการนี้'
    }));
    
    setItems(prev => [...prev, ...newItems]);
    setPage(prev => prev + 1);
    setHasMore(page < 5); // หยุดที่หน้า 5
    setIsLoading(false);
  }, [page, isLoading, hasMore]);
  
  // โหลดข้อมูลแรกตอน mount
  useEffect(() => {
    loadMore();
  }, []);
  
  const sentinelRef = useInfiniteScroll(loadMore, hasMore);
  
  return (
    <div style={{ maxHeight: '60vh', overflowY: 'auto', border: '1px solid #ddd' }}>
      {items.map(item => (
        <div key={item.id} style={{ padding: 16, borderBottom: '1px solid #eee' }}>
          <h4>{item.title}</h4>
          <p style={{ color: '#666', margin: 0 }}>{item.description}</p>
        </div>
      ))}
      
      {/* Sentinel element สำหรับ IntersectionObserver */}
      <div ref={sentinelRef} style={{ padding: 16, textAlign: 'center' }}>
        {isLoading && <p>⏳ กำลังโหลดเพิ่ม...</p>}
        {!hasMore && <p>✅ โหลดครบแล้ว ({items.length} รายการ)</p>}
      </div>
    </div>
  );
}
```

---

## Step 180: Virtualized List {#step-180}

สำหรับ list ขนาดใหญ่มาก (1000+ items) ควรใช้ Virtual Scrolling

```jsx
// Simple Virtual Scroll - render เฉพาะ items ที่มองเห็น
function VirtualList({ items, itemHeight = 50, containerHeight = 400 }) {
  const [scrollTop, setScrollTop] = useState(0);
  
  const visibleCount = Math.ceil(containerHeight / itemHeight);
  const totalHeight = items.length * itemHeight;
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(startIndex + visibleCount + 2, items.length); // +2 buffer
  
  const visibleItems = items.slice(startIndex, endIndex);
  const offsetY = startIndex * itemHeight;
  
  return (
    <div
      style={{ height: containerHeight, overflowY: 'auto', border: '1px solid #ddd' }}
      onScroll={e => setScrollTop(e.target.scrollTop)}
    >
      {/* Total height placeholder */}
      <div style={{ height: totalHeight, position: 'relative' }}>
        {/* Positioned items */}
        <div style={{ position: 'absolute', top: offsetY, width: '100%' }}>
          {visibleItems.map((item, index) => (
            <div
              key={item.id}
              style={{
                height: itemHeight,
                display: 'flex',
                alignItems: 'center',
                padding: '0 16px',
                borderBottom: '1px solid #eee',
                backgroundColor: (startIndex + index) % 2 === 0 ? 'white' : '#fafafa'
              }}
            >
              <span style={{ marginRight: 8, color: '#999', fontSize: 12 }}>
                #{startIndex + index + 1}
              </span>
              {item.name}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// การใช้งาน
function LargeListDemo() {
  const items = Array.from({ length: 10000 }, (_, i) => ({
    id: i + 1,
    name: `รายการที่ ${i + 1} - Lorem ipsum dolor sit amet`
  }));
  
  return (
    <div>
      <h3>Virtual List ({items.length.toLocaleString()} รายการ)</h3>
      <VirtualList items={items} itemHeight={48} containerHeight={400} />
      <p style={{ fontSize: 12, color: '#888' }}>
        * Render เฉพาะ items ที่มองเห็น ประหยัด memory และ performance
      </p>
    </div>
  );
}
```

> **Tip:** สำหรับ production ควรใช้ library เช่น `react-window` หรือ `react-virtualized` แทนที่จะเขียนเอง

---

## Step 181: Grouped Lists {#step-181}

```jsx
function GroupedList({ contacts }) {
  // Group contacts by first letter
  const grouped = contacts.reduce((groups, contact) => {
    const letter = contact.name[0].toUpperCase();
    if (!groups[letter]) {
      groups[letter] = [];
    }
    groups[letter].push(contact);
    return groups;
  }, {});
  
  // Sort groups alphabetically
  const sortedGroups = Object.entries(grouped).sort(([a], [b]) => a.localeCompare(b, 'th'));
  
  return (
    <div>
      {sortedGroups.map(([letter, contacts]) => (
        <div key={letter}>
          <h3 style={{
            position: 'sticky',
            top: 0,
            backgroundColor: '#e8e8e8',
            padding: '4px 16px',
            margin: 0
          }}>
            {letter}
          </h3>
          <ul style={{ listStyle: 'none', padding: 0, margin: 0 }}>
            {contacts.map(contact => (
              <li key={contact.id} style={{
                padding: '12px 16px',
                borderBottom: '1px solid #eee',
                display: 'flex',
                alignItems: 'center',
                gap: 12
              }}>
                <div style={{
                  width: 40,
                  height: 40,
                  borderRadius: '50%',
                  backgroundColor: '#' + Math.floor(Math.random() * 0xffffff).toString(16).padStart(6, '0'),
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                  color: 'white',
                  fontWeight: 'bold'
                }}>
                  {contact.name[0]}
                </div>
                <div>
                  <p style={{ margin: 0, fontWeight: 500 }}>{contact.name}</p>
                  <p style={{ margin: 0, fontSize: 12, color: '#888' }}>{contact.phone}</p>
                </div>
              </li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

---

## Step 182: Drag and Drop List {#step-182}

```jsx
function DraggableList({ initialItems }) {
  const [items, setItems] = useState(initialItems);
  const [draggedId, setDraggedId] = useState(null);
  const [dragOverId, setDragOverId] = useState(null);
  
  const handleDragStart = (e, id) => {
    setDraggedId(id);
    e.dataTransfer.effectAllowed = 'move';
    e.dataTransfer.setData('text/plain', id.toString());
  };
  
  const handleDragOver = (e, id) => {
    e.preventDefault();
    e.dataTransfer.dropEffect = 'move';
    if (id !== draggedId) setDragOverId(id);
  };
  
  const handleDrop = (e, targetId) => {
    e.preventDefault();
    
    if (draggedId === targetId) return;
    
    setItems(prev => {
      const draggedIndex = prev.findIndex(item => item.id === draggedId);
      const targetIndex = prev.findIndex(item => item.id === targetId);
      
      const newItems = [...prev];
      const [draggedItem] = newItems.splice(draggedIndex, 1);
      newItems.splice(targetIndex, 0, draggedItem);
      
      return newItems;
    });
    
    setDraggedId(null);
    setDragOverId(null);
  };
  
  const handleDragEnd = () => {
    setDraggedId(null);
    setDragOverId(null);
  };
  
  return (
    <ul style={{ listStyle: 'none', padding: 0, maxWidth: 400 }}>
      {items.map(item => (
        <li
          key={item.id}
          draggable
          onDragStart={e => handleDragStart(e, item.id)}
          onDragOver={e => handleDragOver(e, item.id)}
          onDrop={e => handleDrop(e, item.id)}
          onDragEnd={handleDragEnd}
          style={{
            padding: '12px 16px',
            marginBottom: 4,
            backgroundColor: 
              draggedId === item.id ? '#e3f2fd' :
              dragOverId === item.id ? '#c8e6c9' : 'white',
            border: '1px solid #ddd',
            borderRadius: 4,
            cursor: 'grab',
            display: 'flex',
            alignItems: 'center',
            gap: 8,
            opacity: draggedId === item.id ? 0.5 : 1,
            transition: 'background-color 0.1s, transform 0.1s',
            transform: dragOverId === item.id ? 'scale(1.02)' : 'scale(1)'
          }}
        >
          <span style={{ color: '#999', cursor: 'grab' }}>⠿</span>
          <span>{item.text}</span>
        </li>
      ))}
    </ul>
  );
}

// การใช้งาน
function App() {
  const items = [
    { id: 1, text: "รายการที่ 1 - Drag me!" },
    { id: 2, text: "รายการที่ 2 - I can be moved" },
    { id: 3, text: "รายการที่ 3 - Drop here" },
    { id: 4, text: "รายการที่ 4 - Reorderable" },
  ];
  
  return <DraggableList initialItems={items} />;
}
```

---

## Step 183: Search and Filter List {#step-183}

```jsx
function SearchableList({ items }) {
  const [query, setQuery] = useState('');
  const [tags, setTags] = useState([]);
  const [selectedTags, setSelectedTags] = useState([]);
  
  // Get all unique tags
  useEffect(() => {
    const allTags = [...new Set(items.flatMap(item => item.tags || []))];
    setTags(allTags);
  }, [items]);
  
  // Filter logic
  const filteredItems = useMemo(() => {
    let result = items;
    
    if (query) {
      const q = query.toLowerCase();
      result = result.filter(item => 
        item.title.toLowerCase().includes(q) ||
        item.description?.toLowerCase().includes(q) ||
        item.author?.toLowerCase().includes(q)
      );
    }
    
    if (selectedTags.length > 0) {
      result = result.filter(item =>
        selectedTags.every(tag => item.tags?.includes(tag))
      );
    }
    
    return result;
  }, [items, query, selectedTags]);
  
  const toggleTag = (tag) => {
    setSelectedTags(prev =>
      prev.includes(tag) ? prev.filter(t => t !== tag) : [...prev, tag]
    );
  };
  
  const clearFilters = () => {
    setQuery('');
    setSelectedTags([]);
  };
  
  const hasFilters = query || selectedTags.length > 0;
  
  return (
    <div>
      {/* Search input */}
      <div style={{ position: 'relative', marginBottom: 12 }}>
        <input
          type="search"
          value={query}
          onChange={e => setQuery(e.target.value)}
          placeholder="ค้นหา..."
          style={{ 
            width: '100%', 
            padding: '10px 36px 10px 12px',
            border: '2px solid #ddd',
            borderRadius: 8
          }}
        />
        {query && (
          <button
            onClick={() => setQuery('')}
            style={{
              position: 'absolute',
              right: 8,
              top: '50%',
              transform: 'translateY(-50%)',
              background: 'none',
              border: 'none',
              cursor: 'pointer'
            }}
          >
            ✕
          </button>
        )}
      </div>
      
      {/* Tag filters */}
      <div style={{ display: 'flex', flexWrap: 'wrap', gap: 8, marginBottom: 16 }}>
        {tags.map(tag => (
          <button
            key={tag}
            onClick={() => toggleTag(tag)}
            style={{
              padding: '4px 12px',
              border: '1px solid #ddd',
              borderRadius: 20,
              cursor: 'pointer',
              backgroundColor: selectedTags.includes(tag) ? '#4CAF50' : 'white',
              color: selectedTags.includes(tag) ? 'white' : 'black',
              fontSize: 13
            }}
          >
            {tag}
          </button>
        ))}
        {hasFilters && (
          <button
            onClick={clearFilters}
            style={{
              padding: '4px 12px',
              border: '1px solid #ff5722',
              borderRadius: 20,
              cursor: 'pointer',
              backgroundColor: 'white',
              color: '#ff5722',
              fontSize: 13
            }}
          >
            ล้างตัวกรอง
          </button>
        )}
      </div>
      
      {/* Results count */}
      <p style={{ color: '#666', fontSize: 14, marginBottom: 8 }}>
        {hasFilters 
          ? `พบ ${filteredItems.length} จาก ${items.length} รายการ`
          : `${items.length} รายการทั้งหมด`
        }
      </p>
      
      {/* Results */}
      {filteredItems.length === 0 ? (
        <div style={{ textAlign: 'center', padding: 40, color: '#888' }}>
          <p>🔍 ไม่พบรายการที่ตรงกับการค้นหา</p>
          <button onClick={clearFilters}>ล้างตัวกรองทั้งหมด</button>
        </div>
      ) : (
        <div>
          {filteredItems.map(item => (
            <div key={item.id} style={{ padding: 16, border: '1px solid #ddd', borderRadius: 8, marginBottom: 8 }}>
              <h3>{item.title}</h3>
              <p>{item.description}</p>
              <div>
                {item.tags?.map(tag => (
                  <span key={tag} style={{
                    display: 'inline-block',
                    padding: '2px 8px',
                    margin: '0 4px',
                    backgroundColor: selectedTags.includes(tag) ? '#e8f5e9' : '#f5f5f5',
                    borderRadius: 12,
                    fontSize: 12
                  }}>
                    {tag}
                  </span>
                ))}
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Step 184: List Performance Optimization {#step-184}

```jsx
// 1. React.memo - ป้องกัน re-render ที่ไม่จำเป็น
const ListItem = React.memo(function ListItem({ item, onDelete }) {
  console.log(`Rendering item: ${item.id}`); // ดู re-renders
  
  return (
    <li>
      <span>{item.name}</span>
      <button onClick={() => onDelete(item.id)}>ลบ</button>
    </li>
  );
});

// 2. useCallback - stabilize callbacks
function OptimizedList({ initialItems }) {
  const [items, setItems] = useState(initialItems);
  
  // ✅ useCallback ทำให้ function reference คงที่
  const handleDelete = useCallback((id) => {
    setItems(prev => prev.filter(item => item.id !== id));
  }, []); // ไม่มี dependencies = function เดิมทุกครั้ง
  
  return (
    <ul>
      {items.map(item => (
        // ListItem จะไม่ re-render ถ้า item และ onDelete ไม่เปลี่ยน
        <ListItem key={item.id} item={item} onDelete={handleDelete} />
      ))}
    </ul>
  );
}

// 3. useMemo - cache expensive computations
function ExpensiveList({ items, filter, sort }) {
  const processedItems = useMemo(() => {
    console.log("Processing items..."); // log เมื่อ recalculate
    
    return items
      .filter(item => filter ? item.category === filter : true)
      .sort((a, b) => a[sort]?.localeCompare(b[sort], 'th') ?? 0);
  }, [items, filter, sort]); // recalculate เมื่อ dependencies เปลี่ยน
  
  return (
    <ul>
      {processedItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// 4. Key stability - ใช้ stable IDs เสมอ
// ❌ ทำให้ React สร้าง component ใหม่ทุกครั้ง
{items.map(item => <Item key={Math.random()} item={item} />)}

// ✅ React รู้ว่า component ไหนคือตัวไหน
{items.map(item => <Item key={item.id} item={item} />)}
```

---

## Step 185: Best Practices {#step-185}

### 1. ใช้ Stable IDs เสมอ

```jsx
// ✅ IDs จาก database
{users.map(user => <UserCard key={user.id} user={user} />)}

// ✅ UUIDs ที่สร้างก่อน render
const createItem = (text) => ({
  id: crypto.randomUUID(), // สร้างครั้งเดียว
  text,
  done: false
});
```

### 2. แยก Component สำหรับ List Items

```jsx
// ✅ แยก component ทำให้ optimize ได้ง่ายกว่า
function UserList({ users }) {
  return users.map(user => <UserCard key={user.id} user={user} />);
}

// แทนที่จะเขียน JSX ยาวๆ ใน map
function UserList({ users }) {
  return users.map(user => (
    <div key={user.id}>
      {/* JSX ยาวมาก... */}
    </div>
  ));
}
```

### 3. จัดการ Empty State

```jsx
function SafeList({ items, emptyMessage = "ไม่มีข้อมูล" }) {
  if (!items || items.length === 0) {
    return <p>{emptyMessage}</p>;
  }
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
}
```

### 4. ไม่ใช้ index เป็น key ถ้า list เปลี่ยนได้

```jsx
// ❌ ปัญหาเมื่อ add/remove/reorder
{items.map((item, index) => <Item key={index} item={item} />)}

// ✅ ดีกว่า
{items.map(item => <Item key={item.id} item={item} />)}
```

### 5. Virtualize list ขนาดใหญ่

```jsx
// ใช้ react-window สำหรับ list ที่มีหลาย 100+ items
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </FixedSizeList>
  );
}
```

### 6. Filter/Sort อยู่นอก render

```jsx
// ✅ ใช้ useMemo ป้องกัน recalculate ทุก render
const sortedAndFiltered = useMemo(() => 
  items
    .filter(item => item.isActive)
    .sort((a, b) => a.name.localeCompare(b.name, 'th')),
  [items]
);
```

---

## Quiz และแบบฝึกหัด {#quiz}

### คำถาม

1. ทำไม React ต้องการ key prop ใน list rendering?
2. ทำไมไม่ควรใช้ index เป็น key เมื่อ list เปลี่ยนได้?
3. Virtualized List แตกต่างจาก Pagination อย่างไร?
4. เมื่อไหรควรใช้ useMemo กับ list processing?
5. Infinite Scroll vs Pagination ควรใช้เมื่อไหร่?

### แบบฝึกหัด

**Exercise 1:** สร้าง `ContactList`:
- ข้อมูล 20+ contacts พร้อม name, phone, email, tags
- Search by name, phone, email
- Filter by tags (หลาย tags ได้)
- Sort by name (A-Z, Z-A)
- แสดง count ของ filtered results

**Exercise 2:** สร้าง `Kanban Board`:
- 3 columns: Todo, In Progress, Done
- drag card ระหว่าง columns ได้
- เพิ่ม/ลบ cards ได้
- แสดง count ใน header ของแต่ละ column

**Exercise 3:** สร้าง `ProductCatalog` with:
- Grid/List view toggle
- Filter by category, price range, rating
- Sort options
- Pagination (10 items/page)
- แสดง total count และ page info

---

## สรุป

ใน Part นี้เราได้เรียนรู้:
- Array.map() สำหรับ render lists
- Key prop และความสำคัญ
- Nested Lists และ Recursive Rendering
- Filter, Sort ก่อน Render
- CRUD operations บน list state
- Pagination และ Infinite Scroll
- Virtualized List สำหรับ performance
- Grouped Lists และ Drag & Drop
- Search and Filter
- Performance Optimization

---

## อ่านต่อ

➡️ **[Part 10: Forms พื้นฐาน](./part-10-forms-basics.md)**

เราจะเรียนรู้ Controlled vs Uncontrolled Components, Form Validation, และการสร้าง Form ที่ซับซ้อน

---

*React Course - Part 09 | สร้างโดย React Course Thailand*
