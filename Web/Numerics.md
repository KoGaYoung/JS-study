# 빅인트
~~~
자바스크립트는 15자리 이상( 2^53 - 1 )을 원시타입 Number로 표현할 수 없다.
~~~

# 처리방법
~~~
big.js 라이브러리 활용

사칙연산자(+, -, *. /) 대신 (add, minus, mul, div)
덧셈  : .add(x)
뺄셈  : .minus(x)
곱셈  : .mul(x)
나눗셈: .div(x)
~~~
~~~javascript
import Big from "big.js"

export const ToBigNumber = (val) => {
    let rtnVal = Big(0)
    rtnVal = val
    if (isNaN(rtnVal) || rtnVal == null || rtnVal == undefined || rtnVal == "" || rtnVal == Infinity || rtnVal == -Infinity) {
        rtnVal = Big(0)
    }

    return Big(rtnVal)
}

let mn_1 = ToBigNumber(0);
let mn_2 = ToBigNumber(1000);
let mn_3 = ToBigNumber(3);

let mn_4 = (mn_1 + mn_2) / 3
let mn_4 = (mn_1.add(mn_2)).div(mn_3) 
let mn_4 = toBigNumber((mn_1.add(mn_2)).div(mn_3))
~~~
