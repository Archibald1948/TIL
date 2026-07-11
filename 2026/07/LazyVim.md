# LazyVim - Neovim 설정 지옥에서 벗어나는 가장 빠른 길

## 들어가며

Neovim을 처음 써보려고 마음먹은 당신, 이런 벽에 부딪힌 적 있나요?

- `init.lua`에 뭘 적어야 할지 감이 안 온다
- 플러그인 하나 설치하는데 매니저부터 골라야 한다 (packer? lazy? vim-plug?)
- LSP, 자동완성, 파일 탐색기를 각각 따로 설정하다 저녁이 다 간다
- 남의 dotfiles를 복붙했더니 왜 되는지도 모른 채 돌아간다

**LazyVim**은 이 모든 삽질을 건너뛰게 해줍니다.
Neovim을 "VSCode처럼 바로 쓸 수 있는 IDE"로 만들어주는, 잘 정돈된 기본 설정 모음(distribution)입니다.
`folke`(lazy.nvim, which-key, tokyonight 저자)가 만들어서 신뢰도도 높습니다.

## 공식 사이트

https://www.lazyvim.org

## LazyVim은 정확히 무엇인가?

```
LazyVim = Neovim + lazy.nvim(플러그인 매니저) + 엄선된 기본 플러그인 + 합리적 기본 키맵

즉, "설정 프레임워크"다.
- 맨땅의 Neovim: 모든 걸 직접 조립 (자유롭지만 오래 걸림)
- LazyVim:       완성품에서 시작해 필요한 것만 덜어내고 더함
```

### 비슷한 것들과의 차이

| 방식               | 특징                                    | 추천 대상              |
| ------------------ | --------------------------------------- | ---------------------- |
| **순정 Neovim**    | 완전한 자유, 완전한 노가다              | Vim 내부를 깊게 파려는 사람 |
| **LazyVim**        | 모듈식 구조, 끄고 켜기 쉬움, 문서 좋음  | 대부분의 실사용자      |
| **NvChad**         | 예쁜 UI, 빠름, 구조가 독특함            | 미관 우선              |
| **AstroNvim**      | 기능 풍부, 다소 무거움                  | 올인원을 원하는 사람   |
| **LunarVim**       | 개발 정체됨                             | (지금은 비추천)        |

핵심은 **LazyVim이 "블랙박스"가 아니라는 점**입니다.
모든 것이 `lazy.nvim`의 플러그인 스펙으로 되어 있어, 언제든 열어보고 덮어쓸 수 있습니다.

## 설치

### 사전 요구사항

```bash
# Neovim 0.9.0 이상 (0.10+ 권장)
nvim --version

# 필수/권장 도구
brew install neovim ripgrep fd lazygit
# ripgrep : 텔레스코프 검색(grep)
# fd      : 빠른 파일 찾기
# lazygit : 내장 Git UI

# Nerd Font (아이콘 표시용) - 터미널 폰트로 지정해야 함
brew install --cask font-jetbrains-mono-nerd-font
```

### 스타터 설치

```bash
# 기존 설정 백업 (있다면)
mv ~/.config/nvim ~/.config/nvim.bak
mv ~/.local/share/nvim ~/.local/share/nvim.bak

# LazyVim 스타터 클론
git clone https://github.com/LazyVim/starter ~/.config/nvim

# .git 제거 후 내 레포로 관리
rm -rf ~/.config/nvim/.git

# 실행 → 첫 실행 시 플러그인 자동 설치
nvim
```

첫 실행하면 `lazy.nvim`이 알아서 플러그인을 전부 내려받습니다. 잠시 기다리면 끝.

## 디렉토리 구조 이해하기

LazyVim의 강력함은 이 단순한 구조에서 나옵니다.

```
~/.config/nvim/
├── init.lua                 # 진입점 (건드릴 일 거의 없음)
├── lua/
│   ├── config/
│   │   ├── autocmds.lua     # 내 자동명령
│   │   ├── keymaps.lua      # 내 키맵 추가
│   │   ├── lazy.lua         # lazy.nvim 부트스트랩
│   │   └── options.lua      # 내 옵션 오버라이드
│   └── plugins/             # ★ 여기가 핵심
│       ├── example.lua      # 플러그인 추가/설정 파일들
│       └── ...
└── lazy-lock.json           # 플러그인 버전 잠금 (커밋할 것)
```

**규칙은 딱 하나**: `lua/plugins/` 안에 `.lua` 파일을 만들면 LazyVim이 자동으로 읽어들입니다.
파일 이름은 자유. 원하는 만큼 쪼개도 됩니다.

## 실전: 커스터마이징 패턴

### 1. 옵션 바꾸기

```lua
-- lua/config/options.lua
-- vim.g / vim.opt 는 LazyVim 기본값 위에 덧씌워진다
vim.opt.relativenumber = false   -- 상대 줄번호 끄기
vim.opt.wrap = true              -- 줄바꿈 켜기
vim.g.autoformat = false         -- 저장 시 자동 포맷 끄기(전역)
```

### 2. 키맵 추가

```lua
-- lua/config/keymaps.lua
local map = vim.keymap.set

-- 저장 단축키
map("n", "<C-s>", "<cmd>w<cr>", { desc = "Save file" })

-- 창 이동을 더 편하게
map("n", "<C-h>", "<C-w>h", { desc = "Go to left window" })
map("n", "<C-l>", "<C-w>l", { desc = "Go to right window" })
```

### 3. 플러그인 추가

```lua
-- lua/plugins/extras.lua
return {
  -- 그냥 문자열만 반환하면 기본 설정으로 설치
  "nvim-tree/nvim-web-devicons",

  -- 옵션과 함께 설치
  {
    "folke/todo-comments.nvim",
    opts = { signs = true },
  },
}
```

### 4. LazyVim 기본 플러그인 설정 덮어쓰기

같은 플러그인 이름으로 스펙을 반환하면 **병합(merge)** 됩니다.

```lua
-- lua/plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  opts = {
    defaults = {
      layout_strategy = "horizontal",
      sorting_strategy = "ascending",
    },
  },
}
```

### 5. 기본 플러그인 아예 끄기

```lua
-- lua/plugins/disabled.lua
return {
  -- enabled = false 로 비활성화
  { "akinsho/bufferline.nvim", enabled = false },
}
```

## LazyVim Extras — 언어/도구 한 방에 켜기

가장 편한 기능. `:LazyExtras` 를 열면 미리 만들어둔 설정 묶음을 체크박스로 켜고 끕니다.

```
:LazyExtras

  lang.typescript   → TS/JS LSP + 포매터 + 디버깅 세트
  lang.python       → Python 개발 세트
  lang.rust         → rust-analyzer + crates.nvim
  lang.docker       → Dockerfile LSP
  coding.copilot     → GitHub Copilot
  editor.harpoon2    → 파일 북마크 점프
  util.dot           → dotfiles 편집 지원
```

체크만 하면 관련 플러그인 · LSP · 포매터가 전부 알아서 설치됩니다.
직접 켠 extras는 `lua/plugins/` 가 아니라 `lazyvim.json` 에 기록됩니다.

## 알아두면 좋은 기본 키맵

LazyVim은 `<Space>`가 리더 키입니다. `which-key`가 뜨니 외울 필요는 없습니다.

```
<Space>           → 명령 팔레트(which-key)가 뜬다, 여기서 다 탐색 가능

<Space><Space>    → 파일 찾기 (telescope)
<Space>/          → 프로젝트 전체 텍스트 검색 (grep)
<Space>e          → 파일 탐색기 (neo-tree) 토글
<Space>gg         → lazygit 실행
<Space>bd         → 현재 버퍼 닫기

<Space>ca         → 코드 액션 (LSP)
<Space>cr         → 이름 바꾸기 (rename)
gd                → 정의로 이동
K                 → 호버 문서 보기
<Space>cd         → 진단(에러) 메시지 보기

<S-h> / <S-l>     → 이전/다음 버퍼 (탭 이동)
<Space>l          → Lazy 플러그인 관리 UI
```

## 유지보수

```
:Lazy             # 플러그인 매니저 UI (설치/업데이트/프로파일)
:Lazy update      # 플러그인 전부 업데이트 → lazy-lock.json 갱신됨
:Lazy sync        # lock 파일 기준으로 동기화
:LazyHealth       # 문제 진단 (누락 도구, 폰트 등)
:LazyExtras       # extras 관리
```

`lazy-lock.json`을 Git으로 커밋해두면, 다른 컴퓨터에서도 **똑같은 플러그인 버전**으로 복원됩니다.
설정을 GitHub에 올려두는 것을 강력히 권장합니다.

## 흔한 문제 해결

```
아이콘이 □ 로 깨져 보임
→ 터미널 폰트를 Nerd Font로 지정하지 않았음

:LazyHealth 에서 ripgrep/fd 없다고 나옴
→ brew install ripgrep fd 로 설치

플러그인이 꼬였을 때
→ :Lazy restore (lock 파일 기준 복구)
→ 그래도 안 되면 ~/.local/share/nvim 지우고 nvim 재실행

설정 바꿨는데 반영 안 됨
→ nvim 완전히 재시작 (일부 설정은 재기동 필요)
```

## 체크리스트

LazyVim 세팅하기:

```
[ ] Neovim 0.9+ 설치
[ ] ripgrep, fd, lazygit 설치
[ ] Nerd Font 설치 및 터미널 폰트로 지정
[ ] LazyVim 스타터 클론 + .git 제거
[ ] 첫 실행으로 플러그인 자동 설치
[ ] options.lua / keymaps.lua 로 취향 반영
[ ] :LazyExtras 로 쓰는 언어 세트 켜기
[ ] lua/plugins/ 에 필요한 플러그인 추가
[ ] 설정을 내 GitHub 레포로 관리 (lazy-lock.json 포함)
```

## 결론

LazyVim은:

✅ **빠른 시작**: 맨땅 설정 없이 완성된 IDE 경험부터 출발
✅ **투명함**: 블랙박스가 아니라 lazy.nvim 스펙의 조합, 언제든 열어볼 수 있음
✅ **모듈식**: 플러그인을 파일 하나로 켜고 끄기
✅ **Extras**: 언어/도구 세트를 체크박스로 관리
✅ **재현성**: lazy-lock.json으로 어디서든 동일 환경 복원

"Neovim을 쓰고는 싶은데 설정에 며칠을 태우기는 싫다"면, **LazyVim에서 시작하세요.**
그리고 익숙해질수록 하나씩 열어보며 나만의 Neovim으로 다듬어 가면 됩니다.
