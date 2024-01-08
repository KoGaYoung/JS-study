# 리액트 버전별 차이 
~~~
2019년 리액트를 처음 접 했을때 15버전이였다. 그 뒤로 18버전까지 릴리즈되었고, 개발에 많은 변화가 생겼다.
앞으로도 최신버전을 잘 따라가보자!
~~~

## v15 
~~~
~~~



# 리액트 복습좀하자
~~~
1. 리액트 Virtual dom 을 사용하는 이유
-> 결론부터 말하면 Virtual Dom은 변경사항을 하나의 가상 돔에 모았다가 DOM에 한 번에 보내는 기술이다.

먼저 Virtual Dom은 화면을 구성하는 dom tree의 (가벼운) 복사본이다.
가벼운 복사본에 변경사항들을 기록하고, 화면에 최신 변경사항들을 모아서 표시한다. (이 문장이 재조정)
변경사항들도 virtual dom을 통해서 diff 알고리즘으로 변경된 부분만 감지해서 최소한의 Dom을 업데이트한다.

1-1. 전통적인 렌더링 방식과 다르게 react에서 virtual dom을 사용해서 얻을 수 있는 이점은? 

전통적인 렌더링 방식에서는 dom tree(초기 로딩시만) + css tree (초기 로딩시만) = Render tree로 합치기(attechment)하고
그뒤에 layout/reflow(배치) -> repaint(그리기) 과정을 거친다.

돔을 조작하면 render tree가 변경되거나, 스타일변경에 따라 repaint만 일어날 수 있다.
그런데 만약 dom을 직접 조작하는 경우 30번의 조작에 대해 30번의 배치와 그리기가 일어날 수 있다.
하지만 virtual dom을 사용할 경우 마지막 최신트리만 화면에 그리기 때문에 배치와 그리기에서 이점을 얻을 수 있다.
(리액트가 항상 빠른건 아니다. 하지만 돔 조작이 많은 spa 또는) 대규모 프로젝트일수록 유리하다!

16 버전에는 fiber라는 재조정 엔진 등장
(virtual dom 공식문서: https://ko.legacy.reactjs.org/docs/faq-internals.html#what-is-the-virtual-dom)
~~~

~~~
2. 화면은 언제 리랜더링(re-rendering)되나요?
-> 화면은 현재 컴포넌트의 state가 변경되거나 부모로부터 받은 props가 전달되었을 때 발생한다.

2-1. 만약 최상단 컴포넌트가 변경되면 모든 컴포넌트가 렌더링되나요?
-> 아니요
(보통 자식컴포넌트에게 props를 전달하기 때문에 부모가 바뀌면 자식이 바뀐다고 오해할 수 있다)
이유는 최상단의 컴포넌트에 props에 영향을 받는 자식컴포넌트는 리랜더링 되겠지만 diff 알고리즘을 통해 변경이 있는 부분만 다시 그린다.
이외에도 pure component로 구현된 경우, shouldComponentUpdate메소드로 제어할경우, React.memo로 최적화 한경우에는 다시 안그립니다
~~~

~~~javascript
3. setState는 동기로 동작할까요 비동기로 동작할까요
-> 비동기로 동작해요

3-1. 비동기로 동작하는 이유는?
-> 한번에 배치처리해서 setState 발생횟수 줄어듬 -> 화면 리렌더링 줄어듬 -> 성능향상

3-2. 비동기로 동작하는 스코프는?
-> 이벤트 헨들러, 라이프사이클 내에서는 한번에 모아서 배치처리한다.
setTimeout, async/awiat에서는 배치처리 하지않는다.

React 18버전 이후에는 자동으로 배치처리된다
ex.
// 18 이전버전
handleClick = () => {
  this.setState({count += 1});
  this.setState({count += 1});
  // 배치처리되어 실제로 count 1만 증가됨
}

// 18 이후버전
handleClick = () => {
  setCount(c => c+1 );
  setCount(c => c+1 );

  // 자동 배치처리되어 2만큼 증가됨
}

3-3. 비동기지만, 동기적으로 사용하고 싶을때가 있음. 어떻게할래?
// 1. callback 사용
this.setState({state: newState}, () => {
  // setState 변경 이후에 돌아갈 로직
}

// 2. lifecycle 사용
// 16.8 이전
componentDidUpdate(prevProps, prevState) {
  if (prevState.value !== this.state.value) {
    // setState 변경 이후에 돌아갈 로직
  }
}

// 16.8 이후 hook
const [value, setValue] = useState(initialValue);

// 이전 상태값을 추적하기 위한 ref
const prevValueRef = useRef();
  
useEffect(() => {
  // componentDidMount와 componentDidUpdate에 해당
  if (prevValueRef.current !== value) {
    // setState 변경 이후에 돌아갈 로직
  }

  // 현재 값을 이전 값으로 업데이트
  prevValueRef.current = value;
}, [value]); // value가 변경될 때마다 실행

// 3. async, await 과 같이 사용
stateAsyncChange(state) {
  // setState는 비동기라 Promise를 반환하는건 아니고 Promise를 통해 setState가 실행되기까지 await할수있음
  return new Promise((resolve) => {
    this.setState(state);
  }
} 

async function1 () {
  awiat this.stateAsyncChange({value: newValue});
  // setState 변경 이후에 돌아갈 로직
}
~~~
