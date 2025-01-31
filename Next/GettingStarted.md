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
Would you like your code inside a `src/` directory? No 주요 코드를 src 폴더 하위에 배치, No 선택 시 모든 코드가 루트에 배치 (앱라우트를 쓰는경우에는 어차피 app에 소스 배치하는 전략을 쓸 수 있기때문에 굳이)
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
 - public : 제공될 정적 자산, public 내부에 logo.svg 올려두면 e.g., https://42dot.vsm.com/logo.svg 경로로 이미지 접근 가능
   - 우리는 cdn 되어있으니 https://cdn.subcription/logo 로 쓰는게 더좋음
 - src
<img src="https://github.com/user-attachments/assets/843cd6fc-2305-4a7e-b358-ed979be0e61c" width="400px" />

### 최상위 파일
- 애플리케이션 구성, 종속성 관리, 미들웨어 실행, 모니터링 도구 통합, 환경 변수 정의
- next.config.js Next에 대한 구성 파일
- instrumentation.ts request,response 추적, 사용자데이터 수집, 분석도구 통합
  - OpenTelemetry를 사용하면 데이터독같은 도구로 데이터독 로깅 쉽게 가능
- middleware.ts Next 요청 미들웨어
  - request <-> response 보낼때, 받을 때 서버단에서 인터셉터하는 것. axios 인터셉터는 클라이언트단에서 인터셉트하는거라 조금 다름
- .env 환경변수 (default)
- .env.local 로컬환경변수 (최우선)
- .env.development 개발환경변수
- .env.production 운영환경변수 (node)
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

### 중첩된 경로 (뭔말임?? 여기서는 depth를 가져갈 수 있다는 의미)
- folder 경로구간
- folder/folder 중첩된 경로 세그먼트

### 동적 경로 -> 경로들을 한 번에 받아올 수 있다는 뜻
- [folder] // e.g., { slug: string }
- [...folder] // e.g., 	{ slug: string[] }
- [[...folder]] // e.g., { slug?: string[] }

e.g., app/blog/[productId]/[trimId]/page.js 이렇게 쓰는게 더 나아보임

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
- page 개별 페이지 정의
  - layout.tsx 파일에서 page.tsx를 명시적으로 렌더링하는 코드를 작성할 필요가 없음 Next.js가 자동으로 이를 처리
- error.js 에러바운더리
- loading.js 서스펜스
- not-found.js 에러바운더리

error.js를 넣으면 Error Boundary 랜더트리가 생성되는 구조   
만약에 루트에 suspense가 있는데 해당 경로에 suspense가 없으면 루트꺼까지 타고올라가서 봄
/page.js
/error.js
/loading.js
/blog/page.js

<img src="https://github.com/user-attachments/assets/863f5e99-95ab-4d80-9aa6-9d22a3327f9d" width="400px"/>

## 프로젝트 구성하기
### 공동배치
- pages.js가 존재해야 라우터로 접근 가능

<img src="https://github.com/user-attachments/assets/35677bd5-15e9-4748-9110-03cfe4471f6f" width="400px" />

<img src="https://github.com/user-attachments/assets/279d89f0-e7e2-49cf-b22c-a3176b301c6b" width="280px" />

### 개인폴더
- 개인 구현 세부 사항
- 밑줄그어 만들 수 있음
- 이 폴더 뿐만 아니라 하위폴더 전부 다 라우팅에서 제외됨.
- e.g., _폴더명

### 경로그룹
- 조직 목적으로 사용됨
- 괄호로 묶어서 만들 수 있음
- 이 폴더만 라우팅에서 제외됨
- e.g., (폴더명)

  
### 일반적인 전략
1. App 은 라우팅 목적으로만 나머지 components, lib는 루트경로에 위치시킴
<img src="https://github.com/user-attachments/assets/b816faac-251c-4847-9e27-9ec238c0b9ad" width="400px">

2. 모든 로직은 모두 app 내부에 위치
<img src="https://github.com/user-attachments/assets/502d22d8-5a90-4b12-88a4-5c903fbab73d" width="400px">

3. 공통로직은 app 내부에, 상세로직은 경로(e.g., /dashborad) 내부에 위치 - 예시 레포에서는 해당 방법 사용중
<img src="https://github.com/user-attachments/assets/6a280247-c5fa-46e6-994c-8f8ffbe1c6ef" width="400px">


# 3. 레이아웃과 페이지(Layouts and Pages)
## 페이지 생성
- 홈페이지 만들려면 app 폴더 내부에 page.js 구성하면 됨
<img src="https://github.com/user-attachments/assets/60c2fdd1-a36c-4ed5-a095-beeef283c806" width="400px">

## 레이아웃 만들기
- 레이아웃은 공통 UI 기능
- 다시 렌더링하지 않음
- 컴포넌트는 children 페이지 또는 다른 레이아웃이 될 수 있는 props 허용해야함
- 디렉토리 루트에 정의되어있는 것은 루트레이아웃. 루트레이아웃은 html, body를 포함해야함

<img src="https://github.com/user-attachments/assets/2c04a9f0-c45e-4a27-99d2-d7f2e6d84494" width="400px">

```tsx
// children 대신 다른 layout 컴포넌트가 들어갈 수 있는지? 들어간다면 어떻게 동작하는지??
// children 이 무조건 들어가야만 하는데, 
// 랜더트리가 app/layout.ts 하위에 app/blog/layout.ts가 구조상 바로 아래에 들어갈 수 있다는뜻
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        {/* Layout UI */}
        {/* Place children where you want to render a page or nested layout */}
        <main>{children}</main>
      </body>
    </html>
  )
}
```

## 중첩된 경로 생성
- 공식문서에서 말하는 '중첩'은 계층(depth)을 가질 수 있다는 의미로 쓰임
- page.js는 홈 url 경로(/)
- blog/page.js는 블로그 url 경로 (/blog)


<img src="https://github.com/user-attachments/assets/5d4d01e1-5b77-4e14-92cf-b4598975568f" width="400px">


```tsx
// import 경로를 보면 lib, ui 폴더가 있다는건데, lib, ui는 경로에 어떻게 포함안시켰는지?? _lib, (lib)도 아닌데
// -> page 컴포넌트없으면 어차피 경로로 인정안함ㅋㅋ
// 이 컴포넌트는 server components임 .
// 서버에서 렌더링에 대한 html만 클라이언트로 내려줌
// 서버에서 getPost 실행을 다하고 html만 내려주기때문에 당연히 api가 실행되는만큼 클라이언트로 페이지보여주는데 시간은 걸리겠지만
// nextjs가 대박인 이유는 서버컴포넌트에서 실행하는 동안에도 서스팬스 로딩을 보여줌!!
// next js에서 fetch api를 래핑해서 캐시 기능을 넣음.
// fetch 에서 cache를 10 분 걸어두면 10분동안은 
// ISR(컴포넌트 캐싱)이랑은 다른개념
// 넥스트는 서버컴포넌트를 기본으로 함. 서버컴포넌트는 html만 사용자한테 보낸다는건 번들사이즈가 줄어드는것.
// 클라이언트사이드렌더링은 빈 html, css, js 받아와서 지지고볶고 하는데 RSC(리액트 서버컴포넌트)html 밖에 없는건 상대적으로 성능이 좋다는것
// 가능하면 서버컴포넌트로 정적으로 만들어버리는게 좋음
 
import { getPosts } from '@/lib/posts'
import { Post } from '@/ui/post'
 
export default async function Page() {
  const posts = await getPosts()
 
  return (
    <ul>
      {posts.map((post) => (
        <Post key={post.id} post={post} />
      ))}
    </ul>
  )
}
```

- 계속해서 중첩할 수 도 있음

<img src="https://github.com/user-attachments/assets/90ffa10c-00eb-4b6d-aa8e-8b6fb0094ebc" width="400px">

```tsx
function generateStaticParams() {}
 
export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

## 중첩 레이아웃 
- 위에서 설명한 RootRayout, WebviewLayout과 같은개념
<img src="https://github.com/user-attachments/assets/433942a9-25c9-49ad-b5d6-cded17e209df" width="400px">

```tsx
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return <section>{children}</section>
}
```

## 페이지 연결
- Link API 넥스트에서 제공함
- next에서도 react-router-dom의 navigate(history)같은 방식으로 변경하고싶다면 useRouter 사용
```tsx
// 이게 개좋은이유가 개발자도구 켜놓고 보면 rsc로 페이지 이동시 doc을 받아오는게아니라
// fetch로 서버컴포넌트의 실행결과값을 보냄
// ssr이지만 csr같은 사용자 경험

import Link from 'next/link'
 
export default async function Post({ post }) {
  const posts = await getPosts()
 
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

# 4. 이미지와 폰트(Images and Fonts)
## 정적자산 처리
- 정적 자산은 루트경로의 public 폴더 하위에 배치

## 이미지 최적화
- next에서 Image api 제공 (최적화, 안정성, 크기조정 기능 등)

```tsx
import Image from 'next/image'
 
export default function Page() {
  return <Image src="" alt="" />
}
```

## 지역이미지 패스

## 원격이미지
- 안전하게 사용하려면 지원되는 url 패턴을 next.config.js에 정의

```tsx
import { NextConfig } from 'next'
 
const config: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
        search: '',
      },
    ],
  },
}
 
export default config
```

## 글꼴 최적화
- next/font 듈 next에서 제공 (외부 네트워크 요청 제거)
- 주로 Layout.js에서 설정

```tsx
import { Geist } from 'next/font/google' 
import localFont from 'next/font/local'

// 구글 글꼴
const geist = Geist({
  subsets: ['latin'],
})

// 로컬 글꼴
const myFont = localFont({
  src: './my-font.woff2',
})

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={geist.className}>
      <body>{children}</body>
    </html>
  )
}
```
- 단일 글꼴 패밀리에 여러 파일을 사용하려는 경우 src배열을 사용
```tsx
const roboto = localFont({
  src: [
    {
      path: './Roboto-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './Roboto-Italic.woff2',
      weight: '400',
      style: 'italic',
    },
    {
      path: './Roboto-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
    {
      path: './Roboto-BoldItalic.woff2',
      weight: '700',
      style: 'italic',
    },
  ],
})
```

# 5. CSS와 스타일링(CSS and Styling)
## CSS 모듈 -> 파일명.module.css

## Global css -> app/global.css, import './global.css'

## tailwind CSS

## Sass

## CSS-in-JS

## 외부꺼 e.g., 부트스트랩, antd


# 6. 데이터 가져오기와 보여주기(Fetching data and streaming)
## 데이터 가져오기
### 서버 구성요소
- fetch로 가져오기
  - axios도 가능한데 왜 fetch로 예시를 들어놧슬까
  - -> 서버컴포넌트는 nodejs에서 돌아간다는 뜻. xmlHttpRequest가 node엔 없고 browser에서만 쓸수있는거임
- orm 또는 db

### 클라이언트 구성 요소
- use 훅
  - 리액트에 추가된 api인데 안써봄 뭔 장점?
  - -> promise객체를 use 훅 안에 넣으면 resolve될 때까지 서스펜스해주는 것, 1회성
- swr, react-query 같은 로컬라이브러리

```tsx
// swr 예제
'use client'
import useSWR from 'swr'
 
const fetcher = (url) => fetch(url).then((r) => r.json())
 
export default function BlogPage() {
  const { data, error, isLoading } = useSWR(
    'https://api.vercel.app/blog',
    fetcher
  )
 
  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>
 
  return (
    <ul>
      {data.map((post: { id: string; title: string }) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```
  
## 스트리밍
- 스트리밍은 Next 15 cannary에서 도입된 플래그, dynamicIO config 옵션이 활성화 되어있다고 가정
- 위에 플래그가 뭔지 잘 모르겟으나, 동적렌더링을 async/await 할 때 느린데이터 요청이 있는 경우 리스트 /blog/[postId] postId 1~10까지 다 차단됨
- 초기 로드시간과 사용자 경험 개선을 위해 HTML을 청크 단위로 나누고, 청크를 점진적으로 서버에서 클라이언트로 전송 

스트리밍 구현방법
1. loading.js 파일
2. <Suspense> 컴포넌트 사용

- 그냥 서스팬스 fallback 적용하는 코드같은데, 이게 청크를 나눠서 점진적으로 클라이언트 전송하는것과 직접관련있음?
  -  suspense, fallback 적용만으로는 fetch 후 state 로 들어오는건 한번에 들어올 것 같은데..
  - -> 서스팬스 내부 코드(BlogList)를 서버에서 돌아서 청크로 쪼개서 주기 때문에 스트리밍이라고 하는것
<img src="https://github.com/user-attachments/assets/fbe5ef3c-7a3e-49f6-9f5e-f12feca07440" width="400px">

```tsx
// app/blog/loading.tsx
export default function Loading() {
  // Define the Loading UI here
  return <div>Loading...</div>
}
```

# 7. 데이터 변형(Mutating Data)
- next에서는 서버함수를 사용하여 서버로부터 받은 데이터 변형 가능 (bff 가능)

## 서버 기능 생성
- use server지시문
- 비동기함수 맨위에 작성
- 별도파일 생성
- 
```ts
// app/actions.ts
'use server'
 
export async function createPost() {}
```

```tsx
'use client'
 
import { createPost } from '@/app/actions'
 
export function Button() {
  return <button formAction={createPost}>Create</button>
}
```

## 서버 기능 호출
### 양식
### 이벤트 헨들러
### 보류상태 표시
### 캐시 재검증
### 리다이렉션 중

# 8. 에러핸들링(Error handling) 
