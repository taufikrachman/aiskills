# TypeScript Strict Mode

You are an expert TypeScript developer. Follow these rules for ALL TypeScript code you generate. Default to the strictest possible configuration.

## tsconfig.json Baseline
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

## Rules

### 1. NEVER use `any`
- Use `unknown` for truly unknown types, narrow with type guards before use.
- Use generics when the type depends on input: `function identity<T>(x: T): T`.
- Use `Record<string, unknown>` for arbitrary objects (better than `any`).

### 2. Discriminated unions over optional fields
```ts
// ❌ Bad: checking optional fields
type Response = { data?: Data; error?: string };

// ✅ Good: discriminated union
type Response = 
  | { status: 'ok'; data: Data }
  | { status: 'error'; error: string };
```

### 3. Narrow types before use
```ts
// ❌ Bad: type assertion
const user = data as User;

// ✅ Good: type guard
function isUser(data: unknown): data is User {
  return typeof data === 'object' && data !== null && 'id' in data && 'email' in data;
}
if (!isUser(data)) throw new Error('Invalid user');
const user = data; // User
```

### 4. Utility types over manual definition
Prefer built-in utilities: `Partial<T>`, `Required<T>`, `Pick<T,K>`, `Omit<T,K>`, `Readonly<T>`, `Record<K,V>`, `NonNullable<T>`, `ReturnType<T>`, `Parameters<T>`.

### 5. Exhaustive switch checks
```ts
type Action = 'create' | 'update' | 'delete';
function handle(action: Action) {
  switch (action) {
    case 'create': return createItem();
    case 'update': return updateItem();
    case 'delete': return deleteItem();
    default: {
      const _exhaustive: never = action; // Compile error if new action added
      return _exhaustive;
    }
  }
}
```

### 6. Zod for runtime validation
Always validate external data (API responses, user input) with Zod:
```ts
import { z } from 'zod';
const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(['admin', 'user']),
});
type User = z.infer<typeof UserSchema>; // Infer type from schema
```

### 7. Type imports and type-only exports
```ts
import type { User } from './types';  // Erased at compile time
export type { UserProps };             // Type-only export
```

### 8. Async error handling
```ts
// ❌ Promise<unknown>
async function fetchData(): Promise<unknown> { ... }

// ✅ Typed result type
type Result<T> = { success: true; data: T } | { success: false; error: string };
async function fetchData(): Promise<Result<User>> { ... }
```

### 9. Function overloads for complex signatures
```ts
function getUser(id: string): Promise<User>;
function getUser(ids: string[]): Promise<User[]>;
function getUser(idOrIds: string | string[]): Promise<User | User[]> { ... }
```

### 10. Branded types for domain safety
```ts
type UserId = string & { __brand: 'UserId' };
type Email = string & { __brand: 'Email' };

function createUser(id: UserId, email: Email) { ... }
// createUser('abc', 'test@test.com'); // ❌ Type error!
createUser('abc' as UserId, 'test@test.com' as Email); // ✅
```
