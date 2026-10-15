---
name: typescript-coding-style
description: >
  Apply personal TypeScript coding standards and architectural preferences
  when writing, reviewing, or refactoring TypeScript code. Use this skill
  whenever the user asks to write TypeScript code, create a Next.js component,
  build an API route, scaffold a project, or review TypeScript for style.
  Also trigger when the user says "write me a TypeScript...", "create a
  component", "create a Next.js page", "scaffold a Next.js project",
  "build an API endpoint", "write a server action", or any TypeScript/React/
  Next.js code generation task, even if they don't explicitly mention style.
  This skill ensures all output matches the user's exact personal preferences
  for formatting, architecture, tooling, and documentation. Trigger for any
  .ts or .tsx file creation, any Next.js work, any React component work,
  or any full-stack TypeScript task.
---

# TypeScript Coding Style: Personal Preferences

This skill encodes the exact TypeScript coding standards and architectural
preferences to apply whenever writing, generating, or reviewing TypeScript
code for this user.

**Always apply this skill when generating any TypeScript code, no exceptions.**

---

## Code Style

### Baseline Rules
- **Language**: TypeScript, never plain JavaScript, no exceptions
- **Standard**: Strict mode, all strict compiler flags enabled
- **Runtime**: Node.js (via Next.js)
- **Package manager**: pnpm, always, never npm or yarn
- **Line length**: 100 characters (Prettier configured)
- **Quotes**: Single quotes, enforced by Prettier
- **Semicolons**: Always, enforced by Prettier
- **Trailing commas**: Always in multi-line, improves diffs

### Type Safety: Maximum Strictness
- Use explicit types on function parameters and return types, always
- Use `unknown` over `any`, if `any` is truly needed, add a comment explaining why
- Use `as const` for literal objects and arrays
- Use discriminated unions over optional fields for state modeling
- Use `readonly` for data that should not mutate
- Use `satisfies` for type-safe object literals with inference
- Use template literal types where they add safety
- Never use `@ts-ignore`, use `@ts-expect-error` with explanation if absolutely needed
```typescript
// CORRECT: explicit types, discriminated union, readonly
type ApiResponse<T> =
  | { readonly status: 'success'; readonly data: T }
  | { readonly status: 'error'; readonly error: string };

function processResponse<T>(response: ApiResponse<T>): T {
  if (response.status === 'error') {
    throw new AppError(response.error);
  }
  return response.data;
}

// WRONG: any, no return type, loose typing
function processResponse(response: any) {
  if (response.status === 'error') {
    throw new Error(response.error);
  }
  return response.data;
}
```

### Enums: Never Use
- Never use TypeScript `enum`, they have runtime quirks and poor tree-shaking
- Use `as const` objects + type extraction instead
```typescript
// CORRECT: const object + derived type
const OrderStatus = {
  PENDING: 'pending',
  CONFIRMED: 'confirmed',
  CANCELLED: 'cancelled',
} as const;

type OrderStatus = (typeof OrderStatus)[keyof typeof OrderStatus];

// WRONG: TypeScript enum
enum OrderStatus {
  PENDING = 'pending',
  CONFIRMED = 'confirmed',
  CANCELLED = 'cancelled',
}
```

### Naming Conventions
- **Files**: kebab-case, `order-service.ts`, `use-auth.ts`
- **Components**: PascalCase file matching component, `OrderCard.tsx`
- **Types/Interfaces**: PascalCase, `OrderItem`, `UserProfile`
- **Functions/variables**: camelCase, `fetchUser`, `orderTotal`
- **Constants**: SCREAMING_SNAKE_CASE, `MAX_RETRIES`, `API_BASE_URL`
- **Zod schemas**: camelCase with `Schema` suffix, `orderSchema`, `userProfileSchema`
- **Boolean variables**: prefix with `is`, `has`, `should`, `can`, `isLoading`, `hasAccess`

### Imports: Organized and Clean
- Use path aliases (`@/`), never deep relative imports like `../../../`
- Group imports: external libs → internal modules → types → styles
- Let ESLint auto-sort, never manually reorder
```typescript
// CORRECT: grouped, aliased
import { notFound } from 'next/navigation';
import { z } from 'zod';

import { db } from '@/lib/db';
import { orderSchema } from '@/lib/schemas/order';
import { AppError } from '@/lib/errors';

import type { Order } from '@/types/order';
```

---

## Project Structure

This is the canonical layout for every Next.js App Router project.
```
project-root/
├── src/
│   ├── app/                        ← Next.js App Router
│   │   ├── layout.tsx              ← Root layout
│   │   ├── page.tsx                ← Home page
│   │   ├── not-found.tsx           ← Global 404
│   │   ├── error.tsx               ← Global error boundary
│   │   ├── loading.tsx             ← Global loading UI
│   │   ├── (auth)/                 ← Route group, auth pages
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── dashboard/
│   │   │   ├── page.tsx
│   │   │   └── layout.tsx
│   │   └── api/                    ← API routes (use sparingly)
│   │       └── webhooks/
│   │           └── route.ts
│   │
│   ├── components/                 ← Shared React components
│   │   ├── ui/                     ← Primitive UI components
│   │   │   ├── Button.tsx
│   │   │   └── Input.tsx
│   │   └── orders/                 ← Feature-scoped components
│   │       ├── OrderCard.tsx
│   │       └── OrderList.tsx
│   │
│   ├── lib/                        ← Core business logic & utilities
│   │   ├── db/                     ← Drizzle setup + schema
│   │   │   ├── index.ts            ← DB client singleton
│   │   │   ├── schema.ts           ← All Drizzle table definitions
│   │   │   └── migrations/         ← Generated migrations
│   │   ├── schemas/                ← Zod validation schemas
│   │   │   ├── order.ts
│   │   │   └── user.ts
│   │   ├── actions/                ← Server Actions
│   │   │   ├── order-actions.ts
│   │   │   └── user-actions.ts
│   │   ├── services/               ← Business logic functions
│   │   │   ├── order-service.ts
│   │   │   └── user-service.ts
│   │   ├── errors.ts               ← Custom error hierarchy
│   │   ├── logger.ts               ← pino setup
│   │   └── utils.ts                ← Pure utility functions
│   │
│   ├── hooks/                      ← Custom React hooks
│   │   └── use-debounce.ts
│   │
│   ├── types/                      ← Shared type definitions
│   │   └── order.ts
│   │
│   └── config/                     ← App configuration
│       └── env.ts                  ← Typed env with Zod
│
├── public/                         ← Static assets
├── drizzle.config.ts               ← Drizzle Kit config
├── next.config.ts                  ← Next.js config
├── tailwind.config.ts              ← Tailwind config
├── tsconfig.json
├── .eslintrc.cjs
├── .prettierrc
├── vitest.config.ts
└── package.json
```

### Key Rules
- `src/app/` is routing only, no business logic in page files
- `src/lib/` is where all domain code lives, services, schemas, actions, DB
- `src/components/` split by `ui/` (primitives) and feature folders
- `src/types/` for shared types only, co-locate types with their module when possible
- Server Actions live in `src/lib/actions/`, never inline in components
- One Drizzle schema file, split only when it gets unmanageable

---

## Server / Client Boundary

This is the most critical architectural decision in App Router. Get it wrong
and you get hydration errors, bundle bloat, or broken patterns.

### Default: Server Components
- Every component is a Server Component unless explicitly marked `'use client'`
- Server Components can directly await data, access DB, read env vars
- Server Components cannot use hooks, browser APIs, or event handlers

### When to Use `'use client'`
- Interactive UI that needs `useState`, `useEffect`, or event handlers
- Components using browser APIs (`window`, `localStorage`, `IntersectionObserver`)
- Third-party libraries that require client-side rendering

### The Pattern: Server Parent, Client Child
```typescript
// src/app/orders/page.tsx: Server Component (default)
import { getOrders } from '@/lib/services/order-service';
import { OrderList } from '@/components/orders/OrderList';

export default async function OrdersPage() {
  const orders = await getOrders();
  return <OrderList orders={orders} />;
}

// src/components/orders/OrderList.tsx: Client Component
'use client';

import { useState } from 'react';
import type { Order } from '@/types/order';

interface OrderListProps {
  readonly orders: readonly Order[];
}

export function OrderList({ orders }: OrderListProps) {
  const [filter, setFilter] = useState('');
  // interactive UI here
}
```

### Rules
- Push `'use client'` as far down the tree as possible
- Never mark a page or layout as `'use client'`, extract interactive parts
- Pass server data down as props, never fetch in client components
- Server Actions are the bridge, call them from client components for mutations

---

## Architecture

### Design Philosophy: Functional-First with Clean Boundaries
- **Pure functions** for business logic, stateless, testable, composable
- **Zod schemas** for all external data, API inputs, form data, env vars
- **Server Actions** for mutations, type-safe, progressively enhanced
- **Services** for complex business logic, plain functions, not classes
- **Repository pattern** via Drizzle, all DB access through `lib/db/`

### Error Handling: Typed Error Hierarchy
```typescript
// src/lib/errors.ts

export class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500,
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} not found: ${id}`, 'NOT_FOUND', 404);
    this.name = 'NotFoundError';
  }
}

export class ValidationError extends AppError {
  constructor(
    message: string,
    public readonly fieldErrors: Record<string, string[]> = {},
  ) {
    super(message, 'VALIDATION_ERROR', 400);
    this.name = 'ValidationError';
  }
}

// Never throw raw Error: always use the hierarchy
// Never swallow errors silently: always log or re-throw
```

### Server Actions: The Mutation Pattern
```typescript
// src/lib/actions/order-actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { z } from 'zod';

import { db } from '@/lib/db';
import { orders } from '@/lib/db/schema';
import { createOrderSchema } from '@/lib/schemas/order';
import { logger } from '@/lib/logger';

type ActionResult<T> =
  | { success: true; data: T }
  | { success: false; error: string };

export async function createOrder(
  input: z.infer<typeof createOrderSchema>,
): Promise<ActionResult<{ id: string }>> {
  const parsed = createOrderSchema.safeParse(input);

  if (!parsed.success) {
    return { success: false, error: parsed.error.message };
  }

  try {
    const [order] = await db
      .insert(orders)
      .values(parsed.data)
      .returning({ id: orders.id });

    revalidatePath('/orders');
    return { success: true, data: { id: order.id } };
  } catch (error) {
    logger.error({ error }, 'Failed to create order');
    return { success: false, error: 'Failed to create order' };
  }
}
```

### Patterns to Always Apply

**Dependency Injection via Function Parameters**
```typescript
// CORRECT: inject dependencies, easy to test
function calculateTotal(
  items: readonly OrderItem[],
  taxRate: number,
): number {
  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  return subtotal * (1 + taxRate);
}

// WRONG: hidden dependency
function calculateTotal(items: readonly OrderItem[]): number {
  const taxRate = getTaxRate(); // where does this come from?
  // ...
}
```

**Exhaustive Switch with never**
```typescript
function getStatusLabel(status: OrderStatus): string {
  switch (status) {
    case 'pending':
      return 'Pending';
    case 'confirmed':
      return 'Confirmed';
    case 'cancelled':
      return 'Cancelled';
    default: {
      const _exhaustive: never = status;
      throw new AppError(`Unknown status: ${_exhaustive}`, 'INVALID_STATUS');
    }
  }
}
```

**Result Type over Thrown Errors for Expected Failures**
```typescript
type Result<T, E = string> =
  | { readonly ok: true; readonly value: T }
  | { readonly ok: false; readonly error: E };

function parseOrderId(raw: string): Result<string> {
  if (!raw.match(/^ord_[a-z0-9]{12}$/)) {
    return { ok: false, error: `Invalid order ID format: ${raw}` };
  }
  return { ok: true, value: raw };
}
```

---

## Data Modeling

### Zod: All External Data

Every piece of data entering the system gets a Zod schema. No exceptions.
```typescript
// src/lib/schemas/order.ts
import { z } from 'zod';

export const orderItemSchema = z.object({
  productId: z.string().min(1),
  quantity: z.number().int().positive(),
  price: z.number().positive(),
});

export const createOrderSchema = z.object({
  userId: z.string().min(1),
  items: z.array(orderItemSchema).min(1),
  notes: z.string().max(500).optional(),
});

// Infer types from schemas: single source of truth
export type OrderItem = z.infer<typeof orderItemSchema>;
export type CreateOrderInput = z.infer<typeof createOrderSchema>;
```

### Drizzle: Database Schema
```typescript
// src/lib/db/schema.ts
import { pgTable, text, integer, timestamp, uuid } from 'drizzle-orm/pg-core';

export const orders = pgTable('orders', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: text('user_id').notNull(),
  status: text('status', {
    enum: ['pending', 'confirmed', 'cancelled'],
  }).notNull().default('pending'),
  totalCents: integer('total_cents').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// Infer types from Drizzle schema
export type Order = typeof orders.$inferSelect;
export type NewOrder = typeof orders.$inferInsert;
```
```typescript
// src/lib/db/index.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

import { env } from '@/config/env';
import * as schema from './schema';

const pool = new Pool({ connectionString: env.DATABASE_URL });
export const db = drizzle(pool, { schema });
```

### Environment Variables: Zod Validated
```typescript
// src/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  DATABASE_URL: z.string().url(),
  NEXT_PUBLIC_APP_URL: z.string().url(),
  API_SECRET: z.string().min(32),
});

export const env = envSchema.parse(process.env);
```

---

## Logging: pino
```typescript
// src/lib/logger.ts
import pino from 'pino';

import { env } from '@/config/env';

export const logger = pino({
  level: env.NODE_ENV === 'production' ? 'info' : 'debug',
  transport:
    env.NODE_ENV === 'development'
      ? { target: 'pino-pretty', options: { colorize: true } }
      : undefined,
});

// Usage: structured context, never string interpolation
logger.info({ orderId, userId }, 'Order created successfully');
logger.error({ error, orderId }, 'Failed to process order');
```

---

## Comments

- **Minimal**: clean, well-named code with TypeScript types is self-documenting
- Add inline comments **only** for:
  - Non-obvious algorithmic logic
  - Workarounds for library bugs or Next.js quirks
  - Business rule rationale that can't be expressed in types
- **Never** comment what the code does, comment **why** when it's not obvious
- Use TSDoc (`/** */`) on exported functions in `lib/`, not on components
```typescript
// CORRECT: explains the why
// Next.js App Router caches fetch by default: opt out for real-time data
const data = await fetch(url, { cache: 'no-store' });

// WRONG: restates the code
// Fetch the data from the URL
const data = await fetch(url);
```

---

## Testing: Vitest

- Every service and utility module gets a corresponding `.test.ts`
- Test **behavior**, not implementation
- Use Vitest's `describe` / `it` / `expect`
- Mock at the boundary, DB, HTTP, external APIs
- Use `vi.mock()` for module mocking
- Do not unit test React components by default, test server logic and utils
```typescript
// src/lib/services/__tests__/order-service.test.ts
import { describe, it, expect, vi } from 'vitest';

import { calculateTotal } from '../order-service';

describe('calculateTotal', () => {
  it('sums item prices with tax', () => {
    const items = [
      { productId: 'p1', quantity: 2, price: 10.0 },
      { productId: 'p2', quantity: 1, price: 25.0 },
    ] as const;

    const result = calculateTotal(items, 0.19);

    expect(result).toBeCloseTo(53.55);
  });

  it('returns zero for empty items', () => {
    expect(calculateTotal([], 0.19)).toBe(0);
  });
});
```
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import path from 'path';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['src/**/*.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

---

## Toolchain

Standard setup for every project:

| Tool | Purpose | Config file |
|---|---|---|
| `eslint` | Linting | `.eslintrc.cjs` |
| `prettier` | Formatting | `.prettierrc` |
| `typescript` | Type checking, strict | `tsconfig.json` |
| `vitest` | Testing | `vitest.config.ts` |
| `pino` | Structured logging | `src/lib/logger.ts` |
| `zod` | Runtime validation | Throughout |
| `drizzle-orm` | Database ORM | `drizzle.config.ts` |
| `lint-staged` + `husky` | Pre-commit enforcement | `package.json` |

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "incremental": true,
    "paths": {
      "@/*": ["./src/*"]
    },
    "plugins": [{ "name": "next" }]
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### .prettierrc
```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always",
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

### .eslintrc.cjs
```javascript
/** @type {import('eslint').Linter.Config} */
module.exports = {
  root: true,
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/strict-type-checked',
    'plugin:@typescript-eslint/stylistic-type-checked',
    'next/core-web-vitals',
    'prettier',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: true,
    tsconfigRootDir: __dirname,
  },
  rules: {
    '@typescript-eslint/no-unused-vars': [
      'error',
      { argsIgnorePattern: '^_', varsIgnorePattern: '^_' },
    ],
    '@typescript-eslint/consistent-type-imports': [
      'error',
      { prefer: 'type-imports' },
    ],
    '@typescript-eslint/no-misused-promises': [
      'error',
      { checksVoidReturn: { attributes: false } },
    ],
  },
};
```

### package.json (relevant scripts + lint-staged)
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint . --ext .ts,.tsx",
    "format": "prettier --write .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:studio": "drizzle-kit studio",
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css}": ["prettier --write"]
  }
}
```

### drizzle.config.ts
```typescript
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/lib/db/schema.ts',
  out: './src/lib/db/migrations',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```
