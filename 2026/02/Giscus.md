# Giscus - GitHub로 관리하는 댓글 시스템

## 들어가며

블로그에 댓글 시스템이 필요한데, Disqus 같은 외부 서비스는 너무 무겁고 추적이 많습니다.
또한 댓글 데이터 소유권도 보장되지 않습니다.

Giscus는 완전히 다른 접근 방식입니다.
**GitHub Discussions를 댓글 저장소로 사용**합니다.
즉, 모든 댓글이 당신의 GitHub 저장소에 저장되고, 당신이 완전히 통제할 수 있습니다.

## 공식 사이트

https://github.com/giscus/giscus

## Giscus의 개념

### 작동 원리

```
블로그 포스트
    ↓
Giscus 위젯 (iframe)
    ↓
GitHub Discussion API
    ↓
GitHub 저장소의 Discussion
    ↓
댓글 데이터 저장
```

### Disqus vs Giscus

```
Disqus:
├─ 외부 서비스 (disqus.com)
├─ 추적 및 광고
├─ 복잡한 관리
├─ 데이터 소유권 불확실
└─ 모바일 환경에서 느림

Giscus:
├─ 당신의 GitHub
├─ 추적 없음 (GitHub 계정 필요)
├─ 간단한 관리
├─ 완전한 데이터 소유
└─ 빠르고 가벼움
```

## Giscus 설치

### 1단계: GitHub App 설치

1. [giscus 앱](https://github.com/apps/giscus) 방문
2. "Install" 클릭
3. 댓글을 저장할 저장소 선택
4. 권한 승인

### 2단계: GitHub Discussions 활성화

저장소 설정에서:

```
Settings
  → Features
    → Discussions (체크박스 활성화)
```

### 3단계: 설정 페이지에서 코드 생성

[giscus.app](https://giscus.app) 방문

```
Repository: your-username/your-repo
Discussion category: Announcements (또는 다른 카테고리)
Theme: Light / Dark
```

## 기본 설정

### 간단한 방법 (HTML)

```html
<!-- 블로그 포스트 하단에 추가 -->
<script
  src="https://giscus.app/client.js"
  data-repo="your-username/your-repo"
  data-repo-id="YOUR_REPO_ID"
  data-category="Comments"
  data-category-id="YOUR_CATEGORY_ID"
  data-mapping="pathname"
  data-strict="0"
  data-reactions-enabled="1"
  data-emit-metadata="0"
  data-input-position="bottom"
  data-theme="light"
  data-lang="ko"
  crossorigin="anonymous"
  async
></script>
```

### React에서 사용

```typescript
// components/Giscus.tsx
import { useEffect } from 'react';

interface GiscusConfig {
  repo: string;
  repoId: string;
  category: string;
  categoryId: string;
  mapping?: 'pathname' | 'url' | 'title' | 'og:title';
  term?: string;
  theme?: 'light' | 'dark' | 'auto';
}

export function GiscusComments({ config }: { config: GiscusConfig }) {
  useEffect(() => {
    const script = document.createElement('script');
    script.src = 'https://giscus.app/client.js';
    script.dataset.repo = config.repo;
    script.dataset.repoId = config.repoId;
    script.dataset.category = config.category;
    script.dataset.categoryId = config.categoryId;
    script.dataset.mapping = config.mapping || 'pathname';
    script.dataset.theme = config.theme || 'light';
    script.async = true;
    script.crossOrigin = 'anonymous';

    const commentsDiv = document.getElementById('giscus-comments');
    if (commentsDiv) {
      commentsDiv.appendChild(script);
    }

    return () => {
      if (commentsDiv) {
        commentsDiv.innerHTML = '';
      }
    };
  }, [config]);

  return <div id="giscus-comments" />;
}

// 사용
export function BlogPost() {
  return (
    <article>
      <h1>블로그 제목</h1>
      <p>블로그 내용...</p>

      <GiscusComments
        config={{
          repo: 'your-username/your-repo',
          repoId: 'YOUR_REPO_ID',
          category: 'Comments',
          categoryId: 'YOUR_CATEGORY_ID',
          mapping: 'pathname',
          theme: 'auto'
        }}
      />
    </article>
  );
}
```

### Next.js에서 사용

```typescript
// app/blog/[slug]/page.tsx
'use client';

import dynamic from 'next/dynamic';

const GiscusComments = dynamic(
  () => import('@/components/Giscus').then(mod => mod.GiscusComments),
  { ssr: false }
);

export default function BlogPost({ params }: { params: { slug: string } }) {
  return (
    <div>
      <article>
        <h1>블로그 제목</h1>
        <p>블로그 내용...</p>
      </article>

      <GiscusComments
        config={{
          repo: 'your-username/your-repo',
          repoId: process.env.NEXT_PUBLIC_GISCUS_REPO_ID!,
          category: 'Comments',
          categoryId: process.env.NEXT_PUBLIC_GISCUS_CATEGORY_ID!,
          mapping: 'pathname',
          theme: 'auto'
        }}
      />
    </div>
  );
}
```

## Giscus 설정 상세 분석

### mapping 옵션

댓글을 어떻게 분류할지 결정합니다.

```javascript
// pathname: 페이지의 경로로 분류 (추천)
// /blog/my-post → 하나의 Discussion
mapping: 'pathname';

// url: 전체 URL로 분류
// https://example.com/blog/my-post → 하나의 Discussion
mapping: 'url';

// title: 페이지 제목으로 분류
// <title>내 블로그 - 첫 번째 글</title> → 하나의 Discussion
mapping: 'title';

// og:title: Open Graph 제목으로 분류
// <meta property="og:title" content="첫 번째 글" /> → 하나의 Discussion
mapping: 'og:title';
```

### 테마 옵션

```javascript
// light: 항상 밝은 테마
theme: 'light';

// dark: 항상 어두운 테마
theme: 'dark';

// auto: 시스템 설정을 따름
theme: 'auto';

// 커스텀 테마 (CSS 변수 사용)
theme: 'transparent_dark';
```

### Discussion Category

Discussion 생성 시 자동으로 분류됩니다.

```
시스템 카테고리:
├─ Announcements (공지)
├─ Ideas (아이디어/제안)
├─ Q&A (질문과 답변)
└─ General (일반)

커스텀 카테고리:
└─ Comments (댓글용으로 추천)
```

**추천:** "Comments" 카테고리를 직접 생성하여 사용

## 프로덕션 레벨 구현

### 1. 환경 변수 관리

```bash
# .env.local
NEXT_PUBLIC_GISCUS_REPO_ID=R_kgDOEXXXXXXX
NEXT_PUBLIC_GISCUS_CATEGORY_ID=DIC_kwDOEXXXXXXoAXXXXX
NEXT_PUBLIC_GISCUS_REPO=your-username/your-repo
```

```typescript
// lib/giscus.ts
export const giscusConfig = {
  repo: process.env.NEXT_PUBLIC_GISCUS_REPO!,
  repoId: process.env.NEXT_PUBLIC_GISCUS_REPO_ID!,
  categoryId: process.env.NEXT_PUBLIC_GISCUS_CATEGORY_ID!,
  category: 'Comments',
  mapping: 'pathname' as const,
  strict: '0',
  reactionsEnabled: '1',
  emitMetadata: '0',
  inputPosition: 'bottom' as const,
  theme: 'auto' as const,
  lang: 'ko',
};
```

### 2. 재사용 가능한 컴포넌트

```typescript
// components/Comments.tsx
'use client';

import { useEffect, useState } from 'react';
import { giscusConfig } from '@/lib/giscus';

export function Comments() {
  const [isDark, setIsDark] = useState(false);

  useEffect(() => {
    // 다크 모드 감지
    const observer = new MutationObserver(() => {
      const isDarkMode = document.documentElement.classList.contains('dark');
      setIsDark(isDarkMode);
    });

    observer.observe(document.documentElement, {
      attributes: true,
      attributeFilter: ['class']
    });

    return () => observer.disconnect();
  }, []);

  useEffect(() => {
    // Giscus 스크립트 로드
    const script = document.createElement('script');
    script.src = 'https://giscus.app/client.js';

    Object.entries({
      ...giscusConfig,
      theme: isDark ? 'dark' : 'light'
    }).forEach(([key, value]) => {
      script.setAttribute(`data-${key}`, String(value));
    });

    script.async = true;
    script.crossOrigin = 'anonymous';

    const commentsDiv = document.getElementById('giscus');
    commentsDiv?.appendChild(script);

    return () => {
      commentsDiv?.innerHTML = '';
    };
  }, [isDark]);

  return <div id="giscus" />;
}
```

### 3. 테마 동기화

```typescript
// 다크 모드와 Giscus 테마 자동 동기화
import { useEffect } from 'react';

export function useGiscusTheme() {
  useEffect(() => {
    const observer = new MutationObserver(() => {
      const theme = document.documentElement.classList.contains('dark')
        ? 'dark'
        : 'light';

      const giscusFrame = document.querySelector(
        'iframe[title="Comments"]',
      ) as HTMLIFrameElement;

      if (giscusFrame) {
        giscusFrame.contentWindow?.postMessage(
          { giscus: { setConfig: { theme } } },
          'https://giscus.app',
        );
      }
    });

    observer.observe(document.documentElement, {
      attributes: true,
      attributeFilter: ['class'],
    });

    return () => observer.disconnect();
  }, []);
}
```

### 4. 스타일링

```css
/* 댓글 영역 커스터마이징 */
#giscus {
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #e5e7eb;
}

#giscus iframe {
  width: 100%;
}

/* 다크 모드 */
:root.dark #giscus {
  color-scheme: dark;
}
```

## 중요 설정 항목

### 1. reactions-enabled

댓글에 이모지 반응을 추가할 수 있습니다.

```javascript
// 활성화
data-reactions-enabled="1"

// 비활성화
data-reactions-enabled="0"
```

### 2. emit-metadata

메타데이터를 GitHub 토론에 저장합니다.

```javascript
// 활성화 (URL, 경로 정보 저장)
data-emit-metadata="1"

// 비활성화
data-emit-metadata="0"
```

### 3. input-position

댓글 입력창의 위치

```javascript
// 하단
data-input-position="bottom"

// 상단
data-input-position="top"
```

## GitHub Discussions 관리

### 댓글 삭제/숨김

GitHub에서 직접 관리할 수 있습니다.

```
저장소
  → Discussions
    → Comments
      → 각 토론
        → 댓글 관리 (삭제, 숨김)
```

### Discussion 고정

중요한 댓글/토론을 고정할 수 있습니다.

```
토론 메뉴
  → "Pin discussion" 클릭
```

### 자동 Discussion 생성

첫 댓글이 달리면 자동으로 Discussion이 생성됩니다.

```
Discussion 생성 규칙:
- URL/Pathname 기준으로 같은 포스트의 댓글은 같은 Discussion에 모음
- 제목: 포스트 URL 또는 제목
- 카테고리: 설정한 카테고리로 자동 분류
```

## 보안 및 주의사항

### 1. GitHub 로그인 필수

Giscus에서 댓글을 작성하려면 GitHub 계정이 필요합니다.

```
장점:
- 스팸 댓글 감소
- 신원 확인
- 저장소 관리자의 신뢰성

단점:
- GitHub 계정이 없는 사람은 댓글 불가
```

### 2. 권한 관리

GitHub App 권한을 필요한 것만 설정하세요.

```
필수 권한:
- Read access to discussions
- Write access to discussions

불필요한 권한 요청 시 거부 가능
```

### 3. 저장소 공개

Giscus가 작동하려면 저장소가 **공개**여야 합니다.

```
Settings
  → Repository visibility
    → Public (공개)
```

## 팀 협업 가이드

```markdown
# Giscus 댓글 시스템 가이드

## 설정 과정

1. GitHub App 설치
2. Discussions 활성화
3. giscus.app에서 설정 생성
4. 코드 복사하여 블로그에 추가

## 댓글 관리

### GitHub에서 관리

- Discussions 탭에서 모든 댓글 확인
- 스팸 댓글 삭제
- 중요한 댓글 고정

### 이메일 알림

- Watch 설정으로 새 댓글 알림 받기
- 댓글에 답글 작성 가능

## 사용자 경험

- GitHub 계정으로 로그인
- 마크다운 지원
- 이모지 반응 가능
- 댓글 수정/삭제 가능

## 백업

모든 댓글이 GitHub에 저장되므로:

- 개별 GitHub 저장소 백업 가능
- 데이터 소유권 100% 보장
- 언제든지 다른 시스템으로 마이그레이션 가능

## 문제 해결

**Q: 댓글이 안 보여요**
A: GitHub Discussions가 활성화되었는지 확인하세요.

**Q: 스팸 댓글이 많아요**
A: GitHub에서 직접 삭제하거나 Discussion 권한을 제한하세요.

**Q: 댓글 이력을 다운로드하고 싶어요**
A: GitHub API를 사용하여 export 가능합니다.
```

## Disqus와의 비교 결론

| 항목             | Disqus          | Giscus        |
| ---------------- | --------------- | ------------- |
| **가격**         | 무료 (광고)     | 무료          |
| **데이터 소유**  | Disqus          | GitHub (당신) |
| **추적**         | 있음            | 없음          |
| **성능**         | 느림            | 빠름          |
| **로그인**       | 자체/SNS        | GitHub        |
| **관리**         | Disqus 대시보드 | GitHub        |
| **마이그레이션** | 어려움          | 쉬움          |
| **커뮤니티**     | 큼              | 중간          |

## 결론

Giscus는:

✅ **가볍고 빠름**: 외부 서비스보다 빠른 로딩
✅ **데이터 소유**: GitHub에 모든 데이터 저장
✅ **추적 없음**: 사용자 프라이버시 보호
✅ **개발자 친화적**: GitHub 기반이므로 개발자가 좋아함
✅ **무료**: 영구적으로 무료

**특히 기술 블로그나 개발자 커뮤니티에 최적화되어 있습니다.**

**지금 바로 Giscus로 댓글 시스템을 업그레이드하세요!**

## 참고 링크

- [Giscus 공식 사이트](https://giscus.app)
- [Giscus GitHub 저장소](https://github.com/giscus/giscus)
- [Giscus GitHub App](https://github.com/apps/giscus)
- [GitHub Discussions 문서](https://docs.github.com/en/discussions)
