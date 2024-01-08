~~~
에러를 해결하다보면 "캐시때문에 그래요"등의 이야기를 들을 수 있다.
정확히 캐시를 이야기 하는것은 아니지만, 캐시처럼 브라우저에 값을 저장해놨다다 사용할 수있는 스토리지도 3가지나 있다.
알아보자!
~~~

---
## 캐시먼저 알아보자
~~~
캐시는 웹 페이지를 빠르게 로드하기 위해 HTML, CSS, JS, 파일, 이미지 등의 리소스를 임시로 저장하는 공간이다
로드 속도 향상과 트래픽 감소에 도움을 준다!
~~~

## 캐시는 어떻게 관리해?
~~~
브라우저 동작원리 페이지에서 학습했던 내용대로 방문하려는 페이지의 도메인(http://domain.com)에 접속하면,
HTML, CSS, 파일, 이미지, JS등을 내려준다.

서버에서 HTTP의 헤더에 캐시를 컨트롤할 수 있는 정책을 넣을 수 있다!(몰랐다)
'Cache-Control' 헤더에 캐시가능여부, 만료시간(max-age), 재검증필요(revalidate) 지정가능하다.
이걸보고 사용자의 하드 드라이브나 메모리에 저장한다 (진짜로 몰랐다)
서버는 no-cache나, must-revalidate 옵션을통해 캐시 안되게도 설정할 수있다.

시간이 만료되거나, 사용자가 삭제하거나, no-cache 옵션이나, 오래된캐시, 세션과 연관된 캐시는 삭제될 수 있다.
~~~

---
## 브라우저 스토리는?
~~~
브라우저 스토리지는 사용자의 데이터를 저장할 때 사용한다. (e.g.사용자의 선호도, 로그인 상태, 페이지 내 사용자의 작업 등)
~~~

## 내가만든 쿠키...사용자의 편의성을 위해 구웠지 
~~~
작은 데이터 조각
주로 세션관리 개인화 추적에 사용(나의 경우 팝업 오늘 다시보지 않기에 가장 많이 사용했다)
만료날짜가 있으며 만료시간이 지나면 없어져용

// 사파리에서 Date 객체 만들어서 사용햇는데 yyyy-mm-dd형태 를 지원하지 않아서 Invaildate 오류나서 수정했던 기억이있음
// 분기처리필요
~~~

~~~javascript
const setCookie = (name, value, days) => {
  let expires = "";

  if (days) {
    let date = new Date();
    expires = "; expires=" + date.toUTCString();
  }

 document.cookie = name + "=" + (value || "")  + expires + "; path=/";
}
~~~

## 로컬스토리지
~~~
데이터 영구저장 가능! (주로 set 전에 삭제소스를 하나 넣어주는게 좋음)
세션상태, 선호도 등에 사용됨(나의경우 api 데이터 빵꾸났을때 활용했다. api데이터가 느리게 오는경우에도 가라데이터처럼 활용할 수 있다)
5MB 내외의 데이터를 저장할 수 있다.
~~~

~~~javascript
localStorage.removeItem('key'); // key가 존재하지않아도 오류안남 약간 removeEventListenr의 느낌이랄까..
localStorage.setItem('key', 'value');

const localitem = localStorage.getItem('key');
~~~

## 세션스토리지
~~~
데이터 영구저장 가능, 단! 브라우저가 닫히면 사라짐
주문 입력폼같은거 저장하면 좋을것같다.

도메인별로 데이터를 저장한다!
~~~

~~~javascript
sessionStorage.removeItem('key'); // key가 존재하지않아도 오류안남 약간 removeEventListenr의 느낌이랄까..
sessionStorage.setItem('key', 'value');

const sessionItem = sessionStorage.getItem('key');
~~~
