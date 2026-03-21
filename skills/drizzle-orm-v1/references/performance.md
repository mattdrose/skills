# Performance Optimization

Connection pooling, query optimization, edge runtime integration, and performance best practices.

## Connection Pooling

### PostgreSQL (node-postgres)

```typescript
import { Pool } from 'pg';
import { drizzle } from 'drizzle-orm/node-postgres';

const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

export const db = drizzle(pool);

process.on('SIGTERM', async () => {
  await pool.end();
});
```

### MySQL (mysql2)

```typescript
import mysql from 'mysql2/promise';
import { drizzle } from 'drizzle-orm/mysql2';

const poolConnection = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  maxIdle: 10,
  idleTimeout: 60000,
  queueLimit: 0,
  enableKeepAlive: true,
  keepAliveInitialDelay: 0,
});

export const db = drizzle(poolConnection);
```

### SQLite (better-sqlite3)

```typescript
import Database from 'better-sqlite3';
import { drizzle } from 'drizzle-orm/better-sqlite3';

const sqlite = new Database('sqlite.db', {
  readonly: false,
  fileMustExist: false,
  timeout: 5000,
});

sqlite.pragma('journal_mode = WAL');
sqlite.pragma('synchronous = normal');
sqlite.pragma('cache_size = -64000');
sqlite.pragma('temp_store = memory');

export const db = drizzle(sqlite);

process.on('exit', () => sqlite.close());
```

## Setting Up with Relations

When using relations, pass them to `drizzle()`:

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { relations } from './relations';

// With connection URL
export const db = drizzle(process.env.DATABASE_URL!, { relations });

// With pool
export const db = drizzle(pool, { relations });
```

No `mode` parameter needed for MySQL dialects.

## Query Optimization

### Select Only Needed Columns

```typescript
// Fetch only needed columns
const users = await db.select({
  id: users.id,
  email: users.email,
  name: users.name,
}).from(users);
```

### Use Indexes Effectively

```typescript
import { pgTable, serial, text, varchar, index } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull(),
  city: text('city'),
  status: text('status'),
}, (table) => ({
  emailIdx: index('email_idx').on(table.email),
  cityStatusIdx: index('city_status_idx').on(table.city, table.status),
}));
```

### Analyze Query Plans

```typescript
import { sql } from 'drizzle-orm';

const plan = await db.execute(
  sql`EXPLAIN ANALYZE SELECT * FROM ${users} WHERE ${users.email} = 'user@example.com'`
);
```

### Pagination Performance

```typescript
// Cursor-based pagination (constant time)
const page = await db.select()
  .from(users)
  .where(gt(users.id, lastSeenId))
  .orderBy(asc(users.id))
  .limit(20);

// Seek method for timestamp-based pagination
const page = await db.select()
  .from(posts)
  .where(lt(posts.createdAt, lastSeenTimestamp))
  .orderBy(desc(posts.createdAt))
  .limit(20);
```

## Edge Runtime Integration

### Cloudflare Workers (D1)

```typescript
import { drizzle } from 'drizzle-orm/d1';

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const db = drizzle(env.DB);

    const users = await db.select().from(users).limit(10);

    return Response.json(users);
  },
};
```

### Vercel Edge (Neon)

```typescript
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';

export const runtime = 'edge';

export async function GET() {
  const sql = neon(process.env.DATABASE_URL!);
  const db = drizzle(sql);

  const users = await db.select().from(users);

  return Response.json(users);
}
```

## Caching Strategies

### In-Memory Cache

```typescript
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, any>({
  max: 500,
  ttl: 1000 * 60 * 5,
});

async function getCachedUser(id: number) {
  const key = `user:${id}`;
  const cached = cache.get(key);

  if (cached) return cached;

  const user = await db.select().from(users).where(eq(users.id, id));
  cache.set(key, user);

  return user;
}
```

### Redis Cache Layer

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedData<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 300
): Promise<T> {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const data = await fetcher();
  await redis.setex(key, ttl, JSON.stringify(data));

  return data;
}
```

### Materialized Views (PostgreSQL)

```typescript
export const userStats = pgMaterializedView('user_stats').as((qb) =>
  qb.select({
    id: users.id,
    name: users.name,
    postCount: sql<number>`COUNT(${posts.id})`,
    commentCount: sql<number>`COUNT(${comments.id})`,
  })
  .from(users)
  .leftJoin(posts, eq(posts.authorId, users.id))
  .leftJoin(comments, eq(comments.userId, users.id))
  .groupBy(users.id)
);

await db.execute(sql`REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats`);
const stats = await db.select().from(userStats);
```

## Batch Operations Optimization

### Chunk Processing

```typescript
async function* chunked<T>(array: T[], size: number) {
  for (let i = 0; i < array.length; i += size) {
    yield array.slice(i, i + size);
  }
}

async function bulkUpdate(updates: { id: number; name: string }[]) {
  for await (const chunk of chunked(updates, 100)) {
    await db.transaction(async (tx) => {
      for (const update of chunk) {
        await tx.update(users)
          .set({ name: update.name })
          .where(eq(users.id, update.id));
      }
    });
  }
}
```

## Connection Management

### Serverless Optimization

```typescript
// Reuse connection across warm starts
let cachedDb: ReturnType<typeof drizzle> | null = null;

export async function handler() {
  if (!cachedDb) {
    const pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      max: 1,
    });
    cachedDb = drizzle(pool);
  }

  const users = await cachedDb.select().from(users);
  return users;
}
```

### HTTP-based Databases (Neon, Turso)

```typescript
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle(sql);

const users = await db.select().from(users);
```

## Read Replicas

```typescript
import { Pool } from 'pg';
import { drizzle } from 'drizzle-orm/node-postgres';

const primaryPool = new Pool({ connectionString: process.env.PRIMARY_DB_URL });
const primaryDb = drizzle(primaryPool);

const replicaPool = new Pool({ connectionString: process.env.REPLICA_DB_URL });
const replicaDb = drizzle(replicaPool);

async function getUsers() {
  return replicaDb.select().from(users);
}

async function createUser(data: NewUser) {
  return primaryDb.insert(users).values(data).returning();
}
```

## Monitoring & Profiling

### Query Logging

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';

const db = drizzle(pool, {
  logger: {
    logQuery(query: string, params: unknown[]) {
      console.log('Query:', query);
      console.log('Params:', params);
    },
  },
});
```

## Best Practices Summary

1. **Always use connection pooling** in long-running processes
2. **Select only needed columns** to reduce network transfer
3. **Add indexes** on frequently queried columns and foreign keys
4. **Use cursor-based pagination** instead of OFFSET for large datasets
5. **Batch operations** when inserting/updating multiple records
6. **Cache expensive queries** with appropriate TTL
7. **Monitor slow queries** and optimize with EXPLAIN ANALYZE
8. **Use prepared statements** for frequently executed queries
9. **Implement read replicas** for high-traffic read operations
10. **Use HTTP-based databases** (Neon, Turso) for edge/serverless
