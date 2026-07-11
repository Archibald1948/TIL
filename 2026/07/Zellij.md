# Zellij - tmux를 몰라도 바로 쓰는 현대적 터미널 멀티플렉서

## 들어가며

터미널 하나로 여러 작업을 동시에 하고 싶을 때, 이런 고민 해본 적 있나요?

- 서버 로그, 코드 편집, 테스트 실행을 창 여러 개 띄워 alt-tab 지옥
- SSH 세션이 끊기면 하던 작업이 다 날아감
- tmux를 써보려니 `Ctrl-b` 조합을 외워야 하고, 설정 파일이 암호 같음
- 화면 분할이 되긴 하는데 지금 무슨 단축키를 눌러야 하는지 매번 까먹음

**Zellij**는 이 문제들을 "안내가 화면에 뜨는" 방식으로 풀어냅니다.
Rust로 만들어진 터미널 멀티플렉서로, tmux의 기능은 그대로 가지면서
**단축키를 외우지 않아도 되도록** 화면 하단에 상황별 힌트를 계속 보여줍니다.

## 공식 사이트

https://zellij.dev

## 멀티플렉서가 왜 필요한가?

```
멀티플렉서 없이:
- 터미널 탭/창을 OS 단에서 여러 개 관리
- SSH 끊기면 → 실행 중이던 프로세스 종료됨 😢
- 레이아웃을 매번 손으로 다시 배치

멀티플렉서와 함께:
- 하나의 터미널 안에서 여러 pane / tab 분할
- 세션이 서버에 "붙어" 있어 detach/attach 가능
- SSH 끊겨도 작업은 계속 살아있음 🚀
- 레이아웃을 파일로 저장해 재사용
```

## Zellij vs tmux

| 항목            | tmux                        | Zellij                          |
| --------------- | --------------------------- | ------------------------------- |
| **언어**        | C                           | Rust                            |
| **학습곡선**    | 가파름 (단축키 암기 필수)   | 완만함 (화면에 힌트 표시)       |
| **기본 설정**   | 거의 없음 (직접 다 해야 함) | 바로 쓸 만한 기본값             |
| **설정 파일**   | `.tmux.conf` (독자 문법)    | `config.kdl` (KDL, 읽기 쉬움)   |
| **레이아웃**    | 스크립트로 구성             | KDL 레이아웃 파일               |
| **플러그인**    | TPM + 셸 스크립트           | WebAssembly 기반 플러그인       |
| **세션 관리**   | 명령어로                    | 명령어 + 내장 세션 매니저 UI    |

tmux가 익숙한 파워유저라면 tmux를 계속 써도 됩니다.
**처음 시작하거나, "왜 이걸 외워야 하지?" 싶은 사람에게 Zellij가 특히 잘 맞습니다.**

## 설치

```bash
# macOS
brew install zellij

# Rust cargo
cargo install --locked zellij

# Arch Linux
pacman -S zellij

# 간단 설치 스크립트
bash <(curl -L zellij.dev/launch)

# 실행
zellij
```

실행하면 하단에 상태바가 뜨고, `Ctrl` 조합 키 힌트들이 보입니다. 이게 Zellij의 핵심 UX입니다.

## 핵심 개념 3가지

```
Session (세션)
└─ 서버에 살아있는 작업 공간. detach 해도 유지됨.
   └─ Tab (탭)
      └─ 하나의 작업 묶음. 여러 개 만들어 이름 붙일 수 있음.
         └─ Pane (페인)
            └─ 실제 셸이 도는 분할 영역. 가로/세로로 쪼갬.
```

- **Pane**: 화면을 나눈 각각의 셸 조각
- **Tab**: pane들의 묶음 (브라우저 탭처럼)
- **Session**: tab들의 묶음, 서버 측에 persist 됨

## 모드(Mode) 기반 조작

Zellij는 "지금 어떤 모드냐"에 따라 키가 달라집니다. 하단 바가 항상 현재 모드를 알려줍니다.

```
Ctrl + p   → Pane 모드   (페인 분할/이동/닫기)
Ctrl + t   → Tab 모드    (탭 생성/이동/이름)
Ctrl + n   → Resize 모드 (페인 크기 조절)
Ctrl + s   → Scroll 모드 (스크롤/검색)
Ctrl + o   → Session 모드(세션 detach/매니저)
Ctrl + h   → Move 모드   (페인 위치 이동)
Ctrl + q   → Quit (종료)

Esc 또는 Enter → 기본(Normal) 모드로 복귀
```

> 외울 필요 없음: `Ctrl` 만 눌러도 상단/하단에 어떤 모드가 있는지 힌트가 뜬다.

## 실전: 자주 쓰는 조작

### 페인 다루기 (Ctrl+p 이후)

```
Ctrl + p, 그 다음:
  n        → 새 페인
  d        → 아래로 분할 (down)
  r        → 오른쪽으로 분할 (right)
  x        → 현재 페인 닫기
  f        → 현재 페인 전체화면 토글 (fullscreen)
  방향키    → 페인 간 포커스 이동
```

### 탭 다루기 (Ctrl+t 이후)

```
Ctrl + t, 그 다음:
  n        → 새 탭
  x        → 탭 닫기
  r        → 탭 이름 변경 (rename)
  숫자키    → 해당 번호 탭으로 이동
  방향키    → 이전/다음 탭
```

### 세션 다루기

```
Ctrl + o, 그 다음:
  d        → detach (세션은 살려두고 빠져나옴)
  w        → 세션 매니저 열기

# 터미널에서 직접
zellij ls                 # 살아있는 세션 목록
zellij attach <이름>       # 세션에 다시 붙기
zellij attach -c work     # work 세션 없으면 만들고 붙기
zellij kill-session <이름> # 세션 종료
```

**핵심**: `Ctrl+o d` 로 detach → SSH를 끊어도 세션은 서버에서 계속 실행 → 나중에 `zellij attach`.

## 설정: config.kdl

Zellij는 KDL 형식을 씁니다. tmux 설정보다 훨씬 읽기 편합니다.

```kdl
// ~/.config/zellij/config.kdl

// 기본 셸
default_shell "zsh"

// 마우스 지원
mouse_mode true

// 복사에 사용할 명령 (macOS)
copy_command "pbcopy"

// UI: 상태바를 간결하게
pane_frames false

// 테마
theme "catppuccin-mocha"

// 시작 시 팁 배너 끄기
show_startup_tips false

// 키바인딩 커스터마이즈 예시
keybinds {
    normal {
        // Alt+숫자 로 탭 바로 이동
        bind "Alt 1" { GoToTab 1; }
        bind "Alt 2" { GoToTab 2; }
    }
}
```

설정 확인/생성:

```bash
# 기본 설정을 파일로 뽑아보기
zellij setup --dump-config > ~/.config/zellij/config.kdl

# 사용 가능한 테마 확인
zellij setup --check
```

## 레이아웃: 작업 환경을 파일로 저장

가장 강력한 기능. 자주 쓰는 화면 배치를 미리 정의해두고 한 번에 띄웁니다.

```kdl
// ~/.config/zellij/layouts/dev.kdl
layout {
    tab name="editor" {
        pane command="nvim"
    }
    tab name="server" split_direction="vertical" {
        pane command="npm" {
            args "run" "dev"
        }
        pane                       // 빈 셸
    }
    tab name="git" {
        pane command="lazygit"
    }
}
```

```bash
# 이 레이아웃으로 세션 시작
zellij --layout dev

# 또는 이름으로 (layouts 디렉토리에 있으면)
zellij --layout dev --session mywork
```

editor / server / git 탭이 원하는 분할로 한 방에 구성됩니다.
프로젝트마다 레이아웃을 만들어두면 "터미널 켜고 세팅"이 사라집니다.

## 알아두면 좋은 것

```
- 마우스 지원이 기본: 클릭으로 페인 포커스, 드래그로 리사이즈, 탭 클릭 이동 가능
- Floating pane: Ctrl+p 후 w → 떠 있는 페인 (잠깐 명령어 실행할 때 편함)
- 스크롤백 검색: Ctrl+s 후 / 로 검색, 에디터로 스크롤백 편집도 가능
- 플러그인: WASM 기반이라 안전. 세션 매니저·상태바 자체가 플러그인으로 동작
- tmux 유저용: --tmux 유사 키 모드는 없지만, keybinds 로 tmux 스타일 흉내 가능
```

## 흔한 문제 해결

```
detach 했는데 세션이 안 보임
→ zellij ls 로 목록 확인, 이름/서버 사용자 계정 확인

색이 이상하게 나옴
→ 터미널이 truecolor 지원하는지 확인 (TERM, COLORTERM)
→ config.kdl 의 theme 변경

한글/이모지 폭이 깨짐
→ 터미널의 유니코드/폰트 설정 확인

단축키가 셸 앱과 충돌 (예: Ctrl+t)
→ config.kdl keybinds 에서 리바인드
```

## 체크리스트

Zellij 세팅하기:

```
[ ] brew install zellij (또는 cargo)
[ ] zellij 실행해 하단 힌트 바 확인
[ ] Pane/Tab/Session 모드 손에 익히기
[ ] Ctrl+o d 로 detach → zellij attach 로 복귀 연습
[ ] --dump-config 로 config.kdl 생성 후 취향대로 수정
[ ] 자주 쓰는 작업의 레이아웃 파일 작성 (layouts/*.kdl)
[ ] SSH 서버에도 설치해 세션 유지 활용
```

## 결론

Zellij는:

✅ **낮은 진입장벽**: 화면에 뜨는 힌트로 단축키 암기 부담 제거
✅ **끊겨도 안전**: detach/attach로 SSH가 끊겨도 작업 유지
✅ **읽기 쉬운 설정**: KDL 기반 config·layout
✅ **재현 가능한 환경**: 레이아웃 파일로 작업 공간을 한 번에 복원
✅ **현대적**: Rust + WASM 플러그인, 마우스 지원 기본

tmux의 강력함은 원하지만 그 학습곡선은 부담스러웠다면, **Zellij로 시작하세요.**
익숙해진 뒤 config.kdl과 레이아웃으로 나만의 작업 환경을 조립해 가면 됩니다.
