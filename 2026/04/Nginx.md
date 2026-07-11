# Nginx - 고성능 웹 서버와 리버스 프록시의 완벽 가이드

## 들어가며

웹 애플리케이션을 배포할 때 이런 상황이 있나요?

```
문제 1: Node.js는 포트 3000에서 실행
- 사용자는 포트 80(HTTP)으로 접근하고 싶음
- 어떻게 연결할까?

문제 2: 여러 백엔드 서버 실행
- 서버1: localhost:3001
- 서버2: localhost:3002
- 서버3: localhost:3003
- 들어온 요청을 어디로 보낼까?

문제 3: HTTPS 설정
- SSL 인증서 설정이 복잡함
- 각 앱에서 처리하기는 어려움

문제 4: 정적 파일 서빙
- 이미지, CSS, JS 파일들을 빠르게 서빙하고 싶음
```

**Nginx**는 이 모든 문제를 우아하게 해결합니다.

## 공식 사이트

https://nginx.org

# 1. Nginx란?

## 핵심 개념

```
Nginx = 고성능 웹 서버 + 리버스 프록시 + 로드 밸런서

할 수 있는 것:
✅ 웹 서버 (정적 파일 서빙)
✅ 리버스 프록시 (요청 전달)
✅ 로드 밸런싱 (여러 서버에 분산)
✅ SSL/TLS 암호화
✅ 캐싱
✅ 압축
✅ URL 재작성
✅ API 게이트웨이
```

## Nginx vs Apache

```
Nginx:
- 매우 빠름
- 가벼움 (메모리 적게 사용)
- 비동기 아키텍처
- 설정이 간단
- 최신 기술 지원

Apache:
- 오래되고 안정적
- 더 무거움
- 동기식 아키텍처
- 설정이 복잡할 수 있음
- 매우 광범위한 모듈
```

# 2. 설치

## Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install nginx

# 시작
sudo systemctl start nginx

# 부팅 시 자동 시작
sudo systemctl enable nginx

# 상태 확인
sudo systemctl status nginx

# 재시작
sudo systemctl restart nginx

# 설정 재로드 (재시작 없음)
sudo systemctl reload nginx
```

## macOS

```bash
# Homebrew 사용
brew install nginx

# 시작
brew services start nginx

# 상태 확인
brew services info nginx
```

## Docker

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY html /usr/share/nginx/html

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build -t my-nginx .
docker run -p 80:80 my-nginx
```

# 3. 기본 설정

## nginx.conf 위치

```bash
# 주요 경로
/etc/nginx/nginx.conf           # 메인 설정
/etc/nginx/sites-available/     # 가능한 사이트
/etc/nginx/sites-enabled/       # 활성화된 사이트
/etc/nginx/conf.d/              # 추가 설정
```

## 기본 nginx.conf 구조

```nginx
# 전체 워커 프로세스 수
worker_processes auto;

# 워커 프로세스가 열 수 있는 최대 파일 수
worker_rlimit_nofile 65535;

events {
  # 워커 프로세스가 동시 처리할 연결 수
  worker_connections 1024;

  # 효율적인 연결 처리
  use epoll;
}

http {
  # MIME 타입 정의
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  # 로그 포맷
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

  # 접근 로그
  access_log /var/log/nginx/access.log main;

  # 성능 최적화
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;

  # Gzip 압축
  gzip on;
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_types text/plain text/css text/xml text/javascript
             application/json application/javascript application/xml+rss
             application/atom+xml image/svg+xml;

  # 가상 호스트 포함
  include /etc/nginx/sites-enabled/*;
}
```

# 4. 가상 호스트 (Virtual Host)

## 기본 웹 사이트 설정

```nginx
server {
  # 포트 80 (HTTP)
  listen 80;
  listen [::]:80;

  # 도메인 이름
  server_name example.com www.example.com;

  # 루트 디렉토리
  root /var/www/example.com;

  # 기본 파일
  index index.html index.htm;

  # 접근 로그
  access_log /var/log/nginx/example.com.access.log;
  error_log /var/log/nginx/example.com.error.log;

  # 정적 파일 위치
  location / {
    try_files $uri $uri/ =404;
  }

  # 에러 페이지
  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;
}
```

## 여러 도메인 설정

```nginx
# /etc/nginx/sites-available/example.com
server {
  listen 80;
  server_name example.com www.example.com;
  root /var/www/example.com;

  location / {
    try_files $uri $uri/ =404;
  }
}

# /etc/nginx/sites-available/blog.com
server {
  listen 80;
  server_name blog.com www.blog.com;
  root /var/www/blog.com;

  location / {
    try_files $uri $uri/ =404;
  }
}

# 활성화
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/blog.com /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

# 5. 리버스 프록시

## 기본 리버스 프록시

```nginx
server {
  listen 80;
  server_name example.com;

  location / {
    # Node.js 앱이 포트 3000에서 실행 중
    proxy_pass http://localhost:3000;

    # 헤더 설정
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # 타임아웃 설정
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;

    # 버퍼링
    proxy_buffering on;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
  }
}
```

## 경로별 라우팅

```nginx
server {
  listen 80;
  server_name api.example.com;

  # API 서버
  location /api {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 웹 서버
  location /web {
    proxy_pass http://localhost:3001;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 정적 파일 (Nginx에서 직접 서빙)
  location /static {
    alias /var/www/static;
    expires 30d;
  }

  # WebSocket 지원
  location /socket {
    proxy_pass http://localhost:3002;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
  }
}
```

# 6. 로드 밸런싱

## Upstream 정의

```nginx
# 업스트림 정의
upstream backend {
  # 라운드 로빈 (기본)
  server localhost:3001;
  server localhost:3002;
  server localhost:3003;
}

# 또는 가중치 적용
upstream backend_weighted {
  server localhost:3001 weight=5;  # 5개 요청 처리
  server localhost:3002 weight=3;  # 3개 요청 처리
  server localhost:3003 weight=2;  # 2개 요청 처리
}

# 서버 상태 확인
upstream backend_with_check {
  server localhost:3001;
  server localhost:3002;
  server localhost:3003;

  # 헬스 체크 (상태 확인)
  server localhost:9999 backup;  # 다른 서버가 다운되면 사용
}

server {
  listen 80;
  server_name api.example.com;

  location / {
    proxy_pass http://backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

## 로드 밸런싱 방법

```nginx
upstream backend {
  # 1. 라운드 로빈 (기본)
  server server1.com;
  server server2.com;
  server server3.com;

  # 2. 최소 연결
  least_conn;

  # 3. IP 해시 (같은 IP는 같은 서버로)
  ip_hash;

  # 4. 일관된 해시
  hash $request_uri consistent;

  # 5. 랜덤
  random;
}
```

# 7. SSL/TLS (HTTPS)

## Let's Encrypt로 인증서 설정

```bash
# Certbot 설치
sudo apt-get install certbot python3-certbot-nginx

# 인증서 발급 및 설정 자동화
sudo certbot --nginx -d example.com -d www.example.com

# 자동 갱신 설정
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

## 수동 SSL 설정

```nginx
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;

  server_name example.com www.example.com;

  # SSL 인증서 경로
  ssl_certificate /etc/ssl/certs/example.com.crt;
  ssl_certificate_key /etc/ssl/private/example.com.key;

  # SSL 설정
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;

  # HSTS (한 달간 HTTPS 강제)
  add_header Strict-Transport-Security "max-age=2592000; includeSubDomains" always;

  location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}

# HTTP를 HTTPS로 리다이렉트
server {
  listen 80;
  listen [::]:80;
  server_name example.com www.example.com;

  return 301 https://$server_name$request_uri;
}
```

# 8. 캐싱

## 브라우저 캐싱

```nginx
server {
  listen 80;
  server_name example.com;

  root /var/www/example.com;

  # 이미지 캐싱 (30일)
  location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
  }

  # HTML 캐싱 없음 (매번 새로 로드)
  location ~* \.html$ {
    expires -1;
    add_header Cache-Control "no-cache, no-store, must-revalidate";
  }

  # 폰트 캐싱 (1년)
  location ~* \.(woff|woff2|ttf|otf)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }

  # API 응답 캐싱 안 함
  location /api/ {
    proxy_pass http://localhost:3000;
    proxy_no_cache 1;
    proxy_cache_bypass 1;
  }
}
```

## Proxy 캐싱

```nginx
# 캐시 정의
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=api_cache:10m
                 max_size=1g inactive=60m use_temp_path=off;

server {
  listen 80;
  server_name api.example.com;

  location /api/ {
    proxy_cache api_cache;
    proxy_pass http://localhost:3000;

    # 캐시 유효 기간
    proxy_cache_valid 200 10m;      # 200 응답 10분
    proxy_cache_valid 404 1m;       # 404 응답 1분
    proxy_cache_valid any 1m;       # 기타 응답 1분

    # 캐시 키 설정
    proxy_cache_key "$scheme$request_method$host$request_uri";

    # 캐시 상태 표시
    add_header X-Cache-Status $upstream_cache_status;

    # 특정 요청은 캐시 하지 않음
    proxy_no_cache $skip_cache;
    proxy_cache_bypass $skip_cache;
  }

  # POST 요청 캐시 건너뛰기
  set $skip_cache 0;
  if ($request_method = POST) {
    set $skip_cache 1;
  }
}
```

# 9. 보안

## 기본 보안 설정

```nginx
server {
  listen 80;
  server_name example.com;

  # 서버 정보 숨기기
  server_tokens off;

  # X-Frame-Options (클릭재킹 방지)
  add_header X-Frame-Options "SAMEORIGIN" always;

  # X-Content-Type-Options (MIME 타입 스니핑 방지)
  add_header X-Content-Type-Options "nosniff" always;

  # X-XSS-Protection (XSS 공격 방지)
  add_header X-XSS-Protection "1; mode=block" always;

  # Content-Security-Policy (CSP)
  add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline';" always;

  # Referrer-Policy
  add_header Referrer-Policy "strict-origin-when-cross-origin" always;

  location / {
    proxy_pass http://localhost:3000;
  }
}
```

## IP 화이트리스트

```nginx
server {
  listen 80;
  server_name admin.example.com;

  # 허용할 IP
  allow 192.168.1.0/24;
  allow 10.0.0.5;

  # 나머지는 거부
  deny all;

  location / {
    proxy_pass http://localhost:3000;
  }
}
```

## Rate Limiting

```nginx
# Rate limit 정의
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
  listen 80;
  server_name api.example.com;

  location /api/ {
    # 초당 10개 요청으로 제한
    limit_req zone=api_limit burst=20 nodelay;

    proxy_pass http://localhost:3000;
  }
}
```

# 10. 성능 최적화

## 기본 최적화

```nginx
http {
  # 버퍼 최적화
  client_body_buffer_size 128k;
  client_max_body_size 10m;
  client_header_buffer_size 1k;
  large_client_header_buffers 4 8k;

  # Keep-alive
  keepalive_timeout 65;
  keepalive_requests 100;

  # Gzip 압축
  gzip on;
  gzip_vary on;
  gzip_min_length 1000;
  gzip_comp_level 6;
  gzip_types text/plain text/css text/xml text/javascript
             application/json application/javascript application/xml+rss;

  # 파일 전송 최적화
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;

  # 해시 크기
  types_hash_max_size 2048;
  server_names_hash_bucket_size 64;
}
```

## 정적 파일 서빙 최적화

```nginx
server {
  listen 80;
  server_name cdn.example.com;

  root /var/www/cdn;

  # 정적 파일 캐싱
  location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
    expires 365d;
    add_header Cache-Control "public, immutable";
    access_log off;  # 로그 기록 안 함
  }

  # 동적 콘텐츠
  location / {
    expires -1;
    add_header Cache-Control "no-cache, no-store, must-revalidate";
  }
}
```

# 11. 모니터링

## 액세스 로그 분석

```nginx
http {
  # 커스텀 로그 포맷
  log_format detailed '$remote_addr - $remote_user [$time_local] '
                      '"$request" $status $body_bytes_sent '
                      '"$http_referer" "$http_user_agent" '
                      'rt=$request_time uct="$upstream_connect_time" '
                      'uht="$upstream_header_time" urt="$upstream_response_time"';

  access_log /var/log/nginx/access.log detailed;
}
```

## 로그 확인

```bash
# 실시간 로그 보기
tail -f /var/log/nginx/access.log

# 상태 코드별 요청 수
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c

# 가장 많이 방문한 페이지
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# 상대 IP별 요청 수
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# 느린 요청 찾기
awk '$NF > 1 {print}' /var/log/nginx/access.log  # 1초 이상
```

# 12. 실전 예제

## React SPA 배포

```nginx
server {
  listen 80;
  server_name app.example.com;

  root /var/www/app/build;
  index index.html;

  # SPA 라우팅: 모든 요청을 index.html로
  location / {
    try_files $uri /index.html;
  }

  # 정적 파일
  location ~* \.(js|css|jpg|jpeg|png|gif|ico|woff|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }

  # API 프록시
  location /api/ {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

## Node.js 앱 배포

```nginx
upstream nodejs_backend {
  server localhost:3001;
  server localhost:3002;
  server localhost:3003;
  keepalive 64;
}

server {
  listen 80;
  server_name api.example.com;

  # 요청 본문 크기 제한
  client_max_body_size 100m;

  location / {
    proxy_pass http://nodejs_backend;
    proxy_http_version 1.1;

    # Keep-alive 설정
    proxy_set_header Connection "";

    # 헤더 설정
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # 타임아웃
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
  }

  # 정적 파일
  location ~* ^/public/ {
    root /var/www/nodejs_app;
    expires 30d;
  }
}
```

## 마이크로서비스 API 게이트웨이

```nginx
upstream auth_service {
  server localhost:4001;
}

upstream user_service {
  server localhost:4002;
}

upstream product_service {
  server localhost:4003;
}

upstream order_service {
  server localhost:4004;
}

server {
  listen 80;
  server_name api.example.com;

  # 인증 서비스
  location /auth/ {
    proxy_pass http://auth_service;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 사용자 서비스
  location /users/ {
    proxy_pass http://user_service;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 상품 서비스
  location /products/ {
    proxy_pass http://product_service;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # 캐싱
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;
  }

  # 주문 서비스
  location /orders/ {
    proxy_pass http://order_service;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # 캐싱 안 함
    proxy_no_cache 1;
    proxy_cache_bypass 1;
  }

  # 헬스 체크
  location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
  }
}
```

# 13. 문제 해결

## 설정 검사

```bash
# 설정 파일 문법 검사
sudo nginx -t

# 상세한 검사
sudo nginx -T
```

## 로그 확인

```bash
# 에러 로그 보기
sudo tail -f /var/log/nginx/error.log

# 특정 에러 검색
grep "error" /var/log/nginx/error.log

# 접근 거부 (403) 확인
grep " 403 " /var/log/nginx/access.log
```

## 일반적인 에러

```
502 Bad Gateway
→ 업스트림 서버가 다운되었거나 응답 없음
→ 업스트림 서버 실행 확인

504 Gateway Timeout
→ 업스트림 서버가 느림
→ proxy_read_timeout 증가

413 Payload Too Large
→ 요청 본문이 너무 큼
→ client_max_body_size 증가

Connection refused
→ 업스트림 서버에 연결할 수 없음
→ 호스트와 포트 확인
```

# 14. 체크리스트

Nginx 설정하기:

```
[ ] Nginx 설치
[ ] 기본 설정 파일 작성
[ ] 가상 호스트 설정
[ ] 리버스 프록시 설정
[ ] SSL/TLS 설정
[ ] 로드 밸런싱 설정
[ ] 캐싱 설정
[ ] 보안 헤더 추가
[ ] 로그 설정
[ ] 성능 최적화
[ ] 모니터링 설정
```

# 결론

Nginx는:

✅ 매우 빠른 성능
✅ 가벼운 리소스 사용
✅ 강력한 리버스 프록시
✅ 쉬운 설정
✅ 대규모 트래픽 처리 가능

**모든 프로덕션 환경에서 Nginx를 사용하세요!**

## 참고 링크

- [Nginx 공식 사이트](https://nginx.org)
- [Nginx 공식 문서](https://nginx.org/en/docs/)
- [NGINX 제품 문서](https://docs.nginx.com)
