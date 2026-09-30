# Part 40: Next.js Database + Prisma ORM

## ข้อมูล Part
- **Steps:** 1191-1235
- **ระดับ:** Intermediate to Advanced
- **เวลาเรียน:** 4 ชั่วโมง
- **Prerequisites:** Part 39 (Authentication)

---

## สารบัญ

1. [Prisma ORM คืออะไร](#1-prisma-orm-คืออะไร)
2. [Schema Definition](#2-schema-definition)
3. [Migrations](#3-migrations)
4. [CRUD Operations](#4-crud-operations)
5. [Relations (One-to-Many, Many-to-Many)](#5-relations)
6. [Prisma Client](#6-prisma-client)
7. [Connection Pooling](#7-connection-pooling)
8. [ตัวอย่าง Blog Database Schema](#8-ตัวอย่าง-blog-database-schema)
9. [Seeding Data](#9-seeding-data)
10. [Quiz](#quiz)

---

## Step 1191: Prisma ORM คืออะไร

### 1. Prisma ORM คืออะไร

**Prisma** คือ Next-generation ORM (Object-Relational Mapping) สำหรับ Node.js และ TypeScript

#### ทำไมต้องใช้ Prisma

```
SQL แบบดั้งเดิม:
const users = await db.query('SELECT * FROM users WHERE id = $1', [userId])
// ❌ ไม่มี Type Safety
// ❌ SQL Injection ถ้าระวังไม่ดี
// ❌ Refactoring ยาก

Prisma:
const user = await prisma.user.findUnique({ where: { id: userId } })
// ✓ Type-safe
// ✓ Auto-complete
// ✓ Refactoring ง่าย
// ✓ Migration Management
```

#### Prisma Components

```
Prisma Schema  → Define Data Model
Prisma Client  → Type-safe Database Client
Prisma Migrate → Database Migration Tool
Prisma Studio  → Database GUI (optional)
```

#### การติดตั้ง

```bash
# ติดตั้ง Prisma
npm install prisma --save-dev
npm install @prisma/client

# Initialize Prisma
npx prisma init

# Initialize กับ Database ที่เลือก
npx prisma init --datasource-provider postgresql
npx prisma init --datasource-provider mysql
npx prisma init --datasource-provider sqlite
npx prisma init --datasource-provider mongodb
```

#### โครงสร้างหลัง Init

```
project/
├── prisma/
│   └── schema.prisma   ← Prisma Schema
├── .env                ← DATABASE_URL
└── ...
```

---

## Step 1194: Schema Definition

### 2. Schema Definition

#### Prisma Schema Basics

```prisma
// prisma/schema.prisma

// Generator สำหรับสร้าง Prisma Client
generator client {
  provider = "prisma-client-js"
}

// Database Connection
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Model = Database Table
model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  password  String?
  image     String?
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// Enum
enum Role {
  USER
  MODERATOR
  ADMIN
}
```

#### Field Types

```prisma
model Example {
  // Primary Key
  id        Int      @id @default(autoincrement())
  // หรือ
  id        String   @id @default(cuid())
  // หรือ
  id        String   @id @default(uuid())
  
  // Scalar Types
  name      String           // VARCHAR
  bio       String?          // nullable
  content   String  @db.Text // TEXT (ยาว)
  age       Int              // INTEGER
  price     Float            // DOUBLE PRECISION
  amount    Decimal          // DECIMAL
  isActive  Boolean          // BOOLEAN
  
  // Date/Time
  createdAt DateTime @default(now())    // TIMESTAMP
  updatedAt DateTime @updatedAt         // Auto-update
  birthday  DateTime?                   // nullable
  
  // JSON
  metadata  Json?            // JSON
  
  // Bytes
  file      Bytes?           // BYTEA
}
```

#### Field Attributes

```prisma
model User {
  id       String  @id @default(cuid())
  email    String  @unique
  username String  @unique
  name     String  @default("Unknown")
  bio      String?
  
  // Index
  searchKey String @index
  
  // Composite Unique
  firstName String
  lastName  String
  @@unique([firstName, lastName])
  
  // Composite Index
  @@index([firstName, lastName])
}
```

---

## Step 1197: Migrations

### 3. Migrations

#### สร้าง Migration ครั้งแรก

```bash
# สร้าง Migration
npx prisma migrate dev --name init

# จะสร้าง:
# prisma/migrations/
# ├── 20240101000000_init/
# │   └── migration.sql
# └── migration_lock.toml
```

#### Migration Workflow

```bash
# Development - สร้าง Migration และ Apply
npx prisma migrate dev --name add_user_role

# Production - Apply Migrations
npx prisma migrate deploy

# Reset Database (Development เท่านั้น)
npx prisma migrate reset

# ดู Migration Status
npx prisma migrate status

# สร้าง Prisma Client (หลัง Schema เปลี่ยน)
npx prisma generate
```

#### Migration File

```sql
-- prisma/migrations/20240101000000_init/migration.sql
-- CreateEnum
CREATE TYPE "Role" AS ENUM ('USER', 'MODERATOR', 'ADMIN');

-- CreateTable
CREATE TABLE "User" (
    "id" TEXT NOT NULL,
    "name" TEXT NOT NULL,
    "email" TEXT NOT NULL,
    "password" TEXT,
    "image" TEXT,
    "role" "Role" NOT NULL DEFAULT 'USER',
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");
```

---

## Step 1200: CRUD Operations

### 4. CRUD Operations

#### Create

```typescript
// สร้าง Record เดียว
const user = await prisma.user.create({
  data: {
    name: 'Alice',
    email: 'alice@example.com',
    role: 'USER',
  },
})

// สร้าง Record พร้อม Relations
const post = await prisma.post.create({
  data: {
    title: 'My Post',
    content: 'Content here',
    author: {
      connect: { id: userId }  // ต่อกับ User ที่มีอยู่
      // หรือ
      create: {  // สร้าง User ใหม่พร้อมกัน
        name: 'Bob',
        email: 'bob@example.com',
      }
    },
    tags: {
      connectOrCreate: [
        {
          where: { name: 'nextjs' },
          create: { name: 'nextjs' },
        }
      ]
    }
  },
  include: { author: true, tags: true },
})

// สร้างหลาย Records พร้อมกัน
const users = await prisma.user.createMany({
  data: [
    { name: 'Alice', email: 'alice@example.com' },
    { name: 'Bob', email: 'bob@example.com' },
  ],
  skipDuplicates: true,  // ไม่ Error ถ้า Email ซ้ำ
})
```

#### Read

```typescript
// หา Record เดียว (ต้องเป็น Unique Field)
const user = await prisma.user.findUnique({
  where: { id: 'user-id' },
  // หรือ
  where: { email: 'alice@example.com' },
})

// หา Record เดียว (ใช้ Field อะไรก็ได้)
const user = await prisma.user.findFirst({
  where: { name: { contains: 'Alice' } },
  orderBy: { createdAt: 'desc' },
})

// หาหลาย Records
const users = await prisma.user.findMany({
  where: {
    role: 'USER',
    createdAt: {
      gte: new Date('2024-01-01'),
    },
  },
  orderBy: { createdAt: 'desc' },
  skip: 0,
  take: 10,
  select: {
    id: true,
    name: true,
    email: true,
    // ไม่เลือก password
  },
  include: {
    posts: {
      where: { published: true },
      take: 5,
    }
  }
})

// นับ Records
const count = await prisma.user.count({
  where: { role: 'USER' }
})

// Aggregate
const stats = await prisma.order.aggregate({
  where: { userId: 'user-id' },
  _sum: { amount: true },
  _avg: { amount: true },
  _min: { amount: true },
  _max: { amount: true },
  _count: true,
})
```

#### Update

```typescript
// Update Record เดียว
const user = await prisma.user.update({
  where: { id: 'user-id' },
  data: {
    name: 'Alice Updated',
    updatedAt: new Date(),
  },
})

// Update หลาย Records
const result = await prisma.user.updateMany({
  where: { role: 'USER' },
  data: { updatedAt: new Date() },
})

// Upsert (Update หรือ Create ถ้าไม่มี)
const user = await prisma.user.upsert({
  where: { email: 'alice@example.com' },
  update: { name: 'Alice Updated' },
  create: {
    email: 'alice@example.com',
    name: 'Alice',
  },
})

// Increment/Decrement
await prisma.post.update({
  where: { id: postId },
  data: {
    viewCount: { increment: 1 },
    likeCount: { decrement: 1 },
  }
})
```

#### Delete

```typescript
// Delete Record เดียว
const user = await prisma.user.delete({
  where: { id: 'user-id' },
})

// Delete หลาย Records
const result = await prisma.user.deleteMany({
  where: {
    createdAt: { lt: new Date('2020-01-01') },
    role: 'USER',
  }
})
```

#### Filter Operators

```typescript
// Comparison
where: { age: { gt: 18, lt: 65 } }  // > 18 AND < 65
where: { age: { gte: 18, lte: 65 } } // >= 18 AND <= 65
where: { age: { not: 25 } }           // ≠ 25

// String
where: { name: { contains: 'Alice' } }
where: { name: { startsWith: 'A' } }
where: { name: { endsWith: 'e' } }
where: { name: { mode: 'insensitive', contains: 'alice' } }

// Array (IN, NOT IN)
where: { role: { in: ['USER', 'MODERATOR'] } }
where: { role: { notIn: ['ADMIN'] } }

// Null Check
where: { deletedAt: null }
where: { deletedAt: { not: null } }

// Logical
where: {
  AND: [{ role: 'USER' }, { active: true }]
}
where: {
  OR: [{ role: 'ADMIN' }, { email: { endsWith: '@company.com' } }]
}
where: {
  NOT: { role: 'ADMIN' }
}
```

---

## Step 1204: Relations

### 5. Relations

#### One-to-Many (1:N)

```prisma
// prisma/schema.prisma
model User {
  id    String @id @default(cuid())
  name  String
  email String @unique
  
  // One User has Many Posts
  posts Post[]
}

model Post {
  id       String @id @default(cuid())
  title    String
  content  String
  authorId String
  
  // Many Posts belong to One User
  author User @relation(fields: [authorId], references: [id])
}
```

```typescript
// Query with One-to-Many
const userWithPosts = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 10,
    }
  }
})

// หา Posts พร้อม Author
const posts = await prisma.post.findMany({
  include: {
    author: {
      select: { id: true, name: true, image: true }
    }
  }
})
```

#### Many-to-Many (N:M)

```prisma
// implicit Many-to-Many (Prisma จัดการ Junction Table)
model Post {
  id   String  @id @default(cuid())
  title String
  tags Tag[]
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  posts Post[]
}

// explicit Many-to-Many (กำหนด Junction Table เอง)
model Post {
  id   String   @id @default(cuid())
  title String
  tags PostTag[]
}

model Tag {
  id    String   @id @default(cuid())
  name  String   @unique
  posts PostTag[]
}

model PostTag {
  postId    String
  tagId     String
  createdAt DateTime @default(now())
  
  post Post @relation(fields: [postId], references: [id])
  tag  Tag  @relation(fields: [tagId], references: [id])
  
  @@id([postId, tagId])
}
```

```typescript
// Query Many-to-Many
const post = await prisma.post.findUnique({
  where: { id: postId },
  include: {
    tags: true,  // implicit
    // หรือ
    tags: {
      include: { tag: true }  // explicit
    }
  }
})

// เพิ่ม Tag ไปยัง Post
await prisma.post.update({
  where: { id: postId },
  data: {
    tags: {
      connect: { id: tagId },
      // หรือสร้างใหม่
      connectOrCreate: {
        where: { name: 'nextjs' },
        create: { name: 'nextjs' },
      }
    }
  }
})
```

#### One-to-One (1:1)

```prisma
model User {
  id      String   @id @default(cuid())
  email   String   @unique
  profile Profile?
}

model Profile {
  id     String @id @default(cuid())
  bio    String?
  avatar String?
  userId String @unique
  
  user User @relation(fields: [userId], references: [id])
}
```

#### Self-Relation

```prisma
// Nested Comments (Parent-Child)
model Comment {
  id       String    @id @default(cuid())
  content  String
  parentId String?
  parent   Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  replies  Comment[] @relation("CommentReplies")
}

// User Follows (N:M Self-relation)
model User {
  id        String @id @default(cuid())
  name      String
  following User[] @relation("UserFollows")
  followers User[] @relation("UserFollows")
}
```

---

## Step 1208: Prisma Client

### 6. Prisma Client

#### Setup

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development'
      ? ['query', 'error', 'warn']
      : ['error'],
  })

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

#### Transactions

```typescript
// Sequential Transactions
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { name: 'Alice', email: 'alice@example.com' } }),
  prisma.post.create({ data: { title: 'Hello', authorId: 'user-id' } }),
])

// Interactive Transactions (ซับซ้อนกว่า)
const result = await prisma.$transaction(async (tx) => {
  // ตรวจสอบ balance
  const user = await tx.user.findUnique({ where: { id: userId } })
  if (user.balance < amount) {
    throw new Error('Insufficient balance')
  }
  
  // ลด balance
  const updated = await tx.user.update({
    where: { id: userId },
    data: { balance: { decrement: amount } }
  })
  
  // สร้าง transaction record
  const transaction = await tx.transaction.create({
    data: { userId, amount, type: 'DEBIT' }
  })
  
  return { user: updated, transaction }
})
```

#### Raw SQL

```typescript
// Raw Query
const users = await prisma.$queryRaw`
  SELECT * FROM "User"
  WHERE email = ${email}
  LIMIT 10
`

// Raw Execute (INSERT, UPDATE, DELETE)
const result = await prisma.$executeRaw`
  UPDATE "User" 
  SET "updatedAt" = NOW()
  WHERE id = ${userId}
`

// ระวัง SQL Injection - ใช้ Template Literals เสมอ
// ❌ อันตราย
const users = await prisma.$queryRawUnsafe(
  `SELECT * FROM "User" WHERE email = '${email}'`
)

// ✅ ปลอดภัย
const users = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE email = ${email}
`
```

#### Prisma Studio

```bash
# เปิด Prisma Studio (Database GUI)
npx prisma studio
# เปิดที่ http://localhost:5555
```

---

## Step 1212: Connection Pooling

### 7. Connection Pooling

#### ปัญหาของ Serverless

```
Serverless/Edge Functions สร้าง Connection ใหม่ทุกครั้ง
จำนวน Connection อาจเกิน Limit ของ Database

การแก้ไข:
1. PgBouncer (สำหรับ PostgreSQL)
2. Prisma Data Proxy
3. Neon / PlanetScale (Serverless Databases)
4. Supabase (มี Connection Pooling Built-in)
```

#### Prisma Data Proxy / Accelerate

```bash
# ติดตั้ง Prisma Client Extension
npm install @prisma/extension-accelerate

# .env
DATABASE_URL="prisma://accelerate.prisma-data.net/?api_key=YOUR_KEY"
```

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'
import { withAccelerate } from '@prisma/extension-accelerate'

const prisma = new PrismaClient().$extends(withAccelerate())

// ใช้งาน Caching
const users = await prisma.user.findMany({
  cacheStrategy: {
    ttl: 60,  // Cache 60 วินาที
    swr: 30,  // Stale-while-revalidate 30 วินาที
  }
})
```

#### Connection Pooling ด้วย pgBouncer

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:6432/mydb"  # ผ่าน pgBouncer

# pgBouncer Config
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
```

#### Neon Serverless

```bash
npm install @neondatabase/serverless

# .env
DATABASE_URL="postgresql://user:password@ep-xxx.us-east-2.aws.neon.tech/mydb?sslmode=require"
```

```typescript
// lib/prisma.ts (for Vercel Edge)
import { Pool, neonConfig } from '@neondatabase/serverless'
import { PrismaNeon } from '@prisma/adapter-neon'
import { PrismaClient } from '@prisma/client'
import ws from 'ws'

neonConfig.webSocketConstructor = ws

const pool = new Pool({ connectionString: process.env.DATABASE_URL })
const adapter = new PrismaNeon(pool)
export const prisma = new PrismaClient({ adapter })
```

---

## Step 1216: Blog Database Schema

### 8. ตัวอย่าง Blog Database Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// =========================================
// Enums
// =========================================
enum Role {
  USER
  MODERATOR
  ADMIN
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

// =========================================
// Models
// =========================================
model User {
  id            String    @id @default(cuid())
  name          String
  email         String    @unique
  emailVerified DateTime?
  password      String?
  image         String?
  bio           String?
  role          Role      @default(USER)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Relations
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  accounts      Account[]
  sessions      Session[]
  profile       Profile?
  
  @@index([email])
}

model Profile {
  id      String  @id @default(cuid())
  website String?
  twitter String?
  github  String?
  userId  String  @unique
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Category {
  id          String @id @default(cuid())
  name        String @unique
  slug        String @unique
  description String?
  image       String?
  
  posts Post[]
}

model Post {
  id          String     @id @default(cuid())
  title       String
  slug        String     @unique
  excerpt     String?    @db.Text
  content     String     @db.Text
  contentHtml String?    @db.Text
  coverImage  String?
  status      PostStatus @default(DRAFT)
  published   Boolean    @default(false)
  publishedAt DateTime?
  viewCount   Int        @default(0)
  
  authorId    String
  categoryId  String
  
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  
  // Relations
  author      User       @relation(fields: [authorId], references: [id])
  category    Category   @relation(fields: [categoryId], references: [id])
  comments    Comment[]
  tags        Tag[]
  likes       Like[]
  
  @@index([slug])
  @@index([authorId])
  @@index([categoryId])
  @@index([publishedAt])
  @@index([status, publishedAt])
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  
  posts Post[]
}

model Comment {
  id        String    @id @default(cuid())
  content   String    @db.Text
  parentId  String?
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  
  authorId  String
  postId    String
  
  // Relations
  author    User      @relation(fields: [authorId], references: [id])
  post      Post      @relation(fields: [postId], references: [id], onDelete: Cascade)
  parent    Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  replies   Comment[] @relation("CommentReplies")
  
  @@index([postId])
  @@index([authorId])
}

model Like {
  userId    String
  postId    String
  createdAt DateTime @default(now())
  
  user User @relation(fields: [userId], references: [id])
  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
  
  @@id([userId, postId])
}

// NextAuth Models
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime
  
  @@unique([identifier, token])
}
```

---

## Step 1220: Seeding Data

### 9. Seeding Data

#### สร้าง Seed File

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client'
import bcrypt from 'bcryptjs'

const prisma = new PrismaClient()

async function main() {
  console.log('Start seeding...')
  
  // Clean existing data
  await prisma.like.deleteMany()
  await prisma.comment.deleteMany()
  await prisma.post.deleteMany()
  await prisma.tag.deleteMany()
  await prisma.category.deleteMany()
  await prisma.profile.deleteMany()
  await prisma.user.deleteMany()
  
  // Create Admin User
  const adminPassword = await bcrypt.hash('Admin1234!', 12)
  const admin = await prisma.user.create({
    data: {
      name: 'Admin User',
      email: 'admin@example.com',
      password: adminPassword,
      role: 'ADMIN',
      emailVerified: new Date(),
      profile: {
        create: {
          website: 'https://myapp.com',
          github: 'adminuser',
        }
      }
    }
  })
  console.log('Created admin:', admin.email)
  
  // Create Regular Users
  const userPassword = await bcrypt.hash('User1234!', 12)
  const users = await Promise.all([
    prisma.user.create({
      data: {
        name: 'Alice Smith',
        email: 'alice@example.com',
        password: userPassword,
        emailVerified: new Date(),
        bio: 'Frontend Developer',
      }
    }),
    prisma.user.create({
      data: {
        name: 'Bob Johnson',
        email: 'bob@example.com',
        password: userPassword,
        emailVerified: new Date(),
        bio: 'Backend Developer',
      }
    }),
  ])
  console.log('Created users:', users.map(u => u.email).join(', '))
  
  // Create Categories
  const categories = await Promise.all([
    prisma.category.create({
      data: { name: 'Next.js', slug: 'nextjs', description: 'Next.js Tutorials' }
    }),
    prisma.category.create({
      data: { name: 'React', slug: 'react', description: 'React Tutorials' }
    }),
    prisma.category.create({
      data: { name: 'TypeScript', slug: 'typescript', description: 'TypeScript Tips' }
    }),
  ])
  console.log('Created categories:', categories.length)
  
  // Create Tags
  const tags = await Promise.all([
    prisma.tag.create({ data: { name: 'nextjs', slug: 'nextjs' } }),
    prisma.tag.create({ data: { name: 'react', slug: 'react' } }),
    prisma.tag.create({ data: { name: 'typescript', slug: 'typescript' } }),
    prisma.tag.create({ data: { name: 'tutorial', slug: 'tutorial' } }),
    prisma.tag.create({ data: { name: 'beginner', slug: 'beginner' } }),
  ])
  console.log('Created tags:', tags.length)
  
  // Create Posts
  const posts = await Promise.all([
    prisma.post.create({
      data: {
        title: 'Getting Started with Next.js 14',
        slug: 'getting-started-nextjs-14',
        excerpt: 'Learn Next.js 14 from scratch',
        content: '# Getting Started\n\nNext.js is a React framework...',
        contentHtml: '<h1>Getting Started</h1><p>Next.js is a React framework...</p>',
        status: 'PUBLISHED',
        published: true,
        publishedAt: new Date(),
        authorId: users[0].id,
        categoryId: categories[0].id,
        tags: {
          connect: [
            { id: tags[0].id },
            { id: tags[3].id },
            { id: tags[4].id },
          ]
        }
      }
    }),
    prisma.post.create({
      data: {
        title: 'Understanding Server Components',
        slug: 'understanding-server-components',
        excerpt: 'Deep dive into React Server Components',
        content: '# Server Components\n\nServer Components allow...',
        contentHtml: '<h1>Server Components</h1>',
        status: 'PUBLISHED',
        published: true,
        publishedAt: new Date(Date.now() - 86400000),
        authorId: users[1].id,
        categoryId: categories[0].id,
        tags: {
          connect: [
            { id: tags[0].id },
            { id: tags[1].id },
          ]
        }
      }
    }),
  ])
  console.log('Created posts:', posts.length)
  
  // Create Comments
  await Promise.all(
    posts.map(post =>
      prisma.comment.create({
        data: {
          content: 'Great post! Very helpful.',
          authorId: users[0].id,
          postId: post.id,
        }
      })
    )
  )
  console.log('Created comments')
  
  // Create Likes
  await Promise.all(
    posts.map(post =>
      prisma.like.create({
        data: {
          userId: users[0].id,
          postId: post.id,
        }
      })
    )
  )
  console.log('Created likes')
  
  console.log('Seeding finished!')
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect())
```

#### Setup Seed ใน package.json

```json
{
  "prisma": {
    "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  }
}
```

```bash
# รัน Seed
npx prisma db seed

# หรือ reset แล้ว seed ใหม่
npx prisma migrate reset  # จะถาม confirm
```

---

## Step 1228: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. ใช้ select แทน include เมื่อทำได้

✓ select ดึงเฉพาะ Fields ที่ต้องการ = เร็วกว่า
✓ include ดึงทุก Fields ของ Relation
✓ ระวัง Over-fetching

## 2. Pagination

✓ ใช้ skip/take สำหรับ Offset Pagination
✓ ใช้ cursor-based สำหรับ Real-time Data
✓ ใช้ count() ร่วมกับ findMany()

## 3. Indexes

✓ เพิ่ม @index สำหรับ Fields ที่ Query บ่อย
✓ Composite Index สำหรับ Query หลาย Fields
✓ @unique สร้าง Index อัตโนมัติ

## 4. N+1 Problem

✓ ใช้ include แทน Loop Query
✗ อย่า Query Database ใน Loop
✓ ใช้ Prisma's dataloader pattern

## 5. Transactions

✓ ใช้ Transaction เมื่อมี Multiple Writes ที่ต้อง Atomic
✓ $transaction([]) สำหรับ Simple Cases
✓ $transaction(async tx => {}) สำหรับ Complex Cases
```

---

## Quiz

### แบบทดสอบ Part 40

**คำถามที่ 1:** ข้อดีของ Prisma ORM เมื่อเทียบกับ Raw SQL คืออะไร?
- A) เร็วกว่าเสมอ
- B) Type-safety, Auto-complete, Migration Management ✓
- C) ใช้ SQL Syntax เหมือนเดิม
- D) ไม่ต้องมี Database

**คำถามที่ 2:** `prisma migrate dev` ต่างจาก `prisma migrate deploy` อย่างไร?
- A) ไม่ต่างกัน
- B) dev ใช้ใน Development (สร้าง Migration), deploy ใช้ใน Production (Apply) ✓
- C) deploy เร็วกว่า
- D) dev ใช้สำหรับ Production

**คำถามที่ 3:** `findUnique` ต่างจาก `findFirst` อย่างไร?
- A) findUnique เร็วกว่า
- B) findUnique ใช้เฉพาะ Unique/PK Fields, findFirst ใช้ Field ใดก็ได้ ✓
- C) findFirst ค้นหา Records ทั้งหมด
- D) ไม่ต่างกัน

**คำถามที่ 4:** N+1 Problem คืออะไร?
- A) การ Query หลาย Tables
- B) Query Parent แล้ว Loop Query Children ทีละคน ทำให้ Query จำนวนมาก ✓
- C) การใช้หลาย Database
- D) Database Error

**คำถามที่ 5:** ทำไม Serverless Functions ถึงมีปัญหา Connection Pooling?
- A) Serverless ช้ากว่า Server ปกติ
- B) Serverless สร้าง Database Connection ใหม่ทุกครั้ง อาจ Exceed Connection Limit ✓
- C) Serverless ไม่รองรับ Database
- D) Connection Pooling ใช้ไม่ได้กับ Prisma

---

## สรุป Part 40

ใน Part นี้เราได้เรียนรู้:

1. **Prisma ORM** - Type-safe Database Access
2. **Schema** - Models, Fields, Enums
3. **Migrations** - dev, deploy, reset
4. **CRUD** - Create, Read, Update, Delete
5. **Relations** - 1:1, 1:N, N:M, Self-relation
6. **Client** - Transactions, Raw SQL
7. **Connection Pooling** - Serverless Solutions
8. **Blog Schema** - ตัวอย่างสมบูรณ์
9. **Seeding** - สร้าง Test Data

---

➡️ **Part ถัดไป:** [Part 41: Next.js Deployment](./part-41-nextjs-deployment.md)
