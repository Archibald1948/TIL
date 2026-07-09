# Claude Code

터미널과 IDE 안에서 동작하는 **AI 코딩 에이전트**. 자연어로 지시하면 코드베이스를 직접 탐색·수정하고, 명령 실행·git 작업까지 멀티스텝으로 처리한다.

## 핵심 요소

- **에이전트형 작업** — 단순 자동완성이 아니라 "기능 추가해줘" 수준의 작업을 파일 탐색 → 수정 → 테스트까지 알아서 수행
- **CLAUDE.md** — 프로젝트 규칙·컨텍스트를 적어두는 메모리 파일. 매 세션 자동으로 읽어 들임
- **Slash 커맨드 / Skills** — `/init`, `/review` 등 내장 명령 + 커스텀 스킬로 반복 작업 정의
- **MCP** — Figma, Gmail, DB 등 외부 도구를 표준 프로토콜로 연결
- **Hooks / 권한 모드** — 도구 호출 가로채기, 자동 실행 허용 범위 제어
- **실행 환경** — CLI, VS Code/JetBrains 확장, 데스크톱·웹 앱

## 설치

```bash
# npm (Node.js 18+)
npm install -g @anthropic-ai/claude-code

# 또는 네이티브 설치 스크립트
curl -fsSL https://claude.ai/install.sh | bash

# 실행
claude
```

> `sudo npm install -g` 는 권한 문제를 일으키니 사용하지 말 것.

## 알아두면 좋을 거

터미널과 vs code에서 쓰는 클로드의 성능은 동일하다. 단지 각각의 도구만의 장단점 정도만 있을 뿐이다.
그리고 cowork도 마찬가지이지만, 클로드 코드도 클로드 크롬 확장 프로그램을 설치해서 내 브라우저를 직접 확인하고 조작할 수 있다.

## 링크

- 제품 소개: https://claude.com/product/claude-code
- 공식 문서: https://code.claude.com/docs
- npm 패키지: https://www.npmjs.com/package/@anthropic-ai/claude-code
