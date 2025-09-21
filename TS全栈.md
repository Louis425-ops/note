# TS全栈

## 数据库配置

### 1. 数据模型定义 (`server/db/schema.ts`)

```typescript
import { sql } from "drizzle-orm";
import { foreignKey, int } from "drizzle-orm/mysql-core";
import {
  index,
  integer,
  pgTableCreator,
  serial,
  timestamp,
  varchar,
  text,
} from "drizzle-orm/pg-core";

export const createTable = pgTableCreator((name) => `chat-roomserver_${name}`);

export const users = createTable(
  "user",
  {
    uid: serial("uid").primaryKey(),
    userName: varchar("user_name", { length: 256 }).notNull().unique(),
    password: varchar("password", { length: 256 }).notNull(),
  },
)
```

### 2. 数据库连接 (`server/db/index.ts`)

```typescript
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import * as schema from "./schema";

const globalForDb = globalThis as unknown as {
  conn: postgres.Sql | undefined;
};

const conn = globalForDb.conn ?? postgres("postgresql://postgres:<pwd>@localhost:5432/chat-room-server");
globalForDb.conn = conn;

export const db = drizzle(conn, { schema });
```

## 数据库CRUD操作

### 查询操作

```typescript
// 基本查询
const existingUser = await db.query.users.findFirst({
  where: eq(users.userName, userName),
});

// 多条查询
const allUsers = await db.query.users.findMany();

// 关联查询（如果有关系）
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: true,
  },
});
```

### 插入操作

```typescript
const [newUser] = await db.insert(users).values({
  userName,
  password,
}).returning();
```

### 更新和删除

```typescript
// 更新
await db.update(users)
  .set({ password: newPassword })
  .where(eq(users.uid, userId));

// 删除
await db.delete(users)
  .where(eq(users.uid, userId));
```

------

## API路由开发

### 路由文件结构

```
src/app/api/users/register/route.ts  → POST /api/users/register
```

### 完整API示例 (`route.ts`)

```typescript
import { NextResponse } from "next/server";
import { z } from "zod";
import { db } from "@/server/db";
import { users } from "@/server/db/schema";
import { eq } from "drizzle-orm";

// Zod数据验证模式
const registerSchema = z.object({
  userName: z.string().min(1),
  password: z.string().min(1),
});

export async function POST(request: Request) {
  try {
    // 1. 解析请求体
    const body = await request.json();
    const { userName, password } = registerSchema.parse(body);

    // 2. 检查用户是否已存在
    const existingUser = await db.query.users.findFirst({
      where: eq(users.userName, userName),
    });

    if (existingUser) {
      return NextResponse.json(
        { error: "Username already taken" },
        { status: 409 }
      );
    }

    // 3. 创建新用户
    const [newUser] = await db.insert(users).values({
      userName,
      password, // 生产环境需要加密
    }).returning();

    // 4. 返回成功结果
    return NextResponse.json({
      uid: newUser.uid,
      userName: newUser.userName,
    });

  } catch (error) {
    // 5. 错误处理
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

// 其他HTTP方法
export async function GET(request: Request) {
  // GET请求处理逻辑
}

export async function PUT(request: Request) {
  // PUT请求处理逻辑
}
```

------

## 高级查询操作

### 条件查询和排序

```typescript
import { eq, desc, asc, lt } from 'drizzle-orm';

// 条件查询
const users = await db.query.users.findMany({
  where: eq(users.verified, true)
});

// 排序
const sortedUsers = await db.query.users.findMany({
  orderBy: [desc(users.createdAt)]
});

// 分页
const paginatedUsers = await db.query.users.findMany({
  limit: 10,
  offset: 20
});
```

### 部分字段查询

```typescript
// 只查询特定字段
const userNames = await db.query.users.findMany({
  columns: {
    userName: true,
    uid: true,
  },
});

// 排除某些字段
const usersWithoutPassword = await db.query.users.findMany({
  columns: {
    password: false,
  },
});
```

### 关联查询

```typescript
// 嵌套关联
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: {
      with: {
        comments: true,
      },
    },
  },
});

// 关联查询的条件过滤
const filteredData = await db.query.posts.findMany({
  where: eq(posts.published, true),
  with: {
    comments: {
      where: (comments, { lt }) => lt(comments.createdAt, new Date()),
      limit: 5,
    },
  },
});
```

## code

### 1. 通用错误处理函数

```typescript
export function handleApiError(error: unknown) {
  if (error instanceof z.ZodError) {
    return NextResponse.json(
      { error: "Validation failed", details: error.errors },
      { status: 400 }
    );
  }
  
  console.error("API Error:", error);
  return NextResponse.json(
    { error: "Internal server error" },
    { status: 500 }
  );
}
```

### 2. 统一响应格式

```typescript
export function createSuccessResponse(data: any, message?: string) {
  return NextResponse.json({
    success: true,
    data,
    message: message || "Operation successful"
  });
}

export function createErrorResponse(message: string, status = 400) {
  return NextResponse.json({
    success: false,
    error: message
  }, { status });
}
```

### 3. 预处理语句

```typescript
// 预处理查询
const findUserById = db.query.users.findFirst({
  where: eq(users.uid, placeholder('id')),
}).prepare('find_user_by_id');

// 执行预处理查询
const user = await findUserById.execute({ id: 123 });
```