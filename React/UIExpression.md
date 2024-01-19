## 목차

```
목표: 컴포넌트 무엇? React 컴포넌트 역할

1-1. 첫 번째 컴포넌트
1-2. 컴포넌트 import 및 export하기
1-3. JSX로 마크업 작성하기
1-4. 중괄호가 있는 JSX 안에서 자바스크립트 사용하기
1-5. 컴포넌트에 props 전달하기
1-6. 조건부 렌더링
1-7. 리스트 렌더링
1-8. 컴포넌트를 순수하게 유지하기
1-9. 트리거시의 UI
1-10. 상용구형성 다하기
```

## 1-1. 첫 번째 컴포넌트

```
코드 작성 -> 재사용할 수 있는 내용 -> 함수로 묶기
HTML작성 -> 재사용 가능한 HTML -> 컴포넌트로 묶기

chakra, Material, Bootstrap등 UI 오픈소스가 있다.
(개인적으로는 백오피스를 개발핼 때 빠르게 구현하기 위해 적절히 활용해도 좋다고 생각한다.
단점으로는 오픈소스는 완벽하게 커스텀하기 어려울 때가 있었다.)

리액트에서 컴포넌트는 항상 JSX 엘리먼트를 반환합니다! (=DOM 요소를 반환)
```

## 1-1. 컴포넌트 정의하기

```
[1단계(재사용 할 수 있도록) 컴포넌트 내보내기]
export default 접두사는 표준 JavaScript 구문

[2단계 함수 정의하기]
function 컴포넌트명() {}

[3단계 리턴구문에 마크업 추가]
```

```js
export default function MyComponent() {
  return (
      {/* 컴포넌트는 시작이 대문자! html은 소문자! (필수) */}
    <div>
      <p>Good Saturday</p>
      <ChildComponent />
    </div>
  );
}
```

### 1-1. 컴포넌트 중첩 (안티패턴)
~~~js
import React, {useState} from "react";

export default function Home() {
  // useState는 1회 생성 후 값을 유지함.
  // 렌더링마다 생성되지 않음
  const [a, setA] = useState(0);

  // 🔴 Never define a component inside another component!
  // 함수는 랜더링마다 생성됨.
  function Company() {
    return <div>company</div>;
  }
  
  return (
    <div>
      <p>Home!!!!</p>
      <p>{a}</p>
      <button onClick={() => setA((a) => a + 1)}>+</button>

      <Company />
    </div>
  );
}

~~~

~~~
안티패턴인 이유:
Home은 1회 렌더링 이후 라이프사이클에 따라 재랜더링이 이루어짐.
Company는 Home이 재랜더링 될 때마다 다시 생성됨.
useState는 다시 안생성되지 않나! -> 그것은 useState, useRef는 라이프사이클에 연동되기때문

Company가 다시 생성된다는 뜻 -> 메모리 사용량 증가. 
Home이 랜더링 될 때마다 Company 재랜더링!
훅으로 사용할 경우 상태값 잃어버릴 위험도있음

해결방안 아래처럼 전역으로 설정, 값전달이 필요할경우 child에 props로 전달
~~~
~~~js
export default function Home() {
  // ...
}

// ✅ Declare components at the top level
function Company() {
  // ...
}
~~~

## 1-2. 컴포넌트 import 및 export하기
~~~
exprot -> 내보내진 컴포넌트(함수, 객체, 원시 값 등)들은 무조건 엄격모드가 됨(MDN참고)
1. [default export 방식]
 - 한 파일 내에서 단 한개의 변수/클래스/함수만을 Export 할 수 있다.
 - from 뒤에 명시한 파일에서 단 한개의 모듈만 가져오기 때문에 as 키워드 없이 원하는대로  이름을 바꿀 수 있다.


2. [name export 방식]
 - 한 파일 내에서 여러개의 변수/클래스/함수를 Export 할 수 있다.
 - Import할 때 as 키워드를 사용해서 다른 이름을 지정할 수 있다.

방식의 선택은 개발자 선호도에 따라서! (통일 되면 좋겠지만요)
default export를 우선적으로 사용하고, 
한 파일에서 여러개를 export 해야하는 경우는 named export를 사용하고 있다.

요약: 임포트떄 중괄호 비구조할당 하면 네임방식
~~~
[모듈은 여기에 정리](../JS/ModuleSystem.md)

~~~js
// 1. [default export 방식]
// import
import MyDefaultComponent from './MyDefaultExport'

// export
const MyComponent = () => { ... }
export default MyComponent;
~~~
~~~js
// 2. [name export 방식]
// e.g., Importing a Single Named Export
import { MyComponent } from './MyComponent'

// e.g., Importing Multiple Named Exports
import { MyComponent, MyComponent2 } from './MyComponent'

// e.g., Giving a Named Import a Different Name by Using 'as'
import { MyComponent2 as MyNewComponent } from './MyComponent'

// e.g., 모두 가져오기 
import * as MainComponents from './MyComponent';

// Exports from ./MyComponent.js file
export const MyComponent = () => { ... }
export const MyComponent2 = () => { ... }
~~~
~~~
엄격모드? 
// 어디서 사용하냐에 따라 전역, 스코프 단위로 엄격모드 적용 가능
"use strict";

엄격모드 사용했을 때 변화:
 - 에러 전달
 - 선언되지 않은 Global변수를 사용할 수 없음
 - 읽기 전용 프로퍼티에는 대입할 수 없음
 - 매개변수 이름이 중복되어서는 안 됨.
 - this 포인터가 가리키는 값이 null, undefined인 경우 전역 객체로 반환되지 않음
 - 예약어를 사용할 수 없음
~~~

## 1-2. 확장자가 없는 경우
~~~js
import Gallery from './Gallery';
~~~
~~~
있어도 없어도 되지만, (주로 린트 적용하면 나와요)
확장자 .js 있는게 ES자바스크립트 문법에 가까움.
없으면 발생하는 문제: 브라우저 호환성 문제, 웹팩 바벨같은 빌드시스템에서 파일 못찾아 오류
리액트로 개발하다보면 webpack같은 도구들이 자동파일 해석해줘서 없는게 개인취향임

cjs에서는? 없는게 더 commonjs에 가깝다.
~~~

## 1-3. JSX로 마크업 작성하기
~~~
웹 인터렉션이 많아지면서 로직이 HTML을 결정 -> 리액트에선 랜더구문에서 HTML + js 함께존재
=> JSX(javascript Syntax eXtension)

JSX 규칙
1. 하나의 루트 엘리먼트로 반환하기 
    이유: JSX는 HTML처럼 보이지만 내부적으로는 일반 JavaScript 객체로 변환됨.
        리턴구문 안에 사용 -> 리턴을한다 -> 리턴하는 값이 2개일 수 없다.
    Fragments는 브라우저상의 HTML 트리 구조에서 흔적을 남기지 않고 그룹화해 줌(필요에따라 적절히 활용)
2. 모든 태그는 닫아주기 
3. 속성 캐멀케이스로(aria-*와 data-*의 어트리뷰트는 HTML에서와 동일하게 대시 사용, aria는 웹접근성용 data-는 개발자 데이터 html에 저장용 -> 자바스크립트는 이전버전의 슈퍼셋이기때문에 이전버전을 항상 지원해야함. 웹표준에 근거한 맥락)
~~~

## 1-4. 중괄호가 있는 JSX 안에서 자바스크립트 사용하기
~~~
1. 문자열 어트리뷰트를 JSX에 전달하려면 작은따옴표나 큰따옴표
2. 중괄호 { } 사이에서 JavaScript를 사용할 수 있음
    ㄴ 문자열, 어트리뷰트 = 다음

3. 이중 중괄호 사용
    ㄴ css 인라인스타일 객체를 넘겨줘야하니까 괄호로 한번 더 묶기
~~~
~~~js
import React, { useState } from "react";

export default function Home() {
  const [name, setName] = useState("gayoung");

  return (
    <>
      <div
        style={{
        // JSX인라인 스타일에선 캐멀케이스 주목, 원래 이거 background-color
          backgroundColor: "black",
          color: "pink",
          width: "300px",
        }}
      >
        <p>hi, {name}</p>
      </div>
    </>
  );
}

~~~

## 1-5. 컴포넌트에 props 전달하기
~~~js
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
~~~

~~~js
// 구조분해할당~, 기본값 지정 가능
function Avatar({ person, size = 10 }) {
  // person과 size는 이곳에서 사용가능합니다.
}
~~~

## 1-5. [배열에 넘기는 JSX를 같이보자](./JS/ArrayManipulate.md)
