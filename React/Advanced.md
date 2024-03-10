~~~
4-1. Ref로 돔 참조하기
4-2. Ref로 값 조작하기
4-2. Effect로 동기화하기
4-3. Effect가 필요하지 않은 경우
4-4. React Effect의 생명주기
4-5. Effect에서 이벤트 분리하기
4-6. Effect 의존성 제거하기
4-7. 커스텀 Hook으로 로직 재사용하기
~~~

# Ref로 돔 참조하기
~~~
생명주기에 걸쳐 값을 유지하지만, 렌더링은 안 일어나게 하고싶을 때는 ref를 사용합니다.
useState와 useRef는 생명주기와 별개로 관리할 수 있는 값 입니다.
렌더링여부에 따라 useState, useRef를 구분해서 사용합니다.
~~~

~~~js
// 이런식으로 사용합니다.
const variable = useRef(0);

variable.current = 10;
~~~

~~~
current 프로퍼티를 통해 수정이 가능합니다.
current는 이 식별자(여기서는 변수)가 가리키는 값을 참조하고있습니다.
참조를 통해 리액트 생명주기에서 벗어나서 관리가 가능합니다.

공식홈페이지 예제에서는 스탑워치(setInterval 을 사용)를 예시로들어
Stop을 구현하기위해 해당 이벤트 timeout ID를 저장할 변수가 필요하다는 예시를 들고있습니다.
~~~

|useRef|useState|
|---|:---|
|state를 바꿔도 리렌더 되지 않습니다.|state를 바꾸면 리렌더 됩니다.|
|Mutable-렌더링 프로세스 외부에서 current 값을 수정 및 업데이트|”Immutable”—state 를 수정하기 위해서는 state 설정 함수를 반드시 사용하여 리렌더 대기열|
|렌더링 중에는 current 값을 읽거나 쓰면 안 됩니다.|언제든지 읽고쓸 수 있지만 snapshot이 존재합니다.|

~~~
위 예제처럼 아래와 같은 상황에서 ref를 사용합니다.
Timeout id를 저장해야할 때
dom 엘리먼트 저장과 조작 (provider 처럼 사용)
JSX를 계산하는 데 필요하지 않은 다른 객체 저장 (저는 이경우에 가장 많이 사용합니다.)
~~~

~~~js
// ref의 동작은 이렇게 되어있다는데. 실제로 이렇게 되어있진 않고 단순화 된것임
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
~~~

# Ref로 돔 조작하기
~~~
react는 대부분의 경우 state의 변경으로 돔을 리액트에서 업데이트해주기 때문에
돔을 직접 조작할 일이 별로 없습니다.

바닐라js를 사용하면 돔 조작할일이 굉장히 많습니다.
그래서, Ref로 돔 조작하는 방법은 바닐라로 조작하는 방법과 아주 조금 다릅니다.
~~~

~~~jsx
// undefined 는 값이 할당되기 전에 쓰이는 값으로 
// 여기에서는 의도적으로 값이 없음을 나타내기위해 null을 쓰는것이 바람직합니다.
const myRef = useRef(null); 

<div ref={myRef}>

// 예를 들어 이렇게 브라우저 API를 사용할 수 있습니다
inputRef.current.focus();
~~~
~~~
공식문서 예제 중 scrollIntoView를 알게되었습니다.
항상 scrollTo만 사용했는데 스크롤 이벤트 기능에 활용할 수 있을까 싶어
두개를 비교해봤습니다.
~~~

| scrollTo                         | scrollIntoView를                                                                                                                                                        |
|----------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 절대값으로 x,y 스크롤 위치를 해당 좌표로 이동시킵니다. | 해당 dom 요소가 사용자 화면(뷰포인트)상에 위치하도록 부모 컨테이너를 이동시킵니다.                                                                                                                       |
| 이동하려는 위치값을 정확히 알아야합니다(계산해야합니다)   | 상대적 위치를 기반으로 스크롤합니다. 가운데로 이동시키는건 아니고 viewPort 안에 이미 완전히 존재한다면 발생하지 않을 수 있습니다.  block: 요소가 뷰포트에 어떻게 위치할지 결정합니다. block: "start", "center", "end", "nearest" 값을 가질 수 있습니다.|

~~~
// ref로 리스트 dom 요소 가져오기
useRef를 여러번 사용해야하는 경우라면 render()함수 아래서 map으로 반복문 돌리지말고,
부모 요소에 ref를 넣어 가져온 후, 
querySelectorAll과 같은 DOM 조작 메서드로 돔 요소들을 추출해서 사용하는 것을 권장합니다.


~~~
~~~JSX
// 하지만 리액트가 관리하는 (state로 연결된) 돔 노드는 ref로 변경하면 오류가 발생합니다.
 <div>
      {/* 2번 */}
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Toggle with setState
      </button>
      {/* 1번 */}
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
~~~

~~~
다른 컴포넌트의 ref는 가져다 쓸 수 없습니다.
~~~
~~~jsx
// before
function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {...}

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
~~~
~~~js
// after
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {...}

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
~~~
~~~
forwardRef로 다른 컴포넌트에 ref 값을 직접 넘겨본 경험이 있습니다.
가독성이그렇게 좋진 않았습니다. 그래서 꼭 필요할때만 쓰자고 생각했습니다.

useImperativeHandle 로 forwardRef 에서 일부만 속성을 노출할 수 있도록 설정할 수 있습니다.
~~~
~~~
리액트의 모든 갱신은 돔을 불러오는 1.렌더링단계, 변경사항을 돔에 반영하는 2. 커밋단계로 나눌 수 있습니다.

ref로 돔을 조작할 때, 리액트는 처음에 렌더링 전이므로 ref변수에 null을 부여하고
커밋 단게에서 돔 요소를 ref 변수에 할당합니다.
주로 이벤트 헨들러에서 ref를 조작합니다.
~~~
~~~
flushSync로 state 변경을 동적으로 플러시하기

(처음 보는 기능이라, 그대로 옮깁니다)


submit()을 하면, 이벤트 핸들러를 통해 state리스트에 추가하고, scrolltoView()를 통해 스크롤을 이동시킵니다.
// 원하는대로 동작하지 않음
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();

그런데, setState는 비동기적으로 일어나므로, 스크롤 이동이 한 아이템 적게 일어납니다.

setState를 flushSync로 감싸면 동기적으로 setStaet를 실행할 수 있게됩니다.
// 원하는대로 동작함
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
~~~
---

# Effect로 동기화하기
~~~
컴포넌트 내부의 로직은 2가지로 나눌 수 있습니다.
1. 렌더링코드 : JSX로 이루어져 순수한 HTML 표현 로직이여야합니다.
2. 이벤트헨들러: 사용자 동작으로 이루어진 이벤트, 혹은 부수효과를 말합니다.

단, 사용자 동작과 관련없는 이벤트들이 있습니다.
특정 이벤트가 아닌 렌더링에 의해 직접 발생하는 이벤트입니다.
e.g., 화면이 나타나면 채팅창을 띄워야하는 경우가 있습니다.

렌더링 변경 시점에서는 커밋 이후가 됩니다. 이를 Effect라고합니다.
이 시점에 데이터를 패칭해오기 유용해보입니다.
~~~

~~~
리액트가 필요 없는 케이스는 다음 장에서 학습합니다.
무작정 useEffect를 추가하지 말라는 의미입니다.
~~~

~~~
useEffect를 정의하는 방법은 아래 단계를 따릅니다.
1. Effect선언 : 기본적으로 useEffect는 렌더링 이후에 실행됩니다.
2. 의존성 지정 : 모든 렌더링마다 useEffect가 발생하는 것이 아닌,
 특정 사용자 이벤트에 의해 발생할 수 있도록 의존성을 지정합니다.
3. 필요한 경우 클린업: removeEventListener, off, fetch 취소(혹은 무시) 등이 있습니다.
~~~

~~~js
// 그냥 ref를 사용한다면, 위의 ref 잘못된 사용방법처럼
// 렌더링 중에 ref dom요소를 수정하기 때문에 우리는 돔 요소 제어에 어려움을 겪을 것 입니다.
// 혹은 ref 자체가 의미 없을지도요.
// 이를 useEffect 함수로 감싸준다면, 렌더링 이후에 ref로 돔 요소를 제어하기 때문에 오류를 피해갈 수 있습니다.
~~~

~~~
의존성 배열에는 여러가지 조건이 들어갈 수 있습니다.
(useEffect 내부에서 사용하는 변수를 의존성 배열에 넣지 않으면 린트 오류가 발생합니다.
린트를 잘 써야겠다는 생각이 또 드는 부분입니다)

일단 의존성 배열에 조건이 한개라도 변경되면 useEffect가 실행됩니다.
(반대로 말하면 의존성 배열 조건이 모두 다 변경되지 않아야만 useEffect를 다시 실행하지 않습니다)
만약 해당 조건에 변경되고 싶지 않다면,
useEffect 내부로직을 수정하여 해당 변수를 사용하지 않도록 조정하거나,
조건을 비교해서 분기처리를 통해 해당 로직을 by pass하도록 만들어줘야합니다.
~~~

~~~js
// 공식문서가 요약을 참 잘해놨습니다.

useEffect(() => {
  // 모든 렌더링 후에 실행됩니다
});

useEffect(() => {
  // 마운트될 때만 실행됩니다 (컴포넌트가 나타날 때)
}, []);

useEffect(() => {
 // 마운트될 때 실행되며, *또한* 렌더링 이후에 a 또는 b 중 하나라도 변경된 경우에도 실행됩니다
}, [a, b]);
~~~

~~~
* 중요
왜 useEffect에서 ref변수는 의존성배열에 넣지 않아도되나요?
일단 넣지 않아도됩니다. 이유는 ref변수가 '안정된 식별성(stable identity)'을 가지기 때문입니다.
리액트 생명주기에서 벗어났다고도 보는데, 언제나 참조하면 같은 객체를 반환하기 때문과 같은 말입니다.

추가로 포함해도 문제없습니다. (만 안그래도 복잡해질 수 있는 조건에 가독성을 해칠 수 있기에 비선호합니다)
~~~

~~~

~~~
~~~
그리고 이벤트리스너나, 소켓의 경우 잘 취소해줘야합니다.
return구문 안에서 취소하면됩니다.

이전 회사에서 가끔 메모리릭(으로 인한 백화현상)이 나는 경우가 발생했는데
나중에 알게되었지만, 이펙트 밖에서 선언된 함수인데 전역스토어 의존성 주입하는 부분의 데이터가 의존성배열에 빠져있어서, 
클린업이 제대로 안되서 메모리릭이 발생했습니다.
(밖에서 선언된 함수를 Effect안으로 넣거나, 의존성배열에 잘 추가하면됩니다)

클린업을 잘 해줘야합니다!
~~~
~~~js
번외로 클린업은 Ref 변수들도 클린업을 잘 해줘야합니다. (초기값으로)
이유는 성능때문입니다. 

interface userList {
 // 매우 복잡한 타입
}

const data = useRef<null | userList>(null);

useEffect(() => { 
   // 매우복잡한 로직

   // 여기가 초기화!
   () => data.current = null;
, []};

직접적으로 공식문서에서 ref 변수를 초기화함으로써 GC를 일어나게한다는 말은 없지만.(메모리 할당 해제)
하지만, 불필요한 참조를 제거함으로써 GC가 수행될 가능성을 높일 수 있습니다.
~~~
### api 더블클릭 방지 (2번 호출 방지)
~~~ js
// post 요청은 가끔 보안, 기능상의 이유로 2번 신청을 프론트에서 1차적으로 걸러줘야합니다. (완벽하게 하려면 BE에서도 중복신청 방지가 들어가야겠지만요.)
const isDoubleClick = useRef(false);

const handleClick = () => {
  // 실행중이면 리턴
  if (isDoubleClick.current) {
    return;
  }
  
  isDoubleClick.current = true;

  fetch('api/url')
    .then(() => {
      // 로직들..
    })
    .finally(() => {
      // 여기서 해제
      isDoubleClick.current = false;
    });
}

<button onClick={handleClick}>클릭!</button>

~~~

~~~
effect 안에서 데이터 패칭하는것에 대해 4가지 불편한 점이 있다고 공식문서에서 설명합니다.
ssr(html을 서버에서 그림 + 클라이언트에서 js다운받아서 데이터 로드)
csr(빈 html + js + css 클라이언트가 가져와서 html 그리고 데이터도 로드)

1. ssr(서버에서 html 내려주고 사용자가 js파일 다운로드해서 데이터를 로드해야함)에서 effect는 동작하지않습니다.
2. 네트워크 폭포를 만듭니다 : 부모 -> 자식 으로 렌더링되면서 부모가 렌더링이 일어나야 자식컴포넌트 시작되면서 그리는것 + 패칭 병렬로 할 수 있는데 무조건 탑다운으로 진행하게됨 (ssr에서는 미리 html그리기에 일부 해소됨)
3. 매번 해당 컴포넌트를 패칭해야함: 컴포넌트가 언마운트되면 항상 다시그려야함. (특히 csr에서 추가적인 혹은 불필요한 패칭 발생)

이런 경우 해결방법으로 패칭 매커니즘을 제공하는 프로덕션 수준 프레임워크(개츠비, 넥스트, 리믹스 등)를 쓰거나
React query, swr, react router +6.4를 적용을 고려할 수 있습니다.
Effect를 내부적으로 사용하면서 요청 중복을 제거하고 
응답을 캐시하고 네트워크 폭포를 피하는 로직을 추가할 것입니다. 
(데이터를 사전에 로드하거나 데이터 요구 사항을 라우트)
~~~

~~~js
// 컴포넌트 외부에 쓸 경우, 리액트 컴포넌트 생명주기에 들어가지 않고
// 애플리케이션 시작 시(새로고침 누르지 않는한) 1번만 실행됨을 보장합니다. (몰랐습니다..)

if (typeof window !== 'undefined') { // 브라우저에서 실행 중인지 확인합니다.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
~~~

# Effect가 필요하지 않은 경우
~~~
~~~

# React Effect의 생명주기
# Effect에서 이벤트 분리하기
# Effect 의존성 제거하기
# 커스텀 Hook으로 로직 재사용하기
