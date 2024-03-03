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
# Effect로 동기화하기
# Effect가 필요하지 않은 경우
# React Effect의 생명주기
# Effect에서 이벤트 분리하기
# Effect 의존성 제거하기
# 커스텀 Hook으로 로직 재사용하기