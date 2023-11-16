# Next.JS 에 들어가기 전 리액트
~~~
리액트는 라이브러리냐 프레임워크냐하는 질문을 많이 받아본 것 같다.
(아직까지는 아직까지는 라이브러리라고 생각한다. 하지만 기능이 다양화되고 문서가 많아지며 best practice들이 많이 나오고있어 시간이 조금 더 지나면 프레임워크가 되지 않을까하는 기대가 있다)

라이브러리는 말그대로 재사용가능한 코드들의 집합이며 개발자가 제어하고,
프레임워크는 코드의 구조에 맞춰 개발자가 코드를 작성한다.

제어의 방식은 라이브러리는 개발자가 제어하고, 프레임워크는 프레임워크에 맞춰 개발자의 코드를 제어한다.
의존성의 경우 프레임워크는 필수 라이브러리는 선택

라이브러리는 더 많은 유연성을 제공하지만, 프레임워크는 구조가 정해져있어 빠른속도로 개발이 가능하고 유지보수가 용이하다는 장점이 있다.
~~~

## Next.JS 프레임워크와 SSR, ISR, SSG
~~~
리액트를 사용해봤다면 Next.JS 프레임워크에 대한 러닝컨브는 그렇게 높지않다.

CSR -> 클라이언트 사이드렌더링 (빈화면 주고 클라이언트에서 html을 그리리고 js코드를 실행한다) React
SSR -> 서버에서 html을 그리고 클라이언트 사이드에서 js를 실행한다 Next


~~~

## Next.js 에서 pages 폴더는 일종의 라우터 역할을 한다.
~~~
src
 ㄴ pages
    ㄴ _app.tsx
    ㄴ index.tsx
    ㄴ preview.tsx
    ㄴ search
      ㄴ [word].tsx
      ㄴ index.tsx
    ㄴ setting
      ㄴ auto-complete.tsx
      ㄴ site.tsx

index.tsx는 루트 경로에 해당하는 페이지 (메인페이지)
setting/site <- 이런 Url 경로가 만들어진다. [word] 는 동적 사이트이다.
_app.tsx는 모든 페이지를 초기화하고 재설정할 수 있는 페이지, 404, 500등 페이지가 존재한다
~~~


## SSR (Server side rendering)
~~~ javascript
export default SSRPage = ({dateTime) : SSRPageProps) => {
  return (
    <main>
      <TimeSection dateTime={dateTime) /> 
    </main>
  )
}

export const getServerSideProps: GetServerSideProps = async() => {
  const res = await axios.get('http://worldtimeapi.org/api/ip');

  return {
    props: {datime: res.data.dateTime}
  }
}
// 클이언트에서 페이지를 요청할 때 getServerSideProps를 실행하고 값을 SSRPage에 전달하여 그린후(렌더링한후) 클라이언트에 전달
~~~

## SSG(static side generation)
~~~javascript
export default SSGPage = ({dateTime) : SSRPageProps) => {
  return (
    <main>
      <TimeSection dateTime={dateTime) /> 
    </main>
  )
}

export const getStaticSideProps: GetStaticSideProps = async() => {
  const res = await axios.get('http://worldtimeapi.org/api/ip');

  return {
    props: {datime: res.data.dateTime}
  }
}
// nextjs에서 페이지 생성할 때 기본으로 적용되는 설정 -> 빌드시에 페이지 미리 생성
// 이 경우에도 빌드시에 getStaticSideProps실행하고 리턴하면 SSGPage를 받아서 랜더링한다.
// 빌드시점에 생기는거라서 시간데이터가 변경되더라도 다시 빌드하기전까지 반영 안됨
~~~

## ISR (Incremental static rendering) SSG랑 비슷한데 일정 시간 지나면 새로 랜더링(=업데이트)
~~~javascript
export default ISR20Page = ({dateTime) : ISR20PageProps) => {
  return (
    <main>
      <TimeSection dateTime={dateTime) /> 
    </main>
  )
}

export const getIncrementalStaticProps: GetIncrementalStaticProps = async() => {
  const res = await axios.get('http://worldtimeapi.org/api/ip');

  return {
    props: {datime: res.data.dateTime},
    revalidate: 20 // 20초마다 새로 렌더링됨
  }
}
// 빌드시에 getIncrementalStaticProps 리턴하면 ISR20Page 받아서 랜더링한다.
// 20초마다 새로 랜더링한다.
~~~
