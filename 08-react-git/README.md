# PHẦN 8: REACTJS & GIT - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: TRUNG BÌNH**
> JD yêu cầu "thành thạo ReactJS ở phía Frontend" và "kinh nghiệm sử dụng Git 
> trong môi trường nhóm". Cần nắm vững concepts cốt lõi.

---

# PHẦN A: REACTJS

## 1. REACT LÀ GÌ?

React = **UI library** (KHÔNG phải framework) cho xây dựng giao diện.
- Component-based: UI chia thành các component nhỏ, tái sử dụng
- Declarative: Mô tả UI muốn có, React lo cập nhật DOM
- Virtual DOM: Hiệu năng cao nhờ so sánh trước khi update thật

### Virtual DOM & Reconciliation:

```
Tại sao cần Virtual DOM?

Thao tác Real DOM rất CHẬM (reflow, repaint).
React dùng Virtual DOM (JS object nhẹ) để optimize:

1. State thay đổi → React tạo Virtual DOM mới
2. So sánh (diff) Virtual DOM mới vs cũ → tìm differences
3. Chỉ update PHẦN THAY ĐỔI trên Real DOM → NHANH!

┌─────────────────────┐
│   Virtual DOM (cũ)   │
│   <div>              │
│     <h1>Hello</h1>   │
│     <p>World</p>     │      ┌──────────────┐
│   </div>             │ DIFF │  Chỉ thay đổi │
├─────────────────────┤─────>│  text "World" │──> Update Real DOM
│   Virtual DOM (mới)  │      │  → "React"    │    (chỉ 1 element)
│   <div>              │      └──────────────┘
│     <h1>Hello</h1>   │
│     <p>React</p> ← changed
│   </div>             │
└─────────────────────┘

Reconciliation = Thuật toán so sánh 2 trees:
- Khác type (<div> → <span>) → Tạo lại toàn bộ subtree
- Cùng type → So sánh attributes, cập nhật khác biệt
- List items → Dùng "key" prop để identify → tránh re-render không cần
```

---

## 2. HOOKS (CỐT LÕI CỦA REACT HIỆN ĐẠI)

### useState - Quản lý state:

```typescript
function Counter() {
  const [count, setCount] = useState(0);
  //     ↑         ↑                ↑
  //  current   function to      initial
  //  value     update state     value
  
  // ⚠️ State updates are ASYNCHRONOUS và BATCHED
  const handleClick = () => {
    setCount(count + 1);     // ❌ Có thể sai nếu gọi liên tục
    setCount(prev => prev + 1); // ✅ Dùng callback form khi dựa vào state trước
  };
  
  // ⚠️ Object/Array state: phải tạo MỚI, không mutate
  const [user, setUser] = useState({ name: 'Alice', age: 25 });
  setUser({ ...user, age: 26 });      // ✅ Spread + update
  // user.age = 26; setUser(user);     // ❌ React không detect thay đổi
  
  return <button onClick={handleClick}>{count}</button>;
}
```

### useEffect - Side effects:

```typescript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // SIDE EFFECT: fetch data, subscriptions, timers...
    
    // ═══ Dependency Array quyết định KHI NÀO chạy ═══
    
    async function fetchUser() {
      const res = await fetch(`/api/users/${userId}`);
      const data = await res.json();
      setUser(data);
    }
    fetchUser();
    
    // CLEANUP function: chạy khi component unmount hoặc trước khi effect chạy lại
    return () => {
      // Cancel subscriptions, clear timers, abort fetch...
      console.log('Cleanup!');
    };
  }, [userId]);
  // ↑ Dependency Array:
  // []        → Chạy 1 lần khi mount (như componentDidMount)
  // [userId]  → Chạy khi userId thay đổi
  // không có  → Chạy MỖI LẦN render (nguy hiểm, tránh dùng)
  
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

### useContext - Chia sẻ state toàn app (tránh prop drilling):

```typescript
// Prop Drilling Problem:
// App → Layout → Sidebar → UserInfo → Avatar → {user.name}
// Phải truyền "user" qua 5 levels dù chỉ Avatar cần!

// ✅ Solution: Context
const AuthContext = createContext<AuthContextType | null>(null);

// Provider (bao ngoài, cung cấp data)
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = async (credentials) => { /* ... */ };
  const logout = () => { /* ... */ };
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Consumer (bất kỳ component con nào cần)
function Avatar() {
  const { user } = useContext(AuthContext);  // Lấy trực tiếp, không cần prop drilling
  return <img src={user?.avatar} />;
}

// Setup
function App() {
  return (
    <AuthProvider>
      <Layout />  {/* Mọi component con đều access được AuthContext */}
    </AuthProvider>
  );
}
```

### useMemo & useCallback (Performance):

```typescript
// useMemo: Cache KẾT QUẢ tính toán (tránh re-compute)
function ProductList({ products, filter }) {
  // ❌ Mỗi render đều filter lại (dù products/filter không đổi)
  const filtered = products.filter(p => p.category === filter);
  
  // ✅ Chỉ filter lại khi products hoặc filter thay đổi
  const filtered = useMemo(
    () => products.filter(p => p.category === filter),
    [products, filter]  // dependencies
  );
  
  return filtered.map(p => <ProductCard key={p.id} product={p} />);
}

// useCallback: Cache FUNCTION reference (tránh child re-render)
function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ Mỗi render tạo function MỚI → Child re-render (dù logic giống)
  const handleClick = () => { console.log('clicked'); };
  
  // ✅ Giữ cùng function reference giữa các renders
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);  // dependencies (trống = không bao giờ tạo lại)
  
  return <ExpensiveChild onClick={handleClick} />;
}

// ⚠️ ĐỪNG dùng useMemo/useCallback cho mọi thứ!
// Chỉ dùng khi:
// 1. Tính toán NẶNG (sort/filter array lớn)
// 2. Truyền function/object xuống React.memo component
// 3. Là dependency của useEffect
```

### useRef - Tham chiếu DOM / giữ giá trị giữa renders:

```typescript
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  // 1. Truy cập DOM element
  const focusInput = () => {
    inputRef.current?.focus();  // Focus input trực tiếp
  };
  
  return <input ref={inputRef} />;
}

function Timer() {
  const intervalRef = useRef<number>();
  
  // 2. Giữ giá trị KHÔNG gây re-render (khác useState)
  useEffect(() => {
    intervalRef.current = setInterval(() => console.log('tick'), 1000);
    return () => clearInterval(intervalRef.current);
  }, []);
}
```

### Custom Hooks - Tái sử dụng logic:

```typescript
// Custom hook = function bắt đầu bằng "use", dùng hooks bên trong

// useFetch: logic fetch data tái sử dụng
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchData() {
      try {
        setLoading(true);
        const res = await fetch(url, { signal: controller.signal });
        const json = await res.json();
        setData(json);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err as Error);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchData();
    return () => controller.abort();  // Cleanup: cancel fetch on unmount
  }, [url]);
  
  return { data, loading, error };
}

// Sử dụng
function UserProfile({ id }) {
  const { data: user, loading, error } = useFetch(`/api/users/${id}`);
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  return <div>{user.name}</div>;
}
```

---

## 3. STATE MANAGEMENT

```
Khi nào cần state management?

Local state (useState):
→ State chỉ 1 component cần: form input, toggle, counter

Lifting state up:
→ 2-3 components cần cùng state: lift lên parent chung

Context API:
→ State cần ở nhiều nơi, nhưng ít update: theme, auth, language

Redux/Zustand (External store):
→ State phức tạp, update thường xuyên, nhiều components
→ Cần middleware (async actions, logging)

React Query / TanStack Query:
→ SERVER STATE (data từ API): caching, refetching, synchronization
→ RECOMMENDED cho API data thay vì Redux

Hierarchy:
useState (1 comp) → lifted state (vài comps) → Context (app-wide, ít update) 
→ Zustand/Redux (complex) → React Query (server state)
```

### React Query (TanStack Query) - Server State:

```typescript
// Thay vì fetch + useState + useEffect + loading + error + cache...
// React Query xử lý TẤT CẢ:

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],           // Cache key
    queryFn: () => fetchUsers(),   // Fetch function
    staleTime: 5 * 60 * 1000,     // Data "tươi" trong 5 phút
    retry: 3,                      // Retry 3 lần nếu fail
  });
  
  if (isLoading) return <Spinner />;
  if (error) return <Error />;
  return data.map(user => <UserCard key={user.id} user={user} />);
}

// Mutation (create/update/delete)
function CreateUser() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: (newUser) => createUser(newUser),
    onSuccess: () => {
      // Invalidate cache → auto refetch user list
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
  
  return <button onClick={() => mutation.mutate({ name: 'Alice' })}>Create</button>;
}
```

---

## 4. PERFORMANCE OPTIMIZATION

```typescript
// 1. React.memo - Skip re-render nếu props không đổi
const ExpensiveComponent = React.memo(({ data }) => {
  // Chỉ re-render khi data thay đổi (shallow comparison)
  return <div>{/* expensive render */}</div>;
});

// 2. Code Splitting - Load component khi cần
const HeavyChart = React.lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />  {/* Load khi render, không load ban đầu */}
    </Suspense>
  );
}

// 3. Virtualization - Render chỉ items visible (cho list dài)
import { FixedSizeList } from 'react-window';

function LongList({ items }) {  // 10000 items
  return (
    <FixedSizeList height={500} itemCount={items.length} itemSize={35}>
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
    // Chỉ render ~15 items visible, không phải 10000!
  );
}
```

---

# PHẦN B: GIT & GIT FLOW

## 1. GIT FUNDAMENTALS

### 3 Vùng làm việc:

```
┌──────────────┐    git add    ┌──────────────┐   git commit   ┌──────────────┐
│  Working     │──────────────>│   Staging    │───────────────>│  Repository  │
│  Directory   │               │   Area       │                │  (.git)      │
│              │<──────────────│   (Index)    │<───────────────│              │
│  (files bạn  │  git checkout │  (chuẩn bị   │   git reset    │  (history    │
│   đang sửa)  │               │   commit)    │                │   commits)   │
└──────────────┘               └──────────────┘                └──────────────┘
                                                                      │
                                                                 git push
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │   Remote     │
                                                               │  (GitHub)    │
                                                               └──────────────┘
```

### Commands quan trọng:

```bash
# ═══ BASIC ═══
git init                          # Khởi tạo repo
git clone <url>                   # Clone repo
git status                        # Xem trạng thái files
git add .                         # Stage ALL changes
git add <file>                    # Stage file cụ thể
git commit -m "feat: add login"   # Commit
git push origin main              # Push lên remote
git pull origin main              # Pull từ remote (fetch + merge)

# ═══ BRANCHING ═══
git branch                        # Liệt kê branches
git branch feature/login          # Tạo branch mới
git checkout feature/login        # Chuyển branch
git checkout -b feature/login     # Tạo + chuyển (shortcut)
git switch feature/login          # Chuyển branch (mới hơn)
git branch -d feature/login       # Xóa branch (đã merge)
git branch -D feature/login       # Xóa branch (force, chưa merge)

# ═══ VIEWING ═══
git log --oneline --graph         # Xem history dạng graph
git diff                          # Xem changes chưa staged
git diff --staged                 # Xem changes đã staged

# ═══ STASH (lưu tạm changes) ═══
git stash                         # Lưu tạm changes
git stash pop                     # Lấy lại changes + xóa khỏi stash
git stash list                    # Liệt kê stash
git stash apply stash@{0}        # Lấy stash cụ thể (không xóa)
```

## 2. MERGE vs REBASE (HAY HỎI PHỎNG VẤN)

```
═══ MERGE ═══
Tạo merge commit, giữ nguyên lịch sử.

main:    A ── B ── C ──────── M (merge commit)
                    \        /
feature:             D ── E

git checkout main
git merge feature

✅ Ưu: Bảo toàn lịch sử đầy đủ, an toàn
❌ Nhược: Lịch sử phức tạp, nhiều merge commits


═══ REBASE ═══
"Di chuyển" commits của branch lên trên branch đích. Lịch sử thẳng.

Trước rebase:
main:    A ── B ── C
                    \
feature:             D ── E

Sau rebase:
main:    A ── B ── C
                    \
feature:             D' ── E' (re-applied trên C)

git checkout feature
git rebase main

Sau đó merge (fast-forward):
main:    A ── B ── C ── D' ── E'

✅ Ưu: Lịch sử sạch, linear (dễ đọc)
❌ Nhược: THAY ĐỔI history → KHÔNG rebase shared branches!


═══ KHI NÀO DÙNG GÌ? ═══
MERGE: 
  - Merge feature → main/develop (qua Pull Request)
  - Khi cần bảo toàn đầy đủ lịch sử
  - Shared branches

REBASE:
  - Update feature branch với changes mới từ main
  - Cleanup commits trước khi tạo PR
  - Local branches (chưa push)
  
⚠️ GOLDEN RULE: KHÔNG rebase branches đã push/share với người khác
```

## 3. RESET vs REVERT

```bash
═══ RESET ═══ (THAY ĐỔI history - nguy hiểm)
# --soft: giữ changes trong staging area
git reset --soft HEAD~1    # Undo commit cuối, changes vẫn staged

# --mixed (default): giữ changes trong working directory
git reset HEAD~1           # Undo commit, changes unstaged

# --hard: XÓA HẾT changes
git reset --hard HEAD~1    # Undo commit + XÓA changes (không recover!)

# ⚠️ Chỉ dùng cho LOCAL commits (chưa push)


═══ REVERT ═══ (TẠO commit mới để undo - an toàn)
git revert <commit-hash>   # Tạo commit mới "ngược lại" commit cũ

A ── B ── C ── D
          git revert C
A ── B ── C ── D ── C' (revert of C)

# ✅ An toàn cho shared branches (không thay đổi history)
```

## 4. BRANCHING STRATEGIES

### Git Flow:

```
main (production) ────●──────────────────────●──────── ← Releases tags
                       \                    /
develop ──────●─────●───●──────●─────●────●─────────── ← Integration
              \   / \  /      \   /      \
feature/      ●──●   ●─●     ●──●       release/
login               search                v1.2
                                           \
                                          hotfix/
                                          fix-bug
                                             \
main ──────────────────────────────────────●──

Branches:
main       → Production code (stable)
develop    → Integration branch (next release)
feature/*  → New features (từ develop, merge vào develop)
release/*  → Chuẩn bị release (từ develop, merge vào main)
hotfix/*   → Fix bug production (từ main, merge vào main + develop)
```

### GitHub Flow (đơn giản hơn):

```
main ──────●──────────────●─────── ← Always deployable
            \            /
feature/    ●──●──●──●──● → Pull Request → Code Review → Merge
login

Chỉ 2 loại branch:
main      → Production, always deployable
feature/* → Tất cả changes qua feature branch + PR

Phù hợp: CI/CD, deploy thường xuyên, team nhỏ-vừa
```

## 5. PULL REQUEST WORKFLOW

```
1. Tạo feature branch từ main/develop
   git checkout -b feature/add-user-search

2. Code, commit (conventional commits)
   git commit -m "feat: add user search API"
   git commit -m "test: add unit tests for search"

3. Push branch
   git push origin feature/add-user-search

4. Tạo Pull Request trên GitHub/GitLab
   - Title: "feat: add user search functionality"
   - Description: What, Why, How, Testing
   - Reviewers: assign team members

5. Code Review
   - Reviewer check code quality, logic, tests
   - Request changes nếu cần
   - Approve khi OK

6. Merge
   - Squash merge: gom tất cả commits thành 1 (clean history)
   - Merge commit: giữ tất cả commits (full history)
   - Rebase merge: linear history

7. Delete branch
```

### Conventional Commits:

```
feat:     Tính năng mới          feat: add user search API
fix:      Fix bug                fix: resolve login timeout issue
docs:     Documentation          docs: update API documentation
style:    Code style (không logic) style: format code with prettier
refactor: Refactor code          refactor: extract validation logic
test:     Testing                test: add unit tests for auth
chore:    Maintenance            chore: update dependencies
perf:     Performance            perf: optimize database queries
ci:       CI/CD                  ci: add GitHub Actions workflow

Breaking change:
feat!: drop support for Node 14
```

---

## TÓM TẮT PHẦN 8

### Câu hỏi React:
**1. "Virtual DOM hoạt động thế nào?"**
→ React dùng JS object (Virtual DOM) để so sánh (diff) trước khi update Real DOM.
Chỉ update phần thay đổi, hiệu quả hơn direct DOM manipulation.

**2. "useMemo vs useCallback?"**
→ useMemo: cache giá trị tính toán. useCallback: cache function reference.
Dùng khi truyền xuống React.memo component hoặc dependency của effect.

**3. "Bạn quản lý state thế nào?"**
→ Local: useState. Shared: lifting up / Context. Server state: React Query.
Complex app state: Zustand/Redux. Chọn đúng level cho đúng use case.

### Câu hỏi Git:
**4. "Merge vs Rebase?"**
→ Merge: tạo merge commit, an toàn, giữ history. 
Rebase: linear history, sạch hơn, chỉ dùng cho local branches.

**5. "Git Flow?"**
→ main (production), develop (integration), feature/*, release/*, hotfix/*.
Feature branches từ develop, merge qua PR + code review.

**6. "Giải quyết merge conflict?"**
→ Pull latest changes, xem conflict markers (<<<, ===, >>>), chọn code đúng,
test, commit. Dùng IDE để giải quyết visual.
