# 프록시(Proxy)
~~~
자바스크립트에서 프록시는 객체의 동작을 가로채 다른 동작을 할 수 있게 해주는 기능이다. (대리 라는 뜻이다, ES6에서 첨 나옴)
(객체 조작 방법 심화버전이다.)
~~~

~~~javascript
let proxy = new Proxy(target, handler);
// target: 프록시 할 원본 객체
// handler: 타겟의 트랩이라고 불리는 메소드들을 가진 객체, get은 읽을때 실행되요 set은 쓸떄 실행되요
~~~

## 예제로 빠르게 이해해보자
~~~javascript
let target = {
  message1: "hello",
  message2: "everyone"
};

let handler = {
  get: function (target, prop, receiver) { // target, .message1~3, value
    return prop in target?
      target[prop]
      :
      `Property ${prop} dosen't exiest`
  }
}

let proxy = new Proxy(target, handler);

console.log(proxy.message1); // 예상: hello
console.log(proxy.message2); // 예상: everyone
console.log(proxy.message3); // 예상: Property message3 dosen't exiest
~~~

# 프록시 활용
~~~javascript
//1. 속성,타입 유효성 검사 (옛날엔 타입스크립트가 없었으니 필요하면 이렇게 하지 않았을까...)

let validator = {
  set: function (obj, prop, value) {
    if (props === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('Age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('Age seems invalid');
      }
    }
    obj[prop] = value;
    return true;
  }
}

let person = new Proxy({}, validator);
person.age = 100; // set 작동 -> return true;
person.age = 'gayoung'; // set 작동 -> type Error


//2. 객체에 대한 작업을 로깅하거나 디버깅할 때 사용
let logger = {
  get: function(target, prop) {
    console.log(`Read ${prop}`);
    return target[prop];
  },
  set: function(target, prop) {
    console.log(`Set ${prop}`);
    target[prop] = value;
    return true;
  }
}

let person = new Proxy({}, logger);

person.name = 'kong'; // Set kong 출력
console.log(person.name); // Read kong 출력 

//3. 객체의 속성이 업데이트 될 때 UI 업데이트 되도록 트리거 가능
function updadeUI(prop, value) {
  console.log(`UI updated: ${prop} is now ${value}`);
}

let bindingHandler = {
  set: function(target, prop, value) {
    target[prop] = value;
    updateUI(prop, value);
    return true;
  }
}

let model = new Proxy({}, bindingHandler);
model.message = 'Hello'; // html 속성이 변경되어도 쓸 수 있을듯

//4. 원본 객체가 가지지 않은 값에 대한 가상의 값 제공

//=> 프록시는 약간의 성능 오버헤드가 있어 꼭 필요한 경우에반 사용
~~~

