# A Comprehensive Guide to TypeScript Generics

TypeScript generics provide a way to create reusable components that work with a variety of types rather than a single one. They are one of the most powerful features of the type system.

## Why Generics?

Without generics, you would need to either use `any` (losing type safety) or create duplicate functions for each type:

```typescript
function identityString(arg: string): string {
  return arg;
}

function identityNumber(arg: number): number {
  return arg;
}
```

With generics, you write a single function:

```typescript
function identity<T>(arg: T): T {
  return arg;
}

const str = identity<string>("hello"); // type: string
const num = identity<number>(42);      // type: number
```

## Generic Constraints

You can restrict what types are allowed using the `extends` keyword:

```typescript
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello");       // OK, string has .length
logLength([1, 2, 3]);     // OK, array has .length
// logLength(42);          // Error: number has no .length
```

## Generic Interfaces

Generics can be applied to interfaces to create flexible data structures:

```typescript
interface Repository<T> {
  getById(id: string): Promise<T>;
  getAll(): Promise<T[]>;
  create(item: T): Promise<T>;
  update(id: string, item: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

interface User {
  id: string;
  name: string;
  email: string;
}

class UserRepository implements Repository<User> {
  private users: Map<string, User> = new Map();

  async getById(id: string): Promise<User> {
    const user = this.users.get(id);
    if (!user) throw new Error(`User ${id} not found`);
    return user;
  }

  async getAll(): Promise<User[]> {
    return Array.from(this.users.values());
  }

  async create(item: User): Promise<User> {
    this.users.set(item.id, item);
    return item;
  }

  async update(id: string, item: Partial<User>): Promise<User> {
    const existing = await this.getById(id);
    const updated = { ...existing, ...item };
    this.users.set(id, updated);
    return updated;
  }

  async delete(id: string): Promise<void> {
    this.users.delete(id);
  }
}
```

## Generic Classes

Classes can also use type parameters:

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  get size(): number {
    return this.items.length;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2
```

## Mapped Types with Generics

Generics combine with mapped types for powerful transformations:

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};

type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

interface Config {
  host: string;
  port: number;
  debug: boolean;
}

type ReadonlyConfig = Readonly<Config>;
type OptionalConfig = Optional<Config>;
```

## Conditional Types

Advanced generic patterns use conditional types:

```typescript
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>;  // "yes"
type B = IsString<number>;  // "no"

type ExtractPromise<T> = T extends Promise<infer U> ? U : T;

type C = ExtractPromise<Promise<string>>; // string
type D = ExtractPromise<number>;          // number
```

## Practical Example: API Client

Here is a real-world example combining several generic patterns:

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
  timestamp: string;
}

interface PaginatedResponse<T> extends ApiResponse<T[]> {
  total: number;
  page: number;
  pageSize: number;
  hasNext: boolean;
}

class ApiClient {
  constructor(private baseUrl: string) {}

  async get<T>(path: string): Promise<ApiResponse<T>> {
    const response = await fetch(`${this.baseUrl}${path}`);
    return response.json();
  }

  async getPaginated<T>(
    path: string,
    page: number = 1,
    pageSize: number = 20,
  ): Promise<PaginatedResponse<T>> {
    const response = await fetch(
      `${this.baseUrl}${path}?page=${page}&pageSize=${pageSize}`,
    );
    return response.json();
  }
}

const client = new ApiClient("https://api.example.com");
const users = await client.getPaginated<User>("/users", 1, 10);
console.log(users.data); // User[]
console.log(users.hasNext); // boolean
```

## Summary

| Feature | Syntax | Use Case |
|---------|--------|----------|
| Generic Function | `function fn<T>(arg: T): T` | Reusable functions |
| Generic Interface | `interface Repo<T>` | Flexible contracts |
| Generic Class | `class Stack<T>` | Type-safe data structures |
| Constraints | `<T extends Base>` | Restrict allowed types |
| Conditional | `T extends U ? X : Y` | Type-level branching |
| Mapped | `{ [P in keyof T]: ... }` | Transform existing types |

Generics are essential for writing type-safe, reusable TypeScript code. They eliminate the need for `any` while keeping your code DRY.

## Further Reading

- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TypeScript Handbook: Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [TypeScript Handbook: Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
