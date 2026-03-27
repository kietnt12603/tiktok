---
name: database-operations
description: Hướng dẫn thao tác database MongoDB + Prisma (thêm model, sửa schema, seed data) — KHÔNG dùng migration
---

# Database Operations — MongoDB + Prisma

Skill này hướng dẫn mọi thao tác liên quan đến database trong dự án TikTok Clone.

## ⚠️ QUY TẮC SẮT

> **TUYỆT ĐỐI KHÔNG** tạo file migration (`prisma/migrations/`).
> **TUYỆT ĐỐI KHÔNG** chạy `prisma migrate dev` hoặc `prisma migrate deploy`.
> **CHỈ** dùng `prisma db push` để đồng bộ schema vào MongoDB.

## Workflow thay đổi schema

### 1. Thêm model mới

Mở file `apps/api/prisma/schema.prisma` và thêm model:

```prisma
model NewModel {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  // Các trường dữ liệu...
  userId    String   @db.ObjectId  // Foreign key luôn cần @db.ObjectId
  createdAt DateTime @default(now())

  // Relations
  user      User     @relation(fields: [userId], references: [id])

  // Indexes
  @@index([userId])
  @@map("new_models")  // Tên collection trong MongoDB
}
```

**Checklist cho model mới:**
- [ ] ID dùng `@id @default(auto()) @map("_id") @db.ObjectId`
- [ ] Foreign key fields có `@db.ObjectId`
- [ ] Có `@@map("collection_name")` để đặt tên collection
- [ ] Có `@@index` cho các trường query thường xuyên
- [ ] Đã thêm relation ở cả 2 phía (model này VÀ model liên quan)

### 2. Push schema vào MongoDB

```bash
# Từ apps/api hoặc root
yarn db:push        # = npx prisma db push
yarn db:generate    # = npx prisma generate (cập nhật Prisma Client)
```

**Lưu ý:** Nếu thay đổi schema có thể mất data (xóa field, đổi tên field), `db push` sẽ cảnh báo. Trong development, chấp nhận `--accept-data-loss` nếu cần:
```bash
npx prisma db push --accept-data-loss
```

### 3. Sửa field trong model

Khi sửa field (thêm/xóa/đổi tên), luôn thực hiện 3 bước:

1. Sửa `schema.prisma`
2. Chạy `yarn db:push`
3. Chạy `yarn db:generate`

### 4. Thêm index

MongoDB cần indexes cho hiệu năng. Thêm vào model:

```prisma
model Video {
  // ...fields

  // Single field index
  @@index([userId])
  @@index([createdAt])

  // Compound index (query theo userId + createdAt)
  @@index([userId, createdAt])

  // Unique constraint
  @@unique([userId, videoId])

  // Text search index (cần tạo riêng qua MongoDB shell)
}
```

Sau đó chạy `yarn db:push` để tạo indexes.

## PrismaService setup (NestJS)

File `apps/api/src/prisma/prisma.service.ts`:

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

File `apps/api/src/prisma/prisma.module.ts`:

```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

## Seed Data

File `apps/api/prisma/seed.ts`:

```typescript
import { PrismaClient } from '@prisma/client';
import { hash } from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  // Xóa data cũ
  await prisma.notification.deleteMany();
  await prisma.bookmark.deleteMany();
  await prisma.like.deleteMany();
  await prisma.comment.deleteMany();
  await prisma.follow.deleteMany();
  await prisma.video.deleteMany();
  await prisma.sound.deleteMany();
  await prisma.user.deleteMany();

  // Tạo users
  const user1 = await prisma.user.create({
    data: {
      username: 'kiet_dev',
      email: 'kiet@example.com',
      passwordHash: await hash('password123', 10),
      displayName: 'Kiệt Dev',
      bio: 'Full-stack developer 🚀',
      isVerified: true,
    },
  });

  const user2 = await prisma.user.create({
    data: {
      username: 'test_user',
      email: 'test@example.com',
      passwordHash: await hash('password123', 10),
      displayName: 'Test User',
    },
  });

  console.log('✅ Seed data created successfully');
  console.log({ user1: user1.id, user2: user2.id });
}

main()
  .catch((e) => {
    console.error('❌ Seed failed:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Chạy seed:
```bash
yarn db:seed
```

## Prisma Studio

Mở giao diện web để xem và chỉnh sửa data trực tiếp:
```bash
yarn db:studio
# Mở browser tại http://localhost:5555
```

## Kiểm tra kết nối MongoDB

```bash
# Test connection
npx prisma db execute --stdin <<< 'db.runCommand({ ping: 1 })'

# Hoặc trong code
npx prisma db push --preview-feature
```

## MongoDB Atlas Setup (Production)

1. Tạo cluster tại https://cloud.mongodb.com
2. Tạo database user
3. Whitelist IP
4. Lấy connection string, thay vào `DATABASE_URL`:
```
mongodb+srv://<user>:<pass>@cluster0.xxxxx.mongodb.net/tiktok?retryWrites=true&w=majority
```

## Lưu ý quan trọng

- **Transactions cần Replica Set**: Nếu dùng MongoDB local, phải config replica set. MongoDB Atlas đã có sẵn.
- **ObjectId validation**: Luôn validate ObjectId trước khi query để tránh Prisma error.
- **Pagination**: Dùng skip/take cho cơ bản. Với dataset lớn, dùng cursor-based pagination.
- **N+1 queries**: Dùng Prisma `include` hoặc `select` để tránh N+1. Không query trong loop.
