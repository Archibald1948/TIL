# Dockerfile 기초 - 이미지를 코드로 정의하기

## 들어가며

지금까지는 남이 만든 이미지(`nginx`, `alpine`)를 가져다 썼습니다. 이제 내 애플리케이션을 이미지로 만들 차례입니다.

```
Dockerfile = 이미지를 만드는 레시피

- 어떤 베이스 위에서 시작할지 (FROM)
- 무엇을 설치하고 복사할지 (RUN, COPY)
- 어떤 명령으로 실행할지 (CMD, ENTRYPOINT)

→ 이 파일이 있으면 누구든 같은 이미지를 재현할 수 있다
```

이 문서에서는 Dockerfile의 명령어들과, 헷갈리기 쉬운 짝(`CMD`/`ENTRYPOINT`, `COPY`/`ADD`, `ARG`/`ENV`)의 차이를 실제 동작으로 확인합니다.

# 1. 첫 Dockerfile

Node 애플리케이션을 예로 듭니다.

```js
// server.js
const express = require("express");
const app = express();
app.get("/", (req, res) => res.json({ ok: true, host: require("os").hostname() }));
app.listen(3000, () => console.log("listening on 3000"));
```

```dockerfile
# Dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --omit=dev

COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

```bash
docker build -t myapp .
docker run -d --name myapp -p 3000:3000 myapp
curl -s http://localhost:3000
# {"ok":true,"host":"a1b2c3d4e5f6"}
```

```
docker build -t myapp .
                      ↑
                  빌드 컨텍스트 (이 디렉터리 전체가 데몬으로 전송됨)
```

# 2. 명령어 정리

| 명령 | 역할 | 레이어 생성 |
|---|---|---|
| `FROM` | 베이스 이미지 지정 | - |
| `WORKDIR` | 작업 디렉터리 설정 (없으면 생성) | 메타데이터 |
| `COPY` | 빌드 컨텍스트의 파일을 이미지로 복사 | O |
| `ADD` | COPY + tar 자동 해제 + URL 다운로드 | O |
| `RUN` | 빌드 시점에 명령 실행 | O |
| `ENV` | 환경 변수 (빌드 + 런타임) | 메타데이터 |
| `ARG` | 빌드 시점 변수 (런타임에는 없음) | 메타데이터 |
| `EXPOSE` | 문서용 포트 선언 | 메타데이터 |
| `USER` | 이후 명령과 실행 시 유저 | 메타데이터 |
| `VOLUME` | 마운트 지점 선언 | 메타데이터 |
| `HEALTHCHECK` | 상태 확인 방법 | 메타데이터 |
| `CMD` | 기본 실행 명령 (덮어쓰기 쉬움) | 메타데이터 |
| `ENTRYPOINT` | 고정 실행 명령 | 메타데이터 |

## FROM

```dockerfile
FROM node:22-alpine              # 태그 지정 (권장)
FROM node:22-alpine@sha256:...   # 다이제스트 고정 (재현성 최상)
FROM scratch                     # 완전히 빈 이미지 (Go 정적 바이너리 등)
```

`FROM ubuntu` 처럼 태그를 생략하면 `latest`가 붙고, 시점에 따라 다른 이미지가 됩니다.

## WORKDIR

```dockerfile
WORKDIR /app        # cd와 비슷하지만, 없으면 만들어줌
COPY . .            # /app 기준 상대 경로
```

```dockerfile
# 하지 말 것: RUN cd 는 다음 명령에 영향을 주지 않음
RUN cd /app         # 이 레이어에서만 유효
COPY . .            # 여전히 / 기준
```

## COPY와 ADD

```dockerfile
COPY package.json ./           # 파일
COPY src/ ./src/               # 디렉터리
COPY --chown=node:node . .     # 소유자 지정하며 복사
COPY --from=builder /app/dist ./dist   # 다른 스테이지에서 복사 (멀티스테이지)
```

`ADD`는 두 가지 마법이 추가됩니다.

```dockerfile
ADD bundle.tar.gz /app/extracted/   # tar를 자동으로 풀어줌
ADD https://example.com/file.txt /app/   # URL에서 다운로드 (권장하지 않음)
```

실제로 확인해보면 차이가 분명합니다.

```dockerfile
ADD bundle.tar.gz /app/extracted/
COPY bundle.tar.gz /app/copied.tar.gz
```

```bash
docker run --rm myimage sh -c 'ls /app/extracted; ls -l /app/copied.tar.gz'
# a.txt                          ← ADD는 압축이 풀림
# 446 /app/copied.tar.gz         ← COPY는 그대로
```

> 기본은 `COPY`를 쓰고, tar를 자동으로 풀어야 할 때만 `ADD`를 쓰세요. URL 다운로드는 캐시/검증 문제 때문에 `RUN curl`로 명시적으로 처리하는 편이 낫습니다.

## RUN

```dockerfile
RUN apk add --no-cache curl                 # shell form
RUN ["/bin/sh", "-c", "apk add curl"]       # exec form

# 여러 명령을 한 레이어로 (레이어 수와 크기 절약)
RUN apk add --no-cache curl git \
 && rm -rf /var/cache/apk/*
```

# 3. ARG와 ENV

```dockerfile
FROM alpine:3.22
ARG APP_VERSION=0.0.1        # 빌드 시점에만 존재
ENV APP_ENV=production       # 빌드 + 런타임 모두 존재
RUN echo "빌드 시점 ARG: $APP_VERSION" > /app/build-info.txt
```

```bash
docker build --build-arg APP_VERSION=1.2.3 -t argdemo .
docker run --rm argdemo sh -c 'cat /app/build-info.txt; echo "런타임 ENV: $APP_ENV"; echo "런타임 ARG: [$APP_VERSION]"'
```

```
빌드 시점 ARG: 1.2.3
런타임 ENV: production
런타임 ARG: []          ← ARG는 런타임에 남지 않음
```

| | ARG | ENV |
|---|---|---|
| 유효 범위 | 빌드 중에만 | 빌드 + 컨테이너 실행 중 |
| 전달 방법 | `--build-arg KEY=값` | `-e KEY=값`으로 덮어쓰기 가능 |
| 시크릿 용도 | 불가 (history에 남음) | 불가 (inspect/history에 노출) |

> `ARG`와 `ENV` 모두 **시크릿을 담으면 안 됩니다**. `docker history`와 `docker inspect`로 값이 그대로 보입니다. 빌드 시 시크릿이 필요하면 `--mount=type=secret`을 쓰세요. ([02. 빌드 캐시와 레이어 최적화](./02-빌드-캐시와-레이어-최적화.md))

# 4. CMD와 ENTRYPOINT

가장 헷갈리는 부분입니다. 직접 만들어서 비교해봅니다.

## CMD만 있는 경우

```dockerfile
FROM alpine:3.22
CMD ["echo", "CMD 기본값"]
```

```bash
docker run --rm cmddemo
# CMD 기본값

docker run --rm cmddemo echo "인자로 교체됨"
# 인자로 교체됨          ← CMD는 통째로 교체된다
```

## ENTRYPOINT + CMD

```dockerfile
FROM alpine:3.22
ENTRYPOINT ["echo", "ENTRYPOINT:"]
CMD ["기본 인자"]
```

```bash
docker run --rm entrydemo
# ENTRYPOINT: 기본 인자

docker run --rm entrydemo "넘긴 인자"
# ENTRYPOINT: 넘긴 인자   ← ENTRYPOINT는 고정, CMD 부분만 교체
```

```
정리하면

CMD              = 기본 명령, docker run 인자로 통째로 교체됨
ENTRYPOINT       = 고정 명령, docker run 인자는 뒤에 "붙는다"
ENTRYPOINT + CMD = 실행 파일은 고정 + 기본 인자만 교체 (가장 많이 쓰는 조합)
```

| 목적 | 권장 |
|---|---|
| 일반 서버 앱 | `CMD ["node", "server.js"]` |
| CLI 도구처럼 동작 | `ENTRYPOINT ["mytool"]` + `CMD ["--help"]` |
| 진입 스크립트 필요 | `ENTRYPOINT ["/entrypoint.sh"]` + `CMD ["app"]` |

ENTRYPOINT를 무시하고 들어가고 싶을 때는 이렇게 합니다.

```bash
docker run --rm -it --entrypoint sh myimage
```

# 5. shell form과 exec form

```dockerfile
CMD node server.js              # shell form → /bin/sh -c "node server.js"
CMD ["node", "server.js"]       # exec form  → 직접 실행
```

## shell form은 변수를 확장한다

```dockerfile
FROM alpine:3.22
ENV NAME=세상
CMD echo "shell form: 안녕 $NAME"
```

```bash
docker run --rm shelldemo
# shell form: 안녕 세상
```

exec form은 셸을 거치지 않으므로 `$NAME`이 문자 그대로 출력됩니다.

## shell form은 셸이 필요하다

```bash
docker run --rm gcr.io/distroless/static-debian12 sh -c 'echo hi'
# exec: "sh": executable file not found in $PATH
```

distroless처럼 셸이 없는 이미지에서는 shell form 자체가 동작하지 않습니다.

## 프로세스 트리 차이

```dockerfile
CMD sleep 300 | cat     # 파이프가 있는 복합 명령
```

```bash
docker exec 컨테이너 ps -o pid,ppid,comm
```

```
PID   PPID  COMMAND
    1     0 sh          ← 셸이 PID 1
    6     1 sleep
    7     1 cat
```

셸이 PID 1이 되면, 컨테이너가 받은 SIGTERM을 셸이 자식에게 전달해주지 않는 한 앱은 신호를 못 받습니다. 그러면 `docker stop`이 타임아웃 후 SIGKILL로 강제 종료하게 되고, 진행 중인 요청이 끊깁니다.

```
결론: 실행 명령은 exec form(["prog", "arg"])으로 쓴다
      복잡한 시작 로직이 필요하면 entrypoint.sh를 만들고 그 안에서 exec 사용
```

```bash
#!/bin/sh
# entrypoint.sh
set -e
echo "마이그레이션 실행"
npm run migrate
exec node server.js    # exec를 붙이면 node가 PID 1을 이어받아 신호를 직접 받음
```

# 6. USER - 비root로 실행

기본은 root입니다. 컨테이너 탈출 취약점과 결합하면 위험하므로, 실행 유저를 낮추는 게 좋습니다.

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
USER node                 # node 이미지에 미리 만들어진 유저 (uid 1000)
EXPOSE 3000
CMD ["node", "server.js"]
```

```bash
docker run --rm myapp id
# uid=1000(node) gid=1000(node) groups=1000(node)

docker run --rm myapp sh -c 'touch /root/x'
# touch: /root/x: Permission denied
```

유저가 없는 베이스 이미지라면 직접 만듭니다.

```dockerfile
RUN addgroup -S app && adduser -S -G app app
USER app
```

> `USER`는 그 이후의 `RUN`에도 적용됩니다. 패키지 설치처럼 root가 필요한 작업은 `USER` 앞에 두세요.

# 7. EXPOSE, VOLUME, HEALTHCHECK

## EXPOSE

```dockerfile
EXPOSE 3000
```

실제로 포트를 여는 게 **아닙니다**. "이 이미지는 3000번을 쓴다"는 문서 역할이고, `docker run -P`가 이 정보를 보고 랜덤 포트를 매핑합니다. 외부 노출은 여전히 `-p`가 필요합니다.

## HEALTHCHECK

```dockerfile
HEALTHCHECK --interval=5s --timeout=2s --retries=2 \
  CMD test -f /app/build-info.txt || exit 1
```

```bash
docker ps --format '{{.Names}} | {{.Status}}'
# hc | Up 7 seconds (healthy)

docker inspect -f '{{.State.Health.Status}}' hc
# healthy
```

웹 서버라면 보통 이렇게 씁니다.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

| 옵션 | 뜻 |
|---|---|
| `--interval` | 검사 주기 |
| `--timeout` | 한 번의 검사 제한 시간 |
| `--start-period` | 시작 직후 실패를 무시하는 기간 |
| `--retries` | 연속 실패 몇 번에 unhealthy로 볼지 |

## VOLUME

```dockerfile
VOLUME /var/lib/mysql
```

해당 경로를 익명 볼륨으로 만듭니다. 익명 볼륨은 관리가 어려우므로, 실행할 때 `-v 이름:경로`로 지정하는 편이 낫습니다.

# 8. .dockerignore

빌드 컨텍스트에서 제외할 파일을 지정합니다. `.gitignore`와 문법이 같습니다.

```
node_modules
npm-debug.log
.git
.env
dist
coverage
Dockerfile*
*.md
```

효과는 실측으로 확인됩니다. 40MB짜리 `node_modules`가 있는 디렉터리에서 `COPY . .`을 하면 이렇게 차이가 납니다.

```
.dockerignore 없음 → 이미지 97.3MB
node_modules 제외  → 이미지 13.4MB
```

| 제외해야 하는 이유 | 설명 |
|---|---|
| 빌드 속도 | 컨텍스트 전송량이 줄어듦 |
| 이미지 크기 | 불필요한 파일이 레이어에 안 들어감 |
| 캐시 적중률 | `.git`이 바뀔 때마다 캐시가 깨지는 일 방지 |
| 보안 | `.env`, 키 파일이 이미지에 들어가는 사고 방지 |

## 정리

- Dockerfile은 이미지를 만드는 레시피이고, 각 명령이 레이어 또는 메타데이터가 된다
- `COPY`가 기본, `ADD`는 tar 자동 해제가 필요할 때만
- `ARG`는 빌드 시점, `ENV`는 런타임까지. **둘 다 시크릿 저장소가 아니다**
- `CMD`는 통째로 교체되는 기본값, `ENTRYPOINT`는 고정 명령. 둘을 조합하는 게 일반적
- 실행 명령은 **exec form**으로 쓴다. shell form은 셸이 필요하고 신호 전달이 꼬일 수 있다
- 시작 스크립트를 쓸 때는 마지막에 `exec`를 붙여 앱이 PID 1을 이어받게 한다
- `USER`로 비root 실행, `EXPOSE`는 문서용일 뿐 포트를 열지 않는다
- `HEALTHCHECK`로 컨테이너 상태를 Docker가 판단할 수 있게 한다
- `.dockerignore`는 빌드 속도, 이미지 크기, 보안에 모두 영향을 준다

## 참고 링크

- Dockerfile 레퍼런스: https://docs.docker.com/reference/dockerfile/
- .dockerignore: https://docs.docker.com/reference/dockerfile/#dockerignore-file
- Dockerfile 모범 사례: https://docs.docker.com/build/building/best-practices/
- 빌드 변수(ARG/ENV): https://docs.docker.com/build/building/variables/
- 빌드 컨텍스트: https://docs.docker.com/build/concepts/context/

이전 문서: [1-기본 / 05. 볼륨과 데이터](../1-기본/05-볼륨과-데이터.md) | 다음 문서: [02. 빌드 캐시와 레이어 최적화](./02-빌드-캐시와-레이어-최적화.md)
