# CORS
~~~
아직까지 개발하다가 CORS 오류를 마주친 적은 없다.
Sentry 오류 중 리액트 라우터 버전 관련 오류를 해결하다가 CORS일 가능성이 있다는 말에 보안 관련 오류구나 찾아본 적이 있다.
하지만 언젠가는 반드시 마주칠 오류기에 미리 알아두자

CORS는 오류가아니라 자원을 안전하게 가져오는 방법이다!
~~~

~~~
웹 브라우저는 http 요청에 따라 다른 정책을 가짐 -> 정책을 위반하면 CORS 오류가 발생함

1. <img> <vedio> <script> <link> Cross-Origin 정책 지원함

2. XMLHttpRequest, fetch는 Same-Origin 정책 지원함
~~~

##예제
<img width="1233" alt="image" src="https://github.com/KoGaYoung/JS-study/assets/36693355/a69298d6-523b-4c37-bff1-1ccbf316ed58">

# CORS 해결법
~~~
CORS는 Cross Origin Resource sharing 의 약자이다

그니까 Origin이 같으면 된단 뜻이다. 
Origin = 프로토콜 + 도메인 + 포트 인데 window.location.origin 이랑 같다.

프로토콜 : http / https
도메인: google.com / naver.com
포트: 3000

다른 Origin을 가지고 있더라도 리소스를 가져오는 방법
1. 클라이언트는 요청 헤더 Origin에 Origin 정보 담아서 보냄 ex. Origin: http://a.com
2. 서버는 응답헤더에 Access-control-allow-origin : 허용할 origin 정보를 표기함 ex. Access-Control-Allow-Origin : http://a.com (서버작업이 필요하긴하다)
3. 웹 브라우저가 요청헤더, 응답헤더 Access-control-allow-origin 값을 보고 허용해줌
~~~
