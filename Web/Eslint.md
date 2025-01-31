## eslint / prettier
~~~
팀원들과 코드를 같이 짜는데 있어서 모두 코드를 짜는 스타일이 다르면 변경점 파악이 어렵고 소스코드 충돌이 자주 발생합니다 :-(

이를 해결하깅 위해 컨벤션을 통해 규칙을 정하고, 최소한의 변경점만 가져가게 되는데요.
코드 품질을 높일 수 있습니다 +_+
eslint와 prettier를 설정하여 주로 문제를 해결합니다!

팀원마다 선호하는 에디터가 다릅니다. 아래 상황에 따라 적절하게 적용이 필요합니다.
(모두 같은 에디터를 사용하는 경우) 에디터를 통해 설정할 수도 있고,
(모두 다른 에디터를 사용하는 경우 ) eslintrc 파일과 prettierrc 파일을 통해 설정할 수도 있습니다.

(참고, 코드에티더상의 컨벤션 우선순위가 더 높음)

.eslintrc.cjs 로만 설정을 하는 방법
IntelliJ: Settings > Languages & Frameworks > JavaScript > Code Quality Tools > ESLint에서 수동 설정

VS Code
settings.json 파일에서 ESLint 패키지를 수동으로 설정하고 ESLint 설치 경로를 지정, "eslint.workingDirectories": [{"mode": "auto"}]
~~~

~~~
에디터별로 auto format on save 적용하기
intellij의 경우 언어 및 프레임워크 > Javascript > 코드 품질 도구 > ESLint > 저장 시 eslint --fix 실행,
visual studio code의 경우 auto format on save
~~~

## 기존 코드에 코딩컨벤션을 적용해야한다면?
~~~
순서대로 해보자.
1. 현재 코드와 airbnb 코딩컨벤션을 비교하여 항목별로 문서화한다. js, ts, react 항목별로, 차이점은 별도기재한다
2. 팀원들과 차이점들을 airbnb 표준에 맞출지, 현재코드로 갈지 논의한다
3. eslintrc, prettier 파일 등에 규칙을 error로 만든다.
4. 너무 많이 error가 발생한다면 warning으로 두고 천천히 수정하는것도 방법이다.
~~~

## 설정을 시작해보자
~~~
패키지매니저로 eslint, prettier를 설정할때는 꼭 --dev 옵션으로 개발환경에서만(devDependencies) 사용할 수 있도록 합니다 :-)
~~~

#@ eslint
~~~
esLint의 경우 주로 airbnb 컨벤션을 사용합니다!
(standard, google 등 다른 컨벤션들도 존재합니다 하지만 React, ES6에도 적용되고 사용량이 가장 많아서 에어비앤비를 추천합니다)
그리고 보통은 프로젝트 생성하면더 eslint 적용 여부가 나와 아래 스탭은 필요하지 않습니다.

1. 프로젝트 루트 디렉토리에서 eslint 설치
npm install eslint --save-dev

2. eslint 설정
npx eslint --init

eslint-config-airbnb@latest eslint@^7.32.0 || ^8.2.0 eslint-plugin-import@^2.25.3 eslint-plugin-jsx-a11y@^6.5.1 eslint-plugin-react@^7.28.0 eslint-plugin-react-hooks@^4.3.0
✔ Would you like to install them now? · No / [Yes]
✔ Which package manager do you want to use? · npm (익숙한 것 사용)

3..eslintrc.js 파일 셋팅
package.json에 설치된 devDependancy를 보고 적절히 추가한다. 필요한 경우 더 추가한다
~~~
<img width="745" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/e0fe52f5-52e3-4981-bc8d-1caf2a9ebbf8">

~~~
4. 나의경우 로컬에서 수정하면서 lint 오류나오면 바로 수정하고싶다
npm install concurrently --save-dev

하고 package.json 스크립트 부분에 아래 명령어 추가한다
"lint": "next lint",
"dev:lint": "concurrently \"npm run dev\" \"npm run lint\""

하면 짠 하고 로컬 띄워놓고 작업하면서 바로바로 eslint 위반 조건을 볼 수 있다.
huskey 설정을 통해 eslint 오류 해결 전까지 커밋을 못하도록 막으면 더 좋다 +_+
-> 커밋 푸시 전에는 elsint 오류를 가능하면 전부 해결하고 올리자!
~~~
<img width="945" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/025f19e2-3391-4f8d-813d-1607ce227018">

