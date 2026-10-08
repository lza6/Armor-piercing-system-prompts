# zod

TypeScript-first schema validation with static type inference.

[![npm](https://img.shields.io/npm/v/zod)](https://www.npmjs.com/package/zod)
[![GitHub stars](https://img.shields.io/github/stars/colinhacks/zod)](https://github.com/colinhacks/zod)

## Introduction

Zod is a TypeScript-first schema declaration and validation library. The term "schema" broadly refers to any data type, from a simple string to a complex nested object.

Zod is designed to be as developer-friendly as possible. The goal is to eliminate duplicative type declarations. With Zod, you declare a validator once and Zod will automatically infer the static TypeScript type.

## Installation

```bash
npm install zod
```

Requirements:
- TypeScript 4.5+
- `strict` mode enabled in `tsconfig.json`

## Basic Usage

### Primitives

```typescript
import { z } from 'zod';

const stringSchema = z.string();
const numberSchema = z.number();
const booleanSchema = z.boolean();
const dateSchema = z.date();
const undefinedSchema = z.undefined();
const nullSchema = z.null();
const bigintSchema = z.bigint();
const symbolSchema = z.symbol();

stringSchema.parse("hello"); // "hello"
stringSchema.parse(123);     // throws ZodError
```

### Objects

```typescript
const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'moderator']),
  metadata: z.record(z.string(), z.unknown()).optional(),
});

type User = z.infer<typeof UserSchema>;
// { name: string; email: string; age?: number; role: 'admin' | 'user' | 'moderator'; metadata?: Record<string, unknown> }

const user = UserSchema.parse({
  name: "Alice",
  email: "alice@example.com",
  role: "admin",
});
```

### Arrays and Tuples

```typescript
const StringArraySchema = z.array(z.string());
StringArraySchema.parse(["a", "b"]); // OK
StringArraySchema.parse([1, 2]);     // throws

const TupleSchema = z.tuple([
  z.string(),
  z.number(),
  z.boolean(),
]);
type Coord = z.infer<typeof TupleSchema>; // [string, number, boolean]
```

### Unions and Discriminated Unions

```typescript
const ResultSchema = z.discriminatedUnion('status', [
  z.object({ status: z.literal('success'), data: z.unknown() }),
  z.object({ status: z.literal('error'), message: z.string() }),
]);

type Result = z.infer<typeof ResultSchema>;
// { status: 'success'; data: unknown } | { status: 'error'; message: string }
```

## String Validations

```typescript
z.string().max(5);
z.string().min(1);
z.string().length(5);
z.string().email();
z.string().url();
z.string().emoji();
z.string().uuid();
z.string().cuid();
z.string().cuid2();
z.string().ulid();
z.string().regex(/^[a-z]+$/);
z.string().includes("needle");
z.string().startsWith("prefix");
z.string().endsWith("suffix");
z.string().datetime();
z.string().ip();
z.string().trim();
z.string().toLowerCase();
z.string().toUpperCase();
```

## Number Validations

```typescript
z.number().gt(5);        // > 5
z.number().gte(5);       // >= 5
z.number().lt(5);        // < 5
z.number().lte(5);       // <= 5
z.number().int();        // integer
z.number().positive();   // > 0
z.number().nonnegative(); // >= 0
z.number().negative();   // < 0
z.number().nonpositive(); // <= 0
z.number().multipleOf(5); // divisible by 5
z.number().finite();     // not Infinity
z.number().safe();       // Number.MIN_SAFE_INTEGER to MAX_SAFE_INTEGER
```

## Transform and Preprocess

```typescript
const CastedNumber = z.string().transform((val) => parseInt(val, 10));
CastedNumber.parse("42"); // 42

const DateFromString = z.string().pipe(z.coerce.date());
DateFromString.parse("2024-01-01"); // Date object

const TrimmedLowercase = z.string().trim().toLowerCase();
TrimmedLowercase.parse("  HELLO  "); // "hello"
```

## Error Handling

```typescript
const result = UserSchema.safeParse({ name: "", email: "bad" });

if (!result.success) {
  console.log(result.error.issues);
  // [
  //   { code: 'too_small', minimum: 1, path: ['name'], message: '...' },
  //   { code: 'invalid_string', validation: 'email', path: ['email'], message: '...' },
  //   { code: 'invalid_type', expected: 'string', path: ['role'], message: '...' },
  // ]

  console.log(result.error.flatten());
  // {
  //   formErrors: [],
  //   fieldErrors: {
  //     name: ['String must contain at least 1 character(s)'],
  //     email: ['Invalid email'],
  //     role: ['Required'],
  //   },
  // }
}
```

## API Summary

| Method | Description |
|--------|-------------|
| `.parse(data)` | Parse data, throw on failure |
| `.safeParse(data)` | Parse data, return result object |
| `.parseAsync(data)` | Async parse (for async refinements) |
| `.optional()` | Make field optional |
| `.nullable()` | Allow null |
| `.default(value)` | Set default value |
| `.transform(fn)` | Transform parsed value |
| `.refine(fn, msg)` | Custom validation |
| `.pipe(schema)` | Chain schemas |
| `z.infer<typeof schema>` | Extract TypeScript type |

## Comparison with Alternatives

| Feature | Zod | Yup | io-ts | Joi |
|---------|-----|-----|-------|-----|
| TypeScript-first | Yes | Partial | Yes | No |
| Static type inference | Yes | Limited | Yes | No |
| Zero dependencies | Yes | No | No | No |
| Async validation | Yes | Yes | Yes | Yes |
| Bundle size (min+gz) | 13KB | 17KB | 8KB | 35KB |

## Links

- [Documentation](https://zod.dev)
- [GitHub](https://github.com/colinhacks/zod)
- [npm](https://www.npmjs.com/package/zod)
- [Discord](https://discord.gg/zod)
