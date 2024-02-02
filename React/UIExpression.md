## 목차

```
목표: 컴포넌트 무엇? React 컴포넌트 역할

(1월 20일 (토))
1-1. 첫 번째 컴포넌트
1-2. 컴포넌트 import 및 export하기
1-3. JSX로 마크업 작성하기
1-4. 중괄호가 있는 JSX 안에서 자바스크립트 사용하기
1-5. 컴포넌트에 props 전달하기

(1월 27일 (토))
1-6. 조건부 렌더링
1-7. 리스트 렌더링
1-8. 컴포넌트를 순수하게 유지하기
1-9. 트리로서의 UI
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

```js
import React, { useState } from "react";

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
```

```
안티패턴인 이유:
Home은 1회 렌더링 이후 라이프사이클에 따라 재랜더링이 이루어짐.
Company는 Home이 재랜더링 될 때마다 다시 생성됨.
useState는 다시 안생성되지 않나! -> 그것은 useState, useRef는 라이프사이클에 연동되기때문

Company가 다시 생성된다는 뜻 -> 메모리 사용량 증가.
Home이 랜더링 될 때마다 Company 재랜더링!
훅으로 사용할 경우 상태값 잃어버릴 위험도있음

해결방안 아래처럼 전역으로 설정, 값전달이 필요할경우 child에 props로 전달
```

```js
export default function Home() {
  // ...
}

// ✅ Declare components at the top level
function Company() {
  // ...
}
```

## 1-2. 컴포넌트 import 및 export하기

```
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
```

[모듈은 여기에 정리](../JS/ModuleSystem.md)

```js
// 1. [default export 방식]
// import
import MyDefaultComponent from './MyDefaultExport'

// export
const MyComponent = () => { ... }
export default MyComponent;
```

```js
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
```

```
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
```

## 1-2. 확장자가 없는 경우

```js
import Gallery from "./Gallery";
```

```
있어도 없어도 되지만, (주로 린트 적용하면 나와요)
확장자 .js 있는게 ES자바스크립트 문법에 가까움.
없으면 발생하는 문제: 브라우저 호환성 문제, 웹팩 바벨같은 빌드시스템에서 파일 못찾아 오류
리액트로 개발하다보면 webpack같은 도구들이 자동파일 해석해줘서 없는게 개인취향임

cjs에서는? 없는게 더 commonjs에 가깝다.
```

## 1-3. JSX로 마크업 작성하기

```
웹 인터렉션이 많아지면서 로직이 HTML을 결정 -> 리액트에선 랜더구문에서 HTML + js 함께존재
=> JSX(javascript Syntax eXtension)

JSX 규칙
1. 하나의 루트 엘리먼트로 반환하기
    이유: JSX는 HTML처럼 보이지만 내부적으로는 일반 JavaScript 객체로 변환됨.
        리턴구문 안에 사용 -> 리턴을한다 -> 리턴하는 값이 2개일 수 없다.
    Fragments는 브라우저상의 HTML 트리 구조에서 흔적을 남기지 않고 그룹화해 줌(필요에따라 적절히 활용)
2. 모든 태그는 닫아주기
3. 속성 캐멀케이스로(aria-*와 data-*의 어트리뷰트는 HTML에서와 동일하게 대시 사용, aria는 웹접근성용 data-는 개발자 데이터 html에 저장용 -> 자바스크립트는 이전버전의 슈퍼셋이기때문에 이전버전을 항상 지원해야함. 웹표준에 근거한 맥락)
```

## 1-4. 중괄호가 있는 JSX 안에서 자바스크립트 사용하기

```
1. 문자열 어트리뷰트를 JSX에 전달하려면 작은따옴표나 큰따옴표
2. 중괄호 { } 사이에서 JavaScript를 사용할 수 있음
    ㄴ 문자열, 어트리뷰트 = 다음

3. 이중 중괄호 사용
    ㄴ css 인라인스타일 객체를 넘겨줘야하니까 괄호로 한번 더 묶기
```

```js
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
```

## 1-5. 컴포넌트에 props 전달하기

```js
export default function Profile() {
  return (
    <Avatar person={{ name: "Lin Lanying", imageId: "1bX5QH6" }} size={100} />
  );
}
```

```js
// 구조분해할당~, 기본값 지정 가능
function Avatar({ person, size = 10 }) {
  // person과 size는 이곳에서 사용가능합니다.
}
```

## 1-5. [배열에 넘기는 JSX를 같이보자](./JS/ArrayManipulate.md)

---

## 1-6. 조건부 렌더링

```jsx
// 조건부 렌더링
if (isPacked) {
  return <li className="item">{name} ✔</li>;
} else {
  return <li className="item">{name}</li>;
}

// 삼항 연산자로 조금더 DRY(중복을 최소화)한 형태
return <li className="item">{isPacked ? name + " ✔" : name}</li>;
```

```
두 에제는 동일할까요?
공식 문서에서는 실제 DOM요소가 아니기에 완전히 똑같다고 인식한다.

조건부 렌더링에서는 <li>태그를 다시그리는 반면
삼항 연산자의 경우 <li>태그는 그대로 두고, 내용물만 다시 그리기 때문에
렌더링 관점에서 효율적이라고 볼 수 있다.

둘 중 하나만 리턴되기때문에 리액트 엔진이 동일한 위치에 같은태그는 같은 컴포넌트로 인식해서 초기화되지않는다
()
```

## 1-6. 조건부 렌더링 && 표현

```jsx
// isPacked 가 false인 경우 && 논리곱 연산자는 false를 리턴함으로써 ✔를 렌더링하지 않는다.
// isPacked 가 false가 아닌 0인 경우에는 0이 렌더링된다. 주의하자.
// false, null, undefeind는 렌더링되지않고, 0인경우 0이, []같은 경우 '✔'를 렌더링한다.
// 논리곱 연산자 좌항이 숫자인경우 !! 연산자 or 숫자 > 0을 통해 불리안값을 만듭니다.
return (
  <li className="item">
    {name} {isPacked && "✔"}
  </li>
);
```

## 1-7. 리스트 렌더링
~~~jsx
<ul>
  <li>Creola Katherine Johnson: mathematician</li>
  <li>Mario José Molina-Pasquel Henríquez: chemist</li>
  <li>Mohammad Abdus Salam: physicist</li>
  <li>Percy Lavon Julian: chemist</li>
  <li>Subrahmanyan Chandrasekhar: astrophysicist</li>
</ul>
~~~

~~~jsx
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
  'Mohammad Abdus Salam: physicist',
  'Percy Lavon Julian: chemist',
  'Subrahmanyan Chandrasekhar: astrophysicist'
];

export default function List() {
  // => 뒤에는 return 생략가능 => {여기에는 리턴 필요}
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
~~~

~~~jsx
import React, { useState, Fragment } from "react";

export const people = [
  {
    id: 0, // JSX에서 key로 사용됩니다.
    name: "Creola Katherine Johnson",
    profession: "mathematician",
    accomplishment: "spaceflight calculations",
    imageId: "MK3eW3A",
  },
  {
    id: 1, // JSX에서 key로 사용됩니다.
    name: "Mario José Molina-Pasquel Henríquez",
    profession: "chemist",
    accomplishment: "discovery of Arctic ozone hole",
    imageId: "mynHUSa",
  },
  {
    id: 2, // JSX에서 key로 사용됩니다.
    name: "Mohammad Abdus Salam",
    profession: "physicist",
    accomplishment: "electromagnetism theory",
    imageId: "bE7W1ji",
  },
  {
    id: 3, // JSX에서 key로 사용됩니다.
    name: "Percy Lavon Julian",
    profession: "chemist",
    accomplishment:
      "pioneering cortisone drugs, steroids and birth control pills",
    imageId: "IOjWm71",
  },
  {
    id: 4, // JSX에서 key로 사용됩니다.
    name: "Subrahmanyan Chandrasekhar",
    profession: "astrophysicist",
    accomplishment: "white dwarf star mass calculations",
    imageId: "lrWQx8l",
  },
];

export default function Page({ dataNum }) {
  const listItems = people.map((person) => (
    <Fragment key={person.id}>
      <h1>{person.name}</h1>
      <p>{person.imageId}</p>
    </Fragment>
  ));

  return <div>{listItems}</div>;
}

~~~
~~~
리스트를 렌더링할 경우 Key는 필수이다.
CRUD가 일어났을 때 리액트가 무슨일이 일어났는지 계산하고 Dom트리의 업데이트를 최소화 할 수 있다.
Key값은 즉석에서 생성하는게 아닌, 리스트에 포함되는 데이터여야한다.

또한 key라는 속성을 사용하기위해 위의 소스처럼 Fragment를 명시해주거나, Div로 감쌀 수 있다.
Fragment를 사용해도 Vritual dom이 people 리스트에 CRUD가 일어나도 변경되는 데이터를 잘 감지할 수있다. (렌더링 최적화 관점에서 굿)

DB data(api fetching data) : id, key값 존재.
로컬테이터: 증분 일련번호나 crypto.randomUUID() 또는 uuid 같은 패키지를 사용

key는 형제 간에 고유해야 합니다.
key는 변경되어서는 안 되며 렌더링중에는 생성하지 마세요.

재정렬로 인해 _위치_가 변경되더라도 key는 React가 생명주기 내내 해당 항목을 식별할 수 있게 해줍니다.
~~~

## 1-8. 컴포넌트를 순수하게 유지하기

~~~
개인적으로 영속적인 데이터를 다루는 백앤드는 OOP를 선호하고,
인터렉션이 중요한 프론트앤드는 FP를 선호한다고 생각합니다.

함수형 프로그래밍을 익혀야하는 이유 중 하나로 리액트에 잘 녹아들기 때문이라고 배웠습니다.

순수함수로서 자신의 일에 집중하고,
몇번을 호출해도 같은 값을 리턴해야합니다.
~~~
[함수형 프로그래밍](../JS/FP.md)

~~~jsx
// 잘못된 예시
let guest = 0;

function Cup() {
  // 나쁜 지점: 이미 존재했던 변수를 변경하고 있다!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
~~~

~~~jsx
// 올바르게된 예시
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      {/* props로 넘기기 */}
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
~~~

~~~
전체 컴포넌트를 <React.StrictMode>로 엄격모드 함으로써
순수 컴포넌트가 아닐경우 에러를 강제적으로 발생시킬 수 있습니다.

가끔 콘솔에서 콘솔이 2번 찍힐떄가 있는데, 이건 엄격모드로
2번 실행시키고 있기 때문이다.
(오랫동안 궁금했던 2번 실행의 이유...React.StrictMode)
~~~

~~~jsx
// 렌더링하는 동안 그냥 만든 변수와 객체를 변경하는 것은 전혀 문제가 없습니다. 

// cups과 [] 배열이 TeaGathering안에서 생성되었기 때문 => 지역 변형
export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
~~~

~~~
이벤트 헨들러는 순수할 필요가 없다. 이유는 렌더링중에 실행되지 않기 떄문이다.
가능하면 렌더링만으로 로직을 표현

순수함수로 컴포넌트를 생성해야 하는 이유
1. 컴포넌트는 다른 환경에서도 실행될 수 있습니다
  예를 들면 서버에서 실행
  동일한 입력에 대해 동일한 결과를 반환하기 때문에 하나의 컴포넌트는 많은 사용자 요청을 처리할 수 있습니다
2. 입력이 변경되지 않은 컴포넌트 렌더링을 건너뛰어 성능을 향상시킬 수 있습니다. 
  순수 함수는 항상 동일한 결과를 반환하므로 캐시하기에 안전합니다.
3. 깊은 컴포넌트 트리를 렌더링하는 도중에 일부 데이터가 변경되는 경우 
  React는 오래된 렌더링을 완료하는 데 시간을 낭비하지 않고 렌더링을 다시 시작할 수 있습니다. 
  순수함은 언제든지 연산을 중단하는 것을 안전하게 합니다.
~~~

## 1-9. 트리로서의 UI
~~~
브라우저는 HTML (DOM)과 CSS (CSSOM)를 모델링하기 위해 트리 구조를 사용

왜 많고많은 자료구조 중 트리를 선택했을까?
1. 계층적 구조 반영: DOM은 부모-자식 관계 형성, CSS는 위에서 아래로 전파.
   트리구조는 이를 자연스럽게 표현가능
2. 효율적인 탐색 및 수정
3. 렌더링 최적화: 트리 구조를 사용함으로써, 
  부모 노드의 변경이 자식 노드에 어떤 영향을 미칠지 쉽게 결정할 수 있으며, 불필요한 렌더링 작업을 최소화
4. 이벤트 버블링, 캡처링 구현 용이
5. 레이아웃 계산 용이: 트리구조는 요소간 상대적인 관계를 명확하게 함
~~~

~~~
최상위 컴포넌트는 루트 컴포넌트에 가장 가까운 컴포넌트이며, 
그 아래의 모든 컴포넌트의 렌더링 성능에 영향을 미치며, 가장 복잡성이 높습니다. 
리프 컴포넌트는 트리의 맨 아래에 있으며 자식 컴포넌트가 없으며 자주 다시 렌더링 됩니다.
~~~
### 1-9 모듈 의존성 트리 
~~~
의존성 트리는 React 앱의 모듈 의존성을 나타냅니다.
개발자가 컴포넌트 분리, 로직 분리 하여 JS 모듈을 만듬.
각 모듈은 import로 불러와야하고 이걸로 의존성 트리 생성

리액트에서 빌드할 때, 필요한 JS를 묶는 번들링 단계가 있음.
번들러가 의존성 트리를 사용하여 포함해야할 모듈 결정

번들크기 체크 방법
1. webpack-bundle-analyzer 을 이용한 번들사이즈 체킹
2. cra 전용 cra-bundle-analyzer를 이용해서 사용함
~~~



---

## 스터디하다가 나온 주제들


```
.jsx 확장자 언제유효?
---
props undefined는 없는값취급됨
null, 0은 default 값 사용 불가능
---
스프레드를 썼을 때 명시적이지 않음
객체로 넘겨져서 안쓰는게 어트리뷰트를 따로 빼서 쓰는게 좋음
...props를 썼을때 렌더링관점에서 이점이 있는지?
---
// 선언식
export default function Button () {}

// 표현식
const Button = () => {}

export default Button


export 자체는 한줄로 써도 문제가업슴.
default에 그런 기능이있는거아닐까?
---
//
const {name, age} = props;

// props 비구조화 할당 로로패턴 성능상 이점이 있는지!
function Child({ name, age}) {

}
---
uuid 패키지 사용 안해도됨
-> crypto,randdomUUID() 내장 라이브러 제공
-> 브라우저 호환성 떨어짐

소스수정!

key가 변경될 경우 예제소스 맨앞에 아이템 추가했을때 전체 렌더링!
```
