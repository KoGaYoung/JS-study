# 모듈 시스템(Module System)
~~~
개발을 하다 .eslintrc.js와 .eslintrc.cjs 파일을 확인했다.
분명 CRA로 개발환경 설정하다보면 CommonJS 모듈을 고를거냐를 본 적은 있다.
모듈은 무엇일까?

설명을 덧붙이면 .eslintrc.js 린트를 설정하는 JavaScript 파일이고,
.eslint.cjs는 CommonJS 모듈이다.
~~~

~~~
자바스크립트에서의 모듈은 여러 기능이 모여있는 하나의 파일을 말한다.
모듈 활용의 장점은 유지보수성, 네임스페이스화(토끼굴..), 재사용성 등이 있다.

자바스크립트에서 Module을 구현하기 위한 방법에는 4가지가 있다
1. AMD (가장 오래됨, require 구문)
2. CommonJS (Node.js 서버를 위해 만들어진 모듈 시스템)
3. UMD(Universal Module Definition) = AMD + CommonJS
4. ES6 -> 자바스크립트도 모듈화를 할 수 있도록 각각의 모듈(파일)마다 독립적인 파일 스코프를 가지고 있어 스코프의 충돌을 방지
~~~

~~~
정리: cjs는 주로 노드에서 사용, js는 클라단에서 사용하는 모듈 시스템
~~~