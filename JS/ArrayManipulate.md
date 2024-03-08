## 문제)

```
react에서 Map으로 리스트를 렌더링할때 필수적으로 key값을 지정해줘야한다.
그런데 key를 index로 지정하면 리렌더링에 불리하다고 공홈에서 읽은적이 있다.

왜 불리할까? 브라우저 렌더링과정과 같이 생각해보자
```

```js
import React from 'react';

function ItemList({ items }) {
  return (
    <div>
      {items.map((item, index) => (
        {/* 여기서 인덱스를 key로 사용 */}
        <div key={index}>
          {item}
        </div>
      ))}
    </div>
  );
}

export default ItemList;
```

~~~
items props는 길이가 10인 아이템이다!
만약 7번째 아이템이 삭제 된다면!?
8~10번째 아이템의 위치와 key={index}값이 변경됨.

8~10번째 아이템들이 전부 리랜더링이됨!

만약 키값을 고유한 값으로 설정했다면?!

1~6은 여전히 그대로고,
7번째 아이템이 사라지고, 8~10번째 아이템은 고유한 key값이 있기 때문에 리랜더링은 일어나지 않음
다만 8~10번째 리스트의 위치 조정으로 reflow, repaint는 일어남

~~~
~~~
react는 css tree + dom tree = render tree => diff알고리즘 -> 실제 변경된 내용만 업데이트 -> attachment -> reflow, repaint 함.

diff알고리즘 -> 실제 변경된 내용만 업데이트한다는 내용임.
~~~

~~~
최적화 하려면, index값을 key로 사용하지않고 고유값을 사용해야한다.
고유값을 갖기 힘든 input같은경우 uuid를 많이 혼용해서 사용한다
~~~

~~~
추가로 
ItemList의 render 함수 내에 있는것보다 독립적인 컴포넌트로 존재할 때,
Item 컴포넌트가 독립적으로 렌더링되기 때문에  특정 아이템에 변경이 있을 때 해당 Item 컴포넌트만 리렌더링되는 이점이 있음

~~~
~~~js
import React from 'react';
import {uuid} from 'uuid4';

function ItemList({ items }) {
  return (
    <div>
      {items.map((item, index) => (
        {/* 여기서 인덱스를 key로 사용 */}
        <Item key={`index_${uuid()}`} data={item}/>
      ))}
    </div>
  );
}

export default ItemList;
~~~

~~~
불변성을 유지하기 위해 우리는 개발하면서 배열 복사를 하는 경우가 잦습니다.
그중에서도 마지막 뎁스까지 복사해주는 깊은 복사 deep copy를요! (<-> 얕은복사 swallowcopy)

대표적으로 3가지 정도의 방법이 있습니다.
1. 직접 for 문을 돌며 복사하기 + copy || spread 연산 사용
2. stringfy() 이용하기
3. lodash(es) 이용하기

결론부터 말하면 팀원과의 협업을 한다면 lodash를 사용한 deepcopy로 통일할 것 같습니다.
1번, 2번이 성능은 효율적일 수 있지만, "바퀴의 재발명" 이라는 것 처럼 매번 동료가 해당 코드를 읽고 해석해야하는 번거로움이 있습니다.

만약 혼자 개발한다고 가정하면, 1, 2번을 써도 되나요? 쓴다면 어떤걸 쓸 건가요?
-> 먼저 성능면에서 본다면 flat 한 구조를 가진 객체의 경우 spread 연산자가 빠르고 직관적입니다.
-> 하지만 중첩된 구조를 가진 복잡한 객체라면 JSON.parse()가 성능이 더 좋습니다.
-> JSON.parse의 경우 날짜 데이터의 경우 parse()를 하며 데이터를 잃어버릴 수 있습니다. (이경우엔 lodash/es deepcopy를 사용하는게 적절해보입니다)

따라서 정리해보면
1. 플랫한 데이터의 경우 spread 연산 사용
2. 중첩된 구조를 가진 데이터의 경우
  2.1. 동료와 협업하는 경우 lodash/es deepcopy를 권장. 반복문을 사용한 spread 연산은 바퀴의 재발명...! (하지맙시다)
  2.2. 혼자 개발한다면, 복잡한 데이터의 경우 날짜 데이터가 없는것이 확실하다면 JSON.stringfy() 활용가능. 확인되지 않는다면 lodash/es deepcopy 사용

~~~
