# 자바스크립트의 동시성
~~~
자바스크립트는 싱글스레드이다.
자바스크립트에서 동시성(Concurrency)는 싱글스레드인 자바스크립트가 비동기 작업들을 어떻게 처리하는지를 말한다.
비동기작업들은 콜 스택, 이벤트 루프, 콜백 큐, 그리고 웹 API와 같은 요소들의 상호작용을 통해 이뤄진다.
~~~

# 자바스크립트의 호출스택(Call stack)
~~~
자바스크립트 엔진은 메모리힙,콜스택 2가지로 이루어져 있습니다.
메모리힙: 변수와 객체에 대한 메모리 할당을 하는 곳. 
콜스택: 코드가 실행될 때 여기에 호출 스택이 쌓이는 곳.
~~~
<img width="323" alt="스크린샷 2021-10-31 오후 11 26 29" src="https://user-images.githubusercontent.com/36693355/139588312-a42b7209-196e-4682-890a-1d6e4439782f.png">

# 콜스택 동작 이해하기
~~~javascript
// 자바스크립트는 단일 스레드언어로 단일 호출 스택을 가진다. 
// 아래와 같이 함수를 실행시키면 실행 컨택스트를 콜 스택에 쌓고(push) 결과를 반환하면 제거(pop)합니다.(LIFO, Last in first out)
function multiply(x, y) {
    return x * y;
}
function printSquare(x) {
    var s = multiply(x, x);
    console.log(s);
}
printSquare(5);
~~~
<img width="748" alt="스크린샷 2021-10-31 오후 11 43 47" src="https://user-images.githubusercontent.com/36693355/139588999-d2c444a2-44aa-4369-bf27-8322d5f68588.png">

~~~
printSquare의 실행컨택스트를 콜스택에 담는다.
multiply의 실행컨택스트를 콜스택에 담는다.
multiply의 리턴 25를 받아서 콜스택에서 제거한다.
console.log(s) 실행 컨택스트를 행시킨다. 콜스택에서 제거한다.
printSquare를 콜스택에서 제거한다.

따라서 어떤 실행 컨텍스트가 콜 스택의 맨 위에 쌓이는 순간이 곧 현재 실행할 코드에 관여하게 되는 시점이다.
~~~


# 자바스크립트는 엔진 딸랑 하나가 끝인가?
<img width="1095" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/9d90ef72-88a2-422e-8ce8-a3f3ce8ca42b"  >

~~~
웹은 단순히 JS 엔진으로만 구성되어 있지 않다.
자바스크립트 내부 비동기 함수들이 존재하는데(ex.dom, setTimeout) 이는 WEB API라고 한다.
이 웹 api에 요청이 들어오면 요청에대한 콜백함수를 콜백 큐(=테스크 큐)에 쌓는다.
그래서 콜스택이 비어있으면 콜백큐에서 콜스택으로 콜백함수를 하나씩 넣어준다.

정리하면
콜스택: 처리할 소스와 함수를 하나하나 순차적으로 스택에 담아서 처리하는곳
웹API: 자바스크립트에서 지원하는 비동기 함수들 DOM AJAX Timeout
콜백큐: Web API에서 받은 콜백함수를 저장하는 곳
이벤트루프: if (callstack is null) callbackQueue.pop() and callstack.push(callbackFunction)
~~~



# 이벤트 루프가 하는일을 좀 더 알아보자!
<img width="584" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/5b1601dc-e885-4747-8ad4-d643b352e378" >

~~~ 
위에서 말했던 콜백 큐는 다양한 Web API 작업을 효율적으로 처리하기 위해 큐를 3가지로 나눠서 처리합니다.
1. (매크로)테스크 큐: setTimeout, setInterval, I/O등 처리,
2. 마이크로테스트 큐: Promise, MutationObserver등 처리 
3. 애니메이션 프레임: requestAnimation 처리


이벤트 루프는 콜스택이 비면, (매크로)테스크 큐의 작업을 하나 가져와서 실행, 그 다음 마이크로테스트 큐의 모든 작업을 실행, 다시 (매크로)테스크 하나 가져옴
이를 계속 반복한다!
(promise가 너무오래걸리면 animation frame이 늦게 실행되면서 랜드블락이 걸리기 때문에 너무 오래걸리는걸 잡아두면 안된다)
~~~

~~~javascript
console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(function() {
  console.log("promise1");
}).then(function() {
  console.log("promise2");
});

requestAnimationFrame(function {
    console.log("requestAnimationFrame");
})

console.log("script end");
~~~
~~~
1. script start 콜스택에서 실행
2. setTimeout이 실행되는데 콜백은 (매크로)테스크 큐에 넣음 (다음 매크로테스크가됨)
3. Promise 실행되고 then 콜백 마이크로 테스크 큐에 넣음
4. requestAnimationFrame도 애니메이션 프레임에 넣음
5. script end 콜스택에서 실행됨
6. 마이크로테스크 다 비움 promise1, promise2 실행됨
7. setTimeout 실행됨
8. requestAnimationFrame 실행됨.
~~~

## 간단하게 확인할 수 있는 클로저 예제
### (사실 setState의 비동기 벌크 동작방식 보다가 클로저 예제가 보였다)
[state](https://ko.react.dev/learn/queueing-a-series-of-state-updates)
~~~javascript
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
~~~
~~~
모든 setNumber 안의 number 변수가 상위스코프의 useState(0)을 바라보기때문에 +3버튼을 누르더라도 화면에 Number는 1로 동작한다
~~~

