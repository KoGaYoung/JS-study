## 목표
~~~
State를 잘 구성하는 방법
State 업데이트 로직을 유지보수하기 용이하게
멀리있는 컴포넌트간에 State를 공유하는 방법
~~~

~~~
3-1. State를 사용해 Input 다루기
3-2. State 구조 선택하기
3-3. 컴포넌트간 State 공유하기
3-4. State를 보존하고 초기화하기
3-5. State로직을 리듀서로 작성하기
3-6. Context를 사용해 데이터를 깊게 전달하기
3-7. Reducer와 Context로 앱 확장하기
~~~

3-1. State를 사용해 Input 다루기

~~~
리액트는 state를 통해 dom을 업데이트(랜더링)해주기 때문에, 
바닐라에 비해 훨신 선언적으로 사용할 수 있습니다.

개발자는 화면에 변경될 내용을 state로 정의하고,
state의 변화를 트리거하는 기능을 setState로 정의하여
트리거와 setState를 바인딩함으로써 선언적으로 사용할 수 있습니다.
~~~

3-2. State 구조 선택하기
~~~
state를 만들 때 고민해볼 것
1. state가 역설을 일으키지 않는지
  = 동시에 발생할 수 없는 상태값은 한 변수로 관리 가능
2. 이미 같은 값을 다른변수에서 가지고 있는지.
 = 불필요한 데이터는 굳이 state로 만들거나/추가하지 않습니다.
3. 한 번에 변경되어야 하는 state는 하나로 묶어보기
4. 깊게 중첩된(nested) state는 피하기. 가능하면 flat된 것이 좋습니다.
~~~

3-3. 컴포넌트간 State 공유하기
~~~
각각의 자식 컴포넌트들이 각자 동작하면, state는 자식에서 관리해야하지만.

두 자식 컴포넌트가 동시에 수정되어야한다면, state를 부모컴포넌트로 끌어올려서 사용할 수 있습니다.
state, setState역할을 하는 데이터와 함수를 자식컴포넌트로 전달하고,
자식컴포넌트는 부모의 state와 setState를 props로 받아서 변경된 props를 통해 화면을 그리고 
트리거되면 부모의 상태값을 setState로 변경시킬 수 있습니다.

만약 부모컴포넌트 수준이 아닌 조상님 컴포넌트라면, 전역 상태를 고려해볼 수 있습니다. 
~~~

~~~
컨트롤드 컴포넌트, 언컨트롤드 컴포넌트.
기술적 용어라기보다는 리액트 컴포넌트를 이해하는 개념.

컨트롤드는 부모로부터 props를 받아서 부모에의해 조작되는 컴포넌트

언컨트롤드컴포넌트는 자체의 state만 가지고 상태가 관리되는 컴포넌트

~~~


3-4. State를 보존하고 초기화하기
~~~
1. 리액트는 위치로 state를 분리합니다.

render(
  <Counter />
  <Counter />
)

이처럼 같은 Counter를 동일하게 두 번 렌더링 하더라도 
Counter 각각 고유의 state를 가지게 됩니다.

2. 또한 state는 컴포넌트가 렌더링을 유지하는동안 state값을 유지합니다.
컴포넌트가 언마운트된다면 state는 값을 잃어버립니다. 
다시 렌더링하면 초기화된 값을 확인할 수 있습니다.


3. 같은자리의 Component는 state를 유지할 수 있습니다.
2번에서 분명 언마운트된다면 state는 값을 잃어버린다했는데 위치로 유지한다는건 무슨소린지 싶습니다.
아래 아주 확실한 예제가 있습니다...

render (
  isStyled ? 
    <Counter />
    :
    <Counter />
)

삼항연산자로 "같은" 컴포넌트를 호출한다는 것은, 같은 위치에 렌더링한다는 의미와도 같습니다.
따라서 isStlyed(boolean) 값이 변경되더라도, (=언마운트 후 렌더링되더라도)
state값을 유지합니다.
(삼항연산자가 아니여도, Counter 컴포넌트 위치가 같다면 state를 유지합니다.)

같은 위치에 렌더링하는것처럼 보이지만,
이 경우는 다른위치로 인식합니다.
render(
  {!isFancy && <Counter />}
  {isFancy && <Counter />}
)

단, 같은위치의 다른 컴포넌트라면 state가 초기화됩니다.
render (
  isStyled ? 
    <p>is loading...</p>
    :
    <Counter />
)

같은컴포넌트를 렌더링하더라도, 감싸는 부모컴포넌트가 다르면 state를 유지할 수 없습니다.
state를 유지하고싶다면 트리구조가 같아야합니다.

render (
  isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )
)

4. 같은위치, 같은 컴포넌트지만 언마운트 시 초기화 되는 로직을 구현하고 싶다면?
 -> 다른 위치에 렌더링하기
 -> key를 이용하여 다른컴포넌트라고 리엑트에 알려주기
~~~


~~~
정은님께서 신기한걸 발견해주셨다.
같은위치에 같은 컴포넌트를 

<>는 렌더링할때 사라지지만, 
리액트 트리구조가 다르다고 인식하기때문에 값이 보전이 안된다!
render(
  isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
      <>
      <Counter isFancy={true} /> 
      </>
)

~~~

~~~
리액트에서는 위치를 기반으로 같은컴포넌트인지 확인한다고 해. key를 주면 같은 위치더라도 다른 컴포넌트로 인식하게 만들 수 있어. 그렇다면 반대로 다른위치더라도, 같은 Key를 사용하면 같은 컴포넌트로 인식하여 state값을 보존할 수 있을까?? 


네: 리액트에서는 key가 컴포넌트의 고유 식별자로 사용되며, 동일한 key가 있으면 해당 컴포넌트를 재사용합니다. 따라서 같은 key를 사용하면 다른 위치에서도 동일한 컴포넌트로 인식하여 state 값을 보존할 수 있습니다.
~~~


### 3-5 state 로직을 reducer로 작성하기
~~~
컴포넌트 안에서 state 변경이 많아진다면, 로직 복잡도가 올라갑니다.
이 때 리듀서를 사용하면 state 변경 로직을 외부 한 군데서 관리할 수 있습니다.
~~~

~~~jsx
// 원본 코드
const TaskApp = () => {
  const [tasks, setTasks] = useState(initialTasks);

  /*
   * 추가 함수
   */
  const handleAddTask = (text) => {
    setTasks([...tasks, {
      id: nextId++,
      text: text,
      done: false
    }]);
  }

  /*
   * task를 토글하거나 저장,
   */
  const handleChangeTask = (task) =>{
    setTasks(tasks.map(t => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    }));
  }

  /*
   * 삭제함수 
   */
  const handleDeleteTas = (taskId) => {
    setTasks(
      tasks.filter(t => t.id !== taskId)
    );
  }

  return (
    <>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
~~~

~~~jsx
// step1 원본 코드 -> state 변경 분리

/**
* 추가
*/
const handleAddTask = (text) => {
  dispatch({
    type: 'added', // 타입은 보통 문자열 
    id: nextId++,
    text: text,
  }); // dispatch 안 내용물이 action 객체
}

/*
* task를 토글하거나 저장,
*/
const handleChangeTask = (task) => {
  dispatch({
    type: 'changed',
    task: task
  });
}

/**
* 삭제
*/
const handleDeleteTask= (taskId) => {
  dispatch({
    type: 'deleted',
    id: taskId
  });
}
~~~

~~~jsx
// step 2 리듀서 함수 작성 

// 주로 두번 째 인자에 type 기재
import { useReducer } from 'react';

const TaskApp = () => {
// state 변화함수, 초기값
 const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
 
 
 return (...);
}
// task 는 state, action은 action 객체
const tasksReducer = (tasks, action) => {

 // 주로 switch문 사용 
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
~~~

~~~
리듀스 장점: 핸들러 (setState) 코드가 비슷하다면, 코드량 줄어듬
테스트(or 디버깅), 가독성: 코드를 파일로 분리가능, 가독성 좋아짐.

리듀스는 필요한 경우에 적절히 사용합시다.
~~~

~~~
스터디 이후로 알게된 사용방법: 유닛테스트 관점에서는 괜찮은것같다.

~~~

### 3-6 Context를 사용해 데이터를 깊게 전달하기
~~~
프롭스 드릴링을 막기위해 useContext를 활용해보자. 에대한 내용입니다.
~~~

~~~jsx
// 1. context 생성하기
// levelContext.tsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
~~~

~~~jsx

// 2. context 활용하기

// LevelContext.tsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
~~~

~~~jsx
// Heading.tsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
~~~

~~~jsx
// Section.tsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section"> // provider로 감싸기
      <LevelContext.Provider value={level}>
        {children} {/* Hading.tsx */
      </LevelContext.Provider>
    </section>
  );
}
~~~

~~~jsx
// Page.tsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
~~~

~~~
컨택스트를 사용했을 때 최적화를 해주지 못하나요?
컴포넌트 1 -> 컴포넌트 2 -> 컴포넌트 3 구조로 되어있을 떄,
컴포넌트 1,3에서만 컨택스트의 state를 사용한다면, 컴포넌트 2는 렌더링 다시 되나요?안되나요?
했을 때 컴포넌트 2도 렌더링 될 수 있습니다.
컨택스트는 렌더링 비효율이 생길수있다.
프롬스 드릴링만 피할 수 있다는 장점만 있다고본다.

하지만 mobx에서는 observer()함수로 감싸진 예제에서만 전역상태를 바라보기때문에
컴포넌트 2를 렌더링 안하게 할 수 있습니다.
~~~

~~~
어디서든 사용가능.
사용 예제: 
- 다크모드 같은 테마 지정
- 사용자 로그인 상태
- 라우팅
- 상태관리

mobX를 사용할때 많이 사용해본 패턴.
~~~
### 3-7 Reducer와 Context로 앱 확장하기
~~~
둘 다 사용하기. 같은내용이라 패스합니다.
~~~
~~~jsx
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {  
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // 네트워크에 접속한다고 가정해봅시다.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'istanbul') {
        resolve();
      } else {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      }
    }, 1500);
  });
}

let form = document.getElementById('form');
let textarea = document.getElementById('textarea');
let button = document.getElementById('button');
let loadingMessage = document.getElementById('loading');
let errorMessage = document.getElementById('error');
let successMessage = document.getElementById('success');
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;

~~~
~~~html
<form id="form">
  <h2>City quiz</h2>
  <p>
    What city is located on two continents?
  </p>
  <textarea id="textarea"></textarea>
  <br />
  <button id="button" disabled>Submit</button>
  <p id="loading" style="display: none">Loading...</p>
  <p id="error" style="display: none; color: red;"></p>
</form>
<h1 id="success" style="display: none">That's right!</h1>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
</style>

~~~

~~~
첫 번째: 컴포넌트의 다양한 시각적 state 확인하기 -> UI의 모든 “state”를 시각화

Empty: 폼은 비활성화된 “제출” 버튼을 가지고 있다.
Typing: 폼은 활성화된 “제출” 버튼을 가지고 있다.
Submitting: 폼은 완전히 비활성화되고 스피너가 보인다.
Success: 폼 대신에 “감사합니다” 메시지가 보인다.
Error: “Typing” state와 동일하지만 오류 메시지가 보인다.

두 번째: 무엇이 state 변화를 트리거하는지 알아내기 
버튼을 누르거나, 필드를 입력하거나, 링크를 이동하는 것 등의 휴먼 인풋
네트워크 응답이 오거나, 타임아웃이 되거나, 이미지를 로딩하거나 하는 등의 컴퓨터 인풋
                                                                                                                                                               
세 번째: 메모리의 state를 useState로 표현하기 
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);

const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);

네 번째: 불필요한 state 변수를 제거하기 
state가 역설을 일으키지는 않나요?
다른 state 변수에 이미 같은 정보가 담겨있진 않나요?
다른 변수를 뒤집었을 때 같은 정보를 얻을 수 있진 않나요? 

const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'

다섯 번째: state 설정을 위해 이벤트 핸들러를 연결하기 
~~~

~~~js
// 완성된 react 컴포넌트
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>That's right!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // 네트워크에 접속한다고 가정해봅시다.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}


~~~
### 챌린지 1 of 3: CSS class를 추가하고 제거하기 
~~~js
import react, {useState} from 'react';

export default function Picture() {
  const [isActive, setIsActive] = useState(false);
  const handleClick= () => {
    setIsActive(isActive => !isActive);
  }
  return (
    <div className={`background ${!isActive ? "background--active" : ""}`}>
      <img
        className={`picture ${isActive ? "picture--active" : ""}`}
        alt="Rainbow houses in Kampung Pelangi, Indonesia"
        src="https://i.imgur.com/5qwVYb1.jpeg"
        onClick={handleClick}
      />
    </div>
  );
}

~~~

### 챌린지 1 of 2: 이름 입력하기
~~~js
import react, {useState} from 'react';
export default function EditProfile() {
  const [isEdit, setEdit] = useState(false);
  const [name, setName] = useState({
    firstName: 'Jane',
    lastName: 'Jacobs'
  });

  /**
  * 수정, 저장 버튼이벤트
  */
  const handleClick = () => {
    setEdit(isEdit => !isEdit);
  }

  const changeName = (value, flag) => {
    if (flag === 'fisrt') {
      setName({...name, firstName: value});
    }
    else {
      setName({...name, lastName: value});
    }
  }
  
  return (
    <form
      onSubmit={e => {
        // 이 부분을 틀렸다!
        // 폼을 제출할 때 웹 브라우저의 기본 동작은 폼 데이터를 서버로 보내고 페이지를 새로고침하는 것인데
        // 이걸막아줌
      e.preventDefault();
    }}>
      <label>
        First name: 
          {
            !isEdit ?
              <b>{name.firstName}</b>
            :
              <input value={name.firstName} onChange={(e) => changeName(e.target.value, 'first')} />
          }
      </label>
      <label>
        Last name:
        {
          !isEdit ?
            <b>{name.lastName}</b>
          :
            <input value={name.lastName} onChange={(e) => changeName(e.target.value, 'last')} />
        }
      </label>
      {/* 버튼 타입을 지정해주지않으면 submit이 기본동작, 
      submit 타입은 form의 onSubmit 함수를 트리거함.
      type을 button으로하면 onSubmit 동작 안함*/}
      <button type="submit" onClick={handleClick}>
        {!isEdit ? 'Edit Profile' : 'Save'}
      </button>
      <p><i>{`Hello, ${name.firstName} ${name.lastName}!`}</i></p>
    </form>
  );
}

~~~
### 챌린지 3 of 3: CSS class를 추가하고 제거하기 
~~~
~~~

---
# 2. State 구조 선택하기
~~~
2.1 연관된 state 그룹화하기. 두 개 이상의 state 변수를 항상 동시에 업데이트한다면, 단일 2.2.2 state 변수로 병합하는 것을 고려
2.3 State의 모순 피하기.
2.4 불필요한 state 피하기. 렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면, 컴포넌트의 state에 해당 정보를 넣지 않아야 합니다.
2.5 State의 중복 피하기.
2.6 깊게 중첩된 state 피하기. 
~~~

## 2.1 연관된 state 그룹화하기.
~~~js
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// 항상 x, y가 한타이밍에 변경된다면
const [position, setPosition] = useState({ x: 0, y: 0 });

// 리액트 18에서는 배치가 업데이트되면서 
// 따로써도 문제없을 정도의 수준인지 
~~~

## 2.2 불필요한 state 피하기 
~~~js
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  // sending과, sent가 같은 기능을 함
  // status 로 관리
   const [status, setStatus] = useState('typing');
~~~

~~~js
 const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

   const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;
~~~

## 2.3 + Props를 state에 미러링하지 마세요. 
~~~js
// State는 첫 번째 렌더링 중에만 초기화되서 props가 변경되도 state가 변경되지 않음
// 미러링이라고하는데 첫번째에만 변경됨을 인지하고 사용해야함
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
~~~

## 2.4 State의 중복 피하기.
~~~js
// 같은 값을 사용한다는건, 값이 하나라도 변한다면 매번 고쳐줘야함.
const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

// 키값으로 관리
    const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);
~~~

## 2.5 깊게 중첩된 state 피하기 
## 2.5.1 메모리 사용량 개선하기 
~~~
메모리 사용량을 개선하기 위해서는 삭제된 항목(그리고 그들의 자식들!)을 “테이블” 객체에서 제거
주로 immer 라이브러리 사용
~~~

### 챌린지 1 of 4: 업데이트되지 않는 컴포넌트 수정하기 
~~~js
import { useState } from 'react';

export default function Clock(props) {
  return (
    <h1 style={{ color: props.color }}>
      {props.time}
    </h1>
  );
}

~~~

### 챌린지 2 of 4: 깨진 포장 목록 수정하기 
~~~js
//불필요한 state 피하기 
  const total = items.length;
  const packed = items
    .filter(item => item.packed)
    .length;

~~~

### 챌린지 3 of 4: 선택 사라짐 수정하기 

~~~
(State의 중복 피하기.)
highlightedId로 변경
~~~

### 챌린지 4 of 4: 다중 선택 구현 

~~~js
// prevCount 최신값 보장
setCount(prevCount => prevCount+1);

// count+1 count 가 디팬던시에 저장해야만 함
// 그냥 썼을 때 크게 문제가되진않음
setCount(count+1);
~~~

~~~
리덕스 툴킷같은걸 사용하면 immer가 자동으로딤
//
인덱스로 만들면 순서여서
배열 업데이트가 안될 수있다.
~~~