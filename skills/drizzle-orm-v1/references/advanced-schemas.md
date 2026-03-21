# Advanced Schemas

Deep dive into complex schema patterns, custom types, and database-specific features in Drizzle ORM v1.

## Custom Column Types

### Enums

```typescript
import { pgEnum, pgTable, serial } from 'drizzle-orm/pg-core';

// PostgreSQL native enum
export const roleEnum = pgEnum('role', ['admin', 'user', 'guest']);

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  role: roleEnum('role').default('user'),
});

// MySQL/SQLite: Use text with constraints
import { mysqlTable, text } from 'drizzle-orm/mysql-core';

export const users = mysqlTable('users', {
  role: text('role', { enum: ['admin', 'user', 'guest'] }).default('user'),
});
```

### Custom JSON Types

```typescript
import { pgTable, serial, json } from 'drizzle-orm/pg-core';
import { z } from 'zod';

const MetadataSchema = z.object({
  theme: z.enum(['light', 'dark']),
  locale: z.string(),
  notifications: z.boolean(),
});

type Metadata = z.infer<typeof MetadataSchema>;

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  metadata: json('metadata').$type<Metadata>(),
});

async function updateMetadata(userId: number, metadata: unknown) {
  const validated = MetadataSchema.parse(metadata);
  await db.update(users).set({ metadata: validated }).where(eq(users.id, userId));
}
```

### Arrays

```typescript
import { pgTable, serial, text } from 'drizzle-orm/pg-core';

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  tags: text('tags').array(),
});

import { arrayContains, arrayContained } from 'drizzle-orm';

await db.select().from(posts).where(arrayContains(posts.tags, ['typescript', 'drizzle']));
```

## Indexes

### Basic Indexes

```typescript
import { pgTable, serial, text, varchar, index, uniqueIndex } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull(),
  name: text('name'),
  city: text('city'),
}, (table) => ({
  emailIdx: uniqueIndex('email_idx').on(table.email),
  nameIdx: index('name_idx').on(table.name),
  cityNameIdx: index('city_name_idx').on(table.city, table.name),
}));
```

### Partial Indexes

```typescript
import { sql } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }),
  deletedAt: timestamp('deleted_at'),
}, (table) => ({
  activeEmailIdx: uniqueIndex('active_email_idx')
    .on(table.email)
    .where(sql`${table.deletedAt} IS NULL`),
}));
```

### Full-Text Search

```typescript
import { pgTable, serial, text, index } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
}, (table) => ({
  searchIdx: index('search_idx').using(
    'gin',
    sql`to_tsvector('english', ${table.title} || ' ' || ${table.content})`
  ),
}));

const results = await db.select().from(posts).where(
  sql`to_tsvector('english', ${posts.title} || ' ' || ${posts.content}) @@ plainto_tsquery('english', 'typescript orm')`
);
```

## Composite Keys

```typescript
import { pgTable, text, primaryKey } from 'drizzle-orm/pg-core';

export const userPreferences = pgTable('user_preferences', {
  userId: integer('user_id').notNull(),
  key: text('key').notNull(),
  value: text('value').notNull(),
}, (table) => ({
  pk: primaryKey({ columns: [table.userId, table.key] }),
}));
```

## Check Constraints

```typescript
import { pgTable, serial, integer, check } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

export const products = pgTable('products', {
  id: serial('id').primaryKey(),
  price: integer('price').notNull(),
  discountPrice: integer('discount_price'),
}, (table) => ({
  priceCheck: check('price_check', sql`${table.price} > 0`),
  discountCheck: check('discount_check', sql`${table.discountPrice} < ${table.price}`),
}));
```

## Generated Columns

```typescript
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  firstName: text('first_name').notNull(),
  lastName: text('last_name').notNull(),
  fullName: text('full_name').generatedAlwaysAs(
    (): SQL => sql`${users.firstName} || ' ' || ${users.lastName}`,
    { mode: 'stored' }
  ),
});
```

## Multi-Tenant Patterns

### Row-Level Security (PostgreSQL)

```typescript
import { pgTable, serial, text, uuid } from 'drizzle-orm/pg-core';

export const tenants = pgTable('tenants', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
});

export const documents = pgTable('documents', {
  id: serial('id').primaryKey(),
  tenantId: uuid('tenant_id').notNull().references(() => tenants.id),
  title: text('title').notNull(),
  content: text('content'),
});

// Apply RLS policy (via migration SQL)
/*
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON documents
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
*/

await db.execute(sql`SET app.current_tenant_id = ${tenantId}`);
```

## Database-Specific Features

### PostgreSQL: JSONB Operations

```typescript
import { pgTable, serial, jsonb } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

export const settings = pgTable('settings', {
  id: serial('id').primaryKey(),
  config: jsonb('config').$type<Record<string, unknown>>(),
});

await db.select().from(settings).where(
  sql`${settings.config}->>'theme' = 'dark'`
);

await db.select().from(settings).where(
  sql`${settings.config} @> '{"notifications": {"email": true}}'::jsonb`
);
```

## Schema Organization

Separate schema tables from relations. Relations are defined centrally with `defineRelations`:

```typescript
// db/schema/users.ts
export const users = pgTable('users', { ... });

// db/schema/posts.ts
export const posts = pgTable('posts', { ... });

// db/schema/index.ts
export * from './users';
export * from './posts';

// db/relations.ts
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

// db/client.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { relations } from './relations';

export const db = drizzle(process.env.DATABASE_URL!, { relations });
```

## Type Inference Helpers

```typescript
import { InferSelectModel, InferInsertModel } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull(),
  name: text('name'),
});

export type User = InferSelectModel<typeof users>;
export type NewUser = InferInsertModel<typeof users>;
export type UserUpdate = Partial<NewUser>;
```

## Best Practices

### Naming Conventions

```typescript
// Consistent naming: camelCase in TS, snake_case in DB
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  firstName: text('first_name'),
  createdAt: timestamp('created_at'),
});
```

### Default Values

```typescript
import { sql } from 'drizzle-orm';

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  slug: text('slug').notNull(),
  views: integer('views').default(0),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').default(sql`CURRENT_TIMESTAMP`),
  uuid: uuid('uuid').defaultRandom(),
});
```
