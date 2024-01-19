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