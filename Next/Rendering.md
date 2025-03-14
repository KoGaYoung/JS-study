# 1. 서버 컴포넌트 (Server Components)

## 서버 컴포넌트
리액트 서버 컴포넌트(RSC)를 사용하면 UI를 서버에서 렌더링하고 선택적으로 캐싱할 수 있습니다.   
Next.js에서 렌더링 작업은 스트리밍과 부분 렌더링을 가능하게 하기 위해 경로 세그먼트별로 추가로 분할됩니다.   
이를 위한 3가지 전략이 있습니다.   

- 정적렌더링 (Static Rendering)
- 동적렌더링 (Dynamic Rendering)
- 스트리밍 (Streaming)

## 서버 컴포넌트 렌더링의 이점

- 데이터 패칭
  - 서버 컴포넌트를 사용하면 데이터 패칭을 서버에서 수행하여, 작업자의 데이터 소스와 가까운 곳에서 처리할 수 있습니다.
    - 라운드 트립 비용이 클라이언트에서 호출하는 것 보다 DB서버 가까이에서 호출하는게 빠르다. 
  - 이러면 렌더링에 필요한 데이터를 패치해오는데 걸리는 시간을 줄이고
  - 클라이언트에서 필요한 요청수를 줄임으로써 성능이 향상됩니다.
- 보안
  - 서버컴포넌트는 민감데이터와 토큰과 API 키와 같은 서버에 있는 로직을 클라이언트에 노출할 위험없이 보호합니다.
    - 서버 컴포넌트의 결과는 사용자에게 보여지지만, 서버컴포넌트의 로직은 사용자에게 노출이 안됨.(rsc_payload만 내려감)
- 캐싱
  - 서버에 렌더링하면서, 결과물은 캐싱되어 후속 요청과 여러 사용자에게 재사용 될 수 있습니다.
  - 이를 통해 성능을 개선하고, 각 요청마다 반복적으로 렌더링하거나 데이터를 가져오는 부담을 줄여 비용을 절감할 수 있습니다.
- 성능
  - 서버컴포넌트는 기본성능을 최적화 할 수 있는 추가적인 도구들을 제공합니다.
  - 예를들어, 너가 전체 클라이언트 컴포넌트로 구성된 앱을 시작할 때, 유저인터렉션이 없는 부분을 서버컴포넌트로 변경하면 클라이언트 사이드 자바스크립트 코드의 양이 줄어듭니다.
  - 이거는 작은 클라이언트사이드 자바스크립트를 다운받고 분석하고 실행하기 때문에 느린 인터 또는 저성능 디바이스 유저에게 유용하다.
- 초기 페이지 로드 및 첫번 째 콘텐츠 페인트(First Content Paint, FCP)
  - 서버에서는, 페이지를 랜더하기 위해 핑료한 자바스크립트를 다운받고 분석하고 실행하는 것을 기다리지 않고,
  - 사용자들에게 즉시 페이지를 보여줄 수 있는 HTML을 생성할 수 있다.
  - (서버컴포넌트는 CSSOM+DOM 계산해서 렌더트리 만들지않고 (논인터렉션이니) 랜더트리 만들어진걸 가져온다는 뜻?)
  - 사실 HTML 서버에서 생성된게 브라우저렌더링에 어느단계까지 만들어지는건지 알고싶음
- 검색 엔진 최적화 및 소셜 네트워크 공유성
  - 서버에서 렌더링된 HTML은 검색 엔진 봇이 페이지를 색인화하는 데 사용할 수 있으며,
  - 소셜 네트워크 봇이 페이지의 미리 보기(소셜 카드)를 생성하는 데 활용될 수 있습니다.
- 스트리밍
  - 서버컴포넌트는 청크단위로 렌더링 작업을 분리할 수 있습니다.
  - 그리고 청크들이 준비되는 대로 클라이언트에 스트리밍할 수 있도록 합니다.
  - 이를 통해 사용자는 전체 페이지가 서버에서 렌더링될 때까지 기다리지 않고 일부 콘텐츠를 더 빨리 확인할 수 있습니다.

## 서버컴포넌트를 Next.js에서 사용하기
디폴트로 넥스트는 서버 컴포넌트가 디폴트입니다.   
이로써 추가적인 설정 없이 서버렌더링을 구현할 수 있습니다.    
그리고 클라이언트 컴포넌트는 필요하면 선택할 수 있습니다.   

## 서버 컴포넌트는 어떻게 렌더링되는지?
서버에서, 넥스트는 렌더링을 조율하기 위해서 React Api를 사용합니다.(useEffect, useState같은것)   
렌더링은 청크단위로 분리됩니다: 개별 라우트 새그먼트와 서스팬스 바운더리(<pages/> 영역을 감싸는 <Suspense fallback={} />)   

각 청크는 두단계로 렌더됩니다.    
- 첫번째: 리액트는 서버컴포넌트를 React Server Component Payload(RSC Payload) 라고 불리는 특별한 데이터 포멧으로 렌더링합니다.
- 두번째: 넥스트는 RSC Payload와 클라이언트 컴포넌트의 자바스크립트 명령(로직)을 사용하여 HTML을 서버에서 렌더링한다.
  - 이게 서버컴포넌트에서 사용자 인터렉션이 없으니까 RSC 페이로드를 보내서 클라이언트가 화면을 즉시 볼 수 있는것까지는 이해했는데,
  - Javascript 명령을 사용하여 서버에서 HTML을 렌더링한다는건 Javascript 코드를 어떻게 한다는건지?
  - -> 답안: 서버컴포넌트 렌더링에 자바스크립트가 여전히 중요한 역할을 함 (두가지 역할)
    - 첫번째는 RSC Payload로는 HTML을 못 만듬. 자바스크립트가 서버에서 돌아가야 만들 수 있음
    - 두번째는 서버컴포넌트 자식컴포넌트인 클라이언트 컴포넌트는 서버에서 완전히 렌더링 할 수 없기에 클라이언트쪽은 HTML안에 자바스크립트를 삽입함. e.g., Script 태그안에 하이드레이션을 위한 코드가 들어감

요약하면   
서버에서 초기렌더링용 HTML을 rsc payload를 통해 만들고,   
클라이언트에 HTML과 rsc payload와 js번들 같은 내용을 보내고,   
클라이언트는 처음에는 html을 받아서 보여주고,      
서버에서 만들어진 html과 클라이언트 가상 DOM을 일치시키고, 그위에 js번들로 받은 인터렉트 기능을 붙이는 작업이 하이드레이션임.   
하이드레이션을 수행함.   

클라이언트에서는,    
- 서버에서 생성된 HTML을 즉시 표시하여 사용자가 빠르게 비상호작용(preview) 상태의 페이지를 볼 수 있도록 합니다. (이는 초기 페이지 로드에만 해당됩니다.)
  -  서버에서 RSC Payload를 생성, 사용자에게 즉시 HTML 표시 (자바스크립트가 서버에서 돌아가서 HTML로 만들어줌, 비상호작용 상태의 페이지임)
-  클라이언트에서 RSC Payload를 받아 트리를 동기화 & DOM 업데이트
  - 이제 Next.js는 서버에서 받은 RSC Payload를 사용하여, 기존의 클라이언트 측 React 트리와 비교(diffs) 함.
  - 변경된 부분(서버에서 렌더링된 UI)이 있다면, 이를 React의 reconciliation(조정) 과정을 통해 DOM에 반영함
  - = 서버 렌더트리와 클라이언트 렌더트리를 맞추는 과정
- Client Components 하이드레이션 진행
  - 이제 Next.js는 Client Components를 하이드레이션해서 인터랙티브하게 만듦. (이 때 비상호작용 컴포넌트 -> 사용자가 버튼누를 수 있는 상호작용 컴포넌트가됨)
  - React는 서버에서 미리 렌더링된 UI를 유지하면서, 클라이언트에서 필요한 JavaScript를 실행하여 동작하게 함.

(rsc payload는 렌더링된 리액트서버컴포넌트 트리의 압축된 바이너리표현임.   
서버컴포넌트의 렌더링된 결과물,    
클라이언트 컴포넌트의 렌더링 위치를 표시한 플레이스홀더 및 JS 파일 참조,   
서버컴포넌트에서 클라이언트 컴포넌트로 전달된 Props   
)

## 서버 컴포넌트 전략
서버 컴포넌트는 정적(Static) 동적(Dynamic), 스트리밍(Streaming) 세 가지 하위 범주가 있습니다.

### 정적렌더링 (Static Rendering) (default)
정적렌더링에서는 라우트(소스코드)가 빌드 시점과 데이터 재검증 후 백그라운드에서 렌더링됩니다.   
렌더링 결과는 캐시되고 콘텐츠 전송 네트워크(Content Delivery Network, CDN)로도 전송할 수 있습니다.   
이 최적화 작업 덕분에 렌더링 결과물을 유저들과 서버요청들 사이에서 공유하게 해줍니다.   
(한 번 작업해두면 여러사용자가 최적화된결과를 볼 수 있다는 뜻)    

정적 렌더링은 해당 경로가 사용자에게 개인화되어있지 않은 데이터를 가지고 있을 때 유용합니다.
e.g., 정적 블로그포스트나 상품페이직 같은 것

### 동적렌더링 (Dynamic Rendering)
동적 렌더링은 사용자가 요청을 할 때마다 경로가 랜더링됩니다.

다이나믹 렌더링은 사용자에게 개인화되어있는 데이터를 가지고있을 떄 또는 요청했을 때만 알 수 있는 정보를 가지고있을 때 유용합니다.
e.g., 쿠키나 URL serach param 같은 것

동적 라우팅 (with cache data)
대부분의 웹사이트는 완전히 정적이거나 동적이지 않습니다. 그 중간(스펙트럼)에 위치해있습니다.   
예를들어, 주기적으로 재검증(재로딩)되는 상품데이터를 캐시하는데,   
개인화된 고객데이터는 캐시하지않는 전자상거래 페이지가 있을 수 있습니다.   
Next 에서는 캐시된 데이터와  캐시되지 않은 데이터를 모두 포함하는 동적 렌더링 라우팅을 사용할 수 있습니다.   
이는 src 페이로드와 데이터가 별도로 캐시되기 때문에 가능합니다.   
이로인해 요청할 때 모든 데이터를 가져오는 성능영햐엥 대해 걱정하지 않고 동적렌더링을 선택할 수 있습니다.   
[전체 경로 캐시](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache) 와 [데이터 캐시](https://nextjs.org/docs/app/building-your-application/caching#data-cache) 에 대해 자세히 알아보세요. 

#### 동적 렌더링으로 전환
렌더링 중에 [동적 API](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-apis) 또는 [패치](https://nextjs.org/docs/app/api-reference/functions/fetch) 옵션({ cache: 'no-store'}) 이 발견되면, 넥스트는 전체경로를 동적렌더링으로 변경합니다.
아래 표에는 정적으로 또는 동적으로 렌더링될 때 동적 API 및 데이터 캐싱이 경로에 영향을 어떻게 미치는지 요약

요약 : 동적 API || 캐시데이터 없으면 동적으로 렌더링됨.    
동적 API 없고 캐시데이터 있으면 정적으로 렌더링됨.   
= 모든 데이터가 캐시되어야 정적으로 렌더링됨.   


| 동적 API | 캐시데이터 | 경로 |
|----------|----------|----------|
| 아니오| 캐시데이터있음(캐시됨)| 정적으로 렌더링됨|
| 예 | 캐시데이터있음(캐시됨)| 동적으로 렌더링됨|
| 아니오| 캐시되지않음 | 동적으로 렌더링됨|
| 예| 캐시되지않음 | 동적으로 렌더링됨|

Next가 사용된 기능과 API를 보고 각 경로에 가장 적합한 렌더링 전략을 자동으로 선택하기 때문에 개발자가 뭐 할건 없음.    
대신 특정 데이터를 캐시하거나 재검증(재로딩) 되는 시기를 선택하고 UI일부를 스트리밍 하도록 선택할 수 있습니다.


### 동적 APIs
사전지식: 동적 API가 있으면 Next는 해당 서버컴포넌트를 동적렌더링함.
동적 API는 요청 시점에만 알 수 있는 정보에 의존합니다. (프리렌더링중에는 알 수 없음)
동적 API에는 이런것들이 있음
- coookies
- headers
- connection
- draftMode
- searchParams
- unstable_noStore


### 스트리밍
넥스트에서 자동으로 적용하는 내장기능으로, 초기 페이지 로딩 성능을 높이고 (코드스플릿팅을 통해) 전체경로 렌더링을 차단하며 
느린 데이터 패치에 로딩이 늦게되는 UI를 개선하는데 도움이 됨.

스트리밍은 청크로 쪼개져서 동시에 (병렬로) 처리되는 것을 말하는게 아니라, 청크가 준비되자마자 클라이언트로 전달되는게 스트리밍임.   
loading.js와 깊은 관련이 있음   
loading.js -> suspense태그가 만들어지고 fallback에 들어감 -> suspense 단위로 청크가 만들어짐 -> suspense 단위로 스트리밍이 일어남    
서버에서 내부적으로 모든 작업을 병렬로 처리하진 못함. 그저 청크를 나누어서 만들자마자 전송해버려서 클라이언트가 점진적으로 렌더링할수 있게함.   
클라이언트는 청크를 받고 즉시 렌더링 할 수 있음. 이것이 마치 동시에 처리되는 것 처럼 보임.   


# 2. 클라이언트 컴포넌트 (Client Components)
클라이언트 컴포넌트는 인터렉티브한 UI를 만들 수 있습니다.
(이 인터렉티브한 UI는 이전 설명에서 나왔듯이 서버에서 프리렌더링이 수행된 컴포넌트)

이전 내용을 통해 내가 알고있는 영역)
서버컴포넌트는 RSC payload를 통해 서버에서 프리렌더링을 수행해.
그리고 이후에 하이드레이션을 통해 클라이언트 컴포넌트가 논인터렉션 -> 인터렉션 컴포넌트로 변경됨.

1. 클라이언트 컴포넌트의 프리렌더링 수행됨 ? O
2. rsc payload에 클라이언트 영역의 html도 포함되어있어? ? 굳이 따지자면 X, 클라이언트 컴포넌트 최종 전체 HTML보다는 여기라고 알려주는 자리표시(자리를위한 placeholder)가 있음
3. 서버에서 프리렌더링된 클라이언트 컴포넌트 영역은 loading.js로 보여져? O 하이드레이션되기 전(해당 영역의 데이터나 컴포넌트 준비가 늦어지면) 로딩UI가 보여지는게 맞음

## 클라이언트 렌더링의 이점 Benefits of Client Rendering
- 인터렉션: 클라이언트는 state, effect, eventLisnter를 사용하기때문에 사용자에게 즉시 피드백을 주고 UI를 업데이트할 수 있다
- Browser APIs: geolocation, loaclStorage와같은 브라우저 api 사용가능

## Next.js에서 클라이언트 컴포넌트 사용하기 Using Client Components in Next.js
'use client' 를 파일 상단에 선언.   
서버 컴포넌트와 클라이언트 컴포넌트 사이에 경계를 선언하는것.   
use client 사용은 그 파일 모든 import된 모듈과 자식컴포넌트들을 클라이언트 번들이 된다는 것임.   

```tsx
'use client'
 
import { useState } from 'react'
 
export default function Counter() {
  const [count, setCount] = useState(0)
 
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

우측처럼 바운더리가 생김   
(페이지.js가 아니고 모바일 네비는 뭐임?)

<img src="https://github.com/user-attachments/assets/df30d400-e6d6-4f41-9305-6ad1bddbb9d2" width="400px" />

"use client" 를 여러개 선언하면 엔트리 포인트를 정의할 수 있음 => 여러개의 클라이언트 번들로 분할 가능. (코드슷플리팅, 캐시, 최적화에 좋음)   
=> 이왕이면 클라이언트 컴포넌트는 use Client를 넣어주고 시작하자   


## 클라이언트 컴포넌트는 어떻게 렌더링되나요? How are Client Components Rendered?
- 전체 페이지 로드(첫방문 or 새로고침) 상황
- 네비게이션(페이지이동) 상황
  - -> 사전지식 next에 네비게이션은 SPA처럼 동작하기위해 데이터만 패치해온다고 알고있음

### 전체 페이지 로드 Full page load
- React의 API를 사용하여 클라이언트 및 서버 구성 요소 모두에 대한 정적 HTML 미리보기를 서버에서 렌더링
- JS번들 다운로드 + 구문분석 + 실행까지 기다릴 필요없이 서버에서 렌더링된 거 즉시 서빙하여 콘텐츠 즉시보여줌. (하이드레이션되기 전상태)

(좀더 자세히)
서버에서는    
첫번째로 . 리액트 서버구성요소 React Server Component Payload(RSC Payload)라는 데이터형태로 렌더링함.    
RSC Payload에는 클라이언트 HTML 전체보다는 여기에 클라이언트컴포넌트 들어간다는 표시(플레이스호더처럼) 만해둠   
두번쨰로 . 서버에서 Javascript가 돌아가면서 RSC 페이로드로 초기 렌더링용 HTML을 만들고 HTML과 rsc payload와 js번들을 클라이언트로 보냄.   
(초기 렌더링용 HTML 만들 떄 renderToString, renderToPipeableStream 사용)   
 e.g., 헤더와 네비바가 서버컴포넌트라면 즉시 보여줌 컨탠츠 영역은 (로딩이 들어갈테니) 로딩으로 보이게됨   

(서버에서는 시퀀스단위로 코드실행하고 html과 rsc_payload를 보내고 JS번들을 안보냄! -> 데이터가 작으니 이것도 이점)

클라이언트에서는 서버로받은 HTML을 즉시보여줌   
클라이언트는 서버로부터 전달받은 HTML과,   
클라이언트에서 계산한 가상돔을 일치시킴. (=조정함)      
이후에 전달받은 JS번들에 있는 인터렉션(이벤트헨들러같은거)를 돔에 같다붙이는데 이게 하이드레이션임.      
클라이언트는 하이드레이션을 수행함.     

### 이후의 내비게이션Subsequent Navigations
이후에 화면 이동은 서버에서 렌더링된 HTML없이 전적으로 클라이언트 렌더링 됨.    
이 말의 뜻은, 클라이언트 구성요서 JS번들을 다운로드해서 파싱함.    
번들이 준비되면 React는 RSC Payload를 사용하여 클라이언트 및 서버구성요소 트리를 조정하고 DOM 업데이트    
(탐색시에는 변경이 필요한 부분만(세그먼트가 달라진 부분만) JS번들과 RSC Payload만을 받아온다는것. 서버에서 만든 html없이.)


## 서버 환경으로 돌아가기Going back to the Server Environment
useClient를 선언했지만, 서버환경으로 돌아가고싶을 때가 있다.
예를들어 그런상황은 클라이언트 번들 사이즈를 줄이거나, 서버에서 데이터를 패치해오거나, 서버에서만 사용할 수 있는 API를 사용할때다.

이론적으로 클라이언트 구성 요소 내부에 중첩되어 있더라도 클라이언트 및 서버 구성 요소와 서버 작업을 섞어서 코드를 서버에 보관할 수 있습니다 .    
자세한 내용은 [Composition Patterns](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns) 페이지에서

# 3. 컴포지트(구성) 패턴 (Composition Patterns)
- 객체들의 관계를 트리 구조로 구성하여 부분과 전체를 나타내는 패턴

서버컴포넌트, 클라이언트 컴포넌트가 이제 뭔지 암.
각각 어떤 상황에서 어떤걸 선택할래?
언제 뭘 고를꺼냐

데이터 패치야? 서버컴포넌트
백앤드리소스? 서버컴포넌트
훅써? 클라이언트 컴포넌트
...

## 서버컴포넌트 패턴

### sharing data between components

기본적으로는 서버에서 최대한 패칭하고, 어쩔 수 없을 땐 클라이언트에서 한다.   
서버에서 데이터 패칭할때는   
첫번째로. 서버 컴포넌트에서 데이터 읽거서 클라이언트로 props로 읽어라.   
두번째로. 서버에서 API 통신을 하는데, 이 데이터를 caching해서 다른 컴포너트에서 사용해도 쉐어링되도록.   


### Keeping Server only Code out of the client enverionment
민감 데이터를 쓸 때 (process.env.API_KEY) 는 서버에서만 쓸수 있는 데이터임. 
import 'server-only' 하면 클라이언트에서 접근 못하는 값이 됨
(client-only 도 있다)   
NEXT_PUBLIC 를 쓰면 클라이언트에서도 접근가능하다

### Using Thrid-party packages and providers
요즘은 서버컴포넌트를 대체로 지원함.
유저 인터렉션이 있는 경우엔 <Carousel /> 같은거 쓰는데 서버컴포넌트 <Carousel />호출하면 에러났음.
클라이언트 컴포넌트로 한번 감싸서 넣어라 라는 회피구문도 제공함

#### using providers
보통 React ContextAPI는 보통 최상단에 넣음.
Layout에 넣으면 됨. 
ThemeContext는 클라이언트 컴포넌트임.
클라이언트 컴포넌트(e.g.,ThemeContext) 를 서버컴포넌트(e.g., children)의 props으로 감싸면 클라이언트 컴포넌트도 서버컴포넌트처럼 동작한다.


## Client Component
### moving Client Component

최대한 넥스트의 장점을 살리기위해 
헤더는 다 서버컴포넌트로 만들고,
serachBar만 클라이언트 컴포넌트로 만들어라.

### passing props from server to Client Components


## Interleaving Server and Client Components
서버에서 클라이언트 렌더링되는 방법 다시 설명
클라이언트 컴포넌트에서는 서버컴포넌트 못 가져옴.


# 4. 부분 사전렌더링 (Partial Prerendering)
## 이 페이지에서는 (On this page)
부분 사전 렌더링은 canary에서만 사용할 수있는 실험적 기능이며 변경될 수 있습니다.

다이나믹 영역이 있으면 원래 서버컴포넌트에서 전체가 동적렌더링을 함.
그림에 나온 영역이 정적렌더링을 한다는거임.
-> 원래는 해당영역 전체를 사용자가 요청했을 때마다 렌더링할 페이지를 만드는데,
이걸 정적렌더링을 통해서 "빌드할 때 다이나믹 영역만 구멍뚫어서 HTML을 만들어 두고" 사용자 요청이오면 렌더링페이지를 만들지 않고 클라이언트한테 바로 보낸다는거임
이게 새로운 신세계인 이유는 원래는 요청 올 때마다 렌더링할 페이지를 만들어 보냈음


Partial Prerendering (PPR) 같은 경로에 정적컴포넌트, 동적 컴포넌트 결합할 수 있게 해줍니다.


빌드를 하면서 NextJS는 가능한 많은 라우트(경로)들을 프리렌더링합니다.
요청이 들어올 때마다 렌더링이 필요한 동적 렌더링 코드가 감지되면, 
관련 컴포넌트를 Suspense 경계로 감쌀 수 있습니다.   

여기서 공홈에서 추천하는 [유튜브 봄](https://www.youtube.com/watch?v=MTcPrTIBkpA)   
블로그같은건 정적렌더링, 개인화된 데이터는 동적렌더링인게 사전지식.     
next에서는 바운더리라는 개념으로 정적과 동적을 비율을 조정할 수 있음.   
만약에 홈에서 로그인된 회원 정보만 개인화된 정보라면 정적비율을 높이고, 동적 비율을 낮추는게 좋겠죠.   
(개인화된 정보를 가져오기 위해서는 동적 API를 사용해야함)
원래라면 동적 API가 발견되면 넥스트는 전체경로를 동적렌더링으로 변경합니다.
이거를 바운더리를 통해서 이영역만 동적렌더링으로 나머지는 정적렌더링으로 수행하려는것.

e.g., 에러바운더리를 쓰면 에러바운더리 안에있는 페이지는 에러나면 에러컴포넌트를 보게됨
e.g., 서스펜스 바운더리를 쓰면 로딩되는동안은 로딩컴포넌트를 보게됨
```
<ErrorBoundary fallback={<Error / > } >
  <pages />
</ErrorBundary>

<Suspense fallback={<Loading />} >
 <pages />
</Suspense>
```

에러바운더리, 로딩서스펜스 처럼 
```
// 유저부분 때문에 전체가 동적렌더링
<Page>
  <Header/>
  <Product />
  <User />
</Page>

// 유저 부분을 런타임 서스펜스바운더리로 감쌈 -> 해당부분만 동적, 나머지는 전체페이지는 정적
// 넥스트가 가능한 많은 경로를 정적 렌더링으로(=프리렌더링) 함
// 동적 컴포넌트를 만든다는것.
<Page>
  <Header/>
  <Product />
  <Suspense fallback={<userSkleton />}>
    <User />
  </Suspense >
</Page>
```

정적 렌더링 (Static Rendering):   
빌드 시점에 HTML 파일을 미리 생성하여, 사용자가 페이지를 요청할 때 이미 완성된 HTML을 제공하는 방식

프리렌더링 (Pre-rendering):
클라이언트가 페이지를 요청하기 전에, 서버에서 HTML을 미리 생성해두는 모든 방식을 포괄하는 개념
프리렌더링에는 정적 렌더링이 포함됨

## 배경 (Background)
PPR(Partial PreRendering)을 사용하면 Next.js 서버가 사전 렌더링된 콘텐츠를 클라이언트로 즉시 보낼 수 있습니다.
부분 프리렌더링이라도 즉시보내면 사용자는 HTML을 바로 볼 수 있음(=폭포수처럼 다운로드+파싱+데이터패칭까지 완료해야만 화면보이는것과 다르게)

페이지에 동적으로 업데이트되어야 하는 여러 요소가 있을 때, 각각의 요소마다 별도로 HTTP 요청을 보내면 매번 서버와 통신해야 하므로 네트워크 왕복 횟수가 늘어나고 성능이 떨어질 수 있습니다.

PPR(Partial Pre-rendering)은 이러한 문제를 해결하기 위해, 미리 렌더링된 정적인 부분과 동적으로 업데이트되어야 하는 부분을 한 번의 HTTP 요청으로 묶어서 클라이언트에 전달합니다.
즉, 각 동적 구성 요소마다 따로 요청하는 대신 한 번에 모두 보내기 때문에, 여러 번의 네트워크 통신을 줄여서 더 빠르고 효율적인 페이지 로딩이 가능해집니다.

## 부분 프리렌더링 사용 (Using Partial Prerendering)
넥스트 15에서 사용가능

### 점진적 도입 (버전 15 캐너리 버전) (Incremental Adoption (Version 15 Canary Versions))
```
pnpm install next@canary

```

```ts
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  experimental: {
    ppr: 'incremental',
  },
}
 
export default nextConfig
```

```tsx
import { Suspense } from 'react'
import { StaticComponent, DynamicComponent, Fallback } from '@/app/ui'
 
export const experimental_ppr = true
 
export default function Page() {
  return (
    <>
      <StaticComponent />
      <Suspense fallback={<Fallback />}>
        <DynamicComponent />
      </Suspense>
    </>
  )
}
```
## 동적 컴포넌트 (Dynamic Components)
동적 렌더링만 있던개념인데, 동적 컴포넌트를 만든다는것.


---------------------번외 스터디하다가 내용--------------------
fetchData()

S
  data = await fetchData();
  component가 돌아서 HTML response
  
C (props) => {  // 언제 쓸 지 모르기 때문에 context로 쓰는게 맞음
  button
  data <- state
  목록리스트 client component;
  fetchData();

  1. const [data, setData] = useState(props); -> 이거임 

  2. const [data, setData] = useState([]); -> 이거아님
   
  useEffect(() => {
    data = await fetchData();
    setData(data);
  }, []);

  
  3. //어디서든 키로 가지고 올 수 있음
  react-query는 dehydrate 할 수 있음(서버에서 패치한 캐시데이터를 가지고올 수 있음)
  useEffecy(() => fetchData(), []);
}


서버 컴포넌트는 rsc로 돌아서 무조건 html로 내려감
ssr은 클라이언트 컴포넌트가 돌아감. 렌더링은 하지만, 자바스크립트 번들까지 내려감.


-----------------------------------------------------------


# 5. 런타임 (Runtimes)

## 
