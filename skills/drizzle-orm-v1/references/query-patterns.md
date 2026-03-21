# Query Patterns

Advanced querying techniques, subqueries, CTEs, and raw SQL in Drizzle ORM v1.

## Subqueries

### SELECT Subqueries

```typescript
import { sql, eq } from 'drizzle-orm';

// Scalar subquery
const avgPrice = db.select({ value: avg(products.price) }).from(products);

const expensiveProducts = await db
  .select()
  .from(products)
  .where(gt(products.price, avgPrice));

// Correlated subquery
const authorsWithPostCount = await db
  .select({
    author: authors,
    postCount: sql<number>`(
      SELECT COUNT(*)
      FROM ${posts}
      WHERE ${posts.authorId} = ${authors.id}
    )`,
  })
  .from(authors);
```

### EXISTS Subqueries

```typescript
const authorsWithPosts = await db
  .select()
  .from(authors)
  .where(
    sql`EXISTS (
      SELECT 1
      FROM ${posts}
      WHERE ${posts.authorId} = ${authors.id}
    )`
  );

const authorsWithoutPosts = await db
  .select()
  .from(authors)
  .where(
    sql`NOT EXISTS (
      SELECT 1
      FROM ${posts}
      WHERE ${posts.authorId} = ${authors.id}
    )`
  );
```

### IN Subqueries

```typescript
const usersWhoCommented = await db
  .select()
  .from(users)
  .where(
    sql`${users.id} IN (
      SELECT DISTINCT ${comments.userId}
      FROM ${comments}
    )`
  );
```

## Common Table Expressions (CTEs)

### Basic CTE

```typescript
import { sql } from 'drizzle-orm';

const topAuthors = db.$with('top_authors').as(
  db.select({
    id: authors.id,
    name: authors.name,
    postCount: sql<number>`COUNT(${posts.id})`.as('post_count'),
  })
    .from(authors)
    .leftJoin(posts, eq(authors.id, posts.authorId))
    .groupBy(authors.id)
    .having(sql`COUNT(${posts.id}) > 10`)
);

const result = await db
  .with(topAuthors)
  .select()
  .from(topAuthors);
```

### Recursive CTE

```typescript
export const employees = pgTable('employees', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  managerId: integer('manager_id').references((): AnyPgColumn => employees.id),
});

const employeeHierarchy = db.$with('employee_hierarchy').as(
  db.select({
    id: employees.id,
    name: employees.name,
    managerId: employees.managerId,
    level: sql<number>`1`.as('level'),
  })
    .from(employees)
    .where(isNull(employees.managerId))
    .unionAll(
      db.select({
        id: employees.id,
        name: employees.name,
        managerId: employees.managerId,
        level: sql<number>`employee_hierarchy.level + 1`,
      })
        .from(employees)
        .innerJoin(
          sql`employee_hierarchy`,
          sql`${employees.managerId} = employee_hierarchy.id`
        )
    )
);

const hierarchy = await db
  .with(employeeHierarchy)
  .select()
  .from(employeeHierarchy);
```

### Multiple CTEs

```typescript
const activeUsers = db.$with('active_users').as(
  db.select().from(users).where(eq(users.isActive, true))
);

const recentPosts = db.$with('recent_posts').as(
  db.select().from(posts).where(gt(posts.createdAt, sql`NOW() - INTERVAL '30 days'`))
);

const result = await db
  .with(activeUsers, recentPosts)
  .select({
    user: activeUsers,
    post: recentPosts,
  })
  .from(activeUsers)
  .leftJoin(recentPosts, eq(activeUsers.id, recentPosts.authorId));
```

## Raw SQL

### Safe Raw Queries

```typescript
import { sql } from 'drizzle-orm';

const userId = 123;
const user = await db.execute(
  sql`SELECT * FROM ${users} WHERE ${users.id} = ${userId}`
);

const result = await db.execute<{ count: number }>(
  sql`SELECT COUNT(*) as count FROM ${users}`
);
```

### SQL Template Composition

```typescript
function whereActive() {
  return sql`${users.isActive} = true`;
}

function whereRole(role: string) {
  return sql`${users.role} = ${role}`;
}

const admins = await db
  .select()
  .from(users)
  .where(sql`${whereActive()} AND ${whereRole('admin')}`);
```

### Dynamic WHERE Clauses

```typescript
import { and, SQL } from 'drizzle-orm';

interface Filters {
  name?: string;
  role?: string;
  isActive?: boolean;
}

function buildFilters(filters: Filters): SQL | undefined {
  const conditions: SQL[] = [];

  if (filters.name) {
    conditions.push(like(users.name, `%${filters.name}%`));
  }

  if (filters.role) {
    conditions.push(eq(users.role, filters.role));
  }

  if (filters.isActive !== undefined) {
    conditions.push(eq(users.isActive, filters.isActive));
  }

  return conditions.length > 0 ? and(...conditions) : undefined;
}

const filters: Filters = { name: 'John', isActive: true };
const users = await db
  .select()
  .from(users)
  .where(buildFilters(filters));
```

## Relational Query Patterns

### Filtering by Relations

Find users who have posts matching certain criteria:

```typescript
const usersWithMatchingPosts = await db.query.users.findMany({
  where: {
    posts: {
      content: { like: 'TypeScript%' },
    },
  },
});
```

### Complex Relational Filters

```typescript
const results = await db.query.users.findMany({
  where: {
    AND: [
      { id: { gt: 10 } },
      { posts: { content: { like: 'M%' } } },
    ],
  },
  with: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      limit: 5,
    },
  },
});
```

### Nested Relations with Offset

```typescript
const postsWithComments = await db.query.posts.findMany({
  limit: 10,
  offset: 5,
  with: {
    comments: {
      offset: 0,
      limit: 3,
      orderBy: { createdAt: 'desc' },
    },
    author: true,
  },
});
```

## Aggregations

### Basic Aggregates

```typescript
import { count, sum, avg, min, max, sql } from 'drizzle-orm';

const userCount = await db.select({ count: count() }).from(users);
const totalRevenue = await db.select({ total: sum(orders.amount) }).from(orders);
const avgPrice = await db.select({ avg: avg(products.price) }).from(products);

const stats = await db
  .select({
    count: count(),
    total: sum(orders.amount),
    avg: avg(orders.amount),
    min: min(orders.amount),
    max: max(orders.amount),
  })
  .from(orders);
```

### GROUP BY with HAVING

```typescript
const prolificAuthors = await db
  .select({
    author: authors.name,
    postCount: count(posts.id),
  })
  .from(authors)
  .leftJoin(posts, eq(authors.id, posts.authorId))
  .groupBy(authors.id)
  .having(sql`COUNT(${posts.id}) > 5`);
```

### Window Functions

```typescript
const rankedProducts = await db
  .select({
    product: products,
    priceRank: sql<number>`RANK() OVER (PARTITION BY ${products.categoryId} ORDER BY ${products.price} DESC)`,
  })
  .from(products);

const ordersWithRunningTotal = await db
  .select({
    order: orders,
    runningTotal: sql<number>`SUM(${orders.amount}) OVER (ORDER BY ${orders.createdAt})`,
  })
  .from(orders);
```

## Prepared Statements

```typescript
const getUserById = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('get_user_by_id');

const user1 = await getUserById.execute({ id: 1 });
const user2 = await getUserById.execute({ id: 2 });
```

## Batch Operations

### Batch Insert

```typescript
const newUsers = await db.insert(users).values([
  { email: 'user1@example.com', name: 'User 1' },
  { email: 'user2@example.com', name: 'User 2' },
  { email: 'user3@example.com', name: 'User 3' },
]).returning();

// Upsert
await db.insert(users)
  .values(bulkUsers)
  .onConflictDoUpdate({
    target: users.email,
    set: { name: sql`EXCLUDED.name` },
  });
```

### Batch Delete

```typescript
await db.delete(users).where(inArray(users.id, [1, 2, 3, 4, 5]));
```

## UNION Queries

```typescript
const allContent = await db
  .select({ id: posts.id, title: posts.title, type: sql<string>`'post'` })
  .from(posts)
  .union(
    db.select({ id: articles.id, title: articles.title, type: sql<string>`'article'` })
      .from(articles)
  );
```

## Distinct Queries

```typescript
const uniqueRoles = await db.selectDistinct({ role: users.role }).from(users);

// DISTINCT ON (PostgreSQL)
const latestPostPerAuthor = await db
  .selectDistinctOn([posts.authorId], { post: posts })
  .from(posts)
  .orderBy(posts.authorId, desc(posts.createdAt));
```

## Locking Strategies

```typescript
// FOR UPDATE (pessimistic locking)
await db.transaction(async (tx) => {
  const user = await tx
    .select()
    .from(users)
    .where(eq(users.id, userId))
    .for('update');

  await tx.update(users)
    .set({ balance: user.balance - amount })
    .where(eq(users.id, userId));
});

// SKIP LOCKED
const availableTask = await db
  .select()
  .from(tasks)
  .where(eq(tasks.status, 'pending'))
  .limit(1)
  .for('update', { skipLocked: true });
```

## Best Practices

### Avoid N+1 Queries

```typescript
// Use relational queries
const authorsWithPosts = await db.query.authors.findMany({
  with: { posts: true },
});

// Or use DataLoader pattern
import DataLoader from 'dataloader';

const postLoader = new DataLoader(async (authorIds: number[]) => {
  const posts = await db.select().from(posts).where(inArray(posts.authorId, authorIds));
  return authorIds.map(id => posts.filter(post => post.authorId === id));
});
```
