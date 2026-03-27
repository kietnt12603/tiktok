---
name: create-nextjs-component
description: Tạo một React component mới cho frontend Next.js theo chuẩn dự án TikTok Clone
---

# Tạo Next.js Component

Skill này hướng dẫn cách tạo component React mới trong frontend Next.js, tuân thủ quy tắc dự án TikTok Clone.

## Cấu trúc component

Mỗi component nằm trong `apps/web/src/components/<category>/`:

```
components/
├── ui/              # Components UI cơ bản
│   ├── button/
│   │   ├── button.tsx
│   │   └── button.module.css
│   ├── modal/
│   ├── input/
│   └── avatar/
├── video/           # Components liên quan video
│   ├── video-player/
│   │   ├── video-player.tsx
│   │   └── video-player.module.css
│   ├── video-card/
│   └── video-feed/
├── user/            # Components liên quan user
├── comment/         # Components comment
└── layout/          # Header, Sidebar, BottomNav
```

## Quy tắc BẮT BUỘC

1. **Mỗi component một thư mục** — Gồm file `.tsx` và `.module.css`
2. **File đặt tên `kebab-case`** — `video-player.tsx`, KHÔNG phải `VideoPlayer.tsx`
3. **Named exports ONLY** — `export function VideoPlayer()`, KHÔNG `export default`
4. **CSS Modules** — Dùng `*.module.css`, class đặt tên `camelCase`
5. **KHÔNG DÙNG TAILWIND** — Chỉ Vanilla CSS / CSS Modules
6. **Dark mode mặc định** — Dùng CSS variables từ design system

## Template: Component

```tsx
// components/video/video-card/video-card.tsx
'use client';

import { useState, useCallback } from 'react';
import Image from 'next/image';
import { Heart, MessageCircle, Share2, Bookmark } from 'lucide-react';
import styles from './video-card.module.css';

interface VideoCardProps {
  video: {
    id: string;
    videoUrl: string;
    thumbnailUrl: string;
    caption: string;
    user: {
      id: string;
      username: string;
      avatarUrl: string;
    };
    likeCount: number;
    commentCount: number;
  };
  isActive?: boolean;
  onLike?: (videoId: string) => void;
}

export function VideoCard({ video, isActive = false, onLike }: VideoCardProps) {
  const [isLiked, setIsLiked] = useState(false);

  const handleLike = useCallback(() => {
    setIsLiked((prev) => !prev);
    onLike?.(video.id);
  }, [video.id, onLike]);

  return (
    <div className={styles.container}>
      {/* Video content */}
      <div className={styles.videoWrapper}>
        {/* Video player here */}
      </div>

      {/* Action buttons */}
      <div className={styles.actions}>
        <button
          className={`${styles.actionButton} ${isLiked ? styles.liked : ''}`}
          onClick={handleLike}
          aria-label="Like video"
        >
          <Heart size={28} fill={isLiked ? 'var(--color-primary)' : 'none'} />
          <span className={styles.count}>{video.likeCount}</span>
        </button>

        <button className={styles.actionButton} aria-label="Comment">
          <MessageCircle size={28} />
          <span className={styles.count}>{video.commentCount}</span>
        </button>

        <button className={styles.actionButton} aria-label="Share">
          <Share2 size={28} />
        </button>

        <button className={styles.actionButton} aria-label="Bookmark">
          <Bookmark size={28} />
        </button>
      </div>

      {/* Caption */}
      <div className={styles.caption}>
        <span className={styles.username}>@{video.user.username}</span>
        <p className={styles.captionText}>{video.caption}</p>
      </div>
    </div>
  );
}
```

## Template: CSS Module

```css
/* components/video/video-card/video-card.module.css */

.container {
  position: relative;
  width: 100%;
  height: 100vh;
  height: 100dvh; /* dynamic viewport height for mobile */
  scroll-snap-align: start;
  background-color: var(--color-bg-primary);
}

.videoWrapper {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.actions {
  position: absolute;
  right: var(--space-3);
  bottom: 120px;
  display: flex;
  flex-direction: column;
  gap: var(--space-5);
  z-index: 10;
}

.actionButton {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--space-1);
  background: none;
  border: none;
  color: var(--color-text-primary);
  cursor: pointer;
  transition: transform var(--transition-fast);
}

.actionButton:active {
  transform: scale(0.9);
}

.liked {
  color: var(--color-primary);
}

.count {
  font-size: var(--font-size-xs);
  font-weight: 600;
}

.caption {
  position: absolute;
  left: var(--space-4);
  bottom: var(--space-8);
  right: 80px;
  z-index: 10;
}

.username {
  font-size: var(--font-size-base);
  font-weight: 700;
  color: var(--color-text-primary);
}

.captionText {
  margin-top: var(--space-1);
  font-size: var(--font-size-sm);
  color: var(--color-text-secondary);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Accessibility */
@media (prefers-reduced-motion: reduce) {
  .actionButton {
    transition: none;
  }
}
```

## Template: Custom Hook

```tsx
// hooks/use-video-feed.ts
'use client';

import { useState, useEffect, useCallback } from 'react';
import { apiClient } from '@/lib/api-client';

interface Video {
  id: string;
  videoUrl: string;
  thumbnailUrl: string;
  caption: string;
  // ...
}

export function useVideoFeed() {
  const [videos, setVideos] = useState<Video[]>([]);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const fetchVideos = useCallback(async () => {
    if (isLoading || !hasMore) return;
    setIsLoading(true);

    try {
      const res = await apiClient.get('/videos/feed', {
        params: { page, limit: 10 },
      });
      setVideos((prev) => [...prev, ...res.data.data]);
      setHasMore(res.data.meta.hasMore);
      setPage((prev) => prev + 1);
    } catch (error) {
      console.error('Failed to fetch videos:', error);
    } finally {
      setIsLoading(false);
    }
  }, [page, isLoading, hasMore]);

  useEffect(() => {
    fetchVideos();
  }, []); // eslint-disable-line react-hooks/exhaustive-deps

  return { videos, isLoading, hasMore, fetchMore: fetchVideos };
}
```

## Checklist sau khi tạo component
- [ ] File đặt tên `kebab-case`
- [ ] Dùng named export
- [ ] CSS dùng module, class `camelCase`
- [ ] Có ARIA labels cho interactive elements
- [ ] Dùng CSS variables từ design system (không hard-code colors)
- [ ] Respect `prefers-reduced-motion` nếu có animation
- [ ] Test trên mobile viewport (375px width)
