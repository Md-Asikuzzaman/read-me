# Prisma + MongoDB + Next.js Setup Guide (2026)

> **Last updated:** April 2026 | Prisma v6.19.x | Next.js 15+

---

## ⚠️ Critical Version Notice

| Package | Version to Use | Reason |
|---|---|---|
| `prisma` | `@6` (v6.19.x) | Prisma v7 does NOT support MongoDB yet |
| `@prisma/client` | `@6` (v6.19.x) | Must match prisma version |
| `next` | `15+` | App Router recommended |

> Prisma v7 dropped MongoDB support. MongoDB support is planned for a future v7 release. **Always pin to v6 until officially announced.**

---

## Prerequisites

- Node.js 18+
- A [MongoDB Atlas](https://www.mongodb.com/atlas) account (free tier works) — must be a **replica set** (Atlas is by default)
- A Next.js project

---

## 1. Create Next.js App

```bash
npx create-next-app@latest my-app --typescript --tailwind --app
cd my-app
```

---

## 2. Install Prisma v6

```bash
npm install -D prisma@6 dotenv
npm install @prisma/client@6
```

> `dotenv` is needed as a dev dependency so the **Prisma CLI** can read your `.env` file. Next.js handles env loading natively at runtime — no extra package needed there.

---

## 3. Initialize Prisma

```bash
npx prisma init --datasource-provider mongodb
```

This creates:
```
prisma/
  schema.prisma
prisma.config.ts
.env
```

---

## 4. Configure `prisma.config.ts`

In Prisma v6, the `datasource url` must stay in `schema.prisma`. Keep `prisma.config.ts` simple:

```ts
// prisma.config.ts
import "dotenv/config";
import { defineConfig } from "prisma/config";

export default defineConfig({
 schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  engine: "classic",
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

> **Do NOT add a `datasource` block here in v6.** Moving the URL to `prisma.config.ts` is a v7-only feature. Doing it in v6 will break `prisma generate`.

---

## 5. Configure `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id    String @id @default(auto()) @map("_id") @db.ObjectId
  name  String
  email String @unique
}
```

### Key MongoDB Schema Rules

- `id` must be `String` type (not `Int`)
- Always add `@id @default(auto()) @map("_id") @db.ObjectId`
- Relations use `String @db.ObjectId` for foreign keys
- No `autoincrement()` — MongoDB doesn't support it

---

## 6. Set Up `.env`

```env
DATABASE_URL="mongodb+srv://username:password@cluster0.mongodb.net/mydb?retryWrites=true&w=majority"
```

Get your connection string from **MongoDB Atlas → Connect → Drivers**.

> Add `.env` to `.gitignore` — never commit secrets.

---

## 7. Push Schema to MongoDB

```bash
npx prisma db push
```

> ❌ **Never** run `npx prisma migrate dev` with MongoDB — it will fail. MongoDB does not support Prisma Migrate. Always use `db push`.

This command:
- Creates collections in MongoDB
- Sets up indexes for unique fields and relations
- Does NOT create migration files

---

## 8. Generate Prisma Client

```bash
npx prisma generate
```

The client gets generated at `app/generated/prisma` (as set in `output`).

---

## 9. Create the Prisma Singleton

Create `lib/prisma.ts` in your project root:

```ts
// lib/prisma.ts
import { PrismaClient } from "../app/generated/prisma";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}

export default prisma;
```

### Why the singleton pattern?

Next.js hot-reloads files in development. Without this, every file save creates a **new** `PrismaClient` with its own connection pool — you'll quickly hit MongoDB's connection limit. Storing the instance on `globalThis` ensures it's reused across hot reloads.

---

## 10. Use Prisma in Your App

### Server Component

```tsx
// app/users/page.tsx
import prisma from "@/lib/prisma";

export default async function UsersPage() {
  const users = await prisma.user.findMany();

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name} — {user.email}</li>
      ))}
    </ul>
  );
}
```

### API Route

```ts
// app/api/users/route.ts
import { NextResponse } from "next/server";
import prisma from "@/lib/prisma";

export async function GET() {
  const users = await prisma.user.findMany();
  return NextResponse.json(users);
}

export async function POST(req: Request) {
  const { name, email } = await req.json();
  const user = await prisma.user.create({
    data: { name, email },
  });
  return NextResponse.json(user, { status: 201 });
}
```

### Server Action

```ts
// app/actions/userActions.ts
"use server";
import prisma from "@/lib/prisma";

export async function createUser(name: string, email: string) {
  const user = await prisma.user.create({
    data: { name, email },
  });
  return user;
}

export async function deleteUser(id: string) {
  await prisma.user.delete({ where: { id } });
}
```

---

## 11. Example: Full Schema with Relations

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  title     String
  content   String?
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String   @db.ObjectId
}
```

After any schema change, always run:

```bash
npx prisma db push
npx prisma generate
```

---

## Common MongoDB + Prisma Gotchas

### ❌ `prisma migrate` is not supported
```bash
# This will FAIL with MongoDB
npx prisma migrate dev

# Always use this instead
npx prisma db push
```

### ❌ Prisma Studio doesn't work with MongoDB
```bash
# This won't work for MongoDB (as of 2026)
npx prisma studio
# Use MongoDB Atlas UI or MongoDB Compass instead
```

### ⚠️ Transactions require a replica set
`prisma.$transaction()` will fail on a standalone `mongod` instance. MongoDB Atlas is a replica set by default and works fine.

### ⚠️ Editor type error about `url` in schema
If your editor shows: *"The datasource property url is no longer supported in schema files"* — this is a **false alarm** from the VS Code Prisma extension applying v7 rules to a v6 project. The CLI (v6.19.x) works correctly and `url` must stay in `schema.prisma`.

### ⚠️ Don't add `datasource` block to `prisma.config.ts` in v6
Doing so will break `prisma generate` with error `P1012: Argument "url" is missing`. This is a v7-only feature.

---

## Project Structure

```
my-app/
├── app/
│   ├── api/
│   │   └── users/
│   │       └── route.ts
│   ├── generated/
│   │   └── prisma/          ← auto-generated, don't edit
│   └── page.tsx
├── lib/
│   └── prisma.ts            ← singleton client
├── prisma/
│   └── schema.prisma        ← your data models
├── prisma.config.ts         ← Prisma CLI config
├── .env                     ← DATABASE_URL (never commit)
└── package.json
```

---

## Quick Reference — Commands

| Command | Purpose |
|---|---|
| `npx prisma init` | Initialize Prisma in project |
| `npx prisma db push` | Sync schema to MongoDB (use instead of migrate) |
| `npx prisma generate` | Regenerate Prisma Client after schema changes |
| `npx prisma db pull` | Introspect existing MongoDB into schema |
| `npx prisma --version` | Check installed Prisma version |

---

## What Changes When Prisma v7 Adds MongoDB

When Prisma v7 eventually supports MongoDB, these things will change:

1. `provider` in generator changes from `"prisma-client-js"` → `"prisma-client"`
2. `url` moves from `schema.prisma` → `prisma.config.ts`
3. `PrismaClient` constructor will require a driver adapter
4. Package ships as ESM — `tsconfig.json` needs `"module": "ESNext"`

Until then, **stay on v6**.

---

## Resources

- [Prisma MongoDB Quickstart](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/mongodb-typescript-mongodb)
- [Prisma v6 → v7 Upgrade Guide](https://www.prisma.io/docs/orm/more/upgrade-guides/upgrading-versions/upgrading-to-prisma-7)
- [MongoDB Atlas](https://www.mongodb.com/atlas)
- [Prisma Config Reference](https://www.prisma.io/docs/orm/reference/prisma-config-reference)
- [prisma/prisma GitHub](https://github.com/prisma/prisma)
