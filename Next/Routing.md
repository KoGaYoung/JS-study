# 1. 레이아웃 및 템플릿 (Layouts and Templates)
next 는 파일 시스템 기반 라우팅을 사용   
레이아웃과 페이지를 만들고 이를 연결하는 방법 기술   

## 페이지 생성
page 컴포넌트는 특정한 경로에서 렌더링되는 UI 컴포넌트입니다.   
app 디렉토리 안 page 컴포넌트에서 html을 return 하면 페이지를 만들 수 있습니다.   

```tsx
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```
<img src="https://github.com/user-attachments/assets/7004b15f-5763-44f4-87fd-cdbfff1ec083" width="400px" />

## 레이아웃 생성
레이아웃은 여러 페이지에서 공유되는 공통 UI 입니다.   
페이지를 이동하면서 레이아웃은 상태를 보존하고, 상호작용을 유지하며, 다시 렌더링하지 않습니다.   

기본적으로 React 컴포넌트를 layout에서 return 해서 레이아웃을 정의할 수 있습니다.   
커뫂넌트는 chldren 페이지 또는 다른 레이아웃 컴포넌트가 될 수 있는 props를 허용해야합니다.   

인덱스 페이지를 자식으로 허용하는 레이아웃을 만들려면 app 디렉토리 아래에 layout 컴포넌트를 만듭니다.   
(/app/page.tsx가 루트 경로(/)의 인덱스 페이지)   
루트레이아웃이라고 부르며, html 과 body 태그를 반드시 포함해야합니다.   

```tsx

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

<img src="https://github.com/user-attachments/assets/98fefdb3-3f8b-4785-bf62-7d994be1c632" width="400px" />


## 중첩된 경로 생성 (뎁스가 있는 경로 생성)
중첩 경로는 여러 URL 세그먼트로 구성됩니다.   

/blog/[slug]
- / 로트 세그먼트
- blog 세그먼트
- [slug] 잎 세그먼트

정리하면   
Next 에서    
폴더는 Url 세그먼트에 매핑되는 경로 세그먼트를 정의하는데 사용   
파일은 page, layout 은 ui 만드는데 사용됨   

```tsx
// app/blog/pages.tsx
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

// app/blog/[slug]/pages.tsx
function generateStaticParams() {}
 
export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

<img src="https://github.com/user-attachments/assets/8a26d279-94b4-441b-b557-90c20acade5a" width="400px" />

## 중첩 레이아웃
레이아웃도 중첩할 수 있음.
e.g., 루트 레이아웃, 블로그 영역 레이아웃


<img src="https://github.com/user-attachments/assets/630315ba-5f42-4456-8d3b-d8075dc3b74b" width="400px" />

루트 레이아웃은 /page.tsx 와 /blog 둘 다를 래핑하고,
블로그 레이아웃은 /blog/page.tsx 와 /blog/slug/page/tsx 를 래핑함.

## 페이지 간 연결
Next에서 제공하는 Link 컴포넌트로 react-router-dom 의 기능을 함.

블로그 게시글 목록을 작성하려면 아래와같이 작성

```tsx
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

# 2. 연결 및 탐색 (Linking and Navigating)
## 연결 및 탐색   
넥스트에서 경로 이동하는 방법은 네가지가 있습니다.   
1. Link 컴포넌트 사용
2. useRouter 훅 사용 (클라이언트 컴포넌트에서만)
3. redirect 함수 사용 (서버 컴포넌트에서만)
4. native History API 사용


 ## 1. Link 컴포넌트 사용
a태그를 (상속받아서) extends하여 만든 넥스트 빌트인 컴포넌트.
 경로간의 클라이언트사이드 이동을 합니다.
가장 기본적이고 권장되는 방법. 자세한 props들은 [여기](https://nextjs.org/docs/app/api-reference/components/link)

## 2. useRouter 훅 사용 (클라이언트 컴포넌트에서만)
얘를 사용하면 클라이언트사이드에서 프로그래밍 방식(헨들러 에서 Js로직으로 페이지이동) 경로를 변경 할 수 있음

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
}
```

## 3. redirect 함수 사용 (서버 컴포넌트에서만)
소스먼저 보세용

```tsx
import { redirect } from 'next/navigation'
 
async function fetchTeam(id: string) {
  const res = await fetch('https://...')
  if (!res.ok) return undefined
  return res.json()
}
 
export default async function Profile({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const id = (await params).id
  if (!id) {
    redirect('/login')
  }
 
  const team = await fetchTeam(id)
  if (!team) {
    redirect('/join')
  }
 
  // ...
}
```

알아두면 좋을정보)
- redirect는 기본적으로 307(임시 리디렉션) 상태코드를 반환.
- 서버작업에서 사용되는 경우 303(기타참조)를 반환. -> POST 요청의 결과로 성공 페이지로 리디렉션하는데 사용됨
- 내부적으로 오류가 발생하므로 try/catch 블록 내부에서 호출
- 렌더링 프로세스 중에 클라이언트 구성요소에서 redirect를 호출할 수 있지만, 이벤트 헨들러에서는 호출 불가능
- redirect 는 절대경로를 허용하며 외부 링크로 리디렉션하는데도 사용할 수 있음
- 렌더링 프로세스 전에 리디렉션 하려면 next.config.js 또는 Middleware를 사용하세요

## native History API 사용
Next는 window.history 객체의 pushState, replaceState를 사용하여
페이지를 다시 로드하지 않고도 브라우저의 기록을 스테이트하는 것을 허용합니다.

그렇다고 pushState와 replaceState를 쓰지말고 next에서 제공하는 usePathname, useSearchParams로 구현가능

```tsx
'use client'
  
import { useSearchParams } from 'next/navigation'
 
export default function SortProducts() {
  const searchParams = useSearchParams()
 
  function updateSorting(sortOrder: string) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }
 
  return (
    <>
      <button onClick={() => updateSorting('asc')}>Sort Ascending</button>
      <button onClick={() => updateSorting('desc')}>Sort Descending</button>
    </>
  )
}

import { usePathname } from 'next/navigation'
 
export function LocaleSwitcher() {
  const pathname = usePathname()
 
  function switchLocale(locale: string) {
    // e.g. '/en/about' or '/fr/contact'
    const newPath = `/${locale}${pathname}`
    window.history.replaceState(null, '', newPath)
  }
 
  return (
    <>
      <button onClick={() => switchLocale('en')}>English</button>
      <button onClick={() => switchLocale('fr')}>French</button>
    </>
  )
}
```

## 라우팅 및 탐새 작동 방식
앱라우터는 라우팅과 이동에 하이브리드 접근방식을 사용합니다.
서버에서 애플리케이션 코드는 자동적으로 세그먼트 단위로 코드스플릿팅됨.
클라이언트에서 Next는 prefetch(사전패치) 하고 캐시합니다.

이것이 의미하는 바는, 유저가 새로운 경로로 이동했을 때, 브라우저는 페이지를 다시 로드하지않고 변경된 경로 세그먼트만 리렌더링합니다.
성능과 사용성을 개선합니다.

1. 코드스플릿팅 : 작은 번들 단위로 만들어짐 전송속도, 실행속도 줄어듬 (레이지로딩할때처럼)
2. 프리패칭 : 백그라운드에서 경로를 미리 로드하는 2가지 방법
- Link 구성요소: 사용자 뷰포트에 표시되면 자동으로 사전패치됨. 
- router.prefetch() : useRouter 후크로 사용

```tsx
import Link from 'next/link';

// 이떄 About 페이지 링크가 화면에 나타나면 Next는 페이지를 미리 다운로드함
// prefetch={false}로 프리패칭 못하게 할 수도 있음
export default function Home() {
  return (
    <div>
      <h1>홈페이지</h1>
      <Link href="/about">About 페이지로 이동</Link>
    </div>
  );
}
```


링크는 loading.js 사용여부에 따라 동작이 다르다.

loading.js가 있는 경우)
- 프리패칭 시, 전체 페이지가 아닌 공유레이아웃(layout.tsx)까지만 가져옴
  - 컴포넌트 트리에서 처음으로 등장하는 loading.js 까지의 레이아웃만 프리패칭됨
  - 동적 경로 전체를 미리 가져오는 부담을 줄이고
  - 로딩상태를 사용자에게 공유할 수 있음

프리패칭 된 데이터는 30초 동안 캐시에 저장됨. 
같은 링크를 30초 내에 방문하면 새로 요청하지않고 캐시된 데이터를 사용하게됨.

```
app
 ├── layout.js        (🌍 최상위 레이아웃)
 ├── page.js          (🏠 "/")
 ├── about
 │   ├── loading.js   (⌛ "About" 페이지 전용 로딩 UI)
 │   ├── layout.js    (📄 "About" 페이지 전용 레이아웃)
 │   ├── page.js      (ℹ️ "/about")
 │   ├── team
 │   │   ├── page.js  (👥 "/about/team")
 │   │   ├── ...

```
사용자가 <Link href="/about/team">가 뷰포인트 내에 존재하는 화면을 보면
about/team 페이지를 방문하기 전에 프리패칭되는 범위는?

✅ app/layout.js (최상위 레이아웃)
✅ app/about/layout.js (about 전용 레이아웃)
❌ app/about/team/page.js → 이 부분은 미리 패칭되지 않음!
❌ app/about/team의 실제 콘텐츠는 로드되지 않음.


3. 캐싱
Nextjs에는 Router cache라는 메모리 내 클라이언트 캐시가 있습니다.
사용자가 프리패칭해온 경로 세그먼트와 방문한 경로의 plaload가 캐시에 저장됩니다.

4. 부분렌더링
이동중에 변경되는 라우터 세그먼트만 다시 클라이언트에서 렌더링됨, 공유되는 부분은 보존됨.

```
app
 ├── layout.js        (🌍 최상위 레이아웃 - 모든 페이지 공통)
 ├── page.js          (🏠 "/")
 ├── dashboard
 │   ├── layout.js    (📊 대시보드 전용 레이아웃)
 │   ├── page.js      (📂 "/dashboard")
 │   ├── analytics
 │   │   ├── page.js  (📈 "/dashboard/analytics")
 │   ├── settings
 │   │   ├── page.js  (⚙️ "/dashboard/settings")
```

✅ /dashboard → /dashboard/analytics로 이동하려고 
사용자가 <Link href="/dashboard/analytics">을 클릭하면?
🔹 렌더링이 일어나는 부분
✅ app/dashboard/analytics/page.js (다시 렌더링됨)

🔹 렌더링되지 않고 유지되는 부분
🚫 app/layout.js (유지됨)
🚫 app/dashboard/layout.js (유지됨)

---

✅ /dashboard/analytics → /dashboard/settings로 이동하려고 
사용자가 <Link href="/dashboard/settings">을 클릭하면?

🔹 렌더링이 일어나는 부분
✅ app/dashboard/settings/page.js (다시 렌더링됨)

🔹 렌더링되지 않고 유지되는 부분

🚫 app/layout.js (유지됨)
🚫 app/dashboard/layout.js (유지됨)

<img src="" width="400px" />

5. 소프트네비게이션
브라우저는 페이지간 이동 시 "하드탐색" 수행
넥스트는 "소프트탐색" 활성화해서 변경된 부분만 다시 렌더링(=4. 부분렌더링) 이를통해 react 상태보존 가능
-> 이래서 layout에 contextAPI 와같은 전역라이브러리 쓰는듯

6. 뒤로 앞으로 탐색
라우터 캐시에 있는 겨올 세그먼트를 재사용함

7. pages와 app 사이의 라우팅
pages 라우터에서 app 라우터로 갈 때 하드 탐색으로 처리한다.

-> 무슨말이냐면 app안에 특정경로가 있는지 추측함. (확률적검사) 넥스트가 잘못판단하여 오류발생확률 0.01% 기본설정되있음.
-> 아래처럼 설정가능한데, 이러면  정확해지지만 클라번들 크기 증가함

module.exports = {
  experimental: {
    clientRouterFilterAllowedRate: 0.005, // 0.005%로 설정 (더 낮게 조정 가능)
  },
};

# 3. 오류처리 (Error Handling)

# 4. UI로딩 및 스트리밍 (Loading UI and Streaming)

# 5. 리다이렉션 (Redirecting)

# 6. 경로 그룹 (Route Groups)

# 7. 동적 경로 (Dynamic Routes)

# 8. 병렬 경로 (Parallel Routes)

# 9. 경로 가로채기 (Intercepting Routes)

# 10. 경로 핸들러 (Route Handlers)

# 11. 미들웨어 (Middleware)

# 12. 국제화? (Internationalization)
