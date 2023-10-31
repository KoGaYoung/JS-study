# 원시타입 VS 참조타입
~~~
원시타입: number, string, symbol, undefined, boolean
참조타입: 객체, 배열, 클래스, 함수 ...
~~~
~~~java script
// 원시타입은 원형의 데이터는 수정이 불가능. 재할당만 가능. 아래 예시의 경우에 old와 new 모두 메모리에 존재한다. 
let a = 'old';
a = 'new'; 
// 참조타입은 원형의 데이터가 수정이 가능하다
let a = [];
a.push(2)
~~~

# 수정이 가능하지만 참조타입 객체의 Immutability(불변성)을 지켜야 하는 이유
~~~
1. 
객체나 배열 같은 참조타입 변수의 경우 값을 바로 바꾸면 안된다.
ex)
let a = { key1 : 1, key2: 2 };
a.key1 = 2; // 이러면 안된다...
 
~~~

# Immutable 쓰는 이유
~~~
객체의 불변성을 지키기 위함
수정이 불가능한(불변)객체를 만들어 추가, 수정, 삭제 시마다 새로운 객체를 반환하도록 한다.
~~~
~~~javascript
import { Map, List, is, fromJS } from 'immutable'; // 라이브러리 임포트
~~~
# Immutable에서 가장 많이 사용하는 get, getIn, set, setIn 예제
~~~
immutable.js는 collection 추상클래스를 참조한다.
~~~
~~~ javascript
// getIn 은 key 값을 Iterable 로 받아 값을 접근
collection.get('key');
collection.getIn(['key1', 'key2']);

// set과 update는 값이 같지 않다면, 항상 새로운 Class 를 반환
// set과 update의 차이점은 update는 함수를 입력받아 Class 의 값을 조작할 수 있게 해줌
collection.set('key', 'value');
collection.setIn(['key1', 'key2'], 'value');
collection.update('key', (value) => value);
collection.updateIn(['key1', 'key2'], (value) => value);
~~~

# 실제 소스에 immutable.js 적용
~~~
개발하면서 주로 json형태의 리스트를 다루는데, 객체는 immutable로 관리하는게 좋다.
이유는 리액트의 가상돔은 변화를 계산하는데 얕은복사를 하여 계산한다.
얕은복사를 할 경우에는 객체를 다루는 도중 원형이 수정되어
연관된 컴포넌트들이 리랜더링 혹은 원치않는 동작이 일어날 수 있다. (+ 이런걸 어느정도 방지하기 위해 기능별로 컴포넌트 영역을 잘 나눠야함)

자 그럼 이뮤터블하게 개발 안하면 못하나? 그건아니다.
Object.assign등으로 객체를 복사해서 원형에 영향을 미치지 않게 할 수 있는데,
공수가 많이 들 뿐더러 얘도 딥카피는 아니고 스왈로카피라 한계가 존재한다. (+성능상 이슈도 존재)

+ immutable 없이는 어떻게 하면 불변성을 지킬 수 있는지.
1) 위의 내용처럼 Object.assign으로 객체 복사하여 조작
2) map, fillter, concat 등 itrator 함수를 활용하여 새로운 객체나 배열을 반환
3) spread(...)를 활용하여 바꾸고싶은 값만 바꿔주기
~~~
~~~
함수형 프로그래밍에서는 값을 항상 불변 상태를 유지한다.
함수 내에서는 객체의 상태를 변화시키지 않고, 항상 새로운 객체를 반환한다.
때문에, 이전에 변경되었던 사항을 신경 쓸 필요가 없으며, 항상 현재 상태만을 고민하면 되기 때문에, 복잡함이 없어진다.
불변객체 는 흐름 기반 프로그래밍이 가능하게 하는 첫 번째 요소이다.
~~~
# Immutable 장단점
~~~
장점
1. 객체의 수정사항에 대해 virtual dom처럼 수정한 부분(구조체)만 다시 만들고, 나머진 재사용해서 속도가 빠르다.
2. 이뮤터블해지면서 이전객체의 변화를 신경쓰지 않아도되서 side effect가 줄어든다.

단점
1. 항상 새로운 객체를 반환하면서 메모리사용이 높다
2. 다시 객체로 변환(.toJS())시 비용이 많이 든다.
~~~
