# 프로토타입(Prototype)이란 무엇인가?
~~~
프로토타입은 객체가 부모를 참조하여 상속을 구현하는 기능입니다.
~~~

## 프로토타입 객체(Prototype Object)
~~~
자바스크립트는 프로토타입기반 언어로 객체지향언어입니다.
즉 객체들은 다른 객체의 '프로토타입을 부모삼아 속성이나 메소드를 상속'받을 수 있습니다.

일반적인 클래스기반 언어들과 다르게 명시적인 인터페이스를 통한 상속 기능이 없어 이를 프로토타입으로 유사하게 구현할 수 있습니다.
~~~

## 프로토타입 체인(Prototype Chain)
~~~
(아래 이미지 참고)js의 모든 객체는 [[prototype]] 이라는 프로퍼티를 갖고있으며 프로토타입 체인(prototype chain)입니다.
모든 프로토타입 체인의 끝은 항상 Object.prototype이고, 종착역인 Object.prototype은 [[prototype]] 속성이 없습니다.
(__proto__ 프로퍼티라고도 하는데 권장하지 않는다함)

객체의 속성이나 메소드에 접근하려고 할 때, 해당 속성/메소드가 없으면 프로토타입체인을 타고 올라가 속성/메소드를 찾습니다. (null 될때까지)
아래 그림처럼 타고올라갈 때 접근만 할 뿐이지 복사되는 메소드나 속성이 복사되진 않습니다.
~~~
![image](https://user-images.githubusercontent.com/36693355/137751282-b480ae38-1563-4d55-9400-07c1c295daef.png)



## 예제로 이해하기
~~~javascript
function Person(first, last, age) {

  this.first = first;
  this.last = last;
  this.age = age;
}
let person1 = new Person('ko', 'gayoung', 25)

// 아래 코드와 같이 new 연산자와 Person이라는 생성자함수를 사용할 경우,
// 생성된 객체 person1의 [[prototype]]는 "생성자함수 Person의 'prototype' 속성에 할당된 객체를 가리키게 됩니다."

// 요약: person1은 객체라 [[prototype]]라는 프로퍼티를 가지게 됨
// person1의 [[prototype]] 프로퍼티는 생성자함수인 Person의 prototype 속성에 할당된 객체를 가리킴

// person1.[[prototype]] 는 { constructor : f Person(first, last, age), [[ProtoType]]: Object }
// person1.[[prototype]] === Person.prototype
// person1.[[prototype]].constructor === Person
~~~

  <img width="957" alt="스크린샷 2021-10-19 오후 8 51 30" src="https://user-images.githubusercontent.com/36693355/137903932-9902c37d-cf63-422b-a3f3-0ee3fae6da5f.png">

## 접근자 [[prototype]] vs prototype 속성 차이
~~~
(위 이미지 참고)

[[prototype]]: 객체가 실제로 사용하는 접근자 프로퍼티

prototype 속성: 함수객체에만 존재하는 프로퍼티로, 함수 객체가 생성자로 사용될 때 인스턴스의 부모 역할을 하는 객체를 가리킴
~~~

# 프로토타입 사용 예시

~~~javascript
// 모든 객체는 Object.prototype를 상속받기 때문에 toString, hasOwnProperty 를 사용가능

// 모든 배열은 Array.Prototype을 상속받기 때문에 map, filter, reduce 같은 배열메소드에 접근가능
const exampleRef = document.getElementsByClassName('li');
Array.prototype.map.call(exampleRef, (e) => {...});

// 메소드 상속 또는 확장 -> 모든 배열 인스턴스가 이 메소드를 사용할 수 있음 (필요에 따라 메모리 절약됨)
Array.prototype.getFrist = function () {
  return this[0];
}
const array1 = [1,2,3];
console.log(array1.getFrist());

// 
~~~


~~~javascript
//create 를 통해 새로운 객체를 할당 가능합니다. 
//getPropertyOf 를 통해 [[prototype]] 값을 리턴 받을 수 있습니다. 
//setPrototypeOf 를 통해 [[prototype]] 를 새로운 객체를 참조하도록 합니다.

let animal = {
  eats: true
};
let rabbit = Object.create(animal);
alert(rabbit.eats); // true
alert(Object.getPrototypeOf(rabbit) === animal); // true
Object.setPrototypeOf(rabbit, {}); // rabbit의 프로토타입을 {}으로 바꿉니다.
~~~
