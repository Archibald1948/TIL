# React Hook Form - 가벼운 폼 관리의 완벽 솔루션

## 들어가며

React에서 폼을 직접 관리할 때 얼마나 번거로운가요?

```typescript
// ❌ 수동 폼 관리 (너무 많은 코드)
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');
const [emailError, setEmailError] = useState('');
const [passwordError, setPasswordError] = useState('');

const handleEmailChange = (e) => {
  setEmail(e.target.value);
  // 검증...
};

const handleSubmit = (e) => {
  e.preventDefault();
  // 유효성 검사...
};
```

**React Hook Form**은 이 모든 번거로움을 제거합니다. 단 몇 줄로 완벽한 폼을 만들 수 있습니다.

## 공식 사이트

https://react-hook-form.com

# 1. React Hook Form이란?

## 핵심 특징

```
✅ 매우 가벼움 (9KB)
✅ 성능 최고 (불필요한 리렌더링 없음)
✅ 배우기 쉬움 (간단한 API)
✅ TypeScript 지원 우수
✅ 의존성 최소 (필수 라이브러리 거의 없음)
✅ 거대한 커뮤니티
```

## React Hook Form vs Redux Form vs Formik

```
React Hook Form:
- 크기: 9KB (가장 작음)
- 성능: 최고
- 배우기: 쉬움
- 번들: 최소

Formik:
- 크기: 40KB
- 성능: 좋음
- 배우기: 중간
- 번들: 중간

Redux Form:
- 크기: 큼
- 성능: 낮음
- 배우기: 어려움
- 번들: 큼
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install react-hook-form
```

매우 간단합니다. 의존성도 최소입니다.

# 3. 기본 사용법

## 가장 간단한 폼

```typescript
import { useForm } from 'react-hook-form'

function SimpleForm() {
  // useForm 훅 사용
  const { register, handleSubmit, formState: { errors } } = useForm()

  // 폼 제출 핸들러
  const onSubmit = (data) => {
    console.log(data)
    // { email: 'test@example.com', password: '123456' }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* register로 입력 필드 등록 */}
      <input
        {...register('email')}
        placeholder="Email"
      />

      {/* 에러 표시 */}
      {errors.email && <p style={{ color: 'red' }}>Email is required</p>}

      <input
        {...register('password')}
        type="password"
        placeholder="Password"
      />
      {errors.password && <p style={{ color: 'red' }}>Password is required</p>}

      <button type="submit">Login</button>
    </form>
  )
}
```

## register의 역할

```typescript
{...register('email')}
// 이것이 자동으로 해주는 것:
// - 상태(state) 관리
// - onChange 핸들러
// - 폼 제출 시 값 수집
```

# 4. 검증 (Validation)

## 기본 검증

```typescript
import { useForm } from 'react-hook-form'

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm()

  const onSubmit = (data) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* 필수 필드 검증 */}
      <input
        {...register('email', {
          required: 'Email is required'
        })}
        placeholder="Email"
      />
      {errors.email && <p>{errors.email.message}</p>}

      {/* 이메일 형식 검증 */}
      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email address'
          }
        })}
        placeholder="Email"
      />
      {errors.email && <p>{errors.email.message}</p>}

      {/* 최소 길이 검증 */}
      <input
        {...register('password', {
          required: 'Password is required',
          minLength: {
            value: 8,
            message: 'Password must be at least 8 characters'
          }
        })}
        type="password"
        placeholder="Password"
      />
      {errors.password && <p>{errors.password.message}</p>}

      {/* 최대 길이 검증 */}
      <input
        {...register('username', {
          required: 'Username is required',
          maxLength: {
            value: 20,
            message: 'Username must be less than 20 characters'
          }
        })}
        placeholder="Username"
      />
      {errors.username && <p>{errors.username.message}</p>}

      {/* 숫자 범위 검증 */}
      <input
        {...register('age', {
          required: 'Age is required',
          min: {
            value: 18,
            message: 'Must be 18 or older'
          },
          max: {
            value: 120,
            message: 'Age must be realistic'
          }
        })}
        type="number"
        placeholder="Age"
      />
      {errors.age && <p>{errors.age.message}</p>}

      <button type="submit">Submit</button>
    </form>
  )
}
```

## 커스텀 검증 (validate)

```typescript
import { useForm } from 'react-hook-form'

function SignupForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors }
  } = useForm()

  // password 필드의 값을 감시
  const password = watch('password')

  const onSubmit = (data) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* 커스텀 검증 - 비밀번호 규칙 */}
      <input
        {...register('password', {
          required: 'Password is required',
          validate: (value) => {
            if (!/[A-Z]/.test(value)) {
              return 'Password must contain uppercase letter'
            }
            if (!/[0-9]/.test(value)) {
              return 'Password must contain number'
            }
            if (!/[!@#$%^&*]/.test(value)) {
              return 'Password must contain special character'
            }
            return true
          }
        })}
        type="password"
        placeholder="Password"
      />
      {errors.password && <p>{errors.password.message}</p>}

      {/* 커스텀 검증 - 비밀번호 확인 */}
      <input
        {...register('confirmPassword', {
          required: 'Confirm password is required',
          validate: (value) => {
            if (value !== password) {
              return 'Passwords do not match'
            }
            return true
          }
        })}
        type="password"
        placeholder="Confirm Password"
      />
      {errors.confirmPassword && <p>{errors.confirmPassword.message}</p>}

      {/* 비동기 검증 - 서버에서 확인 */}
      <input
        {...register('username', {
          required: 'Username is required',
          validate: async (value) => {
            // 서버에 요청
            const response = await fetch(`/api/check-username?username=${value}`)
            const { exists } = await response.json()

            if (exists) {
              return 'Username already taken'
            }
            return true
          }
        })}
        placeholder="Username"
      />
      {errors.username && <p>{errors.username.message}</p>}

      <button type="submit">Sign Up</button>
    </form>
  )
}
```

# 5. TypeScript와 함께 사용

## 타입 안전 폼

```typescript
import { useForm } from 'react-hook-form'

// 폼 데이터 타입 정의
interface LoginFormData {
  email: string
  password: string
  rememberMe: boolean
}

function TypeSafeForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm<LoginFormData>({
    defaultValues: {
      email: '',
      password: '',
      rememberMe: false
    }
  })

  // onSubmit의 data 매개변수가 자동으로 LoginFormData 타입
  const onSubmit = (data: LoginFormData) => {
    console.log(data.email) // ✅ TypeScript에서 자동완성
    // data.invalid // ❌ 에러 - 이 필드는 없음
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email', { required: true })}
        placeholder="Email"
      />
      {errors.email && <p>Email is required</p>}

      <input
        {...register('password', { required: true })}
        type="password"
        placeholder="Password"
      />
      {errors.password && <p>Password is required</p>}

      <label>
        <input {...register('rememberMe')} type="checkbox" />
        Remember me
      </label>

      <button type="submit">Login</button>
    </form>
  )
}
```

# 6. 동적 필드 (배열)

## useFieldArray 사용

```typescript
import { useForm, useFieldArray } from 'react-hook-form'

interface ContactFormData {
  contacts: Array<{
    name: string
    phone: string
  }>
}

function DynamicForm() {
  const {
    register,
    control,
    handleSubmit,
    formState: { errors }
  } = useForm<ContactFormData>({
    defaultValues: {
      // 초기 연락처 1개
      contacts: [{ name: '', phone: '' }]
    }
  })

  // 동적 필드 배열 관리
  const { fields, append, remove } = useFieldArray({
    control,
    name: 'contacts'
  })

  const onSubmit = (data: ContactFormData) => {
    console.log(data)
    // { contacts: [{ name: 'John', phone: '010-1234-5678' }, ...] }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* 각 연락처 필드 */}
      {fields.map((field, index) => (
        <div key={field.id}>
          <input
            {...register(`contacts.${index}.name`, {
              required: 'Name is required'
            })}
            placeholder="Name"
          />
          {errors.contacts?.[index]?.name && (
            <p>{errors.contacts[index]?.name?.message}</p>
          )}

          <input
            {...register(`contacts.${index}.phone`, {
              required: 'Phone is required'
            })}
            placeholder="Phone"
          />
          {errors.contacts?.[index]?.phone && (
            <p>{errors.contacts[index]?.phone?.message}</p>
          )}

          {/* 제거 버튼 */}
          <button
            type="button"
            onClick={() => remove(index)}
          >
            Remove
          </button>
        </div>
      ))}

      {/* 추가 버튼 */}
      <button
        type="button"
        onClick={() => append({ name: '', phone: '' })}
      >
        Add Contact
      </button>

      <button type="submit">Submit</button>
    </form>
  )
}
```

# 7. watch, getValues, setValue

## watch - 필드 감시

```typescript
import { useForm } from 'react-hook-form'

function WatchExample() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors }
  } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      age: 0
    }
  })

  // 특정 필드 감시
  const firstName = watch('firstName')
  const lastName = watch('lastName')

  // 여러 필드 감시
  const { age, email } = watch(['age', 'email'])

  // 모든 필드 감시 (성능 이슈 가능)
  // const allValues = watch()

  const onSubmit = (data) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('firstName')}
        placeholder="First Name"
      />

      <input
        {...register('lastName')}
        placeholder="Last Name"
      />

      <input
        {...register('age', { valueAsNumber: true })}
        type="number"
        placeholder="Age"
      />

      {/* firstName과 lastName 변경 시 실시간 표시 */}
      {firstName && lastName && (
        <p>Hello, {firstName} {lastName}!</p>
      )}

      {age >= 18 && <p>You are an adult</p>}

      <button type="submit">Submit</button>
    </form>
  )
}
```

## getValues - 현재 값 가져오기

```typescript
import { useForm } from 'react-hook-form'

function GetValuesExample() {
  const {
    register,
    handleSubmit,
    getValues,
    formState: { errors }
  } = useForm()

  // watch와 달리 리렌더링 하지 않음
  const handleCustomValidation = () => {
    const values = getValues() // 모든 값 가져오기
    const email = getValues('email') // 특정 필드 값 가져오기

    console.log(values, email)

    // 커스텀 검증 수행
    if (email.includes('test')) {
      alert('Cannot use test email')
    }
  }

  const onSubmit = (data) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email')}
        placeholder="Email"
      />

      <button
        type="button"
        onClick={handleCustomValidation}
      >
        Validate
      </button>

      <button type="submit">Submit</button>
    </form>
  )
}
```

## setValue - 필드값 설정

```typescript
import { useForm } from 'react-hook-form'

function SetValueExample() {
  const {
    register,
    handleSubmit,
    setValue,
    formState: { errors }
  } = useForm()

  const onSubmit = (data) => {
    console.log(data)
  }

  const loadDefaults = () => {
    // 기본값 자동 설정
    setValue('email', 'default@example.com')
    setValue('name', 'John Doe')
  }

  const clearForm = () => {
    // 폼 초기화
    setValue('email', '')
    setValue('name', '')
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email')}
        placeholder="Email"
      />

      <input
        {...register('name')}
        placeholder="Name"
      />

      <button type="button" onClick={loadDefaults}>
        Load Defaults
      </button>

      <button type="button" onClick={clearForm}>
        Clear
      </button>

      <button type="submit">Submit</button>
    </form>
  )
}
```

# 8. 실전 예제

## 프로필 수정 폼

```typescript
import { useForm } from 'react-hook-form'
import { useState } from 'react'

interface Profile {
  name: string
  bio: string
  website: string
  avatar: FileList
}

function ProfileForm({ initialData }) {
  const [isLoading, setIsLoading] = useState(false)

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset
  } = useForm<Profile>({
    defaultValues: initialData
  })

  const onSubmit = async (data: Profile) => {
    setIsLoading(true)
    try {
      const formData = new FormData()
      formData.append('name', data.name)
      formData.append('bio', data.bio)
      formData.append('website', data.website)
      if (data.avatar?.[0]) {
        formData.append('avatar', data.avatar[0])
      }

      const response = await fetch('/api/profile', {
        method: 'PUT',
        body: formData
      })

      if (response.ok) {
        const result = await response.json()
        reset(result) // 서버 응답으로 폼 업데이트
        alert('Profile updated successfully')
      }
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* 이름 */}
      <div>
        <label>Name</label>
        <input
          {...register('name', {
            required: 'Name is required',
            minLength: { value: 2, message: 'Name must be at least 2 characters' }
          })}
        />
        {errors.name && <span>{errors.name.message}</span>}
      </div>

      {/* 소개 */}
      <div>
        <label>Bio</label>
        <textarea
          {...register('bio', {
            maxLength: { value: 500, message: 'Bio must be less than 500 characters' }
          })}
        />
        {errors.bio && <span>{errors.bio.message}</span>}
      </div>

      {/* 웹사이트 */}
      <div>
        <label>Website</label>
        <input
          {...register('website', {
            pattern: {
              value: /^(https?:\/\/)?(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
              message: 'Invalid URL'
            }
          })}
        />
        {errors.website && <span>{errors.website.message}</span>}
      </div>

      {/* 프로필 사진 */}
      <div>
        <label>Avatar</label>
        <input
          {...register('avatar')}
          type="file"
          accept="image/*"
        />
        {errors.avatar && <span>{errors.avatar.message}</span>}
      </div>

      <button
        type="submit"
        disabled={isSubmitting || isLoading}
      >
        {isLoading ? 'Saving...' : 'Save Profile'}
      </button>
    </form>
  )
}
```

## 다단계 폼 (Wizard)

```typescript
import { useForm } from 'react-hook-form'
import { useState } from 'react'

function MultiStepForm() {
  const [step, setStep] = useState(1)

  const {
    register,
    handleSubmit,
    formState: { errors },
    trigger
  } = useForm({
    mode: 'onBlur',
    defaultValues: {
      personalInfo: { name: '', email: '' },
      address: { street: '', city: '', zip: '' },
      payment: { cardNumber: '', expiry: '' }
    }
  })

  const onSubmit = (data) => {
    if (step < 3) {
      setStep(step + 1)
    } else {
      console.log('Final submission:', data)
    }
  }

  const goBack = () => {
    if (step > 1) setStep(step - 1)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Step 1: 개인 정보 */}
      {step === 1 && (
        <div>
          <h2>Personal Information</h2>

          <input
            {...register('personalInfo.name', { required: 'Name is required' })}
            placeholder="Name"
          />
          {errors.personalInfo?.name && (
            <p>{errors.personalInfo.name.message}</p>
          )}

          <input
            {...register('personalInfo.email', {
              required: 'Email is required',
              pattern: {
                value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
                message: 'Invalid email'
              }
            })}
            placeholder="Email"
          />
          {errors.personalInfo?.email && (
            <p>{errors.personalInfo.email.message}</p>
          )}
        </div>
      )}

      {/* Step 2: 주소 */}
      {step === 2 && (
        <div>
          <h2>Address</h2>

          <input
            {...register('address.street', { required: 'Street is required' })}
            placeholder="Street"
          />
          {errors.address?.street && (
            <p>{errors.address.street.message}</p>
          )}

          <input
            {...register('address.city', { required: 'City is required' })}
            placeholder="City"
          />
          {errors.address?.city && (
            <p>{errors.address.city.message}</p>
          )}

          <input
            {...register('address.zip', { required: 'ZIP is required' })}
            placeholder="ZIP Code"
          />
          {errors.address?.zip && (
            <p>{errors.address.zip.message}</p>
          )}
        </div>
      )}

      {/* Step 3: 결제 */}
      {step === 3 && (
        <div>
          <h2>Payment</h2>

          <input
            {...register('payment.cardNumber', { required: 'Card number is required' })}
            placeholder="Card Number"
          />
          {errors.payment?.cardNumber && (
            <p>{errors.payment.cardNumber.message}</p>
          )}

          <input
            {...register('payment.expiry', { required: 'Expiry is required' })}
            placeholder="MM/YY"
          />
          {errors.payment?.expiry && (
            <p>{errors.payment.expiry.message}</p>
          )}
        </div>
      )}

      {/* 버튼 */}
      <div>
        {step > 1 && (
          <button type="button" onClick={goBack}>
            Back
          </button>
        )}

        <button type="submit">
          {step < 3 ? 'Next' : 'Complete'}
        </button>
      </div>

      <p>Step {step} of 3</p>
    </form>
  )
}
```

# 9. 성능 최적화

## 불필요한 리렌더링 방지

```typescript
import { useForm } from 'react-hook-form'

function PerformanceOptimized() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors }
  } = useForm()

  // ✅ 좋음: 필요한 필드만 감시 (리렌더링 최소화)
  const email = watch('email')

  // ❌ 피할 것: 모든 필드 감시 (불필요한 리렌더링)
  // const allValues = watch()

  // ❌ 피할 것: onChange에서 setState
  // const [email, setEmail] = useState('')

  const onSubmit = (data) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} placeholder="Email" />
      <input {...register('password')} type="password" placeholder="Password" />

      {/* email 변경 시만 리렌더링 */}
      {email && <p>Email: {email}</p>}

      <button type="submit">Submit</button>
    </form>
  )
}
```

# 10. 팀 협업 Best Practices

## 재사용 가능한 폼 훅

```typescript
// hooks/useLoginForm.ts
import { useForm } from 'react-hook-form'

interface LoginData {
  email: string
  password: string
}

export function useLoginForm() {
  return useForm<LoginData>({
    mode: 'onBlur',
    defaultValues: {
      email: '',
      password: ''
    }
  })
}

// components/LoginForm.tsx
import { useLoginForm } from '@/hooks/useLoginForm'

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useLoginForm()

  const onSubmit = (data) => {
    console.log(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: true })} />
      {errors.email && <p>Email is required</p>}

      <input {...register('password', { required: true })} type="password" />
      {errors.password && <p>Password is required</p>}

      <button type="submit">Login</button>
    </form>
  )
}
```

# 11. 자주 묻는 질문

## Q: watch vs getValues?

**A:**

```typescript
// watch - 리렌더링 포함 (UI 업데이트 필요할 때)
const email = watch('email'); // 값 변경 시 리렌더링

// getValues - 리렌더링 없음 (현재 값만 필요할 때)
const email = getValues('email'); // 리렌더링 없음
```

## Q: 서버 에러를 폼에 반영하려면?

**A:**

```typescript
import { useForm } from 'react-hook-form'

function Form() {
  const { register, handleSubmit, setError, formState: { errors } } = useForm()

  const onSubmit = async (data) => {
    try {
      await api.submit(data)
    } catch (error) {
      // 서버 에러를 폼 필드에 설정
      setError('email', {
        type: 'manual',
        message: error.response.data.message
      })
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <p>{errors.email.message}</p>}
      <button type="submit">Submit</button>
    </form>
  )
}
```

## Q: 전체 폼을 초기화하려면?

**A:**

```typescript
import { useForm } from 'react-hook-form'

function Form() {
  const { register, handleSubmit, reset } = useForm({
    defaultValues: {
      email: '',
      password: ''
    }
  })

  const onSubmit = (data) => {
    console.log(data)
    reset() // 기본값으로 초기화
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      <input {...register('password')} type="password" />
      <button type="submit">Submit</button>
    </form>
  )
}
```

# 12. 체크리스트

React Hook Form 시작하기:

```
[ ] react-hook-form 설치
[ ] useForm 훅 기본 사용
[ ] register로 필드 등록
[ ] 검증 규칙 추가
[ ] watch로 필드 감시
[ ] TypeScript 타입 정의
[ ] useFieldArray로 동적 필드 구현
[ ] 성능 최적화 적용
[ ] 팀 협업 규칙 정의
```

# 결론

React Hook Form은:

✅ 매우 가볍고 빠름
✅ 배우기 쉬운 API
✅ TypeScript 완벽 지원
✅ 불필요한 리렌더링 없음
✅ 거대한 커뮤니티

**React에서 폼 관리는 React Hook Form으로 시작하세요!**

## 참고 링크

- [React Hook Form 공식 사이트](https://react-hook-form.com)
- [React Hook Form 공식 문서](https://react-hook-form.com/get-started)
- [React Hook Form GitHub](https://github.com/react-hook-form/react-hook-form)
