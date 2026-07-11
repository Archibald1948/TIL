# Storybook - 컴포넌트 개발의 완전한 혁신

## 들어가며

당신이 리액트 컴포넌트를 개발할 때, 매번 전체 애플리케이션을 실행해서 그 컴포넌트를 테스트했던 경험이 있나요?

```
문제 상황:

Button 컴포넌트를 수정했다면:
1. npm start
2. 애플리케이션 대기 (3-5초)
3. 버튼이 있는 페이지로 이동
4. 버튼 상태 변경 (hover, disabled 등)
5. 수정이 필요하면 1번부터 반복...

→ 아주 비효율적입니다!
```

Storybook은 이 모든 번거로움을 제거합니다. **각 컴포넌트를 독립적으로 개발하고 테스트**할 수 있는 환경을 제공합니다. 컴포넌트의 모든 상태(정상, 로딩, 에러 등)를 한눈에 볼 수 있고, 상호작용까지 테스트할 수 있습니다.

### 공식사이트 가이드라인

https://storybook.js.org/tutorials/intro-to-storybook/react/ko/get-started/

### 유튜브 영상

https://www.youtube.com/playlist?list=PLwtj4R9Hld3GUdKDhl5YcXWRUNkrtIkX0

## Storybook의 개념

### Storybook이란?

```
Storybook = 컴포넌트 개발 환경 + 문서화 도구 + 테스트 플랫폼

특징:
1. 컴포넌트 독립 실행 환경
   - 전체 앱 없이 컴포넌트만 개발
   - Hot reload로 빠른 피드백

2. 모든 상태 시각화
   - 정상 상태
   - 로딩 상태
   - 에러 상태
   - 빈 상태
   - 모든 Props 조합

3. 상호작용 테스트
   - 클릭, 입력 등 사용자 상호작용
   - 액션 기록

4. 자동 문서화
   - PropTypes 자동 추출
   - 코드 예제 자동 생성

5. 협업 도구
   - 디자이너와 개발자 협업
   - 컴포넌트 리뷰
```

### 예시로 이해하기

```
일반 개발 흐름:
App.tsx → 여러 페이지 → 여러 컴포넌트 → Button
                                    ↑
                            여기에 도달하기까지
                            복잡한 네비게이션 필요

Storybook 흐름:
Storybook → Button.stories.tsx → 직접 Button 개발
                             ↑
                    즉시 컴포넌트 개발 가능!
```

## Storybook 설치

### 1단계: 자동 설치

```bash
# 기존 React 프로젝트에 설치
npx storybook@latest init

# 또는 수동 설치
npm install -D storybook @storybook/react
```

### 2단계: 폴더 구조

```
my-app/
├── src/
│   ├── components/
│   │   ├── Button.tsx
│   │   └── Button.stories.tsx      # Storybook 파일
│   ├── App.tsx
│   └── index.tsx
├── .storybook/
│   ├── main.ts                     # Storybook 설정
│   └── preview.ts                  # 전역 설정
├── package.json
└── tsconfig.json
```

### 3단계: 실행

```bash
# Storybook 개발 서버 시작
npm run storybook

# localhost:6006 에서 확인
```

## 첫 Story 작성

### 기본 Story

```typescript
// components/Button.tsx
import React from 'react';

interface ButtonProps {
  label: string;
  onClick?: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}

export function Button({
  label,
  onClick,
  variant = 'primary',
  size = 'medium',
  disabled = false
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
}
```

```typescript
// components/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

// 메타 정보 (Storybook에서 어떻게 표시할지)
const meta = {
  title: 'Components/Button',           // Storybook 사이드바 경로
  component: Button,                    // 컴포넌트
  parameters: {
    layout: 'centered'                  // 가운데 정렬
  },
  tags: ['autodocs']                    // 자동 문서 생성
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Story 1: Primary 버튼
export const Primary: Story = {
  args: {
    label: 'Click Me',
    variant: 'primary'
  }
};

// Story 2: Secondary 버튼
export const Secondary: Story = {
  args: {
    label: 'Click Me',
    variant: 'secondary'
  }
};

// Story 3: Danger 버튼
export const Danger: Story = {
  args: {
    label: 'Delete',
    variant: 'danger'
  }
};

// Story 4: 비활성화 버튼
export const Disabled: Story = {
  args: {
    label: 'Disabled',
    disabled: true
  }
};

// Story 5: 작은 버튼
export const Small: Story = {
  args: {
    label: 'Small',
    size: 'small'
  }
};

// Story 6: 큰 버튼
export const Large: Story = {
  args: {
    label: 'Large',
    size: 'large'
  }
};

// Story 7: 클릭 이벤트 테스트
export const Interactive: Story = {
  args: {
    label: 'Click me!',
    onClick: () => alert('Button clicked!')
  }
};
```

### Storybook에서 보이는 형태

```
Components
  └─ Button
      ├─ Primary       (파란 버튼)
      ├─ Secondary     (회색 버튼)
      ├─ Danger        (빨간 버튼)
      ├─ Disabled      (비활성화 상태)
      ├─ Small         (작은 크기)
      ├─ Large         (큰 크기)
      └─ Interactive   (클릭 가능)

각 Story를 클릭하면:
- 미리보기 영역에서 실시간으로 컴포넌트 렌더링
- Props 조정 가능
- 자동 생성된 문서
- 상호작용 가능
```

## 고급 기능

### 1. Args (Props 제어)

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta = {
  title: 'Components/Button',
  component: Button,
  
  // Props 정의 (Storybook에서 제어 가능)
  argTypes: {
    label: {
      control: 'text',                   // 텍스트 입력
      description: '버튼 텍스트'
    },
    variant: {
      control: 'select',                 // 드롭다운 선택
      options: ['primary', 'secondary', 'danger'],
      description: '버튼 스타일'
    },
    size: {
      control: 'select',
      options: ['small', 'medium', 'large'],
      description: '버튼 크기'
    },
    disabled: {
      control: 'boolean',                // 체크박스
      description: '비활성화 여부'
    },
    onClick: {
      action: 'clicked'                  // 클릭 감지
    }
  }
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Default story - 모든 props 조정 가능
export const Default: Story = {
  args: {
    label: 'Button',
    variant: 'primary',
    size: 'medium',
    disabled: false
  }
};
```

### 2. Interactions (사용자 상호작용)

```typescript
import { fn } from '@storybook/test';
import { userEvent, within, expect } from '@storybook/test';

export const WithInteraction: Story = {
  args: {
    label: 'Click Me',
    onClick: fn()                        // 클릭 추적
  },
  // 자동으로 실행되는 인터랙션 시나리오
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button');
    
    // 1단계: 버튼 클릭
    await userEvent.click(button);
    
    // 2단계: onClick이 호출되었는지 확인
    expect(args.onClick).toHaveBeenCalled();
  }
};
```

### 3. Controls (Props 실시간 제어)

```typescript
// Storybook에서 자동으로 제어 UI 생성
export const Default: Story = {
  args: {
    label: 'Button',
    variant: 'primary',
    size: 'medium'
  }
};

// Storybook UI에서:
// - label 입력창에서 텍스트 변경
// - variant 드롭다운에서 선택
// - size 라디오 버튼에서 선택
// → 즉시 미리보기 업데이트
```

### 4. Docs (자동 문서 생성)

```typescript
const meta = {
  title: 'Components/Button',
  component: Button,
  
  parameters: {
    docs: {
      description: {
        component: '일반적인 버튼 컴포넌트입니다.'
      }
    }
  },
  
  argTypes: {
    label: {
      description: '버튼에 표시될 텍스트',
      table: {
        type: { summary: 'string' }
      }
    },
    variant: {
      description: '버튼의 시각적 스타일',
      table: {
        type: { summary: 'primary | secondary | danger' },
        defaultValue: { summary: 'primary' }
      }
    }
  }
} satisfies Meta<typeof Button>;
```

**Storybook에서 생성되는 문서:**
```
Button

일반적인 버튼 컴포넌트입니다.

Props:
- label (string): 버튼에 표시될 텍스트
- variant (primary | secondary | danger): 버튼의 시각적 스타일 (기본값: primary)
- size (small | medium | large): 버튼 크기 (기본값: medium)
- disabled (boolean): 비활성화 여부 (기본값: false)
- onClick (function): 클릭 핸들러

Examples:
<Button label="Click Me" variant="primary" />
<Button label="Delete" variant="danger" disabled />
```

## 실전 예제

### Input 컴포넌트 Story

```typescript
// components/Input.tsx
import React, { ChangeEvent } from 'react';

interface InputProps {
  value: string;
  onChange: (e: ChangeEvent<HTMLInputElement>) => void;
  placeholder?: string;
  type?: 'text' | 'email' | 'password';
  error?: string;
  disabled?: boolean;
}

export function Input({
  value,
  onChange,
  placeholder = 'Enter text...',
  type = 'text',
  error,
  disabled = false
}: InputProps) {
  return (
    <div className="input-wrapper">
      <input
        type={type}
        value={value}
        onChange={onChange}
        placeholder={placeholder}
        disabled={disabled}
        className={error ? 'input error' : 'input'}
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  );
}
```

```typescript
// components/Input.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { useState } from 'react';
import { fn } from '@storybook/test';
import { Input } from './Input';

const meta = {
  title: 'Components/Input',
  component: Input,
  
  argTypes: {
    value: {
      control: 'text',
      description: '입력값'
    },
    placeholder: {
      control: 'text',
      description: '플레이스홀더 텍스트'
    },
    type: {
      control: 'select',
      options: ['text', 'email', 'password'],
      description: '입력 타입'
    },
    error: {
      control: 'text',
      description: '에러 메시지'
    },
    disabled: {
      control: 'boolean',
      description: '비활성화 여부'
    },
    onChange: {
      action: 'changed'
    }
  }
} satisfies Meta<typeof Input>;

export default meta;
type Story = StoryObj<typeof meta>;

// 기본 입력창
export const Default: Story = {
  args: {
    value: '',
    placeholder: 'Enter your name...',
    onChange: fn()
  }
};

// 이메일 입력
export const Email: Story = {
  args: {
    type: 'email',
    placeholder: 'Enter your email...',
    onChange: fn()
  }
};

// 비밀번호 입력
export const Password: Story = {
  args: {
    type: 'password',
    placeholder: 'Enter password...',
    onChange: fn()
  }
};

// 에러 상태
export const WithError: Story = {
  args: {
    value: 'invalid-email',
    error: 'Please enter a valid email',
    onChange: fn()
  }
};

// 비활성화 상태
export const Disabled: Story = {
  args: {
    value: 'Disabled input',
    disabled: true,
    onChange: fn()
  }
};

// 상호작용 포함 (값 변경 테스트)
export const Interactive: Story = {
  args: {
    value: '',
    onChange: fn()
  },
  render: (args) => {
    const [value, setValue] = React.useState('');
    
    return (
      <Input
        {...args}
        value={value}
        onChange={(e) => {
          setValue(e.target.value);
          args.onChange(e);
        }}
      />
    );
  }
};
```

### Card 컴포넌트 Story (복잡한 컴포넌트)

```typescript
// components/Card.tsx
import React, { ReactNode } from 'react';

interface CardProps {
  title: string;
  description: string;
  image?: string;
  children?: ReactNode;
  onClick?: () => void;
  isLoading?: boolean;
  isError?: boolean;
}

export function Card({
  title,
  description,
  image,
  children,
  onClick,
  isLoading = false,
  isError = false
}: CardProps) {
  if (isLoading) {
    return (
      <div className="card loading">
        <div className="skeleton">로딩 중...</div>
      </div>
    );
  }

  if (isError) {
    return (
      <div className="card error">
        <p>에러가 발생했습니다.</p>
      </div>
    );
  }

  return (
    <div className="card" onClick={onClick}>
      {image && <img src={image} alt={title} className="card-image" />}
      <div className="card-content">
        <h2 className="card-title">{title}</h2>
        <p className="card-description">{description}</p>
        {children}
      </div>
    </div>
  );
}
```

```typescript
// components/Card.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Card } from './Card';

const meta = {
  title: 'Components/Card',
  component: Card,
  parameters: {
    layout: 'padded'
  }
} satisfies Meta<typeof Card>;

export default meta;
type Story = StoryObj<typeof meta>;

// 기본 카드
export const Default: Story = {
  args: {
    title: 'Card Title',
    description: 'This is a card description'
  }
};

// 이미지가 있는 카드
export const WithImage: Story = {
  args: {
    title: 'Product Card',
    description: 'Amazing product',
    image: 'https://via.placeholder.com/300x200'
  }
};

// 자식 컴포넌트가 있는 카드
export const WithChildren: Story = {
  args: {
    title: 'Card with Action',
    description: 'Card with custom content'
  },
  render: (args) => (
    <Card {...args}>
      <button>Learn More</button>
    </Card>
  )
};

// 로딩 상태
export const Loading: Story = {
  args: {
    title: 'Loading Card',
    description: 'This will not show',
    isLoading: true
  }
};

// 에러 상태
export const Error: Story = {
  args: {
    title: 'Error Card',
    description: 'This will not show',
    isError: true
  }
};

// 모든 상태 한눈에 보기 (Grid 레이아웃)
export const AllStates: Story = {
  render: () => (
    <div style={{ display: 'grid', gridTemplateColumns: 'repeat(2, 1fr)', gap: '20px' }}>
      <Card title="Normal" description="Normal state" />
      <Card
        title="With Image"
        description="Card with image"
        image="https://via.placeholder.com/300x200"
      />
      <Card title="Loading" description="Loading state" isLoading />
      <Card title="Error" description="Error state" isError />
    </div>
  )
};
```

## Storybook 설정

### .storybook/main.ts

```typescript
import type { StorybookConfig } from '@storybook/react-webpack5';

const config: StorybookConfig = {
  // 자동 감지할 Story 파일 패턴
  stories: [
    '../src/**/*.stories.{js,jsx,ts,tsx}'
  ],

  // Storybook addons (추가 기능)
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-coverage',
    '@storybook/addon-a11y',  // 접근성 테스트
    '@storybook/addon-themes'  // 테마 전환
  ],

  // 프레임워크
  framework: {
    name: '@storybook/react-webpack5',
    options: {}
  },

  // 문서 자동 생성
  docs: {
    autodocs: 'tag'
  }
};

export default config;
```

### .storybook/preview.ts

```typescript
import type { Preview } from '@storybook/react';

const preview: Preview = {
  // 모든 Story에 적용되는 기본 파라미터
  parameters: {
    layout: 'centered',
    actions: { argTypesRegex: '^on[A-Z].*' }
  },

  // 전역 decorators (모든 Story 감싸기)
  decorators: [
    // Theme provider
    (Story) => (
      <div style={{ theme: 'light' }}>
        <Story />
      </div>
    ),
    
    // Redux provider 등
    (Story) => (
      <Provider store={store}>
        <Story />
      </Provider>
    )
  ]
};

export default preview;
```

## 프로덕션 레벨 설정

### 1. CSS 모듈 지원

```typescript
// .storybook/main.ts 추가
const config: StorybookConfig = {
  // ...
  webpackFinal: async (config) => {
    config.module?.rules?.push({
      test: /\.module\.css$/,
      use: [
        'style-loader',
        {
          loader: 'css-loader',
          options: { modules: true }
        }
      ]
    });
    return config;
  }
};
```

### 2. TypeScript 지원

```typescript
// tsconfig.json에 추가
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "types": ["@storybook/test"]
  },
  "include": [
    "src",
    ".storybook"
  ]
}
```

### 3. 접근성 테스트 (a11y)

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta = {
  title: 'Components/Button',
  component: Button,
  
  // a11y 테스트 설정
  parameters: {
    a11y: {
      config: {
        rules: [
          {
            id: 'color-contrast',
            enabled: true
          }
        ]
      }
    }
  }
} satisfies Meta<typeof Button>;
```

### 4. 시각적 회귀 테스트

```typescript
import { expect, waitFor } from '@storybook/test';

export const WithVisualTest: Story = {
  args: {
    label: 'Button'
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button');
    
    // 버튼이 렌더링될 때까지 대기
    await waitFor(() => {
      expect(button).toBeInTheDocument();
    });
  }
};
```

## 팀 협업

### Storybook 공유

```bash
# 정적 Storybook 빌드
npm run build-storybook

# dist/storybook 디렉토리 생성 → 웹서버에 배포
```

### 디자이너와의 협업

```typescript
// 컴포넌트와 디자인 시스템 일치시키기
const meta = {
  title: 'Design System/Colors',
  component: ColorPalette,
  
  parameters: {
    figma: 'https://www.figma.com/...',  // Figma 링크
    docs: {
      description: {
        component: '공식 디자인 시스템의 컬러입니다.'
      }
    }
  }
};
```

## Storybook Commands

### 기본 명령어

```bash
# 개발 모드 (hot reload 지원)
npm run storybook

# 정적 빌드 생성
npm run build-storybook

# 원격 서버에서 실행
storybook dev -p 9009

# 프로덕션 모드 빌드
npm run build-storybook -- --configuration-mode=static
```

## 공식 자료

```
공식 사이트: https://storybook.js.org/
문서: https://storybook.js.org/docs/react/get-started/introduction
GitHub: https://github.com/storybookjs/storybook
커뮤니티: https://discord.gg/storybook
```

## 팀 협업 가이드

```markdown
# Storybook 개발 규칙

## 컴포넌트 구조

모든 컴포넌트마다 Story 파일 작성 필수:

```
Button.tsx              → 컴포넌트
Button.stories.tsx      → Story (필수!)
Button.test.tsx         → 테스트 (선택)
Button.module.css       → 스타일
```

## Story 작성 체크리스트

- [ ] 기본 상태
- [ ] 모든 variant 표시
- [ ] 모든 크기 표시
- [ ] 에러/로딩 상태
- [ ] 비활성화 상태
- [ ] 상호작용 테스트

## argTypes 필수 정의

모든 props에 대해:
- description: 무엇인가?
- control: 어떻게 제어할 것인가?
- table: 타입과 기본값

## 성능 최적화

- Storybook 빌드 시간을 5초 이하로 유지
- 큰 컴포넌트는 분리하여 lazy load
- 불필요한 addons 제거

## 배포

- CI/CD에서 자동 빌드
- 프리뷰 URL 공유
- 모든 PR에 Storybook 링크 포함

## 검토 프로세스

1. 컴포넌트 구현
2. Story 파일 작성
3. PR에 Story 포함
4. 팀원 리뷰 (Storybook 확인)
5. 머지 전 Story 검증

## 문제 해결

**Q: Story가 로드되지 않음**
A: .storybook/main.ts의 stories 패턴 확인

**Q: 이미지가 안 보여요**
A: public 폴더에 이미지 저장하고 /image.png로 참조

**Q: Props가 제어되지 않음**
A: argTypes와 args를 모두 정의했는지 확인

**Q: 전역 스타일이 적용 안 됨**
A: .storybook/preview.ts에 import 추가
```

## Storybook vs 다른 도구

| 기능 | Storybook | Styleguidist | Ladle | Bit |
|------|-----------|-------------|-------|-----|
| **컴포넌트 개발** | ✅ 최고 | ✅ 좋음 | ✅ 좋음 | ✅ 좋음 |
| **문서화** | ✅ 최고 | ✅ 좋음 | ❌ 약함 | ✅ 좋음 |
| **테스트** | ✅ 최고 | ❌ 약함 | ❌ 약함 | ❌ 약함 |
| **커뮤니티** | ✅ 최대 | ✅ 중간 | ❌ 작음 | ✅ 중간 |
| **학습곡선** | 중간 | 낮음 | 낮음 | 높음 |
| **성능** | 중간 | 빠름 | 매우 빠름 | 중간 |

## 실제 사용 사례

### 1. 디자인 시스템 문서화

```typescript
// 회사의 디자인 시스템을 Storybook으로 문서화
// Button, Card, Input, Modal 등 모든 컴포넌트
// → 디자이너와 개발자 간 일치 보장
// → 신규 입사자 온보딩 도구로 사용
```

### 2. 컴포넌트 라이브러리

```typescript
// 오픈소스 UI 라이브러리
// Chakra UI, MUI, shadcn/ui 모두 Storybook 사용
// → 사용자가 모든 컴포넌트를 직접 테스트
// → 문서화와 테스트를 한 번에
```

### 3. 팀 협업

```typescript
// PR에 Storybook 링크 포함
// → 코드 리뷰 전에 시각적으로 확인
// → UI/UX 팀이 변경 사항 검증
// → 커뮤니케이션 비용 75% 감소
```

## 결론

Storybook은:

✅ **개발 속도**: 컴포넌트 개발 시간 50% 단축
✅ **품질**: 모든 상태를 테스트할 수 있음
✅ **문서화**: 자동으로 생성되는 문서
✅ **협업**: 디자이너, QA, 개발자 간 소통 개선
✅ **재사용성**: 컴포넌트의 모든 사용 예제 명시
✅ **유지보수**: 변경 사항 영향 범위 즉시 파악

**특히 다음 경우에 필수입니다:**

- 컴포넌트 라이브러리 개발
- 대규모 팀 협업
- 디자인 시스템 구축
- 복잡한 UI 컴포넌트

**지금 바로 Storybook을 시작하세요!**

## 참고 링크

- [Storybook 공식 사이트](https://storybook.js.org/)
- [Storybook 공식 문서](https://storybook.js.org/docs)
- [Storybook GitHub 저장소](https://github.com/storybookjs/storybook)
- [Intro to Storybook 튜토리얼](https://storybook.js.org/tutorials/intro-to-storybook/)
