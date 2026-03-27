# TikTok Clone - Project Guidelines

## Tổng quan dự án

- **Repository**: https://github.com/kietnt12603/tiktok.git

Đây là ứng dụng **TikTok Clone** — một nền tảng chia sẻ video ngắn với đầy đủ tính năng tương tự TikTok. Ứng dụng hỗ trợ xem video dạng cuộn dọc (vertical scroll), tương tác (like, comment, share), tạo và đăng video, theo dõi người dùng, và khám phá nội dung.

---

## Tech Stack

### Frontend
- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: CSS Modules + Vanilla CSS (không dùng TailwindCSS trừ khi được yêu cầu rõ ràng)
- **State Management**: Zustand
- **HTTP Client**: Axios với interceptors
- **Video Player**: Custom HTML5 Video Player (với Intersection Observer)
- **Icons**: Lucide React
- **Font**: Google Fonts — Inter, Be Vietnam Pro

### Backend
- **Runtime**: Node.js 24
- **Framework**: Nestjs
- **Language**: TypeScript
- **Database**: MongoDB với Prisma ORM
- **Cache**: Redis
- **Auth**: JWT (access + refresh token)
- **File Storage**: AWS S3 hoặc MinIO (self-hosted)
- **Video Processing**: FFmpeg (thumbnail generation, transcoding)
- **Real-time**: Socket.IO (notifications, live comments)

### DevOps
- **Package Manager**: Yarn v4 Berry (4.13.0)
- **Monorepo**: Yarn Workspaces + Turborepo (nếu cần tách packages)
- **Linting**: ESLint + Prettier
- **Testing**: Vitest (unit), Playwright (E2E)
- **Containerization**: Docker + Docker Compose

---

## Cấu trúc thư mục

```
tiktok/
├── claude.md                    # File hướng dẫn này
├── package.json
├── .yarnrc.yml                  # Yarn v4 config
├── .yarn/                       # Yarn Berry releases & plugins
│   └── releases/
│       └── yarn-4.13.0.cjs
│
├── apps/
│   ├── web/                     # Next.js frontend
│   │   ├── src/
│   │   │   ├── app/             # App Router pages
│   │   │   │   ├── (main)/      # Layout chính (feed, profile, explore)
│   │   │   │   ├── (auth)/      # Layout xác thực (login, register)
│   │   │   │   ├── upload/      # Trang upload video
│   │   │   │   └── layout.tsx
│   │   │   ├── components/      # React components
│   │   │   │   ├── ui/          # Components UI cơ bản (Button, Modal, Input...)
│   │   │   │   ├── video/       # VideoPlayer, VideoCard, VideoFeed
│   │   │   │   ├── user/        # UserAvatar, UserCard, FollowButton
│   │   │   │   ├── comment/     # CommentList, CommentInput
│   │   │   │   └── layout/      # Header, Sidebar, BottomNav
│   │   │   ├── hooks/           # Custom React hooks
│   │   │   ├── stores/          # Zustand stores
│   │   │   ├── lib/             # Utilities, API client, helpers
│   │   │   ├── types/           # TypeScript type definitions
│   │   │   └── styles/          # Global CSS, CSS variables, design tokens
│   │   ├── public/              # Static assets
│   │   └── next.config.ts
│   │
│   └── api/                     # NestJS backend API server
│       ├── src/
│       │   ├── modules/         # NestJS feature modules
│       │   │   ├── auth/        # AuthModule (controller, service, guard, strategy)
│       │   │   ├── users/       # UsersModule (controller, service, dto)
│       │   │   ├── videos/      # VideosModule (controller, service, dto)
│       │   │   ├── comments/    # CommentsModule (controller, service, dto)
│       │   │   ├── follow/      # FollowModule (controller, service)
│       │   │   ├── search/      # SearchModule (controller, service)
│       │   │   └── notifications/ # NotificationsModule + gateway
│       │   ├── common/          # Shared guards, pipes, filters, interceptors
│       │   ├── prisma/          # PrismaModule + PrismaService
│       │   ├── config/          # ConfigModule setup
│       │   ├── app.module.ts    # Root module
│       │   └── main.ts          # Entry point (bootstrap)
│       └── prisma/
│           └── schema.prisma    # MongoDB Prisma schema
│
├── packages/                    # Shared packages (nếu monorepo)
│   ├── shared-types/            # Shared TypeScript types
│   └── utils/                   # Shared utilities
│
├── docker-compose.yml
├── .env.example
└── turbo.json
```

---

## Quy tắc code

### Naming Conventions
- **Files/Folders**: `kebab-case` (ví dụ: `video-player.tsx`, `use-auth.ts`)
- **Components**: `PascalCase` (ví dụ: `VideoPlayer`, `UserAvatar`)
- **Functions/Variables**: `camelCase` (ví dụ: `getUserById`, `isLoading`)
- **Constants**: `UPPER_SNAKE_CASE` (ví dụ: `MAX_VIDEO_DURATION`, `API_BASE_URL`)
- **Types/Interfaces**: `PascalCase` với prefix `I` cho interfaces nếu cần phân biệt (ví dụ: `User`, `VideoResponse`, `IAuthContext`)
- **CSS Classes**: `camelCase` trong CSS Modules (ví dụ: `styles.videoCard`, `styles.actionButton`)

### Component Structure
```tsx
// 1. Imports (thứ tự: react → next → third-party → local)
import { useState, useEffect } from 'react';
import Image from 'next/image';
import { Heart } from 'lucide-react';
import { useAuthStore } from '@/stores/auth-store';
import styles from './video-card.module.css';

// 2. Types
interface VideoCardProps {
  video: Video;
  onLike?: (videoId: string) => void;
}

// 3. Component
export function VideoCard({ video, onLike }: VideoCardProps) {
  // hooks
  // state
  // derived state
  // effects
  // handlers
  // render
}
```

### Quy tắc quan trọng
1. **Không dùng `any`** — Luôn định nghĩa type cụ thể
2. **Không dùng `export default`** — Dùng named exports cho tất cả components và functions
3. **Mỗi component một file** — Không gom nhiều components vào một file
4. **Custom hooks cho logic phức tạp** — Tách business logic ra custom hooks
5. **Error boundaries** — Wrap các section chính bằng Error Boundary
6. **Loading states** — Luôn hiển thị skeleton/spinner khi đang tải dữ liệu
7. **Optimistic updates** — Cập nhật UI trước khi API response (like, follow, comment)
8. **Memoization** — Dùng `useMemo`/`useCallback` cho expensive computations và callbacks truyền xuống children

---

## Design System

### Color Palette
```css
:root {
  /* Primary */
  --color-primary: #fe2c55;          /* TikTok red/pink */
  --color-primary-hover: #e5284d;
  --color-primary-light: rgba(254, 44, 85, 0.1);

  /* Secondary */
  --color-secondary: #25f4ee;        /* TikTok cyan */
  --color-secondary-hover: #20d9d4;

  /* Neutral - Dark Mode (default) */
  --color-bg-primary: #000000;
  --color-bg-secondary: #121212;
  --color-bg-elevated: #1e1e1e;
  --color-bg-hover: #2a2a2a;

  --color-text-primary: #ffffff;
  --color-text-secondary: #a0a0a0;
  --color-text-tertiary: #6a6a6a;

  --color-border: #2f2f2f;
  --color-divider: #1f1f1f;

  /* Semantic */
  --color-success: #34d399;
  --color-warning: #fbbf24;
  --color-error: #ef4444;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.4);
  --shadow-lg: 0 8px 24px rgba(0, 0, 0, 0.5);

  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;

  /* Typography */
  --font-sans: 'Inter', 'Be Vietnam Pro', system-ui, sans-serif;
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.25rem;    /* 20px */
  --font-size-2xl: 1.5rem;    /* 24px */
  --font-size-3xl: 2rem;      /* 32px */

  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-normal: 250ms ease;
  --transition-slow: 350ms ease;

  /* Z-index */
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-modal-backdrop: 300;
  --z-modal: 400;
  --z-toast: 500;
}
```

### Responsive Breakpoints
```css
/* Mobile first approach */
/* sm: 640px  — Mobile landscape */
/* md: 768px  — Tablet */
/* lg: 1024px — Desktop */
/* xl: 1280px — Large desktop */
/* 2xl: 1536px — Extra large */
```

### Animations
- Sử dụng CSS `@keyframes` và `transition` cho animations
- Mọi animation phải respect `prefers-reduced-motion`
- Micro-interactions: hover effects, button press, like animation (heart burst)
- Page transitions: fade-in, slide-up
- Video feed: smooth snap scrolling

---

## Video Player — Implementation Guidelines

> ⚠️ **CRITICAL**: Đây là phần phức tạp nhất của dự án. Video player phải xử lý đúng lifecycle để tránh crash trình duyệt.

### Intersection Observer (Bắt buộc)
Sử dụng `IntersectionObserver` để quản lý auto-play/pause video trong feed. **KHÔNG BAO GIỜ** cho phép nhiều video phát cùng lúc.

```tsx
// hooks/use-video-visibility.ts
import { useEffect, useRef, useState } from 'react';

export function useVideoVisibility(threshold = 0.7) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const video = videoRef.current;
    if (!video) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsVisible(entry.isIntersecting);

        if (entry.isIntersecting) {
          // Chỉ play video khi nó chiếm >= 70% viewport
          video.play().catch(() => {
            // Autoplay bị browser block → hiện nút play
          });
        } else {
          // Pause ngay khi video ra khỏi viewport
          video.pause();
          // Reset về đầu nếu video cuộn xa hơn 1 item
          // video.currentTime = 0;
        }
      },
      {
        threshold, // 0.7 = video phải hiện 70% mới play
        rootMargin: '0px',
      }
    );

    observer.observe(video);

    return () => {
      observer.unobserve(video);
      video.pause();
    };
  }, [threshold]);

  return { videoRef, isVisible };
}
```

### Quy tắc Video Player
1. **Một video phát tại một thời điểm** — Khi video A bắt đầu play, tất cả video khác phải pause
2. **Threshold 70%** — Video chỉ auto-play khi chiếm ít nhất 70% viewport
3. **Preload strategy** — Chỉ preload `metadata` cho video ngoài viewport, preload `auto` cho video hiện tại và 1 video kế tiếp
4. **Xử lý autoplay policy** — Browsers block autoplay có sound → mặc định muted, user tap để bật sound
5. **Memory management** — Cleanup video elements khi component unmount, revoke object URLs
6. **Loading skeleton** — Hiện thumbnail + skeleton loader khi video đang buffer
7. **Error handling** — Xử lý khi video load fail (network error, format không hỗ trợ)
8. **Double-tap to like** — Detect double-tap trên mobile (300ms window) hiện heart animation
9. **Progress bar** — Custom progress bar với seek functionality, không dùng native controls
10. **Playback controls** — Long press để tua nhanh 2x, tap để play/pause

### Feed Virtualization
```tsx
// Chỉ render 3-5 video items trong DOM tại một thời điểm
// Video ở xa viewport (> 2 items) → unmount hoàn toàn
// Video gần viewport (1-2 items) → mount nhưng chỉ preload metadata
// Video trong viewport → play
```

---

## Tính năng chính

### Core Features
1. **Video Feed** — Cuộn dọc full-screen, auto-play, preload video kế tiếp
2. **Video Upload** — Quay/upload video, cắt, thêm nhạc, filter, caption
3. **Video Player** — Play/pause, progress bar, volume, playback speed
4. **Interactions** — Like (double-tap), comment, share, bookmark
5. **User Profile** — Avatar, bio, video grid, follower/following counts
6. **Follow System** — Follow/unfollow, Following feed
7. **Explore/Discover** — Trending hashtags, search users/videos/sounds
8. **Notifications** — Real-time notifications cho likes, comments, follows
9. **Authentication** — Đăng ký, đăng nhập, OAuth (Google, Facebook)

### Tính năng nâng cao
- **Recommendation Engine** — Gợi ý video dựa trên watch history
- **Duet/Stitch** — Quay video phản ứng/ghép video
- **Live Streaming** — Phát trực tiếp (WebRTC)
- **Direct Messages** — Nhắn tin riêng
- **Effects/Filters** — AR filters, beauty mode
- **Sound Library** — Kho nhạc, trending sounds

---

## API Conventions

### REST API Structure
```
Base URL: /api/v1

# Auth
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout

# Users
GET    /users/:id
PATCH  /users/:id
GET    /users/:id/videos
GET    /users/:id/liked
POST   /users/:id/follow
DELETE /users/:id/follow

# Videos
GET    /videos/feed          # For You feed (paginated)
GET    /videos/following     # Following feed
GET    /videos/:id
POST   /videos              # Upload
DELETE /videos/:id
POST   /videos/:id/like
DELETE /videos/:id/like
POST   /videos/:id/view     # Track view

# Comments
GET    /videos/:id/comments
POST   /videos/:id/comments
DELETE /comments/:id
POST   /comments/:id/like

# Search
GET    /search?q=&type=users|videos|hashtags

# Notifications
GET    /notifications
PATCH  /notifications/read
```

### Response Format
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "hasMore": true
  }
}
```

### Error Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Video caption is required",
    "details": [...]
  }
}
```

---

## Database Schema (Key Models)

> **Lưu ý MongoDB + Prisma**: MongoDB dùng `@db.ObjectId` cho ID fields, `@map("_id")` cho primary key, và không hỗ trợ `@relation` với foreign keys giống SQL. Prisma sẽ quản lý references qua `fields` và `references` nhưng không tạo foreign key constraints ở DB level.

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  username      String    @unique
  email         String    @unique
  passwordHash  String
  displayName   String
  bio           String?
  avatarUrl     String?
  isVerified    Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  videos        Video[]
  likes         Like[]
  comments      Comment[]
  followers     Follow[]  @relation("following")
  following     Follow[]  @relation("follower")
  bookmarks     Bookmark[]

  @@map("users")
}

model Video {
  id            String    @id @default(auto()) @map("_id") @db.ObjectId
  userId        String    @db.ObjectId
  user          User      @relation(fields: [userId], references: [id])
  videoUrl      String
  thumbnailUrl  String
  caption       String?
  hashtags      String[]
  soundId       String?   @db.ObjectId
  sound         Sound?    @relation(fields: [soundId], references: [id])
  duration      Int       // seconds
  viewCount     Int       @default(0)
  isPublic      Boolean   @default(true)
  createdAt     DateTime  @default(now())

  likes         Like[]
  comments      Comment[]
  bookmarks     Bookmark[]

  @@index([userId])
  @@index([createdAt])
  @@index([viewCount])
  @@map("videos")
}

model Like {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  userId    String   @db.ObjectId
  videoId   String   @db.ObjectId
  user      User     @relation(fields: [userId], references: [id])
  video     Video    @relation(fields: [videoId], references: [id])
  createdAt DateTime @default(now())

  @@unique([userId, videoId])
  @@index([videoId])
  @@map("likes")
}

model Comment {
  id        String    @id @default(auto()) @map("_id") @db.ObjectId
  userId    String    @db.ObjectId
  videoId   String    @db.ObjectId
  content   String
  parentId  String?   @db.ObjectId  // for replies
  user      User      @relation(fields: [userId], references: [id])
  video     Video     @relation(fields: [videoId], references: [id])
  parent    Comment?  @relation("replies", fields: [parentId], references: [id], onDelete: NoAction, onUpdate: NoAction)
  replies   Comment[] @relation("replies")
  likeCount Int       @default(0)
  createdAt DateTime  @default(now())

  @@index([videoId])
  @@index([parentId])
  @@map("comments")
}

model Follow {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  followerId  String   @db.ObjectId
  followingId String   @db.ObjectId
  follower    User     @relation("follower", fields: [followerId], references: [id])
  following   User     @relation("following", fields: [followingId], references: [id])
  createdAt   DateTime @default(now())

  @@unique([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
  @@map("follows")
}

model Bookmark {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  userId    String   @db.ObjectId
  videoId   String   @db.ObjectId
  user      User     @relation(fields: [userId], references: [id])
  video     Video    @relation(fields: [videoId], references: [id])
  createdAt DateTime @default(now())

  @@unique([userId, videoId])
  @@map("bookmarks")
}

model Sound {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  title       String
  artistName  String
  audioUrl    String
  coverUrl    String?
  duration    Int      // seconds
  useCount    Int      @default(0)
  createdAt   DateTime @default(now())

  videos      Video[]

  @@index([useCount])
  @@map("sounds")
}

model Notification {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  userId      String   @db.ObjectId   // người nhận
  fromUserId  String   @db.ObjectId   // người gửi
  type        String   // "like" | "comment" | "follow" | "mention"
  referenceId String?  @db.ObjectId   // videoId hoặc commentId
  isRead      Boolean  @default(false)
  createdAt   DateTime @default(now())

  @@index([userId, isRead])
  @@index([createdAt])
  @@map("notifications")
}
```

### MongoDB-specific Notes
- **Không dùng migrations**: MongoDB + Prisma dùng `prisma db push` thay vì `prisma migrate`
- **Indexes**: Định nghĩa `@@index` trong schema, Prisma sẽ tự tạo indexes trên MongoDB
- **ObjectId**: Tất cả ID và foreign key fields phải dùng `@db.ObjectId`
- **Collection mapping**: Dùng `@@map("collection_name")` để map model sang tên collection MongoDB
- **Embedded documents**: Cân nhắc dùng `type` (composite types) cho nested data nếu không cần query riêng
- **No foreign key constraints**: MongoDB không enforce foreign keys ở DB level, chỉ ở application level qua Prisma
- **Transactions**: MongoDB replica set required cho transactions (dùng MongoDB Atlas hoặc config replica set local)

### ⚠️ CRITICAL RULES cho AI khi làm việc với Database

> **TUYỆT ĐỐI KHÔNG** tạo file migration (`prisma/migrations/`). MongoDB không dùng migration files.
> Khi thay đổi schema, **CHỈ** sửa file `prisma/schema.prisma` rồi chạy `prisma db push` để đồng bộ trực tiếp vào DB.

```bash
# ✅ ĐÚNG — Cập nhật schema trực tiếp
npx prisma db push

# ❌ SAI — KHÔNG BAO GIỜ dùng migrate với MongoDB
npx prisma migrate dev    # ← KHÔNG DÙNG
npx prisma migrate deploy # ← KHÔNG DÙNG
```

Workflow khi thay đổi database:
1. Sửa `prisma/schema.prisma`
2. Chạy `npx prisma db push` (hoặc `yarn db:push`)
3. Chạy `npx prisma generate` (hoặc `yarn db:generate`) để cập nhật Prisma Client
4. Restart dev server nếu cần

---

## Performance Guidelines

1. **Video Optimization**
   - Lazy load video ngoài viewport
   - Preload 1-2 videos tiếp theo trong feed
   - Sử dụng HLS/DASH cho adaptive bitrate streaming
   - Tạo multiple resolutions (360p, 480p, 720p, 1080p)
   - Thumbnail generation bằng FFmpeg

2. **Frontend Performance**
   - Virtualized list cho video feed (chỉ render videos trong viewport)
   - Image optimization với `next/image`
   - Code splitting theo routes
   - Service Worker cho offline caching
   - Intersection Observer cho infinite scroll

3. **Backend Performance**
   - Redis cache cho hot data (trending videos, user profiles)
   - MongoDB indexing cho frequent queries (compound indexes, text indexes)
   - MongoDB Aggregation Pipeline cho complex queries (trending, recommendations)
   - Rate limiting cho API endpoints
   - Connection pooling qua Prisma (MongoDB connection string `maxPoolSize`)
   - CDN cho static assets và video delivery

4. **SEO**
   - Server-side rendering cho public pages (video, profile)
   - Open Graph meta tags cho video sharing
   - Structured data (JSON-LD) cho video content
   - Dynamic sitemap generation

---

## Git Conventions

### Branch Naming
```
feature/video-feed
fix/video-playback-ios
refactor/auth-middleware
chore/update-dependencies
```

### Commit Messages (Conventional Commits)
```
feat: add video upload with progress bar
fix: resolve video autoplay on iOS Safari
refactor: extract video player into custom hook
docs: update API documentation
chore: upgrade Next.js to 14.2
```

---

## Commands

```bash
# Cài đặt dependencies
yarn install

# Chạy development
yarn dev                  # Chạy tất cả apps
yarn workspace web dev    # Chỉ chạy frontend
yarn workspace api dev    # Chỉ chạy backend

# Build
yarn build

# Lint & Format
yarn lint
yarn format

# Test
yarn test                 # Unit tests
yarn test:e2e             # E2E tests

# Database (MongoDB + Prisma)
yarn db:push              # Push schema to MongoDB (thay cho migrate)
yarn db:generate          # Generate Prisma Client
yarn db:seed              # Seed data
yarn db:studio            # Open Prisma Studio
yarn db:reset             # Reset database (drop + push + seed)

# Docker
docker-compose up -d      # Start services (MongoDB, Redis, MinIO)
```

---

## Lưu ý quan trọng

> **Accessibility**: Đảm bảo keyboard navigation, ARIA labels, và screen reader support cho tất cả interactive elements.

> **Mobile First**: Thiết kế mobile first — trải nghiệm trên điện thoại là ưu tiên hàng đầu. Desktop là secondary layout.

> **Dark Mode Default**: Giao diện mặc định là dark mode (giống TikTok). Hỗ trợ light mode tùy chọn.

> **Localization Ready**: Sử dụng i18n keys cho text, không hard-code strings. Hỗ trợ tiếng Việt và tiếng Anh.

> **Security**: Validate input ở cả client và server. Sanitize user-generated content. Implement CSRF protection. Rate limit authentication endpoints.
