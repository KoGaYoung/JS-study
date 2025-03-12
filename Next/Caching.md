# 1. On this page
Next.js는 렌더링 작업과 데이터 요청을 캐싱함으로써 페이지 로딩 속도가 빨라지고 서버에 부담이 줄어듭니다.      
이 페이지에서는    
 - Next.js의 캐싱 메커니즘 (=어떻게 캐싱을 적용하여 성능을 향상시키는지)
 - 이를 구성하는 데 사용할 수 있는 API (캐싱 동작을 조정할 수 있는 API)
 - 그리고 이들이 서로 상호작용하는 방식   
에 대해 배웁니다.

참고: 이 페이지는 Next.js가 내부적으로 어떻게 작동하는지 이해하는 데 도움을 주지만, Next.js를 효과적으로 사용하기 위한 필수 지식은 아닙니다.
기본 설정만으로도 대부분의 경우 최적의 성능을 발휘할 수 있도록 설계되어 있습니다.

---

# 2. Overview

목적에 따라 다른 캐싱을 적용하기 위한 캐싱 매커니즘 개요

| 메커니즘       | 무엇을 캐싱하는가           | 위치       | 목적                                  | 기간                          |
|---------------|---------------------------|------------|-------------------------------------|-------------------------------|
| 요청 기억 Request Memoization   | 함수의 반환값              | 서버       | React 컴포넌트 트리 내에서 데이터 재사용 | 요청 단위 라이프사이클               |
| 데이터 캐시 Data Cache  | 데이터                    | 서버       | 사용자 요청과 배포 사이에서 데이터 저장   | 지속적(재검증 가능)                 |
| 전체 라우트 캐시 Full Route Cache | HTML 및 RSC 페이로드       | 서버       | 렌더링 비용 감소 및 성능 향상           | 지속적(재검증 가능)                 |
| 라우터 캐시 Router Cache  | RSC 페이로드               | 클라이언트  | 페이지 이동 시 서버 요청 횟수 감소       | 사용자 세션 또는 시간 기반            |

기본적으로, Next.js는 성능 향상과 비용 절감을 위해 가능한 한 많은 것을 캐싱합니다.   
별도로 캐싱을 해제하지 않는 한, 정적 라우트(Static Routes)는 빌드 시에 미리 렌더링되고 데이터 요청 역시 자동으로 캐싱됩니다.    

다이어그램은 네 가지 메커니즘에 대한 Next.js의 기본 캐싱 동작을 보여줍니다.   
빌드 시점과 라우트를 처음 방문할 때의 HIT, MISS, SET 과정을 나타냅니다.   
- MISS: 새롭게 가져와야하는상황
- SET: 새롭게 가져온거 저장
- HIT: 캐시 된거 갖고옴

<img src="https://github.com/user-attachments/assets/e6d731f1-faa5-48e8-ba27-1e8cabc366ab" width="400px" />

이미지 해설)   
CLIENT: 클라이언트 캐시(브라우저 스토리지)   
SERVER: 서버 측 캐시(Next.JS 내부 캐시)   
데이터소스: DB, 외부 API   

빌드 시점에 일어나는 일)   
/a 정적인 라우트(Static Route) 빌드 시점에 이미 HTML, RSC Payload가 생성되어 Full Route Cache에 저장(SET)   

요청 시점에 일어나는 일)   
최초로 접속하면, 클라이언트에서 라우트캐시를 확인하는데 MISS 일 수 있음 -> 가져와서 SET함   
HIT인 경우에는 캐싱해 둔 HTML/RSC Payload를 빠르게 전달함.   

라우트가 정적으로 렌더링되는지(Static) 동적으로 렌더링되는지(Dynamic),    
데이터가 이미 캐싱되어 있는지 여부, 그리고 방문이 최초인지 후속 방문인지에 따라 캐싱 동작이 달라집니다.   
필요에 따라 각 라우트와 데이터 요청별로 캐싱 정책을 세밀하게 조정할 수도 있습니다.   

---


# 3. Request Memoization
Next는 Fetch API를 래핑하여 요청들을 자동으로 메모제이션하도록 만들었습니다.   
리액트 컴포넌트 여러 위치에서 같은 URL을 가진 Fetch 요청을 하더라도 실제 실행은 한 번만 일어나는 것을 의미합니다.   
(GET메소드 일 떄만, URL과 쿼리스트링까지 같아야함)

예를들면, Page -> Layout -> 외 자식들 컴포넌트에서 죄다 getItem을 쓸 때,    
여러번 요청해도 실제로는 1번만 실행된다는 뜻입니다.

```tsx
async function getItem() {
  // `fetch`가 자동적으로 결과를 기억함(=캐시됨)
  const res = await fetch('https://.../item/1')
  return res.json()
}

// 두번 호출되더라도 실제로는 한 번만 호출됨 (첫 번째 요청은 호출)
const item = await getItem() // cache MISS

// 같은 경로 안에서는 어디든지 캐시 히트됨 
const item = await getItem() // cache HIT
```

<img src="https://github.com/user-attachments/assets/b685e2e9-5479-42e7-9191-d40cbe0d7ddf" width="400px" />

걍 첨에는 미스나서 디비가서 데이터 가져오고 SET해놓고   
그뒤에는 계속 HIT 한단 얘기   

요청 기억은 Next.JS 기능이 아니라 React.JS 기능입니다.   
다른 매커니즘과 어떻게 상호작용하는지 보여주기위해 공홈에 기술한것임    

메모화는 fetch의 GET 메소드에서만 발생함. 
React 컴포넌트 트리에만 적용됨 즉 generateMetadata, generateStaticParams, Layouts, Pages, 그리고 다른 서버컴포넌트들에서만 적용됨

라우트핸들러(Route Handlers)는 리액트 컴포넌트 트리가 아니므로 fetch요청이 적용되지 않음    
=> [라우트 핸들러](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)는 route.js/ts 파일을 말함.   
API를 대체하기 위한 경로겸 API 메소드

## 3.1 Duration 캐싱 기간
서버가 특정 요청을 처리하는 한 번의 렌더링 동안만 메모화된 데이터가 유지됩니다. 렌더링이 끝나면 캐시가 사라집니다.    

## 3.2 Revalidating 재검증
이 메모화는 서버 요청 간에 공유되지 않고, 오직 렌더링 과정에서만 유효합니다.       
따라서 “이 캐시를 갱신해야 하나?” 하는 고민 없이, 별도 재검증을 할 필요가 없습니다.       

## 3.3 Opting out
메모화는 GET 메서드에서만 동작하며, POST나 DELETE 같은 메서드는 자동으로 메모화되지 않습니다.    
이 기본 동작 자체가 React 최적화의 일부이므로, 따로 끄지 않는 것을 권장합니다.    
요청을 취소(abort)하기 위해 AbortController의 signal 옵션을 사용할 수 있지만,     
이는 메모화 자체를 비활성화하는 게 아니라, 진행 중인 요청을 중단할 때만 쓰입니다.

---

# 4. Data Cache
데이터 캐시는 서버 차원에서 데이터를 영구적으로(또는 재검증 가능하게) 보관해 두는 캐시입니다.    
(= 요청을 통해 받아왔던 DB에 있는 데이터를 캐싱)    
서버 요청이 새로 들어와도 또는 배포(Deployment)가 이루어져도, 이미 캐시에 저장된 데이터를 재사용할 수 있습니다.    
이게 가능한 이유는 Next가 fetch API를 래핑하여 서버의 각 요청이 자체적인 영구적 캐싱방식을 설정할 수 있도록 만들었기 때문임.    
fetch의 cache 옵션(예: 'force-cache', 'no-store')과 next.revalidate 옵션을 통해 캐싱 동작을 세부 조정합니다.    


<img src="https://github.com/user-attachments/assets/a64d09e8-a088-4b8c-8b34-d937f8ac7a72" width="400px"/>

이미지해설)   
렌더링 도중 'force-cache' 옵션을 가진 fetch 요청이 처음 호출되면, Next.js는 Data Cache에 캐시된 응답이 있는지 확인    
캐시된 응답이 발견되면, 즉시 반환되고 메모화됩니다.    
캐시된 응답이 없으면, 데이터 소스에 요청을 보내 결과를 가져오고, 이를 Data Cache에 저장한 뒤 메모화   

캐시되지 않은 데이터(예: cache 옵션이 정의되지 않았거나 { cache: 'no-store' }를 사용하는 경우)라면, 결과는 항상 데이터 소스에서 가져오고 메모화   
데이터가 캐시되었든 캐시되지 않았든, 동일한 데이터에 대한 중복 요청을 막기 위해 요청은 항상 메모화됩니다.

기본 설정은 **'no-store'**

요청 기억, 데이터 캐시 두 메커니즘 모두 캐시된 데이터를 재사용하여 성능을 향상시키지만,    
Data Cache는 들어오는 요청과 배포 전반에 걸쳐 지속적으로 유지되는 반면,    
요청 기억는 하나의 요청 라이프사이클 동안에만 유지

## 4.1 Duration 캐싱 기간
Data Cache는 재검증(revalidate)하거나 사용하지 않도록 설정(opt-out)하지 않는 한, 들어오는 요청과 배포 전반에 걸쳐 지속됩니다.
배포를하면 메모리가 다시올라가기떄문에 캐시가 사라진다.

## 4.2 Revalidating 재검증
캐시된 데이터는 다음 두 가지 방법으로 재검증할 수 있습니다:

- Time-based Revalidation 시간 기반 재검증: 일정 시간이 지나고 새로운 요청이 발생했을 때 데이터를 재검증   
- On-demand Revalidation 온디맨드 재검증: 이벤트(예: 폼 제출)에 따라 데이터를 재검증합니다.
  - 온디맨드 재검증은 태그 기반(tag-based) 또는 경로 기반(path-based) 방식을 사용해 한 번에 여러 데이터 그룹을 재검증할 수 있음

### 4.2.1 Time-based Revalidation 시간 기반 재검증
일정 간격으로 데이터를 재검증하려면, fetch의 next.revalidate 옵션을 사용해 리소스의 캐시 수명(초 단위)을 설정할 수 있음    
또는, Route Segment Config 옵션을 사용해    
해당 세그먼트 내의 모든 fetch 요청을 설정하거나, fetch를 사용할 수 없는 상황에 대해 구성할 수도 있음

```ts
// Revalidate at most every hour
fetch('https://...', { next: { revalidate: 3600 } })
```

<img src="https://github.com/user-attachments/assets/0b9c0155-bd4c-4ecd-80af-d6093683f5c3" width="400px" />

이미지설명)   
60초라고 되어있지만 실제로 60초뒤 바로 재검증을 수행하지 않음.   
60초 이후에도 동일한 (오래된) 캐시된 데이터를 리턴함   
Next 백그라운드 내부에서 데이터 재검증(STALE) 수행 하고 그때 SET해서 최신화함.   
이건 stale-while-revalidate 동작방식과 유사함   


### 4.2.2 On-demand Revalidation 온디맨드 재검증
데이터는 경로(revalidatePath)나 캐시 태그(revalidateTag)를 통해 온디맨드 방식으로 재검증할 수 있음

<img src="https://github.com/user-attachments/assets/c2539ec8-6034-4e0e-be1b-4096a56f0543" width="400px" />

이미지설명)   
fetch 요청이 처음 호출될 때, 외부 데이터 소스에서 데이터를 가져와 Data Cache에 저장함   
온디맨드 재검증이 트리거되면, 해당하는 캐시 항목이 캐시에서 제거(purge)됨    
이후 요청이 들어오면, 다시 캐시 MISS가 발생하고 외부 데이터 소스에서 데이터를 가져와 Data Cache에 저장   


## 4.3 Opting out
만약 fetch의 응답을 캐싱하고 싶지 않다면, 다음과 같은 방법을 사용할 수 있습니다
```ts
let data = await fetch('https://api.vercel.app/blog', { cache: 'no-store' })
```
---


# 5. Full Route Cache

관련있는 용어:    
 빌드 시점에 애플리케이션의 라우트를 렌더링하고 캐싱하는 과정을 가리켜, 
 ```Automatic Static Optimization```, ```Static Site Generation```, 또는 ```Static Rendering```이라는 용어를 서로 바꿔 사용

Next.js는 빌드 시점에 라우트를 자동으로 렌더링하고 캐싱합니다.   
이는 모든 요청마다 서버에서 렌더링하는 대신,    
캐시된 라우트를 제공하도록 해주어 더 빠른 페이지 로딩이 가능하게 하는 최적화 기법입니다.   
(=캐시된 라우트는 정적페이지가 서버나 CDN에 저장되있는것)   

Full Route Cache가 어떻게 동작하는지 이해하려면,   
React가 렌더링을 어떻게 처리하고, Next.js가 그 결과를 어떻게 캐싱하는지 아래 5단계로 살펴봐야함

## 5.1 step 1. React Rendering on the Server
서버에서 Next.js는 React의 API를 사용해 렌더링을 조정합니다.   
이때 렌더링 작업은 개별 라우트 세그먼트(/)와 Suspense 경계를 기준으로 여러 조각으로 나뉩니다.   

조각은 두 단계로 랜더링됨    
React는 서버 컴포넌트를 스트리밍에 최적화 된 데이터포멧인 RSC payload(React Server Component Payload)로 렌더링 함    
Nextsms RSC Payload와 클라이언트 컴포넌트.js를 사용해 서버에서 HTML을 렌더링함    

여러 조각으로 수행하는 작업은 모두 마칠 때까지 기다렸다가 캐싱하거나 응답을 보내는 것이 아니라,    
작업이 완료되는 대로 스트리밍 방식으로 응답을 보낼 수 있음을 의미함

## 5.1 step 2. Next.js Caching on the Server (Full Route Cache)
Next.js의 기본 동작은,    
1단계에서 서버에서 라우트의 렌더링 결과(React Server Component Payload와 HTML결과물)를 캐싱하는 것입니다.(여기가 핵심)    
이는 빌드 시점에 정적으로 렌더링된 페이지나 재검증 시에도 적용됨    

## 5.1 step 3. React Hydration and Reconciliation on the Client

HTML은 클라이언트 및 서버 컴포넌트의 비(非)인터랙티브한 초기 미리보기를 빠르게 보여주는 데 사용됩니다.     
RSC payload는 클라이언트와 렌더링된 서버 컴포넌트 트리를 비교(조정)하고, DOM을 업데이트하는 데 사용    
클라이언트 컴포넌트 js는 클라이언트 컴포넌트를 하이드레이션하여, 애플리케이션을 인터랙티브하게 만듭니다.    
(내가 알고있는 하이드레이션 말하는거 맞음)    

## 5.1 step 4. Next.js Caching on the Client (Router Cache)

Full Route cache(RSC Payload와 HTML결과물)은 서버에 저장되지만,    
RSC payload는 클라이언트 Router Cache (브라우저 스토리지)에 저장함.    

개별 경로 세그먼트별로 나뉜 별도의 인메모리 캐시입니다.    
이 Router Cache는 이전에 방문한 라우트를 저장하고,     
향후 라우트를 미리 가져와서 페이지 이동 경험을 향상시키는 데 사용됨

## 5.1 step 5. Subsequent Navigations
Next.js는 Router Cache에 React Server Component Payload가 있는지 확인합니다.    
만약 있다면, 서버로 새 요청을 보내지 않음

## 5.2 Static and Dynamic Rendering
빌드 시점에 라우트가 캐싱되는지 여부는, 그 라우트가 정적으로 렌더링되는지 동적으로 렌더링되는지에 달려 있음    
정적 라우트는 기본적으로 캐싱되지만, 동적 라우트는 요청 시점에 렌더링되며 캐싱되지 않음    

<img src="https://github.com/user-attachments/assets/185c1fdd-56af-4dff-90a8-0572914ed904" width="400px" /> 

## 5.3 Duration 지속시간
기본적으로 Full Route Cache는 지속적입니다.    
이는 렌더링 결과가 사용자가 달라져도 요청은 캐싱된 상태로 유지된다는 것을 의미    

## 5.4 Invalidation 무효화
두 가지 방법있음     
- 재배포(Redeploying): Data Cache와 달리, Full Route Cache는 새로운 배포가 이루어질 때 초기화됨    
- 데이터 재검증(Revalidating Data): Data Cache를 재검증하면, 서버에서 컴포넌트를 다시 렌더링하고 새 렌더링 결과를 캐싱함으로써, 결과적으로 Router Cache도 무효화   

## 5.5 Opting out
Full Route Cache를 사용하지 않도록 설정(Opt-out)하거나, 달리 말해 모든 요청마다 동적으로 컴포넌트를 렌더링하도록 하려면    

세 가지 방법 있음    
- 동적 API(Dynamic API)를 사용    
- dynamic = 'force-dynamic' 또는 revalidate = 0 라우트 세그먼트 설정 옵션을 사용하는 방법
  - : 이렇게 하면 Full Route Cache와 Data Cache를 건너뜀
  - 즉, 서버로 들어오는 모든 요청마다 컴포넌트가 렌더링되고 데이터를 가져오게 됩니다.
  - 다만, Router Cache는 클라이언트 측 캐시이므로 계속 적용
- Data Cache를 사용하지 않는 방법
  - : 만약 어떤 라우트가 캐싱되지 않는 fetch 요청을 포함하고 있다면,
  - 이는 해당 라우트를 Full Route Cache에서도 제외시킴


---


## 6. Client-side Router Cache
Next.js는 인메모리 클라이언트 사이드 라우터 캐시를 가지고 있으며, 라우터 캐시엔 RSC 페이로드를 저장해 둠.    
이 캐시는 레이아웃(layouts), 로딩 상태(loading states), 페이지(pages) 단위로 나눠져있음.    
( = 5.1 step 4. 에서 RSC payload는 클라이언트 Router Cache (브라우저 스토리지)에 저장함.)     

사용자가 페이지를 이동할 때, Next.js는 방문한 라우트 세그먼트를 캐싱하고,     
사용자가 이동할 가능성이 있는 라우트를 사전 로딩(prefetch) 합니다

뒤로/앞으로 이동이 즉시 이루어지고,     
라우트 간 이동 시 전체 페이지 리로드가 발생하지 않으며,     
React 상태와 브라우저 상태가 유지 됨.

라우터 캐시 활용:
- 레이아웃은 캐싱되어 탐색 시 재사용되며, 이를 통해 부분 렌더링이 가능함
- 로딩 상태 또한 캐싱되어, 탐색 시 즉시 재사용됨으로써 빠른 네비게이션을 제공
- 페이지는 기본적으로 캐싱되지 않지만, 브라우저에서 뒤로/앞으로 이동할 때는 재사용됩니다.
  - staleTimes라는 실험적 설정 옵션을 사용하면 페이지 세그먼트에 대한 캐싱을 활성화할 수 있음
-  Next.js와 서버 컴포넌트에만 특정적으로 적용되는 것이며, 결과는 유사하지만 브라우저의 bfcache와는 다름

## 6.1 Duration 유지기간
브라우저 임시 메모리 지속시간은 2가지 요소에 의해 결정됨
- Session: 페이지 이동시엔 유지되지만 페이지 새로고침시에는 지워짐
- 자동 무효화 기간: 정적페이지는 5분, 동적페이지는 캐싱적용 안됨. (완전 사전로딩시에는 정적 동적 모두 5분)
  - ```staleTimes```라는 실험적 설정 옵션을 사용하면 위에서 언급된 자동 무효화 시간을 조정가능

## 6.2 Invalidation 무효화
무효화 두가지 방법
- 서버액션 안에서
  -  ```revalidatePath(경로 기반)```나 ```revalidateTag(캐시 태그 기반)```를 사용해 온디맨드 방식으로 데이터를 재검증(revalidate)
   - 서버액션은 클라이언트에서 호출하니까, 클라이언트에서 연결되어있기 때문에 여기서 가능함
  -  ```cookies.set``` 또는 ```cookies.delete```를 사용하면 Router Cache가 무효화되어,
    -  (예: 인증 등) 쿠키를 사용하는 라우트가 오래된 상태로 남지 않도록 방지
- ```router.refresh``` 호출

## 6.3 Opting out
Next.js 15부터는 페이지 세그먼트가 기본적으로 캐싱에서 제외(opt-out)됨   
Link> 컴포넌트의 prefetch 속성을 false로 설정하면 사전 로딩(prefetching) 자체를 해제할 수도 있음

---


# 7. Cache Interactions
서로 다른 캐싱 메커니즘을 구성할 때, 이들이 어떻게 상호작용하는지 이해하는 것이 중요합니다.


## 7.1 Data Cache and Full Route Cache

Data Cache를 재검증하거나 사용하지 않도록 하면, 데이터를 기반으로 하는 Full Route Cache가 무효화됩니다.   
반대로 Full Route Cache를 무효화하거나 사용하지 않아도, Data Cache는 영향을 받지 않습니다.   
즉, 일부 컴포넌트만 요청 시점에 실시간 데이터를 가져오고, 나머지는 캐시된 데이터를 사용하는 하이브리드 방식이 가능하다는 의미입니다.   

## 7.2 Data Cache and Client-side Router cache

**서버 액션(Server Action)**에서 revalidatePath 또는 revalidateTag를 사용하면, Data Cache와 Router Cache를 동시에 즉시 무효화할 수 있습니다.   
Route Handler는 특정 라우트와 직접 연결되지 않으므로, 여기서 Data Cache를 재검증해도 Router Cache는 자동으로 무효화되지 않습니다.   
따라서 클라이언트 측에서는 하드 리프레시나 자동 무효화 기간이 지나야 새로운 데이터가 반영될 수 있습니다.   

---

서버에서 RSC Payload와 JS 번들을 저장하는데,   
따로 라우트캐시에 RSC Payload를 저장하는 이유는?   

RSC Payload를 라우트캐시(클라이언트쪽 브라우저스토리지)에 저장해놓는 이유는   
RSC Payload는 서버컴포넌트가 어디까진지 클라이언트는 어디서부터인지 표시해둔 HTML 데이터를 가지고있음.   

그러면 클라이언트에서 RSCPayload로 갖고있는 서버컴포넌트만 짜잔하고 먼저 렌더링해줄 수 있음
+ RSCPayload가 어떤 js 번들을 보는지도 가지고있음   


# 8. APIs
## 8.1 <Link>
## 8.2 router.prefetch
## 8.3 router.refresh
## 8.4 fetch
## 8.5 fetch options.cache
## 8.6 fetch options.next.revalidate
## 8.7 fetch options.next.tags and revalidateTag
## 8.8 revalidatePath
## 8.9 Dynamic APIs
### 8.9.1 cookies
## 8.10 Segment Config Options
## 8.11 generateStaticParams
## 8.12 React cache function
