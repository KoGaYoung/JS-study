~~~
DOM에서 이벤트가 발생하면, 자바스크립트는 DOM 트리를 타고 올라가며 이벤트 전파 매커니즘을 전달해요. (Event Propagation)

이벤트 전파는 캡처링 -> 타겟팅 -> 버블링으로 이뤄지는데, 
개발자는 단계별로 이벤트를 제어할 수 있게됩니다. 
(스코프 체인과는 별개입니다, DOM을 타고가는거에요)
~~~

1. 캡처링과 버블링
 ![image](https://github.com/KoGaYoung/JS-study/assets/36693355/899a4663-db70-4ecc-912d-d53c5c3c88fb)

~~~
캡처링: Window -> Document -> Html -> Body -> Table -> Tbody -> Tr -> Td 순으로 이벤트 헨들러가 있는지 검사하고 호출함
타겟팅: 이벤트가 여기서 실행됨
버블링: 타겟팅 된 요소에서 부터 부모 이벤트 헨들러가 있는지 검사하고 실행함
~~~

~~~
버블링, 캡처링 같은 경우 돔 요소에 디버깅을 찍고 눈으로 바로 확인이 가능하다.
(php기반 프로젝트 진행할 때 vanilla.JS로 개발할 때 버블링을 기능상 이유로 몇번 막았디)

실행결과: 부모캡처 -> 자식캡처 -> 자식버블 -> 부모버블
~~~
![image](https://github.com/KoGaYoung/JS-study/assets/36693355/4fc066af-77a3-447d-b4ed-fa36fc2c7cde)

# 버블링을 막는 방법
~~~
event.stopPropagation()
현재 이벤트가 상위로 전달되는걸 막음. 같은 돔 요소에 다른 이벤트 리스너들은 못 막음 

event.stopImmediatePropagation()
현재 요소의 모든 이벤트들을 중지함.

// func1 먼저 실행되면 아래 func2는 실행됨
var func1 = window.addEventListenter('click', function(event) {
  event.stopPropagation();
})

// func2 먼저 실행되면 아래 func2는 실행안됨
var func2 = window.addEventListenter('click', function(event) {
  event.stopImmediatePropagation();
})
~~~

## 델리게이션
~~~
. 이벤트 델리게이션 (Event Delegation)
개념: 이벤트 델리게이션은 단일 이벤트 리스너를 부모 요소에 할당하여, 그 부모의 모든 자식 요소들의 이벤트를 처리하는 기법입니다.
장점: 이 방법은 메모리 사용량을 줄이고, 동적으로 요소가 추가/삭제되는 경우에도 이벤트 핸들링을 유지할 수 있게 합니다.
예시: 예를 들어, 리스트에 있는 모든 항목에 대해 클릭 이벤트를 처리하고 싶다면, 각 항목에 개별 이벤트 리스너를 추가하는 대신, 리스트 자체에 하나의 이벤트 리스너를 추가하여 이벤트를 처리할 수 있습니다.

~~~
