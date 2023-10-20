# 프로토타입(Prototype)이란 무엇인가?
~~~
프로토타입은 객체가 부모를 참조하여 상속하는 기능입니다.
~~~
~~~
자바스크립트는 프로토타입기반 언어로 객체지향언어입니다. 
하지만 다른 언어들과 다르게 인터페이스를 통한 상속 기능이 없어 이를 프로토타입으로 구현할 수 있습니다.
아래 코드와 같이 new로 생성 할 경우 [[prototype]] 혹은 __proto__라는 프로퍼티를 가지게 됩니다.
~~~
~~~javascript
function Person(first, last, age) {

  this.first = first;
  this.last = last;
  this.age = age;
}
let person1 = new Person('ko', 'gayoung', 25)

// person1의 __proto__속성은 원형 부모의 prototype 참조
// [[prototype]]인 prototype 객체 내부에는 인스턴스가 사용할 메소드가 저장되어있다.
// constructor는 Person 함수를 가리킨다.
~~~
~~~
(아래 이미지 참고)js의 모든 객체는 프로토타입이라는 프로퍼티를 갖고있으며 프로토타입 체인(prototype chain)입니다.
모든 프로토타입 체인의 끝은 항상 Object.prototype이고, 종착역인 Object.prototype은 __proto__ 속성이 없습니다.
~~~
![image](https://user-images.githubusercontent.com/36693355/137751282-b480ae38-1563-4d55-9400-07c1c295daef.png)
~~~
프로토타입 체인에서 한 객체의 메소드와 속성들이 다른 객체로 복사되는 것이 아니다. 
위 이미지에서 보시다 시피 체인을 타고 올라가며 접근할 뿐이다.
~~~

~~~
(아래 이미지 참고)
prototype vs __proto__
prototype 은 함수 객체만이 가지는 프로퍼티이고 함수 객체가 생성자로 사용될 때 인스턴스의 부모 역할을 하는 객체를 가리킵니다. ex)Person's prototype 
__proto__ 프로퍼티는 모든 객체가 가지고 있는 속성이며 객체가 생성될 때 자신의 부모 객체인 prototype 을 참조합니다. ex)Person's prototype
~~~
  <img width="957" alt="스크린샷 2021-10-19 오후 8 51 30" src="https://user-images.githubusercontent.com/36693355/137903932-9902c37d-cf63-422b-a3f3-0ee3fae6da5f.png">


~~~javascript
//create 를 통해 새로운 객체를 할당 가능합니다. 
//getPropertyOf 를 통해 __proto__ 값을 리턴 받을 수 있습니다. 
//setPrototypeOf 를 통해 __proto__ 를 새로운 객체를 참조하도록 합니다.

let animal = {
  eats: true
};
let rabbit = Object.create(animal);
alert(rabbit.eats); // true
alert(Object.getPrototypeOf(rabbit) === animal); // true
Object.setPrototypeOf(rabbit, {}); // rabbit의 프로토타입을 {}으로 바꿉니다.
~~~
