~~~
웹의 기본적인 구조와 동작방법을 이해해봅시다
~~~

# 브라우저 구성요소

![image](https://github.com/KoGaYoung/JS-study/assets/36693355/0a0d672b-6254-451e-8f93-69a87f88339c)

~~~
사용자 인터페이스: 주소검색창, 버튼 등 GUI
브라우저 엔진: 위에꺼랑 아래꺼 사이 동작 제어
렌더링 엔진: Html, css 파싱해서 화면에 그려줌. 브라우저마다 사용하는 엔진 다름 (그때문에 크로스브라우징 발생)

자료저장소: 로컬스토리지, 쿠키 등
통신: http, https 통신
js엔진: 자바스크립트 v8 엔진
ui 백앤드: input, button, select 등 기본적인 위젯 그리는 인터페이스
~~~

# 화면에 요소들이 그려지는 방법
~~~
css Tree + Dom Tree = (합치기(Attachment)) => render Tree -> 정확한 위치와 픽셀 크기를 계산 (배치(Layout) or Reflow) -> 진짜 화면에 그리기(patining)
~~~

# reflow
~~~
HTML요소의 크기나 위치 등의 레이아웃 수치가 변하면 해당 요소의 영향을 받는 자식 노드나 부모 노드들을 포함하여 Layout(Reflow)과정을 다시 수행
- DOM 노드의 추가, 제거
- DOM 노드의 위치 변경
- DOM 노드의 크기 변경(margin, padding, border, width, height 등..)
- CSS3 애니메이션과 트랜지션
- 폰트 변경, 텍스트 내용 변경
- 이미지 크기 변경
- offset, scrollTop, scrollLeft과 같은 계산된 스타일 정보 요청
- 페이지 초기 렌더링
- 윈도우 리사이징
~~~

# repaint
~~~
화면의 구조가 변경되지 않는 화면 변화의 경우 Repaint만 발생한다. 예를 들면 opacity, background-color, visibility, outline 등의 스타일 변경 시에는 Repaint만 동작한다.
~~~

# 주소창에 naver 를 치면 무슨 일이 일어날까
~~~
1. 도메인 요청 naver.com (프로토콜은 디폴트가 http임)
2. DNS 서버에서 ip 주소로 바꿔줌 125.209.222.141 (통신사 DNS에서 캐시있으면 바로 리턴 없으면 루트DNS 접속해서 찾아옴)
3. 얻어온 IP주소에 페이지 달라고 http 요청을 TCP로 보냄 (요롷게)
~~~
<img width="1019" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/19409059-4185-43bb-8bdc-fe6d293be2b2">

~~~
4. 필요한 html과 js파일들을 받아온다. (csr이면 빈 html를 줄꺼고 ssr이면 완성된 html를 주겠지...)
5. 받아온 화면으로 요소 그리기 (랜더트리 만들어서 배치 페인트!)
~~~

# 가끔 HTML만 보여지다가 CSS가 갑자기 뿅 하고 나오는 경우가 있다. (FOUC, Flash of Unstyled Content) 현상 왜 일어날까?
~~~
https://velog.io/@soorokim/%EB%A0%8C%EB%8D%94%EB%A7%81-%EA%B3%BC%EC%A0%95
https://web.dev/articles/critical-rendering-path/render-blocking-css?hl=ko
이 글을 읽으며 CSS트리와 DOM트리가 발생하기 전에는 Render 트리가 발생할 수 없다는걸 알게되었다.

그런데 도대체 FOUC가 발생하는걸까?
분명 css는 렌더링 차단 리소스로 취급되는데말이다.
GPT한테 답을 좀 구해보았다. 이유는 CSS 파싱이 지연될 경우 FOUC가 발생할 수 있다고 하는데,
아무래도 CSS 스타일시트는 일반적으로 HTML 문서가 로드된 후에 로드됩니다
CSS 스타일시트는 일반적으로 HTML 문서가 로드된 후에 로드되는데, CSS 스타일시트는 완전히 파싱되기전까지는 동작하지 않는다.
그래서 브라우저가 HTML 구문을 분석하는 동안에는 스타일업이 문서를 보여주고, css 스타일 시트가 파싱이 완료되면 화면에 스타일을 보여준다.

아직도 풀리지 않는 궁금증은 스타일시트가 완전히 파싱되어야만 스타일이 적용된 dom을 보여준다는데,
CSS트리와 DOM트리가 발생하기 전에는 Render 트리가 발생할 수 없다는 말은
창과 방패같다.

아무래도 스타일 시트가 파싱되는 속도에 따라
속도가 빠르면 css트리, dom트리 둘다 만들어서 화면에 그리고,
속도가 느리면 dom을 먼저 그리는것같다.
(하지만 스타일이 적용되지 않은 화면은 사용자경험이 안좋기때문에 css와 dom을 둘다 불러오는거잖아! -> 그래서 css가 차단리소스인거잖아!)
(하지만 사용자 경험 측면에서 생각해보면 html 보여주기 vs 빈화면 보여주기 했을땐 html만이라도 보여주는 선택을 무조건 할것이다.)
사용자의 인터넷 연결 속도나 컴퓨터의 처리 능력에 따라 다르기 때문에 그 기준은 명확하게 알 수 없어서 아쉬웠다.

Fouc를 해결하기 위한 방법으로는
1. Head 태그에 스타일을 배치하여 style 먼저 불러오도록 함
2. Async와 defer를 사용하여 비동기로 처리하는 방법이 있다.
~~~
