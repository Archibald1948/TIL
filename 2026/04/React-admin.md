# React-admin - 관리자 패널의 완벽 솔루션

## 들어가며

관리자 패널을 만들 때 이런 문제들이 있나요?

```
관리자 패널 개발의 어려움:
❌ 데이터 테이블을 직접 구현
❌ 페이지네이션 기능 구현
❌ 필터링과 정렬 기능
❌ 폼 검증과 에러 처리
❌ 권한 관리
❌ 레이아웃과 디자인
❌ 반복적인 CRUD 작업

시간이 너무 오래 걸립니다!
```

**React-admin**은 이 모든 것을 포함한 완벽한 관리자 패널 라이브러리입니다. 몇 줄의 코드로 전문가 수준의 관리자 패널을 만들 수 있습니다.

## 공식 사이트

https://marmelab.com/react-admin

# 1. React-admin이란?

## 핵심 개념

```
React-admin = React 기반의 관리자 패널 프레임워크

특징:
✅ 자동 CRUD 인터페이스
✅ 데이터 테이블 (페이지네이션, 정렬, 필터)
✅ 폼 (유효성 검사, 에러 처리)
✅ 권한 관리 (Authentication/Authorization)
✅ 다국어 지원 (i18n)
✅ 테마 커스터마이징
✅ REST/GraphQL API 지원
✅ 반응형 디자인
```

## React-admin의 장점

```
✅ 빠른 개발 (몇 줄로 완성)
✅ 전문가 수준의 UI/UX
✅ 커스터마이징 가능
✅ 큰 커뮤니티
✅ 많은 플러그인과 확장
✅ 프로덕션 레벨
```

# 2. 설치

## 패키지 설치

```bash
# 기본 설치
npm install react-admin ra-core ra-ui-materialui

# Material-UI 의존성
npm install @mui/material @emotion/react @emotion/styled

# 데이터 제공자 (REST API)
npm install ra-data-simple-rest

# 또는 다른 데이터 제공자
npm install ra-data-json-server
npm install ra-data-graphql-simple
```

# 3. 기본 설정

## 가장 간단한 관리자 패널

```javascript
import React from 'react';
import {
  Admin,
  Resource,
  ListGuesser,
  EditGuesser,
  CreateGuesser,
} from 'react-admin';
import { DataProvider } from 'ra-data-simple-rest';

const dataProvider = new DataProvider('https://jsonplaceholder.typicode.com');

function App() {
  return (
    <Admin dataProvider={dataProvider}>
      {/* 사용자 리소스 */}
      <Resource
        name="users"
        list={ListGuesser}
        edit={EditGuesser}
        create={CreateGuesser}
      />

      {/* 포스트 리소스 */}
      <Resource
        name="posts"
        list={ListGuesser}
        edit={EditGuesser}
        create={CreateGuesser}
      />

      {/* 댓글 리소스 */}
      <Resource
        name="comments"
        list={ListGuesser}
        edit={EditGuesser}
        create={CreateGuesser}
      />
    </Admin>
  );
}

export default App;
```

이것만으로도 완전한 관리자 패널이 만들어집니다!

# 4. 리스트 뷰 (List View)

## 기본 리스트

```javascript
import {
  List,
  Datagrid,
  TextField,
  EmailField,
  DateField,
  BooleanField,
  NumberField,
  EditButton,
  DeleteButton,
  ShowButton,
} from 'react-admin';

export const UserList = () => (
  <List>
    <Datagrid rowClick="edit">
      <TextField source="id" />
      <TextField source="name" />
      <EmailField source="email" />
      <TextField source="phone" />
      <BooleanField source="active" />
      <DateField source="createdAt" />
      <EditButton />
      <DeleteButton />
    </Datagrid>
  </List>
);
```

## 필터링 추가

```javascript
import {
  List,
  Datagrid,
  TextField,
  Filter,
  TextInput,
  BooleanInput,
} from 'react-admin';

const UserFilter = (props) => (
  <Filter {...props}>
    <TextInput label="검색" source="q" alwaysOn />
    <TextInput source="name" />
    <TextInput source="email" />
    <BooleanInput source="active" />
  </Filter>
);

export const UserList = () => (
  <List filters={<UserFilter />}>
    <Datagrid rowClick="edit">
      <TextField source="id" />
      <TextField source="name" />
      <TextField source="email" />
      <BooleanField source="active" />
    </Datagrid>
  </List>
);
```

## 페이지네이션과 정렬

```javascript
import { List, Datagrid, TextField, Pagination } from 'react-admin';

export const UserList = () => (
  <List
    pagination={<Pagination rowsPerPageOptions={[10, 25, 50, 100]} />}
    sort={{ field: 'name', order: 'ASC' }}
  >
    <Datagrid>
      <TextField source="id" sortable />
      <TextField source="name" sortable />
      <TextField source="email" sortable />
    </Datagrid>
  </List>
);
```

# 5. 에딧 뷰 (Edit View)

## 기본 에딧 폼

```javascript
import {
  Edit,
  SimpleForm,
  TextInput,
  EmailInput,
  NumberInput,
  BooleanInput,
  DateInput,
  SelectInput,
  SaveButton,
  DeleteButton,
} from 'react-admin';

export const UserEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput disabled source="id" />
      <TextInput source="name" required />
      <EmailInput source="email" required />
      <TextInput source="phone" />
      <NumberInput source="age" />
      <SelectInput
        source="role"
        choices={[
          { id: 'user', name: 'User' },
          { id: 'admin', name: 'Admin' },
        ]}
      />
      <BooleanInput source="active" />
      <DateInput source="createdAt" disabled />
      <SaveButton />
      <DeleteButton />
    </SimpleForm>
  </Edit>
);
```

## 유효성 검사

```javascript
import {
  Edit,
  SimpleForm,
  TextInput,
  required,
  minLength,
  maxLength,
  email,
  number,
  regex,
  minValue,
  maxValue,
} from 'react-admin';

export const UserEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput
        source="name"
        validate={[required(), minLength(3), maxLength(50)]}
      />

      <TextInput source="email" validate={[required(), email()]} />

      <TextInput
        source="phone"
        validate={[regex(/^\d{3}-\d{4}-\d{4}$/, '형식: 010-1234-5678')]}
      />

      <NumberInput
        source="age"
        validate={[number(), minValue(0), maxValue(120)]}
      />

      {/* 커스텀 검증 */}
      <TextInput
        source="username"
        validate={[
          required(),
          (value) => {
            if (value && value.length < 3) {
              return '최소 3자 이상';
            }
            return undefined;
          },
        ]}
      />
    </SimpleForm>
  </Edit>
);
```

## 조건부 필드

```javascript
import { Edit, SimpleForm, TextInput, BooleanInput } from 'react-admin';

export const UserEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="name" required />
      <BooleanInput source="isCompany" />

      {/* FormDataConsumer를 사용해 다른 필드 값 접근 */}
      <FormDataConsumer>
        {({ formData, ...rest }) =>
          formData.isCompany && (
            <TextInput
              source="companyName"
              required
              validate={required()}
              {...rest}
            />
          )
        }
      </FormDataConsumer>
    </SimpleForm>
  </Edit>
);
```

# 6. 생성 뷰 (Create View)

```javascript
import {
  Create,
  SimpleForm,
  TextInput,
  EmailInput,
  PasswordInput,
  SelectInput,
  required,
  email,
} from 'react-admin';

export const UserCreate = () => (
  <Create>
    <SimpleForm>
      <TextInput source="name" label="이름" validate={required()} />

      <EmailInput
        source="email"
        label="이메일"
        validate={[required(), email()]}
      />

      <PasswordInput source="password" label="비밀번호" validate={required()} />

      <SelectInput
        source="role"
        label="역할"
        choices={[
          { id: 'user', name: 'User' },
          { id: 'admin', name: 'Admin' },
        ]}
        defaultValue="user"
      />
    </SimpleForm>
  </Create>
);
```

# 7. 관계 처리

## 일대다 관계

```javascript
import {
  List,
  Datagrid,
  TextField,
  ReferenceField,
  EditButton,
} from 'react-admin';

// 포스트 리스트 (사용자 참조)
export const PostList = () => (
  <List>
    <Datagrid rowClick="edit">
      <TextField source="id" />
      <TextField source="title" />

      {/* 사용자 ID를 통해 사용자 이름 표시 */}
      <ReferenceField source="userId" reference="users">
        <TextField source="name" />
      </ReferenceField>

      <EditButton />
    </Datagrid>
  </List>
);

// 포스트 에딧 (사용자 선택)
import {
  Edit,
  SimpleForm,
  TextInput,
  ReferenceInput,
  SelectInput,
} from 'react-admin';

export const PostEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="title" required />
      <TextInput source="body" multiline />

      {/* 사용자 선택 */}
      <ReferenceInput source="userId" reference="users">
        <SelectInput optionText="name" />
      </ReferenceInput>
    </SimpleForm>
  </Edit>
);
```

## 다대일 관계 (ArrayInput)

```javascript
import {
  Edit,
  SimpleForm,
  TextInput,
  ArrayInput,
  SimpleFormIterator,
} from 'react-admin';

export const UserEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="name" required />

      {/* 여러 주소 */}
      <ArrayInput source="addresses">
        <SimpleFormIterator>
          <TextInput source="street" />
          <TextInput source="city" />
          <TextInput source="zipCode" />
        </SimpleFormIterator>
      </ArrayInput>
    </SimpleForm>
  </Edit>
);
```

# 8. 권한 관리 (Authorization)

## 권한 확인

```javascript
import { usePermissions } from 'react-admin';

function Dashboard() {
  const { permissions } = usePermissions();

  return <div>{permissions === 'admin' && <div>Admin only content</div>}</div>;
}
```

## 리소스별 권한

```javascript
import { Admin, Resource } from 'react-admin'

function App() {
  const authProvider = {
    login: (credentials) => { /* ... */ },
    logout: () => { /* ... */ },
    checkAuth: () => { /* ... */ },
    checkError: (error) => { /* ... */ },
    getPermissions: () => Promise.resolve(['posts:read', 'users:write'])
  }

  return (
    <Admin authProvider={authProvider} dataProvider={dataProvider}>
      {/* 모두에게 표시 */}
      <Resource name="posts" list={PostList} />

      {/* admin만 표시 */}
      <Resource
        name="users"
        list={UserList}
        edit={UserEdit}
        create={UserCreate}
      />

      {/* 권한 확인 후 표시 */}
      <Resource
        name="settings"
        list={usePermissions() => usePermissions().permissions === 'admin' ? SettingsList : null}
      />
    </Admin>
  )
}
```

# 9. 커스텀 뷰

## 대시보드

```javascript
import { Card, CardHeader, CardContent } from '@mui/material';
import { useDataProvider, useNotify } from 'react-admin';
import { useEffect, useState } from 'react';

function Dashboard() {
  const dataProvider = useDataProvider();
  const notify = useNotify();
  const [stats, setStats] = useState({});

  useEffect(() => {
    // 통계 데이터 로드
    Promise.all([
      dataProvider.getList('users', { pagination: { page: 1, perPage: 1 } }),
      dataProvider.getList('posts', { pagination: { page: 1, perPage: 1 } }),
      dataProvider.getList('comments', { pagination: { page: 1, perPage: 1 } }),
    ])
      .then(([users, posts, comments]) => {
        setStats({
          users: users.total,
          posts: posts.total,
          comments: comments.total,
        });
      })
      .catch(() => notify('통계 로드 실패', { type: 'error' }));
  }, [dataProvider, notify]);

  return (
    <div>
      <Card>
        <CardHeader title="대시보드" />
        <CardContent>
          <p>사용자: {stats.users}</p>
          <p>포스트: {stats.posts}</p>
          <p>댓글: {stats.comments}</p>
        </CardContent>
      </Card>
    </div>
  );
}

export default Dashboard;
```

## 커스텀 페이지

```javascript
import { Admin, Resource, CustomRoutes } from 'react-admin';
import { Route } from 'react-router-dom';

function App() {
  return (
    <Admin dataProvider={dataProvider}>
      <Resource name="users" list={UserList} />
      <CustomRoutes>
        <Route path="/reports" element={<ReportsPage />} />
        <Route path="/settings" element={<SettingsPage />} />
      </CustomRoutes>
    </Admin>
  );
}
```

# 10. 데이터 제공자 (Data Provider)

## 커스텀 데이터 제공자

```javascript
import { DataProvider } from 'ra-core'

const customDataProvider = {
  getList: (resource, params) => {
    // API 호출
    return fetch(`/api/${resource}?...`)
      .then(res => res.json())
      .then(data => ({
        data: data.items,
        total: data.total
      }))
  },

  getOne: (resource, params) => {
    return fetch(`/api/${resource}/${params.id}`)
      .then(res => res.json())
      .then(data => ({
        data: data
      }))
  },

  create: (resource, params) => {
    return fetch(`/api/${resource}`, {
      method: 'POST',
      body: JSON.stringify(params.data)
    })
      .then(res => res.json())
      .then(data => ({
        data: data
      }))
  },

  update: (resource, params) => {
    return fetch(`/api/${resource}/${params.id}`, {
      method: 'PUT',
      body: JSON.stringify(params.data)
    })
      .then(res => res.json())
      .then(data => ({
        data: data
      }))
  },

  delete: (resource, params) => {
    return fetch(`/api/${resource}/${params.id}`, {
      method: 'DELETE'
    })
      .then(() => ({
        data: {}
      }))
  },

  deleteMany: (resource, params) => {
    return Promise.all(
      params.ids.map(id =>
        fetch(`/api/${resource}/${id}`, { method: 'DELETE' })
      )
    )
      .then(() => ({
        data: params.ids
      }))
  },

  getMany: (resource, params) => {
    return fetch(`/api/${resource}?ids=${params.ids.join(',')}`)
      .then(res => res.json())
      .then(data => ({
        data: data.items
      }))
  },

  getManyReference: (resource, params) => {
    return fetch(`/api/${resource}?${params.target}=${params.id}`)
      .then(res => res.json())
      .then(data => ({
        data: data.items,
        total: data.total
      }))
  }
}

// 사용
<Admin dataProvider={customDataProvider} />
```

# 11. 테마 커스터마이징

```javascript
import { Admin } from 'react-admin';
import { createTheme } from '@mui/material/styles';

const customTheme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
    background: {
      default: '#f5f5f5',
    },
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
  },
});

function App() {
  return (
    <Admin dataProvider={dataProvider} theme={customTheme}>
      {/* 리소스들 */}
    </Admin>
  );
}
```

# 12. 다국어 지원 (i18n)

```javascript
import { Admin } from 'react-admin';
import polyglotI18nProvider from 'ra-i18n-polyglot';
import englishMessages from 'ra-language-english';
import koreanMessages from 'ra-language-korean';

const messages = {
  ko: koreanMessages,
  en: englishMessages,
};

const i18nProvider = polyglotI18nProvider((locale) => messages[locale], 'ko');

function App() {
  return (
    <Admin dataProvider={dataProvider} i18nProvider={i18nProvider}>
      {/* 리소스들 */}
    </Admin>
  );
}
```

# 13. 체크리스트

React-admin 프로젝트 시작하기:

```
[ ] react-admin 설치
[ ] 데이터 제공자 설정
[ ] 기본 리소스 정의
[ ] 리스트 뷰 커스터마이징
[ ] 에딧/생성 뷰 작성
[ ] 유효성 검사 추가
[ ] 권한 관리 구현
[ ] 대시보드 생성
[ ] 테마 커스터마이징
[ ] 다국어 지원 추가
```

# 결론

React-admin은:

✅ 빠른 관리자 패널 구축
✅ 전문가 수준의 UI/UX
✅ 풍부한 기능
✅ 높은 커스터마이징성
✅ 프로덕션 준비 완료

**관리자 패널은 React-admin으로 만드세요!**

## 참고 링크

- [React-admin 공식 사이트](https://marmelab.com/react-admin/)
- [공식 문서(Documentation)](https://marmelab.com/react-admin/documentation.html)
- [React-admin GitHub](https://github.com/marmelab/react-admin)
