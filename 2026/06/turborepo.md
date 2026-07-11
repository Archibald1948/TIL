## 개요

**Turborepo**는 JavaScript/TypeScript 모노레포를 위한 고성능 빌드 시스템이다.  
Jared Palmer가 개발했으며, 2021년 Vercel에 인수되었다. 현재는 Go에서 **Rust**로 완전히 포팅되어 더 빠른 성능과 안정성을 제공한다.

> 출처: [Vercel Blog - Finishing Turborepo's migration from Go to Rust](https://vercel.com/blog/finishing-turborepos-migration-from-go-to-rust), [13labs - Turborepo vs Nx](https://www.13labs.au/compare/turborepo-vs-nx)

## 왜 필요한가? — 모노레포의 문제점

모노레포(Monorepo)는 여러 앱/패키지를 하나의 레포지토리에서 관리하는 방식이다.  
코드 공유, 의존성 관리, 원자적 커밋 등의 장점이 있지만 **규모가 커질수록 빌드가 느려지는** 문제가 생긴다.

```
# 모노레포에서 순진하게 빌드를 돌리면...
패키지 A 빌드 → 패키지 B 빌드 → 패키지 C 빌드 → ...
→ 변경사항이 없어도 매번 전체 빌드 실행됨 😢
```

Turborepo는 이 문제를 **캐싱 + 병렬 실행 + 태스크 오케스트레이션**으로 해결한다.

> 출처: [Turborepo 공식 문서 - Introduction](https://turborepo.dev/docs)

## 핵심 개념

### 1. Content-Aware Caching (콘텐츠 기반 해시 캐싱)

Turborepo는 소스 파일, 환경 변수, 의존성 버전을 입력값으로 해시를 계산하고,  
동일한 해시가 존재하면 이전 캐시 결과를 즉시 재사용한다.

```
# 첫 번째 빌드 (30초)
turbo run build  →  모두 빌드, 결과 캐싱

# 두 번째 빌드 - 변경 없음 (< 1초)
turbo run build  →  캐시에서 즉시 복원 🚀

# 일부 패키지 변경 후 빌드 (5초)
turbo run build  →  변경된 패키지 + 그에 의존하는 패키지만 재빌드
```

> 출처: [DEV Community - Monorepo Tools Comparison 2025](https://dev.to/_d7eb1c1703182e3ce1782/monorepo-tools-comparison-turborepo-vs-nx-vs-lerna-in-2025-15a6)

### 2. Remote Caching (원격 캐시)

팀원 간, CI/CD 파이프라인 간 캐시를 공유한다.  
즉, **내가 이미 빌드한 결과를 CI가 다시 빌드하지 않는다.**

```bash
turbo login   # Vercel 계정으로 로그인
turbo link    # 원격 캐시 연결
```

- Vercel Remote Cache는 **모든 플랜에서 무료** (Vercel 미호스팅 프로젝트도 사용 가능)
- Self-hosted 옵션으로 `turborepo-remote-cache` (S3, GCS, Azure Blob 등 지원)

> 출처: [Strapi Blog - Turborepo Guide](https://strapi.io/blog/turborepo-guide), [Vercel Docs - Deploying Turborepo](https://vercel.com/docs/monorepos/turborepo)

### 3. Task Orchestration (태스크 오케스트레이션)

태스크 간 의존 관계를 정의하면, Turborepo가 **최적의 순서와 최대 병렬성**으로 태스크를 실행한다.

```
packages/ui 빌드 → apps/web 빌드  (직렬)
packages/ui 빌드 → apps/admin 빌드 (직렬)
apps/web 빌드    ↕  apps/admin 빌드 (병렬)
```

> 출처: [Turborepo 공식 문서 - Introduction](https://turborepo.dev/docs)

## 디렉토리 구조

```
my-monorepo/
├── apps/
│   ├── web/          # Next.js 앱
│   └── api/          # Express 백엔드
├── packages/
│   ├── ui/           # 공유 React 컴포넌트
│   ├── eslint-config/ # 공유 ESLint 설정
│   └── tsconfig/     # 공유 TypeScript 설정
├── turbo.json        # Turborepo 설정 파일
└── package.json
```

> 출처: [DEV Community - Complete Guide to Turborepo](https://dev.to/araldhafeeri/complete-guide-to-turborepo-from-zero-to-production-3ehb)

## turbo.json 설정 (v2.x 기준)

> ⚠️ v1의 `pipeline` 키는 v2에서 `tasks`로 변경됨. 2024년 이전 튜토리얼은 구버전 키를 사용하니 주의.

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### `dependsOn` 문법

| 표현식         | 의미                                                    |
| -------------- | ------------------------------------------------------- |
| `"^build"`     | 의존하는 **모든 패키지**의 `build`가 먼저 완료되어야 함 |
| `"build"`      | **같은 패키지** 내의 `build`가 먼저 완료되어야 함       |
| `[]` (빈 배열) | 의존성 없음, 즉시 병렬 실행 가능                        |

> 출처: [jsonic.io - turbo.json Explained](https://jsonic.io/guides/turbo-json), [Vercel Academy - Update Turborepo Pipeline](https://vercel.com/academy/production-monorepos/update-turborepo-pipeline)

## 빠른 시작

```bash
# 새 모노레포 생성
npx create-turbo@latest

# 기존 프로젝트에 추가
npm install --save-dev turbo

# 빌드 실행
turbo run build

# 영향받은 패키지만 빌드 (CI 최적화)
turbo run build --affected
```

> 출처: [Turborepo 공식 문서 - Introduction](https://turborepo.dev/docs), [Turborepo run 레퍼런스](https://turbo.build/repo/docs/reference/run)

## 경쟁 도구 비교

| 도구          | 특징                                             | 적합한 경우                   |
| ------------- | ------------------------------------------------ | ----------------------------- |
| **Turborepo** | 설정 최소화, 캐싱 + 병렬 실행에 집중, Rust 기반  | JS/TS 모노레포 입문           |
| **Nx**        | 코드 생성, 의존성 그래프 시각화, 풍부한 플러그인 | 대규모 엔터프라이즈, Angular  |
| **Lerna**     | 다중 npm 패키지 버전 관리/배포 특화              | 여러 패키지를 npm에 배포할 때 |
| **Bazel**     | 폴리글랏(다언어), 초대형 확장성                  | FAANG급 엔터프라이즈          |

> 출처: [DEV Community - Monorepo Tools Comparison 2025](https://dev.to/_d7eb1c1703182e3ce1782/monorepo-tools-comparison-turborepo-vs-nx-vs-lerna-in-2025-15a6)

## 언제 쓰면 좋은가?

✅ **쓰기 좋은 경우**

- 여러 앱이 공통 컴포넌트/타입/설정을 공유할 때
- CI/CD 빌드 시간을 단축하고 싶을 때
- Next.js + 공유 패키지 구조로 프로젝트를 확장할 때

❌ **굳이 안 써도 되는 경우**

- 앱이 하나뿐인 단순한 프로젝트
- 패키지 간 공유 코드가 거의 없는 경우

> 출처: [thecodebeast.com - Should Your Startup Use TurboRepo in 2025?](https://thecodebeast.com/monorepo-madness-should-your-startup-embrace-turborepo-in-2025/)

## 한 줄 요약

**Turborepo = 모노레포에서 "변경된 것만 빌드"를 자동으로 해주는 Rust 기반 빌드 오케스트레이터**

## 참고 링크

- [Turborepo 공식 문서](https://turbo.build/repo/docs)
- [turbo.json 레퍼런스](https://turbo.build/repo/docs/reference/run)
- [Vercel Blog - Go → Rust 마이그레이션](https://vercel.com/blog/finishing-turborepos-migration-from-go-to-rust)
- [Vercel Academy - Production Monorepos](https://vercel.com/academy/production-monorepos/turborepo-basics)
