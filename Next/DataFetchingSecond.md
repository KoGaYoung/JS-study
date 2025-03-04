# 2. Server Actions and Mutations

## 2.1 On this page
ServerAction은 
- 서버에서 실행되는 비동기함수
- 서버 & 클라이언트 컴포넌트에서 호출 가능
- 주로 폼 제출과 데이터변경(mutation)에 사용
(Mutation 은 데이터 변경하는 작업을 말함, CRUD 같은 것들)

---

## 2.2 Convention
- 1 Server Action을 정의할 때는 ```use server```를 사용
- 2 ```use server```를 함수 맨 위에 붙이면 그 함수가 서버에서 실행됨
- 3 파일 맨 위에 ```use server```를 붙이면 그 파일에 있는 모든 함수가 Server Action이 됨


### 2.2.1 Server Components

### 예제 2: 특정 함수만 서버에서 실행 (=2 ```use server```를 함수 맨 위에 붙이면 그 함수가 서버에서 실행됨)
```tsx
// Page 컴포넌트 자체는 클라이언트와 서버 모두에서 실행될 수 있음
export default function Page() {
  // create() 함수 안에서 "use server"를 사용 ->  이 함수만 서버에서 실행됨
  async function create() {
    "use server";
    // 데이터 변경 로직 (예: DB에 저장)
  }

  return "...";
}
```

### 2.2.2 Client Components
✅ 클라이언트 컴포넌트(Client Component)에서 서버 액션을 사용하려면?   
✅ 서버 액션을 별도의 파일로 분리하고, 그 파일에서 ```use server```를 선언하면 됨    
✅ 그런 다음 클라이언트 컴포넌트에서 import해서 사용하면 됨    

예제: Client Component에서 Server Action 사용하기    

1️⃣ 서버 액션을 정의하는 파일 (`app/actions.ts`)
"use server"가 맨 위에 있으므로    
-> 이 파일에 있는 모든 함수(create)는 자동으로 서버에서 실행됨   

```tsx
"use server"; // 파일 맨 위에 선언

export async function create() {
  // 서버에서 실행되는 로직 (예: DB에 데이터 추가)
}
```
2️⃣ 클라이언트 컴포넌트에서 서버 액션 사용 (app/button.tsx)
"use client"를 선언했으므로 이 컴포넌트는 클라이언트에서 실행됨   
create() 함수는 서버에서 실행되지만, Button 컴포넌트는 클라이언트에서 실행됨   
버튼을 클릭하면 서버 액션이 서버에서 비동기로 실행됨   
```
"use client"; // 클라이언트 컴포넌트 선언

import { create } from "./actions"; // 서버 액션 import

export function Button() {
  return <button onClick={() => create()}>Create</button>;
}
```

### 2.2.3 Passing actions as props
✅ 서버 액션을 클라이언트 컴포넌트에 props로 전달할 수 있음    
✅ props 이름이 ```action``` 또는 ```Action```으로 끝나면 Next.js에서 서버 액션으로 인식   
✅ TypeScript 플러그인은 이를 허용하지만, 실제로 실행될 때 올바른 함수인지 확인함   

### 예제: 서버 액션을 클라이언트 컴포넌트에 전달하기  
1️⃣ **서버 액션 정의 (`app/actions.ts`)**

```tsx
`useServer`;

export async function updateItem(formData: FormData) {
  // 서버에서 실행되는 로직 (예: DB 업데이트)
}
```
2️⃣ 클라이언트 컴포넌트에서 서버 액션 받기 (app/client-component.tsx)
updateItemAction을 props로 받아서 <form action={updateItemAction}>에 전달   
폼이 제출되면 updateItemAction이 실행되는데, 이건 서버 액션이므로 서버에서 실행됨
```tsx
`useClient`;

export default function ClientComponent({
  updateItemAction,
}: {
  updateItemAction: (formData: FormData) => void;
}) {
  return <form action={updateItemAction}>{/* ... */}</form>;
}
```
3️⃣ 서버 액션을 클라이언트 컴포넌트에 전달 (app/page.tsx)
updateItem(서버 액션)을 updateItemAction props로 ClientComponent에 전달   
클라이언트 컴포넌트에서는 이를 form action으로 사용   

```tsx
import ClientComponent from "./client-component";
import { updateItem } from "./actions";

export default function Page() {
  return <ClientComponent updateItemAction={updateItem} />;
}
```
---

## 2.3 Behavior
Next.js의 Server Actions에 대한 동작 방식과 특징

- 폼(form)을 통한 호출:
  - <form> 요소의 action 속성을 사용해 서버 액션을 호출할 수 있습니다.

- 진보된 지원:
  - JavaScript가 로드되지 않았거나 비활성화된 경우에도 폼이 정상적으로 제출되어 서버 액션이 실행됩니다.

- 클라이언트 컴포넌트와의 차이:
  - 클라이언트 컴포넌트에서는 JavaScript가 로드되지 않았을 때도 제출이 대기되며, 클라이언트가 준비되면 우선적으로 처리됩니다. 또한, 클라이언트 컴포넌트에서 폼 제출 시 페이지 새로고침 없이 처리됩니다.
  - => 좀더쉽게: 클라 컴포넌트 JS로드되기 전에도 브라우저만 준비되면 바로 처리함. =기능도 사용성도좋음

- 호출 방법 다양성:
  - 서버 액션은 <form>뿐만 아니라 버튼, 이벤트 핸들러, useEffect, 타사 라이브러리 등 여러 방법으로 호출할 수 있습니다.

- Next.js의 캐싱 및 재검증과 통합:
  - 서버 액션을 호출하면 Next.js는 업데이트된 UI와 새로운 데이터를 한 번의 서버 통신으로 받아올 수 있습니다.

- POST 방식만 허용:
  - 서버 액션은 내부적으로 POST 메서드를 사용하며, 오직 POST 방식으로만 호출할 수 있습니다.

- 직렬화 가능한 인자와 반환값:
  - 서버 액션의 인자와 반환값은 React에서 직렬화할 수 있는 값이어야 합니다.

- 함수로서의 재사용성:
  - 서버 액션은 일반 함수이므로 애플리케이션 내 다른 곳에서도 재사용할 수 있습니다.

- 상속받는 설정:
  - 서버 액션은 사용되는 페이지나 레이아웃의 런타임 설정과 라우트 세그먼트 구성(예: maxDuration 등)을 그대로 상속받습니다.


---

## 2.4 Examples
### 2.4.1 Forms
React는 기본 HTML <form> 요소를 확장하여, action 속성을 사용하면 서버 액션을 바로 호출할 수 있게 합니다.   
폼이 제출되면, 해당 액션 함수는 자동으로 폼의 데이터를 담은 FormData 객체를 받습니다.   
따라서, 별도로 React의 useState를 사용해 각 필드의 값을 관리할 필요 없이, FormData의 기본 메서드로 데이터를 쉽게 가져올 수 있습니다.

```tsx
export default function Page() {
  async function createInvoice(formData: FormData) {
    'use server'
 
    const rawFormData = {
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    }
 
    // mutate data
    // revalidate cache
  }
 
  return <form action={createInvoice}>...</form>
}
```
### 2.4.2 Passing additional arguments
 서버 액션에 추가 인자를 전달하는 두 가지 방법 중 bind 메소드를 사용하는 방법

 bind 메소드 사용:   
클라이언트 컴포넌트에서 updateUser 함수를 호출할 때,    
userId라는 인자를 미리 전달하기 위해 updateUser.bind(null, userId)를 사용합니다.    

이렇게 하면, 폼이 제출될 때 자동으로 첫 번째 인자로 userId가 전달되고,     
두 번째 인자로는 폼 데이터를 담은 FormData 객체가 전달됩니다.

```tsx
'use client'
 
import { updateUser } from './actions'
 
export function UserProfile({ userId }: { userId: string }) {
  const updateUserWithId = updateUser.bind(null, userId)
 
  return (
    <form action={updateUserWithId}>
      <input type="text" name="name" />
      <button type="submit">Update User Name</button>
    </form>
  )
}
```
대안 방법:    
또 다른 방법은 <input type="hidden" ...> 요소를 이용하여 폼 안에 숨겨진 필드로 인자를 전달하는 것입니다.    
다만 이 방법은 HTML에 인자가 그대로 노출되므로 보안에 주의해야 합니다.

### 2.4.3 Nested form elements
폼 내부에 있는 중첩 요소들(예: <button>, <input type="submit">, <input type="image">)에서도 서버 액션을 호출할 수 있습니다.    
이 요소들은 별도의 formAction 속성이나 이벤트 핸들러를 사용할 수 있어, 하나의 폼 안에서 여러 서버 액션을 각각 다르게 처리할 수 있습니다.   

예를 들어, 게시글을 작성하는 폼에서 "게시" 버튼과 "임시 저장" 버튼을 따로 만들고 싶다면,    
각 버튼에 서로 다른 서버 액션을 지정할 수 있습니다.  

### 2.4.4 Programmatic form submission

사용자가 특정 키 조합(⌘ + Enter 혹은 Ctrl + Enter)을 누르면, 자바스크립트 코드로 폼 제출을 자동으로 실행하는 방법
```tsx
'use client'
 
export function Entry() {
  const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    if (
      (e.ctrlKey || e.metaKey) &&
      (e.key === 'Enter' || e.key === 'NumpadEnter')
    ) {
      e.preventDefault()
      e.currentTarget.form?.requestSubmit()
    }
  }
 
  return (
    <div>
      <textarea name="entry" rows={20} required onKeyDown={handleKeyDown} />
    </div>
  )
}
```
### 2.4.5 Server-side form validation
서버 측에서 폼 검증을 하는 방법

- 기본 클라이언트 검증:
  - HTML의 required나 type="email"과 같은 속성들을 사용하면 브라우저에서 기본적인 입력값 검증을 할 수 있습니다.
- 서버 측 검증:
  - 더 복잡한 검증이 필요할 때는, 예를 들어 zod 같은 라이브러리를 사용하여 폼 데이터를 검증할 수 있습니다.
    - 서버 액션 함수(createUser)에서 폼 데이터(FormData)를 받아서, zod 스키마로 해당 데이터의 형식이나 값을 검증합니다.
    - 만약 데이터가 유효하지 않으면, 오류 메시지를 포함한 객체를 바로 반환하여 더 이상 진행하지 않습니다.
    - 데이터가 올바르면, 실제 데이터 변경(데이터베이스 업데이트 등)을 수행합니다.
```ts
// app/actions.ts
'use server'
 
import { z } from 'zod'
 
const schema = z.object({
  email: z.string({
    invalid_type_error: 'Invalid Email',
  }),
})
 
export default async function createUser(formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
  })
 
  // Return early if the form data is invalid
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }
 
  // Mutate data
}
```
- 결과 처리와 사용자 피드백:
  - 서버 측 검증 후, 검증 결과(예: 오류 메시지)를 직렬화 가능한 객체 형태로 반환할 수 있습니다.
    - 클라이언트 컴포넌트에서는 React의 useActionState 훅을 사용하여 서버 액션의 상태(예: 검증 오류 메시지, 진행 중 여부 등)를 받아와 사용자에게 보여줍니다.
    - 이때 서버 액션 함수의 첫 번째 인자는 이전 상태(prevState) 또는 초기 상태(initialState)로 전달되어, 결과에 따라 사용자 인터페이스(UI)를 업데이트 할 수 있습니다.
    - 예를 들어, 이메일 형식이 올바르지 않은 경우 오류 메시지를 표시하거나, 모든 검증이 통과하면 대시보드 페이지로 리디렉션할 수 있습니다.

```ts
// app/actions.ts
'use server'
 
import { redirect } from 'next/navigation'
 
export async function createUser(prevState: any, formData: FormData) {
  const res = await fetch('https://...')
  const json = await res.json()
 
  if (!res.ok) {
    return { message: 'Please enter a valid email' }
  }
 
  redirect('/dashboard')
}
```
```tsx
// app/ui/signup.tsx
'use client'
 
import { useActionState } from 'react'
import { createUser } from '@/app/actions'
 
const initialState = {
  message: '',
}
 
export function Signup() {
  const [state, formAction, pending] = useActionState(createUser, initialState)
 
  return (
    <form action={formAction}>
      <label htmlFor="email">Email</label>
      <input type="text" id="email" name="email" required />
      {/* ... */}
      <p aria-live="polite">{state?.message}</p>
      <button disabled={pending}>Sign up</button>
    </form>
  )
}
```

### 2.4.6 Pending states
서버 액션이 실행되는 동안 로딩 상태를 표시할 수 있는 방법을 설명

```useActionState``` 훅:    
```useActionState``` 훅은 ```pending```이라는 불리언 값을 제공합니다.   
이 ```pending``` 값은 서버 액션이 실행 중일 때 ```true```가 되어, 로딩 인디케이터나 버튼 비활성화 등 사용자에게 진행 중임을 알릴 때 사용할 수 있습니다.   

대안: ```useFormStatus``` 훅:   
```useFormStatus``` 훅을 사용하면 액션이 실행되는 동안 로딩 상태를 표시할 수 있습니다.   
이 훅을 사용할 경우 별도의 컴포넌트를 만들어 로딩 인디케이터를 렌더링해야 합니다.

예제 1 (app/ui/button.tsx):   
```useFormStatus``` 훅을 사용하여 ```pending``` 값을 받아옵니다.
```pending```이 ```true```일 때 버튼을 비활성화(disabled) 시킵니다.
이렇게 작성하면, 폼 제출 도중 버튼이 비활성화되어 중복 제출을 방지할 수 있습니다.
```tsx
'use client'
 
import { useFormStatus } from 'react-dom'
 
export function SubmitButton() {
  const { pending } = useFormStatus()
 
  return (
    <button disabled={pending} type="submit">
      Sign Up
    </button>
  )
}
```
SubmitButton 컴포넌트를 폼 안에 넣어 사용합니다.   
이렇게 하면, 폼이 제출되면서 서버 액션이 진행 중일 때 자동으로 버튼이 비활성화됩니다.
```tsx
import { SubmitButton } from './button'
import { createUser } from '@/app/actions'
 
export function Signup() {
  return (
    <form action={createUser}>
      {/* Other form elements */}
      <SubmitButton />
    </form>
  )
}
```

### 2.4.7 Optimistic updates
서버 액션이 완료되기 전에 미리 UI를 업데이트하는 "낙관적 업데이트(optimistic update)" 방식
```tsx
'use client'

// React의 useOptimistic 훅과 서버 액션 send 함수를 임포트합니다.
import { useOptimistic } from 'react'
import { send } from './actions'
 
type Message = {
  message: string
}
 
export function Thread({ messages }: { messages: Message[] }) {
  // useOptimistic 훅을 사용하여 낙관적 업데이트(optimistic update)를 구현합니다.
  // 첫 번째 인자로 초기 메시지 배열(messages)를 전달하고,
  // 두 번째 인자로는 새로운 메시지를 기존 상태에 추가하는 함수를 제공합니다.
  const [optimisticMessages, addOptimisticMessage] = useOptimistic<
    Message[],
    string
  >(messages, (state, newMessage) => [...state, { message: newMessage }])

  // 폼이 제출될 때 호출될 함수 (비동기 함수)를 정의합니다.
  const formAction = async (formData: FormData) => {
    // 폼 데이터에서 'message' 값을 추출하여 문자열로 변환합니다.
    const message = formData.get('message') as string
    // 낙관적 업데이트: 서버 응답을 기다리지 않고 바로 UI에 새 메시지를 추가합니다.
    addOptimisticMessage(message)
    // 이후 서버 액션 send를 호출하여 실제로 메시지를 서버에 전송합니다.
    await send(message)
  }
 
  return (
    <div>
      {optimisticMessages.map((m, i) => (
        <div key={i}>{m.message}</div>
      ))}
  {/* 폼 제출 시 formAction 함수가 실행되어 메시지 전송 및 UI 업데이트를 처리합니다. */}
      <form action={formAction}>
        <input type="text" name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  )
}
```

### 2.4.8 Event handlers
서버 액션을 반드시 <form> 요소 내에서만 사용할 필요 없이, onClick과 같은 이벤트 핸들러를 통해서도 호출할 수 있다는 것을 설명

```Tsx
// app/view-count.tsx

'use client'
 
import { incrementViews } from './actions'
import { useState, useEffect } from 'react'
 
export default function ViewCount({ initialViews }: { initialViews: number }) {
  const [views, setViews] = useState(initialViews)
 
  useEffect(() => {
    const updateViews = async () => {
      const updatedViews = await incrementViews()
      setViews(updatedViews)
    }
 
    updateViews()
  }, [])
 
  return <p>Total Views: {views}</p>
}
```
### 2.4.9 useEffect
useEffect 훅을 사용하면 컴포넌트가 마운트될 때 또는 특정 의존성 값이 변경될 때 서버 액션을 호출할 수 있습니다.
- 자동 트리거:
  - 컴포넌트가 렌더링되면서 특정 작업(예: 조회수 업데이트)을 자동으로 실행할 수 있습니다.
- 글로벌 이벤트에 따른 호출:
  - 예를 들어, 앱 단축키의 onKeyDown 이벤트나 무한 스크롤 구현을 위한 Intersection Observer와 같은 글로벌 이벤트에 반응해 서버 액션을 실행할 수 있습니다.
- 의존성 기반 업데이트:
  - 특정 데이터가 변경될 때마다 자동으로 서버 액션을 호출하여 최신 상태를 유지할 수 있습니다.

```tsx
// app/view-count.tsx
'use client'
 
import { incrementViews } from './actions'
import { useState, useEffect } from 'react'
 
export default function ViewCount({ initialViews }: { initialViews: number }) {
  const [views, setViews] = useState(initialViews)
 
  useEffect(() => {
    const updateViews = async () => {
      const updatedViews = await incrementViews()
      setViews(updatedViews)
    }
 
    updateViews()
  }, [])
 
  return <p>Total Views: {views}</p>
}
```
### 2.4.10 Error Handling

### 2.4.11 Revalidating data

### 2.4.12 Redirecting

### 2.4.13 Cookies

---

## 2.5 Security
Server Action이 기본적으로 공개된 HTTP 엔드포인트로 동작하기 때문에 보안에 유의해야 함   
Next.js가 이를 보완하기 위해 제공하는 몇 가지 보안 기능에 대해 설명

1.기본 보안 전제:   
Server Action을 만들고 export하면, 별도로 다른 코드에서 사용하지 않더라도 공개된 HTTP 엔드포인트처럼 동작합니다.   
따라서, 인증이나 권한 체크를 제대로 하지 않으면 누구나 호출할 수 있는 위험이 있습니다.   

2.보안 기능:   
Next.js는 Server Action을 보다 안전하게 만들기 위해 아래와 같은 기능들을 제공합니다.   

2.1보안 액션 ID (Secure action IDs):   
Next.js는 클라이언트가 Server Action을 호출할 때 사용할 수 있도록 암호화되고 예측할 수 없는 ID를 생성합니다.   
이 ID는 빌드 사이에 주기적으로 재생성되어 보안성이 향상됩니다.   

2.2미사용 코드 제거 (Dead code elimination):   
사용되지 않는 Server Action들은 클라이언트 번들에서 제거되어, 제3자가 접근할 수 없게 됩니다.   
추가 참고 사항:   

3.보안 ID는 빌드 시 생성되며 최대 14일 동안 캐시됩니다.   
새로운 빌드가 시작되거나 빌드 캐시가 무효화될 때 ID가 재생성됩니다.   
이로 인해 인증 레이어가 없는 경우의 위험을 줄일 수 있지만, 여전히 Server Action은 공개 HTTP 엔드포인트처럼 취급해야 합니다.   

### 2.5.1 Authentication and authorization
사용자에게 작업을 수행할 권한이 있는지 확인
```ts
'use server'
 
import { auth } from './lib'
 
export function addItem() {
  const { user } = auth()
  if (!user) {
    throw new Error('You must be signed in to perform this action')
  }
 
  // ...
}
```
### 2.5.2 Closures and encryption
컴포넌트 내부에서 Server Action을 정의하면, 그 액션이 컴포넌트의 외부 변수에도 접근할 수 있는 "클로저(closure)"가 형성되는것을 말함

예를 들어, 컴포넌트 내부에 publish 액션을 정의할 때,    
이 액션은 컴포넌트 밖에서 정의된 publishVersion 변수에 접근할 수 있습니다.    
즉, 액션 내부에서 해당 변수 값을 읽거나 사용할 수 있다는 뜻입니다.   

또한, Next.js에서는 보안을 위해 이러한 Server Action을 호출할 때 암호화된 ID를 사용하여,    
외부에서 직접 액션을 추적하거나 조작하기 어렵도록 합니다.

요약하면:

클로저: 컴포넌트 내에 정의된 함수(서버 액션)가 외부 변수에 접근할 수 있는 기능.
암호화: Next.js가 서버 액션을 안전하게 호출할 수 있도록 암호화된 ID를 생성하여 사용함.

### 2.5.3 Overwriting encryption keys (advanced)

### 2.5.4 Allowed origins (advanced)

---

## 2.6 Additional resources
## 2.7 Next Steps

# 3. Incremental Static Regeneration(ISR)

## 3.1 On this page
Incremental Static Regeneration (ISR)은 정적 페이지를 미리 생성해두면서도,    
페이지 내용이 변경되었을 때 전체 사이트를 다시 빌드하지 않고도 필요한 페이지만 갱신할 수 있는 기능입니다. 

정적 페이지 업데이트   
서버 부하 감소   
자동 캐시 관리   
빠른 빌드   

(ISR)을 사용하여 블로그 게시글 페이지를 동적으로 갱신하는 방법을 보여줌

첫 렌더링은 빌드 시 생성됨, 60초마다 서버에서 서버사이드 렌더링, 60초 이내 요청은 모두 같은페이지로 보여짐

```tsx
// 블로그 게시글 데이터를 위한 타입 정의
interface Post {
  id: string
  title: string
  content: string
}

// revalidate 상수: 요청이 들어올 때마다 최대 60초 간격으로 캐시를 무효화합니다.
// 즉, 페이지가 60초마다 백그라운드에서 업데이트될 수 있습니다.
export const revalidate = 60

// dynamicParams: 빌드 시 미리 생성하지 않은 경로가 요청되면,
// 서버에서 즉시 페이지를 생성할지 여부를 결정합니다.
// true이면 요청 시 서버 사이드 렌더링으로 페이지를 동적으로 생성합니다.
// false이면, 생성되지 않은 경로는 404 에러를 반환합니다.
export const dynamicParams = true

// generateStaticParams 함수는 빌드 시 실행되어,
// 미리 생성할 경로(여기서는 게시글 id 목록)를 반환합니다.
export async function generateStaticParams() {
  // API를 통해 모든 블로그 게시글 데이터를 가져옵니다.
  const posts: Post[] = await fetch('https://api.vercel.app/blog').then((res) =>
    res.json()
  )
  // 각 게시글의 id를 문자열로 변환하여 경로 파라미터 배열로 반환합니다.
  return posts.map((post) => ({
    id: String(post.id),
  }))
}

// 페이지 컴포넌트: URL의 파라미터로 게시글 id를 전달받아 해당 게시글을 렌더링합니다.
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  // 비동기로 전달받은 params에서 id 값을 추출합니다.
  const { id } = await params
  // 게시글 id를 이용해 API에서 해당 게시글 데이터를 가져옵니다.
  const post: Post = await fetch(`https://api.vercel.app/blog/${id}`).then(
    (res) => res.json()
  )
  // 가져온 게시글 데이터를 이용해 제목과 내용을 화면에 렌더링합니다.
  return (
    <main>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </main>
  )
}

```
---

## 3.2 Reference
### 3.2.1 Route segment config
- revalidate
- dynamicParams

### 3.2.2 Functions
- revalidatePath : 특정 경로의 페이지를 수동으로 갱신(재검증)할 때 사용하는 함수입니다.
- revalidateTag : 태그와 연관된 페이지들을 한 번에 갱신할 때 사용하는 함수입니다.
---

## 3.3 Examples
### 3.3.1 시간 기반 재검증 (Time-based revalidation)
이것은 /blog에서 블로그 게시글 목록을 가져와서 표시합니다.   
한 시간이 지나면, 페이지를 다음 방문할 때 해당 페이지의 캐시가 무효화됩니다.   
그 후, 백그라운드에서 최신 블로그 게시글을 포함한 새로운 버전의 페이지가 생성됩니다.  


```tsx
// revalidate: 캐시를 무효화하는 시간을 초 단위로 지정함.
// 3600초 = 1시간마다 캐시를 무효화합니다.
// 한 시간이 지나면, 페이지를 다음 방문 시 캐시가 무효화됩니다.
export const revalidate = 3600 // 매 시간마다 캐시 무효화
```

가능하면 높은 재검증 시간을 설정하는 것을 권장합니다. 예를 들어, 1초 대신 1시간을 사용하세요.   
만약 더 정밀한 제어가 필요하다면, on-demand revalidation(요청 기반 재검증, 2가지)을 고려해 보세요.   
실시간 데이터가 필요하다면, 동적 렌더링(dynamic rendering)으로 전환하는 것을 고려해 보세요.  

### 3.3.2 On-demand revalidation with revalidatePath
revalidatePath를 사용한 요청 기반 재검증 (On-demand revalidation with revalidatePath)
더 정확한 재검증 방법을 위해, revalidatePath 함수를 사용하여 페이지를 필요에 따라 무효화합니다.    
예를 들어, 새 게시글을 추가한 후에 createPost 서버 액션이 revalidatePath 를 호출함으로써 해당경로 캐시삭제
```tsx
'use server'

// next/cache 모듈에서 revalidatePath 함수를 임포트합니다.
import { revalidatePath } from 'next/cache'

// createPost 함수: 새로운 게시글을 추가한 후 호출되는 서버 액션
export async function createPost() {
  // /posts 경로의 캐시를 무효화합니다.
  // 이것은 해당 경로의 캐시를 삭제하여,
  // 서버 컴포넌트가 최신 데이터를 다시 가져올 수 있도록 합니다.
  revalidatePath('/posts')
}
```

### 3.3.3 On-demand revalidation with revalidateTag
revalidateTag를 사용한 요청 기반 재검증 (On-demand revalidation with revalidateTag)

대부분의 사용 사례에서는 전체 경로를 재검증(revalidatePath)하는 것을 선호합니다.   
만약 더 세밀한 제어가 필요하다면, revalidateTag 함수를 사용할 수 있습니다.   
예를 들어, 개별 fetch 호출에 태그를 지정할 수 있습니다.   

만약 ORM을 사용하거나 데이터베이스에 연결하는 경우, unstable_cache를 사용할 수 있습니다.

그 후, Server Actions 또는 Route Handler에서 revalidateTag를 사용할 수 있습니다.

개별 fetch 호출에 태그 지정 예제 (app/blog/page.tsx):
```tsx
// 기본 페이지 컴포넌트를 비동기로 정의합니다.
export default async function Page() {
  // fetch 호출에 next 옵션을 사용하여 'posts' 태그를 지정합니다.
  const data = await fetch('https://api.vercel.app/blog', {
    next: { tags: ['posts'] }, // 'posts' 태그 지정
  })
  // 데이터를 JSON으로 파싱합니다.
  const posts = await data.json()
  // 추가 로직이 있을 수 있습니다.
  // ...
}
```
Server Actions 또는 Route Handler에서 revalidateTag 사용 예제 (app/actions.ts):
```tsx
'use server'

// next/cache 모듈에서 revalidateTag 함수를 임포트합니다.
import { revalidateTag } from 'next/cache'

// createPost 함수: 새로운 게시글을 생성한 후 호출되는 서버 액션
export async function createPost() {
  // 캐시에서 'posts' 태그가 지정된 모든 데이터를 무효화합니다.
  revalidateTag('posts') // 'posts' 태그로 지정된 모든 데이터를 무효화
}
```

### 3.3.4 Handling uncaught exceptions
처리되지 않은 예외 처리
- 데이터 재검증 중 오류가 발생하면, 마지막에 성공적으로 생성된 데이터가 캐시에서 계속 제공됩니다.
- 다음 요청 시 Next.js가 다시 데이터 재검증을 시도합니다.

### 3.3.5 Customizing the cache location
캐시 위치 사용자 지정
- 페이지 캐싱 및 재검증 (Incremental Static Regeneration 사용)은 동일한 공유 캐시를 사용합니다.
- Vercel에 배포할 경우, ISR 캐시는 자동으로 내구성 스토리지에 영구 저장됩니다.
- 자체 호스팅 시, Next.js 서버의 파일 시스템(디스크)에 ISR 캐시가 저장됩니다.
  - 이 기능은 Pages와 App Router를 사용하는 자체 호스팅 환경에서 자동으로 작동합니다.
- 만약 캐시된 페이지와 데이터를 내구성 스토리지에 영구 저장하거나, 여러 컨테이너나 Next.js 애플리케이션 인스턴스 간에 캐시를 공유하고 싶다면, Next.js 캐시 위치를 구성할 수 있습니다.


---

## 3.4 Troubleshooting
### 3.4.1 Debugging cached data in local development
로컬 개발 환경에서 캐시된 데이터를 디버깅하기

만약 fetch API를 사용 중이라면, 어떤 요청이 캐시되고 있는지 혹은 캐시되지 않은 요청인지 파악하기 위해 추가 로깅을 활성화할 수 있습니다.

```
// next.config.js 파일
module.exports = {
  logging: {
    fetches: {
      fullUrl: true, // 요청의 전체 URL을 로그에 기록합니다. 이를 통해 어떤 요청이 캐시되었는지 확인할 수 있습니다.
    },
  },
}
```

### 3.4.2 Verifying correct production behavior
프로덕션에서 올바르게 동작하는지 검증하기
- 프로덕션 환경에서 페이지가 제대로 캐시되고 재검증되는지 확인하려면,
  로컬에서 next build를 실행한 후 next start를 실행하여 프로덕션 Next.js 서버를 구동해보세요.
- 이렇게 하면 ISR(Incremental Static Regeneration)의 동작을 실제 프로덕션 환경과 같이 테스트할 수 있습니다.
- 추가 디버깅을 위해 .env 파일에 다음 환경 변수를 추가하세요:

```tsx
// env
# NEXT_PRIVATE_DEBUG_CACHE=1
# 이 환경 변수를 설정하면 Next.js 서버 콘솔에 ISR 캐시 히트와 미스가 기록됩니다.
# 이를 통해 어떤 페이지가 next build 시 생성되었는지, 요청에 따라 어떤 페이지가 업데이트되는지 확인할 수 있습니다.
NEXT_PRIVATE_DEBUG_CACHE=1
```
---

## 3.5 Caveats
주의사항
- ISR은 Node.js 런타임(기본값)을 사용할 때만 지원됩니다.
- 정적 내보내기(Static Export)를 생성할 때는 ISR을 지원하지 않습니다.
- 정적으로 렌더링된 라우트에 여러 개의 fetch 요청이 있고, 각 요청이 서로 다른 재검증 주기를 가지고 있다면, ISR에는 가장 짧은 주기가 적용됩니다. 하지만, 데이터 캐시에서는 각각의 재검증 주기가 그대로 반영됩니다.
- 라우트에 사용된 fetch 요청 중 하나라도 재검증 시간이 0이거나 명시적으로 no-store인 경우, 해당 라우트는 동적으로 렌더링됩니다.
- 요청 기반 ISR에서는 미들웨어가 실행되지 않습니다. 즉, 미들웨어에 설정된 경로 재작성이나 로직이 적용되지 않으므로, 반드시 정확한 경로를 재검증하도록 하세요. 예를 들어, 재작성된 /post-1이 아닌, /post/1을 사용해야 합니다.


