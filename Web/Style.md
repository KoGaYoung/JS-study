~~~
css, scss만 주로 사용을 해왔다.
하지만 emotion, tailwind 등 많은 라이브러리가 나와서 새로 프로젝트를 진행한다면 고려를 안해볼 수 없게되었다.

~~~

# css vs scss
~~~
css는 Cascading Style Sheets의 약자로 스타일 언어입니다

(Sass는 개발 경험이 좋지 않았고 대체가능하며 scss보다 상대적으로 css와의 호환도 안좋아 앞으로도 고려하지 않을 것이라 패스)

SCSS는 문법적으로 멋진 Css입니다.

어떻게 멋진지?

중괄호로 중첩을 표현하고 세미콜론으로 속성을 구분합니다.
~~~

### 1. SCSS 변수할당
### 자주 사용하는 스타일을 변수로 만들어 활용할 수 있습니다.
~~~css
/* CSS */
body {
  font: 100% Helvetica, sans-serif;
  color: #333;
}

/* SCSS */
$font-stack: Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
~~~

### 2. SCSS 중첩(Nesting) 구문
### 가독성(유지보수성) up
~~~css
/* CSS */
nav ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
nav li {
  display: inline-block;
}
nav a {
  display: block;
  padding: 6px 12px;
  text-decoration: none;
}

/* SCSS */
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
~~~

### 3. SCSS 모듈화(Modularity)
### 스타일 파일을 분할하고 모듈화 가능
~~~css
/* _base.scss */
$font-stack: Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}

/* styles.scss */
@use 'base';

.inverse {
  background-color: base.$primary-color;
  color: white;
}
~~~
### 4. 믹스인(Mixins)
### 5. 확장/상속


~~~
새로나온 tailwind, emoton

scss와 차이점이 있다면?
css와 scss는 파일시스템을 통해 관리함으로써 전역/지역 스타일을 선택할 수 있고,

emotion과 같은 라이브러리는 css-in-js로 지역스타일이 디폴트입니다.
Global 컴포넌트를 통해 애플리케이션의 전역 스타일을 주입할 수도 있습니다.

현재까지는 React 로만 프로젝트를 구성한다면 emotion을 도입할 것 같다.
이유는 css, scss와 호환 잘됨(마이그레이션 쉬움)
CSS-in-js로 지역스타일이 디폴트라 복잡한 전역상태관리에서 어느정도 해방됨
테일윈드보다 가독성이 훨!씬!좋다

단, next프로젝트를 도입한다고하면 서버컴포넌트에서도 스타일을 지원하는 테일윈드는 필수불가결한요소가 될 것 같다..!
~~~