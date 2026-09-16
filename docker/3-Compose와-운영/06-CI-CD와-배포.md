# CI/CD와 배포 - 빌드 자동화부터 서버에 올리기까지

## 들어가며

로컬에서 만든 이미지를 서버에 올리는 가장 원시적인 방법은 이렇습니다.

```bash
docker build -t myapp .
docker save myapp | ssh server 'docker load'
ssh server 'docker restart myapp'
```

돌아가긴 합니다. 하지만

```
- 내 맥북(arm64)에서 만든 이미지가 서버(amd64)에서 안 돌아감
- "누가 언제 무엇을 배포했는지" 기록이 없음
- 테스트를 건너뛰고 올릴 수 있음
- 배포 중 다운타임 발생
- 롤백하려면 이전 이미지를 다시 만들어야 함
```

이 문서는 GitHub Actions로 빌드·푸시를 자동화하고, 서버에 안전하게 올리는 방법을 다룹니다.

# 1. 기본 워크플로

가장 단순한 형태입니다. 푸시하면 이미지를 빌드해서 레지스트리에 올립니다.

```yaml
# .github/workflows/docker.yml
name: docker

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - name: Docker Hub 로그인
        uses: docker/login-action@v4
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Buildx 설정
        uses: docker/setup-buildx-action@v4

      - name: 빌드 & 푸시
        uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: myuser/myapp:latest
```

액션 버전은 주기적으로 올라갑니다. 작성 시점 기준 최신 메이저는 이렇습니다.

| 액션 | 메이저 | 역할 |
|---|---|---|
| `docker/build-push-action` | v7 | 빌드와 푸시 |
| `docker/setup-buildx-action` | v4 | BuildKit 빌더 준비 |
| `docker/login-action` | v4 | 레지스트리 로그인 |
| `docker/metadata-action` | v6 | 태그·라벨 자동 생성 |
| `docker/setup-qemu-action` | v4 | 다른 아키텍처 에뮬레이션 |

# 2. GitHub Container Registry 쓰기

Docker Hub는 무료 계정에 이미지 풀 제한이 있습니다. GitHub를 쓰고 있다면 **ghcr.io**가 더 편합니다. 별도 계정 없이 `GITHUB_TOKEN`으로 로그인됩니다.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write        # ghcr 푸시에 필요

    steps:
      - uses: actions/checkout@v7

      - uses: docker/login-action@v4
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

# 3. 태그 전략

`latest` 하나만 쓰면 "지금 서버에 뭐가 떠 있는지" 알 수 없고 롤백도 못 합니다. `metadata-action`이 규칙에 맞는 태그를 자동으로 만들어 줍니다.

```yaml
      - name: 메타데이터 추출
        id: meta
        uses: docker/metadata-action@v6
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch          # main
            type=ref,event=pr              # pr-42
            type=semver,pattern={{version}}        # 1.2.3
            type=semver,pattern={{major}}.{{minor}} # 1.2
            type=sha,prefix=,format=short  # a1b2c3d
            type=raw,value=latest,enable={{is_default_branch}}

      - uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

실무에서 쓸 만한 조합입니다.

```
커밋 SHA 태그   a1b2c3d     항상 붙임. 정확히 무엇이 배포됐는지 추적 가능
버전 태그       1.2.3       릴리스 태그를 밀 때
latest          최신        편의용. 운영 배포 기준으로는 쓰지 않기
```

**운영 배포는 SHA나 버전 태그로 지정**하고, 롤백은 이전 태그로 다시 배포하면 됩니다.

# 4. 캐시로 빌드 시간 줄이기

CI는 매번 깨끗한 머신에서 돌기 때문에 아무 설정이 없으면 캐시가 전혀 없습니다. 의존성 설치부터 다시 합니다.

## GitHub Actions 캐시

```yaml
      - uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

`mode=max`는 중간 레이어까지 저장해서 적중률이 높습니다. `docker/build-push-action`을 쓰면 캐시 서비스 접속 정보가 자동으로 설정됩니다(직접 `docker buildx`를 호출할 때만 별도 설정이 필요).

GitHub 캐시는 용량 제한과 만료가 있습니다. 여러 이미지를 빌드한다면 scope를 나눠 서로 덮어쓰지 않게 합니다.

```yaml
          cache-from: type=gha,scope=api
          cache-to: type=gha,mode=max,scope=api
```

## 레지스트리 캐시

캐시를 레지스트리에 저장하는 방식입니다. CI 플랫폼을 옮겨도 쓸 수 있습니다.

```yaml
          cache-from: type=registry,ref=ghcr.io/myorg/myapp:buildcache
          cache-to: type=registry,ref=ghcr.io/myorg/myapp:buildcache,mode=max
```

## 캐시가 잘 먹는 Dockerfile

캐시 설정을 해도 Dockerfile이 나쁘면 소용없습니다.

```dockerfile
# ❌ 소스가 한 글자만 바뀌어도 npm ci 부터 다시
COPY . .
RUN npm ci

# ✅ 의존성 파일이 바뀔 때만 npm ci
COPY package*.json ./
RUN npm ci
COPY . .
```

# 5. 멀티 아키텍처 이미지

맥북(arm64)에서 개발하고 서버(amd64)에 배포한다면, 또는 반대라면 두 아키텍처를 모두 담은 이미지를 만듭니다.

```yaml
      - uses: docker/setup-qemu-action@v4     # 교차 빌드용 에뮬레이션
      - uses: docker/setup-buildx-action@v4

      - uses: docker/build-push-action@v7
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
```

로컬에서도 가능합니다.

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:test .

docker image ls --tree myapp:test
# IMAGE            ID             DISK USAGE   CONTENT SIZE
# myapp:test       07940f632099        130MB         57.8MB
# ├─ linux/amd64   00391d396d7d       28.8MB        28.8MB
# └─ linux/arm64   6afd7d6e4ba0        102MB           29MB
```

QEMU 에뮬레이션 빌드는 느립니다(특히 컴파일이 있는 언어). 빌드 시간이 문제라면 각 아키텍처를 네이티브 러너에서 빌드하고 매니페스트로 합치는 방법이 있습니다. GitHub는 arm64 러너를 제공합니다.

# 6. 테스트까지 포함한 파이프라인

```yaml
name: ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - name: 스택 띄우고 테스트
        run: |
          docker compose -f compose.yaml -f compose.ci.yaml up -d --wait
          docker compose exec -T api npm test

      - name: 정리
        if: always()
        run: docker compose down -v

  build:
    needs: test            # 테스트 통과해야 빌드
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v7
      - uses: docker/login-action@v4
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/setup-buildx-action@v4
      - id: meta
        uses: docker/metadata-action@v6
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=sha,prefix=,format=short
            type=raw,value=latest,enable={{is_default_branch}}
      - uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: 취약점 스캔
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/${{ github.repository }}:${{ steps.meta.outputs.version }}
          severity: CRITICAL,HIGH
          exit-code: "1"
```

`up -d --wait`이 여기서 진가를 발휘합니다. DB가 준비되기 전에 테스트가 시작되는 사고를 막아줍니다.

# 7. 단일 서버에 배포하기

개인 프로젝트나 작은 서비스라면 서버 한 대 + Compose로 충분합니다.

## 서버 쪽 구성

```yaml
# /srv/myapp/compose.yaml
name: myapp

services:
  api:
    image: ghcr.io/myorg/myapp:${TAG:-latest}
    restart: unless-stopped
    env_file: .env
    ports:
      - "127.0.0.1:3000:3000"      # 리버스 프록시만 접근
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 3
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  caddy:                            # 리버스 프록시 + 자동 HTTPS
    image: caddy:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy-data:/data

volumes:
  caddy-data:
```

## 배포 워크플로

```yaml
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: SSH로 배포
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /srv/myapp
            echo "TAG=${{ needs.build.outputs.tag }}" > .env.tag
            docker compose pull
            docker compose up -d --wait
            docker image prune -f
```

## 무중단에 가깝게

Compose만으로 완전한 무중단은 어렵지만, 다운타임을 크게 줄일 수는 있습니다.

```
1. healthcheck 를 정확히 작성 (앱이 실제로 요청을 받을 수 있을 때만 healthy)
2. 리버스 프록시(Caddy, Traefik, nginx)를 앞에 두기
3. docker compose up -d --wait 로 새 컨테이너가 healthy 된 뒤 전환
4. 앱이 SIGTERM 을 받으면 진행 중 요청을 마치고 종료하도록 구현 (graceful shutdown)
5. stop_grace_period 로 종료 대기 시간 확보
```

```yaml
services:
  api:
    stop_grace_period: 30s    # SIGTERM 후 30초 기다렸다가 SIGKILL
```

진짜 롤링 업데이트가 필요하면 Swarm이나 Kubernetes 영역입니다(7번 문서).

## 자동 업데이트 도구

새 이미지가 올라오면 알아서 갱신해 주는 도구도 있습니다. 대표적인 Watchtower는 원래 저장소(`containrrr/watchtower`)가 2025년 12월에 아카이브됐고, 지금은 포크(`nicholas-fedor/watchtower`)가 유지보수되고 있습니다.

```yaml
  watchtower:
    image: nickfedor/watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock    # ⚠️ 5번 문서의 위험성 참고
    command: --interval 3600 --cleanup
```

편하지만 **docker.sock을 넘겨주는 구조**라 보안 트레이드오프가 큽니다. 운영 환경이라면 CI에서 SSH로 밀어 넣는 방식이 더 통제하기 쉽습니다.

# 8. 컨테이너를 그냥 맡기는 방법 (PaaS)

서버를 직접 관리하기 싫다면 이미지만 주면 알아서 돌려주는 서비스들이 있습니다.

| 서비스 | 특징 |
|---|---|
| Google Cloud Run | 요청 있을 때만 실행, 0까지 축소, 사용량 과금 |
| AWS ECS / Fargate | AWS 생태계와 밀착, 서버 관리 없이 컨테이너 실행 |
| Fly.io | 엣지 배포, Dockerfile만 있으면 배포 |
| Render / Railway | GitHub 연결하면 자동 빌드·배포, 개인 프로젝트에 적합 |

```
직접 서버 + Compose   저렴, 자유도 높음, 운영 부담 있음
PaaS                 비쌀 수 있음, 운영 부담 적음, 스케일 자동
Kubernetes           규모가 커질 때, 학습·운영 비용 큼
```

개인 프로젝트나 소규모 서비스라면 **VPS 한 대 + Compose + 리버스 프록시**가 가성비가 가장 좋습니다.

# 9. 자주 하는 실수

```
❌ CI에서 latest 태그만 푸시
   → 배포된 버전 추적 불가, 롤백 불가. SHA 태그 병행

❌ 캐시 설정 없이 매번 풀 빌드
   → cache-from/to 만 넣어도 빌드 시간이 크게 줄어듦

❌ 로컬에서 빌드한 arm64 이미지를 amd64 서버에 배포
   → exec format error. platforms 지정

❌ 배포 스크립트에 비밀번호 하드코딩
   → CI 시크릿 사용

❌ 테스트 없이 바로 빌드·배포
   → needs: test 로 순서 강제

❌ healthcheck 없이 up -d 하고 바로 프록시 전환
   → 준비 안 된 컨테이너로 트래픽이 감

❌ 배포 후 오래된 이미지가 디스크를 채움
   → docker image prune -f 를 배포 스크립트 끝에
```

## 정리

- GitHub Actions 조합은 `login-action@v4` → `setup-buildx-action@v4` → `build-push-action@v7`, 태그는 `metadata-action@v6`로 생성
- GitHub를 쓴다면 ghcr.io가 편함. `permissions: packages: write` 필요
- 태그는 커밋 SHA를 항상 붙이고, 운영 배포는 SHA/버전 태그로 지정해 롤백 가능하게
- 캐시는 `type=gha,mode=max` 또는 레지스트리 캐시. Dockerfile 레이어 순서가 먼저
- 아키텍처가 다르면 `platforms: linux/amd64,linux/arm64`로 멀티 아키텍처 빌드
- 파이프라인은 테스트(`up -d --wait` + 테스트) → 빌드·푸시 → 스캔 → 배포 순서
- 소규모 배포는 서버에서 `docker compose pull && up -d --wait` + 리버스 프록시로 충분
- 무중단은 healthcheck + graceful shutdown + `stop_grace_period`가 기본기

## 참고 링크

- GitHub Actions로 빌드: https://docs.docker.com/build/ci/github-actions/
- GHA 캐시 백엔드: https://docs.docker.com/build/cache/backends/gha/
- 레지스트리 캐시 백엔드: https://docs.docker.com/build/cache/backends/registry/
- 멀티 플랫폼 빌드: https://docs.docker.com/build/building/multi-platform/
- build-push-action: https://github.com/docker/build-push-action
- metadata-action: https://github.com/docker/metadata-action
- trivy-action: https://github.com/aquasecurity/trivy-action
- GitHub Actions 문서: https://docs.github.com/en/actions
- Cloud Run: https://cloud.google.com/run/docs
- AWS ECS [한국어]: https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/Welcome.html
- Fly.io: https://fly.io/docs/

이전 문서: [05. 보안](./05-보안.md) | 다음 문서: [07. 오케스트레이션 입문](./07-오케스트레이션-입문.md)
