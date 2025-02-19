# 1. 리다이렉팅 Redirecting
## 이번페이지에서는
Next에서 redirect는 자동으로 페이지를 이동시키는 것을 말합니다.
e.g., 비로그인 상태에서는 로그인 화면으로 이동

사용가능한 옵션, 예제, 리다이렉션을 관리하는 방법을 배웁니다.

| API | 목적 | 어디 | 상태 코드 |
|----------|----------|----------|----------|
| redirect | 데이터변경(mutation) 또는 이벤트 후 페이지이동 | 서버 컴포넌트, 서버액션, 라우트핸들러 | 307(임시), 303(서버액션) |
| permanentRedirect | 데이터변경(mutation) 또는 이벤트 후 페이지이동 | 서버 컴포넌트, 서버액션, 라우트핸들러 | 308(영구) |
| useRouter | 클라이언트사이드 이동 수행 시 | 클라이언트 컴포넌트에서 이벤트 핸들러 | N/A |
| redirects in next.config.js | 라우트(경로)기반 들어오는 요청에 대한 페이지이동 | next.config.js 파일 | 307(임시), 308(영구) |
| NextResponse.redirect | 조건에 따라 들어오는 요청에대한 페이지이동 | 미들웨어 | Any |

## redirect function
```redirect``` 함수는 사용자를 다른 URL로 보내버리는 것이 가능하다.   
```redirect```는 서버컴포넌트, 라우트 핸들러, 서버액션에서 호출 가능합니다.   

```tsx
'use server'
 
import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'
 
export async function createPost(id: string) {
  try {
    // Call database
  } catch (error) {
    // Handle errors
  }

  // /posts 페이지에 표시되는 게시글 목록이 캐시(저장된 데이터)로 인해 오래된 상태일 수 있는데, 이를 최신 상태로 업데이트(재검증)해요.
  revalidatePath('/posts')

  // Navigate to the new post page
  redirect(/post/${id}) 
}
```
### Good to know:
```redirect```을 호출하면 307(임시)로 상태코드가 반환됩니다.   
(= <b>"잠시"</b>동안 다른 곳으로 이동한다는 뜻)   

서버액션을 사용할때는 ```redirect```은 303(서버액션)을 반환합니다.   
(= e.g.,POST 요청 후 성공 페이지로 이동할 때)   

```redirect``` 함수는 내부적으로 에러를 발생시키기 때문에,   
try/catch 블록 안에서 호출하면 안 됩니다.   
(= 반드시 try/catch 밖에서 사용)

```redirect```는 렌더링 과정중에는 클라이언트 컴포넌트에서도 사용할 수 있지만 이벤트 핸들러안에서는 안됩니다.   
(= 이벤트 헨들러에서는 useRouter 훅을 사용해야한다)   
-> createPost 를 클라이언트 컴포넌트에서 못쓰기 때문에,    
-> "새로운 글 쓰기" 버튼을 클라이언트 인터렉션 을통해 클릭하면 submit()등을 통해 createPost를 호출(like api 와 같다고 보면 됨)   

```redirect```는 절대경로 URL을 인자로 받아서 외부 사이트로 페이지이동 할 수 있다.   

렌더링 과정 전에 페이지이동을 하고 싶다면 next.config.js 또는 Middleware를 사용해야합니다   

## permanentRedirect function

```permanentRedirect``` 는 사용자를 <b>"영구적"</b>으로 다른 경로로 이동하게 할 수 있습니다.   
서버컴포넌트, 라우트 핸들러, 서버액션에서 사용할 수 있습니다.   

```permanentRedirect``` 는 엔터티의 표준 URL을 변경하는 데이터변경(mutation) 또는 이벤트 후에 자주 사용됩니다.     
엔터티의 표준 URL 의미(=공식적으로 대표되는 URL이 바뀌는 것)    
e.g., https://homepage.com/{username} 에서 사용자 이름이 바뀌는 경우     
(= 이전 페이지로 돌아가도 변경된 사용자 이름으로 보여야함)    

```tsx
'use server'
 
import { permanentRedirect } from 'next/navigation'
import { revalidateTag } from 'next/cache'
 
export async function updateUsername(username: string, formData: FormData) {
  try {
    // Call database
  } catch (error) {
    // Handle errors
  }
 
  revalidateTag('username') // Update all references to the username
  permanentRedirect(`/profile/${username}`) // Navigate to the new user profile
}
```

### Good to know:
```permanentRedirect``` sms 308(영구) 코드를 디폴트로 반환한다    
```permanentRedirect``` 는 절대경로 URL를 받아서 외부링크로 리다이렉트 하는것도 가능하다.   
```permanentRedirect``` 를 렌더링과정 전에 하고싶으면 ```next.config.js``` or ```middleware```를 써라.   


## useRouter() hook
react-router-dom을 넥스트안에서 지원하는것과 같음.
클라이언트사이드 이벤트헨들러는 useRouter()를 사용

```tsx
'use client'
 
import { useRouter } from 'next/navigation'
 
export default function Page() {
  const router = useRouter()
 
  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  )
```

### Good to know:
로직에서 사용하는거 아니면 Link태그 사용 권장


## next.config.js 로 redirects
```next.config.js``` 파일에서 제공하는 ```redirect``` 옵션은 들어오는 요청을 다른 경로로 페이지 이동하게끔 해줍니다.   
URL 이 업데이트되어 변경된 경우, 이전 URL -> New URL 로 리다이렉트     
미리 어떤 URL이 어떤 URL로 연결되어야 하는지 목록을 작성해 두고, 이를 적용할 수 있음   

요청 경로(path)뿐 아니라 요청에 포함된 헤더, 쿠키, 쿼리(query) 등을 기준으로 조건을 설정할 수 있어서, 유연하게 페이지 이동을 구성할 수 있습니다.   

```tsx
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  async redirects() {
    return [
      // Basic redirect
      {
        source: '/about',
        destination: '/',
        permanent: true,
      },
      // Wildcard path matching
      {
        source: '/blog/:slug',
        destination: '/news/:slug',
        permanent: true,
      },
    ]
  },
}
 
export default nextConfig
```

### Good to know:
307 (임시 리디렉션): URL이 잠시 동안 다른 곳으로 이동함을 의미합니다.   
308 (영구 리디렉션): URL이 영구적으로 변경되었음을 의미합니다.   
permanent 옵션을 설정하면 어떤 상태 코드가 반환될지 결정할 수 있어요.   

플랫폼 별 리다이렉트 갯수 제한)     
Vercel에서는 최대 1,024개의 리디렉트 규칙만 사용할 수 있습니다.   
만약 1,000개 이상의 리디렉트를 관리해야 한다면, Middleware를 사용해서 커스텀 솔루션을 만드는 것을 고려   

```next.config.js``` 의 페이지 이동 규칙은 ```Middleware``` 보다 먼저 실행 됨.   

## NextResponse.redirect in Middleware

```NextResponse.redirect``` in ```Middleware``` 는 요청이 완료되기 전에 특정 조건에 따라 다른 사용자를 URL로 보내주는 기능입니다.   
(=next.config.js와 차이점은 조건(분기)가 있냐 없냐인 듯)   

e.g., 로그인하지 않은 사용자가 관리자 화면에 들어올 수 없음.   
```tsx
import { NextResponse, NextRequest } from 'next/server'
import { get } from '@vercel/edge-config'
 
type RedirectEntry = {
  destination: string
  permanent: boolean
}
 
export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  const redirectData = await get(pathname)
 
  if (redirectData && typeof redirectData === 'string') {
    const redirectEntry: RedirectEntry = JSON.parse(redirectData)
    const statusCode = redirectEntry.permanent ? 308 : 307
    return NextResponse.redirect(redirectEntry.destination, statusCode)
  }
 
  // No redirect found, continue without redirecting
  return NextResponse.next()
}
```

### Good to know:
```next.config.js``` 의 페이지 이동 규칙은 ```Middleware``` 보다 먼저 실행 됨.   

## Managing redirects at scale (advanced)
대규모 프로젝트에서 (많은 수의) 페이지 이동을 관리를 효율적으로 처리하는 방법
```next.config.js``` 의 ```redirect``` 옵션은 정적인 규칙들을 미리 정의해두는 방식임.   
규칙 수가 많거나 자주 바뀌면 ```NextResponse.redirect``` in ```Middleware```가 더 유용할 수 있음

### 1. 리디렉트 맵 ( Creating and storing a redirect map )
이전 URL과 새로운 URL을 매핑한 목록   
```js
{
  "/old": {
    "destination": "/new",
    "permanent": true
  },
  "/blog/post-old": {
    "destination": "/blog/post-new",
    "permanent": true
  }
}
```


### 2. Middleware를 활용한 동적 리디렉트 ( Optimizing data lookup performance )
요청 URL을 확인하고 리다이렉트 맵에서 규칙을 찾아서 ```NextResponse.redirect```로 처리      
리다이렉트 규칙을 코드 안에서 프로그래밍 방식으로 관리할 수 있음 (앱 재배포 안해도 됨)   

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server'
import { get } from '@vercel/edge-config'
 
type RedirectEntry = {
  destination: string
  permanent: boolean
}
 
export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  const redirectData = await get(pathname)
 
  if (redirectData && typeof redirectData === 'string') {
    const redirectEntry: RedirectEntry = JSON.parse(redirectData)
    const statusCode = redirectEntry.permanent ? 308 : 307
    return NextResponse.redirect(redirectEntry.destination, statusCode)
  }
 
  // No redirect found, continue without redirecting
  return NextResponse.next()
}
```
데이터 조회 성능 최적화
 - 많은 수의 리디렉트를 빠르게 확인하기 위해 Bloom filter와 같은 알고리즘을 활용
 - 데이터 조회 속도를 개선하고, Middleware의 성능 저하를 방지
---

# 2. 경로 그룹 Route Groups
## 이번페이지에서 On this page
app 디렉토리에서는 기본적으로 폴더 구조가 URL 경로와 동일하게 반영됨.   
```Route Group``` 를 사용하면 폴더를 URL 경로에 포함시키지 않고도 프로젝트 파일을 논리적으로 그룹화할 수 있음   
(= ()랑 _같은 애들)    

- URL 경로를 변경하지 않으면서 파일이나 컴포넌트를 그룹으로 묶을 수 있게 해줍니다.   
- 사이트 섹션, 의도, 팀 등으로 라우트를 깔끔하게 분류할 때 유용   
- 동일한 라우트 세그먼트 내에서 중첩 레이아웃을 사용할 수 있게 해줌
  - 여러 개의 중첩 레이아웃이나 루트 레이아웃을 같은 세그먼트 내에서 관리할 수 있음
  - 공통 세그먼트에 속하는 일부 라우트에만 레이아웃이나 로딩 스켈레톤을 적용할 수 있음

e.g., /app/blog/post/page.tsx가 있다면 URL은 /blog/post가 되지만   
(blog)와 같이 Route Group으로 지정하면 URL에는 반영되지 않고, 내부적으로 파일을 그룹화하는 용도로만 사용됨

## 컨벤션 Convention
폴더 이름을 괄호로 묶어 경로 그룹을 만들 수 있습니다.```folderName```

## 예시 Examples

### URL 경로에 영향을 주지 않고 경로를 구성 Organize routes without affecting the URL path

필요한 Url은 /about, /blog, /account 인데,   
about과 blog는 마켓팅의 용도로 쓰이고 account는 쇼핑몰 관련으로 쓰인다.   
(marketing), (shop) 경로로 묶으면, url에는 marketing이 들어가지 않으면서 파일만 한데 모을 수 있다.

<img src="https://github.com/user-attachments/assets/aec7225a-d7fd-43ae-8b4d-9a7c088ffa2a" width="400px"/> 


### 레이아웃에 특정 세그먼트 선택 Opting specific segments into a layout
경로 그룹은,    
파일만 한데 모으는 기능만 있는게 아니라, marketing, shop 폴더별로 공통 UI(레이아웃.js)를 추가할 수도 있다.

<img src="https://github.com/user-attachments/assets/57624b46-27b1-4786-bc11-ec4635d12ed9" width="400px"/> 

### 특정 경로에 스켈레톤을 적재하도록 선택 Opting for loading skeletons on a specific route
(overview) 경로에 ```loading.js``` 도 추가할 수 있음    
아래 사진의 경우 /dashboard/(orverview)/경로 하위에 있는 ```page.tsx``` 에만 ```loading.js``` 적용 됨.

<img src="https://github.com/user-attachments/assets/45a04c75-4e7d-437b-bcfd-ffe21227fd65" width="400px"/> 

### 여러개의 루트 레이아웃 생성 Creating multiple root layouts

여러 루트 레이아웃을 만들려면 최상위 ```layout.js```파일을 제거하고 각 경로 그룹 내에 ```layout.js```파일을 추가함.   
이 경우 ```(marketing)/layout.js``` 와 ```(shop)/layout.js``` 는 완전히 별개의 UI를 가질 수 있음   
단 이 경우엔 ```html```, ```body``` 추가해야함.   


### Good to know:
- Route Group의 이름은 단순히 파일과 라우트를 정리하기 위한 용도
- 동일한 URL 충돌 주의:
  - 예를 들어, (marketing)/about/page.js와 (shop)/about/page.js 안됨 오류남
-  여러 루트 레이아웃을 사용한다면, 홈 페이지인 page.js 파일은 반드시 하나의 Route Group 안에 있어야 합니다.
  -  app/(marketing)/page.js -> 실제 경로: /
  -  app/(shop)/page.js -> 실제 경로: /
  -  이렇게는 사용하면 안됨  page.js는 app/(marketing) or app/(shop) 한 군데만 있어야함.
---


# 3. 동적 라우트 Dynamic Routes
## On this page
```Dynamic Segments```를 사용하면 URL의 일부(세그먼트)를 정확하게 정하지(만들지) 않아도, 동적인 데이터에 따라 URL이 만들어집니다.   
- 요청 시 채워지는 경우
  - ```/post/[slug]``` 는 요청이 들어왔을 때 ```/post/my-first-post```로 채워짐
- 빌드 시 pre-render 되는 경우
  - 빌드 시 ```getStaticPaths```와 같은 동적경로를 미리 계산함


## Convention
Dynamic Segment는 폴더 이름을 대괄호([])로 감싸서 생성합니다.    
예를 들어, [id]나 [slug] 같은 이름을 사용하면, 해당 폴더는 URL의 동적 부분을 의미하게 됩니다.

```
app/
 └── [id]/
      └── layout.js // layout.js은 /[id] 경로에만 적용됨
      └── page.js
```

사용자가 /123 URL에 접근하면,    
[id] 폴더는 id 라는 이름의 동적 세그먼트를 나타내며,    
실제 URL의 값 123이 해당 ```params``` prop을 통해 ```layout```, ```page```, ```route```, 그리고 ```generateMetadata``` 함수 등에 전달로 전달됩니다.    

## Example

```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const slug = (await params).slug
  return <div>My Post: {slug}</div>
}
```
| Route | Example URL | params |
|----------|----------|----------|
| app/blog/[slug]/page.js | /blog/a | { slug: 'a' } |
| app/blog/[slug]/page.js | /blog/b | { slug: 'b' } |
| app/blog/[slug]/page.js | /blog/c | { slug: 'c' } |

## Good to know
동적 세그먼트에서 전달되는 params prop이 비동기(promise)로 제공됩니다. 그래서 값을 사용하려면 아래 두 가지 방법 중 하나를 사용
- async/await 사용
- React의 use() 함수 사용

## Generating Static Params
```generateStaticParams``` 함수는 동적 세그먼트(예: [slug])와 함께 사용되어, 빌드 시점에 미리 정적인 경로들을 생성할 수 있게 도와줍니다.   
   

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
 
  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

```/app/blog/[slug]/page.tsx``` 경로일 때 ```generateStaticParams``` 함수는 API를 통해 모든 ```pages.tsx```의 데이터를 가져오고 각 포스트의 slug를 반환   
반환된 slug 값들을 기반으로 각 포스트의 URL이 빌드 시에 미리 생성되므로, 사용자가 요청할 때마다 동적으로 페이지를 생성할 필요가 없음

 ```generateStaticParams``` 내부에서 fetct를 쓰면 자동으로 momoized 됨 (=캐싱됨)
 즉, 동일한 요청은 한 번만 실행되어 빌드 시간을 단축시켜 줍니다.

이 코드를 보면, fetch('https://.../posts')로 posts를 받아올 수 있음    
posts = [1,2,3,4,5,6,7,8,9,10] 인데, 만약 페이지가 추가되어 posts = [1,2,3,4,5,6,7,8,9,10,11]이 되면 어떤 상황이 생겨?    
 -> [1,2,3,4,5,6,7,8,9,10] 각 동적 경로의 페이지를 정적으로 생성   
 -> 새로운 포스트(예: 11번)가 추가되어 posts 배열이 [1,2,3,4,5,6,7,8,9,10,11]이 되더라도, 이미 빌드된 결과에는 11번에 해당하는 페이지가 포함되어 있지 않음    
 -> 즉, 별도의 재빌드나 ISR(Incremental Static Regeneration) 같은 기능을 사용하지 않으면, 새로운 포스트 페이지는 자동으로 생성되지 않아 404 오류가 발생할 수 있음   

(동적 경로지만, 빌드전까지 수정되지 않을 경우에만 써야겠다)

## Catch-all Segments
세그먼트 한번에 받아오기
동적 세그먼트는 괄호 안에 줄임표를 추가하여 이후의 모든 세그먼트를 포함 하도록 확장할 수 있습니다 [...folderName].
순서보장

| Route | Example URL | params |
|----------|----------|----------|
| app/shop/[...slug]/page.js | /shop/a | { slug: ['a'] } |
| app/shop/[...slug]/page.js | /shop/a/b | { slug: ['a', 'b'] } |
| app/shop/[...slug]/page.js | /shop/a/b/c | { slug: ['a', 'b', 'c'] } |

## Optional Catch-all Segments
포괄적인 세그먼트는 매개 변수를 이중 대괄호로 묶어 옵셔널(?)으로 만들 수 있습니다 : [[...folderName]].

| Route | Example URL | params |
|----------|----------|----------|
| app/shop/[[...slug]]/page.js | /shop | { slug: undefined } |
| app/shop/[[...slug]]/page.js | /shop/a | { slug: ['a'] } |
| app/shop/[[...slug]]/page.js | /shop/a/b | { slug: ['a', 'b'] } |
| app/shop/[[...slug]]/page.js | /shop/a/b/c | { slug: ['a', 'b', 'c'] } |

## TypeScript
```tsx
//app/blog/[slug]/page.tsx
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  return <h1>My Page</h1>
}
```

| Route | params Type Definition |
|----------|----------|
| app/blog/[slug]/page.js | { slug: string } |
| app/shop/[...slug]/page.js | { slug: string[] } |
| app/shop/[[...slug]]/page.js | { slug?: string[] } |
| app/[categoryId]/[itemId]/page.js | { categoryId: string, itemId: string } |

---

# 4. 병렬 라우트 Parallel Routes
## On this page
```Parallel Routes```는 Next.js 앱 디렉토리에서 하나의 레이아웃 내에 여러 개의 페이지(또는 UI 섹션)를 동시에 혹은 조건에 따라 렌더링할 수 있도록 해줍니다.    
예를 들어 대시보드 페이지를 생각해보면, 대시보드에는 여러 섹션(예: 팀 정보, 분석 데이터 등)이 동시에 보여져야 할 때가 있음

자세한설명)
- 동시렌더링
  - 팀 관련 정보와 분석 데이터를 각각 별도의 컴포넌트로 분리하여 동시에 렌더링할 수 있음
  - 한쪽 섹션이 로딩되는 동안 다른 섹션은 이미 보여줄 수 있어 사용자 경험이 향상됨
- 조건부 렌더링
  - 상황에 따라 특정 섹션만 보여주거나 숨길 수 있음.
  - 예를 들어, 사용자의 권한에 따라 분석 데이터 섹션을 표시하지 않을 수도 있습니다. 
- 공유 레이아웃
  - ```Parallel Routes``` 사용하면,
  - 하나의 레이아웃 파일 내에서 여러 동적 경로를 동시에 관리할 수 있어
  - 레이아웃의 일관성을 유지하면서도 다양한 데이터를 독립적으로 처리할 수 있음

아래 사진은 @team/A, @analytics/B를 동시에 보여줘야 하는 상황.


<img src="https://github.com/user-attachments/assets/a4edcad3-170f-48d8-bbf9-262eb99303a0" width="400px" />


## Slots
```Named Slots```사용하면 하나의 페이지(레이아웃) 안에 여러 페이지를 동시에 보여줄 수 있음

```
app/
 └── dashboard/
      ├── layout.js
      ├── page.js
      ├── @analytics/
      │    └── page.js
      └── @team/
           └── page.js
```
폴더 이름 앞에 **@**를 붙여서 각각 @analytics와 @team이라는 슬롯(구역)을 만들면
대시보드 레이아웃 내에서 지정한 위치에 @team/A과 @analytics/B 페이지가 동시에 렌더링됩니다.

```tsx
// app/layout.tsx
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```

```Named Slots``` 들은 부모 layout의 props로 전달받을 수 있다.   
현재상황에서는 루트레이아웃은 @analytics와 @team을 Props로 전달받는다.   

슬롯은 URL의 일부로 간주되지 않아서, 실제 경로에 영향을 주지 않습니다   
/@analytics/views라고 하더라도, URL에는 @analytics가 나타나지 않고 /views로 표시됨
(= 그룹경로 같은앤데 병렬 라우트만 할 수 있는건가? -> 맞음 (단 병렬라우트할때만 써야함) )

+ @A랑 @B 가 병렬적으로 동시에 렌더링한다는게,   
 @A와 @B는 서버에서 동시에 렌더링을 시작해 독립적으로 처리함(속도가 빠름)    
다만, 부모 레이아웃은 서버 컴포넌트이기 때문에 두 결과가 모두 준비된 후에 최종적으로 한 번에 클라이언트에게 전달되어 클라이언트는 한 번에 보게됨    

## Active state and navigation
Next.js는 각 슬롯마다 현재 활성화된 서브페이지(상태)를 추적합니다.   
- Soft Navigation (클라이언트 내비게이션)
  - 사용자가 링크를 클릭하는 등 클라이언트 사이드 내비게이션을 할 때, URL이 바뀌어도 한 슬롯 내의 서브페이지만 바뀌고, 다른 슬롯들은 이전 활성 상태를 그대로 유지합니다.
  - 예를 들어, 한 슬롯에서 새로운 페이지로 전환되더라도 다른 슬롯은 사용자가 이전에 본 상태 그대로 남아 있게 됩니다.
  - E.g., app/blog/detail -> app/blog/introduce 이 경우 detail->introduce 영역만 바뀜 blog/layout까지는 그대로 남아있음

```
app
ㄴ blog
   ㄴlayout.js
   ㄴ page.js // /blog
   ㄴdetail
     ㄴ page.js // blog/detail
   ㄴintroduce
     ㄴ page.js // blog/introduce
```
- Hard Navigation (전체 페이지 새로고침)
- 브라우저를 새로고침하거나 URL을 직접 입력하는 등 전체 페이지가 로드되면, Next.js는 URL과 일치하지 않는 슬롯의 활성 상태를 알 수 없게 됩니다.
- 이 경우, 해당 슬롯에는 기본적으로 default.js 파일이 렌더링됩니다. 만약 default.js가 없으면 404 페이지가 나타납니다.

```
app/dashboard/
app/dashboard/
 ├── layout.js
 ├── page.js          // URL: /dashboard
 ├── @analytics/
 │     ├── page.js     // URL: /dashboard/analytics (클라이언트 내비게이션 시 활성화될 슬롯)
 │     └── default.js  // 하드 내비게이션 시 활성 슬롯 정보가 없으면 보여줄 기본 콘텐츠
 └── @team/
       ├── page.js     // URL: /dashboard/team (클라이언트 내비게이션 시 활성화될 슬롯)
       └── default.js  // 하드 내비게이션 시 기본 콘텐츠
```
사용자가 /dashboard URL에 있을 때 브라우저를 새로고침하면, URL에는 활성 슬롯(@analytics, @team)에 대한 정보가 포함되어 있지 않습니다.   
Next.js는 어떤 슬롯이 활성 상태여야 하는지 판단할 수 없게 되고, 각 슬롯에 대해 별도로 정의된 default.js 파일의 내용을 렌더링합니다.   

@analytics 폴더는 슬롯임을 나타내기 위해 이름에 @를 붙인 것이며, 슬롯은 URL 경로에 포함되지 않습니다.   
즉, 파일 구조가 app/dashboard/@analytics/page.js라고 하더라도, 실제 URL에는 @analytics가 나타나지 않고 부모 경로인 /dashboard의 일부로 렌더링됩니다.   

### default.js
초기 렌더링이나, 하드 탐색인 경우 default.js를 suspense로 정의할 수 있음

@team은 default가 없고 @analyics는 default 가 있는 경우

<img src="https://github.com/user-attachments/assets/b44c4dd1-9f2c-4483-9f74-2ed9391c98b2" width="400px"/>


/settings 경로로 이동하면, @team 슬롯은 /settins 페이지에 렌더링된다. (현재 @analytics 슬롯을 유지한채로.)

1. Soft Navigation (클라이언트 내비게이션) 시나리오:
- /settings URL로 이동하면, @team 슬롯에서는 /settings 페이지를 보여주고,
- 동시에 @analytics 슬롯은 원래 보던 페이지(예: /analytics 페이지)를 그대로 유지합니다.
- 즉, 슬롯 간 상태를 서로 독립적으로 유지할 수 있어요.
2. Hard Navigation (브라우저 새로고침) 시나리오:
- 새로고침하면 Next.js는 어떤 슬롯에 어떤 페이지가 활성화되어 있었는지 기억하지 못합니다.
- 그래서 @analytics 슬롯에는 기본(default.js) 페이지를 렌더링하려고 시도해요.
- 만약 @analytics에 default.js가 없다면, 404 페이지를 대신 보여줍니다.

Hard Navigation (브라우저 새로고침) 하면 Next에서 그냥 처음부터 서버에서 layout부터 해서 클라이언트로 받아와서 @analytics @team 둘 다 다시받아오면 되는거아니야?    
@analytics 슬롯에는 기본(default.js) 페이지를 렌더링 시도하는게 이해가 안가   
 -> Parallel Routes에서는 URL에 명시되지 않은 슬롯에 대해서는 하드 새로고침 시 “어떤 페이지를 렌더링해야 하는지” 정보를 알 수 없기 때문입니다.   
 (상위 layout 에서 병렬로 렌더링 되는거라서)

[유튜브](https://www.youtube.com/watch?v=wi8kF8UniUI) 
보니까 suspese로 묶인애들 병렬라우트 슬롯으로 만듬
-> 각각 따로 섹션별로 그려짐

그냥 layout 하위에 team, analytics 만들면 되지 왜 병렬 라우트를 써야하는지 알려줘
그니까, team -> analtics로 이동하면 team의 상태값들이 사라지는데
병렬라우트를 이용하면 team 섹션과 analytics 섹션의 상태값을 다 유지할 수 있단느거지? -> O

완전히 화면이 반갈죽 된 화면같은거 쓸 때,
e.g., 1 홈화면과 로그인화면 다 /home url을 쓰는 경우
앱안에서 관리자같은 경우에는 관리자전용기능 이스터에그같은것을 보고 싶을 때
e.g., 2. 인터셉트 라우트를 구현하기 위해서(인터셉트 라우트를 구현하기 위해 병렬라우트는 필수조건임)

### useSelectedLayoutSegment(s)

## Examples

### Conditional Routes

### Tab Groups

### Modals

#### Opening the modal

#### Closing the modal

### Loading and Error UI
---

