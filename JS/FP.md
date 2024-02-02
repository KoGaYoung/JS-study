![image](https://github.com/KoGaYoung/JS-study/assets/36693355/01d12adf-3769-43ca-a893-47289d210993)

# 함수형 프로그래밍
~~~
명령형 프로그래밍, 객체지향 프로그래밍에 이은 프로그래밍 방법으로, 상태변경과 변경가능한 데이터를 피하는 코딩 스타일을 말합니다.

주로 백앤드(spring)에서는 객체지향 OOP를 선호하고 프론트에서는 함수형(FP)를 선호하는 경향이 많은데,
백엔드는 디비에 저장되는 영속성 데이터를 다루는 특성 때문에 객체지향을 선호하고 (데이터 모델링, 재사용성, 코드의 구조화 장점)
프론트는 인터렉션이 중요한 특성 때문에 FP를 선호하는것같다! (불변성, 순수 함수, 고차 함수 등의 개념을 통해 예측 가능하고 테스트하기 쉬운 코드를 작성가능)
~~~

~~~
번외로 선언적으로 코드를 짜는게 좋다! 는 말을 많이들어서, 선언형도 잠시 알아본다.
(선언형은 아래 링크를 통해 이해하는데 좋았음)
https://pyjun01.github.io/v/declarative-ui/?trk=feed_main-feed-card_reshare_feed-article-content
~~~

## 함수형 프로그래밍 원칙
~~~
1. 입출력이 순수한 함수여야한다
2. 부작용이 없어야한다
3. 함수와 데이터를 중점으로 생각한다 (객체지향과는 다른 선언적 프로그래밍)

-> 자바스크립트는 객체지향언어로 this와 같은 클로저 기능이 있어 완벽한 함수형 프로그래밍은 할 수가 없다.
~~~

### 함수형 프로그래밍 특징
~~~
1. 일급함수: 자바스크립트에서 함수는 일급 객체입니다. 변수에 할당할 수 있고, 다른함수의 인자로 전달할 수 있고, 함수를 반환할 수도 있습니다. (= 함수를 값처럼 사용할 수 있다는 뜻입니다)
2. 순수함수: 동일 입력에 대해 동일 출력이 나옴. 외부상태 의존안하고 사이드이펙 없음
3. 불변성: 복사본 생성하고 원본데이터 변경하지 않음
4. 고차함수: 함수를 다른인자로 전달, 함수를 리턴함
5. 함수 조합: 재사용성 + 간결
~~~

## 함수형 프로그래밍 커링(curring)
~~~js
const curry = fn => fn2 => a => b => fn(a,b) + fn2(a,b)

const add = (a,b) => a+b;

const multiply = (a, b) => a*b;

curry(add)(multiply)(3,5); // 23
~~~
~~~
함수를 반환하는 함수 (재사용, 코드)
-> 함수는 반복적인 동작을 재사용을 위한 코드.
-> 함수 자체를 재사용 할수 있도록 함수를 위한 함수를 만듬.
~~~
## 함수형 프로그래밍 예제
~~~javascript
// 선언
function addFunc(x) {
  return x + 10;
}

console.log(addFunc(5)); // 10
~~~

# 자바스크립트에서 함수는 일급객체다
~~~javascript
// 1. 변수 할당 가능
const morning = function (name) { return `hi ${name}, goodmorning` }

// 2. 파라미터로 함수를 전달 가능
function (addFunc, a, b) {
  return addFunc(a, b)
}

// 3. 다른 함수 반환 가능
function compareN (n) {
  return m => m > n;
}
~~~

## 내가 사용했던 코드를 돌아보자 (OOP와 FP를 적절히 혼용하며 사용)
~~~javascript
// 객체지향 패턴 + 함수형 프로그래밍 예제

/**
 * [객체지향 패턴]
 * 메소드와 변수가 oDrops 객체 안에 들어있음
 * this 바인딩사용
 * 
 * 함수형 프로그래밍
 * 메소드가 gDomReady의 콜백으로 전달됨(일급함수 = gDomReady, oDrops의 함수들)
 * 함수들을 init 에서 조합하며 사용함
 */
let oDrops = {
  fnSetTimeDropsRemainQty: 0,

  // 초기
  init: function() {
    this.drawFNQ();
    this.fnViewList();
  },

  // 진행률 계산
  calc: function() {},

  // 자주묻는 질문 영역 계산
  drawFNQ: function() {},

  // data fetch
  fnViewList: function() {
     clearTimeout(this.fnSetTimeDropsRemainQty);
  },
}

gDomReady(function() {
    oDrops.init()
});
~~~
