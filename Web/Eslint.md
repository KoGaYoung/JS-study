## eslint / prettier
~~~
팀원들과 코드를 같이 짜는데 있어서 모두 코드를 짜는 스타일이 다르면 변경점 파악이 어렵고 소스코드 충돌이 자주 발생합니다 :-(

이를 해결하깅 위해 컨벤션을 통해 규칙을 정하고, 최소한의 변경점만 가져가게 되는데요.
코드 품질을 높일 수 있습니다 +_+
eslint와 prettier를 설정하여 주로 문제를 해결합니다!

eslint: js나 ts 문법이나 규칙에 위배되지 않는지 검사
prettier: 줄바꿈, 스페이스 등 코딩에디터(intelliJ, vsCode)에서 일관성있게 작성하도록 도와주는 도구
~~~

## 설정을 시작해보자
~~~
패키지매니저로 eslint, prettier를 설정할때는 꼭 --dev 옵션으로 개발환경에서만(devDependencies) 사용할 수 있도록 합니다 :-)
~~~

#@ eslint
~~~
esLint의 경우 주로 airbnb 컨벤션을 사용합니다!
(standard, google 등 다른 컨벤션들도 존재합니다 하지만 React, ES6에도 적용되고 사용량이 가장 많아서 에어비앤비를 추천합니다)

1. 프로젝트 루트 디렉토리에서 elsint 설치
npm install eslint --save-dev
(ex. npx install eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-react-hooks eslint-plugin-jsx-a11y --save-dev)
(타입스크립트 사용 시 npm install @typescript-eslint/eslint-plugin @typescript-eslint/parser --save-dev)

2. eslint 설정 시작
npx eslint --init

✔ How would you like to use ESLint? · style (fix까지 해줘요)
✔ What type of modules does your project use? · esm 
✔ Which framework does your project use? · react
✔ Does your project use TypeScript? · [No] / Yes (no를 해야 airbnb가 나와요)
✔ Where does your code run? · browser
✔ How would you like to define a style for your project? · guide
✔ Which style guide do you want to follow? · airbnb
✔ What format do you want your config file to be in? · JavaScript (.eslintrc파일 js확장자로)
Checking peerDependencies of eslint-config-airbnb@latest
The config that you've selected requires the following dependencies:

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
-> 커밋 푸시 전에는 elsint 오류를 가능하면 전부 해결하고 올리자!
~~~
<img width="945" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/025f19e2-3391-4f8d-813d-1607ce227018">

