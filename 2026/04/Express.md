# Express - Node.js 웹 프레임워크의 완벽 가이드

## 들어가며

Node.js로 웹 서버를 만들 때 이런 문제들이 있나요?

```javascript
// ❌ 순수 Node.js로 HTTP 서버 만들기
const http = require('http')

const server = http.createServer((req, res) => {
  // 라우팅을 직접 구현해야 함
  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' })
    res.end('<h1>Home</h1>')
  } else if (req.url === '/about' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' })
    res.end('<h1>About</h1>')
  } else if (req.url === '/users' && req.method === 'POST') {
    // 요청 본문 파싱을 직접 구현
    let body = ''
    req.on('data', (chunk) => {
      body += chunk
    })
    req.on('end', () => {
      // 처리...
    })
  }
  // ... 계속 반복
})

server.listen(3000)

❌ 너무 복잡하고 반복적
❌ 미들웨어 지원 안 함
❌ 에러 처리가 번거로움
```

**Express**는 이 모든 번거로움을 제거합니다. 간단하고 우아한 API로 강력한 웹 서버를 만들 수 있습니다.

## 공식 사이트

https://expressjs.com/ko

# 1. Express란?

## 핵심 개념

```
Express = Node.js용 미니멀한 웹 프레임워크

할 수 있는 것:
✅ 라우팅 (경로에 따른 처리)
✅ 미들웨어 지원
✅ 템플릿 엔진 통합
✅ 정적 파일 서빙
✅ 요청/응답 처리
✅ 에러 핸들링
✅ 세션/쿠키 관리
```

## Express의 철학

```
Minimal and flexible web application framework.

최소한의 기능으로 최대한 유연하게.
개발자가 필요한 것만 선택해서 사용할 수 있다.
```

# 2. 설치 및 기본 설정

## 설치

```bash
# npm 초기화
npm init -y

# Express 설치
npm install express

# 선택 패키지 설치
npm install body-parser cors helmet nodemon
npm install -D @types/express @types/node
```

## 기본 서버

```javascript
// app.js
const express = require('express');

const app = express();
const PORT = 3000;

// 라우트 정의
app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

// 서버 실행
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

## package.json 스크립트

```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

## 실행

```bash
npm start      # 일반 실행
npm run dev    # 개발 모드 (파일 변경 시 자동 재시작)
```

# 3. 라우팅 기초

## HTTP 메서드별 라우팅

```javascript
const express = require('express');
const app = express();

// GET 요청
app.get('/', (req, res) => {
  res.send('GET 요청');
});

// POST 요청
app.post('/', (req, res) => {
  res.send('POST 요청');
});

// PUT 요청
app.put('/users/:id', (req, res) => {
  res.send(`PUT 요청 - ID: ${req.params.id}`);
});

// DELETE 요청
app.delete('/users/:id', (req, res) => {
  res.send(`DELETE 요청 - ID: ${req.params.id}`);
});

// 모든 메서드
app.all('/data', (req, res) => {
  res.send('모든 메서드 처리');
});

app.listen(3000);
```

## URL 파라미터

```javascript
// 동적 라우트
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`사용자 ID: ${userId}`);
});

// 여러 파라미터
app.get('/posts/:postId/comments/:commentId', (req, res) => {
  const { postId, commentId } = req.params;
  res.send(`포스트 ${postId}의 댓글 ${commentId}`);
});

// 정규식 패턴
app.get(/^\/users\/(\d+)$/, (req, res) => {
  res.send(`사용자 ID: ${req.params[0]}`);
});
```

## 쿼리 파라미터

```javascript
app.get('/search', (req, res) => {
  // /search?q=javascript&page=1
  const { q, page } = req.query;
  res.send(`검색어: ${q}, 페이지: ${page}`);
});

// 전체 쿼리 객체
app.get('/filter', (req, res) => {
  console.log(req.query); // { q: 'javascript', page: '1', ... }
  res.send(req.query);
});
```

# 4. 미들웨어 (Middleware)

## 미들웨어란?

```javascript
// 미들웨어는 req → middleware → res의 흐름
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} ${req.method} ${req.url}`);
  next(); // 다음 미들웨어로 진행
});
```

## 순서대로 실행

```javascript
const express = require('express');
const app = express();

// 미들웨어 1
app.use((req, res, next) => {
  console.log('미들웨어 1');
  next();
});

// 미들웨어 2
app.use((req, res, next) => {
  console.log('미들웨어 2');
  next();
});

// 라우트 핸들러
app.get('/', (req, res) => {
  console.log('라우트 핸들러');
  res.send('응답');
});

// 요청이 오면 출력 순서:
// 미들웨어 1
// 미들웨어 2
// 라우트 핸들러
```

## 일반적인 미들웨어

```javascript
const express = require('express');
const app = express();

// JSON 파싱
app.use(express.json());

// URL 인코딩된 데이터 파싱
app.use(express.urlencoded({ extended: true }));

// 정적 파일 서빙
app.use(express.static('public'));

// 커스텀 미들웨어
app.use((req, res, next) => {
  req.timestamp = new Date();
  next();
});

// 특정 경로에만 미들웨어 적용
app.use('/admin', (req, res, next) => {
  if (!isAuthenticated(req)) {
    return res.status(401).send('Unauthorized');
  }
  next();
});

app.get('/admin/users', (req, res) => {
  res.send('Admin users page');
});
```

## 에러 핸들링 미들웨어

```javascript
// 에러 핸들러는 4개의 매개변수를 받아야 함
app.use((err, req, res, next) => {
  console.error(err.stack);

  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';

  res.status(status).json({
    error: {
      status,
      message,
    },
  });
});
```

# 5. 요청/응답 처리

## 요청 (Request) 객체

```javascript
app.post('/api/users', (req, res) => {
  // 요청 본문
  console.log(req.body); // { name: 'John', email: 'john@example.com' }

  // URL 파라미터
  console.log(req.params);

  // 쿼리 파라미터
  console.log(req.query);

  // 헤더
  console.log(req.headers);
  console.log(req.get('Authorization'));

  // 메서드
  console.log(req.method); // 'POST'

  // 경로
  console.log(req.path); // '/api/users'
  console.log(req.url); // '/api/users?sort=name'

  // IP 주소
  console.log(req.ip);
  console.log(req.ips);

  // 쿠키
  console.log(req.cookies);
});
```

## 응답 (Response) 객체

```javascript
app.get('/', (req, res) => {
  // 문자열 또는 객체 전송
  res.send('Hello');
  res.send({ name: 'John' });

  // JSON 전송
  res.json({ status: 'ok' });

  // HTML 전송
  res.type('text/html').send('<h1>Hello</h1>');

  // 파일 전송
  res.download('file.pdf');
  res.sendFile(__dirname + '/file.pdf');

  // 상태 코드와 함께
  res.status(201).json({ id: 1, name: 'John' });
  res.status(404).send('Not Found');

  // 헤더 설정
  res.set('Content-Type', 'application/json');
  res.set({ 'X-Custom': 'value' });

  // 쿠키 설정
  res.cookie('userId', '123', { maxAge: 900000 });

  // 리다이렉트
  res.redirect('/users');
  res.redirect(301, 'https://example.com');

  // 렌더링 (템플릿 엔진 사용)
  res.render('index', { title: 'Home' });
});
```

# 6. 실전 예제

## REST API 만들기

```javascript
const express = require('express');
const app = express();

// 미들웨어
app.use(express.json());

// 임시 데이터
let users = [
  { id: 1, name: 'John', email: 'john@example.com' },
  { id: 2, name: 'Jane', email: 'jane@example.com' },
];

// 모든 사용자 조회
app.get('/api/users', (req, res) => {
  res.json(users);
});

// 특정 사용자 조회
app.get('/api/users/:id', (req, res) => {
  const user = users.find((u) => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json(user);
});

// 사용자 추가
app.post('/api/users', (req, res) => {
  // 입력값 검증
  if (!req.body.name || !req.body.email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }

  // 새 사용자 생성
  const newUser = {
    id: Math.max(...users.map((u) => u.id)) + 1,
    name: req.body.name,
    email: req.body.email,
  };

  users.push(newUser);
  res.status(201).json(newUser);
});

// 사용자 수정
app.put('/api/users/:id', (req, res) => {
  const user = users.find((u) => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  // 수정
  if (req.body.name) user.name = req.body.name;
  if (req.body.email) user.email = req.body.email;

  res.json(user);
});

// 사용자 삭제
app.delete('/api/users/:id', (req, res) => {
  const index = users.findIndex((u) => u.id === parseInt(req.params.id));

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  const deletedUser = users.splice(index, 1);
  res.json(deletedUser[0]);
});

// 에러 핸들러
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## 라우터 분리

```javascript
// routes/users.js
const express = require('express')
const router = express.Router()

// 라우터에 정의
router.get('/', (req, res) => {
  res.json([...])
})

router.get('/:id', (req, res) => {
  res.json({...})
})

router.post('/', (req, res) => {
  res.status(201).json({...})
})

module.exports = router

// app.js
const express = require('express')
const userRoutes = require('./routes/users')

const app = express()

app.use(express.json())

// 라우터 마운트
app.use('/api/users', userRoutes)

app.listen(3000)
```

# 7. 템플릿 엔진

## EJS 사용

```bash
npm install ejs
```

```javascript
const express = require('express');
const app = express();

// EJS 설정
app.set('view engine', 'ejs');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('index', { title: 'Home', name: 'John' });
});

app.listen(3000);
```

## EJS 템플릿 (views/index.ejs)

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
  </head>
  <body>
    <h1><%= title %></h1>
    <p>Hello, <%= name %>!</p>

    <!-- 조건문 -->
    <% if (name === 'John') { %>
    <p>You are John</p>
    <% } %>

    <!-- 반복문 -->
    <ul>
      <% users.forEach(user => { %>
      <li><%= user.name %></li>
      <% }) %>
    </ul>
  </body>
</html>
```

# 8. 인증 (Authentication)

## JWT 인증

```bash
npm install jsonwebtoken
```

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');

const app = express();
const SECRET = 'your-secret-key';

app.use(express.json());

// 로그인 (토큰 발급)
app.post('/login', (req, res) => {
  const user = {
    id: 1,
    name: 'John',
    email: 'john@example.com',
  };

  const token = jwt.sign(user, SECRET, { expiresIn: '1h' });

  res.json({ token });
});

// 인증 미들웨어
function verifyToken(req, res, next) {
  const token = req.headers['authorization']?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Token not provided' });
  }

  try {
    const decoded = jwt.verify(token, SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// 보호된 라우트
app.get('/api/profile', verifyToken, (req, res) => {
  res.json(req.user);
});

app.listen(3000);
```

# 9. 정적 파일 서빙

```javascript
const express = require('express');
const app = express();

// 정적 파일 디렉토리
app.use(express.static('public'));

// 특정 경로로 매핑
app.use('/static', express.static('public'));

// 여러 디렉토리
app.use(express.static('public'));
app.use(express.static('uploads'));

// 가상 경로 프리픽스
app.use('/files', express.static('uploads'));

app.listen(3000);
```

```
프로젝트 구조:
public/
  ├── css/
  │   └── style.css
  ├── js/
  │   └── script.js
  └── index.html

// public/index.html
<link rel="stylesheet" href="/css/style.css">
<script src="/js/script.js"></script>

// 또는 /static 매핑 시
<link rel="stylesheet" href="/static/css/style.css">
```

# 10. CORS 설정

```bash
npm install cors
```

```javascript
const express = require('express');
const cors = require('cors');

const app = express();

// 모든 CORS 요청 허용
app.use(cors());

// 특정 도메인만 허용
app.use(
  cors({
    origin: 'https://example.com',
    methods: ['GET', 'POST'],
    credentials: true,
  }),
);

// 특정 라우트에만 적용
app.get('/api/public', cors(), (req, res) => {
  res.json({ data: 'public' });
});

app.get('/api/private', (req, res) => {
  res.json({ data: 'private' });
});

app.listen(3000);
```

# 11. 에러 처리

## 에러 핸들링 패턴

```javascript
const express = require('express');
const app = express();

// 커스텀 에러 클래스
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

app.get('/api/users/:id', (req, res, next) => {
  try {
    const user = findUser(req.params.id);

    if (!user) {
      // 에러를 다음 미들웨어로 전달
      return next(new AppError('User not found', 404));
    }

    res.json(user);
  } catch (err) {
    next(err);
  }
});

// 전역 에러 핸들러 (모든 라우트 뒤)
app.use((err, req, res, next) => {
  console.error(err);

  const status = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';

  res.status(status).json({
    error: {
      status,
      message,
    },
  });
});

app.listen(3000);
```

# 12. 환경 변수

```bash
npm install dotenv
```

```
// .env
PORT=3000
NODE_ENV=development
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydb
SECRET_KEY=your-secret-key
```

```javascript
require('dotenv').config();

const express = require('express');
const app = express();

const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';

console.log(`Server running in ${NODE_ENV} mode`);

app.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
});
```

# 13. 로깅

## Morgan 미들웨어

```bash
npm install morgan
```

```javascript
const express = require('express');
const morgan = require('morgan');

const app = express();

// 기본 로깅
app.use(morgan('combined'));

// 커스텀 로깅
morgan.token('user', (req) => req.user?.id || 'anonymous');
app.use(morgan(':remote-addr - :user [:date[clf]] ":method :url" :status'));

// 파일에 로깅
const fs = require('fs');
const accessLogStream = fs.createWriteStream('access.log', { flags: 'a' });
app.use(morgan('combined', { stream: accessLogStream }));

app.listen(3000);
```

# 14. 데이터베이스 연결

## MongoDB 예제

```bash
npm install mongoose
```

```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();

// 데이터베이스 연결
mongoose.connect('mongodb://localhost/mydb', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// 스키마 정의
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  createdAt: { type: Date, default: Date.now },
});

const User = mongoose.model('User', userSchema);

// 라우트
app.get('/api/users', async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.post('/api/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

app.listen(3000);
```

# 15. 실행 환경별 설정

```javascript
const express = require('express');
const app = express();

// 개발 환경
if (process.env.NODE_ENV === 'development') {
  app.use((req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next();
  });
}

// 프로덕션 환경
if (process.env.NODE_ENV === 'production') {
  app.use((req, res, next) => {
    // 로깅을 파일로
    // 에러 상세 정보 숨기기
    next();
  });
}

// 공통 설정
app.use(express.json());
app.set('trust proxy', 1);

app.listen(process.env.PORT || 3000);
```

# 16. 체크리스트

Express 프로젝트 시작하기:

```
[ ] npm init
[ ] express 설치
[ ] app.js 생성
[ ] 기본 라우트 작성
[ ] 미들웨어 설정
[ ] 에러 핸들링
[ ] 환경 변수 설정
[ ] 데이터베이스 연결
[ ] 정적 파일 서빙
[ ] 로깅 설정
[ ] CORS 설정
[ ] 프로덕션 배포 설정
```

# 결론

Express는:

✅ 간단하고 배우기 쉬움
✅ 강력하고 유연함
✅ 큰 커뮤니티와 많은 플러그인
✅ 프로덕션 레벨의 안정성
✅ 빠른 개발 속도

**Node.js 웹 개발의 표준 프레임워크입니다!**

## 참고 링크

- [Express 공식 사이트](https://expressjs.com)
- [Express 한국어 문서](https://expressjs.com/ko/)
- [API 레퍼런스](https://expressjs.com/en/4x/api.html)
- [Express GitHub](https://github.com/expressjs/express)
