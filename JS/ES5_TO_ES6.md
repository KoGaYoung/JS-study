# ES5 -> ES6
~~~
ES는 자바스크립트 문법 버전이며 ECMAScript(에크마스크립트)의 약자다.

대체로 최신버전의 ES6를 사용하지만,
레거시 코드를 건드릴 일도 있고, 어떤게 최신문법에 맞게 개발하고 있는 것인지 알기위해서는 정리가 필요하다고 생각한다.
(ES7나왔을때도 더 파악하기 용이할것이다 +_+)
~~~

## let const 추가
~~~javascript
// 기존에는 모든걸 다 담아주는 var밖에 없었습니다.
// var는 중복선언 가능, 호이스팅, 스코프에 따라 전역으로 동작하면서 사이드이펙, 계속 변수가 살아있으니 메모리낭비 유발ㅠ

// 가변 변수인 let과 불변 변수인 const가 추가되었습니다
// 1. 블록스코프
{
  const variable = 'variable1';
  console.log(variable); // 'variable1'
}

// 2. 중복 선언 불가능
{
  let variable = 'variable1';
  let variable = 'variable2'; // syntax error
}

// 3. 호이스팅 안됨
{
  console.log(variable); // reference error
  let variable = 'variable1';
}
~~~

## arrow function 추가
~~~javascript
function func1 () {
  console.log('func1 log');
}

// arror function으로 변경할때는 function을 빼고 func, ()를  = () => 로 변경한다
func1 = () => {
  console.log('func1 log');
}

// this 바인딩 방식도 차이가 있음
// 옛날 함수는 this를 호출한 객체를 가리킴
function func1 () {
  this.value = 0;
  setTimeout(function(){
    console.log(this.value); // 단독으로 호출하면 window객체, 엄격모드일때는 undefined
  }, 1000)
}

func1 = () => {
  this.value = 0;
  setTimeout(() => {
    console.log(this.value); // 화살표 함수는 this는 함수가 생성될 때 값을 기억하는 렉시컬 바인딩, arrowFunction의 this와 동일함
  }, 1000)
}
~~~

## default parameter 가능
~~~javascript
function func1(isOpen) {
  console.log(isOpen); // true or false
}

// 화살표 함수도 같이적용해볼까!
func1 = (isOpen=false) => {
  console.log(isOpen); // defulat is false;
}
~~~

## template literal 가능
~~~javascript
var name = 'gayoung';
var age = '27';

var introduceES5 = "hello my name is " + name + " age is " + age;

// es6
const introduceES6 = `hello my name is ${name} age is ${age}`;
~~~

## Multiline string 가능
~~~javascript
var introduceES5 = "hello my name is gayoung\n" + "age is 27";

const introduceES6 = `hello my name is gayoung
age is 27`;
~~~

## class 추가됨
~~~javascript
자바스크립트는 프토토타입 기반의 객체지향 언어임에도 ES6에 와서야 class 도입됨. 그전까지는 prototype으로 구현

class classExample {
  private count;
  constructor(newCount) {
    this.count = newCount;
  }
}

const newClass1 = new classExample(0);
~~~

## 모듈
~~~javascript
// 모듈이 나오기 전에는 CommonJS(node에서사용)하거나 AMD(비동기모듈정의)와 같은 비표준방식이 사용됨
// export, import 방식이 도입됨

export function add(x, y) { return x + y; }

import { add } from './math';
console.log(add(2, 3)); // 5
~~~

## 비구조화 할당
~~~javascript
// 비구조화를 잘 쓰면 이쁜 코드를 짤 수 있다.
const numberArray = [1,2,3];
const [one, two, three] = numberArray; // one: 1, two: 2, three: 3

const objectStructure = {
  name: 'gayoung',
  age: 27
};
const { name, age } = objectStructure;
~~~

## Promise
~~~javascript
// 콜백함수에서 콜백지옥을 해결하기위해 Promise 객체 등장

function fetchWithCallback(url, callback) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (xhr.readyState === XMLHttpRequest.DONE) {
            if (xhr.status === 200) {
                callback(null, xhr.responseText);
            } else {
                callback(new Error('Request failed: ' + xhr.status), null);
            }
        }
    };
    xhr.open('GET', url, true);
    xhr.send();
}

fetchWithCallback('/api/data', function(err, data) {
    if (err) {
        console.error(err);
    } else {
        console.log(data);
    }
});

// --
function fetchWithPromise(url) {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.DONE) {
                if (xhr.status === 200) {
                    resolve(xhr.responseText);
                } else {
                    reject(new Error('Request failed: ' + xhr.status));
                }
            }
        };
        xhr.open('GET', url, true);
        xhr.send();
    });
}

fetchWithPromise('/api/data')
    .then(data => console.log(data))
    .catch(error => console.error(error));

~~~

## string 메소드
~~~javascript
const introduceES6 = 'hello my name is gayoung age is 27';

introduceES6.includes('my'); // true
introduceES6.startsWith('hello'); // true
introduceES6.endsWith('27'); // true
~~~
