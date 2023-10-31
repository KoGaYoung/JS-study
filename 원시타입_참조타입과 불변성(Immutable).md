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
1. 예측 가능한 코드로 버그를 줄이기 위해서
2. 불변성을 유지하면 돔에서 변경내역을 추적하거나 복제할때 쓰는 비용 줄어서 성능향상
3. + 리액트에 잘 녹아듬 
~~~

# 불변성을 유지하는 방법
~~~
1. Object.assign() 을 이용한 복사 / (배열의 경우)concat을 이용한 복사

const origin = { name: gayoung, age: 27 };
const new1 = Object.assgin({}, origin, { age: 26 }; // 만나이 적용 ㅋㅋ

// new1는 { name: gayoungm age: 26 } 새배열됨

2. 스프레드 연산자를 이용한 복사
const new2 = {...origin, age: 26 };

3. map, filter, reduce 와 같은 새로운 배열을 반환하는 함수 사용
~~~

# 위 3가지 방법의 문제점 => swallow copy(얕은복사)
~~~
1 depth가 넘어갈 경우 depp copy(깊은복사) 하지 못하기 때문에 문제가 발생할 수 있다.

해결하기 위해 재귀적으로 복사 함수를 사용하거나
lodash/cloneDeep(), immer, immutable 라이브러리를 사용하는 방법이 있다.
~~~

# 라이브러리 비교 
~~~
1. lodash/cloneDeep() 
장점: 간단하고 사용하기 쉬움
단점: 데이터 구조가 복잡해지면 성능이슈

2. immer
기존 데이터 수정하는것처럼 불변성 유지할 수 있는 가독성좋은 라이브러리
장점: 원본 데이터를 직접 수정하는 것처럼 보이는 코드를 작성
단점: 약간의 성능이슈, 약간의 러닝커브

3. immutableJS
데이터 구조( Map, List, is, fromJS 둥)를 제공하는 라이브러리
장점: 성능 최적화
단점: 기존코드랑 호환 잘 안될 수 있음, 약간의 러닝커브, fromJS <-> toJS 왔다갔다할때 비용 큼
~~~

#
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

