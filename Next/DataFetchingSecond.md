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

---

## 2.4 Examples
### 2.4.1 Forms
### 2.4.2 Passing additional arguments
### 2.4.3 Nested form elements
### 2.4.4 Programmatic form submission
### 2.4.5 Server-side form validation
### 2.4.6 Pending states

### 2.4.7 Optimistic updates

### 2.4.8 Event handlers

### 2.4.9 useEffect

### 2.4.10 Error Handling

### 2.4.11 Revalidating data

### 2.4.12 Redirecting

### 2.4.13 Cookies

---

## 2.5 Security
### 2.5.1 Authentication and authorization

### 2.5.2 Closures and encryption

### 2.5.3 Overwriting encryption keys (advanced)

### 2.5.4 Allowed origins (advanced)

---

## 2.6 Additional resources
## 2.7 Next Steps

# 3. Incremental Static Regeneration(ISR)

## 3.1 On this page

---

## 3.2 Reference
### 3.2.1 Route segment config
### 3.2.2 Functions

---

## 3.3 Examples
### 3.3.1 Time-based revalidation
### 3.3.2 On-demand revalidation with revalidatePath
### 3.3.3 On-demand revalidation with revalidateTag
### 3.3.4 Handling uncaught exceptions
### 3.3.5 Customizing the cache location

---

## 3.4 Troubleshooting
### 3.4.1 Debugging cached data in local development
### 3.4.2 Verifying correct production behavior

---

## 3.5 Caveats

---

## 3.6 Version history
