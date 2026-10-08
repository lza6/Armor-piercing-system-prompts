# Building a Type-Safe Event Emitter in TypeScript

A deep dive into creating a fully type-safe event emitter that provides autocomplete and compile-time checks for event names and payload types.

## The Problem

Standard event emitters in Node.js and the browser lack type safety:

```typescript
import { EventEmitter } from 'events';

const emitter = new EventEmitter();

// No type checking on event name or payload
emitter.emit('user:created', { id: 1, name: 'Alice' });
emitter.on('user:craeted', (data) => { // typo goes unnoticed
  console.log(data); // data is 'any'
});
```

## Type-Safe Implementation

### Step 1: Define the Event Map

```typescript
type EventMap = Record<string, unknown>;

type EventKey<T extends EventMap> = string & keyof T;
type EventCallback<T> = (payload: T) => void;
```

### Step 2: Build the Emitter Class

```typescript
interface TypedEventEmitter<T extends EventMap> {
  on<K extends EventKey<T>>(event: K, callback: EventCallback<T[K]>): this;
  off<K extends EventKey<T>>(event: K, callback: EventCallback<T[K]>): this;
  emit<K extends EventKey<T>>(event: K, payload: T[K]): boolean;
  once<K extends EventKey<T>>(event: K, callback: EventCallback<T[K]>): this;
  listenerCount<K extends EventKey<T>>(event: K): number;
  removeAllListeners<K extends EventKey<T>>(event?: K): this;
}

class SafeEventEmitter<T extends EventMap> implements TypedEventEmitter<T> {
  private listeners = new Map<string, Set<EventCallback<unknown>>>();
  private onceListeners = new WeakSet<EventCallback<unknown>>();

  on<K extends EventKey<T>>(event: K, callback: EventCallback<T[K]>): this {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback as EventCallback<unknown>);
    return this;
  }

  off<K extends EventKey<T>>(event: K, callback: EventCallback<T[K]>): this {
    const set = this.listeners.get(event);
    if (set) {
      set.delete(callback as EventCallback<unknown>);
      if (set.size === 0) this.listeners.delete(event);
    }
    return this;
  }

  emit<K extends EventKey<T>>(event: K, payload: T[K]): boolean {
    const set = this.listeners.get(event);
    if (!set || set.size === 0) return false;

    for (const callback of set) {
      callback(payload);
      if (this.onceListeners.has(callback)) {
        set.delete(callback);
        this.onceListeners.delete(callback);
      }
    }
    return true;
  }

  once<K extends EventKey<T>>(event: K, callback: EventCallback<T[K]>): this {
    this.onceListeners.add(callback as EventCallback<unknown>);
    return this.on(event, callback);
  }

  listenerCount<K extends EventKey<T>>(event: K): number {
    return this.listeners.get(event)?.size ?? 0;
  }

  removeAllListeners<K extends EventKey<T>>(event?: K): this {
    if (event) {
      this.listeners.delete(event);
    } else {
      this.listeners.clear();
    }
    return this;
  }
}
```

### Step 3: Usage with Full Type Safety

```typescript
// Define your event map
interface AppEvents {
  'user:created': { id: string; name: string; email: string };
  'user:deleted': { id: string; reason?: string };
  'order:placed': { orderId: string; items: string[]; total: number };
  'order:shipped': { orderId: string; trackingNumber: string };
  'system:error': { code: number; message: string; stack?: string };
  'system:ready': void;
}

const bus = new SafeEventEmitter<AppEvents>();

// Full autocomplete on event names
bus.on('user:created', (payload) => {
  // payload is typed as { id: string; name: string; email: string }
  console.log(`New user: ${payload.name} (${payload.email})`);
});

bus.on('order:placed', (payload) => {
  // payload is typed as { orderId: string; items: string[]; total: number }
  console.log(`Order ${payload.orderId}: $${payload.total}`);
});

// Compile error: 'user:craeted' is not a valid event name
// bus.on('user:craeted', () => {});

// Compile error: payload type mismatch
// bus.emit('user:created', { id: '1' }); // missing name, email

// Correct usage
bus.emit('user:created', {
  id: '1',
  name: 'Alice',
  email: 'alice@example.com',
});
```

## Advanced Patterns

### Wildcard Events

```typescript
type WildcardEventMap<T extends EventMap> = T & {
  '*': { event: keyof T; payload: T[keyof T] };
};

class WildcardEmitter<T extends EventMap> extends SafeEventEmitter<
  WildcardEventMap<T>
> {
  emit<K extends EventKey<WildcardEventMap<T>>>(
    event: K,
    payload: WildcardEventMap<T>[K],
  ): boolean {
    const result = super.emit(event, payload);
    if (event !== '*') {
      super.emit('*' as any, { event, payload } as any);
    }
    return result;
  }
}
```

### Async Event Handlers

```typescript
type AsyncEventCallback<T> = (payload: T) => Promise<void> | void;

class AsyncEventEmitter<T extends EventMap> {
  private listeners = new Map<string, Set<AsyncEventCallback<unknown>>>();

  on<K extends EventKey<T>>(
    event: K,
    callback: AsyncEventCallback<T[K]>,
  ): this {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback as AsyncEventCallback<unknown>);
    return this;
  }

  async emit<K extends EventKey<T>>(
    event: K,
    payload: T[K],
  ): Promise<boolean> {
    const set = this.listeners.get(event);
    if (!set || set.size === 0) return false;

    const promises = Array.from(set).map((cb) => cb(payload));
    await Promise.all(promises);
    return true;
  }
}
```

### Namespaced Events

```typescript
type NamespacedEvents = {
  [K in `${'user' | 'order' | 'system'}:${string}`]: unknown;
};

function createNamespace<
  Prefix extends string,
  Events extends Record<string, unknown>,
>(prefix: Prefix, emitter: SafeEventEmitter<any>) {
  return {
    on<K extends keyof Events & string>(
      event: K,
      cb: EventCallback<Events[K]>,
    ) {
      emitter.on(`${prefix}:${event}` as any, cb as any);
    },
    emit<K extends keyof Events & string>(event: K, payload: Events[K]) {
      emitter.emit(`${prefix}:${event}` as any, payload as any);
    },
  };
}
```

## Testing

```typescript
import { describe, it, expect, vi } from 'vitest';

describe('SafeEventEmitter', () => {
  it('should call registered listeners', () => {
    const emitter = new SafeEventEmitter<AppEvents>();
    const handler = vi.fn();

    emitter.on('user:created', handler);
    emitter.emit('user:created', {
      id: '1',
      name: 'Alice',
      email: 'alice@example.com',
    });

    expect(handler).toHaveBeenCalledWith({
      id: '1',
      name: 'Alice',
      email: 'alice@example.com',
    });
  });

  it('should handle once listeners', () => {
    const emitter = new SafeEventEmitter<AppEvents>();
    const handler = vi.fn();

    emitter.once('user:deleted', handler);
    emitter.emit('user:deleted', { id: '1' });
    emitter.emit('user:deleted', { id: '2' });

    expect(handler).toHaveBeenCalledTimes(1);
  });

  it('should remove listeners', () => {
    const emitter = new SafeEventEmitter<AppEvents>();
    const handler = vi.fn();

    emitter.on('system:error', handler);
    emitter.off('system:error', handler);
    emitter.emit('system:error', { code: 500, message: 'fail' });

    expect(handler).not.toHaveBeenCalled();
  });

  it('should report listener count', () => {
    const emitter = new SafeEventEmitter<AppEvents>();
    emitter.on('user:created', () => {});
    emitter.on('user:created', () => {});

    expect(emitter.listenerCount('user:created')).toBe(2);
  });
});
```

## Performance Considerations

| Operation | Map + Set Implementation | Array Implementation |
|-----------|-------------------------|---------------------|
| Add listener | O(1) | O(1) |
| Remove listener | O(1) | O(n) |
| Emit (n listeners) | O(n) | O(n) |
| Memory per event | ~100 bytes overhead | ~50 bytes overhead |

The `Map` + `Set` approach is preferred when listeners are frequently added and removed. The array approach uses less memory for static listener sets.

## Summary

- Use a generic `EventMap` type parameter to enforce event name and payload types at compile time
- `WeakSet` for tracking once-listeners avoids memory leaks
- The pattern integrates with existing Node.js `EventEmitter` patterns
- Wildcard and async variants extend the base pattern for specific use cases

## References

- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [Node.js EventEmitter](https://nodejs.org/api/events.html)
- [TypeScript Playground](https://www.typescriptlang.org/play)
