# Neon - 서버리스 PostgreSQL의 미래

## 들어가며

전통적인 PostgreSQL 서버를 관리하는 것은 힘듭니다.
데이터베이스 인스턴스를 항상 실행 중으로 유지해야 하고, 트래픽에 따른 수동 스케일링을 해야 하며, 백업과 보안을 신경 써야 합니다.

Neon은 이 모든 문제를 완전히 해결합니다.
서버리스 PostgreSQL 플랫폼으로, 자동 스케일링, 즉각적인 분기(branching), 관리형 호스팅을 제공합니다.

AWS Lambda와 유사하게 사용한 만큼만 비용을 지불하면서도 완전한 PostgreSQL 기능을 사용할 수 있습니다.

## 공식 사이트

https://github.com/neondatabase/neon

## 전통적 PostgreSQL vs Neon

### 아키텍처 비교

```
전통적 PostgreSQL:
┌─────────────────────────────────────┐
│  AWS RDS / Self-hosted Server       │
│  ├─ 고정 인스턴스 (t3.medium 등)    │
│  ├─ 항상 실행 중 (비용 지속 발생)   │
│  ├─ 수동 스케일링 필요              │
│  └─ 관리자 책임 큼                  │
└─────────────────────────────────────┘

Neon:
┌─────────────────────────────────────┐
│  Neon Serverless PostgreSQL         │
│  ├─ 자동 스케일 Compute Units       │
│  ├─ 미사용 시 자동 중지 (비용 0)    │
│  ├─ 자동 스케일링                   │
│  └─ 완전 관리형                     │
└─────────────────────────────────────┘
```

### 비용 비교 (월간 예상 비용)

```
일반적인 중소 프로젝트 (2개 환경):

전통적 PostgreSQL (AWS RDS):
├─ db.t3.small x 2: $40 (항상 실행)
├─ 스토리지 (100GB): $25
├─ 백업: $15
└─ 총계: 약 $80/월 (미사용 시에도 지불)

Neon:
├─ Compute (on-demand): $10-20
├─ 스토리지 (100GB): $0 (무료)
├─ 분기 (5개): $0 (무료)
└─ 총계: 약 $10-20/월 (미사용 시 비용 거의 0)

절감: 70-80%
```

## Neon의 핵심 기능

### 1. 자동 스케일링 (Auto-scaling)

#### Compute Units (CU)

Neon의 비용 모델은 Compute Units 기반입니다.

```
1 CU = 1 vCPU + 4GB RAM

프로젝트의 부하에 따라 자동으로 스케일:
- 저트래픽: 0.5 CU (유휴 상태에서 중지됨)
- 중간 트래픽: 1-2 CU
- 고트래픽: 3-4 CU

비용: $0.16 per CU per hour
```

#### 자동 스케일링 설정

```javascript
// Neon API를 통한 Compute 설정
const neonApiUrl = 'https://console.neon.tech/api/v2';

async function configureAutoscaling(projectId) {
  const response = await fetch(
    `${neonApiUrl}/projects/${projectId}/endpoints`,
    {
      method: 'PATCH',
      headers: {
        Authorization: `Bearer ${process.env.NEON_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        // 최소 Compute Units
        autoscaling_limit_min_cu: 0.5,
        // 최대 Compute Units
        autoscaling_limit_max_cu: 4,
        // 자동 중단 시간 (미사용 시)
        suspend_timeout_seconds: 300, // 5분 후 자동 중단
      }),
    },
  );

  return response.json();
}
```

### 2. Branching (분기)

Neon의 가장 혁신적인 기능입니다.
프로덕션 데이터베이스의 즉각적인 스냅샷을 생성할 수 있습니다.

#### 분기의 원리

```
프로덕션 데이터베이스 (스냅샷)
│
├─ dev 분기 (개발자 A)
│   └─ 독립적인 읽기/쓰기 가능
│
├─ staging 분기
│   └─ QA 환경
│
└─ feature/user-auth 분기
    └─ 새로운 기능 개발
```

**특징:**

- 스냅샷이 즉각적입니다 (몇 밀리초)
- 스토리지 효율적입니다 (Copy-on-Write)
- 프로덕션에 영향을 주지 않습니다
- 비용이 저렴합니다 (추가 스토리지만 청구)

#### 분기 생성

```bash
# Neon CLI 설치
npm install -g @neondatabase/cli

# 분기 생성
neon branches create dev --parent main

# 결과:
# Branch created: dev
# Connection string: postgresql://user:password@dev.us-east-2.neon.tech/neondb
```

#### JavaScript에서 분기 관리

```javascript
const axios = require('axios');

class NeonBranchManager {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://console.neon.tech/api/v2';
  }

  // 분기 생성
  async createBranch(projectId, branchName, parentBranch = 'main') {
    const response = await axios.post(
      `${this.baseUrl}/projects/${projectId}/branches`,
      {
        branch: {
          name: branchName,
          parent_id: parentBranch,
        },
      },
      {
        headers: {
          Authorization: `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json',
        },
      },
    );

    return response.data;
  }

  // 분기 삭제
  async deleteBranch(projectId, branchId) {
    const response = await axios.delete(
      `${this.baseUrl}/projects/${projectId}/branches/${branchId}`,
      {
        headers: {
          Authorization: `Bearer ${this.apiKey}`,
        },
      },
    );

    return response.data;
  }

  // 분기 목록 조회
  async listBranches(projectId) {
    const response = await axios.get(
      `${this.baseUrl}/projects/${projectId}/branches`,
      {
        headers: {
          Authorization: `Bearer ${this.apiKey}`,
        },
      },
    );

    return response.data.branches;
  }

  // 분기 정보 조회
  async getBranch(projectId, branchId) {
    const response = await axios.get(
      `${this.baseUrl}/projects/${projectId}/branches/${branchId}`,
      {
        headers: {
          Authorization: `Bearer ${this.apiKey}`,
        },
      },
    );

    return response.data.branch;
  }
}

// 사용 예
const manager = new NeonBranchManager(process.env.NEON_API_KEY);

(async () => {
  // 새 분기 생성
  const newBranch = await manager.createBranch(
    'my-project-id',
    'feature/user-auth',
    'main',
  );

  console.log('분기 생성됨:', newBranch.name);
  console.log('Connection string:', newBranch.connection_strings.default);

  // 모든 분기 조회
  const branches = await manager.listBranches('my-project-id');
  console.log(
    '전체 분기:',
    branches.map((b) => b.name),
  );
})();
```

### 3. Connection Pooling (연결 풀링)

Neon은 자동으로 PgBouncer를 제공합니다.

```
전통적 설정:
┌─ 애플리케이션 (100개 연결)
├─ 애플리케이션 (100개 연결)
├─ 애플리케이션 (100개 연결)
└─ PostgreSQL (최대 400개)  ← 압박!

Neon (PgBouncer 포함):
┌─ 애플리케이션 (100개 요청)
├─ 애플리케이션 (100개 요청)
├─ 애플리케이션 (100개 요청)
└─ PgBouncer (10개 실제 연결) ← 효율적!
   └─ PostgreSQL
```

```javascript
// Node.js에서 연결 풀링 설정
const { Pool } = require('pg');

const pool = new Pool({
  // Neon의 pooled 엔드포인트 사용
  connectionString: process.env.DATABASE_URL_POOLED,

  // 풀 설정
  max: 20, // 최대 클라이언트 수
  idleTimeoutMillis: 30000, // 유휴 타임아웃
  connectionTimeoutMillis: 2000,
});

// 쿼리 실행
async function getUser(id) {
  const result = await pool.query('SELECT * FROM users WHERE id = $1', [id]);
  return result.rows[0];
}

module.exports = { pool, getUser };
```

### 4. 시점 복구 (Point-in-Time Recovery)

데이터베이스를 특정 시점으로 되돌릴 수 있습니다.

```javascript
// CLI를 통한 PITR
// neon branches create restored-backup --parent main --lsn <lsn_value>

// 또는 API를 통해
const axios = require('axios');

async function restoreFromBackup(projectId, targetLsn) {
  const response = await axios.post(
    `${baseUrl}/projects/${projectId}/branches`,
    {
      branch: {
        name: 'restored-backup',
        parent_id: 'main',
        lsn: targetLsn, // 특정 시점의 LSN
      },
    },
    { headers: { Authorization: `Bearer ${apiKey}` } },
  );

  return response.data;
}
```

## 프로덕션 레벨 설정

### 1. 환경별 데이터베이스 구성

```yaml
# neon.config.yaml

projects:
  my-app:
    # 프로덕션 환경
    production:
      branch: main
      compute_size: large # 4 CU
      autoscale: true
      min_cu: 2
      max_cu: 8
      backup_retention: 30 # 30일
      ssl: true

    # 스테이징 환경
    staging:
      branch: staging
      compute_size: small # 1 CU
      autoscale: true
      min_cu: 0.5
      max_cu: 2
      backup_retention: 7 # 7일
      ssl: true
      parent: main

    # 개발 환경
    development:
      branch: dev
      compute_size: small # 1 CU
      autoscale: true
      min_cu: 0.5
      max_cu: 1
      backup_retention: 3
      ssl: false
      auto_suspend: true
      auto_suspend_delay: 300 # 5분
      parent: main
```

### 2. 마이그레이션 관리

```typescript
// migration.ts - TypeORM 마이그레이션
import { MigrationInterface, QueryRunner } from 'typeorm';

export class CreateUsersTable1234567890 implements MigrationInterface {
  name = 'CreateUsersTable1234567890';

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE TABLE "users" (
        "id" uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        "email" varchar NOT NULL UNIQUE,
        "name" varchar NOT NULL,
        "created_at" TIMESTAMP DEFAULT now(),
        "updated_at" TIMESTAMP DEFAULT now()
      )
    `);

    // 인덱스 생성
    await queryRunner.query(`
      CREATE INDEX "idx_users_email" ON "users" ("email")
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "users"`);
  }
}
```

```bash
# Neon 분기에서 마이그레이션 실행
DATABASE_URL="postgresql://user:pass@dev.us-east-2.neon.tech/neondb" \
npm run typeorm migration:run

# 결과 확인
DATABASE_URL="postgresql://user:pass@dev.us-east-2.neon.tech/neondb" \
npm run typeorm migration:show
```

### 3. 개발 워크플로우

```typescript
// src/database.ts
import { createPool } from '@neon/serverless';

interface DatabaseConfig {
  environment: 'production' | 'staging' | 'development';
  branch?: string;
}

export async function getDatabase(config: DatabaseConfig) {
  const env = process.env.NODE_ENV || 'development';

  // 환경별 연결 문자열 결정
  let connectionString: string;

  switch (config.environment) {
    case 'production':
      connectionString = process.env.DATABASE_URL!;
      break;
    case 'staging':
      connectionString = process.env.DATABASE_URL_STAGING!;
      break;
    case 'development':
      connectionString = process.env.DATABASE_URL_DEV!;
      break;
  }

  // 연결 풀 생성
  const pool = createPool({
    connectionString,
    max: 10,
    idleTimeoutMillis: 30000,
  });

  return pool;
}

// 사용 예
async function createUser(name: string, email: string) {
  const pool = await getDatabase({ environment: 'development' });

  const result = await pool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [name, email],
  );

  return result.rows[0];
}
```

### 4. .env 설정

```bash
# .env.production
DATABASE_URL="postgresql://user:password@ep-prod.us-east-2.neon.tech/production?sslmode=require"
DATABASE_URL_POOLED="postgresql://user:password@ep-prod.us-east-2.neon.tech/production?sslmode=require&pgbouncer=true"

# .env.staging
DATABASE_URL_STAGING="postgresql://user:password@ep-staging.us-east-2.neon.tech/staging?sslmode=require"
DATABASE_URL_STAGING_POOLED="postgresql://user:password@ep-staging.us-east-2.neon.tech/staging?sslmode=require&pgbouncer=true"

# .env.development (로컬 개발)
DATABASE_URL_DEV="postgresql://user:password@ep-dev.us-east-2.neon.tech/development?sslmode=require"
DATABASE_URL_DEV_POOLED="postgresql://user:password@ep-dev.us-east-2.neon.tech/development?sslmode=require&pgbouncer=true"

# Neon API 키
NEON_API_KEY="your-api-key-here"
```

### 5. 스크립트 마이그레이션 정책

```yaml
# migration-policy.yaml

migrations:
  # 버전 관리
  naming: YYYYMMDD_HH_mm_ss_description.sql

  # 예시: 20250129_14_30_00_create_users_table.sql

  # 마이그레이션 체크리스트
  pre_migration:
    - 현재 데이터베이스 백업 확인
    - 스테이징에서 먼저 실행
    - 예상 실행 시간 확인

  post_migration:
    - 성공 여부 확인
    - 롤백 계획 수립
    - 성능 영향 분석

  rollback:
    - 각 마이그레이션에 DOWN 스크립트 필수
    - 2개 버전까지 자동 롤백 가능
    - 긴급 시 복구 계획 보유

environments:
  # 각 환경별 마이그레이션 정책
  production:
    - 영업 시간 외에만 실행
    - DBA 승인 필수
    - 자동 롤백 비활성화
    - 모니터링 강화

  staging:
    - 자유로운 실행
    - 프로덕션 직전에 최종 검증

  development:
    - 완전 자유
    - 언제든 롤백 가능
```

## Neon 기반 개발 팀 협업

### 1. PR별 분기 전략

```bash
#!/bin/bash
# scripts/create-pr-database.sh
# PR이 생성되면 자동으로 데이터베이스 분기 생성

PR_NUMBER=$1
NEON_PROJECT_ID=$2

# Neon에서 분기 생성
branch_name="pr-${PR_NUMBER}"

neon branches create $branch_name --parent main

# GitHub PR에 connection string 추가
echo "Database created: $branch_name"
```

### 2. 개발 워크플로우

```
1. 로컬 개발 시작
   └─ 로컬 dev 분기에서 개발

2. PR 생성
   └─ CI가 자동으로 pr-{number} 분기 생성
   └─ 자동 테스트는 이 분기에서 실행

3. PR 승인
   └─ 스테이징 환경에서 최종 검증

4. 병합
   └─ PR 분기 자동 삭제
   └─ 프로덕션으로 배포

5. 완료
   └─ 모니터링
```

### 3. 팀 협업 규칙 문서화

````markdown
# Neon 데이터베이스 관리 가이드

## 개발자 가이드

### 로컬 설정

```bash
git clone <repo>
npm install

# 환경 변수 설정
cp .env.example .env.development

# 마이그레이션 실행
npm run migrate:dev
```
````

### 개발 분기 사용

- 각 개발자는 자신의 dev 분기 사용
- 프로덕션 환경에 절대 직접 쿼리하지 않기

### 마이그레이션 작성

1. 변경 스크립트 작성 (UP)
2. 롤백 스크립트 작성 (DOWN)
3. 스테이징에서 테스트
4. PR 생성
5. CI에서 자동 검증

### 주의사항

- 프로덕션 데이터 복사 금지 (보안)
- 대용량 스토리지 작업은 사전 공지
- 스토리지 임계값 모니터링

## DBA 가이드

### 프로덕션 배포

- 영업 시간 외에만 실행
- 자동 롤백 비활성화 상태 확인
- 실시간 모니터링

### 성능 최적화

- 쿼리 실행 계획 분석
- 인덱스 최적화
- 연결 풀 모니터링

### 재해 복구

- 일일 백업 확인
- 월 1회 복구 훈련
- 문서화 유지

```

## 비용 최적화

### 1. 비용 계산 예시

```

프로젝트 규모: 중소 스타트업

환경 구성:
├─ Production (main): 2 CU, $0.32/시간
├─ Staging (staging): 1 CU, $0.16/시간
└─ Development (dev): 0.5 CU, $0.08/시간 (자동 중단 설정)

월간 비용:

- Production: 2 _ $0.16 _ 730 = $232/월
- Staging: 1 _ $0.16 _ 730 = $116/월 (업무 시간만 실행)
- Development: 0.5 _ $0.16 _ 100 = $8/월 (대부분 중단)
- 스토리지 (100GB): $0 (무료)

총계: ~$356/월

````

### 2. 비용 절감 방법

```yaml
최적화 전략:

1. 자동 중단 활용
   - 미사용 분기 자동 중단
   - 절감: 20-30%

2. Compute 크기 조정
   - 실제 부하에 맞게 조정
   - 절감: 15-20%

3. 분기 정리
   - 불필요한 분기 삭제
   - 절감: 10-15%

4. 연결 풀링 최적화
   - PgBouncer 설정 조정
   - 리소스 효율화

총 절감 가능: 45-65%
````

## 결론

Neon은 PostgreSQL의 미래입니다:

✅ **비용 효율적**: 전통적 데이터베이스 대비 70-80% 비용 절감
✅ **개발 경험**: 즉각적인 분기로 개발 속도 향상
✅ **확장성**: 자동 스케일링으로 성장에 자동 대응
✅ **안정성**: 자동 백업, PITR, 재해 복구
✅ **관리 부담**: 완전 관리형으로 인프라 걱정 해결

특히 스타트업과 성장 중인 팀에게 Neon은 게임 체인저입니다.
데이터베이스 인프라가 아닌 비즈니스 로직에 집중할 수 있게 해줍니다.

**Neon으로 지금 바로 시작하세요!**

## 참고 링크

- [Neon 공식 사이트](https://neon.com)
- [Neon 공식 문서](https://neon.com/docs)
- [Neon GitHub](https://github.com/neondatabase/neon)
