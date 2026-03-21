# Drizzle vs Prisma Comparison

Feature comparison, migration guide, and decision framework for choosing between Drizzle and Prisma.

## Quick Comparison

| Feature | Drizzle ORM | Prisma |
|---------|-------------|--------|
| **Type Safety** | Compile-time inference | Generated types |
| **Bundle Size** | **~35KB** | ~230KB |
| **Runtime** | **Zero dependencies** | Heavy runtime |
| **Cold Start** | **~10ms** | ~250ms |
| **Query Performance** | **Faster (native SQL)** | Slower (translation layer) |
| **Learning Curve** | Moderate (SQL knowledge helpful) | Easier (abstracted) |
| **Migrations** | SQL-based | Declarative schema |
| **Raw SQL** | **First-class support** | Limited support |
| **Edge Runtime** | **Fully compatible** | Limited support |
| **Ecosystem** | Growing | Mature |

## When to Choose Drizzle

### Choose Drizzle if you need:

1. **Performance-critical applications** - Microservices, high-throughput APIs, serverless/edge
2. **Minimal bundle size** - Client-side DB, edge runtimes, mobile apps
3. **SQL control** - CTEs, window functions, raw SQL, DB-specific optimizations
4. **Type inference over generation** - No build step, immediate TypeScript feedback

## Feature Comparison

### Schema Definition

**Drizzle** (TypeScript-first):

```typescript
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { defineRelations } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').notNull().references(() => users.id),
});
```

Relations defined separately:

```typescript
import { defineRelations } from 'drizzle-orm';
import * as schema from './schema';

export const relations = defineRelations(schema, (r) => ({
  users: {
    posts: r.many.posts(),
  },
  posts: {
    author: r.one.users({
      from: r.posts.authorId,
      to: r.users.id,
    }),
  },
}));
```

**Prisma** (Schema DSL):

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int
  author   User   @relation(fields: [authorId], references: [id])
}
```

### Querying

**Drizzle** - SQL builder:

```typescript
import { eq, like, and, gt } from 'drizzle-orm';

const user = await db.select().from(users).where(eq(users.id, 1));

const results = await db.select()
  .from(users)
  .where(
    and(
      like(users.email, '%@example.com'),
      gt(users.createdAt, new Date('2024-01-01'))
    )
  );
```

**Drizzle** - Relational queries (object-based):

```typescript
const user = await db.query.users.findFirst({
  where: { id: 1 },
});

const results = await db.query.users.findMany({
  where: {
    email: { like: '%@example.com' },
    createdAt: { gt: new Date('2024-01-01') },
  },
  with: { posts: true },
  orderBy: { createdAt: 'desc' },
  limit: 10,
});
```

**Prisma** (Fluent API):

```typescript
const user = await prisma.user.findUnique({ where: { id: 1 } });

const results = await prisma.user.findMany({
  where: {
    email: { endsWith: '@example.com' },
    createdAt: { gt: new Date('2024-01-01') },
  },
  include: { posts: true },
  orderBy: { createdAt: 'desc' },
  take: 10,
});
```

### Database Setup

**Drizzle**:

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { relations } from './relations';

const db = drizzle(process.env.DATABASE_URL!, { relations });
```

**Prisma**:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();
```

### Migrations

**Drizzle** (SQL-based):

```bash
npx drizzle-kit generate
npx drizzle-kit migrate
```

**Prisma** (Declarative):

```bash
npx prisma migrate dev --name add_users
```

### Type Generation

**Drizzle** (Inferred at compile time):

```typescript
type User = typeof users.$inferSelect;
type NewUser = typeof users.$inferInsert;
```

**Prisma** (Generated after schema change):

```typescript
// Run: npx prisma generate
import { User, Post } from '@prisma/client';
```

### Raw SQL

**Drizzle** (First-class):

```typescript
import { sql } from 'drizzle-orm';

const result = await db.execute(
  sql`SELECT * FROM ${users} WHERE ${users.email} = ${email}`
);

const customQuery = await db
  .select({
    user: users,
    postCount: sql<number>`COUNT(${posts.id})`,
  })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .groupBy(users.id);
```

**Prisma** (Limited):

```typescript
const result = await prisma.$queryRaw`
  SELECT * FROM users WHERE email = ${email}
`;
```

## Performance Benchmarks

### Query Execution Time (1000 queries)

| Operation | Drizzle | Prisma | Difference |
|-----------|---------|--------|------------|
| findUnique | 1.2s | 3.1s | **2.6x faster** |
| findMany (10 rows) | 1.5s | 3.8s | **2.5x faster** |
| findMany (100 rows) | 2.1s | 5.2s | **2.5x faster** |
| create | 1.8s | 4.1s | **2.3x faster** |
| update | 1.7s | 3.9s | **2.3x faster** |

### Cold Start Times (AWS Lambda)

| Database | Drizzle | Prisma |
|----------|---------|--------|
| PostgreSQL | ~50ms | ~300ms |
| MySQL | ~45ms | ~280ms |
| SQLite | ~10ms | ~150ms |

## Migration from Prisma to Drizzle

### Step 1: Install Drizzle

```bash
npm install drizzle-orm@beta
npm install -D drizzle-kit@beta
```

### Step 2: Introspect Existing Database

```bash
npx drizzle-kit pull
# Generates schema.ts AND relations.ts with defineRelations syntax
```

### Step 3: Convert Queries

```typescript
// Prisma → Drizzle mapping

// findUnique
await prisma.user.findUnique({ where: { id: 1 } });
await db.select().from(users).where(eq(users.id, 1));

// findMany with filters (relational query style)
await prisma.user.findMany({ where: { role: 'admin' } });
await db.query.users.findMany({ where: { role: 'admin' } });

// create
await prisma.user.create({ data: { email: 'user@example.com' } });
await db.insert(users).values({ email: 'user@example.com' }).returning();

// update
await prisma.user.update({ where: { id: 1 }, data: { name: 'John' } });
await db.update(users).set({ name: 'John' }).where(eq(users.id, 1));

// delete
await prisma.user.delete({ where: { id: 1 } });
await db.delete(users).where(eq(users.id, 1));

// Relations
await prisma.user.findMany({ include: { posts: true } });
await db.query.users.findMany({ with: { posts: true } });
```

### Step 4: Remove Prisma

```bash
npm test
npm uninstall prisma @prisma/client
rm -rf prisma/
```

## Decision Matrix

| Requirement | Drizzle | Prisma |
|-------------|---------|--------|
| Need minimal bundle size | Yes | No |
| Edge runtime deployment | Yes | Limited |
| Team unfamiliar with SQL | No | Yes |
| Complex raw SQL queries | Yes | No |
| Rapid prototyping | Moderate | Yes |
| Performance critical | Yes | No |
| Mature ecosystem | Growing | Yes |
| Zero dependencies | Yes | No |
