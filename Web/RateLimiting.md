## 디바운스와 쓰로틀링
~~~
디바운스(debounce): 일정 시간동안 요청이 온 것 중 '처음이나(리딩엣지), 마지막(트레일링엣지)' 온것만 보낸다!
-> 검색할 때 많이 사용한다. 입력 할 때마다 바로바로 요청을 보내면 cpu사용량 너무많아지고 화면이 아주 어지럽게 바뀐다.
약 300ms줘서 사용하면 좀 더 나은 사용자 경험을 얻을 수 있다.

쓰로틀링(throttling): 일정 시간동안동안 호출 횟수를 조정한다는 뜻이다!
일정 시간동안 이벤트를 1번만 발생시킨다,
일정시간이지나면 이벤트를 또 발생시킨다.
-> 스크롤 이벤트에 많이 사용한다. 특히 ios환경에 트랙패드나 매직마우스는 스크롤이벤트가 엄청 많이탄다.....- _-
이벤트 횟수를 적당히 조절할 때 사용했다 (무한스크롤 등)

이벤트 호출 횟수를 제안함으로써 최적화와 사용자 경험을 둘 다 잡을 수 있다.
-> 리딩엣지 디바운싱와 쓰로틀링은 일정시간동안 모든 요청을 무시한다는 것에서 동일하게 동작하는것 보이나,
리딩엣지 디바운싱은 설정한 시간안에 모든요청을 무시함
쓰로틀링은 지속적인 요청이 들어올 경우 타이머 시간이 자니면 요청을 허용함
~~~

## 디바운스 사용 에제
~~~javascript
const $input = document.querySelector('#input');
const $app = document.querySelector('#app');

const callAjaxRequest = (e) => {
    $app.insertAdjacentHTML('beforeend', `<p>ajax 요청: ${e.target.value}</p>`);
}

const debounce = (func) => {
    // 클로저를 이용해 내부에 timer 변수 선언
    let timer;

    return (e) => {
        if (timer) {
            // 현재 돌아가고있는 타이머를 취소함
            clearTimeout(timer);
        }

        // 500ms의 타이머를 다시 실행함
        timer = setTimeout(func(e), 500);
    }
}

$input.addEventListener('input', debouce(callAjaxRequest));
~~~
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
const $app = document.querySelector('#app');
const $countNumber = document.querySelector('#input');
let count = 0;

const throttle = (func) => {
    // 내부에 timer 변수 선언
    let timer;

    return () => {
        // timer가 없을시에만 실행
        if (!timer) {
            timer = setTimeout(() => {
                timer = null;
                // 1000ms가 지난 후에 timer를 초기화와 동시에 지정해놓은 함수 실행
                func();
            }
        }, 1000;
    }
}

const increaseCountNumber = () => {
    $countNumber.innerText = count++;
}

$app.addEventListener('scroll', throttle(increaseCountNumber));
~~~
~~~javascript
import { throttle } from 'lodash/es';

const handleScroll = throttle(() => {
    console.log('1초마다 스크롤);
}, 1000);

window.addEventListener('scroll', handleScroll);
~~~
