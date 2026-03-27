---
name: project-setup
description: Khởi tạo toàn bộ dự án TikTok Clone từ đầu (monorepo, frontend, backend, prisma)
---

# Project Setup - TikTok Clone

Skill này hướng dẫn cách khởi tạo toàn bộ dự án TikTok Clone monorepo từ đầu.

## Điều kiện tiên quyết
- Node.js 24 đã cài
- Yarn v4 Berry (4.13.0) đã cài
- MongoDB đang chạy (local hoặc Atlas)
- Git đã cài

## Các bước thực hiện

### Bước 1: Khởi tạo monorepo root

```bash
# Tại thư mục g:\code\tiktok
yarn init -2

# Tạo file package.json root với workspaces
```

Cập nhật `package.json` root:
```json
{
  "name": "tiktok-clone",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,css,md}\"",
    "test": "turbo run test",
    "test:e2e": "turbo run test:e2e",
    "db:push": "yarn workspace api db:push",
    "db:generate": "yarn workspace api db:generate",
    "db:seed": "yarn workspace api db:seed",
    "db:studio": "yarn workspace api db:studio",
    "db:reset": "yarn workspace api db:reset"
  }
}
```

### Bước 2: Tạo frontend (Next.js 16)

```bash
# Tạo thư mục
mkdir -p apps/web

# Khởi tạo Next.js — luôn chạy --help trước để xem options
npx -y create-next-app@latest apps/web --typescript --app --src-dir --no-tailwind --eslint --use-yarn
```

**LƯU Ý**: Flag `--no-tailwind` là BẮT BUỘC. Dự án dùng Vanilla CSS / CSS Modules.

### Bước 3: Tạo backend (NestJS)

```bash
# Tạo NestJS project
npx -y @nestjs/cli new apps/api --package-manager yarn --strict --skip-git
```

Sau đó cài thêm các dependencies cho backend:
```bash
cd apps/api
yarn add @prisma/client mongoose
yarn add -D prisma
yarn add @nestjs/config @nestjs/jwt @nestjs/passport passport passport-jwt
yarn add -D @types/passport-jwt
yarn add class-validator class-transformer
yarn add @nestjs/platform-socket.io @nestjs/websockets socket.io
```

### Bước 4: Khởi tạo Prisma cho MongoDB

```bash
cd apps/api
npx prisma init --datasource-provider mongodb
```

Cập nhật file `apps/api/prisma/schema.prisma` theo schema đã định nghĩa trong `claude.md`.

Thêm scripts vào `apps/api/package.json`:
```json
{
  "scripts": {
    "db:push": "prisma db push",
    "db:generate": "prisma generate",
    "db:seed": "ts-node prisma/seed.ts",
    "db:studio": "prisma studio",
    "db:reset": "prisma db push --force-reset && ts-node prisma/seed.ts"
  }
}
```

### Bước 5: Tạo file `.env`

Tạo file `apps/api/.env`:
```env
DATABASE_URL="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/tiktok?retryWrites=true&w=majority"
JWT_SECRET="your-super-secret-jwt-key"
JWT_REFRESH_SECRET="your-super-secret-refresh-key"
REDIS_URL="redis://localhost:6379"
S3_BUCKET="tiktok-videos"
S3_REGION="ap-southeast-1"
S3_ACCESS_KEY=""
S3_SECRET_KEY=""
PORT=4000
```

Tạo file `apps/web/.env.local`:
```env
NEXT_PUBLIC_API_URL="http://localhost:4000/api/v1"
NEXT_PUBLIC_WS_URL="http://localhost:4000"
```

### Bước 6: Cài Turborepo

```bash
# Ở root
yarn add -D turbo
```

Tạo file `turbo.json`:
```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "dev": {
      "persistent": true,
      "cache": false
    },
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "lint": {},
    "test": {}
  }
}
```

### Bước 7: Push schema vào MongoDB

```bash
# ⚠️ KHÔNG DÙNG prisma migrate — Chỉ dùng db push cho MongoDB
cd apps/api
npx prisma db push
npx prisma generate
```

### Bước 8: Chạy dev server

```bash
# Ở root, chạy tất cả
yarn dev

# Hoặc chạy riêng
yarn workspace web dev     # Frontend: http://localhost:3000
yarn workspace api dev     # Backend:  http://localhost:4000
```

## Kiểm tra sau khi setup
- [ ] `yarn install` chạy không lỗi
- [ ] `yarn workspace web dev` mở được http://localhost:3000
- [ ] `yarn workspace api dev` mở được http://localhost:4000
- [ ] `yarn db:push` đồng bộ schema vào MongoDB thành công
- [ ] `yarn db:studio` mở được Prisma Studio
