# 1. Images
## 1.1 OnThis page

Web Almanac에 따르면, 이미지들은 일반 웹사이트 페이지 무게의 큰 부분을 차지하며,   
웹사이트의 LCP 성능에 상당한 영향을 미칠 수 있습니다.   
Next.js Image 컴포넌트는 HTML <img> 요소를 확장하여 자동 이미지 최적화를 위한 기능을 제공합니다:

- Size Optimization:
  - 각 기기에 맞는 적절한 크기의 이미지를 자동으로 제공하며, WebP와 같은 최신 이미지 포맷을 사용합니다.

- Visual Stability:
  - 이미지가 로딩되는 동안 레이아웃 이동(shift)을 자동으로 방지합니다.

- Faster Page Loads:
  - 이미지는 뷰포트에 들어올 때만 로드되며, 네이티브 브라우저 지연 로딩(lazy loading)과 선택적 블러업(blur-up) 플레이스홀더를 사용합니다.

- Asset Flexibility:
  - 원격 서버에 저장된 이미지를 포함하여, 필요에 따라 이미지 크기를 동적으로 조정할 수 있습니다.

---

## 1.2 Usage
```
import Image from 'next/image'
```
### 1.2.1 Local Images
로컬 이미지를 사용하려면, .jpg, .png, 또는 .webp 이미지 파일들을 import 하세요.

Next.js는 import된 파일을 기반으로 
이미지의 내재된 너비(intrinsic width)와 높이(intrinsic height)를 자동으로 결정합니다.   
이 값들은 이미지의 비율을 결정하고,   

이미지 로딩 중 누적 레이아웃 이동(Cumulative Layout Shift)을 방지하는 데 사용됩니다.   
(갑자기 화면 와락 움직이는거)   
=로컬 이미지는 내 로컬에있으니까 사이즈를 이미 판별해준다   
```tsx
import Image from 'next/image'
// 로컬 이미지 파일을 import 합니다.
import exampleImg from '../public/example.jpg'

export default function LocalImageComponent() {
  return (
    <div>
      {/* Next.js Image 컴포넌트를 사용하여 로컬 이미지를 렌더링합니다.
          Next.js가 exampleImg 파일을 기반으로 자동으로 너비와 높이를 결정합니다. */}
      <Image
        src={exampleImg}     // 로컬 이미지 파일을 소스로 사용
        alt="Example image"  // 이미지에 대한 대체 텍스트
      // width={500} automatically provided
      // height={500} automatically provided
      // blurDataURL="data:..." automatically provided
      // placeholder="blur" // Optional blur-up while loading
      />
    </div>
  )
}
```

동적으로 import() 또는 require()를 사용하면 안 됨.   
정적으로 가져와야 Next가 빌드시 분석가능   
import myModule from 'myModule' // ✅ 정적 import


### 1.2.2 Remote Images
원격 이미지 (Remote Images)   
- 원격 이미지를 사용하려면, src 속성에 URL 문자열을 사용해야 합니다.   
- Next.js는 빌드 과정에서 원격 파일에 접근할 수 없기 때문에, width, height, (선택적으로) blurDataURL 속성을 수동으로 제공해야 합니다.   
- width와 height 속성은 이미지의 올바른 종횡비를 추론하고, 이미지 로딩 시 발생할 수 있는 레이아웃 이동을 방지하는 데 사용됩니다.   
  단, 이 속성들은 실제 렌더링되는 이미지 크기를 결정하지 않습니다.   

공홈 소스 참고
첫 번째 예제 (app/page.js):    
Next.js의 <Image> 컴포넌트를 사용해서 원격 이미지를 렌더링하는 방법을 보여줌

두 번째 예제 (next.config.js):   
Next.js가 최적화를 위해 원격 이미지의 출처를 안전하게 제한할 수 있도록 설정하는 방법을 보여줌

추가 사항:   
상대 경로를 사용하여 이미지 src를 지정하고 싶다면, 로더(loader)를 사용해야 합니다.

### 1.2.3 Domains
원격 이미지를 최적화할 때, Next.js의 내장 이미지 최적화 API를 사용하려면 기본 로더(loader) 설정을 그대로 두고,   
Image 컴포넌트의 src 속성에 절대 URL을 입력하면 됩니다.   

애플리케이션을 악의적인 사용으로부터 보호하기 위해,   
next/image 컴포넌트에서 사용하려는 원격 호스트 이름의 목록을 정의해야 합니다.
```tsx
import Image from 'next/image'

export default function Page() {
  return (
    <div>
      {/* 
      Next.js Image 컴포넌트를 사용하여 원격 이미지를 렌더링합니다.
      - 기본 로더를 사용하므로 특별한 loader 옵션을 지정하지 않습니다.
      - src 속성에 절대 URL을 입력합니다.
      */}
      <Image
        src="https://example.com/my-image.jpg" // 절대 URL
        alt="Example image"
      />
    </div>
  )
}
```
```
/*
next.config.js 예제

Next.js에서 내장 이미지 최적화 API를 안전하게 사용하려면,
허용할 원격 호스트를 명시적으로 설정해야 합니다.
아래 설정은 'example.com' 도메인에서 오는 이미지들만 허용하도록 구성합니다.
*/
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',            // HTTPS 프로토콜 사용
        hostname: 'example.com',        // 허용할 호스트 이름
        port: '',                     // 기본 포트 사용
        pathname: '/**',              // 모든 경로에 대해 허용
      },
    ],
  },
}
```
### 1.2.4 Loaders
- 앞서 예제에서 로컬 이미지에 대해 부분 URL ("/me.png")이 제공된 것을 볼 수 있습니다.
  - 이는 로더 아키텍처 덕분에 가능한 것입니다.

- 로더는 이미지의 URL을 생성하는 함수입니다.
  - 제공된 src를 수정하고, 다양한 크기로 이미지를 요청하기 위한 여러 URL을 생성합니다.
  - 이 여러 URL은 자동으로 srcset을 생성하는 데 사용되어, 방문자에게 뷰포트에 맞는 적절한 크기의 이미지가 제공됩니다.

- Next.js 애플리케이션의 기본 로더는 내장 이미지 최적화 API를 사용합니다.
  - 이 API는 웹상의 어디서든 이미지를 최적화하고, Next.js 웹 서버에서 직접 제공됩니다.

- 만약 이미지를 CDN이나 이미지 서버에서 직접 제공하고 싶다면,
  - 몇 줄의 JavaScript로 자신만의 로더 함수를 작성할 수 있습니다.

- 각 이미지에 대해 loader prop을 사용하거나,
  - 애플리케이션 수준에서는 loaderFile 설정을 통해 로더를 정의할 수 있습니다.
 

```
// 예제: Custom Loader를 사용한 이미지 로딩

import Image from 'next/image'

// myLoader: 사용자 정의 로더 함수
// - src: 원본 이미지 경로
// - width: 요청된 이미지 너비
// - quality: 이미지 품질 (선택적, 기본값은 75)
const myLoader = ({ src, width, quality }) => {
  // CDN 또는 이미지 서버의 URL 형식에 맞게 URL을 생성합니다.
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}

export default function MyImage() {
  return (
    <div>
      {/*
        loader prop을 사용하여 사용자 정의 로더(myLoader)를 지정합니다.
        src에 부분 경로("me.png")를 사용할 수 있으며, 이 경우 myLoader 함수가
        전체 URL을 생성하여 적절한 크기의 이미지를 요청합니다.
      */}
      <Image
        loader={myLoader}        // 사용자 정의 로더 함수 지정
        src="me.png"             // 부분 URL (예: 로컬 파일 경로)
        alt="My image"           // 이미지 대체 텍스트
        width={500}              // 요청할 이미지 너비 (종횡비 추론용)
        height={500}             // 요청할 이미지 높이 (종횡비 추론용)
      />
    </div>
  )
}

```
---


## 1.3 Priority
- 각 페이지에서 Largest Contentful Paint (LCP) 요소가 되는 이미지에는 priority 속성을 추가해야 합니다.
  - 이는 Next.js가 해당 이미지를 미리 로드(preload)하도록 하여 LCP 성능을 향상시킵니다.

- LCP 요소는 일반적으로 페이지 뷰포트에 보이는 가장 큰 이미지나 텍스트 블록입니다.
  - next dev를 실행할 때, 만약 LCP 요소가 priority 속성이 없는 <Image>라면 콘솔에 경고가 표시됩니다.

- LCP 이미지를 식별한 후, 아래와 같이 priority 속성을 추가할 수 있습니다.

```js
  import Image from 'next/image'
import profilePic from '../public/me.png'
 
export default function Page() {
    // Next.js Image 컴포넌트에 priority 속성을 추가하여,
    // 이 이미지가 LCP 요소임을 표시하고, 미리 로드되도록 합니다.
  return <Image src={profilePic} alt="Picture of the author" priority />
}
```
---


## 1.4 Image Sizing
- 문제점:   
  - 이미지 로딩 시 크기가 지정되지 않으면, 이미지가 로드되는 동안 페이지의 다른 요소들이 밀리면서 레이아웃 쉬프트가 발생합니다.
  - 이는 사용자 경험에 악영향을 미치고 Core Web Vitals의 중요한 지표인 Cumulative Layout Shift에 부정적인 영향을 줍니다.

- 해결 방법:
  - next/image 컴포넌트는 레이아웃 쉬프트를 방지하기 위해 이미지를 반드시 크기 지정해야 하며, 이를 위한 세 가지 방법을 제공합니다:
  - 자동 크기 지정:
    - 정적 import를 사용하여 이미지의 내재된 크기를 자동으로 결정.
  - 수동 크기 지정:
    - width와 height 속성을 명시적으로 제공하여 이미지의 종횡비를 고정.
  - 암시적 크기 지정:
    - fill 속성을 사용하여 이미지가 부모 요소의 크기에 맞게 확장되도록 함.
  - 크기 정보를 모를 경우:
    - fill 속성을 사용하여 부모 요소의 크기에 따라 이미지 크기를 조정.
    - 이미지 파이프라인을 통해 이미지를 정규화하거나,
    - API 호출을 수정해 이미지 URL과 함께 크기 정보를 반환하도록 함.
    - 
---


## 1.5 Styling
html 스타일 쓴다 . styled-jsx쓰지말라함. (정적이지 않기때문에)

fill 쓸때는 position: relative로 써야한다

---


## 1.6 Examples
## 1.6.1 Responsive
반응형 display :flex 수직으로 레이아웃 배치됨 
로컬이미지는 자동계산됨

## 1.6.2 Fill Container
로컬이미지를 fill을 써서ㅓ div relative에 맞추겠따는 뜻

## 1.6.3 Background Image

---


## 1.7 Other Properties

---


## 1.8 Configuration
API Reference
