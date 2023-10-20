# 호이스팅(hoisting)이란?
~~~
전역변수와 함수를 선언된 스코프의 상단으로 끌어올리는 것, 선언과 할당이 분리가 된다고도 할 수 있다.
~~~

# 호이스팅 예제
~~~
아래 코드를 보면 다른 언어들 같은 경우에는 x를 선언하기 전인데 console.log(x)를 실행하면 오류가 발생하지 않고 undefined라고 한다.
이유는 오른쪽 함수처럼 인터프리터가 내부적(실행문맥상)으로 변수의 선언부분을 상단으로 끌어 올리기 때문이다.
~~~
~~~java script 
// 코드상의 함수                                //실제 실행 문맥(컨텍스트)
function foo() {                            function foo() {
  console.log(x); // undefined                  var x;
  var x = 10;                                   console.log(x); // undefined
  console.log(x); // 10                         x = 10;
}                                               console.log(x); // 10
                                            }  
                                            
// var만 undefined로 초기화
// let과 const는 호이스팅은 하지만 undefined로 초기화하지 않아 [Cannot access 'x' before initialization]에러가 발생한다
~~~

~~~
함수와 변수 모두 선언문일 때만 초기화가 된다
~~~
~~~java script
//함수 표현식(Function express)                   // 함수 선언식(Function declaration)      // 함수 선언시 실제 실행문맥(컨택스트)
d();	// ERROR, undefined는 호출할 수 없기 때문에    j();                                   function j () {
                                                                                           console.log('j');
var d = function () {                          function j () {                         }
	console.log('I am inside function d');           console.log('j');                   
};                                             };                                      j();

d();                                           j();                                    j();
~~~
<!-- # 호이스팅은 왜 발생하는가? 

인터프리터가 변수와 함수의 메모리 공간을 선언 전에 미리 할당하는 것을 의미합니다
인터프리터가 실행전에 실행 문맥(컨텍스트)를 만들기위해 최초에 처음부터 끝까지 훑어 나가며 변수 정보를 수집하는데,
실행 컨텍스트가 관여할 코드들은 실행되기 전의 상태이지만 자바스크립트 엔진은 이미 해당 환경에 속한 코드의 변수명들을 모두 알고 있게 됩니다.
https://www.zigae.com/javascript-basic/
https://poiemaweb.com/js-execution-context
# 컨텍스트는 무엇이고 왜 이런 동작을 하는가? -->
                                     
                                     
# 실제 소스에 호이스팅 적용
~~~
코드의 가독성과 유지보수를 위해 Hoisting이 일어나지 않도록 한다. 
Hoisting을 제대로 모르더라도 함수와 변수를 가급적 코드 상단부에서 선언하면, Hoisting으로 인한 스코프 꼬임 현상은 방지할 수 있다.
함수는 표현식으로 사용하고 변수는 let/const를 사용한다.
~~~
                                            
                                            
