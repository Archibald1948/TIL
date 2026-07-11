# Docker - 개발과 배포의 혁명

## 들어가며

당신이 개발한 애플리케이션이 로컬에서는 완벽하게 작동하는데, 서버에 배포하면 에러가 발생한 적이 있나요?

```
이유:
- 개발 PC: Node 16, Python 3.9, PostgreSQL 12
- 테스트 서버: Node 14, Python 3.8, PostgreSQL 11
- 프로덕션: Node 18, Python 3.10, PostgreSQL 13

→ 환경이 다르면 코드가 다르게 작동합니다!
```

Docker는 이 모든 문제를 근본적으로 해결합니다.
**애플리케이션과 모든 의존성을 하나의 컨테이너로 패킹**하여, 어디서나 동일하게 실행되도록 합니다.

## 공식 사이트

https://www.docker.com

## Docker의 개념

### 가상화와의 차이

```
가상 머신:
┌─────────────────┐
│ Host OS (Ubuntu)│
├─────────────────┤
│ Hypervisor      │
├─────────────────┤
│ Guest OS (CentOS)
│ (전체 OS 필요!) │
│ 크기: 1-2GB    │
└─────────────────┘

Docker (컨테이너):
┌─────────────────┐
│ Host OS (Ubuntu)│
├─────────────────┤
│ Docker Engine   │
├─────────────────┤
│ 격리된 파일시스템
│ (OS 공유)      │
│ 크기: 100MB    │
└─────────────────┘
```

### 핵심 개념

```
Image (이미지):
- 애플리케이션의 완전한 스냅샷
- 설정, 의존성, 코드 모두 포함
- 일종의 "청사진"

Container (컨테이너):
- Image에서 실행되는 인스턴스
- 격리된 환경에서 실행
- 실제 프로세스

Registry (레지스트리):
- Image를 저장하는 저장소
- Docker Hub (공개)
- Private Registry (비공개)
```

## Docker 설치

### macOS/Windows

```bash
# Docker Desktop 설치
# https://www.docker.com/products/docker-desktop
# dmg 파일 다운로드 후 설치

# 설치 확인
docker --version
docker run hello-world
```

### Linux (Ubuntu)

```bash
# 패키지 설치
sudo apt-get update
sudo apt-get install docker.io

# 권한 설정
sudo usermod -aG docker $USER
newgrp docker

# 설치 확인
docker --version
```

## Dockerfile 작성

### Node.js 애플리케이션

```dockerfile
# Dockerfile

# 기본 이미지 (Ubuntu + Node.js)
FROM node:18-alpine

# 작업 디렉토리 설정
WORKDIR /app

# package.json과 package-lock.json 복사
COPY package*.json ./

# 의존성 설치
RUN npm ci --only=production

# 소스 코드 복사
COPY . .

# 포트 노출
EXPOSE 3000

# 헬스체크
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js

# 애플리케이션 실행
CMD ["node", "src/index.js"]
```

### Python 애플리케이션

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0"]
```

### 다단계 빌드 (멀티스테이지)

```dockerfile
# Stage 1: 빌드
FROM node:18 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: 실행 (작은 이미지)
FROM node:18-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

EXPOSE 3000
CMD ["node", "dist/index.js"]

# 최종 이미지 크기: Stage 1 (1GB) → Stage 2 (200MB)
```

## Docker Compose

### 기본 설정

```yaml
# docker-compose.yml

version: '3.9'

services:
  # 애플리케이션
  app:
    build: . # 현재 디렉토리의 Dockerfile 사용
    ports:
      - '3000:3000' # 호스트:컨테이너
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      - db
      - redis
    volumes:
      - ./src:/app/src # 실시간 코드 반영
    restart: unless-stopped

  # PostgreSQL 데이터베이스
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U user']
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 캐시
  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data

# 데이터 저장소 정의
volumes:
  postgres_data:
  redis_data:
```

### 실행

```bash
# 모든 서비스 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f app

# 특정 서비스 재시작
docker-compose restart app

# 모든 서비스 중지
docker-compose down

# 볼륨 삭제 (데이터 손실!)
docker-compose down -v
```

## Docker 기본 명령어

### 이미지 관리

```bash
# 이미지 빌드
docker build -t my-app:1.0 .

# 이미지 조회
docker images

# 이미지 삭제
docker rmi my-app:1.0

# 이미지 푸시 (Docker Hub)
docker tag my-app:1.0 username/my-app:1.0
docker push username/my-app:1.0

# 이미지 풀 (다운로드)
docker pull ubuntu:22.04
```

### 컨테이너 관리

```bash
# 컨테이너 실행
docker run -d --name my-container -p 3000:3000 my-app:1.0

# 컨테이너 목록
docker ps              # 실행 중인 컨테이너
docker ps -a           # 모든 컨테이너

# 컨테이너 접속
docker exec -it my-container /bin/bash

# 컨테이너 로그
docker logs my-container
docker logs -f my-container  # 실시간 로그

# 컨테이너 중지/시작
docker stop my-container
docker start my-container

# 컨테이너 삭제
docker rm my-container

# 컨테이너 통계
docker stats my-container
```

## 프로덕션 레벨 설정

### 1. 멀티 환경 설정

```bash
# 개발 환경
docker-compose -f docker-compose.dev.yml up

# 프로덕션 환경
docker-compose -f docker-compose.prod.yml up -d

# 테스트 환경
docker-compose -f docker-compose.test.yml up
```

```yaml
# docker-compose.prod.yml

version: '3.9'

services:
  app:
    image: username/my-app:1.0 # 이미 빌드된 이미지 사용
    restart: always
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL} # 환경 변수로 주입
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ['CMD', 'curl', '-f', 'http://localhost:3000/health']
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15-alpine
    restart: always
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${DB_USER}']
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### 2. .dockerignore 파일

```
node_modules/
npm-debug.log
.git
.env.local
.env.*.local
.DS_Store
dist/
build/
.pytest_cache/
__pycache__/
```

### 3. 헬스체크

```typescript
// src/healthcheck.js
import http from 'http';
import db from './db';

async function healthcheck() {
  try {
    // 데이터베이스 연결 확인
    await db.query('SELECT 1');

    const req = http.get('http://localhost:3000/health', (res) => {
      if (res.statusCode === 200) {
        process.exit(0);
      } else {
        process.exit(1);
      }
    });

    req.on('error', (err) => {
      console.error(err);
      process.exit(1);
    });

    req.end();
  } catch (error) {
    console.error('Health check failed:', error);
    process.exit(1);
  }
}

healthcheck();
```

### 4. 리소스 제한

```yaml
services:
  app:
    build: .
    # CPU 제한: 0.5 = 50% of one CPU
    cpus: '0.5'
    # 메모리 제한
    mem_limit: 512m
    memswap_limit: 1g
    # 재시작 정책
    restart: unless-stopped
```

## 네트워킹

### 포트 매핑

```bash
# 호스트 포트 3000 → 컨테이너 포트 3000
docker run -p 3000:3000 my-app

# 호스트 포트 8080 → 컨테이너 포트 3000
docker run -p 8080:3000 my-app

# 모든 호스트 인터페이스
docker run -p 0.0.0.0:3000:3000 my-app
```

### 컨테이너 간 통신

```yaml
version: '3.9'

services:
  app:
    build: .
    environment:
      # 컨테이너 이름으로 통신 가능
      DATABASE_URL: postgresql://user:pass@db:5432/myapp

  db:
    image: postgres:15
```

```javascript
// Node.js에서 접근
const dbHost = 'db'; // 서비스 이름
const connectionString = `postgresql://user:pass@${dbHost}:5432/myapp`;
```

## 볼륨과 저장소

### 볼륨 유형

```yaml
services:
  app:
    volumes:
      # 1. Named volume (관리형)
      - app_data:/app/data

      # 2. Bind mount (호스트 경로)
      - ./src:/app/src

      # 3. Anonymous volume
      - /app/temp

volumes:
  app_data:
    driver: local
```

### 데이터 백업

```bash
# 데이터베이스 백업
docker exec my-postgres pg_dump -U user mydb > backup.sql

# 데이터베이스 복구
docker exec -i my-postgres psql -U user mydb < backup.sql

# 볼륨 백업
docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz /data
```

## 보안 최적화

### 1. 최소 권한 원칙

```dockerfile
# root가 아닌 사용자로 실행
FROM node:18-alpine

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app
COPY --chown=nodejs:nodejs . .

USER nodejs

CMD ["node", "index.js"]
```

### 2. 시크릿 관리

```bash
# Docker Secrets 사용 (Swarm mode)
echo "my-secret-password" | docker secret create db_password -

# 또는 환경 변수 파일
docker run --env-file .env.prod my-app

# 또는 docker-compose
docker-compose --env-file .env.prod up
```

### 3. 이미지 스캔

```bash
# 취약점 스캔
docker scan my-app:1.0

# 기본 이미지 최신화
FROM node:18-alpine  # 항상 최신 태그 사용
```

## 팀 협업 가이드

````markdown
# Docker 개발 규칙

## 로컬 개발 환경 설정

```bash
docker-compose up -d
# 또는
docker-compose -f docker-compose.dev.yml up
```
````

## 이미지 빌드 및 푸시

```bash
# 이미지 빌드
docker build -t my-app:$(git rev-parse --short HEAD) .

# Docker Hub에 푸시
docker tag my-app:${TAG} username/my-app:${TAG}
docker push username/my-app:${TAG}
```

## Dockerfile 작성 규칙

1. 멀티스테이지 빌드 사용
2. .dockerignore로 불필요한 파일 제외
3. 최소 권한으로 실행
4. 적절한 헬스체크 정의
5. 환경 변수로 설정 관리

## CI/CD 통합

GitHub Actions 예시:

```yaml
name: Build and Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build image
        run: docker build -t my-app:${GITHUB_SHA} .
      - name: Push to Docker Hub
        run: docker push my-app:${GITHUB_SHA}
```

## 문제 해결

**Q: "docker: command not found"**
A: Docker를 설치하고 $PATH에 추가했는지 확인하세요.

**Q: "Cannot connect to the Docker daemon"**
A: Docker Desktop을 시작하거나 `sudo` 권한 확인하세요.

**Q: 컨테이너 내부에서 호스트에 접근할 수 없어요**
A: `host.docker.internal` (macOS/Windows) 또는 `172.17.0.1` (Linux) 사용

**Q: 데이터가 컨테이너 재시작 후 사라져요**
A: 볼륨을 정의하여 영구 저장소 사용하세요.

```

## Docker vs 가상 머신

| 항목 | Docker | 가상 머신 |
|------|--------|----------|
| **이미지 크기** | 100MB | 1-2GB |
| **시작 시간** | 1초 | 1분 |
| **리소스 오버헤드** | 적음 | 높음 |
| **성능** | 네이티브 수준 | 10-20% 오버헤드 |
| **이식성** | 최고 | 좋음 |
| **학습곡선** | 중간 | 낮음 |

## 결론

Docker는:

✅ **일관성**: 개발 = 테스트 = 프로덕션
✅ **격리**: 각 애플리케이션이 독립적으로 실행
✅ **효율성**: VM보다 빠르고 가벼움
✅ **확장성**: 쿠버네티스로 쉽게 확장
✅ **표준화**: 업계 표준 (거의 모든 회사에서 사용)

**특히 마이크로서비스, 클라우드 배포, DevOps에 필수적입니다.**

**지금 바로 Docker를 시작하세요!**
```

## 참고 링크

- [Docker 공식 사이트](https://www.docker.com)
- [Docker 공식 문서](https://docs.docker.com)
- [Docker Hub](https://hub.docker.com)
- [Docker Compose 문서](https://docs.docker.com/compose/)
