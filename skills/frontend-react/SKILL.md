---
name: frontend-react
description: React expert for *.jsx/*.tsx files. Covers hooks (useState, useEffect), components, context, Redux, Next.js basics. Use for any React question.
model: opus
---

# React Skill

Apply these React patterns and practices when working with React code.

## Code Style

- Use functional components exclusively
- Use meaningful component and prop names
- Avoid comments - code should be self-documenting
- Maximum 2 props inline, use object destructuring for more
- One component per file
- Co-locate related files (component, styles, tests, types)

## Component Patterns

### Props Interface
```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary'
  disabled?: boolean
  onClick: () => void
  children: React.ReactNode
}

function Button({ variant, disabled = false, onClick, children }: ButtonProps) {
  return (
    <button
      className={styles[variant]}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  )
}
```

### Composition Over Props
```tsx
// Prefer composition
function Card({ children }: { children: React.ReactNode }) {
  return <div className={styles.card}>{children}</div>
}

function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className={styles.header}>{children}</div>
}

function CardBody({ children }: { children: React.ReactNode }) {
  return <div className={styles.body}>{children}</div>
}

// Usage
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
</Card>
```

### Render Props Pattern
```tsx
interface DataLoaderProps<T> {
  url: string
  children: (data: T, loading: boolean) => React.ReactNode
}

function DataLoader<T>({ url, children }: DataLoaderProps<T>) {
  const { data, loading } = useFetch<T>(url)
  return <>{children(data, loading)}</>
}
```

## Hooks Patterns

### Custom Hooks
```tsx
function useToggle(initial = false): [boolean, () => void] {
  const [value, setValue] = useState(initial)
  const toggle = useCallback(() => setValue(v => !v), [])
  return [value, toggle]
}

function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debounced
}
```

### useEffect Patterns
```tsx
// Fetch data
useEffect(() => {
  const controller = new AbortController()

  async function fetchData() {
    try {
      const response = await fetch(url, { signal: controller.signal })
      const data = await response.json()
      setData(data)
    } catch (e) {
      if (e instanceof Error && e.name !== 'AbortError') {
        setError(e)
      }
    }
  }

  fetchData()
  return () => controller.abort()
}, [url])

// Event listeners
useEffect(() => {
  function handleResize() {
    setWidth(window.innerWidth)
  }

  window.addEventListener('resize', handleResize)
  return () => window.removeEventListener('resize', handleResize)
}, [])
```

### useMemo and useCallback
```tsx
// Expensive computations
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
)

// Stable callback references
const handleSubmit = useCallback(
  async (data: FormData) => {
    await api.submit(data)
    onSuccess()
  },
  [onSuccess]
)
```

## State Management

### Local State
```tsx
const [user, setUser] = useState<User | null>(null)
const [status, setStatus] = useState<'idle' | 'loading' | 'error'>('idle')
```

### Reducer for Complex State
```tsx
type Action =
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: User[] }
  | { type: 'FETCH_ERROR'; error: string }

interface State {
  users: User[]
  loading: boolean
  error: string | null
}

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null }
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, users: action.payload }
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.error }
  }
}
```

### Context for Shared State
```tsx
interface AuthContextValue {
  user: User | null
  login: (credentials: Credentials) => Promise<void>
  logout: () => void
}

const AuthContext = createContext<AuthContextValue | null>(null)

function useAuth(): AuthContextValue {
  const context = useContext(AuthContext)
  if (!context) throw new Error('useAuth must be within AuthProvider')
  return context
}

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null)

  const login = useCallback(async (credentials: Credentials) => {
    const user = await api.login(credentials)
    setUser(user)
  }, [])

  const logout = useCallback(() => setUser(null), [])

  const value = useMemo(
    () => ({ user, login, logout }),
    [user, login, logout]
  )

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>
}
```

## Performance

### Prevent Unnecessary Renders
```tsx
// Memoize expensive components
const MemoizedList = memo(function List({ items }: ListProps) {
  return items.map(item => <ListItem key={item.id} item={item} />)
})

// Custom comparison
const MemoizedItem = memo(
  function Item({ item }: ItemProps) { /* ... */ },
  (prev, next) => prev.item.id === next.item.id
)
```

### Lazy Loading
```tsx
const Dashboard = lazy(() => import('./pages/Dashboard'))
const Settings = lazy(() => import('./pages/Settings'))

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  )
}
```

## Project Structure

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── Button.module.css
│   └── index.ts
├── hooks/
│   ├── useAuth.ts
│   └── useDebounce.ts
├── pages/
│   ├── Dashboard.tsx
│   └── Settings.tsx
├── services/
│   └── api.ts
├── types/
│   └── user.ts
└── App.tsx
```

## Quality Checklist

When writing React code, verify:
- [ ] Components have typed props interface
- [ ] useEffect has proper cleanup
- [ ] useEffect dependencies are complete
- [ ] useMemo/useCallback used for expensive operations
- [ ] Keys are stable and unique in lists
- [ ] Error boundaries handle component errors
- [ ] Loading and error states handled
- [ ] No inline object/function props causing re-renders
