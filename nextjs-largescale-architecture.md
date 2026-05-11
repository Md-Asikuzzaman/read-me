# Next.js Large-Scale Application Architecture

A practical, battle-tested folder structure for enterprise Next.js applications. Built around the App Router, TypeScript, and a clean separation of concerns.

---

## Table of Contents

- [Folder Structure](#folder-structure)
- [Layer Responsibilities](#layer-responsibilities)
- [Service Layer](#service-layer)
- [API Routes](#api-routes)
- [Components](#components)
- [State Management](#state-management)
- [Hooks](#hooks)
- [Lib vs Utils](#lib-vs-utils)
- [Config & Env](#config--env)
- [Types](#types)
- [File Naming Conventions](#file-naming-conventions)
- [Import Rules](#import-rules)
- [When to Add a Layer](#when-to-add-a-layer)

---

## Folder Structure

```
src/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Route group — auth pages
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/              # Route group — protected pages
│   │   ├── layout.tsx
│   │   ├── products/
│   │   │   ├── page.tsx
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx
│   │   │   └── loading.tsx
│   │   ├── orders/
│   │   │   └── page.tsx
│   │   └── settings/
│   │       └── page.tsx
│   ├── api/                      # API Route Handlers
│   │   ├── products/
│   │   │   ├── route.ts          # GET /api/products, POST /api/products
│   │   │   └── [id]/
│   │   │       └── route.ts      # GET, PATCH, DELETE /api/products/:id
│   │   ├── orders/
│   │   │   └── route.ts
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.ts
│   ├── layout.tsx                # Root layout
│   ├── page.tsx                  # Home page
│   ├── error.tsx                 # Global error boundary
│   └── not-found.tsx
│
├── services/                     # Business logic layer
│   ├── product/
│   │   ├── product.service.ts
│   │   ├── product.schema.ts
│   │   ├── product.types.ts
│   │   ├── product.constant.ts
│   │   ├── product.mapper.ts
│   │   └── index.ts
│   ├── order/
│   │   ├── order.service.ts
│   │   ├── order.schema.ts
│   │   ├── order.types.ts
│   │   ├── order.constant.ts
│   │   ├── order.mapper.ts
│   │   └── index.ts
│   ├── user/
│   │   ├── user.service.ts
│   │   ├── user.schema.ts
│   │   ├── user.types.ts
│   │   └── index.ts
│   └── notification/
│       ├── notification.service.ts
│       └── index.ts
│
├── components/                   # UI Components
│   ├── ui/                       # Base design system (shadcn, custom primitives)
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   ├── Table.tsx
│   │   └── index.ts
│   ├── features/                 # Domain-specific components
│   │   ├── product/
│   │   │   ├── ProductCard.tsx
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductForm.tsx
│   │   │   └── index.ts
│   │   ├── order/
│   │   │   ├── OrderSummary.tsx
│   │   │   ├── OrderTable.tsx
│   │   │   └── index.ts
│   │   └── auth/
│   │       ├── LoginForm.tsx
│   │       └── index.ts
│   └── layouts/                  # Shared layout components
│       ├── Navbar.tsx
│       ├── Sidebar.tsx
│       └── Footer.tsx
│
├── hooks/                        # React hooks (client-side logic)
│   ├── useProducts.ts
│   ├── useOrders.ts
│   ├── useAuth.ts
│   └── useDebounce.ts
│
├── store/                        # Global client state (Zustand / Jotai)
│   ├── cart.store.ts
│   ├── ui.store.ts
│   └── index.ts
│
├── lib/                          # Framework wiring & third-party clients
│   ├── db.ts                     # Prisma / Drizzle client
│   ├── auth.ts                   # NextAuth config
│   ├── api-client.ts             # Base fetch wrapper for external APIs
│   ├── email.ts                  # Resend / Nodemailer
│   ├── queue.ts                  # BullMQ / Inngest
│   └── cache.ts                  # Redis client
│
├── utils/                        # Pure functions, no side effects
│   ├── format.ts                 # formatCurrency, formatDate etc.
│   ├── cn.ts                     # classnames helper
│   ├── pagination.ts
│   └── string.ts
│
├── config/                       # App configuration
│   ├── env.ts                    # Zod-parsed env variables
│   ├── constants.ts              # App-wide constants
│   └── routes.ts                 # Typed route paths
│
├── types/                        # Shared global types only
│   ├── api.types.ts              # Shared API response shapes
│   ├── next-auth.d.ts            # NextAuth type augmentation
│   └── global.d.ts
│
└── middleware.ts                 # Next.js middleware (auth guards, redirects)
```

---

## Layer Responsibilities

Each layer has exactly one job. Nothing bleeds into another.

| Layer | Responsibility | Can import from |
|---|---|---|
| `app/` | Routing, layouts, page composition | `services`, `components`, `hooks`, `store` |
| `app/api/` | HTTP request/response only | `services`, `lib` |
| `services/` | Business logic, validation | `lib`, `utils`, `config` |
| `components/` | UI rendering | `hooks`, `store`, `utils`, `config` |
| `hooks/` | React state, data fetching | `services` (via fetch), `store`, `utils` |
| `store/` | Global client state | `types`, `utils` |
| `lib/` | Third-party client setup | `config` |
| `utils/` | Pure helpers | Nothing |
| `config/` | Env and constants | Nothing |

---

## Service Layer

The core of your application. Every domain gets its own folder.

### File breakdown

```
services/product/
 ├── product.service.ts    → business logic, orchestration
 ├── product.schema.ts     → Zod schemas + inferred types
 ├── product.types.ts      → TS interfaces, domain models
 ├── product.constant.ts   → enums, error strings, config values
 ├── product.mapper.ts     → raw API/DB response → domain type
 └── index.ts              → public barrel — only expose what's needed
```

### product.types.ts

```ts
export type Product = {
  id: string
  name: string
  price: number
  category: 'electronics' | 'clothing' | 'food'
  inStock: boolean
  createdAt: Date
}

export type CreateProductDTO = Omit<Product, 'id' | 'createdAt'>
export type UpdateProductDTO = Partial<CreateProductDTO>
```

### product.schema.ts

```ts
import { z } from 'zod'

export const createProductSchema = z.object({
  name: z.string().min(2).max(100),
  price: z.number().positive(),
  category: z.enum(['electronics', 'clothing', 'food']),
  inStock: z.boolean(),
})

export const updateProductSchema = createProductSchema.partial()

export type CreateProductInput = z.infer<typeof createProductSchema>
export type UpdateProductInput = z.infer<typeof updateProductSchema>
```

### product.constant.ts

```ts
export const PRODUCT_ERRORS = {
  NOT_FOUND: 'Product not found',
  CREATE_FAILED: 'Failed to create product',
  UPDATE_FAILED: 'Failed to update product',
  DELETE_FAILED: 'Failed to delete product',
} as const

export const PRODUCT_LIMITS = {
  MAX_NAME_LENGTH: 100,
  DEFAULT_PAGE_SIZE: 20,
} as const
```

### product.mapper.ts

```ts
import { Product } from './product.types'

type RawProduct = {
  id: string
  product_name: string   // API returns snake_case
  product_price: number
  is_available: boolean
  created_date: string
}

export const productMapper = {
  toDomain: (raw: RawProduct): Product => ({
    id: raw.id,
    name: raw.product_name,
    price: raw.product_price,
    category: 'electronics',       // map from raw.category_id lookup
    inStock: raw.is_available,
    createdAt: new Date(raw.created_date),
  }),

  toDomainList: (raws: RawProduct[]): Product[] =>
    raws.map(productMapper.toDomain),
}
```

### product.service.ts

```ts
import { apiClient } from '@/lib/api-client'
import { productMapper } from './product.mapper'
import { createProductSchema, updateProductSchema } from './product.schema'
import { PRODUCT_ERRORS } from './product.constant'
import { CreateProductDTO, UpdateProductDTO } from './product.types'

export const productService = {

  getById: async (id: string) => {
    const res = await apiClient.get(`/products/${id}`)
    if (res.status === 404) throw new Error(PRODUCT_ERRORS.NOT_FOUND)
    if (!res.ok) throw new Error(PRODUCT_ERRORS.CREATE_FAILED)
    return productMapper.toDomain(await res.json())
  },

  getAll: async (filters?: { category?: string; page?: number }) => {
    const params = new URLSearchParams(filters as any)
    const res = await apiClient.get(`/products?${params}`)
    if (!res.ok) throw new Error(PRODUCT_ERRORS.CREATE_FAILED)
    const data = await res.json()
    return productMapper.toDomainList(data.items)
  },

  create: async (input: CreateProductDTO) => {
    const data = createProductSchema.parse(input)    // validate first
    const res = await apiClient.post('/products', data)
    if (!res.ok) throw new Error(PRODUCT_ERRORS.CREATE_FAILED)
    return productMapper.toDomain(await res.json())
  },

  update: async (id: string, input: UpdateProductDTO) => {
    const data = updateProductSchema.parse(input)
    const res = await apiClient.patch(`/products/${id}`, data)
    if (res.status === 404) throw new Error(PRODUCT_ERRORS.NOT_FOUND)
    if (!res.ok) throw new Error(PRODUCT_ERRORS.UPDATE_FAILED)
    return productMapper.toDomain(await res.json())
  },

  delete: async (id: string) => {
    const res = await apiClient.delete(`/products/${id}`)
    if (res.status === 404) throw new Error(PRODUCT_ERRORS.NOT_FOUND)
    if (!res.ok) throw new Error(PRODUCT_ERRORS.DELETE_FAILED)
  },

}
```

### index.ts

```ts
// Only expose the public surface. Nothing internal leaks out.
export { productService } from './product.service'
export type { Product, CreateProductDTO, UpdateProductDTO } from './product.types'
export { createProductSchema, updateProductSchema } from './product.schema'
export { PRODUCT_ERRORS, PRODUCT_CATEGORIES } from './product.constant'
```

---

## API Routes

Route handlers are thin. They validate the request and delegate to the service. No business logic lives here.

```ts
// app/api/products/route.ts

import { NextRequest, NextResponse } from 'next/server'
import { productService, createProductSchema } from '@/services/product'

export async function GET(req: NextRequest) {
  try {
    const { searchParams } = new URL(req.url)
    const category = searchParams.get('category') ?? undefined

    const products = await productService.getAll({ category })
    return NextResponse.json({ data: products })
  } catch (error) {
    return NextResponse.json({ error: 'Failed to fetch products' }, { status: 500 })
  }
}

export async function POST(req: NextRequest) {
  try {
    const body = await req.json()
    const input = createProductSchema.parse(body)   // validate at the boundary
    const product = await productService.create(input)
    return NextResponse.json({ data: product }, { status: 201 })
  } catch (error: any) {
    if (error.name === 'ZodError') {
      return NextResponse.json({ error: error.errors }, { status: 400 })
    }
    return NextResponse.json({ error: 'Failed to create product' }, { status: 500 })
  }
}
```

---

## Components

Three tiers, strict separation.

### ui/ — base primitives

```ts
// components/ui/Button.tsx
// Generic, reusable, no domain knowledge

type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger'
  loading?: boolean
  children: React.ReactNode
} & React.ButtonHTMLAttributes<HTMLButtonElement>

export function Button({ variant = 'primary', loading, children, ...props }: ButtonProps) {
  return (
    <button
      className={cn('btn', `btn-${variant}`, loading && 'btn-loading')}
      disabled={loading}
      {...props}
    >
      {children}
    </button>
  )
}
```

### features/ — domain components

```ts
// components/features/product/ProductCard.tsx
// Knows about the Product type, nothing else

import { Product } from '@/services/product'

type Props = { product: Product }

export function ProductCard({ product }: Props) {
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>{formatCurrency(product.price)}</p>
      <span>{product.inStock ? 'In stock' : 'Out of stock'}</span>
    </div>
  )
}
```

---

## State Management

Only global client state lives in the store. Server state (data from APIs) belongs in React Query / SWR hooks.

```ts
// store/cart.store.ts
import { create } from 'zustand'
import { Product } from '@/services/product'

type CartItem = { product: Product; quantity: number }

type CartStore = {
  items: CartItem[]
  addItem: (product: Product) => void
  removeItem: (productId: string) => void
  clearCart: () => void
  total: () => number
}

export const useCartStore = create<CartStore>((set, get) => ({
  items: [],

  addItem: (product) =>
    set((state) => {
      const existing = state.items.find((i) => i.product.id === product.id)
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.product.id === product.id
              ? { ...i, quantity: i.quantity + 1 }
              : i
          ),
        }
      }
      return { items: [...state.items, { product, quantity: 1 }] }
    }),

  removeItem: (productId) =>
    set((state) => ({
      items: state.items.filter((i) => i.product.id !== productId),
    })),

  clearCart: () => set({ items: [] }),

  total: () =>
    get().items.reduce((sum, i) => sum + i.product.price * i.quantity, 0),
}))
```

---

## Hooks

Hooks wrap data fetching and expose clean state to components. They keep components dumb.

```ts
// hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { Product } from '@/services/product'

const QUERY_KEYS = {
  all: ['products'] as const,
  detail: (id: string) => ['products', id] as const,
}

export function useProducts(filters?: { category?: string }) {
  return useQuery({
    queryKey: [...QUERY_KEYS.all, filters],
    queryFn: async () => {
      const params = new URLSearchParams(filters as any)
      const res = await fetch(`/api/products?${params}`)
      if (!res.ok) throw new Error('Failed to fetch')
      const data = await res.json()
      return data.data as Product[]
    },
  })
}

export function useCreateProduct() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (input: Omit<Product, 'id' | 'createdAt'>) => {
      const res = await fetch('/api/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(input),
      })
      if (!res.ok) throw new Error('Failed to create')
      return res.json()
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.all })
    },
  })
}
```

---

## Lib vs Utils

These two are commonly confused. The rule is simple.

| | `lib/` | `utils/` |
|---|---|---|
| Has side effects | Yes | Never |
| Connects to external systems | Yes | No |
| Pure function | No | Always |
| Examples | `db.ts`, `auth.ts`, `email.ts` | `format.ts`, `cn.ts`, `string.ts` |

### lib/api-client.ts

```ts
// lib/api-client.ts — base fetch wrapper, handles auth headers globally
import { env } from '@/config/env'

const baseHeaders = {
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${env.API_SECRET}`,
}

export const apiClient = {
  get: (path: string) =>
    fetch(`${env.API_BASE_URL}${path}`, {
      headers: baseHeaders,
      next: { revalidate: 60 },
    }),

  post: (path: string, body: unknown) =>
    fetch(`${env.API_BASE_URL}${path}`, {
      method: 'POST',
      headers: baseHeaders,
      body: JSON.stringify(body),
    }),

  patch: (path: string, body: unknown) =>
    fetch(`${env.API_BASE_URL}${path}`, {
      method: 'PATCH',
      headers: baseHeaders,
      body: JSON.stringify(body),
    }),

  delete: (path: string) =>
    fetch(`${env.API_BASE_URL}${path}`, {
      method: 'DELETE',
      headers: baseHeaders,
    }),
}
```

### utils/format.ts

```ts
// utils/format.ts — pure, no imports from lib or services
export const formatCurrency = (amount: number, currency = 'USD') =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount)

export const formatDate = (date: Date | string) =>
  new Intl.DateTimeFormat('en-US', { dateStyle: 'medium' }).format(new Date(date))

export const formatRelativeTime = (date: Date) => {
  const diff = Date.now() - date.getTime()
  const minutes = Math.floor(diff / 60000)
  if (minutes < 60) return `${minutes}m ago`
  const hours = Math.floor(minutes / 60)
  if (hours < 24) return `${hours}h ago`
  return `${Math.floor(hours / 24)}d ago`
}
```

---

## Config & Env

Parse and validate env variables once at startup. Never access `process.env` directly outside this file.

```ts
// config/env.ts
import { z } from 'zod'

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  API_BASE_URL: z.string().url(),
  API_SECRET: z.string().min(1),
  DATABASE_URL: z.string().url().optional(),
  NEXTAUTH_SECRET: z.string().min(1),
  NEXTAUTH_URL: z.string().url(),
})

export const env = envSchema.parse(process.env)
```

```ts
// config/routes.ts — typed route paths, no magic strings in components
export const ROUTES = {
  home: '/',
  login: '/login',
  dashboard: '/dashboard',
  products: {
    list: '/products',
    detail: (id: string) => `/products/${id}`,
    create: '/products/new',
  },
  orders: {
    list: '/orders',
    detail: (id: string) => `/orders/${id}`,
  },
} as const
```

---

## Types

Domain types live co-located inside their service module. The global `types/` folder is only for things shared across multiple domains.

```ts
// types/api.types.ts — shared API response envelope
export type ApiResponse<T> = {
  data: T
  message?: string
}

export type ApiError = {
  error: string
  details?: unknown
}

export type PaginatedResponse<T> = {
  data: T[]
  total: number
  page: number
  limit: number
  hasMore: boolean
}
```

---

## File Naming Conventions

| File | Convention | Example |
|---|---|---|
| Components | PascalCase | `ProductCard.tsx` |
| Hooks | camelCase with `use` prefix | `useProducts.ts` |
| Services | `[domain].service.ts` | `product.service.ts` |
| Schemas | `[domain].schema.ts` | `product.schema.ts` |
| Types | `[domain].types.ts` | `product.types.ts` |
| Constants | `[domain].constant.ts` | `product.constant.ts` |
| Mappers | `[domain].mapper.ts` | `product.mapper.ts` |
| Stores | `[domain].store.ts` | `cart.store.ts` |
| Utilities | camelCase | `format.ts`, `cn.ts` |
| Lib clients | camelCase | `api-client.ts`, `db.ts` |

---

## Import Rules

These rules prevent circular dependencies and keep layers clean.

```
✅ app/page.tsx         → import from services/, components/, hooks/
✅ app/api/route.ts     → import from services/ only
✅ services/            → import from lib/, utils/, config/
✅ components/          → import from hooks/, store/, utils/
✅ hooks/               → import from store/, utils/
✅ store/               → import from types/, utils/
✅ lib/                 → import from config/ only
✅ utils/               → import nothing from this project

❌ services/            → NEVER import from components/, hooks/, store/
❌ utils/               → NEVER import from services/, lib/, store/
❌ lib/                 → NEVER import from services/, components/
❌ components/          → NEVER import directly from lib/
```

Always import a service through its `index.ts` barrel:

```ts
// ✅ correct
import { productService } from '@/services/product'

// ❌ wrong — reaches into internals
import { productService } from '@/services/product/product.service'
```

---

## When to Add a Layer

Don't add layers preemptively. Start simple and let the codebase tell you when it needs more structure.

| Pain point | Solution |
|---|---|
| Route handler is getting long | Move logic to a service |
| Service is doing too much | Split into smaller focused services |
| Multiple services share logic | Extract to a shared util or a `common/` service |
| fetch() headers repeated everywhere | Create `lib/api-client.ts` |
| DB queries duplicated across services | Add a repository layer |
| Same computed value used in many components | Extract to a hook |
| Component fetching data AND rendering | Separate into a data hook + dumb component |
| `process.env.X` scattered everywhere | Centralise in `config/env.ts` |

> Start without the repository. Add it when you feel the pain of not having it. The same applies to every layer here.

---

## Quick Reference

```
Route Handler → thin, validate input, call service, return response
Service       → business logic, call API/DB, use mapper, throw named errors
Mapper        → transform raw external shape → your domain type
Schema        → Zod validation, infer TS types from it
Types         → TS interfaces for domain models and DTOs
Constant      → error strings, enums, magic values
Hook          → React Query/SWR wrapper, exposes clean state to components
Store         → global client-only state (cart, UI state, user prefs)
Lib           → third-party client setup (db, auth, email, queue)
Utils         → pure functions, zero side effects, zero imports
Config        → env parsing, typed route paths, app-wide constants
```
