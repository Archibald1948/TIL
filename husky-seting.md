# Husky로 커밋 컨벤션 검증 자동화하기

## 📌 왜 도입했나

협업이든 1인 TIL이든 커밋 메시지가 들쭉날쭉하면 나중에 `git log` 한 줄 보는 것조차 피곤하다.
이 글에서는 TIL 저장소에 **husky + commitlint** 조합으로
"커밋 컨벤션 검증" 기능 하나만 깔끔하게 붙인 과정을 기록한다.

- **husky**: Git hook을 손쉽게 관리하게 해주는 도구
- **commitlint**: 커밋 메시지가 컨벤션(여기서는 [Conventional Commits](https://www.conventionalcommits.org/))을 따르는지 검사
- 두 개를 합치면 → `commit-msg` hook에서 컨벤션을 어긴 커밋을 **차단**

## 1. 사전 준비

이 저장소는 원래 순수 마크다운 모음이라 `package.json`이 없었다.
husky는 npm 패키지이므로 먼저 Node 프로젝트로 초기화해야 한다.

```bash
node -v   # v22.14.0
npm -v    # 10.9.2

npm init -y
```

`package.json`이 생기면 준비 끝.

## 2. husky와 commitlint 설치

devDependency로 세 개를 한 번에 설치한다.

```bash
npm install --save-dev husky @commitlint/cli @commitlint/config-conventional
```

| 패키지 | 역할 |
| --- | --- |
| `husky` | Git hook 설정 자동화 |
| `@commitlint/cli` | 커밋 메시지를 검사하는 CLI |
| `@commitlint/config-conventional` | Conventional Commits 규칙 프리셋 |

## 3. husky 초기화

```bash
npx husky init
```

이 명령 하나로 다음 작업이 한꺼번에 일어난다.

1. `.husky/` 디렉터리 생성
2. 샘플 `pre-commit` hook 파일 생성 (`npm test` 한 줄)
3. `package.json`의 `scripts`에 `"prepare": "husky"` 자동 추가

`prepare` 스크립트 덕분에 나중에 누군가 이 저장소를 클론하고 `npm install`만 해도
hook이 자동으로 활성화된다. 직접 등록할 필요가 없어 편하다.

## 4. 필요 없는 hook 정리

이번 목표는 **커밋 컨벤션 검증만** 붙이는 것이라
`npx husky init`이 만들어 둔 기본 `pre-commit`은 지운다.

```bash
rm .husky/pre-commit
```

> lint-staged 같은 걸 추가로 도입할 계획이 있다면 남겨둬도 되지만,
> "딱 필요한 것만"이라는 원칙에 맞춰 일단 제거했다.

## 5. `commit-msg` hook 작성

`.husky/commit-msg` 파일을 만들고 아래 한 줄만 적어준다.

```sh
npx --no -- commitlint --edit "$1"
```

- `$1`에는 Git이 만든 임시 커밋 메시지 파일 경로가 들어온다.
- `--edit` 옵션으로 그 파일을 읽어 검사한다.
- `--no` 플래그는 인터넷에서 패키지를 새로 받지 않게 막아준다(이미 설치된 commitlint만 사용).

> husky v9부터는 예전처럼 셔뱅(`#!/usr/bin/env sh`)이나
> `. "$(dirname -- "$0")/_/husky.sh"` 부트스트랩 줄을 적지 않아도 된다.
> 실행 권한만 있으면 OK.

```bash
chmod +x .husky/commit-msg
```

## 6. commitlint 규칙 정의 — `commitlint.config.js`

루트에 `commitlint.config.js`를 만들고 사용할 규칙을 명시한다.

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'test',
        'chore',
        'perf',
        'ci',
        'build',
        'revert',
      ],
    ],
    'subject-case': [0],
    'subject-empty': [2, 'never'],
    'type-empty': [2, 'never'],
  },
};
```

규칙은 `[level, applicable, value]` 형태로 적는다.

- `level`: `0`=off, `1`=warning, `2`=error
- `applicable`: `always` | `never`
- `value`: 규칙별 값

이 설정의 의미는 이렇다.

- 커밋 타입은 위 11종 중 하나여야 함 (`feat`, `fix`, …)
- 제목 대소문자 규칙은 **비활성화** (한글 메시지를 자주 쓰기 때문)
- 제목과 타입은 비어 있으면 안 됨

## 7. 동작 확인

설정이 끝났으면 commitlint를 직접 호출해서 검증한다.

```bash
echo "잘못된 커밋 메시지" | npx commitlint
# ✖ subject may not be empty [subject-empty]
# ✖ type may not be empty [type-empty]

echo "feat: add husky setup" | npx commitlint
# (출력 없음 = 통과)
```

실제 커밋에서는 이런 모습이 된다.

```bash
git commit -m "오늘 한 거"
# husky - commit-msg hook exited with code 1 (error)
# → 차단됨

git commit -m "chore: add husky and commitlint"
# → 통과
```

## 8. Conventional Commits 치트시트

| type | 언제 |
| --- | --- |
| `feat` | 새 기능 |
| `fix` | 버그 수정 |
| `docs` | 문서만 변경 (README, TIL 글 등) |
| `style` | 포맷팅, 세미콜론 같은 비기능 변경 |
| `refactor` | 동작 변화 없는 코드 개선 |
| `test` | 테스트 추가/수정 |
| `chore` | 빌드/도구/설정 변경 (husky 설치 같은 거) |
| `perf` | 성능 개선 |
| `ci` | CI 설정 변경 |
| `build` | 빌드 시스템, 의존성 변경 |
| `revert` | 이전 커밋 되돌리기 |

기본 포맷:
```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

예: `docs(react): hooks 정리 추가`

## 9. 정리

이번에 추가/생성한 파일은 결국 다음 4개뿐이다.

```
.husky/commit-msg          # 커밋 메시지 검증 hook
commitlint.config.js       # 검증 규칙
package.json               # 의존성 + prepare 스크립트
package-lock.json
```

가볍게 붙였지만 효과는 확실하다.
앞으로는 컨벤션을 어긴 커밋이 애초에 만들어지지 않으므로,
`git log`만 봐도 어떤 종류의 작업이었는지가 한눈에 들어온다.

다음 단계로 욕심을 내본다면:
- **lint-staged** + `pre-commit`으로 마크다운 lint 자동화
- **commitizen**으로 대화형 커밋 작성 UX
- GitHub Actions에서 PR 제목까지 commitlint로 검사

필요해질 때 그때 붙이면 된다. 오버엔지니어링은 금물.
