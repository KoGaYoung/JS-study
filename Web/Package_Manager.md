~~~
패키지 매니저는 흔히 Front-End를 개발할 때 필요한 도구로, 
npm, yarn, pnpm 등이 대표적입니다.

프로젝트를 관리해주는 프로젝트 매니저(Project manager)가 있다면
레포를 관리해주는 패키지 매니저(Package manager)가 있어요 - _-+

패키지매니저는 개발의 시작과 끝을 함께하는 중요한 도구입니다.
~~~

# NPM
~~~
npm: Node.js를 설치하면 같이 설치되어있음. npx는 npm 5.2버전 이상을 말함. 

장점: 아주 다양한 패키지
npm의 단점(정확히는 npm install 패키지 -g의 단점): 보통 한번 설치한 패키지를 계속 두고 사용하는데 눈에 보이지 않아 관리가 안됨
(특히 여러 레포를 동시에 개발중인 경우 영향을 받음. 예시로 cra할 때 cra가 업데이트가 자주 일어나는 패키지라 옛날 Npm 쓰면 옛날 cra 되서 안좋음)
package.json이 일관성없이 업데이트되는 문제

npx는 항상 최신 패키지를 바라봐서 npm install 패키지명 -g의 단점을 보완해줌
결론: npx를 쓰자.
~~~

# YARN
~~~
yarn: npm의 단점들을 커버하고 성능이 향상된 패키지매니저(이후에 Npm이 많이 발전해서 성능상으로는 이제 큰 차이가 없다는 의견이 많음)

npm과 명령어가 좀 다름 (ex. npm install vs yarn add)

yarn의 경우 2.x버전부터는 yarn berry라고 하는데, PnP(Plug in Play기능으로) zero-intsall 가능하게 해줌.
PnP는 node_modules를 생성하지 않고 각 패키지 버전에 단일 참조를 생성해서 설치속도 빠름
(모노레포 구성할 경우 레포가 늘어날 수록 의존하는 패키지들이 많아 제로인스톨로 관리하면 편리. 디펜던시가 꼬인다면 고생한다..)
단점: 제로인스톨이라고 install을 아예 안하는건 아님.. 용량 짱큼

결론: 모노레포 + 제로인스톨 구성시 얀베리를 써보자.
~~~

# pnpm
~~~
그리고 새로 알게된 pnpm 

모노레포 지원함 (yarn -> pnpm 이전 작업 글들이 많음, 시장의 선택?)
내 컴퓨터 안에 패키지들을 글로벌하게 관리해줌(고스트 디펜던시 막아줌)
멀티레포인 경우 레포마다 패키지 설치를 해야했는데, pnpm은 하나의 글로벌 패키지로 관리함.
제로인스톨은 지원안하지만 하드 링크와 심볼릭 링크를 사용하여 node_modules 폴더의 크기를 줄이고 설치 시간을 단축해줌.
패키지 설치가 1회만 이뤄지기 때문에 쾌적한 내컴퓨터 가능

터보레포라는 비로 모노레포를 위한 고성능 빌드 시스템과 잘맞음
~~~

~~~
// 세줄요약) 
npm: 근본
yarn: yarn berry + mono repo + zero install
pnpm : tuborepo + mono repo + global package
~~~

## npm to pnpm 넘어가보기
~~~
빌드, 배포 속도 단축
다양한 레포에서의 일관성있는 디펜던시 설정 관점에서 적용해보기

~~~