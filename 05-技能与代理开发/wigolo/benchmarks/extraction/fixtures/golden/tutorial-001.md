# Building Custom React Hooks: A Practical Guide

React hooks let you extract and reuse stateful logic across components. This tutorial walks through building five custom hooks from scratch, each solving a real problem you will encounter in production applications.

## Prerequisites

- React 18 or later
- Basic understanding of `useState`, `useEffect`, and `useRef`
- A project set up with TypeScript (plain JavaScript examples are noted where different)

## 1. useLocalStorage — Persistent State

The built-in `useState` hook loses its value on page refresh. This hook syncs state with `localStorage` so values persist across sessions.

### Step 1: Define the Hook Signature

```tsx
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T | ((prev: T) => T)) => void] {
```

The hook accepts a storage key and an initial value. It returns a tuple matching the `useState` API.

### Step 2: Initialize State from Storage

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });
```

The lazy initializer function runs only on the first render. It reads from `localStorage` and falls back to the initial value if the key does not exist or parsing fails.

### Step 3: Sync Changes to Storage

```tsx
  const setValue = useCallback(
    (value: T | ((prev: T) => T)) => {
      try {
        const valueToStore = value instanceof Function ? value(storedValue) : value;
        setStoredValue(valueToStore);
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      } catch (error) {
        console.warn(`Error writing localStorage key "${key}":`, error);
      }
    },
    [key, storedValue],
  );

  return [storedValue, setValue] as const;
}
```

### Step 4: Use It

```tsx
function SettingsPanel() {
  const [theme, setTheme] = useLocalStorage("theme", "light");
  const [fontSize, setFontSize] = useLocalStorage("fontSize", 16);

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      <input
        type="range"
        min={12}
        max={24}
        value={fontSize}
        onChange={(e) => setFontSize(Number(e.target.value))}
      />
    </div>
  );
}
```

> **Tip:** Add a `useEffect` that listens for `storage` events if you need cross-tab synchronization.

## 2. useDebounce — Delayed Value Updates

Debouncing prevents expensive operations (API calls, search queries) from firing on every keystroke.

### Step 1: Build the Hook

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

Every time `value` changes, the effect sets a new timeout. If `value` changes again before the timeout fires, the cleanup function clears the previous timer.

### Step 2: Use It with a Search Input

```tsx
function SearchPage() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);
  const [results, setResults] = useState<SearchResult[]>([]);

  useEffect(() => {
    if (!debouncedQuery) {
      setResults([]);
      return;
    }

    const controller = new AbortController();

    fetch(`https://api.example.com/search?q=${encodeURIComponent(debouncedQuery)}`, {
      signal: controller.signal,
    })
      .then((res) => res.json())
      .then((data) => setResults(data.results))
      .catch((err) => {
        if (err.name !== "AbortError") console.error(err);
      });

    return () => controller.abort();
  }, [debouncedQuery]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Search..." />
      <ul>
        {results.map((r) => (
          <li key={r.id}>{r.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

> **Tip:** The `AbortController` in the effect cleanup prevents stale responses from overwriting newer results.

## 3. useFetch — Data Fetching with Loading and Error States

A minimal data-fetching hook that handles loading, error, and success states.

### Step 1: Define the Return Type

```tsx
interface UseFetchResult<T> {
  data: T | null;
  error: Error | null;
  isLoading: boolean;
  refetch: () => void;
}
```

### Step 2: Implement the Hook

```tsx
function useFetch<T>(url: string, options?: RequestInit): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [fetchCount, setFetchCount] = useState(0);

  const refetch = useCallback(() => {
    setFetchCount((c) => c + 1);
  }, []);

  useEffect(() => {
    const controller = new AbortController();
    setIsLoading(true);
    setError(null);

    fetch(url, { ...options, signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);
        return res.json();
      })
      .then((json: T) => {
        setData(json);
        setIsLoading(false);
      })
      .catch((err) => {
        if (err.name !== "AbortError") {
          setError(err);
          setIsLoading(false);
        }
      });

    return () => controller.abort();
  }, [url, fetchCount]);

  return { data, error, isLoading, refetch };
}
```

### Step 3: Use It

```tsx
function UserProfile({ userId }: { userId: string }) {
  const { data: user, error, isLoading, refetch } = useFetch<User>(
    `https://api.example.com/users/${userId}`,
  );

  if (isLoading) return <Spinner />;
  if (error) return <ErrorBanner message={error.message} onRetry={refetch} />;
  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

> **Tip:** For production use, consider libraries like TanStack Query or SWR that handle caching, deduplication, and background refetching. This hook is useful for understanding the fundamentals.

## 4. useMediaQuery — Responsive Logic in JavaScript

CSS media queries work for styling, but sometimes you need responsive behavior in your component logic.

### Step 1: Implement the Hook

```tsx
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() => {
    if (typeof window === "undefined") return false;
    return window.matchMedia(query).matches;
  });

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);

    const handler = (event: MediaQueryListEvent) => {
      setMatches(event.matches);
    };

    mediaQuery.addEventListener("change", handler);
    setMatches(mediaQuery.matches);

    return () => {
      mediaQuery.removeEventListener("change", handler);
    };
  }, [query]);

  return matches;
}
```

### Step 2: Use It

```tsx
function Navigation() {
  const isMobile = useMediaQuery("(max-width: 768px)");
  const prefersReducedMotion = useMediaQuery("(prefers-reduced-motion: reduce)");
  const isDarkMode = useMediaQuery("(prefers-color-scheme: dark)");

  if (isMobile) {
    return <HamburgerMenu animated={!prefersReducedMotion} />;
  }

  return <DesktopNav />;
}
```

> **Tip:** The SSR check (`typeof window === "undefined"`) prevents errors during server-side rendering. The initial value defaults to `false` and updates on the client after hydration.

## 5. useClickOutside — Close Dropdowns and Modals

This hook detects clicks outside a referenced element, commonly used for closing dropdown menus, modals, and popovers.

### Step 1: Implement the Hook

```tsx
function useClickOutside(ref: RefObject<HTMLElement | null>, handler: () => void) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      const el = ref.current;
      if (!el || el.contains(event.target as Node)) {
        return;
      }
      handler();
    };

    document.addEventListener("mousedown", listener);
    document.addEventListener("touchstart", listener);

    return () => {
      document.removeEventListener("mousedown", listener);
      document.removeEventListener("touchstart", listener);
    };
  }, [ref, handler]);
}
```

### Step 2: Use It

```tsx
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);

  useClickOutside(dropdownRef, () => setIsOpen(false));

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Options</button>
      {isOpen && (
        <ul className="dropdown-menu">
          <li>Edit</li>
          <li>Duplicate</li>
          <li>Delete</li>
        </ul>
      )}
    </div>
  );
}
```

> **Tip:** Wrap the handler in `useCallback` at the call site to prevent unnecessary effect reruns.

## Testing Custom Hooks

Use `@testing-library/react` with `renderHook` to test hooks in isolation:

```tsx
import { renderHook, act } from "@testing-library/react";
import { useLocalStorage } from "./useLocalStorage";

describe("useLocalStorage", () => {
  beforeEach(() => {
    window.localStorage.clear();
  });

  it("returns the initial value when storage is empty", () => {
    const { result } = renderHook(() => useLocalStorage("key", "default"));
    expect(result.current[0]).toBe("default");
  });

  it("updates the value and persists to storage", () => {
    const { result } = renderHook(() => useLocalStorage("key", "default"));

    act(() => {
      result.current[1]("updated");
    });

    expect(result.current[0]).toBe("updated");
    expect(window.localStorage.getItem("key")).toBe('"updated"');
  });

  it("reads existing values from storage", () => {
    window.localStorage.setItem("key", '"existing"');

    const { result } = renderHook(() => useLocalStorage("key", "default"));
    expect(result.current[0]).toBe("existing");
  });
});
```

## Common Mistakes to Avoid

1. **Missing dependency arrays** — omitting dependencies from `useEffect` causes stale closures. Always include every value the effect reads.

2. **Returning new objects on every render** — if your hook returns an object, memoize it with `useMemo` to prevent unnecessary re-renders in consumers.

3. **Not cleaning up** — event listeners, timers, and subscriptions must be removed in the cleanup function. Skipping cleanup causes memory leaks.

4. **Calling hooks conditionally** — hooks must be called in the same order every render. Never put a hook inside an `if` block or loop.

## Summary

| Hook              | Purpose                    | Key APIs Used                    |
|-------------------|----------------------------|----------------------------------|
| `useLocalStorage` | Persistent state           | `useState`, `localStorage`       |
| `useDebounce`     | Delayed updates            | `useState`, `useEffect`          |
| `useFetch`        | Data fetching              | `useState`, `useEffect`, `fetch` |
| `useMediaQuery`   | Responsive JS logic        | `useState`, `useEffect`, `matchMedia` |
| `useClickOutside` | Detect external clicks     | `useEffect`, `useRef`            |

Custom hooks are the primary mechanism for code reuse in modern React. They compose naturally, can be tested independently, and keep your components focused on rendering.

## Further Reading

- [React Docs: Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [React Docs: Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [TanStack Query](https://tanstack.com/query/latest)
- [usehooks.com](https://usehooks.com/) — collection of community hooks
