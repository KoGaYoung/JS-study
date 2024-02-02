[2. 상호작용성 더하기]
~~~
2-1. 이벤트에 응답하기
2-2. State: 컴포넌트의 기억 저장소
2-3. 렌더링 그리고 컴포넌트
2-4. 소스코드로서의 state
2-5. State 업데이트 규칙
2-6. 겉값 state 업데이트하기
2-7. 바뀐 state 업데이트하기
~~~

## 2-1. 이벤트에 응답하기
~~~
이벤트 헨들러

특징)
1. 주로 컴포넌트 내부에 정의됨
2. handle 이란 이름 prefix가 붙음.
3. 즉시실행함수로 선언하지 않도록 주의

랜더 함수 안에서 인라인으로 사용되는 경우가 많은데,
개인적으로는 랜더함수 안에 로직이 있는 것을 선호하지 않음. (가독성 떨어짐)
또한 코드 재사용성과 최적화 측면에서도 함수는 랜더함수 바깥으로 올리는 것을 선호함

~~~

~~~jsx
/* html을 마지막 인자로 받음 */
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
      {/* onClick 이벤트와 html 전달 */}
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}
~~~

### 이벤트헨들러 네이밍 prefix
~~~
생각해보면 필자는 컨벤션에 맞춰 적당히 명명을 해왔던 것 같다.
이번 기회에 prefix 규칙을 알고 정확히 사용하자.

기본적으로는 컴포넌트 내부에서 사용되는 이벤트헨들러에 handle 접두사를 붙이는게 맞다.
(3음절 안넘어가게 의미를 잘 부여하자.)

on 접두사는 외부(부모) 컴포넌트로부터 받은 props 이벤트헨들러에 사용된다.
(하지만 기본적으로 사용되는 이름으로 사용하지 말자. 아래와같은 상황이 발생된다.)
~~~

~~~jsx

const ChildComponent = (props) => {
// const ChildComponent = ({onClick, ...props}) => { 이렇게 비구조할당해서 예방도 가능
    return (
        <div>
            {/* props에 onClick 기본이름으로 명명된 이벤트가 있어 커스텀 버튼에서도 그대로 onClick으로 사용될 여지 있음 */}
            <CustomButton {...props} />

            <button onClick={props.onClick} />
        </div>
    )
}


~~~

## 이벤트 전파
~~~
onScroll을 제외한 React 내의 모든 이벤트는 전파된다.

e.stopPropagation()으로 버블링을 막아줄 수 있다.

stopPropagation로 이벤트를 중단시켜도 로그를 확인하기위해 부모쪽에 capture를 추가해주면 모든 이벤트를 추적할 수 있다.
(나중을 위해 알아두면 좋을 것 같다)

e.preventDefault()는 브라우저 기본동작을 막아준다. 주로 폼태그 내부 버튼 클릭이벤트에서 전송을 막기위해 사용된다/
~~~

## 2-2. State: 컴포넌트의 기억 저장소

~~~
State는 리액트의 라이프사이클과 같이 움직이는 화면에 움직이는 값 입니다.
16.3 이전 버전에서는 생성자 함수를 통해 생성 후 setState를 통해 업데이트. 
16.3 이후 버전에서는 훅을 통해 useState로 업데이트합니다.

동일 컴포넌트를 여러번 호출하면, 같은 이름의 State라도 전혀 관련없이 동작합니다.
리액트는 어떤 State가 변경되었는지 어떻게 알 수 있을까요?
-> 호출되는 순서는 항상 동일하기에, 호출 순서에 의지한다.
또한 리엔트 엔진은 컴포넌트마다 state 배열 카운터를 가지고({0: 0, 1: ƒ setState(nextState)}) 스테이트가 변경될 때마다 카운트를 올린다!
(자세한 내용은 여기에: https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)
~~~

## 2-3. 렌더링 그리고 컴포넌트

~~~
1. 렌더링 트리거 (초기렌더링, state 변경 시 발생)
2. 렌더링: 초기에는 Root 컴포넌트 호출, 이후에는 state 변경으로 트리거를 만들어낸 컴포넌트를 호출
3. 리액트가 DOM을 변경사항에 커밋: (appendChild(), 이후에는 변경사항만 커밋)
~~~

## 컴포넌트 최적화
~~~
만약 부모컴포넌트의 state 변경사항이 생긴다면, 
트리거가 발생하기 때문에 render 함수가 실행되고 자식컴포넌트들은 다시 호출된다.(렌더링 된다)
그렇다면 부모 컴포넌트가 렌더링되면 자식컴포넌트는 "반드시" 렌더링 된다고 할 수 있을까?

결론부터 말하면 부모컴포넌트에서 자식으로 넘기는 props가 변경되지 않거나,
memo로 감싸진경우 리액트가 어느정도 불필요한 렌더링을 방지하여 최적화를 해준다.
(성급한 최적화는 지양해야한다, 필요할 때 적절히 하는게 중요하다)
~~~

[링크가 잘못되어있어 제보도 하나 해봤다](https://github.com/reactjs/react.dev/issues/6603)

