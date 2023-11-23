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



~~~
