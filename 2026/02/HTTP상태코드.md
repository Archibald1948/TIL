# HTTP 상태 코드 완벽 가이드 - 1XX부터 5XX까지 총정리

[HTTP status code tech interview](https://gyoogle.dev/blog/web-knowledge/HTTP%20status%20code.html)

### 상태 코드 완벽 참조표

```
1XX Informational (정보)
100 Continue (계속)
101 Switching Protocols (프로토콜 전환)

2XX Success (성공)
200 OK (성공)
201 Created (생성됨)
202 Accepted (수락됨)
204 No Content (내용 없음)
206 Partial Content (부분 내용)

3XX Redirection (리다이렉트)
300 Multiple Choices (여러 선택지)
301 Moved Permanently (영구 이동)
302 Found (임시 이동)
303 See Other (다른 위치 보기)
304 Not Modified (수정되지 않음)
305 Use Proxy (프록시 사용)
307 Temporary Redirect (임시 리다이렉트)
308 Permanent Redirect (영구 리다이렉트)

4XX Client Error (클라이언트 에러)
400 Bad Request (잘못된 요청)
401 Unauthorized (인증 필요)
402 Payment Required (결제 필요)
403 Forbidden (접근 금지)
404 Not Found (찾을 수 없음)
405 Method Not Allowed (허용되지 않은 메서드)
409 Conflict (충돌)
410 Gone (사라짐)
413 Payload Too Large (요청 본문이 너무 큼)
415 Unsupported Media Type (지원되지 않는 미디어 타입)
429 Too Many Requests (너무 많은 요청)
431 Request Header Fields Too Large (요청 헤더 필드가 너무 큼)

5XX Server Error (서버 에러)
500 Internal Server Error (내부 서버 에러)
501 Not Implemented (구현되지 않음)
502 Bad Gateway (잘못된 게이트웨이)
503 Service Unavailable (서비스 이용 불가)
504 Gateway Timeout (게이트웨이 타임아웃)
505 HTTP Version Not Supported (지원되지 않는 HTTP 버전)
```

## 들어가며

웹 개발을 하면서 "404 Not Found", "500 Internal Server Error" 같은 에러 메시지를 본 적이 있으신가요?

이들은 모두 **HTTP 상태 코드**입니다. HTTP 상태 코드는 클라이언트(브라우저)가 서버에 요청을 보냈을 때, 서버가 그 요청을 어떻게 처리했는지를 알려주는 3자리 숫자입니다.

```
요청 → 서버 → 응답 (상태 코드 포함)
                    ↓
         1XX: 진행 중
         2XX: 성공
         3XX: 리다이렉트
         4XX: 클라이언트 에러
         5XX: 서버 에러
```

이 가이드에서는 HTTP 상태 코드의 모든 것을 다룹니다. 각 코드가 뭔지, 왜 발생하는지, 어떻게 해결하는지를 실제 코드와 함께 배울 것입니다.

# 1XX (정보 응답) - 요청이 진행 중

1XX 상태 코드는 **요청을 받았고 처리 중**임을 나타냅니다. 임시 응답으로, 실제 응답이 아닙니다.

## 100 Continue (계속)

### 의미

클라이언트가 요청을 계속 보낼 수 있음을 나타냅니다. 주로 대용량 파일 업로드 시 사용됩니다.

### 발생 상황

```
클라이언트가 큰 요청 본문을 보내기 전에:
1. "Expect: 100-continue" 헤더를 보냄
2. 서버가 "100 Continue" 응답
3. 클라이언트가 본문을 계속 보냄

예: 100MB 파일 업로드 시
```

### 코드 예제

```typescript
// Node.js/Express에서 처리
app.post('/upload', (req, res) => {
  // 클라이언트의 Expect 헤더 확인
  if (req.get('expect') === '100-continue') {
    // 100 Continue 응답
    res.status(100).send();

    // 그 다음 본문 처리
    let uploadedSize = 0;
    req.on('data', (chunk) => {
      uploadedSize += chunk.length;
      console.log(`업로드됨: ${uploadedSize} bytes`);
    });
  }
});
```

```javascript
// JavaScript 클라이언트 (Node.js)
const http = require('http');

const options = {
  hostname: 'example.com',
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/octet-stream',
    'Content-Length': 1000000,
    Expect: '100-continue', // 100 Continue 기대
  },
};

const req = http.request(options, (res) => {
  console.log(`상태 코드: ${res.statusCode}`);
});

// 100 Continue를 기다림
req.on('continue', () => {
  console.log('서버가 계속 진행하라고 허가');
  // 대용량 데이터 전송
  req.write(largeFileData);
  req.end();
});

req.end();
```

### 해결 방법

```typescript
// 서버에서 자동으로 처리하도록
app.use(
  express.json({
    limit: '50mb',
  }),
);

app.post('/upload', (req, res) => {
  // 100 Continue는 Express가 자동 처리
  const fileSize = req.get('content-length');

  if (fileSize > 50 * 1024 * 1024) {
    return res.status(413).json({ error: 'File too large' });
  }

  res.json({ success: true });
});
```

## 101 Switching Protocols (프로토콜 전환)

### 의미

클라이언트의 Upgrade 요청에 응하여 다른 프로토콜로 전환합니다. WebSocket 연결 시 발생합니다.

### 발생 상황

```
HTTP → WebSocket 업그레이드:
1. 클라이언트: "Upgrade: websocket" 헤더 전송
2. 서버: "101 Switching Protocols" 응답
3. 연결이 WebSocket으로 전환됨
```

### 코드 예제

```javascript
// 클라이언트 (WebSocket 연결)
const socket = new WebSocket('ws://example.com/chat');

socket.addEventListener('open', (event) => {
  console.log('WebSocket 연결됨');
  socket.send('Hello Server!');
});

socket.addEventListener('message', (event) => {
  console.log('서버로부터:', event.data);
});

socket.addEventListener('close', (event) => {
  console.log('WebSocket 연결 종료');
});

socket.addEventListener('error', (event) => {
  console.error('WebSocket 에러:', event);
});
```

```javascript
// 서버 (Node.js + ws 라이브러리)
const WebSocket = require('ws');
const http = require('http');

const server = http.createServer();

// WebSocket 서버 생성
const wss = new WebSocket.Server({ server });

wss.on('connection', (ws) => {
  console.log('클라이언트 연결됨');

  ws.on('message', (message) => {
    console.log('클라이언트로부터:', message);

    // 모든 클라이언트에게 브로드캐스트
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });

  ws.on('close', () => {
    console.log('클라이언트 연결 종료');
  });

  ws.on('error', (error) => {
    console.error('WebSocket 에러:', error);
  });
});

server.listen(8080);
```

### 디버깅 방법

```typescript
// WebSocket 연결 상태 모니터링
class WebSocketManager {
  constructor(url) {
    this.url = url;
    this.ws = null;
  }

  connect() {
    console.log(`[WebSocket] ${this.url}에 연결 시도...`);

    this.ws = new WebSocket(this.url);

    this.ws.addEventListener('open', () => {
      console.log('[WebSocket] 연결됨 (상태: OPEN)');
      console.log(`[WebSocket] readyState: ${this.ws.readyState}`);
    });

    this.ws.addEventListener('error', (event) => {
      console.error('[WebSocket] 에러:', event);
      console.error('[WebSocket] readyState:', this.ws.readyState);
    });

    this.ws.addEventListener('close', (event) => {
      console.log('[WebSocket] 연결 종료');
      console.log('[WebSocket] 코드:', event.code);
      console.log('[WebSocket] 이유:', event.reason);
    });
  }

  send(message) {
    if (this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(message);
    } else {
      console.error(
        `[WebSocket] 메시지 전송 불가. ` + `현재 상태: ${this.ws.readyState}`,
      );
    }
  }
}
```

# 2XX (성공) - 요청 성공적 처리

2XX 상태 코드는 요청이 성공적으로 처리되었음을 의미합니다.

## 200 OK (성공)

### 의미

요청이 성공적으로 처리되었습니다. 가장 일반적인 성공 응답입니다.

### 발생 상황

```
GET /api/users → 200 OK + 사용자 데이터
POST /api/posts → 200 OK + 생성된 포스트
PUT /api/users/1 → 200 OK + 업데이트된 사용자
```

### 코드 예제

```typescript
// Express 서버
app.get('/api/users', (req, res) => {
  const users = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ];

  // 200 OK (기본값)
  res.json(users);

  // 명시적으로
  res.status(200).json(users);
});

app.post('/api/users', (req, res) => {
  const newUser = {
    id: 3,
    name: req.body.name,
  };

  // 201 Created가 더 정확하지만,
  // 200 OK도 유효함
  res.status(200).json({
    message: 'User created',
    data: newUser,
  });
});
```

```javascript
// JavaScript 클라이언트
async function getUsers() {
  try {
    const response = await fetch('/api/users');

    if (response.ok) {
      // 200-299
      const data = await response.json();
      console.log('사용자 목록:', data);
    } else {
      console.error('에러:', response.status);
    }
  } catch (error) {
    console.error('네트워크 에러:', error);
  }
}

getUsers();
```

## 201 Created (생성됨)

### 의미

요청이 성공했으며, 새로운 리소스가 생성되었습니다. POST 요청 시 권장됩니다.

### 발생 상황

```
POST /api/users (새 사용자 생성) → 201 Created + 생성된 사용자 데이터
POST /api/posts (새 포스트) → 201 Created + 생성된 포스트
```

### 코드 예제

```typescript
// Express에서 201 응답
app.post('/api/users', async (req, res) => {
  try {
    const { name, email } = req.body;

    // 데이터베이스에 저장
    const newUser = await User.create({
      name,
      email,
    });

    // 201 Created 응답
    // Location 헤더에 새 리소스의 URI 포함
    res.status(201).location(`/api/users/${newUser.id}`).json({
      message: 'User created successfully',
      data: newUser,
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

```javascript
// 클라이언트에서 201 처리
async function createUser(userData) {
  try {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });

    if (response.status === 201) {
      const newUser = await response.json();
      const location = response.headers.get('Location');

      console.log('사용자 생성됨:', newUser);
      console.log('새 리소스 URI:', location);

      // 페이지 이동 또는 목록 갱신
      window.location.href = location;
    } else {
      console.error('생성 실패:', response.status);
    }
  } catch (error) {
    console.error('요청 실패:', error);
  }
}
```

## 202 Accepted (수락됨)

### 의미

요청을 받았지만, 처리가 완료되지 않았습니다. 비동기 작업에 사용됩니다.

### 발생 상황

```
비동기 작업 (이메일 발송, 동영상 인코딩, 대량 처리):
1. 클라이언트가 요청
2. 서버가 "202 Accepted" 응답
3. 서버가 백그라운드에서 처리
4. 나중에 완료됨
```

### 코드 예제

```typescript
// Express - 202 Accepted
app.post('/api/videos/encode', async (req, res) => {
  const { videoId } = req.body;

  // 즉시 202 응답
  res
    .status(202)
    .location(`/api/videos/${videoId}/status`)
    .json({
      message: 'Video encoding has been accepted',
      statusUrl: `/api/videos/${videoId}/status`,
    });

  // 백그라운드에서 인코딩 작업 시작
  // 클라이언트는 statusUrl을 폴링하여 진행 상황 확인
  encodeVideoInBackground(videoId)
    .then(() => {
      console.log('동영상 인코딩 완료');
    })
    .catch((error) => {
      console.error('인코딩 실패:', error);
    });
});

// 진행 상황 조회
app.get('/api/videos/:videoId/status', async (req, res) => {
  const { videoId } = req.params;

  const status = await VideoJob.findOne({ videoId });

  if (!status) {
    return res.status(404).json({ error: 'Job not found' });
  }

  res.json({
    status: status.state, // pending, processing, completed, failed
    progress: status.progress, // 0-100
    result: status.state === 'completed' ? status.result : null,
  });
});
```

```javascript
// 클라이언트 - 202 처리 (폴링)
async function encodeVideo(videoId) {
  // 1. 인코딩 요청
  const acceptResponse = await fetch('/api/videos/encode', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ videoId }),
  });

  if (acceptResponse.status === 202) {
    const { statusUrl } = await acceptResponse.json();
    console.log('인코딩 작업 시작. 진행 상황 조회:', statusUrl);

    // 2. 진행 상황 폴링
    return pollVideoStatus(statusUrl);
  }
}

async function pollVideoStatus(statusUrl) {
  const maxAttempts = 60; // 최대 60회 시도
  let attempt = 0;

  return new Promise((resolve, reject) => {
    const interval = setInterval(async () => {
      attempt++;

      try {
        const response = await fetch(statusUrl);
        const data = await response.json();

        console.log(`[${attempt}/${maxAttempts}] 진행 상황: ${data.progress}%`);

        if (data.status === 'completed') {
          clearInterval(interval);
          console.log('인코딩 완료:', data.result);
          resolve(data.result);
        } else if (data.status === 'failed') {
          clearInterval(interval);
          reject(new Error('인코딩 실패'));
        } else if (attempt >= maxAttempts) {
          clearInterval(interval);
          reject(new Error('타임아웃'));
        }
      } catch (error) {
        console.error('상태 조회 실패:', error);
        if (attempt >= maxAttempts) {
          clearInterval(interval);
          reject(error);
        }
      }
    }, 2000); // 2초마다 확인
  });
}
```

## 204 No Content (내용 없음)

### 의미

요청이 성공했지만, 반환할 내용이 없습니다. DELETE 요청이나 폼 제출 후 주로 사용됩니다.

### 발생 상황

```
DELETE /api/users/1 → 204 No Content
PUT /api/users/1 (업데이트만) → 204 No Content
```

### 코드 예제

```typescript
// Express - 204 응답
app.delete('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;

    await User.destroy({ where: { id } });

    // 204 No Content (응답 본문 없음)
    res.status(204).send();

    // 또는
    res.sendStatus(204);
  } catch (error) {
    res.status(500).json({ error: 'Delete failed' });
  }
});

app.put('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;

    await User.update(req.body, { where: { id } });

    // 204 또는 200
    res.status(204).send();
  } catch (error) {
    res.status(400).json({ error: 'Update failed' });
  }
});
```

```javascript
// 클라이언트 - 204 처리
async function deleteUser(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`, {
      method: 'DELETE',
    });

    if (response.status === 204) {
      console.log('사용자 삭제됨');
      // 응답 본문이 없으므로 바로 처리
      refreshUserList();
    } else {
      console.error('삭제 실패:', response.status);
    }
  } catch (error) {
    console.error('요청 실패:', error);
  }
}
```

## 206 Partial Content (부분 내용)

### 의미

클라이언트의 Range 요청에 응하여 파일의 일부만 반환합니다. 동영상 스트리밍, 다운로드 재개 시 사용됩니다.

### 발생 상황

```
클라이언트: Range: bytes=0-999 (처음 1000바이트만 원함)
서버: 206 Partial Content + 처음 1000바이트 반환

예: 큰 동영상 파일 스트리밍
```

### 코드 예제

```typescript
// Express - Range 요청 처리
const fs = require('fs');
const path = require('path');

app.get('/api/files/:filename', (req, res) => {
  const filename = req.params.filename;
  const filepath = path.join('/downloads', filename);

  try {
    const stat = fs.statSync(filepath);
    const fileSize = stat.size;

    const range = req.headers.range;

    if (!range) {
      // Range 요청 없음 - 전체 파일 반환
      res.header('Content-Length', fileSize);
      res.header('Accept-Ranges', 'bytes');
      return res.sendFile(filepath);
    }

    // Range 요청 처리
    const parts = range.replace(/bytes=/, '').split('-');
    const start = parseInt(parts[0], 10);
    const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;

    // 유효성 검사
    if (start >= fileSize || start > end) {
      res.status(416).set('Content-Range', `bytes */${fileSize}`).send();
      return;
    }

    const chunkSize = end - start + 1;

    // 206 Partial Content 응답
    res.status(206);
    res.set('Content-Range', `bytes ${start}-${end}/${fileSize}`);
    res.set('Content-Length', chunkSize);
    res.set('Content-Type', 'application/octet-stream');
    res.set('Accept-Ranges', 'bytes');

    // 해당 부분만 전송
    const stream = fs.createReadStream(filepath, { start, end });
    stream.pipe(res);
  } catch (error) {
    res.status(404).json({ error: 'File not found' });
  }
});
```

```javascript
// 클라이언트 - 동영상 스트리밍
async function streamVideo(videoUrl, videoElement) {
  try {
    // 파일 크기 먼저 확인
    const headResponse = await fetch(videoUrl, { method: 'HEAD' });
    const fileSize = headResponse.headers.get('content-length');

    console.log('동영상 크기:', fileSize, 'bytes');

    // Range 요청으로 작은 청크 받기
    const chunkSize = 1024 * 1024; // 1MB
    let start = 0;

    while (start < fileSize) {
      const end = Math.min(start + chunkSize - 1, fileSize - 1);

      const response = await fetch(videoUrl, {
        headers: {
          Range: `bytes=${start}-${end}`,
        },
      });

      if (response.status === 206) {
        const chunk = await response.arrayBuffer();
        console.log(`받음: bytes ${start}-${end}`);

        // 청크를 비디오 버퍼에 추가
        // (실제로는 더 복잡한 처리 필요)
      }

      start = end + 1;
    }

    console.log('스트리밍 완료');
  } catch (error) {
    console.error('스트리밍 실패:', error);
  }
}

// 또는 HTML5 <video> 태그 사용 (브라우저가 자동 처리)
// <video src="large-video.mp4" controls></video>
// 브라우저가 자동으로 Range 요청을 보냄
```

### 디버깅

```typescript
// Network Inspector에서 확인
// Request Headers:
//   Range: bytes=0-1048575
// Response Headers:
//   206 Partial Content
//   Content-Range: bytes 0-1048575/5242880
```

# 3XX (리다이렉트) - 추가 조치 필요

3XX 상태 코드는 클라이언트가 추가 조치를 해야 함을 의미합니다. 주로 새 URL로 이동해야 함을 나타냅니다.

## 301 Moved Permanently (영구 이동)

### 의미

리소스가 새 URL로 영구적으로 이동했습니다. 브라우저와 검색 엔진이 새 URL을 기억합니다.

### 발생 상황

```
구 URL → 새 URL (영구적으로)

예:
/old-page → /new-page
/about → /about-us

검색 엔진이 새 URL로 색인 업데이트함
```

### 코드 예제

```typescript
// Express - 301 리다이렉트
app.get('/old-url', (req, res) => {
  res.status(301).redirect('/new-url');
});

// 또는 명시적으로
app.get('/about', (req, res) => {
  res.redirect(301, '/about-us');
});

// 전체 도메인 변경
app.get('/products', (req, res) => {
  res.redirect(301, 'https://newdomain.com/products');
});
```

```javascript
// 클라이언트 (fetch는 자동 따라감)
const response = await fetch('/old-url');
console.log(response.url); // /new-url로 자동 따라감
```

## 302 Found (임시 이동)

### 의미

리소스가 일시적으로 다른 URL로 이동했습니다. 원래 URL은 유효합니다.

### 발생 상황

```
임시적인 리다이렉트:

예:
- 점검 중 임시 페이지로 이동
- A/B 테스트를 위한 임시 리다이렉트
- 세션 기반 리다이렉트
```

### 코드 예제

```typescript
// Express - 302 리다이렉트
app.get('/checkout', (req, res) => {
  // 로그인 확인
  if (!req.session.user) {
    // 302 (기본값)
    res.redirect('/login');
  } else {
    res.render('checkout');
  }
});

// A/B 테스트
app.get('/homepage', (req, res) => {
  const variant = Math.random() > 0.5 ? 'a' : 'b';

  if (variant === 'a') {
    res.redirect(302, '/homepage-a');
  } else {
    res.redirect(302, '/homepage-b');
  }
});

// 유지보수 중 임시 페이지
app.get('/main-service', (req, res) => {
  res.redirect(302, '/maintenance-page');
});
```

## 304 Not Modified (수정되지 않음)

### 의미

클라이언트가 가진 캐시된 버전이 최신입니다. 서버가 파일을 다시 전송할 필요가 없습니다.

### 발생 상황

```
1. 클라이언트가 If-Modified-Since 헤더로 요청
2. 서버가 파일 수정 시간 확인
3. 수정되지 않았으면 304 응답 (본문 없음)
4. 클라이언트가 캐시된 버전 사용
```

### 코드 예제

```typescript
// Express - 304 캐시 처리
app.get('/api/config', (req, res) => {
  const lastModified = new Date('2024-01-29T00:00:00Z');

  res.set('Cache-Control', 'public, max-age=3600');
  res.set('Last-Modified', lastModified.toUTCString());
  res.set('ETag', '"abc123"');

  // If-Modified-Since 헤더 확인
  const ifModifiedSince = req.get('if-modified-since');

  if (ifModifiedSince) {
    const clientDate = new Date(ifModifiedSince);

    if (clientDate >= lastModified) {
      // 304 Not Modified
      res.status(304).send();
      return;
    }
  }

  // ETag 확인
  const ifNoneMatch = req.get('if-none-match');
  if (ifNoneMatch === '"abc123"') {
    res.status(304).send();
    return;
  }

  // 전체 응답 전송
  res.json({
    apiVersion: '2.0',
    features: ['auth', 'storage'],
  });
});
```

```javascript
// 클라이언트 - 캐시 처리
async function fetchConfig() {
  let config = null;
  let lastETag = localStorage.getItem('config-etag');
  let lastModified = localStorage.getItem('config-modified');

  const headers = {};
  if (lastETag) headers['If-None-Match'] = lastETag;
  if (lastModified) headers['If-Modified-Since'] = lastModified;

  const response = await fetch('/api/config', { headers });

  if (response.status === 304) {
    // 캐시된 데이터 사용
    config = JSON.parse(localStorage.getItem('config-data'));
    console.log('캐시된 데이터 사용');
  } else if (response.ok) {
    // 새 데이터 받음
    config = await response.json();

    // 캐시 업데이트
    localStorage.setItem('config-data', JSON.stringify(config));
    localStorage.setItem('config-etag', response.headers.get('ETag'));
    localStorage.setItem(
      'config-modified',
      response.headers.get('Last-Modified'),
    );

    console.log('새 데이터 받음');
  }

  return config;
}
```

### 브라우저 개발자 도구에서 확인

```
Network 탭:
- Response Headers에서 "304 Not Modified" 확인
- Size 열에서 "from cache" 또는 "from disk cache" 표시
- 네트워크 트래픽 감소 확인
```

## 307 Temporary Redirect (임시 리다이렉트)

### 의미

302와 유사하지만, HTTP 메서드를 유지합니다. POST 요청이 POST로 리다이렉트됩니다.

### 발생 상황

```
302 vs 307:

302 (Found):
POST /form → 302 → GET /success  (메서드 변경 가능)

307 (Temporary Redirect):
POST /form → 307 → POST /new-form  (메서드 유지)
```

### 코드 예제

```typescript
// Express - 307 사용
app.post('/form-submit', (req, res) => {
  // 폼 데이터 처리
  const processed = processFormData(req.body);

  // 307로 리다이렉트하면서 POST 메서드 유지
  // 새 엔드포인트도 POST 메서드여야 함
  res.redirect(307, '/form-confirmation');
});

app.post('/form-confirmation', (req, res) => {
  // 리다이렉트된 POST 데이터 받음
  const data = req.body;
  res.json({ success: true, data });
});
```

# 4XX (클라이언트 에러) - 클라이언트의 요청이 잘못됨

4XX 상태 코드는 클라이언트의 요청에 문제가 있음을 의미합니다.

## 400 Bad Request (잘못된 요청)

### 의미

서버가 요청을 이해할 수 없습니다. 클라이언트의 요청 형식이 잘못되었습니다.

### 발생 상황

```
- 잘못된 JSON 형식
- 필수 파라미터 누락
- 잘못된 헤더
- 파라미터 타입 오류
```

### 코드 예제

```typescript
// Express - 400 에러 처리
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;

  // 필수 필드 검사
  if (!name || !email) {
    return res.status(400).json({
      error: 'Bad Request',
      message: 'name and email are required',
    });
  }

  // 이메일 형식 검사
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    return res.status(400).json({
      error: 'Bad Request',
      message: 'Invalid email format',
      received: email,
    });
  }

  // 정상 처리
  res.json({ success: true });
});

// Joi를 사용한 유효성 검사
const Joi = require('joi');

const userSchema = Joi.object({
  name: Joi.string().required().min(2).max(50),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(0).max(150),
});

app.post('/api/users-validated', (req, res) => {
  const { error, value } = userSchema.validate(req.body);

  if (error) {
    return res.status(400).json({
      error: 'Bad Request',
      message: error.details.map((d) => d.message).join(', '),
    });
  }

  // value는 검증된 데이터
  res.json({ success: true, data: value });
});
```

```javascript
// 클라이언트 - 400 에러 처리
async function createUser(userData) {
  try {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData),
    });

    if (response.status === 400) {
      const error = await response.json();
      console.error('입력 오류:', error.message);

      // 폼에 에러 메시지 표시
      showFormError(error.message);
    } else if (response.ok) {
      console.log('사용자 생성됨');
    }
  } catch (error) {
    console.error('요청 실패:', error);
  }
}
```

### 디버깅

```bash
# curl로 테스트
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John"}'  # email 누락

# 응답: 400 Bad Request
```

## 401 Unauthorized (인증 필요)

### 의미

인증이 필요합니다. 로그인하지 않은 사용자입니다.

### 발생 상황

```
- 토큰 없음
- 토큰 만료됨
- 유효하지 않은 토큰
- 로그인 필요
```

### 코드 예제

```typescript
// Express - JWT 인증 미들웨어
const jwt = require('jsonwebtoken');

function authMiddleware(req, res, next) {
  const authHeader = req.get('Authorization');

  if (!authHeader) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Missing authorization header',
    });
  }

  // "Bearer token" 형식 확인
  const token = authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Invalid authorization header format',
    });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Unauthorized',
        message: 'Token expired',
        expiredAt: error.expiredAt,
      });
    }

    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Invalid token',
    });
  }
}

// 보호된 라우트
app.get('/api/profile', authMiddleware, (req, res) => {
  // req.user는 인증된 사용자 정보
  res.json({
    user: req.user,
    message: 'This is a protected route',
  });
});

// 로그인 엔드포인트
app.post('/login', (req, res) => {
  const { username, password } = req.body;

  // 사용자 검증 (실제로는 DB에서 확인)
  if (username === 'test' && password === 'password') {
    const token = jwt.sign(
      { id: 1, username: 'test' },
      process.env.JWT_SECRET,
      { expiresIn: '1h' },
    );

    res.json({
      token,
      expiresIn: 3600,
    });
  } else {
    res.status(401).json({
      error: 'Invalid credentials',
    });
  }
});
```

```javascript
// 클라이언트 - 토큰 관리
class ApiClient {
  constructor() {
    this.token = localStorage.getItem('token');
  }

  setToken(token) {
    this.token = token;
    localStorage.setItem('token', token);
  }

  async request(url, options = {}) {
    const headers = {
      ...options.headers,
      'Content-Type': 'application/json',
    };

    // 토큰 추가
    if (this.token) {
      headers['Authorization'] = `Bearer ${this.token}`;
    }

    const response = await fetch(url, {
      ...options,
      headers,
    });

    if (response.status === 401) {
      // 토큰 만료 - 로그인 페이지로 이동
      localStorage.removeItem('token');
      this.token = null;
      window.location.href = '/login';
      return null;
    }

    return response;
  }

  async login(username, password) {
    const response = await fetch('/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password }),
    });

    if (response.ok) {
      const { token } = await response.json();
      this.setToken(token);
      return true;
    } else if (response.status === 401) {
      console.error('로그인 실패: 잘못된 자격증명');
      return false;
    }
  }

  async getProfile() {
    const response = await this.request('/api/profile');
    if (response?.ok) {
      return response.json();
    }
  }
}

// 사용
const api = new ApiClient();

// 로그인
await api.login('test', 'password');

// 보호된 리소스 접근
const profile = await api.getProfile();
console.log(profile);
```

## 403 Forbidden (접근 금지)

### 의미

인증은 되었지만, 권한이 없습니다. 이 리소스에 접근할 수 없습니다.

### 발생 상황

```
- 관리자만 가능한 작업
- 다른 사용자의 데이터 접근
- 구독 필요
- 특정 IP 차단
```

### 코드 예제

```typescript
// Express - 권한 확인
function adminMiddleware(req, res, next) {
  if (req.user?.role !== 'admin') {
    return res.status(403).json({
      error: 'Forbidden',
      message: 'Admin access required',
    });
  }
  next();
}

app.delete('/api/users/:id', authMiddleware, adminMiddleware, (req, res) => {
  // 관리자만 접근 가능
  const { id } = req.params;
  User.destroy({ where: { id } });
  res.json({ message: 'User deleted' });
});

// 리소스 소유자 확인
function ownershipMiddleware(req, res, next) {
  const { id } = req.params;
  const userId = req.user.id;

  Post.findByPk(id).then((post) => {
    if (post.userId !== userId && req.user.role !== 'admin') {
      return res.status(403).json({
        error: 'Forbidden',
        message: 'You do not own this resource',
      });
    }
    next();
  });
}

app.put('/api/posts/:id', authMiddleware, ownershipMiddleware, (req, res) => {
  // 자신의 포스트만 수정 가능
  const { id } = req.params;
  Post.update(req.body, { where: { id } });
  res.json({ success: true });
});
```

## 404 Not Found (찾을 수 없음)

### 의미

요청한 리소스를 찾을 수 없습니다. 가장 일반적인 에러입니다.

### 발생 상황

```
- 존재하지 않는 URL
- 삭제된 리소스
- 잘못된 ID
```

### 코드 예제

```typescript
// Express - 404 처리
app.get('/api/users/:id', async (req, res) => {
  const { id } = req.params;

  const user = await User.findByPk(id);

  if (!user) {
    return res.status(404).json({
      error: 'Not Found',
      message: `User with id ${id} not found`,
    });
  }

  res.json(user);
});

// 존재하지 않는 모든 라우트
app.use((req, res) => {
  res.status(404).json({
    error: 'Not Found',
    message: `${req.method} ${req.path} not found`,
  });
});
```

```javascript
// 클라이언트 - 404 처리
async function getUser(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);

    if (response.status === 404) {
      console.error('사용자를 찾을 수 없습니다');
      showErrorPage('사용자를 찾을 수 없습니다');
    } else if (response.ok) {
      const user = await response.json();
      displayUser(user);
    }
  } catch (error) {
    console.error('요청 실패:', error);
  }
}
```

## 409 Conflict (충돌)

### 의미

요청이 현재 상태와 충돌합니다. 보통 중복 데이터나 상태 불일치입니다.

### 발생 상황

```
- 중복된 이메일 등록
- 이미 존재하는 리소스 생성
- 동시성 문제
```

### 코드 예제

```typescript
// Express - 409 에러
app.post('/api/users', async (req, res) => {
  const { email } = req.body;

  // 중복 확인
  const existingUser = await User.findOne({ where: { email } });

  if (existingUser) {
    return res.status(409).json({
      error: 'Conflict',
      message: 'Email already exists',
      existingUser: {
        id: existingUser.id,
        email: existingUser.email,
      },
    });
  }

  // 새 사용자 생성
  const newUser = await User.create(req.body);
  res.status(201).json(newUser);
});

// 낙관적 잠금 (Optimistic Locking)
app.put('/api/posts/:id', async (req, res) => {
  const { id } = req.params;
  const { content, version } = req.body;

  const post = await Post.findByPk(id);

  if (post.version !== version) {
    // 다른 사용자가 수정함
    return res.status(409).json({
      error: 'Conflict',
      message: 'Resource was modified by another user',
      currentVersion: post.version,
      submittedVersion: version,
      currentContent: post.content,
    });
  }

  // 버전 증가하며 업데이트
  post.content = content;
  post.version += 1;
  await post.save();

  res.json(post);
});
```

```javascript
// 클라이언트 - 409 처리
async function registerUser(email) {
  try {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    });

    if (response.status === 409) {
      const error = await response.json();
      console.error('이 이메일은 이미 가입되어 있습니다');
      console.error('기존 사용자:', error.existingUser);

      // 로그인 페이지로 이동
      window.location.href = `/login?email=${encodeURIComponent(email)}`;
    } else if (response.status === 201) {
      console.log('가입 완료');
    }
  } catch (error) {
    console.error('요청 실패:', error);
  }
}
```

## 429 Too Many Requests (너무 많은 요청)

### 의미

요청 한도를 초과했습니다. 일시적으로 요청을 줄여야 합니다.

### 발생 상황

```
- API 속도 제한 (Rate Limiting)
- DDoS 방어
- 시스템 보호
```

### 코드 예제

```typescript
// Express - Rate Limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 최대 100 요청
  message: 'Too many requests',
  standardHeaders: true, // Rate-Limit-* 헤더 포함
  legacyHeaders: false, // X-RateLimit-* 헤더 비활성화
});

// 모든 요청에 적용
app.use(limiter);

// 특정 라우트에만 적용
const strictLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1분
  max: 5, // 1분에 5 요청만
  skipSuccessfulRequests: true, // 성공한 요청은 카운트 안함
});

app.post('/login', strictLimiter, (req, res) => {
  // 로그인 처리
  res.json({ token: '...' });
});

// 커스텀 Rate Limiter
const userRequests = new Map();

function customRateLimiter(maxRequests = 10, windowMs = 60000) {
  return (req, res, next) => {
    const userId = req.user?.id || req.ip;
    const now = Date.now();

    if (!userRequests.has(userId)) {
      userRequests.set(userId, []);
    }

    const userRequestTimes = userRequests.get(userId);

    // 윈도우 외의 요청 제거
    const recentRequests = userRequestTimes.filter(
      (time) => now - time < windowMs,
    );

    if (recentRequests.length >= maxRequests) {
      const resetTime = Math.ceil((recentRequests[0] + windowMs - now) / 1000);

      res.set('Retry-After', resetTime);
      return res.status(429).json({
        error: 'Too Many Requests',
        message: `Max ${maxRequests} requests per ${windowMs / 1000} seconds`,
        retryAfter: resetTime,
      });
    }

    recentRequests.push(now);
    userRequests.set(userId, recentRequests);

    // Rate limit 정보를 헤더에 추가
    res.set('X-RateLimit-Limit', maxRequests);
    res.set('X-RateLimit-Remaining', maxRequests - recentRequests.length);
    res.set('X-RateLimit-Reset', new Date(now + windowMs).toISOString());

    next();
  };
}

app.post(
  '/api/search',
  customRateLimiter(20, 60000), // 분당 20 요청
  (req, res) => {
    // 검색 처리
    res.json({ results: [] });
  },
);
```

```javascript
// 클라이언트 - 429 처리 (재시도 로직)
class ApiClientWithRetry {
  async request(url, options = {}, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        const response = await fetch(url, options);

        if (response.status === 429) {
          const retryAfter =
            response.headers.get('Retry-After') || Math.pow(2, attempt) * 1000; // Exponential backoff

          console.warn(
            `Rate limit 초과. ${retryAfter}ms 후 재시도... ` +
              `(${attempt}/${maxRetries})`,
          );

          // 대기 후 재시도
          await new Promise((resolve) =>
            setTimeout(resolve, parseInt(retryAfter) * 1000),
          );

          continue;
        }

        return response;
      } catch (error) {
        if (attempt === maxRetries) throw error;

        // 지수 백오프: 1초, 2초, 4초...
        const delay = Math.pow(2, attempt - 1) * 1000;
        console.warn(`요청 실패. ${delay}ms 후 재시도...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  async search(query) {
    const response = await this.request('/api/search', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query }),
    });

    if (response.ok) {
      return response.json();
    } else {
      throw new Error(`검색 실패: ${response.status}`);
    }
  }
}

// 사용
const api = new ApiClientWithRetry();
api
  .search('javascript')
  .then((results) => console.log(results))
  .catch((error) => console.error('최종 실패:', error));
```

# 5XX (서버 에러) - 서버에 문제 발생

5XX 상태 코드는 서버에서 요청을 처리하지 못했음을 의미합니다.

## 500 Internal Server Error (내부 서버 에러)

### 의미

서버에서 예상하지 못한 에러가 발생했습니다. 가장 일반적인 서버 에러입니다.

### 발생 상황

```
- 처리되지 않은 예외
- 데이터베이스 연결 실패
- 메모리 부족
- 코드의 논리 에러
```

### 코드 예제

```typescript
// Express - 에러 처리
app.get('/api/data', (req, res) => {
  try {
    // 무언가 수행
    const result = riskyOperation();
    res.json(result);
  } catch (error) {
    console.error('에러 발생:', error);

    res.status(500).json({
      error: 'Internal Server Error',
      message: 'An unexpected error occurred',
      requestId: req.id, // 디버깅을 위한 요청 ID
    });
  }
});

// 전역 에러 핸들러
app.use((error, req, res, next) => {
  console.error('처리되지 않은 에러:', error);

  // 에러를 로깅 시스템에 전송
  logError({
    message: error.message,
    stack: error.stack,
    url: req.url,
    method: req.method,
    userId: req.user?.id,
    timestamp: new Date(),
  });

  const statusCode = error.statusCode || 500;

  res.status(statusCode).json({
    error: 'Internal Server Error',
    message:
      process.env.NODE_ENV === 'production'
        ? 'An unexpected error occurred'
        : error.message,
    requestId: req.id,
  });
});

// 처리되지 않은 Promise 거부
process.on('unhandledRejection', (reason, promise) => {
  console.error('처리되지 않은 거부:', reason);
  // 에러 로깅
  logError({
    message: 'Unhandled Rejection',
    reason: reason,
    promise: promise,
  });
});

// 처리되지 않은 예외
process.on('uncaughtException', (error) => {
  console.error('처리되지 않은 예외:', error);
  // 에러 로깅
  logError({
    message: 'Uncaught Exception',
    error: error,
  });

  // 프로세스 종료 (재시작 필요)
  process.exit(1);
});
```

### 디버깅

```typescript
// 서버 로그 분석
// application.log 파일에서:
// [2024-01-29 10:30:45] ERROR: Cannot read property 'map' of undefined
// [2024-01-29 10:30:45] at /app/routes/users.js:45:12

// 원인 파악:
// 1. 변수가 undefined임
// 2. 존재하지 않는 라이브러리 메서드 호출
// 3. 데이터베이스 쿼리 실패

// 해결:
app.get('/api/users', async (req, res) => {
  try {
    const users = await User.findAll();

    // users가 null인지 확인
    if (!users) {
      return res.status(500).json({
        error: 'Database query failed',
      });
    }

    // users.map() 호출 안전
    const userList = users.map((user) => ({
      id: user.id,
      name: user.name,
    }));

    res.json(userList);
  } catch (error) {
    console.error('[ERROR] /api/users:', error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});
```

## 502 Bad Gateway (잘못된 게이트웨이)

### 의미

게이트웨이나 프록시가 상위 서버로부터 유효하지 않은 응답을 받았습니다.

### 발생 상황

```
- 업스트림 서버 다운
- 리버스 프록시 설정 오류
- 로드 밸런서 문제
- API 게이트웨이 실패
```

### 코드 예제

```typescript
// Nginx 설정 (502 방지)
server {
    listen 80;
    server_name example.com;

    upstream backend {
        server app1:3000;
        server app2:3000;
        server app3:3000;

        # 헬스 체크
        check interval=3000 rise=2 fall=3 timeout=1000;
    }

    location / {
        proxy_pass http://backend;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_write_timeout 30s;

        # 재시도 설정
        proxy_next_upstream error timeout invalid_header
                           http_500 http_502 http_503;
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
    }
}
```

```typescript
// Node.js에서 상위 서버 호출 시 에러 처리
app.get('/api/external-data', async (req, res) => {
  try {
    const response = await fetch('http://upstream-service:8080/data', {
      timeout: 5000, // 5초 타임아웃
    });

    if (!response.ok) {
      // 상위 서버가 에러 응답
      console.error(
        `Upstream error: ${response.status} ${response.statusText}`,
      );
      return res.status(502).json({
        error: 'Bad Gateway',
        message: 'Upstream service error',
      });
    }

    const data = await response.json();
    res.json(data);
  } catch (error) {
    console.error('Upstream connection error:', error);

    res.status(502).json({
      error: 'Bad Gateway',
      message: 'Unable to connect to upstream service',
    });
  }
});

// 서킷 브레이커 패턴 (자동 복구)
class CircuitBreaker {
  constructor(asyncFunction, options = {}) {
    this.asyncFunction = asyncFunction;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.successCount = 0;
    this.failureThreshold = options.failureThreshold || 5;
    this.successThreshold = options.successThreshold || 2;
    this.timeout = options.timeout || 60000; // 1분
    this.nextAttempt = Date.now();
  }

  async call(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
      this.successCount = 0;
    }

    try {
      const result = await this.asyncFunction(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;

    if (this.state === 'HALF_OPEN') {
      this.successCount++;
      if (this.successCount >= this.successThreshold) {
        this.state = 'CLOSED';
        console.log('Circuit breaker CLOSED (recovered)');
      }
    }
  }

  onFailure() {
    this.failureCount++;

    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.log('Circuit breaker OPEN (protecting)');
    }
  }
}

// 사용
const upstreamBreaker = new CircuitBreaker(
  async () => {
    const response = await fetch('http://upstream:8080/data');
    if (!response.ok) throw new Error('Upstream error');
    return response.json();
  },
  { failureThreshold: 3, timeout: 30000 },
);

app.get('/api/data', async (req, res) => {
  try {
    const data = await upstreamBreaker.call();
    res.json(data);
  } catch (error) {
    console.error('Circuit breaker error:', error.message);

    res.status(502).json({
      error: 'Bad Gateway',
      message: 'Service temporarily unavailable',
    });
  }
});
```

## 503 Service Unavailable (서비스 이용 불가)

### 의미

서버가 임시적으로 요청을 처리할 수 없습니다. 유지보수 중 또는 과부하 상태입니다.

### 발생 상황

```
- 서버 점검/유지보수
- 과부하 상태
- 데이터베이스 재시작
- 일시적 리소스 부족
```

### 코드 예제

```typescript
// Express - 유지보수 모드
let maintenanceMode = false;

app.use((req, res, next) => {
  if (maintenanceMode) {
    const retryAfter = Math.ceil((maintenanceEndTime - Date.now()) / 1000);

    res.set('Retry-After', retryAfter);
    return res.status(503).json({
      error: 'Service Unavailable',
      message: 'Server is under maintenance',
      estimatedTime: retryAfter, // 초 단위
    });
  }
  next();
});

// 유지보수 시작
app.post('/admin/maintenance/start', adminAuth, (req, res) => {
  maintenanceMode = true;
  const duration = req.body.duration || 3600; // 기본 1시간
  maintenanceEndTime = Date.now() + duration * 1000;

  res.json({
    message: 'Maintenance mode enabled',
    duration,
  });
});

// 유지보수 종료
app.post('/admin/maintenance/end', adminAuth, (req, res) => {
  maintenanceMode = false;
  res.json({ message: 'Maintenance mode disabled' });
});

// 메모리 부족 시 자동 503
const os = require('os');

setInterval(() => {
  const freeMemory = os.freemem();
  const totalMemory = os.totalmem();
  const usedPercent = ((totalMemory - freeMemory) / totalMemory) * 100;

  if (usedPercent > 90) {
    maintenanceMode = true;
    console.error('Memory usage critical: ' + usedPercent.toFixed(2) + '%');
  } else if (usedPercent < 70) {
    maintenanceMode = false;
  }
}, 5000);
```

```javascript
// 클라이언트 - 503 처리 (재시도)
async function fetchWithRetry(url, maxAttempts = 5) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const response = await fetch(url);

      if (response.status === 503) {
        const retryAfter = response.headers.get('Retry-After');
        const waitTime = retryAfter
          ? parseInt(retryAfter) * 1000
          : Math.pow(2, attempt) * 1000;

        console.log(
          `서버 이용 불가. ${waitTime}ms 후 재시도... ` +
            `(${attempt}/${maxAttempts})`,
        );

        await new Promise((resolve) => setTimeout(resolve, waitTime));
        continue;
      }

      if (response.ok) {
        return response.json();
      }

      throw new Error(`HTTP ${response.status}`);
    } catch (error) {
      if (attempt === maxAttempts) {
        throw new Error(`최종 실패 (${maxAttempts}회 재시도): ${error}`);
      }

      const delay = Math.pow(2, attempt - 1) * 1000;
      console.warn(`시도 ${attempt} 실패. ${delay}ms 후 재시도...`);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
}

// 사용
fetchWithRetry('/api/data')
  .then((data) => console.log('데이터:', data))
  .catch((error) => console.error(error));
```

## 504 Gateway Timeout (게이트웨이 타임아웃)

### 의미

게이트웨이가 상위 서버로부터 시간 내에 응답을 받지 못했습니다.

### 발생 상황

```
- 상위 서버가 응답 없음
- 네트워크 지연
- 장시간 처리 작업
- DNS 조회 실패
```

### 코드 예제

```typescript
// Node.js - 타임아웃 처리
app.get('/api/slow-operation', async (req, res) => {
  const timeout = 30000; // 30초

  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Operation timeout')), timeout);
  });

  try {
    // 슬로우 쿼리 또는 외부 API 호출
    const result = await Promise.race([slowDatabaseQuery(), timeoutPromise]);

    res.json(result);
  } catch (error) {
    if (error.message === 'Operation timeout') {
      console.error('요청 타임아웃');
      return res.status(504).json({
        error: 'Gateway Timeout',
        message: 'Operation took too long',
      });
    }

    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// Nginx - 타임아웃 설정
// location / {
//   proxy_pass http://backend;
//   proxy_connect_timeout 10s;
//   proxy_send_timeout 30s;
//   proxy_read_timeout 30s;
// }
```

# 프로덕션 체크리스트

## 에러 처리

- [ ] 모든 엔드포인트에서 try-catch
- [ ] 전역 에러 핸들러 구현
- [ ] 상황에 맞는 상태 코드 반환
- [ ] 에러 메시지 명확함
- [ ] 프로덕션에서는 민감 정보 숨김

## 보안

- [ ] 401/403 권한 검사
- [ ] Rate limiting 구현
- [ ] CORS 설정
- [ ] 입력 검증 (400 방지)
- [ ] SQL Injection 방지

## 성능

- [ ] 타임아웃 설정
- [ ] 캐싱 구현 (304)
- [ ] 서킷 브레이커 (502 방지)
- [ ] 로드 밸런싱
- [ ] 모니터링 및 로깅

## 테스트

- [ ] 각 상태 코드별 테스트
- [ ] 에러 시나리오 테스트
- [ ] 부하 테스트 (429, 503)
- [ ] 복구 시나리오 테스트

# 결론

HTTP 상태 코드를 이해하면:

✅ 웹 통신의 흐름을 명확히 파악
✅ 에러를 빠르게 진단 및 해결
✅ 사용자 경험 개선
✅ API 신뢰성 향상
✅ 프로덕션 문제 예방

**각 상태 코드에 맞는 적절한 응답을 반환하고, 사용자에게 명확한 정보를 제공하는 것이 좋은 API의 특징입니다!**

## 참고 링크

- [MDN - HTTP 응답 상태 코드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [MDN - HTTP 개요](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [RFC 9110 - HTTP Semantics (Status Codes)](https://www.rfc-editor.org/rfc/rfc9110.html#name-status-codes)
