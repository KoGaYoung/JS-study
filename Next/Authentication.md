1. Authentication (인증):
사용자가 자신이 누구라고 주장하는 사람이 맞는지를 확인하는 과정   
이건 보통 사용자 이름과 비밀번호 같은 걸로 본인을 증명해야함   
2. Session Management (세션 관리):   
사용자가 로그인한 상태를 여러 요청(request) 사이에서 추적하는 걸 말함
3. Authorization (권한 부여):   
사용자가 어떤 페이지(route)나 데이터를 접근할 수 있는지를 결정   

인증라이브러리 사용 추천   
다음과 같은 기능을 기본으로 제공:     
- 인증(Authentication)   
- 세션 관리(Session Management)    
- 권한 부여(Authorization)   

추가 기능:
- 소셜 로그인 (예: 구글, 깃허브 로그인 등)
- 다중 인증 (MFA, 2단계 인증)
- 역할 기반 접근 제어 (RBAC)

 ---

 인증 : 회원가입 및 로그인 기능

 (Server Actions는 항상 서버에서 실행되기 때문에,
인증 로직을 처리하는 데 보안적으로 안전한 환경을 제공)

 1. 사용자 인증 정보 받기 (Capture user credentials)

```tsx
// app/ui/signup-form.tsx
import { signup } from '@/app/actions/auth'
 
export function SignupForm() {
  return (
    <form action={signup}>
      <div>
        <label htmlFor="name">Name</label>
        <input id="name" name="name" placeholder="Name" />
      </div>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" type="email" placeholder="Email" />
      </div>
      <div>
        <label htmlFor="password">Password</label>
        <input id="password" name="password" type="password" />
      </div>
      <button type="submit">Sign Up</button>
    </form>
  )
}
```

```tsx
// app/actions/auth.ts
export async function signup(formData: FormData) {}
```

2. 폼 필드를 서버에서 검증하기(Validate form fields on the server)
서버액션을 사용해서 서버 측에서 폼 입력값 검증 가능
인증 제공자(아마 서버에서 에러메시지를 적절하게 주지않는다면?) ZOD나 Yup 같은 스키마 검증 라이브러리 사용

```ts
//app/lib/definitions.ts
import { z } from 'zod'
 
export const SignupFormSchema = z.object({
  name: z
    .string()
    .min(2, { message: 'Name must be at least 2 characters long.' })
    .trim(),
  email: z.string().email({ message: 'Please enter a valid email.' }).trim(),
  password: z
    .string()
    .min(8, { message: 'Be at least 8 characters long' })
    .regex(/[a-zA-Z]/, { message: 'Contain at least one letter.' })
    .regex(/[0-9]/, { message: 'Contain at least one number.' })
    .regex(/[^a-zA-Z0-9]/, {
      message: 'Contain at least one special character.',
    })
    .trim(),
})
 
export type FormState =
  | {
      errors?: {
        name?: string[]
        email?: string[]
        password?: string[]
      }
      message?: string
    }
  | undefined
```
스키마에 맞지 않으면 서버요청으로 보내버리기 전에 중단
```ts
//app/actions/auth.ts
import { SignupFormSchema, FormState } from '@/app/lib/definitions'
 
export async function signup(state: FormState, formData: FormData) {
  // Validate form fields
  const validatedFields = SignupFormSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
    password: formData.get('password'),
  })
 
  // If any form fields are invalid, return early
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }
 
  // Call the provider or db to create a user...
}
```

다시 SignUp 컴포넌트로 돌아와서 useActionState 훅을 사용하면 폼을 제출하는 동안 검증 메시지를 사용자에게 노출
```tsx
// app/ui/signup-form.tsx
'use client'
 
import { signup } from '@/app/actions/auth'
import { useActionState } from 'react'
 
export default function SignupForm() {
  const [state, action, pending] = useActionState(signup, undefined)
 
  return (
    <form action={action}>
      <div>
        <label htmlFor="name">Name</label>
        <input id="name" name="name" placeholder="Name" />
      </div>
      {state?.errors?.name && <p>{state.errors.name}</p>}
 
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" placeholder="Email" />
      </div>
      {state?.errors?.email && <p>{state.errors.email}</p>}
 
      <div>
        <label htmlFor="password">Password</label>
        <input id="password" name="password" type="password" />
      </div>
      {state?.errors?.password && (
        <div>
          <p>Password must:</p>
          <ul>
            {state.errors.password.map((error) => (
              <li key={error}>- {error}</li>
            ))}
          </ul>
        </div>
      )}
      <button disabled={pending} type="submit">
        Sign Up
      </button>
    </form>
  )
}
```

팁1 : React 19에서는 useFormStatus가 반환하는 객체에    
data, method, action 같은 추가 정보들이 포함되어있어서 상세하게 컨트롤 가능   
19이전 버전은 pending 상태만 제공함   
팁2: 데이터를 변경(생성, 수정 등)하기 전에,   
반드시 그 사용자가 해당 작업을 할 수 있는 권한(Authorization)이 있는지 확인   

3. 사용자 생성 또는 로그인 시도 (Create a user or check user credentials)
- 회원가입이라면: 새로운 사용자 계정을 생성하고
- 로그인이라면: 사용자가 이미 존재하는지 확인하고 비밀번호가 맞는지 검사


```tsx
export async function signup(state: FormState, formData: FormData) {
  // 1. Validate form fields
  // ...
 
  // 2. Prepare data for insertion into database
  const { name, email, password } = validatedFields.data
  // e.g. Hash the user's password before storing it
  const hashedPassword = await bcrypt.hash(password, 10)
 
  // 3. Insert the user into the database or call an Auth Library's API
  const data = await db
    .insert(users)
    .values({
      name,
      email,
      password: hashedPassword,
    })
    .returning({ id: users.id })
 
  const user = data[0]
 
  if (!user) {
    return {
      message: 'An error occurred while creating your account.',
    }
  }
 
  // TODO:
  // 4. Create user session
  // 5. Redirect user
}
```
사용자 계정을 성공적으로 생성했거나,   
입력한 사용자 정보가 올바른지 검증이 완료되었다면,   
이제 **세션(session)**을 만들어야 함

세션은 **사용자의 로그인 상태(auth state)**를 추적하는 데 사용됨

세션 관리 방식에 따라, 세션은
- 쿠키에 저장되거나
- 데이터베이스에 저장되거나
- 쿠키와 데이터베이스를 함께 사용하는 방식도 있음

---
세션 관리(Session Management)   :세션 관리는 사용자가 로그인한 상태를 요청이 여러 번 오가는 동안에도 유지시켜주는 역할
- 세션 또는 토큰 생성
- 저장
- 갱신(리프레시)
- 삭제

	1.	무상태(Stateless) 세션
- 브라우저의 쿠키에 저장
- 브라우저는 요청마다 이 쿠키를 서버에 보내고, 서버는 이걸 확인해서 로그인 상태인지 판별
- 구현간단, 상대적으로 보안취약

	2.	데이터베이스 기반 세션
- 세션 데이터를 서버의 데이터베이스에 저장하고, 브라우저에는 암호화된 세션 ID만 전달
- 구현이 복잡하고 서버 자원을 더 많이 사용함

참고:   
두 방식 중 하나만 써도 되고, 둘을 조합해서 써도 됨   
하지만 우리는 iron-session이나 Jose 같은 세션 관리 라이브러리 사용을 추천

--- 
무상태 세션 (Stateless Sessions)
1. 시크릿 키(secret key)를 생성해서 세션에 서명할 때 사용
  1-1.  키는 **환경 변수(env)**로 저장
2. 세션 데이터 암호화/복호화 로직을 작성
3. Next.js의 cookies API를 사용해서 쿠키를 직접 관리 
4. 갱신(refresh), 로그아웃 시에 세션을 삭제하는 기능


시크릿 키 생성 명령어
```
openssl rand -base64 32
```

생성된 시크릿키 env 파일에 넣기
```
// .env
SESSION_SECRET=your_secret_key
```

시크릿키 사용
```
// app/lib/session.js
const secretKey = process.env.SESSION_SECRET
```

암호화하고 복호화하는 단계   
여기서는 앞서 말한 세션 관리 라이브러리 중 Jose를 사용   

```tsx
// app/lib/session.ts
import 'server-only'
import { SignJWT, jwtVerify } from 'jose'
import { SessionPayload } from '@/app/lib/definitions'
 
const secretKey = process.env.SESSION_SECRET
const encodedKey = new TextEncoder().encode(secretKey)
 
export async function encrypt(payload: SessionPayload) {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(encodedKey)
}
 
export async function decrypt(session: string | undefined = '') {
  try {
    const { payload } = await jwtVerify(session, encodedKey, {
      algorithms: ['HS256'],
    })
    return payload
  } catch (error) {
    console.log('Failed to verify session')
  }
}
```

팁: **세션에 담을 데이터(payload)**는 꼭 최소한의, 고유한 정보만 넣어야 함
사용자 ID, 사용자 권한 등    
넣으면 안되는 정보: 전화번호, 이메일, 신요카드정보, 비밀번호 또는 그외 민감정보


쿠키에 셋팅   
권장 옵션들을 함께 반드시 서버에서 설정   
HttpOnly(XSS(스크립트 공격) 방어), Secure(HTTPS 연결에서만), SameSite(CSRF 공격 방지), Max-Age 또는 Expires(유효 기간) , Path


```ts
// app/lib/session.ts
import 'server-only'
import { cookies } from 'next/headers'
 
export async function createSession(userId: string) {
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  const session = await encrypt({ userId, expiresAt })
  const cookieStore = await cookies()
 
  cookieStore.set('session', session, {
    httpOnly: true,
    secure: true,
    expires: expiresAt,
    sameSite: 'lax',
    path: '/',
  })
}
```

여기까지 했으면 Server Action 코드로 돌아가 보면,   
createSession() 함수를 호출해서 세션을 만들고,   
redirect() API를 사용해서 사용자를 로그인 후 이동할 페이지로 리디렉트할 수 있음   

```ts
import { createSession } from '@/app/lib/session'
 
export async function signup(state: FormState, formData: FormData) {
  // Previous steps:
  // 1. Validate form fields
  // 2. Prepare data for insertion into database
  // 3. Insert the user into the database or call an Library API
 
  // Current steps:
  // 4. Create user session
  await createSession(user.id)
  // 5. Redirect user
  redirect('/profile')
}
```

세션 갱신 (Updating / Refreshing Sessions)

사용자가 앱을 다시 방문했을 때도 로그인이 유지되게 하려면,   
**세션의 만료 시간을 연장(refresh)**할 수 있음

예를 들어 로그인한 사용자가 앱에 다시 접근할 때, 세션을 갱신해서 자동 로그아웃이 되지 않도록 유지

```ts
// app/lib/session.ts
import 'server-only'
import { cookies } from 'next/headers'
import { decrypt } from '@/app/lib/session'
 
export async function updateSession() {
  const session = (await cookies()).get('session')?.value
  const payload = await decrypt(session)
 
  if (!session || !payload) {
    return null
  }
 
  const expires = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
 
  const cookieStore = await cookies()
  cookieStore.set('session', session, {
    httpOnly: true,
    secure: true,
    expires: expires,
    sameSite: 'lax',
    path: '/',
  })
}
```

세션삭제
```
// app/lib/session.ts
import 'server-only'
import { cookies } from 'next/headers'
 
export async function deleteSession() {
  const cookieStore = await cookies()
  cookieStore.delete('session')
}
```

```ts
// app/actions/auth.ts
import { cookies } from 'next/headers'
import { deleteSession } from '@/app/lib/session'
 
export async function logout() {
  deleteSession()
  redirect('/login')
}
```
---
2.	데이터베이스 기반 세션

1. 세션 테이블 생성    
데이터베이스에 세션 데이터를 저장할 전용 테이블을 만들어야 함     
또는 Auth 라이브러리에서 자동으로 처리해주는지 확인    
(예: NextAuth.js는 세션 테이블을 자동으로 생성해줌)

2. 세션 생성, 갱신, 삭제 기능 구현
- 사용자가 로그인하면 → 세션을 DB에 삽입(insert)
- 사용자가 다시 방문하면 → 세션 갱신(update)
- 로그아웃 시 → 세션 삭제(delete)

3. 세션 ID 암호화 후 브라우저에 저장(option, recommend)
- 사용자의 브라우저에는 세션 ID만 저장하고,   
이 ID는 암호화된 상태여야 해요.   
- 쿠키와 데이터베이스가 동기화 상태를 유지해야 하고,   
이건 특히 Middleware에서 인증 상태를 빠르게 판단(optimistic auth check) 할 때 도움이 됨

```ts
// app/lib/session.ts
import cookies from 'next/headers'
import { db } from '@/app/lib/db'
import { encrypt } from '@/app/lib/session'
 
export async function createSession(id: number) {
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
 
  // 1. Create a session in the database
  const data = await db
    .insert(sessions)
    .values({
      userId: id,
      expiresAt,
    })
    // Return the session ID
    .returning({ id: sessions.id })
 
  const sessionId = data[0].id
 
  // 2. Encrypt the session ID
  const session = await encrypt({ sessionId, expiresAt })
 
  // 3. Store the session in cookies for optimistic auth checks
  const cookieStore = await cookies()
  cookieStore.set('session', session, {
    httpOnly: true,
    secure: true,
    expires: expiresAt,
    sameSite: 'lax',
    path: '/',
  })
}
```

---

권한 부여(Authorization)   : 사용자가 인증을 마치고 세션이 생성된 후에는, 이제 그 사용자가 앱 내에서 어떤 작업을 할 수 있는지 제어

권한 체크의 두 가지 방식
	1.	Optimistic (낙관적 체크)
- 브라우저의 쿠키에 저장된 세션 데이터를 기반으로   
사용자가 특정 페이지나 기능에 접근할 수 있는지 빠르게 확인
예시:
- 관리자 UI를 숨기거나 보여줄 때
- 특정 역할(role)에 따라 페이지로 리디렉션할 때
- 빠른 반응이 필요한 작업에 적합하지만, 보안 수준은 낮음

2.	Secure (보안 중심 체크)
- 데이터베이스에 저장된 세션 정보를 기반으로 접근 권한을 확인

예시:
- 중요한 데이터를 가져올 때
- 민감한 기능(예: 결제 처리)에 접근할 때
- 더 안전한 방식이지만, 성능은 조금 느릴 수 있어요.

두 방식 모두에 적용 가능한 추천 패턴
- **Data Access Layer (DAL)**를 만들어서 모든 권한 로직을 한 곳에 모아 관리하세요.
- **DTO (Data Transfer Object)**를 사용해서 필요한 데이터만 클라이언트에 전달하세요.
- 선택적으로 Middleware를 사용해서 optimistic 체크도 할 수 있음

 Middleware를 활용한 Optimistic 체크 (선택사항)
 	접근 권한이 없는 사용자를 미리 걸러내거나 리디렉션할 때
	•	정적(static) 경로를 보호할 때, 예: 유료 콘텐츠
 DB에서 세션을 읽지 말고, 오직 쿠키에 있는 세션 정보만 읽어야함 (= 즉, Middleware에서는 “optimistic 체크만” )


중요한 보안 검사는 데이터와 가장 가까운 곳, 즉 서버나 DB 접근 직전에 수행해야 함
이를 위해 **Data Access Layer(DAL)**를 사용하는 것이 좋음

Data Access Layer(DAL) 만들기

DAL은 데이터 요청과 권한 검증 로직을 한 곳에 모아두는 구조예요.
이렇게 하면 코드 재사용성도 높고, 보안도 강화돼요.
사용자의 세션을 검증하는 함수 (예: verifySession())
- 세션이 유효한지 검사하고
- 유효하지 않으면 리디렉션하거나
- 유효하면 사용자 정보를 반환해야 해요

```ts
  //app/lib/dal.ts
  import 'server-only'
 
import { cookies } from 'next/headers'
import { decrypt } from '@/app/lib/session'
 
export const verifySession = cache(async () => {
  const cookie = (await cookies()).get('session')?.value
  const session = await decrypt(cookie)
 
  if (!session?.userId) {
    redirect('/login')
  }

 // 이 함수의 반환값을 **렌더링 중에만 메모이즈(캐싱)**하면 성능도 최적화
  return { isAuth: true, userId: session.userId }
})
```

그다음에는 이 verifySession() 함수를   
다음과 같은 곳에서 호출해서 사용하면 됨
- 데이터 요청할 때 (예: getServerSideProps, server functions, etc)
- Server Actions 내부에서
- Route Handlers (app/api/ 안의 API 핸들러)


이렇게 하면:
- 사용자 세션이 유효한지 확인하고
- 유효하지 않다면 요청을 막거나 리디렉션 처리하고
- 유효하다면 필요한 사용자 정보를 반환해서 다음 작업에 사용

이런 흐름을 모든 서버 요청마다 일관되게 처리할 수 있음

```ts
// app/lib/dal.ts
export const getUser = cache(async () => {
  const session = await verifySession()
  if (!session) return null
 
  try {
    const data = await db.query.users.findMany({
      where: eq(users.id, session.userId),
      // Explicitly return the columns you need rather than the whole user object
      columns: {
        id: true,
        name: true,
        email: true,
      },
    })
 
    const user = data[0]
 
    return user
  } catch (error) {
    console.log('Failed to fetch user')
    return null
  }
})
```

---
데이터 전송 객체 (DTO, Data Transfer Objects) 사용하기

데이터를 가져올 때는,
애플리케이션에서 실제로 사용할 정보만 반환하는 게 좋아요.
→ 전체 객체를 그대로 반환하는 건 피하는 게 좋습니다.

DTO는 보안도 지키고, 성능도 최적화하는 좋은 방법

```ts
// app/lib/dto.ts
import 'server-only'
import { getUser } from '@/app/lib/dal'
 
function canSeeUsername(viewer: User) {
  return true
}
 
function canSeePhoneNumber(viewer: User, team: string) {
  return viewer.isAdmin || team === viewer.team
}
 
export async function getProfileDTO(slug: string) {
  const data = await db.query.users.findMany({
    where: eq(users.slug, slug),
    // Return specific columns here
  })
  const user = data[0]
 
  const currentUser = await getUser(user.id)
 
  // Or return only what's specific to the query here
  return {
    username: canSeeUsername(currentUser) ? user.username : null,
    phonenumber: canSeePhoneNumber(currentUser, user.team)
      ? user.phonenumber
      : null,
  }
}
```

데이터 요청과 권한 검사 로직을 DAL(Data Access Layer)에 모아두고,
DTO(Data Transfer Object)를 사용해서 필요한 데이터만 전달

---

Server Components

서버 컴포넌트에서 인증(권한) 체크를 하는 건,
특히 역할(Role)에 따라 UI를 다르게 보여줘야 할 때 매우 유용
**관리자(admin)**는 설정 페이지를 볼 수 있고
	•	**일반 사용자(user)**는 마이페이지만 보며
	•	**비로그인 사용자(unauthorized)**는 아무것도 못 보는 식으로 구성

 ---

 레이아웃(Layouts)에서의 인증 체크 주의사항

 Next.js의 Partial Rendering(부분 렌더링) 때문에,
Layout에서 인증 체크를 할 때는 주의가 필요해요.

왜냐하면:

레이아웃은 페이지 이동 시 다시 렌더링되지 않기 때문에,
매번 사용자 세션을 다시 체크하지 않게 될 수 있어요.

따라서 인증 체크는 다음 두 곳 중 하나에서 수행하는 게 좋아요:
	1.	데이터를 불러오는 곳 근처
	2.	조건부로 컴포넌트를 렌더링하는 곳 내부

	•	❌ 잘못된 방식:
→ Layout에서 직접 “이 사용자가 로그인한 사용자냐?“를 검사함
→ 문제: 페이지 이동해도 Layout은 재실행되지 않기 때문에 체크가 안 될 수 있어요.
	•	✅ 추천 방식:
→ Layout 안에서는 그냥 getUser() 함수로 사용자 데이터를 가져오고,
→ 그 getUser() 함수 안에서 DAL을 통해 인증 체크를 수행하도록 만드는 거예요.

----

Server Actions

Server Actions도 외부에서 접근 가능한 API처럼
보안에 신경 써서 다뤄야 해요.

즉, 사용자가 해당 작업(데이터 수정 등)을 수행할 권한이 있는지
꼭 확인

----

Route Handlers (app/api/)

Route Handler도 마찬가지로,
공개된 API와 똑같이 보안적으로 신중하게 다뤄야 해요.

누군가 fetch('/api/secret') 같은 식으로
브라우저에서 직접 접근할 수 있기 때문이에요.
