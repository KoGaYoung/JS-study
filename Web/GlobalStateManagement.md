# Front-End 생태계에 상태관리 라이브러리가 쏟아지고있다.
### 근본 Redux, MobX -> recoil, zustand, zotai

## 이력 or 트랜드
~~~
Redux, MobX의 경우 컴포넌트로 이루어진 프론트앤드 라이브러리, 프레임워크의 전역상태를 관리하기 위해 나왔다.
하지만 보일러플레이트가 많아서 이를 해결하기 위해 recoil이나왔고,

이후 전역상태관리 스토어에 api 관련소스들이 큰 부분을 차지하면서 React-query를 많이 도입하기 시작한것같다.
(서버데이터를 입맛대로 가져오기위한 graphQL과는 좀 다르다, react-query랑 같이쓰면 좋다고한다)

react-query를 도입하면서 조금 단순한 zustand, zotai로 많이 넘어갔다.
아래는 상태관리 라이브러리의 특징과 장단점을 알아두자(항상 정답은 없고 프로젝트별 최선만 있을뿐..)

그리고 최근 상태관리 라이브러리들이 성능개선했다고하는데 useSyncExternalStore를 사용한 기능들을 주로 활용해서 성능개선을 했다고한다.


https://yozm.wishket.com/magazine/detail/2233/
~~~

### Redux


### Mobx


### recoil


### zustand


### zotai
