# Docker Compose - 여러 컨테이너를 한 파일로 정의하기

## 들어가며

웹 서버 + DB + 캐시로 이루어진 앱을 `docker run`만으로 띄운다고 해봅시다.

```bash
docker network create myapp
docker volume create myapp-db

docker run -d --name cache --network myapp redis:alpine

docker run -d --name db --network myapp \
  -e POSTGRES_PASSWORD=devpass -e POSTGRES_USER=app -e POSTGRES_DB=appdb \
  -v myapp-db:/var/lib/postgresql postgres:18-alpine

docker run -d --name web --network myapp -p 8080:80 \
  -v "$PWD/site:/usr/share/nginx/html:ro" \
  --restart unless-stopped nginx:alpine
```

문제가 한둘이 아닙니다.

```
1. 이 명령어들을 누가 기억하나?     → 팀원마다 다르게 실행함
2. 순서를 지켜야 함                  → DB가 준비되기 전에 앱이 뜨면 죽음
3. 정리하려면 또 하나씩 지워야 함     → rm, network rm, volume rm
4. 환경별로 조금씩 달라짐            → 개발은 포트 노출, 운영은 비노출
```

**Docker Compose**는 이걸 파일 하나로 선언합니다. `compose.yaml`에 "무엇이 필요한지" 적어두면 `docker compose up` 한 줄로 전부 뜹니다.

# 1. Compose v1과 v2, 그리고 파일 이름

옛날 자료를 보면 `docker-compose`(하이픈)를 씁니다. 이건 파이썬으로 만든 **v1**이고 2023년 7월에 지원이 끝났습니다. 지금은 Docker CLI에 내장된 **v2 이상**(Go로 작성)을 쓰고, 명령어는 하이픈 없이 `docker compose`입니다.

```bash
docker compose version
# Docker Compose version v5.0.1
```

```
docker-compose up   ❌ 옛날 방식 (v1, 지원 종료)
docker compose up   ✅ 지금 방식
```

파일 이름도 바뀌었습니다.

| 파일 이름 | 설명 |
|---|---|
| `compose.yaml` | **권장.** Compose 명세가 정한 기본 이름 |
| `compose.yml` | 허용 |
| `docker-compose.yaml` / `docker-compose.yml` | 옛날 이름. 아직 인식하지만 새로 만들 땐 쓰지 않음 |

그리고 파일 맨 위의 `version: "3.8"` 같은 키는 **더 이상 쓰지 않습니다.** 지금은 있어도 무시되고 경고만 뜹니다. 옛날 블로그 글을 복사할 때 가장 먼저 지워야 할 줄입니다.

```yaml
version: "3.8"   # ❌ 레거시. 지우세요
name: myapp      # ✅ 프로젝트 이름
services:
  ...
```

# 2. compose.yaml 기본 구조

최상위 키는 크게 6개입니다.

```yaml
name: myapp        # 프로젝트 이름 (컨테이너/네트워크/볼륨 이름의 접두사)

services:          # 컨테이너로 실행할 것들 (필수)
  web: ...
  db: ...

networks:          # 네트워크 정의 (생략하면 default 하나가 자동 생성)
  backend: ...

volumes:           # 이름 있는 볼륨 정의
  db-data:

configs:           # 설정 파일 주입
secrets:           # 비밀 값 주입
```

`name`을 적지 않으면 **디렉터리 이름**이 프로젝트 이름이 됩니다. 폴더 이름이 `app`이면 컨테이너는 `app-web-1`이 되고, 다른 폴더에도 `app`이 있으면 헷갈리니 명시하는 편이 낫습니다.

리소스 이름은 이런 규칙으로 만들어집니다.

```
컨테이너   <프로젝트>-<서비스>-<번호>     tilcheck-web-1
네트워크   <프로젝트>_<네트워크>          tilcheck_default
볼륨       <프로젝트>_<볼륨>              tilcheck_cache-data
```

# 3. 서비스에 자주 쓰는 키

실제로 돌려본 예제입니다. 이 파일 하나에 자주 쓰는 키가 거의 다 들어 있습니다.

```yaml
name: tilcheck

services:
  web:
    image: nginx:alpine          # 사용할 이미지 (build와 택일)
    ports:
      - "19080:80"               # 호스트:컨테이너
    volumes:
      - ./site:/usr/share/nginx/html:ro   # 바인드 마운트 (ro = 읽기 전용)
    environment:
      APP_ENV: ${APP_ENV:-local} # .env 값 주입, 없으면 local
    depends_on:
      cache:
        condition: service_healthy        # cache가 정상(healthy) 상태가 될 때까지 대기
    restart: unless-stopped      # 재시작 정책
    logging:
      driver: json-file
      options:
        max-size: "10m"          # 로그 파일 로테이션
        max-file: "3"
    deploy:
      resources:
        limits:
          cpus: "0.50"           # CPU 0.5개
          memory: 256M           # 메모리 상한
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost/"]
      interval: 5s
      timeout: 3s
      retries: 3
      start_period: 5s

  cache:
    image: redis:alpine
    command: ["redis-server", "--save", "", "--appendonly", "no"]  # 기본 명령 덮어쓰기
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 3s
      timeout: 2s
      retries: 5
    volumes:
      - cache-data:/data         # 이름 있는 볼륨

volumes:
  cache-data:
```

## image vs build

```yaml
services:
  # 이미 만들어진 이미지를 받아서 쓸 때
  web:
    image: nginx:alpine

  # 내 소스로 직접 빌드할 때
  api:
    build:
      context: .                 # 빌드 컨텍스트 경로
      dockerfile: Dockerfile     # 기본값이라 생략 가능
      target: dev                # 멀티스테이지에서 특정 스테이지만
      args:
        NODE_VERSION: "24"
    image: myapp-api:dev         # 빌드 결과에 붙일 태그 (선택)
```

## ports는 문자열로 쓰기

```yaml
ports:
  - "19080:80"     # ✅ 따옴표 필수
  - 19080:80       # ⚠️ YAML이 60진수로 해석하는 사고의 원인
  - "127.0.0.1:19080:80"   # 로컬에서만 접근 가능하게
  - "19080:80/udp"
```

## volumes의 세 가지 형태

```yaml
volumes:
  - db-data:/var/lib/postgresql        # 이름 있는 볼륨: 데이터 보존용
  - ./site:/usr/share/nginx/html:ro    # 바인드 마운트: 내 소스 연결
  - /app/node_modules                  # 익명 볼륨: 호스트 것으로 덮이는 걸 막을 때
```

바인드 마운트 경로는 **`./`로 시작**해야 합니다. `site:/...`라고 쓰면 `site`라는 이름의 볼륨을 찾습니다.

# 4. depends_on과 healthcheck

`depends_on`만 쓰면 "컨테이너가 시작됐다"까지만 보장합니다. DB 프로세스가 접속을 받을 준비가 됐는지는 모릅니다.

```yaml
depends_on:
  - db        # ❌ db 컨테이너가 시작되기만 하면 바로 앱을 띄움
```

그래서 **healthcheck + condition** 조합을 씁니다.

```yaml
services:
  db:
    image: postgres:18-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d appdb"]
      interval: 3s        # 검사 주기
      timeout: 3s         # 한 번의 검사 제한 시간
      retries: 10         # 몇 번 실패하면 unhealthy
      start_period: 10s   # 이 기간 실패는 retries에 세지 않음 (기동 유예)

  api:
    depends_on:
      db:
        condition: service_healthy
```

`condition`에 쓸 수 있는 값입니다.

| 값 | 의미 |
|---|---|
| `service_started` | 컨테이너가 시작되면 통과 (기본값) |
| `service_healthy` | healthcheck가 healthy가 되어야 통과 |
| `service_completed_successfully` | 컨테이너가 종료 코드 0으로 끝나야 통과 (마이그레이션 잡 등) |

실제로 `docker compose up -d --wait`을 하면 순서가 이렇게 나옵니다.

```
 Container tilcheck-cache-1  Started
 Container tilcheck-cache-1  Waiting
 Container tilcheck-cache-1  Healthy     ← cache가 healthy가 된 다음에
 Container tilcheck-web-1    Starting    ← web이 시작됨
 Container tilcheck-web-1    Started
 Container tilcheck-web-1    Healthy
```

`--wait`은 모든 서비스가 healthy(healthcheck가 없으면 running)가 될 때까지 기다렸다가 프롬프트를 돌려줍니다. CI에서 "띄우고 바로 테스트"할 때 꼭 필요합니다.

## 상태 확인

```bash
docker compose ps
# NAME               IMAGE          SERVICE   STATUS                    PORTS
# tilcheck-cache-1   redis:alpine   cache     Up 17 seconds (healthy)   6379/tcp
# tilcheck-web-1     nginx:alpine   web       Up 14 seconds (healthy)   0.0.0.0:19080->80/tcp

# 헬스체크 실행 이력 보기
docker inspect tilcheck-web-1 --format '{{range .State.Health.Log}}{{.Start}} exit={{.ExitCode}}{{"\n"}}{{end}}'
```

# 5. 환경변수와 .env

Compose에는 **성격이 다른 두 가지 환경변수**가 있습니다. 이걸 구분 못 해서 헷갈리는 경우가 많습니다.

```
① compose.yaml 안에서 치환되는 값   ${VAR}   ← .env 파일이 여기에 쓰임
② 컨테이너 안에 들어가는 값          environment / env_file
```

## ① 파일 보간용 .env

프로젝트 폴더의 `.env`는 **compose.yaml을 읽을 때** 자동으로 적용됩니다.

```bash
# .env
APP_ENV=dev
TAG=1.2.3
```

```yaml
services:
  web:
    image: myapp:${TAG}
    environment:
      APP_ENV: ${APP_ENV:-local}   # 없으면 local
      PORT: ${PORT:?PORT는 필수}    # 없으면 에러 내고 중단
```

치환 결과는 `config`로 바로 확인합니다.

```bash
docker compose config
# services:
#   web:
#     environment:
#       APP_ENV: dev        ← .env가 적용됨
#     image: myapp:1.2.3
```

| 문법 | 의미 |
|---|---|
| `${VAR}` | 값이 없으면 빈 문자열 |
| `${VAR:-기본값}` | 비어 있거나 없으면 기본값 |
| `${VAR:?메시지}` | 비어 있거나 없으면 에러로 중단 |
| `$$` | 달러 기호 자체 (컨테이너 안 셸 변수를 쓸 때) |

## ② 컨테이너에 주입하는 값

```yaml
services:
  api:
    environment:          # 직접 나열
      NODE_ENV: production
      DATABASE_URL: postgres://app:devpass@db:5432/appdb
    env_file:             # 파일에서 통째로
      - .env.api
      - path: .env.local
        required: false   # 없어도 에러 내지 않음
```

`env_file`의 파일은 **컨테이너에만** 들어가고 `${}` 치환에는 쓰이지 않습니다. 반대로 `.env`는 치환에 쓰이지만 자동으로 컨테이너에 들어가지는 않습니다.

`--env-file`로 다른 파일을 지정할 수도 있습니다.

```bash
docker compose --env-file .env.staging config
```

# 6. profiles로 선택적 서비스 만들기

개발용 관리 도구(pgAdmin, 시드 스크립트 등)는 평소엔 띄우고 싶지 않습니다.

```yaml
services:
  web:
    image: nginx:alpine

  tools:
    image: alpine:latest
    profiles: ["tools"]      # 이 프로필을 켜야만 실행됨
    command: ["sleep", "infinity"]
```

```bash
docker compose up -d                      # web만 뜸
docker compose --profile tools up -d      # web + tools

# 정의된 프로필 목록
docker compose config --profiles
# tools

# 프로필 없는 서비스 목록
docker compose config --services
# cache
# web
```

환경변수로도 켤 수 있습니다: `COMPOSE_PROFILES=tools docker compose up -d`

# 7. 파일 여러 개로 나누기

## override 파일 (자동 병합)

같은 폴더에 `compose.override.yaml`이 있으면 **자동으로** 덮어씁니다. 공통 설정은 `compose.yaml`, 로컬 전용 설정은 override에 두는 방식입니다.

```yaml
# compose.yaml
services:
  app:
    image: alpine:latest
    environment:
      MODE: base
```

```yaml
# compose.override.yaml
services:
  app:
    environment:
      MODE: override
      EXTRA: "1"
```

```bash
docker compose config | grep -A3 environment
#     environment:
#       EXTRA: "1"
#       MODE: override      ← override가 이김
```

## -f로 직접 조합

```bash
# 왼쪽부터 순서대로 병합, 뒤에 오는 파일이 우선
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

병합 규칙은 기억해 둘 만합니다.

```
스칼라 값(image, restart 등)   → 뒤 파일이 덮어씀
맵(environment, labels)        → 키 단위로 병합
리스트(ports, volumes)         → 합쳐짐 (중복 제거 안 됨)
command, entrypoint            → 통째로 교체
```

## include로 다른 compose 파일 가져오기

```yaml
name: tilinc

include:
  - db/compose.yaml     # 다른 폴더의 compose 파일을 그대로 합침

services:
  web:
    image: nginx:alpine
```

## extends로 공통 조각 재사용

```yaml
# common.yaml
services:
  base:
    environment:
      LOG_LEVEL: info
    restart: unless-stopped
```

```yaml
services:
  web:
    image: nginx:alpine
    extends:
      file: common.yaml
      service: base
```

```bash
docker compose config
#   web:
#     environment:
#       LOG_LEVEL: info
#     image: nginx:alpine
#     restart: unless-stopped
```

`include`는 **서비스를 통째로 가져오는 것**이고, `extends`는 **한 서비스의 설정을 상속**하는 것입니다.

# 8. 자주 쓰는 명령어

```bash
# 띄우기
docker compose up -d                 # 백그라운드로 시작
docker compose up -d --wait          # healthy 될 때까지 기다림
docker compose up -d --build         # 이미지 새로 빌드해서 시작
docker compose up -d --force-recreate --remove-orphans
docker compose up -d --pull always   # 이미지 항상 새로 받기

# 상태 / 로그
docker compose ps                    # 이 프로젝트 컨테이너
docker compose ps --all              # 종료된 것까지
docker compose ls                    # 머신 전체의 compose 프로젝트 목록
docker compose logs -f web           # 특정 서비스 로그 따라가기
docker compose logs --tail 100 -t    # 마지막 100줄 + 타임스탬프
docker compose stats                 # 리소스 사용량 (최신 Compose에 포함)
docker compose top                   # 컨테이너 안 프로세스

# 들어가서 작업
docker compose exec web sh           # 실행 중인 컨테이너에 들어가기
docker compose exec db psql -U app -d appdb
docker compose run --rm api npm test # 일회성 컨테이너로 명령 실행

# 설정 확인 (디버깅의 시작)
docker compose config                # 병합·치환된 최종 설정
docker compose config --services     # 서비스 이름만
docker compose config --quiet        # 문법 검사만 (CI에서 유용)

# 정리
docker compose stop                  # 멈추기만 (컨테이너 유지)
docker compose down                  # 컨테이너 + 네트워크 삭제
docker compose down -v               # 볼륨까지 삭제 (데이터 날아감)
docker compose down --rmi local      # 로컬 빌드 이미지까지 삭제
```

`docker compose run`과 `exec`의 차이는 자주 헷갈립니다.

```
exec   이미 떠 있는 컨테이너 안에서 실행    → 서비스가 running이어야 함
run    새 컨테이너를 만들어서 실행          → 포트 매핑 안 됨(--service-ports 필요), --rm 권장
```

# 9. 자주 하는 실수

```
❌ version: "3.8" 를 그대로 복사
   → 레거시. 지우세요

❌ ports를 따옴표 없이 "3000:3000"
   → YAML 해석 사고. 항상 문자열로

❌ depends_on만 믿고 DB 연결
   → condition: service_healthy + healthcheck 로

❌ 컨테이너끼리 localhost로 통신
   → 같은 네트워크에서는 서비스 이름이 호스트명 (postgres://db:5432)

❌ .env를 git에 커밋
   → .gitignore에 넣고, .env.example만 커밋

❌ down -v 를 습관적으로 사용
   → 이름 있는 볼륨(DB 데이터)까지 삭제됨

❌ 운영에서 이미지 태그를 latest로
   → 무엇이 떠 있는지 알 수 없어짐. 버전 태그 고정
```

## 정리

- `docker-compose`(v1)는 끝났고 지금은 `docker compose`, 파일은 `compose.yaml`, `version:` 키는 쓰지 않음
- 최상위 키는 `name`, `services`, `networks`, `volumes`, `configs`, `secrets`
- 기동 순서는 `depends_on` + `condition: service_healthy` + 서비스의 `healthcheck`로 보장하고, CI에서는 `up -d --wait`
- `.env`는 compose 파일 치환용, `environment`/`env_file`은 컨테이너 주입용으로 역할이 다름
- `profiles`로 선택적 서비스, `compose.override.yaml`·`-f`·`include`·`extends`로 환경별 구성 분리
- 문제가 생기면 먼저 `docker compose config`로 병합·치환 결과를 확인
- 같은 네트워크 안에서는 **서비스 이름이 곧 호스트명**

## 참고 링크

- Compose 개요: https://docs.docker.com/compose/
- Compose 파일 레퍼런스: https://docs.docker.com/reference/compose-file/
- services 키 전체 목록: https://docs.docker.com/reference/compose-file/services/
- 기동 순서 제어: https://docs.docker.com/compose/how-tos/startup-order/
- 환경변수: https://docs.docker.com/compose/how-tos/environment-variables/
- profiles: https://docs.docker.com/compose/how-tos/profiles/
- compose CLI 레퍼런스: https://docs.docker.com/reference/cli/docker/compose/
- Compose 예제 모음: https://github.com/docker/awesome-compose
- [한국어] 44bits - 도커 컴포즈로 개발 환경 구성하기: https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose

이전 문서: [05. 레지스트리와 배포](../2-이미지-빌드/05-레지스트리와-배포.md) | 다음 문서: [02. 개발 환경 구성](./02-개발-환경-구성.md)
