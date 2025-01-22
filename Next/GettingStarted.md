# 1. 설치(Installation)

## 시스템 요구사항
- 노드 18 이상

## NextJS 시작하기 - 자동설치
cra(create-react-app)와 같은 기능.
```
npx create-next-app@latest
```

```
What is your project named? 프로젝트 명 입력
Would you like to use TypeScript? Yes 타입스크립트 사용
Would you like to use ESLint? Yes ESLint 사용
Would you like to use Tailwind CSS? Yes 서버컴포넌트에서는 Tailwind 만 지원함
Would you like your code inside a `src/` directory? No 주요 코드를 src 폴더 하위에 배치, No 선택 시 모든 코드가 루트에 배치
Would you like to use App Router? (recommended) Yes 14 이후 추가된 앱라우터 적용
Would you like to use Turbopack for `next dev`?  Yes 초고속 웹팩 대체 빌드 도구 (turbopack 은 단일레포용, turborepo는 모노레포용, npx create-turbo@latest 는 모노레포용 명령어)
Would you like to customize the import alias (`@/*` by default)? Yes 단축구문 사용
What import alias would you like configured? @/* @/* 형태로 사용
```

## 수동설치는 고생일 뿐 패스.

## 개발서버 실행
- app/page.tsx 에서 시작함
```
pnpm run dev
```

## 최소 TypeScript 버전:v4.5.2

## 린트 설정은 create-next-app로 해서 패스

## 경로 별칭은 tsconfig.json 에서 설정
```jsx
{
  "compilerOptions": {
    "baseUrl": "src/"
  }
}
```
```jsx
// Before
import { Button } from '../../../components/button'
 
// After
import { Button } from '@/components/button'
```
 ---
 
# 2. 프로젝트 구조(Project Structure)

## 폴더 및 파일 규칙

### 최상위 폴더
 - app : 앱라우터
 - public : 제공될 정적 자산
 - src
<img src="https://github.com/user-attachments/assets/843cd6fc-2305-4a7e-b358-ed979be0e61c" width="400px" />

### 최상위 파일
- 애플리케이션 구성, 종속성 관리, 미들웨어 실행, 모니터링 도구 통합, 환경 변수 정의
- next.config.js Next에 대한 구성 파일
- instrumentation.ts request,response 추적, 사용자데이터 수집, 분석도구 통합 (OpenTelemetry를 사용하면 데이터독같은 도구로 데이터독 로깅 쉽게 가능)
- middleware.ts Next 요청 미들웨어
- .env 환경변수
- .env.local 로컬환경변수
- .env.development 개발환경변수
- .env.production 운영환경변수
- .eslintrc.json Eslintjson ESLinst 구성파일
- next-env.d.ts Next.js에 대한 Typescript 선언 파일

### 라우팅 파일 (뒤에서 자세히 설명)
- layout
- page
- loading
- not-found
- error
- global-error
- route
- template
- default

### 중첩된 경로 (뭔말임?)
- folder 경로구간
- folder/folder 중첩된 경로 세그먼트

### 동적 경로
- [folder] // e.g., { slug: string }
- [...folder] // e.g., 	{ slug: string[] }
- [[...folder]] // e.g., { slug?: string[] }

```tsx
// app/blog/[slug]/page.js
// 동적라우팅에서 동적경로를 params props로 받아올 수 있다는 의미
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const slug = (await params).slug
  return <div>My Post: {slug}</div>
}
```

### 경로 그룹
- @folder alias 된 경로
- (.)folder 같은 depth 경로
- (..)folder 한 계층 위 경로
- (..)(..)folder 두 계층 위 경로
- (...) 루트에서 경로

### 메타데이터 파일 규칙(쓸 때 문서보기)

## 구성 요소 계층
- layout 모든 페이지에 적용되는 공통 레이아웃 설계(헤더, 네비게이션바, 푸터등을 설정)
- page 개별 페이지 정의(layout.tsx 파일에서 page.tsx를 명시적으로 렌더링하는 코드를 작성할 필요가 없음 Next.js가 자동으로 이를 처리)
- error.js 에러바운더리
- loading.js 서스펜스
- not-found.js 에러바운더리
<img src="https://github.com/user-attachments/assets/863f5e99-95ab-4d80-9aa6-9d22a3327f9d" width="400px"/>

## 프로젝트 구성하기
### 공동배치
- pages.js가 존재해야 라우터로 접근 가능

<img src="https://github.com/user-attachments/assets/35677bd5-15e9-4748-9110-03cfe4471f6f" width="400px" />

<img src="https://github.com/user-attachments/assets/279d89f0-e7e2-49cf-b22c-a3176b301c6b" width="280px" />

### 개인폴더
- 개인 구현 세부 사항
- 밑줄그어 만들 수 있음
- 라우팅에서 제외됨.
- e.g., _폴더명

### 경로그룹
- 조직 목적으로 사용됨
- 괄호로 묶어서 만들 수 있음
- 라우팅에서 제외됨
- e.g., (폴더명)

  
### 일반적인 전략
1. App 은 라우팅 목적으로만 나머지 components, lib는 루트경로에 위치시킴
<img src="https://github.com/user-attachments/assets/b816faac-251c-4847-9e27-9ec238c0b9ad" width="400px">

2. 모든 로직은 모두 app 내부에 위치
<img src="https://github.com/user-attachments/assets/502d22d8-5a90-4b12-88a4-5c903fbab73d" width="400px">

3. 공통로직은 app 내부에, 상세로직은 경로(e.g., /dashborad) 내부에 위치 - 예시 레포에서는 해당 방법 사용중
<img src="https://github.com/user-attachments/assets/6a280247-c5fa-46e6-994c-8f8ffbe1c6ef" width="400px">


# 3. 레이아웃과 페이지(Layouts and Pages)

# 4. 이미지와 폰트(Images and Fonts)

# 5. CSS와 스타일링(CSS and Styling)

# 6. 데이터 가져오기와 보여주기(Fetching data and streaming)

# 7. 데이터 변형(Mutating Data)

# 8. 에러핸들링(Error handling) 
