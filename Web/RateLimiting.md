## 디바운스와 쓰로틀링
~~~
디바운스(debounce): 일정 시간동안 요청이 온 것 중 마지막에 온것만 보낸다!
-> 검색할 때 많이 사용한다. 입력 할 때마다 바로바로 요청을 보내면 cpu사용량 너무많아지고 화면이 아주 어지럽게 바뀐다.
약 300ms줘서 사용하면 좀 더 나은 사용자 경험을 얻을 수 있다.

쓰로틀링(throttling): 일정 시간동안 요청이 온 것 중 첫번째 것만 보내고 나머지는 무시한다.
-> 스크롤 이벤트에 많이 사용한다. 특히 ios환경에 트랙패드나 매직마우스는 스크롤이벤트가 엄청 많이탄다.....- _-
이벤트 횟수를 적당히 조절할 때 사용했다

이벤트 호출 횟수를 제안함으로써 최적화와 사용자 경험을 둘 다 잡을 수 있다.
~~~

## 디바운스 사용 에제
~~~javascript
// es로 가져와야 트리쉐이킹하면서 모듈 최적화됨(작지만 성능향상)
import { debounce ) from 'lodash/es';

// 직접 구현할 수 있지만 lodash에 있는거 가져다 쓰면 편하다..!
const handleSerach = debounce((e) => {
    console.log('검색', e.target.value);
}, 300);

<input type="text" onKeyUp={handleSerach} /> 
~~~

~~~javascript
import { throttle } from 'lodash/es';

const handleScroll = throttle(() => {
    console.log('1초마다 스크롤);
}, 1000);

window.addEventListener('scroll', handleScroll);
~~~
