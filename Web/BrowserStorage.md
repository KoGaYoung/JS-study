# 브라우저 스토리지
~~~
에러를 해결하다보면 "캐시때문에 그래요"등의 이야기를 들을 수 있다.
정확히 캐시를 이야기 하는것은 아니지만, 캐시처럼 브라우저에 값을 저장해놨다다 사용할 수있는 스토리지가 3가지 있다.

(개발자도구랑 같이 다루려다 너무 분량이 많아질 것 같아 따로 다룬다)
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
~~~

~~~javascript
sessionStorage.removeItem('key'); // key가 존재하지않아도 오류안남 약간 removeEventListenr의 느낌이랄까..
sessionStorage.setItem('key', 'value');

const sessionItem = sessionStorage.getItem('key');
~~~
