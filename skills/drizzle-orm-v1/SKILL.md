---
name: drizzle-orm-v1
description: "Type-safe SQL ORM for TypeScript (v1 RC). Use when working with drizzle-orm v1, defineRelations, relational queries v2, or drizzle-orm@beta."
progressive_disclosure:
  entry_point:
    summary: "Type-safe SQL ORM for TypeScript (v1 RC) with zero runtime overhead"
    when_to_use: "When working with drizzle-orm v1 or related functionality."
    quick_start: "1. Review the core concepts below. 2. Apply patterns to your use case. 3. Follow best practices for implementation."
  references:
    - advanced-schemas.md
    - performance.md
    - query-patterns.md
    - vs-prisma.md
---
# Drizzle ORM v1

Modern TypeScript-first ORM with zero dependencies, compile-time type safety, and SQL-like syntax. Optimized for edge runtimes and serverless environments.

## Quick Start

### Installation

```bash
# Core ORM (v1 RC)
npm install drizzle-orm@beta

# Database driver (choose one)
npm install pg            # PostgreSQL
npm install mysql2        # MySQL
npm install better-sqlite3 # SQLite

# Drizzle Kit (migrations)
npm install -D drizzle-kit@beta
```

### Basic Setup

```typescript
// db/schema.ts
import { pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});

// db/relations.ts
import { defineRelations } from 'drizzle-orm';
import * as schema from './schema';

export const relations = defineRelations(schema, (r) => ({
  // relations go here
}));

// db/client.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { relations } from './relations';

export const db = drizzle(process.env.DATABASE_URL!, { relations });
```

### First Query

```typescript
import { db } from './db/client';
import { users } from './db/schema';
import { eq } from 'drizzle-orm';

// Insert
const newUser = await db.insert(users).values({
  email: 'user@example.com',
  name: 'John Doe',
}).returning();

// Select
const allUsers = await db.select().from(users);

// Where
const user = await db.select().from(users).where(eq(users.id, 1));

// Update
await db.update(users).set({ name: 'Jane Doe' }).where(eq(users.id, 1));

// Delete
await db.delete(users).where(eq(users.id, 1));
```

## Schema Definition

### Column Types Reference

| PostgreSQL | MySQL | SQLite | TypeScript |
|------------|-------|--------|------------|
| `serial()` | `serial()` | `integer()` | `number` |
| `text()` | `text()` | `text()` | `string` |
| `integer()` | `int()` | `integer()` | `number` |
| `boolean()` | `boolean()` | `integer()` | `boolean` |
| `timestamp()` | `datetime()` | `integer()` | `Date` |
| `json()` | `json()` | `text()` | `unknown` |
| `uuid()` | `varchar(36)` | `text()` | `string` |

### Common Schema Patterns

```typescript
import { pgTable, serial, text, varchar, integer, boolean, timestamp, json, unique } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  passwordHash: varchar('password_hash', { length: 255 }).notNull(),
  role: text('role', { enum: ['admin', 'user', 'guest'] }).default('user'),
  metadata: json('metadata').$type<{ theme: string; locale: string }>(),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  emailIdx: unique('email_unique_idx').on(table.email),
}));

// Infer TypeScript types
type User = typeof users.$inferSelect;
type NewUser = typeof users.$inferInsert;
```

## Relations

All relations are defined in a single place using `defineRelations`. The `r` parameter provides autocomplete for all tables, columns, and relation functions (`one`, `many`, `through`).

### One-to-Many

```typescript
// db/schema.ts
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';

export const authors = pgTable('authors', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').notNull().references(() => authors.id),
});

// db/relations.ts
import { defineRelations } from 'drizzle-orm';
import * as schema from './schema';

export const relations = defineRelations(schema, (r) => ({
  authors: {
    posts: r.many.posts(),
  },
  posts: {
    author: r.one.authors({
      from: r.posts.authorId,
      to: r.authors.id,
    }),
  },
}));

// Query with relations
const authorsWithPosts = await db.query.authors.findMany({
  with: { posts: true },
});
```

You can define `many` without `one` by specifying `from`/`to` on the `many` side:

```typescript
export const relations = defineRelations(schema, (r) => ({
  authors: {
    posts: r.many.posts({
      from: r.authors.id,
      to: r.posts.authorId,
    }),
  },
}));
```

### Many-to-Many

Use `through` for direct many-to-many relations via a junction table:

```typescript
// db/schema.ts
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

export const groups = pgTable('groups', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
});

export const usersToGroups = pgTable('users_to_groups', {
  userId: integer('user_id').notNull().references(() => users.id),
  groupId: integer('group_id').notNull().references(() => groups.id),
}, (table) => ({
  pk: primaryKey({ columns: [table.userId, table.groupId] }),
}));

// db/relations.ts
import { defineRelations } from 'drizzle-orm';
import * as schema from './schema';

export const relations = defineRelations(schema, (r) => ({
  users: {
    groups: r.many.groups({
      from: r.users.id.through(r.usersToGroups.userId),
      to: r.groups.id.through(r.usersToGroups.groupId),
    }),
  },
  groups: {
    participants: r.many.users(),
  },
}));

// Query directly through junction table
const usersWithGroups = await db.query.users.findMany({
  with: { groups: true },
});
```

### Self-Referencing Relations

```typescript
export const relations = defineRelations(schema, (r) => ({
  users: {
    invitee: r.one.users({
      from: r.users.invitedBy,
      to: r.users.id,
    }),
    posts: r.many.posts(),
  },
}));
```

### Relation Options

- **`alias`**: Disambiguate multiple relations to the same table (replaces `relationName`)
- **`optional`**: Set to `false` to make the relation required at the type level

```typescript
export const relations = defineRelations(schema, (r) => ({
  posts: {
    author: r.one.users({
      from: r.posts.authorId,
      to: r.users.id,
      alias: "author_post",
      optional: false,
    }),
  },
}));
```

### Predefined Filters

Apply filters directly in the relation definition:

```typescript
export const relations = defineRelations(schema, (r) => ({
  groups: {
    verifiedUsers: r.many.users({
      from: r.groups.id.through(r.usersToGroups.groupId),
      to: r.users.id.through(r.usersToGroups.userId),
      where: { verified: true },
    }),
  },
}));

const groupsWithVerified = await db.query.groups.findMany({
  with: { verifiedUsers: true },
});
```

### Splitting Relations

Use `defineRelationsPart` to organize relations across files, then merge:

```typescript
// relations/users.ts
import { defineRelationsPart } from 'drizzle-orm';
import * as schema from '../schema';

export const userRelations = defineRelationsPart(schema, (r) => ({
  users: {
    posts: r.many.posts(),
  },
}));

// relations/posts.ts
export const postRelations = defineRelationsPart(schema, (r) => ({
  posts: {
    author: r.one.users({
      from: r.posts.authorId,
      to: r.users.id,
    }),
  },
}));

// db/client.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { userRelations } from './relations/users';
import { postRelations } from './relations/posts';

const db = drizzle(process.env.DATABASE_URL!, {
  relations: { ...userRelations, ...postRelations },
});
```

## Relational Queries

### Object-Based Filtering

```typescript
// Simple equality
const user = await db.query.users.findMany({
  where: { id: 1 },
});

// Multiple conditions (implicit AND)
const admins = await db.query.users.findMany({
  where: { role: 'admin', isActive: true },
});

// Comparison operators
const recentUsers = await db.query.users.findMany({
  where: {
    age: { gt: 18 },
    name: { like: 'John%' },
  },
});

// OR
const results = await db.query.users.findMany({
  where: {
    OR: [
      { id: { gt: 10 } },
      { name: { like: 'John%' } },
    ],
  },
});

// NOT
const nonAdmins = await db.query.users.findMany({
  where: {
    NOT: { role: 'admin' },
  },
});

// RAW for complex expressions
const custom = await db.query.users.findMany({
  where: {
    RAW: (table) => sql`${table.age} BETWEEN 25 AND 35`,
  },
});

// Complex nested
const complex = await db.query.users.findMany({
  where: {
    AND: [
      {
        OR: [
          { RAW: (table) => sql`LOWER(${table.name}) LIKE 'john%'` },
          { name: { ilike: 'jane%' } },
        ],
      },
      { RAW: (table) => sql`${table.age} BETWEEN 25 AND 35` },
    ],
  },
});
```

### Filtering by Relations

```typescript
const usersWithPosts = await db.query.usersTable.findMany({
  where: {
    id: { gt: 10 },
    posts: {
      content: { like: 'M%' },
    },
  },
});
```

### Object-Based Ordering

```typescript
const sorted = await db.query.users.findMany({
  orderBy: { id: 'asc' },
});

const multiSort = await db.query.users.findMany({
  orderBy: { createdAt: 'desc' },
});
```

### Offset on Related Objects

```typescript
await db.query.posts.findMany({
  limit: 5,
  offset: 2,
  with: {
    comments: {
      offset: 3,
      limit: 3,
    },
  },
});
```

## SQL Builder Queries

### Filtering

```typescript
import { eq, ne, gt, gte, lt, lte, like, ilike, inArray, isNull, isNotNull, and, or, between } from 'drizzle-orm';

// Equality
await db.select().from(users).where(eq(users.email, 'user@example.com'));

// Comparison
await db.select().from(users).where(gt(users.id, 10));

// Pattern matching
await db.select().from(users).where(like(users.name, '%John%'));

// Multiple conditions
await db.select().from(users).where(
  and(
    eq(users.role, 'admin'),
    gt(users.createdAt, new Date('2024-01-01'))
  )
);

// IN clause
await db.select().from(users).where(inArray(users.id, [1, 2, 3]));

// NULL checks
await db.select().from(users).where(isNull(users.deletedAt));
```

### Joins

```typescript
import { eq } from 'drizzle-orm';

// Inner join
const result = await db
  .select({ user: users, post: posts })
  .from(users)
  .innerJoin(posts, eq(users.id, posts.authorId));

// Left join
const result = await db
  .select({ user: users, post: posts })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId));

// Multiple joins with aggregation
import { count, sql } from 'drizzle-orm';

const result = await db
  .select({
    authorName: authors.name,
    postCount: count(posts.id),
  })
  .from(authors)
  .leftJoin(posts, eq(authors.id, posts.authorId))
  .groupBy(authors.id);
```

### Pagination & Sorting

```typescript
import { desc, asc } from 'drizzle-orm';

await db.select().from(users).orderBy(desc(users.createdAt));
await db.select().from(users).limit(10).offset(20);
```

## Transactions

```typescript
await db.transaction(async (tx) => {
  await tx.insert(users).values({ email: 'user@example.com', name: 'John' });
  await tx.insert(posts).values({ title: 'First Post', authorId: 1 });
});

const tx = db.transaction(async (tx) => {
  const user = await tx.insert(users).values({ ... }).returning();

  if (!user) {
    tx.rollback();
    return;
  }

  await tx.insert(posts).values({ authorId: user.id });
});
```

## Migrations

### Drizzle Kit Configuration

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

### Migration Workflow

Migration files are grouped into folders per migration (SQL file + snapshot together). No shared `journal.json`.

```bash
# Generate migration
npx drizzle-kit generate

# Apply migration
npx drizzle-kit migrate

# Push schema directly (dev only)
npx drizzle-kit push

# Introspect existing database (also generates relations.ts)
npx drizzle-kit pull

# Drizzle Studio (database GUI)
npx drizzle-kit studio
```

### Generating Relations from Database

`drizzle-kit pull` generates a `relations.ts` file using `defineRelations` syntax:

```bash
npx drizzle-kit pull
# Output: drizzle/schema.ts and drizzle/relations.ts
```

## Validators

Validator packages are bundled with `drizzle-orm`:

```typescript
// Zod
import { createSelectSchema, createInsertSchema } from 'drizzle-orm/zod';

// Valibot
import { createSelectSchema, createInsertSchema } from 'drizzle-orm/valibot';

// TypeBox
import { createSelectSchema, createInsertSchema } from 'drizzle-orm/typebox';

// ArkType
import { createSelectSchema, createInsertSchema } from 'drizzle-orm/arktype';
```

## Navigation

### Detailed References

- **[Advanced Schemas](./references/advanced-schemas.md)** - Custom types, composite keys, indexes, constraints, multi-tenant patterns. Load when designing complex database schemas.

- **[Query Patterns](./references/query-patterns.md)** - Subqueries, CTEs, raw SQL, prepared statements, batch operations. Load when optimizing queries or handling complex filtering.

- **[Performance](./references/performance.md)** - Connection pooling, query optimization, N+1 prevention, prepared statements, edge runtime integration. Load when scaling or optimizing database performance.

- **[vs Prisma](./references/vs-prisma.md)** - Feature comparison, migration guide, when to choose Drizzle over Prisma. Load when evaluating ORMs or migrating from Prisma.

## Red Flags

**Stop and reconsider if:**
- Using `any` or `unknown` for JSON columns without type annotation
- Building raw SQL strings without using `sql` template (SQL injection risk)
- Not using transactions for multi-step data modifications
- Fetching all rows without pagination in production queries
- Missing indexes on foreign keys or frequently queried columns
- Using `select()` without specifying columns for large tables
- Using old `relations()` import from `drizzle-orm` instead of `defineRelations`
- Passing `schema` to `drizzle()` instead of `relations`
- Using `fields`/`references` instead of `from`/`to` in relation definitions

## Performance Benefits vs Prisma

| Metric | Drizzle | Prisma |
|--------|---------|--------|
| **Bundle Size** | ~35KB | ~230KB |
| **Cold Start** | ~10ms | ~250ms |
| **Query Speed** | Baseline | ~2-3x slower |
| **Memory** | ~10MB | ~50MB |
| **Type Generation** | Runtime inference | Build-time generation |

## Integration

- **typescript-core**: Type-safe schema inference with `satisfies`
- **nextjs-core**: Server Actions, Route Handlers, Middleware integration
- **Database Migration**: Safe schema evolution patterns
