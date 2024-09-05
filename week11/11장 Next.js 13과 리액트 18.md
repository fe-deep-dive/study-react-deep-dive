
# 11장 Next.js 13과 리액트 18

Next.js13은 많은 변화가 있었다. 크게 눈에 띄는 변화는 이렇다. 
- 리액트 18을 채택
- 레이아웃 지원 강화
- 바벨 대신 러스트 기반 SWC 출시
- 웹팩을 대체할 Turbopack 출시

## 11.1 app 디렉터리 등장

**13 버전 이전의 공통 레이아웃 구현 방법**

모든 페이지는 각각의 물리적으로 구별된 파일로 독립되어 있다. 페이지 공통으로 무언가 집어 넣을 수 있는 곳은 _document와 _app이 유일하다. 이 파일들은 서로 다른 목적을 지니고 있다.

- _document: 페이지에서 쓰이는 `<html>`, `<body>` 태그를 수정하거나, 서버 사이드 렌더링 시 styled-components같은 일부 CSS-in-JS를 지원하기 위한 코드를 삽입하는 용도이다. 오직 서버에서만 작동하므로 이벤트 핸들러나 클라이언트 로직을 붙이는 것을 금지하고 있다.
- _app: _app은 페이지를 초기화하기 위한 용도로 다음과 같은 작업이 가능하다고 명시돼 있다.
    - 페이지 변경 시에 유지하고 싶은 레이아웃
    - 페이지 변경 시 상태유지
    - componentDidCatch를 활용한 에러 핸들링
    - 페이지간 추가적인 데이터 삽입
    - global CSS 주입

페이지 공통 레이아웃을 유지할 수 있는 방법은 _app이 유일했다. 그러나, 이 방식은 _app에서만 가능해 제한적이고, 각 페이지별 서로 다른 레이아웃을 유지할 수 있는 여지도 부족하다. 이러한 한계를 극복하기 위해 Next.js의 app 레이아웃이 등장했다.

/app 라우팅은 베타 버전이기 때문에 next.config.js에 옵션을 활성화해야 사용할 수 있다. 

```jsx
const nextConfig = {
  experimental: {
    appDir: true,
  },
  // ...
}
```

Next.js 13.4.0 이상 버전을 사용한다면 해당 옵션을 활성화하지 않아도 된다.

## 11.1.1 라우팅

**라우팅을 정의하는 법**

파일 시스템 기반으로 라우팅이 되지만 약간의 차이가 있다. 

- Next.js 12 이하: /pages/a/b.tsx 또는 pages/a/b/index.tsx는 모두 동일한 주소로 변환된다. 즉, 파일명이 index라면 이 내용은 무시된다.
- Next.js 13 app: /app/a/b는 /a/b로 변환되며, 파일명은 무시된다. 폴더명까지만 주소로 변환된다.

즉, Next.js 13의 app 디렉터리 내부의 파일명은 라우팅 명칭에 영향을 주지 않는다. app 내부에서 가질 수 있는 파일명은 예약어로 제한된다. 

### **layout.js**

layout.js는 페이지의 기본적인 레이아웃을 구성하는 요소이다. 해당 폴더에 layout이 있다면 그 하위 폴더 및 주소에 모두 영향을 준다.


```jsx
// app/layout.tsx
import { ReactNode } from 'react'

export default function Layout({ children }: { children: ReactNode }) {
  return (
    <html lang="ko">
      <head>
        <title>안녕하세요!</title>
      </head>
      <body>
        <h1>웹페이지에 오신 것을 환영합니다.</h1>
        <main>{children}</main>
      </body>
    </html>
  )
}

// app/blog/layout.tsx
import { ReactNode } from 'react'

export default function BlogLayout({ children }): { children: ReactNode }) {
  return <section>{children}</section>
}
```

루트에는 단 하나의 Layout을 만들어 둘 수 있다. 이 layout은 모든 페이지에 영향을 미치는 공통 레이아웃이다. html, head 등 공통적인 내용을 다루는 곳으로 보면 된다.

<details>
  <summary>
    styled-components와 같은 CSS-in-js의 초기화
  </summary>

  styled-components와 같은 CSS-in-js의 초기화는 _document에서 CSS-in-JS의 스타일을 모두 모은 다음, 서버 사이드 렌더링 시에 이를 함께 렌더링하는 방식으로 적용했다. _document가 없어짐으로써 루트의 레이아웃에 적용하는 방식으로 변경되었다. 

```jsx
// lib/registry.tsx
'use client'
 
import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'
 
export default function StyledComponentsRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet())
 
  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement()
    styledComponentsStyleSheet.instance.clearTag()
    return <>{styles}</>
  })
 
  if (typeof window !== 'undefined') return <>{children}</>
 
  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  )
}

// app/layout.tsx
import StyledComponentsRegistry from './lib/registry'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  )
}
```

- use client: 클라이언트 컴포넌트를 의미하는 지시자로, 리액트 18에서 새롭게 등장한 개념이다. 자세한 내용은 11.2절에서 알아보자.
- useServerInsertedHTML: 과거 useFlushEffects라는 이름의 훅이었는데, 좀 더 명확한 useServerInsertedHTML로 변경됐다. 이 훅은 10.2절에서 설명한 useInsertionEffect를 기반으로 하는 훅으로, CSS-in-JS 라이브러리와 같이 서버에서 추가해야 할 HTML을 넣는 용도로 만들어졌다.

즉, _document에서 추가하던 서버 사이드 스타일을 이제 새로운 방식을 활용해 루트의 layout에서 집어넣게끔 변경했다. [Next.js 문서 참고](https://nextjs.org/docs/app/building-your-application/styling/css-in-js#styled-components)

</details>

그리고 페이지 하위에 추가되는 layout은 해당 주소 하위에만 적용된다. 앞의 레이아웃을 활용하면 다음과 같은 구조가 된다. 

```html
<html lang="ko">
  <head>
    <title>안녕하세요!</title>
  </head>
  <body>
    <h1>웹페이지에 오신 것을 환영합니다.</h1>
    <main><section>{무언가...}</section></main>
  </body>
</html>
```

layout을 이용해 주소별 공통 UI를 포함할 수 있고 _app과 _document를 대신해 웹페이지를 시작하는 데 필요한 공통 코드를 삽입할 수 있다. 그리고 이 공통 코드는 오로지 자신과 자식 라우팅에만 영향을 끼쳐 레이아웃을 더욱 유연하게 구성할 수 있게 됐다. 

또 다른 장점은 _document.jsx에서만 처리할 수 있었던 부자연스러움이 사라졌다. 기존에는 이나 에 스타일을 추가하는 등의 작업을 하려면 _document.jsx를 사용해야 했을 뿐만 아니라 Next.js에서 제공하는 태그인 <HTML/>, <Body/>, <Head/>를 사용해야 했다. 그러나 app 디렉터리에서는 좀 더 자연스럽게 코드를 작성할 수 있게 됐다.

**layout의 주의 사항**

- layout은 app 디렉터리 내부에서는 예약어이다. 무조건 layout.{js|jsx|ts|tsx}로 사용해야 하고 다른 목적으로는 사용할 수 없다.
- layout은 children을 props로 받아 렌더링해야 한다.
- layout 내부에는 반드시 export default로 내보내는 컴포넌트가 있어야 한다.
- layout 내부에서도 api요청과 같은 비동기 작업을 수행할 수 있다.

### **page.js**

page.js는 일반적으로 다뤘던 페이지를 의미한다. page과 다음과 같이 구성돼 있다고 가정해 보자. 

```jsx
export default function BlogPage() {
  return <>여기에 블로그 글</>
}
```

이 page는 앞에서 구성했던 layout을 기반으로 리액트 컴포넌트를 노출한다. 이 page가 받은 props는 다음과 같다. 

- params: 옵셔널 값으로, 앞서 설명한 […id]와 같은 동적 라우트 파라미터를 사용할 경우 해당 파라미터에 값이 들어온다.
- searchParams: URL의 `?a=1` 과 같은 URLSearchParams를 의미한다. `{ a : ‘1’ }` 이라는 객체 값이 오게 된다. 이 값은 layout에서는 제공하지 않는다. 그 이유는 layout은 페이지 탐색 중에는 리렌더링을 수행하지 않기 때문이다. search parameter에 의존적인 작업을 해야 한다면 page 내부에서 수행해야 한다.

page는 다음과 같은 규칙을 가지고 있다.

- page.{js|jsx|ts|tsx}로 사용해야 한다.
- export default로 내보내는 컴포넌트가 있어야 한다.

### **error.js**

**error.js는 해당 라우팅 영역에서 사용하는 공통 에러 컴포넌트이다. 이 error.js를 사용하면 특정 라우팅별로 서로 다른 에러 UI를 렌더링할 수 있다.**

```jsx
'use client'

import { useEffect } from 'react'

export default function Error({ error, reset }: {
  error: Error,
  reset: () => void
}) {
  useEffect(() => {
    // eslint-disable-next-line no-console
    console.log('logging error:', error)
  }, [error])

  return (
    <div className="space-y-4">
      <div className="text-sm text-vercel-pink">
        <strong className="font-bold">Error:</strong> {error?.message}
      </div>
      <div>
        <button
          className="rounded-lg px-3 py-1 text-sm font-medium bg-gray-700 text-gray-100 hover:bg-gray-500 hover:text-white"
          onClick={() => reset()}
        >
          에러 리셋
        </button>
      </div>
    </div>
  )
}
```

error 페이지는 에러 정보를 담고 있는 `error: Error` 객체와 에러 바운더리를 초기화할 `reset: () ⇒ void`를 props로 받는다. 주의할 점은 에러 바운더리는 클라이언트에서만 동작하므로 클라이언트 컴포넌트여야 한다. 그리고 layout에서 에러가 발생할 경우 error 컴포넌트로 이동하지 않는다. layout에서 발생한 에러를 처리하고 싶다면 상위 컴포넌트의 error를 사용하거나, app의 루트 에러 처리를 담당하는 app/global-error.js 페이지를 생성하면 된다. 

reset()를 통해 오류에서 복구를 시도할 수 있다. 에러 바운더리의 내용을 리렌더링 시도하고 성공하면 리렌더링한 결과로 대체된다. 

### not-found.js

특정 라우팅 하위의 주소를 찾을 수 없는 404 페이지를 렌더링할 때 사용된다. error와 마찬가지로 전체 애플리케이션에서 404를 노출하고 싶다면 app/not-found.js를 생성하면 된다. 이 컴포넌트는 서버 컴포넌트로 구성하면 된다. 

### loading.js

리액트 Suspense를 기반으로 해당 컴포넌트가 불러오는 중임을 나타날 때 사용할 수 있다. ‘use client’ 지시자를 사용해 클라이언트에서 렌더링되게 할 수도 있다.

### route.js

/pages/api와 동일하게 /app/api를 기준으로 디렉터리 라우팅을 지원하며, /api에 대해서도 파일명 라우팅이 없어졌다. 그 대신 디렉터리가 라우팅 주소를 담당하며 파일명은 route.js로 통일됐다. 

route.ts 파일 내부에 REST API의 get, post와 같은 매서드명을 예약어로 선언하면 HTTP 요청에 맞게 해당 메서드를 호출한다. 한 가지 흥미로운 점은 app/api외에도 다른 곳에서도 선언해도 작동한다.

```jsx
// app/internal-api/hello/route.ts
// app/api가 아니어도 동작한다. 
import { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  return new Response(JSON.stringify({ name: 'hello' }), {
    status: 200,
    headers: {
      'content-type': 'application/json',
    },
  })
}
```
라우팅 명칭에 자유도가 생긴 대신 route.ts가 존재하는 폴더 내부에는 page.tsx가 존재할 수 없다. 

이 route의 함수들이 받을 수 있는 파라미터는 다음과 같다.

- request: NextRequest 객체이며, fetch의 Request를 확장한 Next.js만의 Request이다. 이 객체에는 API요청과 관련된 cookie, headers 등을 포함해 nextUrl 같은 주소 객체도 확인할 수 있다.
- context: params만을 가지고 있는 객체이며, 동적 라우팅 파리미터 객체가 포함돼 있다. 이 객체는 Next.js에서 별도 인터페이스를 제공하지 않으므로 주소의 필요에 따라 원하는 형식으로 선언하면 된다.

```jsx
// app/api/posts/[id]/route.ts
import type { NextRequest } from 'next/server'

export async function GET(
  req: NextRequest,
  context: { params: { id: string } }, // 주소의 필요에 따라 원하는 형식으로 선언
) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${context.params.id}`,
  )
	// ...
  return new Response(JSON.stringify(result), {
    status: 200,
    headers: {
      'content-type': 'application/json',
    },
  })
}
```

## 11.2 리액트 서버 컴포넌트

### 11.2.1 기존 리액트 컴포넌트와 서버 사이드 렌더링의 한계

먼저, 리액트 컴포넌트와 서버 사이드 렌더링 개념을 복습해 보자.  

리액트 컴포넌트는 클라이언트에서 작동하며, 브라우저에서 자바스크립트 코드 처리가 이뤄진다. 예를 들어, 리액트로 만들어진 페이지를 방문하면 실행에 필요한 코드를 다운로드하고 리액트 컴포넌트 트리를 만든 후, DOM에 렌더링한다. 

서버 사이드 렌더링은 미리 서버에서 DOM을 만들어 오고, 클라이언트는 이렇게 만들어진 DOM을 기준으로 하이드레이션을 진행한다. 이후 브라우저에서는 상태를 추적하고, 이벤트 핸들러를 DOM에 추가하고, 응답에 따라 렌더링 트리를 변경하기도 한다. 

이 구조는 한계점이 있다. 

- 자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없음: 게시판 등 사용자가 작성한 HTML에 위험한 태그를 제거하기 위해 사용되는 유명한 npm 라이브러리로 sanitize-html이 있다.
```jsx

import sanitizeHtml from 'sanitize-html' // 206K (63.3K gzipped)

function Board({ text }: { text: string }) {
	const html = useMemo(() => sanitizeizeHtml((text), [text]);
	
	return <div dangerouslySetInnerHTML={{ __html: html }} />
}
```
이 컴포넌트는 63.3KB에 달하는 sanitize-html을 필요로해 브라우저에서 해당 라이브러리를 다운로드하고 실행까지 거쳐야 한다. 이는 사용자 기기의 부담을 주게 된다. 이 컴포넌트를 서버에서만 렌더링하고 클라이언트는 결과만 받게 한다면 클라이언트는 무거운 라이브러리를 다운로드, 실행하지 않고도 사용자에게 컴포넌트를 렌더링 할 수 있다.

- 백엔드 리소스에 대한 직접적인 접근 불가능: 클라이언트에서 직접 데이터베이스에 액세스하거나 백엔드의 파일 시스템에 직접 접근해 클라이언트에 데이터를 제공하기 위한 수고로움이 줄어든다.
  ```jsx
  import db from 'db'
  
  async function Board({ id }: { id: string }) {
    const text = await db.board.get(id);
    return <>text</>
  }
  ```

- 자동 코드 분할이 불가능: 코드 분할이란 하나의 거대한 코드 번들 대신, 코드를 여러 작은 단위로 나누어 필요할 때만 동적으로 지연 로딩해 앱을 초기화하는 속도를 높여주는 기법이다. 리액트의 lazy를 통해 구현할 수 있다. 그러나 lazy를 일일이 감싸주어야 하므로 누락하는 실수가 생길 수 있다. 그리고 컴포넌트가 호출되고(Photo) if 문을 판단하기 전까지 어떤 지연 로딩한 컴포넌트를 불러올지 결정할 수 없어 성능상 이점을 누릴 수 없다.
  ```jsx
  import { lazy } from 'react'
  
  const OldPhotoRenderer = lazy(() => import('./OldPhotoRenderer.js'));
  const NewPhotoRenderer = lazy(() => import('./NewPhotoRenderer.js'));
  
  function Photo(props) {
    if (FeatureFlags.useNewPhotoRenderer) {
      return <NewPhotoRenderer {...props} />
    } else {
      return <OldPhotoRenderer {...photos} />
    }
  }
  ```
  이 코드 분활을 서버에서 자동으로 수행해 준다면 자연스럽게 성능 이점을 누릴 수 있고, 어떤 컴포넌트를 미리 불러와서 클라이언트에 내려줄지 서버에서 결정해 코드 분활 이점을 활용할 수 있다.

- 연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어려움: 하나의 요청으로 컴포넌트가 렌더링되고, 또 그 컴포넌트의 렌더링 결과로 또 다른 컴포넌트를 렌더링하는 시나리오를 상상해보자. 이 상황에서 최초 컴포넌트의 요청과 렌더링이 끝나기 전까지 하위 컴포넌트의 요청과 렌더링이 끝나지 않는다. 이러한 작업을 서버에서 수행한다면 서버에서 데이터를 불러오고 렌더링이 이뤄지므로 발생하는 지연을 줄이고 클라이언트에서는 반복적으로 요청을 수행할 필요도 없어진다. 서버에서는 필요에 따라 백엔드 데이터에 접근하거나 지속적으로 데이터를 불러와 클라이언트에서보다 효율적으로 컴포넌트를 렌더링 할 수 있게 된다.

- 추상화 비용 증가:
    
    ```jsx
    // Note.js
    import db from 'db'
    import NoteWithMarkdown from 'NoteWithMarkdown.js'
    import { fetchNote } from 'services.js'
    
    function Note({ id }) {
      const [note, setNote] = useState(null);
      
      useEffect(() => {
        ;(async () => {
          const result = await fetchNote();
          const response = await result.json();
        })()
        
        if (note === null) {
          return null;
        }
      },[]);
      
      return <NoteWithMarkdown note={note}/>
    }
    
    // NoteWithMarkdown.js
    import marked from 'marked'
    import sanitizeHtml from 'sanitize-html'
    
    function NoteWithMarkdown({ note }) {
      const html = sanitizeHtml(marked(note.text));
      return <div dangerouslySetInnerHTML={{ __html: html }} />
    }
    ```
    
    이 코드는 클라이언트에서 어떤 정보를 불러오고 화면에 렌더링하는 컴포넌트이다. 이 코드는 Import, useEffect 실행, API 호출 등 내용은 복잡하지만 결국 사용자가 보는 것은 단순한 결과물일 것이다.
    
    ```html
    <div>
      <!-- 노트 -->
    </div>
    ```
    
    이러한 작업을 서버에서 미리 다 계산해서 내려준다면 클라이언트에서 복잡한 작업을 하지 않아도 되므로 속도가 빨라질 것이고, 클라이언트에서 전송되는 결과물 또한 단순해 가벼워질 것이다. 코드 추상화에 따른 비용은 서버에서만 지불하면 된다.

### 11.2.2 서버 컴포넌트란?

서버 컴포넌트는 서버와 클라이언트 모두에서 컴포넌트를 렌더링 할 수 있는 기법을 의미한다. 일부는 서버에서 렌더링되어 서버에서 할 수 있는 일은 서버가 처리하게 두고, 일부는 클라이언트에서 렌더링되어 서버가 할 수 없는 작업을 클라이언트인 브라우저에서 수행된다. 여기서 명심해야 하는 사실은 클라이언트 컴포넌트는 서버 컴포넌트를 import할 수 없다. 클라이언트 컴포넌트에서는 서버 컴포넌트를 실행할 방법이 없기 때문이다. 

**리액트의 서버 컴포넌트 관리 방법**

![스크린샷 2024-09-05 오후 8 04 55](https://github.com/user-attachments/assets/6c42f9fa-f44c-4563-aaf8-129d9b41e8fb)

모든 컴포넌트는 서버 컴포넌트가 될 수도 있고 클라이언트 컴포넌트가 될 수 있으므로 위와 같이 클라이언트 및 서버 컴포넌트가 혼재된 상황은 자연스럽다. 어떻게 이런 구조가 가능한 것일까? 그 비밀은 흔히 children으로 자주 사용되는 ReactNode에 달려 있다. 

```jsx
// ClientComponent.jsx
'use client'
// import ServerComponent from './ServerComponent.jsx' ❌ 클라이언트 컴포넌트에서 서버 컴포넌트를 불러올 수 없다.
export default function ClientComponent({ children }){
  return (
    <div>
      <h1>클라이언트 컴포넌트</h1>
      {children}
    </div>
  )
}

// ServerComponent
export default function ServerComponent(){
  return <span>서버 컴포넌트</span>
}

// ParentServerComponet.jsx
// 이 컴포넌트는 서버 컴포넌트일 수도, 클라이언트 컴포넌트일 수도 있다.
// 따라서 두 군데 모두에서 사용할 수 있다.
import ClientComponent from './ClientComponent'
import SercerComponent from './ServerComponent'
export default function ParentServerComponent(){
  return (
    <ClientComponent>
      <SercerComponent/>
    </ClientComponent>
  )
}
```

이 코드는 리액트 컴포넌트 트리를 설계할 때 어떠한 제약이 생기는지를 나타낸다. 서버 컴포넌트, 클라이언트 컴포넌트, 공용 컴포넌트의 차이와 제약사항에 대해 각각 알아보자.

- 서버 컴포넌트
    - 요청이 오면 그 순간 서버에서 딱 한 번 실행되므로 상태를 가질 수 없어 useState, useReducer 등의 훅은 사용할 수 없다.
    - 렌더링 생명주기도 사용할 수 없다. 한 번 렌더링되면 그걸로 끝이므로 useEffect, useLayoutEffect를 사용할 수 없다.
    - effect나 state에 의존하는 사용자 정의 훅 또한 사용할 수 없다. 다만 서버에서 제공할 수 있는 기능만 사용하는 훅은 사용 가능하다.
    - 서버에서 실행되기 때문에 DOM API, window, document 등에 접근할 수 없다.
    - 데이터베이스, 내부 서비스, 파일 시스템 등 서버에만 있는 데이터를 async/await로 접근할 수 있다. 컴포넌트 자체가 async한 것이 가능하다.
    - 서버 컴포넌트, div, span, p 같은 요소, 클라이언트 컴포넌트를 렌더링할 수 있다.
- 클라이언트 컴포넌트
    - 브라우저 환경에서만 실행되므로 서버 컴포넌트를 import 하거나 서버 전용 훅이나 유틸리티를 불러올 수 없다.
    - 클라이언트 컴포넌트가 자식으로 서버 컴포넌트를 가질 수 있다. 그 이유는 클라이언트 컴포넌트는 이미 서버에서 만들어진 트리를 삽입해서 보여주면 되기 때문이다.
    - 이 두 가지 예외 사항을 제외하면 일반적으로 우리가 알고 있는 리액트 컴포넌트와 같다. state와 effect를 사용할 수 있고 브라우저 API도 사용할 수 있다.
- 공용 컴포넌트(shared components)
    - 이 컴포넌트는 서버와 클라이언트 모두에서 사용할 수 있다. 공통으로 사용할 수 있는 만큼 서버 컴포넌트와 클라이언트 컴포넌트의 모든 제약을 받는 컴포넌트가 된다.
 
리액트가 이 세 가지 컴포넌트를 판단하는 방법을 알아보자. 리액트는 컴포넌트를 서버에서 실행 가능한 공용 컴포넌트로 판단한다. 대신, 클라이언트 컴포넌트라는 것을 명시적으로 선언하려면 파일 맨 첫 줄에 ‘use client’라고 작성하면 된다. 이때, 클라이언트 컴포넌트가 서버 컴포넌트를 import 하는 상황에서는 빌드 에러가 발생한다. 

리액트 서버 컴포넌트를 사용하려면 여러 가지 제약 요소로 인해 번들러나 특정 프레임워크의 도움을 받는 것이 필수적이다. 리액트 팀에서 제공하는 공식 예제에서는 웹팩과 react-server-dom-webpack을 만들어 활용했고, 서버 컴포넌트 제안 문서에서는 Next.js 팀과 협업하고 있음을 언급했으며, 애플리케이션의 라우팅 시스템과 번들러 사이의 통합이 필요하다는 것을 강조했고 Next.js와 같은 한 개 이상의 프레임워크 팀과 리액트 서버 컴포넌트의 초기 구축을 위해 노력하고 있음을 언급했다. 

### 11.2.3 서버 사이드 렌더링과 서버 컴포넌트의 차이

서버 사이드 렌더링은 응답받은 페이지 전체를 HTML로 렌더링하는 과정을 서버에서 수행한 후 그 결과를 클라이언트에 내려준다. 그리고 이후 클라이언트에서 하이드레이션 과정을 거쳐 서버의 결과물을 확인하고 이벤트를 붙이는 등의 작업을 수행한다. 서버 사이드 렌더링의 목적은 정적인 HTML을 빠르게 내려주는 데 초점을 두고 있다. 그러므로 인터렉션을 위해 클라이언트에서 자바스크립트를 코드를 다운로드하고, 파싱하고, 실행하는 데 비용이 든다. 

서버 컴포넌트를 활용해 서버에서 렌더링할 수 있는 컴포넌트는 서버에서 완성해 제공받은 다음, 클라이언트 컴포넌트는 서버 사이드 렌더링으로 초기 HTML을 빠르게 전달받을 수 있다. 이 두 가지 방법을 모두 결합하면 클라이언트 및 서버 컴포넌트를 모두 빠르게 보여줄 수 있고, 동시에 클라이언트에서 내려받아야 하는 자바스크립트 양도 줄어 브라우저의 부담을 덜 수도 있다.

결론적으로 둘은 대체제가 아닌 상호보완하는 개념으로 봐야 할 것이다. 리액트 팀에서도 미래에는 두 가지 기법이 모두 쓰일 수 있는 가능성을 암시하고 있다. 아직 리액트 서버 컴포넌트 자체도 베타이기 때문에 향후 어떤 방식으로 제공될지 기대해 봐도 즣을 것이다.

### 11.2.4 서버 컴포넌트는 어떻게 작동하는가?

리액트 서버 컴포넌트를 렌더링하기 위해 어떠한 일들이 일어나는지 핵심 내용만 간단하게 살펴보자. 여기서 사용한 예제 코드는 리액트 팀이 2021년에 공식적으로 제공한 server-components-demo를 포크한 prisma/server-components-demo다. 이 포크를 사용한 이유는 기존 리액트 팀의 예제 애플리케이션은 PostgressSQL을 이용해 구현돼 있기 때문에 별도로 데이터베이스를 설치하고 설정해야 하는 번거로움이 있기 때문이다. 

> 이 저장소는 리액트 서버 컴포넌트 v1, 즉 파일명을 규칙으로 하던 때를 기준으로 작성돼 있다. 현재 버전에서는 ‘use client’로 클라이언트 컴포넌트를 선언한다.

```jsx
// server/api.server.js

app.get(
  '/',
  handleErrors(async function(_req, res) {
    await waitForWebpack();
    const html = readFileSync(
      path.resolve(__dirname, '../build/index.html'),
      'utf8'
    );
    // Note: this is sending an empty HTML shell, like a client-side-only app.
    // However, the intended solution (which isn't built out yet) is to read
    // from the Server endpoint and turn its response into an HTML stream.
    res.send(html);
  })
);
```

사용자가 최초에 들어왔을 때 수행하는 작업은 오로지 index.html을 제공하는 것 뿐이다. 

1. 서버가 렌더링 요청을 받는다. 서버가 렌더링 과정을 수행해야 하므로 리액트 서버 컴포넌트를 사용하는 모든 페이지는 항상 서버에서 시작된다. 즉, 루트에 있는 컴포넌트는 항상 서버 컴포넌트이다. 예제의 구조는 현재 다음과 같다. 이 예제에서 /react 주소로 요청을 보내면 서버는 서버 렌더링을 시작한다.
![스크린샷 2024-09-05 오후 8 07 02](https://github.com/user-attachments/assets/e654761b-5b3d-47a0-9cf0-f43bf091e9f6)
이 예제에서 /react 주소로 요청을 보내면 서버는 서버 렌더링을 시작한다. 

2. 서버는 받은 요청에 따라 컴포넌트를 JSON으로 직렬화한다. 이때 서버에서 렌더링할 수 있는 것은 직렬화해서 내보내고, 클라이언트 컴포넌트로 표시된 부분은 해당 공간을 placeholder 형식으로 비워두고 나타낸다. 브라우저는 이후에 이 결과물을 받아서 다시 역직렬화한 다음 렌더링을 수행한다.
```text
// /react로 최초 메인 페이지에 대한 렌더링 요청을 했을 때 받는 응답
M1: {"id":"./src/SearchField.client.js","chunks": ["client5"], "name":""}
M2: {"id":"./src/EditButton.client.js","chunks": ["client1"], "name":'"}
S3: "react.suspense"
JO: ["$", "div", null, {"className": "main", "children": [["$", "section", null, {"className": "col sidebar", "children": [L"$", "section", null, {"className": "sidebar-header", "chil-dren": [L"$", "img", null, {"className": "logo", "src": "logo.svg", "width": "22px", "height": "20px-
", "alt":"", "role": "presentation"}], ["$", "strong", null, {"children": "React Notes"}]]}], ["$","-section", null, {"className": "sidebar-menu", "role", "menubar", "children": [["$", "@1", null, {}, ["$", "@2", null, {"noteId":null, "children": "New"}]]}], ["$", "nav", null, {"children": ["$", "$3", null, {"-
fallback": ["$", "div", null, {"children": ["$", "ul", null, {"className": "notes-list skeleton-contain-
er",”children": [["$","li", null, ... (이하 생략)
```
이 같은 데이터 형태를 와이어 포맷이라 하며, 서버는 이 값을 스트리밍해 클라이언트에 제공한다. 하나씩 그 형태를 살펴보자.

- M: M으로 시작하는 줄은 클라이언트 컴포넌트를 의미한다. 클라이언트 번들에서 해당 함수를 렌더링하기 위해 필요한 정보가 어디(chunk)에 담겨 있는지 참조를 전달한다. 이 예제에서는 SearchFiled.client.js, EditButton.client.js, SideBarNote.client.js가 클라이언트의 모듈의 참조로 전해졌다.
- S: Suspense를 의미한다.
- J: 서버에서 렌더링된 서버 컴포넌트다. J0은 App.server.js를 표현한 것이다. 여기에는 렌더링에 필요한 모든 element, className, props, children 정보 등이 들어있다. 여기서 한 가지 흥미로운 것은 @1, @2와 같은 요소이다.
    
    ```jsx
    [
      "$", 
      "section" 
      null,
      {
        "className": "sidebar-menu".
        "role": "menubar",
        "children": [
          ["$", "@1", null, {}],
          [
            "$",
            "@2", 
            null,
            {
              "noteId": null,
              "children": "New"
            }
          ]
        ]
      }
    ]
    ```
    
    이 정보는 나중에 렌더링이 완료됐을 때 들어가야 할 컴포넌트를 의미한다. @1은 M1이 렌더링되면 저 @1자리에 @M1이 들어가야하는 것을 의미한다. 이처럼 서버에서는 클라이언트에서 리액트 컴포넌트 트리 구성에 필요한 정보를 최대한 많이, 그리고 경제적인 포맷으로 클라이언트에 전달한다.

3. 브라우저가 리액트 컴포넌트 트리를 구성한다. 브라우저가 서버로 스트리밍으로 JSON 결과물을 받았다면 이 구문을 다시 파싱한 결과물을 바탕으로 트리를 재구성해 컴포넌트를 만들어 나간다. M1과 같은 형태의 클라이언트 컴포넌트를 받았다면 클라이언트에서 렌더링을 진행할 것이고, 서버에서 만들어진 결과물을 받았다면 이 정보를 기반으로 리액트 트리를 그대로 만들 것이다. 그리고 최종적으로 이 트리를 렌더링해 브라우저의 DOM에 커밋한다. 

지금까지 살펴본 바를 토대로 리액트 서버 컴포넌트의 작동 방식의 특별한 점을 살펴보자.

- 서버는 클라이언트로 정보를 보낼 때 스트리밍 형태로 보냄으로써 클라이언트가 줄 단위로 JSON을 읽고 컴포넌트를 렌더링 할 수 있어 브라우저에서는 되도록 빨리 사용자에게 결과물을 보여줄 수 있다.
- 컴포넌트들은 하나의 번들러 작업에 포함돼 있지 않고 각 컴포넌트별로 번들링이 별개로 돼 있어 필요에 따라 컴포넌트를 지연해서 받거나 따로 받는 등의 작업이 가능해졌다.
- 서버 컴포넌트는 결과물이 HTML이 아닌 JSON 형태로 보내진다. 클라이언트의 최종 목표는 리액트 컴포넌트 트리를 서버 컴포넌트와 클라이언트 컴포넌트의 두 가지로 조화롭게 구성하는 것이다. 따라서 단순한 JSON으로 받아 리액트 컴포넌트 트리의 구성을 최대한 빠르게 할 수 있도록 한다.

> 이러한 특징으로 인해 생기는 제약 사항은 서버 컴포넌트에서 클라이언트 컴포넌트로 props를 넘길 때 반드시 직렬화 가능한 데이터를 넘겨야 한다. 서버에서 클라이언트로 데이터를 보내주는 것은 JSON을 통해 이뤄지기 때문에 class나 Date 등은 불가능하다.
> 

결론적으로 리액트 서버 컴포넌트는 완전히 새로운 개념이며, 기존의 리액트 컴포넌트가 가진 한계를 극복하기 위해 만들어졌다. 앞으로 서버 사이드 렌더링과 리액트 서버 컴포넌트가 함께 어울리면서 완전히 새로운 구조를 엿볼 수 있게 될 것이다.

## 11.3 Next.js에서의 리액트 서버 컴포넌트

Next.js에서도 기본적인 서버 컴포넌트의 제약은 동일하다. 서버 컴포넌트는 클라이언트 컴포넌트를 불러올 수 없으며, 클라이언트 컴포넌트는 서버 컴포넌트를 children props로 받는 것만 가능하다. 그리고 앞서 루트 컴포넌트는 무조건 서버 컴포넌트가 된다고 언급했는데, Next.js의 루트 컴포넌트는 각 페이지에 존재하는 page.js이다. 그리고 layout.js도 서버 컴포넌트로 작동한다. 

### 11.3.1 새로운 fetch 도입과 getServerSidePros, getStaticProps, getInitialProps의 삭제

과거 Next.js의 서버 사이드 렌더링과 정적 페이지 제공을 위해 사용한 getServerSideProps, getStaticProps, getInitialProps가 /app 디렉터리 내부에서는 삭제됐다. 그 대신 모든 데이터 요청은 웹에서 제공하는 표준 API인 fetch를 기반으로 이뤄진다.

```jsx
async function getDate() {
  // 데이터를 불러온다.
  const result = await fetch('example.com')
  
  if (!result.ok) {
    // 이렇게 에러를 던지면 가장 가까운 에러 바운더리에 전달된다. 
    throw new Error('데이터 불러오기 실패')
  }
  
  return result.json()
}

// async 서버 컴포넌트 페이지
export default async function Page() {
  const data = await getData()
  
  return (
    <main>
      <Children data={data} />
    </main>
  )
}
```

서버에서 데이터를 직접 불러올 수 있게 됐다. 또한 컴포넌트가 비동기적으로 작동하는 것이 가능해 서버 컴포넌트는 데이터가 불러와지면 비로소 페이지가 렌더링되어 클라이언트로 전달될 것이다.

> ~~2023년 5월 기준으로 아직 타입스크립트가 이러한 비동기 컴포넌트를 정식으로 지원하지 않다.~~ 타입스크립트 5.1.3부터 지원한다.
> 

추가로 리액트팀은 이 fetch API를 확장해 같은 서버 컴포넌트 트리 내에서 동일한 요청이 있다면 재요청이 발생하지 않도록 요청 중복을 방지했다.

![스크린샷 2024-09-05 오후 8 09 47](https://github.com/user-attachments/assets/68213a6c-7dfb-4987-862f-f8f4e84ef12e)

요즘 인기를 끌고 있는 SWR과 React Query와 비슷하게, 해당 fetch 요청에 대한 내용을 서버에서는 렌더링이 한 번 끝날 때까지 캐싱하며, 클라이언트에서는 별도의 지시자나 요청이 없는 이상 해당 데이터를 최대한 캐싱해 중복된 요청을 방지한다. 

### 11.3.2 정적 렌더링과 동적 렌더링

Next.js 13에서는 이제 정적인 라우팅에 대해서는 기본적으로 빌드 타임에 렌더링을 미리 해두고 캐싱해 재사용할 수 있게끔 해뒀고, 동적인 라우팅에 대해서는 서버에 매번 요청이 올 때마다 컴포넌트를 렌더링하도록 변경했다. 다음 예제를 보자.

```jsx
// app/page.tsx
async function fetchData() {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts`)
  const data = await res.json()
  return data;
}

export default async function Page() {
  const data: Array<any> = await fetchData()
  
  return (
    <ul>
      {data.map((item, key) => (
        <li key={key}>{item.id}</li>
      ))}
    </ul>
  )
}
```

이 예제는 특정 API 엔드 포인트에서 데이터를 불러와 페이지에서 렌더링하는 구조를 가진 서버 컴포넌트이다. 이 주소는 정적으로 결정돼 있기 때문에 빌드 시에 해당 주소로 미리 요청을 해서 데이터를 가져온 다음 렌더링한 결과를 빌드에 넣어둔다.

![스크린샷 2024-09-05 오후 8 10 23](https://github.com/user-attachments/assets/c3541904-7c74-4abf-af94-de0b7f710fbb)

반면 해당 주소를 정적으로 캐싱하지 않는 방법도 있다. 

```jsx
// app/page.tsx
async function fetchData() {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts`,
    // no-cache 옵션을 추가했다.
    { cache: 'no-cache' },
    // Next.js에서 제공하는 옵션을 사용해도 동일하다.
    // { next: { revalidate: 0 }}
  )
  const data = await res.json()
  return data;
}

export default async function Page() {
  const data: Array<any> = await fetchData()
  
  return (
    <ul>
      {data.map((item, key) => (
        <li key={key}>{item.id}</li>
      ))}
    </ul>
  )
}
```

이렇게 옵션을 설정하면 Next.js는 요청이 올 때마다 fetch 요청 이후에 렌더링을 수행하게 된다. 

이 밖에도 함수 내부에서 Next.js가 제공하는 next/headers나 next/cookie 같은 헤더 정보와 쿠키 정보를 불러오는 함수를 사용하면 동적인 연산을 바탕으로 결과를 반환하는 것으로 인식해 정적 렌더링 대상에서 제외된다.

만약 동적인 주소이지만 특정 주소에 대해서 캐싱하고 싶은 경우 generateStaticParams를 사용하면 된다.

```jsx
export function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }]
}

async function fetchData(id: string) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`)
  const data = await res.json()
  return data;
}

export default async function Page({
  params,
}: {
  params: { id: string }
  children?: React.ReactNode
}) {
  const data = await fetchData()
  
  return (
    <div>
      <h1>{data.title}</h1>
    </div>
  )
}
```
![스크린샷 2024-09-05 오후 8 10 52](https://github.com/user-attachments/assets/13cf05f0-8735-4e2e-95aa-e74d38211b75)

fetch 옵션에 따른 작동 방식을 정리하면 다음과 같다.

- `fetch(URL, { cache: 'force-cache' })`: 기본값으로 getStaticProps와 유사하게 불러온 데이터를 캐싱해 해당 데이터로만 관리한다.
- `fetch(URL, { cache: 'no-store' })` , `fetch(URL, { next: {revalidate: 0} })`: getServerSideProps와 유사하게 캐싱하지 않고 매번 새로운 데이터를 불러온다.
- `fetch(URL, { next: {revalidate: 10} })`: getStaticProps에 revalidate를 추가한 것과 동일하며, 정해진 유효시간 동안에는 캐싱하고, 이 유효 시간이 지나면 캐시를 파기한다.

### 11.3.3 캐시와 mutating, 그리고 revalidating

Next.js는 fetch의 기본 동작을 재정의해 `fetch(URL, { next: {revalidate?: number | false} })` 를 제공하는데, 이를 바탕으로 데이터의 유효 시간을 정할 수 있고 이 시간이 지나면 다시 데이터를 불러와 페이지를 렌더링하는 것이 가능하다. 캐시와 갱신이 이뤄지는 과정은 다음과 같다. 

1. 최초로 해당 라우트로 요청이 올 때는 미리 정적으로 캐시해 둔 데이터를 보여준다.
2. 이 캐시된 초기 요청은 revalidate에 선언된 값(sec)만큼 유지된다. 
3. 만약 해당 시간이 지나도 일단은 캐시된 데이터를 보여준다.
4. Next.js는 캐시된 데이터를 보여주는 한편, 시간이 경과했으므로 백그라운드에서 다시 데이터를 불러온다.
5. 4번의 작업이 성공적으로 끝나면 캐시된 데이터를 갱신하고, 그렇지 않으면 과거의 데이터를 보여준다.

만약 이러한 캐시를 전체적으로 무효화하고 싶다면 router에 추가된 refresh 메서드로 router.refresh()를 사용하면 된다. 이는 브라우저의 히스토리에 영향을 미치지 않고, 오로지 서버에서 루트부터 데이터를 전체적으로 가져와서 갱신하게 된다. 그리고 이 작업은 브라우저나 리액트의 state에 영향을 미치지 않는다.

### 11.3.4 스트리밍을 활용한 점진적인 페이지 불러오기

리액트에는 HTML을 작은 단위로 쪼개 완성되는 대로 클라이언트로 점진적으로 보내는 스트리밍이 도입됐다. 스트리밍을 활용하면 먼저 데이터가 로드되는 컴포넌트를 빠르게 보여주는 방법이 가능하다. 이는 사용자가 일부라도 페이지와 인터렉션을 할 수 있다는 것을 의미하며, 나아가 TTFB와 FCP를 개선하는 데 큰 도움을 준다. 

이 스트리밍을 활용할 수 있는 방법은 두 가지가 있다. 

- 경로에 loading.tsx 배치: loading은 앞서 잠깐 소개한 것처럼 예약어로 존재하는 컴포넌트로, 렌더링이 완료되기 전에 보여줄 수 있는 컴포넌트를 배치할 수 있는 파일이다. loading 파일을 배치한다면 자동으로 다음 구조와 같이 Suspense가 배치된다.
    
    ```jsx
    <Suspense fallback={<Loading />}>
      <Page />
    </Suspense>
    ```
- Suspense 배치: 좀 더 세분화된 제어를 하려면 직접 리액트의 Suspense를 배치한다.
    
    ```jsx
    import { Suspense } from 'react'
    import { Notes, Peoples } from './Components'
    
    export default function Posts() {
      return (
        <section>
          <Suspense fallback={<Skeleton/>}>
            <Notes/>
          </Suspense>
          <Suspense fallback={<Skeleton/>}>
            <Peoples/>
          </Suspense>
        </section>
      )
    }
    ```
    

Loading이 Suspense를 기반으로 만들어진 Next.js 규칙이기 때문에 Suspense를 사용하는 것도 동일한 혜택을 누릴 수 있다. 스트리밍을 활용해 서버 렌더링이 가능해지고, 리액트는 로딩이 끝난 컴포넌트 순서대로 하이드레이션을 수행해 가능한 한 사용자에게 빠르게 상호작용이 가능한 페이지를 제공할 수 있게 된다.

## 11.4 웹팩의 대항마, 터보팩의 등장(beta)

이번 Next.js 13에서는 웹팩의 후계자를 자처하고 있는 터보팩이 출시됐다. 터보팩은 웹팩 대비 최대 700배, Vite 대비 최대 10배 빠르다고 하며, 러스트 기반으로 작성됐기 때문에 가능하다고 소개하고 있다.

다만 Next.js 13.1.x를 기준으로 베타이며(현재도 베타임), 현재는 개발 모드에서만 제한적으로 사용가능하기 때문에 실제 프로덕션 모드에까지 사용할 수 있기까지는 어느 정도 시간이 걸릴 것으로 보인다. 

## 11.5 서버 액션(alpha)

Next.js 13.4.0이 릴리스되면서 Next.js 팀은 서버 액션이라고 하는 새로운 기능을 선보였다. 이 기능은 서버에 직접 접근해 데이터 요청 등을 수행할 수 있다. 서버 컴포넌트와 다르게, 특정 함수 실행 그 자체만을 서버에서 수행할 수 있다는 장점이 있다. 그리고 그 실행 결과에 따라 다양한 작업을 수행할 수 있다. 이 서버 액션을 활성화하려면 next.config.js에서 실험 기능을 활성화해야 한다.

```jsx
const nextConfig = {
  experimental: {
    serverActions: true
  },
}
```

서버 액션을 만들려면 먼저 함수 내부 또는 파일 상단에 ‘use server’ 지시자를 선언해야 한다. 그리고 함수는 반드시 async여야 한다. 

```jsx
async function serverAction() {
  'use server';
  // 서버에 바로 접근하는 코드
}

// 이 파일 내부의 모든 내용이 서버 액션으로 간주된다. 
'use server'; 
export async function myAction() {
  // ...
  // 서버에 바로 접근하는 코드
}
```

> Next.js 13.4.0 기준으로 "next dev --turbo”를 실행하면 서버 액션을 수행할 수 없다. 이는 아직 터보팩이 서버 액션까지 완벽하게 지원하지 않기 떄문으로 보인다.

### 11.5.1 form의 action

`<form/>`의 action props를 추가해 이 양식 데이터를 처리할 URI를 넘겨 줄 수 있다. 서버 액션으로 form.action 함수를 만들어보자.

```jsx
export default function Page() {
  async function handleSubmit() {
    'use server'

    console.log('해당 작업은 서버에서 수행합니다. 따라서 CORS 이슈가 없습니다.')

    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
      method: 'post',
      body: JSON.stringify({
        title: 'foo',
        body: 'bar',
        userId: 1,
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8',
      },
    })

    const result = await response.json()
    console.log(result)
  }
  return (
    <div className="space-y-4">
      <h1 className="text-xl font-medium text-gray-400/80">form</h1>

      <div className="space-y-4">
        <ul className="list-disc space-y-2 pl-4 text-sm text-gray-300">
          <li>아래 버튼을 누르면 서버에서 직접 form 요청을 보냅니다.</li>
          <li>
            <form action={handleSubmit}>
              <button type="submit">form 요청 보내보기</button>
            </form>
          </li>
        </ul>
      </div>
    </div>
  )
}
```

이 에제는 form.action에 handleSubmit이라는 서버 액션을 만들어 props로 넘겨주었다. 이 handleSubmit 이벤트를 발생시키는 것은 클라이언트지만 실제로 함수 자체가 수행되는 것은 서버가 된다.

![스크린샷 2024-09-05 오후 8 12 37](https://github.com/user-attachments/assets/99221145-fc23-4bf3-9f02-5d6453bda448)

form 버튼을 클릭했을 때 네트워크 탭을 확인하면 /server-action/form으로 요청이 수행되고, 페이로드에는 앞서 코드에서 보낸 post요청이 아닌 ACTION_ID라는 액션 구분자만 있는 것을 볼 수 있다. 그리고 이를 처리하는 서버에서는 다음과 같은 내용이 미리 빌드돼 있는 것을 볼 수 있다. 

```jsx
// 해당 페이지에서 수행하는 서버 액션을 모아둔다.
const actions = {
'6720cf7ee4d6be4a09be858d53b2700ee1116230': () => Promise.resolve(/* import() eager */).then(__webpack_require__.bind(__webpack_require__, 5948)).then(mod => mod["$$ACTION_0"]),
}

// ...

// 해당 페이지
function Page() {
    async function handleSubmit() {
        return $$ACTION_0(handleSubmit.$$bound);
    }
    // ...
}

// ...
async function $$ACTION_0(closure) {
    console.log("해당 작업은 서버에서 수행합니다. 따라서 CORS 이슈가 없습니다.");
    const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "post",
        body: JSON.stringify({
            title: "foo",
            body: "bar",
            userId: 1
        }),
        headers: {
            "Content-type": "application/json; charset=UTF-8"
        }
    });
    const result = await response.json();
    console.log(result);
}

```

위 코드와 실행 결괄르 미루어 봤을 때, 서버 액션을 실행하면 클라이언트에서는 현재 라우트 주소와 ACTION_ID만 보내고 그 외에는 아무것도 실행하지 않는 것을 알 수 있다. 그리고 서버에서는 요청받은 라우트 주소와 ACTION_ID를 바탕으로, 실행해야 할 내용을 찾고 서버에서 직접 실행한다. 이를 위해 ‘use server’로 선언돼 있는 내용을 빌드 시점에 미리 클라이언트에서 분리해 클라이언트 번들링 결과물에는 포함되지 않고 서버에서만 실행되는 서버 액션을 만든 것을 확인할 수 있다. 

서버 액션의 또다른 장점은 폼과 실제 노출하는 데이터가 연동돼 있을 때 더욱 효과적으로 사용할 수 있다.

```jsx
// key value sotrage. 서버에서만 사용할 수 있는 패키지다.
import kv from '@vercel/kv'
import { revalidatePath } from 'next/cache'

interface Data {
  name: string
  age: number
}

export default async function Page({ params }: { params: { id: string } }) {
  const key = `test:${params.id}`
  const data = await kv.get<Data>(key)

  async function handleSubmit(formData: FormData) {
    'use server'

    const name = formData.get('name')
    const age = formData.get('age')

    await kv.set(key, {
      name,
      age,
    })
    
    // 페이지의 캐시를 재검증하여 페이지의 변경된 부분만 새로고침 없이 업데이트한다.
    revalidatePath(`/server-action/form/${params.id}`) 
  }

  return (
    <div className="space-y-4">
      <h1 className="text-xl font-medium text-gray-400/80">form with data</h1>
      <h2 className="text-l font-medium text-gray-400/80">
        서버에 저장된 정보: {data?.name} {data?.age}
      </h2>

      <div className="space-y-4">
        <ul className="list-disc space-y-2 pl-4 text-sm text-gray-300">
          <li>아래 버튼을 누르면 서버에서 직접 form 요청을 보냅니다.</li>
          <form action={handleSubmit}>
            <li>
              <label htmlFor="name">이름: </label>
              <input
                type="text"
                id="name"
                name="name"
                defaultValue={data?.name}
                placeholder="이름을 입력해주세요."
              />
            </li>

            <li>
              <label htmlFor="age">나이: </label>
              <input
                type="number"
                id="age"
                name="age"
                defaultValue={data?.age}
                placeholder="나이를 입력해주세요."
              />
            </li>

            <li>
              <button type="submit">submit</button>
            </li>
          </form>
        </ul>
      </div>
    </div>
  )
}
```

@vercel/kv는 서버에서만 접근할 수 있는 Redis 스토리지로 이를 기반으로 서버 액션에서 어떻게 양식 데이터를 다룰 수 있는지 나타낸다. 

Page 컴포넌트는 서버 컴포넌트로, `const data = await kv.get<Data>(key)`에서 직접 서버 요청을 수행해 데이터를 가져와 JSX를 렌더링한다.

그리고 form 태그에 서버 액션인 handleSubmit을 추가해 formData를 기반으로 데이터를 가져와 다시 데이터베이스인 kv에 업데이트한다. 

이 업데이트가 성공적으로 마무리됐다면 마지막으로 revalidatePath를 통해 해당 주소의 캐시 데이터를 갱신해 컴포넌트를 재렌더링하게 했다.
![스크린샷 2024-09-05 오후 8 13 54](https://github.com/user-attachments/assets/13088eda-e683-4156-adac-eb37acf5636b)

서버에 ACTION_ID와 실행에 필요한 데이터만 보내고, 직접적인 업데이트는 수행하지 않는다. 그리고 이 서버 액션의 실행이 완료되면 data 객체가 revalidatePath로 갱신되어 업데이트된 최신 데이터로 불러온다. 이러한 최신 데이터를 불러오는 동작은 페이지 내부에 loading.jsx가 있다면 더욱 뚜렷하게 확인할 수 있다. 

지금까지 살펴본 내용을 봤을 때는 PHP 같은 전통적인 서버 기반 웹 애플리케이션과 크게 다를 바 없어 보인다. 하지만 주목해야할 가장 큰 차이는 이 모든 과정이 페이지 새로고침 없이 수행된다는 것이다. 따라서 서버에 데이터 수정을 요청하는 한편, 클라이언트에서는 업데이트를 완료한 후 새로운 결과를 받을 때까지 사용자에게 로딩 중이라는 것을 알 수 있는 인터렉션을 구성할 수도 있다. 

한 가지 더 주목해야 할 것은 handleSubmit에서 수행한 revalidatePath다. 이는 인수로 넘겨받은 경로의 캐시를 초기화해서 해당 URL에서 즉시 새로운 데이터를 불러온다. Next.js에서는 server mutation이라고 한다. server mutation으로 실행할 수 있는 함수는 다음과 같다. 

- redirect: `import { redirect } from ‘next/navigation’` 로 사용할 수 있으며, 특정 주소로 리다이렉트 할 수 있다.
- revalidatePath: `import { revalidate } from ‘next/cache’` 로 사용할 수 있으며, 해당 주소의 캐시를 즉시 업데이트한다.
- revalidateTag: `import { revalidateTag } from ‘next/cache’` 로 캐시 태그는 fetch 요청 시에 다음과 같이 추가할 수 있다. `fetch('https://localhost:8080/api/something', { next: { tags: [''] } })`  이렇게 태그를 추가하면 다양한 fetch 요청을 특정 태그 값으로 구분할 수 있으며, 이 특정 태그가 추가된 fetch 요청을 모두 초기화한다.
    
    ```jsx
    // page.jsx
    const res = await fetch('https://baseurl.com', { next: { tags: ['mytag'] } });
    
    // form.jsx
    'use server'
    
    import { revalidateTag } from 'next/cache'
    
    export default async function action() {
      revalidateTag('mytag')
    }
    ```
    

이처럼 form을 서버 액션과 함께 사용하면 form을 기반으로 한 데이터 추가 및 수정 요청을 좀 더 자연스럽게 수행할 수 있다. 그리고 Next.js에서 관리하는 캐시를 효과적으로 초기화할 수 있으므로 사용자에게 더욱 자연스러운 사용자 경험을 안겨줄 수 있다.

### 11.5.2 input의 submit과 image의 formAction

form.action과 마찬가지로 `input type='submit'` 또는 `input type='image'` 에 formAction prop으로도 서버액션을 추가할 수 있다. 사용법은 앞에서 살펴본 것과 동일하다.

### 11.5.3 startTransition과의 연동

startTransition을 사용해 서버 액션을 실행할 수 있다. 

> `useTransition`은 UI를 차단하지 않고 상태를 업데이트할 수 있는 React Hook입니다.

```jsx
// server-action/index.ts
'use server'

import kv from '@vercel/kv'
import { revalidatePath } from 'next/cache'
import { cookies } from 'next/headers'

export async function updateData(
  id: string,
  data: { name: string; age: number },
) {
  const key = `test:${id}`

  await kv.set(key, {
    name: data.name,
    age: data.age,
  })

  revalidatePath(`/server-action/form/${id}`)
}

// client-component.tsx
'use client'
import { useCallback, useTransition } from 'react'
import { updateData } from '#server-action'
import { SkeletonBtn } from '#components/components'

export function ClientButtonComponent({ id }: { id: string }) {
  const [isPending, startTransition] = useTransition()

  const handleClick = useCallback(() => {
    startTransition(() => updateData(id, { name: '기본값', age: 0 }))
  }, [])

  return isPending ? (
    <SkeletonBtn />
  ) : (
    <button onClick={handleClick}>기본값으로 돌리기</button>
  )
}
import kv from '@vercel/kv'
import { ClientButtonComponent } from '#components/server-action/client-component'

interface Data {
  name: string
  age: number
}
```

useTransition을 사용하면 얻을 수 있는 장점은 이전과 동일한 로직을 구현하면서도 page 단위의 loading.jsx를 사용하지 않아도 된다는 것이다. isPending을 활용해 startTransition으로 서버 액션이 실행됐을 때 해당 버튼을 숨기고 로딩 버튼을 노출함으로써 페이지 단위의 로딩이 아닌 컴포넌트 단위의 로딩 처리가 가능해진다. 이와 동시에 revalidatePath과 같은 server mutation도 마찬가지로 처리할 수 있다.

### 11.5.4 server mutation이 없는 작업

별도의 server mutation을 실행하지 않는다면 이벤트 핸들러에서 처리하면 된다. 

```jsx
export default function Page() {
  async function handleClick() {
    'use server'
    
    // server mutation이 필요 없는 작업
  }
  return <button onClick={handleClick}>form 요청 보내기</button>
}
```

### 11.5.5 서버 액션 사용 시 주의할 점

- 서버 액션은 클라이언트 컴포넌트 내에서 정의될 수 없다. 클라이언트 컴포넌트에서 서버 액션을 쓰고 싶을 때는 앞의 `startTransition` 예제처럼 `‘use server’`로 서버 액션만 모여 있는 파일을 별도로 import해야한다.
- 서버 액션은 props로 클라이언트 컴포넌트에 넘기는 것 또한 가능하다. 즉 서버에서만 실행될 수 있는 자원은 반드시 파일 단위로 분리해야 한다.

## 11.6 그 밖의 변화

- 프로젝트 전체 라우트에서 쓸 수 있는 미들웨어 강화
- SEO를 쉽게 작성할 수 있는 기능 추가
- 정적으로 내부 링크를 분석할 수 있는 기능
  
자세한 내용은 vercel의 릴리스 가이드를 참고하자.

## 11.7 Next.js 13 코드 맛보기

### 11.7.1 getServerSideProps와 비슷한 서버 사이드 렌더링 구현해보기

서버 컴포넌트에서 fetch를 수행하고, 이 fetch에 별다른 cache 옵션을 제공하지 않는다면 기존의 getServerSideProps와 매우 유사하게 작동한다.

```jsx
import { fetchPostById } from '#services/server'

export default async function Page({ params }: { params: { id: string } }) {
  const data = await fetchPostById(params.id, { cache: 'no-cache' })

  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-medium text-gray-100">{data.title}</h1>
      <p className="font-medium text-gray-400">{data.body}</p>
    </div>
  )
}
```

```jsx
// 렌더링된 HTML 생략
```

최초 요청 시에 HTML을 살펴보면 기존에 getServerSideProps와 마찬가지로 미리 렌더링되어 완성된 HTML이 내려오는 것을 확인할 수 있다. 여기서 눈여겨볼 것은 뒤에 오는 script이다.

```jsx
<script>
    (self.__next_f = self.__next_f || []).push([0])
</script>
<script>
    self.__next_f.push([1, "1:HL[\"/_next/static/css/app/layout.css?v=1725535387904\",{\"as\":\"style\"}]\n0:\"$L2\"\n"])
</script>
<script>
    self.__next_f.push([1, "3:I{\"id\":\"(app-client)/./node_modules/next/dist/client/components/app-router.js\",\"chunks\":[\"webpack:static/chunks/webpack.js\"],\"name\":\"\",\"async\":false}\n5:I{\"id\":\"(app-client)/./node_modules/next/dist/client/components/error-boundary.js\",\"chunks\":[\"webpack:static/chunks/webpack.js\"],\"name\":\"\",\"async\":false}\n6:I{\"id\":\"(app-client)/./src/components/Sidebar.tsx\",\"chunks\":[\"app/layout:static/chunks/app/layout.js\"],\"name\":\"\",\"async\":false}\n7:I{\"id\":\"(app-client)/./node_modules/next/dist/client/components/layout-r"])
</script>
<script>
    self.__next_f.push([1, "outer.js\",\"chunks\":[\"app-client-internals:static/chunks/app-client-internals.js\"],\"name\":\"\",\"async\":false}\n8:I{\"id\":\"(app-client)/./node_modules/next/dist/client/components/render-from-template-context.js\",\"chunks\":[\"app-client-internals:static/chunks/app-client-internals.js\"],\"name\":\"\",\"async\":false}\n"])
</script>
<script>
    self.__next_f.push([1, "2:[[[\"$\",\"link\",\"0\",{\"rel\":\"stylesheet\",\"href\":\"/_next/static/css/app/layout.css?v=1725535387904\",\"precedence\":\"next_static/css/app/layout.css\"}]],[\"$\",\"$L3\",null,{\"assetPrefix\":\"\",\"initialCanonicalUrl\":\"/ssr/2\",\"initialTree\":[\"\",{\"children\":[\"ssr\",{\"children\":[[\"id\",\"2\",\"d\"],{\"children\":[\"__PAGE__\",{}]}]}]},\"$undefined\",\"$undefined\",true],\"initialHead\":[\"$L4\",null],\"globalErrorComponent\":\"$5\",\"notFound\":[\"$\",\"html\",null,{\"lang\":\"en\",\"children\":[\"$\",\"body\",null,{\"className\":\"overflow-y-scroll\",\"children\":[[\"$\",\"$L6\",null,{}],[\"$\",\"div\",null,{\"className\":\"lg:pl-72\",\"children\":[\"$\",\"div\",null,{\"className\":\"mx-auto max-w-4xl space-y-8 px-2 pt-20 lg:py-8 lg:px-8\",\"children\":[\"$\",\"div\",null,{\"className\":\"rounded-lg p-px shadow-lg\",\"children\":[\"$\",\"div\",null,{\"className\":\"rounded-lg p-3.5 lg:p-6\",\"children\":[\"$undefined\",[[\"$\",\"title\",null,{\"children\":\"404: This page could not be found.\"}],[\"$\",\"div\",null,{\"style\":{\"fontFamily\":\"system-ui,\\\"Segoe UI\\\",Roboto,Helvetica,Arial,sans-serif,\\\"Apple Color Emoji\\\",\\\"Segoe UI Emoji\\\"\",\"height\":\"100vh\",\"textAlign\":\"center\",\"display\":\"flex\",\"flexDirection\":\"column\",\"alignItems\":\"center\",\"justifyContent\":\"center\"},\"children\":[\"$\",\"div\",null,{\"children\":[[\"$\",\"style\",null,{\"dangerouslySetInnerHTML\":{\"__html\":\"body{color:#000;background:#fff;margin:0}.next-error-h1{border-right:1px solid rgba(0,0,0,.3)}@media (prefers-color-scheme:dark){body{color:#fff;background:#000}.next-error-h1{border-right:1px solid rgba(255,255,255,.3)}}\"}}],[\"$\",\"h1\",null,{\"className\":\"next-error-h1\",\"style\":{\"display\":\"inline-block\",\"margin\":\"0 20px 0 0\",\"padding\":\"0 23px 0 0\",\"fontSize\":24,\"fontWeight\":500,\"verticalAlign\":\"top\",\"lineHeight\":\"49px\"},\"children\":\"404\"}],[\"$\",\"div\",null,{\"style\":{\"display\":\"inline-block\"},\"children\":[\"$\",\"h2\",null,{\"style\":{\"fontSize\":14,\"fontWeight\":400,\"lineHeight\":\"49px\",\"margin\":0},\"children\":\"This page could not be found.\"}]}]]}]}]]]}]}]}]}]]}]}],\"asNotFound\":false,\"children\":[[\"$\",\"html\",null,{\"lang\":\"en\",\"children\":[\"$\",\"body\",null,{\"className\":\"overflow-y-scroll\",\"children\":[[\"$\",\"$L6\",null,{}],[\"$\",\"div\",null,{\"className\":\"lg:pl-72\",\"children\":[\"$\",\"div\",null,{\"className\":\"mx-auto max-w-4xl space-y-8 px-2 pt-20 lg:py-8 lg:px-8\",\"children\":[\"$\",\"div\",null,{\"className\":\"rounded-lg p-px shadow-lg\",\"children\":[\"$\",\"div\",null,{\"className\":\"rounded-lg p-3.5 lg:p-6\",\"children\":[\"$\",\"$L7\",null,{\"parallelRouterKey\":\"children\",\"segmentPath\":[\"children\"],\"error\":\"$undefined\",\"errorStyles\":\"$undefined\",\"loading\":\"$undefined\",\"loadingStyles\":\"$undefined\",\"hasLoading\":false,\"template\":[\"$\",\"$L8\",null,{}],\"templateStyles\":\"$undefined\",\"notFound\":\"$undefined\",\"notFoundStyles\":\"$undefined\",\"asNotFound\":false,\"childProp\":{\"current\":[\"$L9\",null],\"segment\":\"ssr\"},\"styles\":[]}]}]}]}]}]]}]}],null]}]]\n"])
</script>
<script>
    self.__next_f.push([1, "4:[[[\"$\",\"meta\",null,{\"charSet\":\"utf-8\"}],null,null,null,null,null,null,null,null,null,null,[\"$\",\"meta\",null,{\"name\":\"viewport\",\"content\":\"width=device-width, initial-scale=1\"}],null,null,null,null,null,null,null,null,null,null,[]],[null,null,null,null],null,null,[null,null,null,null,null],null,null,null,null,null]\n"])
</script>
<script>
    self.__next_f.push([1, "a:I{\"id\":\"(app-client)/./src/components/Tab.tsx\",\"chunks\":[\"app/head/layout:static/chunks/app/head/layout.js\"],\"name\":\"Tab\",\"async\":false}\n"])
</script>
<script>
    self.__next_f.push([1, "9:[\"$\",\"div\",null,{\"className\":\"space-y-9\",\"children\":[[\"$\",\"div\",null,{\"className\":\"flex justify-between\",\"children\":[\"$\",\"div\",null,{\"className\":\"flex flex-wrap gap-2 items-center\",\"children\":[[\"$\",\"$La\",\"/ssrundefined\",{\"item\":{\"text\":\"Home\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr1\",{\"item\":{\"text\":\"Leanne Graham\",\"slug\":\"1\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr2\",{\"item\":{\"text\":\"Ervin Howell\",\"slug\":\"2\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr3\",{\"item\":{\"text\":\"Clementine Bauch\",\"slug\":\"3\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr4\",{\"item\":{\"text\":\"Patricia Lebsack\",\"slug\":\"4\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr5\",{\"item\":{\"text\":\"Chelsey Dietrich\",\"slug\":\"5\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr6\",{\"item\":{\"text\":\"Mrs. Dennis Schulist\",\"slug\":\"6\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr7\",{\"item\":{\"text\":\"Kurtis Weissnat\",\"slug\":\"7\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr8\",{\"item\":{\"text\":\"Nicholas Runolfsdottir V\",\"slug\":\"8\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr9\",{\"item\":{\"text\":\"Glenna Reichert\",\"slug\":\"9\"},\"path\":\"/ssr\"}],[\"$\",\"$La\",\"/ssr10\",{\"item\":{\"text\":\"Clementina DuBuque\",\"slug\":\"10\"},\"path\":\"/ssr\"}]]}]}],[\"$\",\"div\",null,{\"children\":[\"$\",\"$L7\",null,{\"parallelRouterKey\":\"children\",\"segmentPath\":[\"children\",\"ssr\",\"children\"],\"error\":\"$undefined\",\"errorStyles\":\"$undefined\",\"loading\":\"$undefined\",\"loadingStyles\":\"$undefined\",\"hasLoading\":false,\"template\":[\"$\",\"$L8\",null,{}],\"templateStyles\":\"$undefined\",\"notFound\":\"$undefined\",\"notFoundStyles\":\"$undefined\",\"asNotFound\":false,\"childProp\":{\"current\":[\"$\",\"$L7\",null,{\"parallelRouterKey\":\"children\",\"segmentPath\":[\"children\",\"ssr\",\"children\",[\"id\",\"2\",\"d\"],\"children\"],\"error\":\"$undefined\",\"errorStyles\":\"$undefined\",\"loading\":\"$undefined\",\"loadingStyles\":\"$undefined\",\"hasLoading\":false,\"template\":[\"$\",\"$L8\",null,{}],\"templateStyles\":\"$undefined\",\"notFound\":\"$undefined\",\"notFoundStyles\":\"$undefined\",\"asNotFound\":false,\"childProp\":{\"current\":[\"$Lb\",null],\"segment\":\"__PAGE__\"},\"styles\":[]}],\"segment\":[\"id\",\"2\",\"d\"]},\"styles\":[]}]}]]}]\n"])
</script>
<script>
    self.__next_f.push([1, "b:[\"$\",\"div\",null,{\"className\":\"space-y-4\",\"children\":[[\"$\",\"h1\",null,{\"className\":\"text-2xl font-medium text-gray-100\",\"children\":\"qui est esse\"}],[\"$\",\"p\",null,{\"className\":\"font-medium text-gray-400\",\"children\":\"est rerum tempore vitae\\nsequi sint nihil reprehenderit dolor beatae ea dolores neque\\nfugiat blanditiis voluptate porro vel nihil molestiae ut reiciendis\\nqui aperiam non debitis possimus qui neque nisi nulla\"}]]}]\n"])
</script>
```

과거 getServerSideProps를 사용하는 애플리케이션에는 `<script id='__NEXT_DATA__' type='application/json'>` 라는 특별한 태그가 추가돼 있었고, 서버에서 미리 만들어진 정보를 바탕으로 클라이언트에서 하이드레이션을 수행했다.

리액트 18에서는 서버 컴포넌트에서 렌더링한 결과를 직렬화 가능한 데이터로 클라이언트에서 제공하고, 클라이언트는 이를 바탕으로 하이드레이션을 진행한다. 각 스크립트는 하나의 서버 컴포넌트 단위를 의미하며, 예제 코드의 마지막 스크립트에서 이 마지막 서버 컴포넌트의 흔적을 발견할 수 있다. 

이번에는 다른 유저를 클릭해서 같은 서버 컴포넌트의 다른 id를 제공하는 시나리오를 확인해보자. 다른 id를 선택하고 네트워크 탭을 확인하면 다음과 같은 정보를 확인할 수 있다.

![스크린샷 2024-09-05 오후 8 56 29](https://github.com/user-attachments/assets/c8015119-f895-4b63-a4e4-2f6339c0c944)

과거 getServerSideProps를 사용하던 당시에는 [id].json 형태로 요청을 보내 새로운 getServerSideProps의 실행 결과를 JSON 형태로 받았다면 리액트 18부터는 서버 컴포넌트의 렌더링 결과를 컴포넌트별로 직렬화된 데이터로 받아 이 데이터를 바탕으로 클라이언트에서 하이드레이션하는 데 사용한다. 

### 11.7.2 getStaticProps와 비슷한 정적인 페이지 렌더링 구현해보기

Next.js 13 이전까지는 정적 페이지 생성을 위해 getStaticProps나 getStaticPaths를 이용해 사전에 미리 생성 가능한 path를 모아둔 다음, 이 경로에 대해 내려줄 props를 미리 빌드하는 형식으로 구성돼 있었다. 이러한 방법은 headless CMS 같이 사용자 요청에 앞서 미리 빌드해둘 수 있는 페이지를 생성하는 데 매우 효과적이었다. 

Next.js 13에서 app 디렉터리가 생겨나면서 이 둘은 사라졌지만 이와 유사한 방식을 fetch와 cache를 이용해 구현할 수 있다.

```jsx
import { fetchPostById } from '#services/server'

export async function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }, { id: '4' }]
}

export default async function Page({ params }: { params: { id: string } }) {
  const data = await fetchPostById(params.id)

  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-medium text-gray-100">{data.title}</h1>
      <p className="font-medium text-gray-400">{data.body}</p>
    </div>
  )
}
```

generateStaticParams를 사용해 주소인 /app/ssg/[id]에서 [id]로 사용 가능한 값을 객체 배열로 모아뒀다. 그리고 Page 컴포넌트에서 이 각각의 id를 props로 받을 때 어떻게 작동할지 미리 정해졌다. 또 한 가지 주목할 것은 fetchPostById이다. fetchPostById에 별다른 옵션을 주지 않았는데, 이것은 가능한 모든 cache 값을 사용하도록 설정한 것과 같다.

<summary>
  <details>Next.js에서 사용가능한 cache 옵션</details>
    - force-cache: 캐시가 존재한다면 해당 캐시 값을 반환하고, 캐시가 존재하지 않으면 서버에서 데이터를 불러와 가져온다(기본값).
    - no-store: 캐시를 절대 사용하지 않고, 매 요청마다 새롭게 값을 불러온다.
    또는 `fetch(https://…, { next: {revalidate: false | 0 | number } } });` 를 사용해 캐시를 초 단위로 줄 수 있다.
</summary>

![스크린샷 2024-09-05 오후 8 57 22](https://github.com/user-attachments/assets/0a28bf2f-3a38-4c93-9402-dd5fc0079c75)

빌드한 결과 generateStaticParams로 선언한 모든 경우의 수에 대해 미리 페이지를 생성해둔 것을 확인할 수 있다. 따라서 실제 페이지에 접근할 때는 별다른 작업 없이 이 HTML만으로 페이지를 확인할 수 있으므로 접속 속도가 빠르다.

정적으로 미리 빌드해 두는 것뿐만 아니라 캐시를 활용하는 것도 가능하다. 이러한 방식을 Next.js에서 ‘Incremental Static Regeneration’이라 하는데, 정적으로 생성된 페이지를 점진적으로 갱신하는 것을 의미한다. 일정 기간 동안은 캐시를 통해 가져와 빠르게 렌더링하고, 시간이 지나면 새롭게 데이터를 불러오는 방식으로 페이지를 구성할 수 있다.

```jsx
import { fetchPostById } from '#services/server'

export const dynamicParams = true

export const revalidate = 15 // revalidate this page every 60 seconds

export async function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }, { id: '4' }]
}

export default async function Page({ params }: { params: { id: string } }) {
  const data = await fetchPostById(params.id)

  console.log(`generate page ${params.id}`)

  return (
    <div className="space-y-4">
      <div className="self-start whitespace-nowrap rounded-lg bg-gray-700 px-3 py-1 text-sm font-medium tabular-nums text-gray-100">
        마지막 렌더링 시간 (프로덕션 모드만 확인 가능): UTC{' '}
        {new Date().toLocaleTimeString()}
      </div>
      <h1 className="text-2xl font-medium text-gray-100">{data.title}</h1>
      <p className="font-medium text-gray-400">{data.body}</p>
    </div>
  )
}
```

이 페이지는 캐시 유효시간이 지난 이후에 다시 페이지를 방문해 보면 서버에서 페이지를 다시 생성해 마지막 렌더링 시간이 갱신된다. 이 컴포넌트는 서버 컴포넌트이므로 new Date()는 서버에서 빌드되는 시점을 의미한다. 이렇게 캐시된 페이지는 앞서 설명한 서버 액션인 revalidatePath를 통해 캐시를 강제로 갱신하는 것 또한 가능하다. 

### 11.7.3 로딩, 스트리밍, 서스펜스

Next.js 13에서는 스트리밍과 리액트의 서스펜스를 활용해 컴포넌트가 렌더링 중이라는 것을 나타낼 수 있다.

```jsx
import { Suspense } from 'react'

import { PostByUserId, Users } from './components'

export default async function Page({ params }: { params: { id: string } }) {
  return (
    <div className="space-y-8 lg:space-y-14">
      <Suspense fallback={<div>유저 목록을 로딩중입니다.</div>}>
        {/* 타입스크립트에서 Promise 컴포넌트에 대해 에러를 내기 때문에 임시 처리 */}
        {/* @ts-expect-error Async Server Component */}
        <Users />
      </Suspense>

      <Suspense
        fallback={<div>유저 {params.id}의 작성 글을 로딩중입니다.</div>}
      >
        {/* @ts-expect-error Async Server Component */}
        <PostByUserId userId={params.id} />
      </Suspense>
    </div>
  )
}

import { sleep } from '#lib/utils'
import { fetchPosts, fetchUsers } from '#services/server'

export async function Users() {
  // Suspense를 보기 위해 강제로 지연시킵니다.
  await sleep(3 * 1000)
  const users = await fetchUsers()

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}

export async function PostByUserId({ userId }: { userId: string }) {
  await sleep(5 * 1000)
  const allPosts = await fetchPosts()
  const posts = allPosts.filter((post) => post.userId === parseInt(userId, 10))

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

이 코드는 두 개의 서버 컴포넌트에서 fetch 작업을 하고, 이 두 서버 컴포넌트를 부모 컴포넌트에서 Suspense를 걸어두고 불러오는 예제이다. 이 컴포넌트가 그려지는 과정을 살펴보면 다음과 같다.
![스크린샷 2024-09-05 오후 8 57 58](https://github.com/user-attachments/assets/18297f09-905e-4bc3-93e1-10669afd66a4)
위 코드에서는 목록에서 3초, 작성 글에서는 5초의 강제 대기 시간을 갖는다.

![스크린샷 2024-09-05 오후 8 58 09](https://github.com/user-attachments/assets/c94cf7fc-241c-4fb5-85a8-689b1f3fa103)

![스크린샷 2024-09-05 오후 8 59 11](https://github.com/user-attachments/assets/572abd20-abf1-45c6-869c-1fa8decc3e83)

서서히 fetch 작업이 완료되면서 화면이 렌더링되는 것을 볼 수 있다. 그리고 개발자 도구에서 확인해보면 페이지 렌더링에 소요된 시간만큼 네트워크 요청도 발생한 것을 볼 수 있다. 

[스크린샷 2024-09-05 오후 8 58 29](https://github.com/user-attachments/assets/903da41f-24a9-4901-be53-ea72fbfea478)

```jsx
const main = async () => {
  const response = awiat fetch('/streaming/8',
    {
      // 옵션 생략
   });
  
	const reader = response.body.pipeThrough(new TextDecoderStream()).getReader()
	
	while(true) {
	  const { value, done } = await reader.read()
	  if (done) break
	  console.log('======================')
	  console.log(value)
	}
	
	console.log('Response fully received')
}

// 실행 결과 생략
```

서버 컴포넌트의 렌더링 결과를 직렬화해서 내려주는 것을 확인할 수 있다. 한 가지 더 눈 여겨봐야 할 것은 스트림을 통해 내려오는 데이터 단위(chunk)다. 최초 데이터는 서버에서 fetch 등의 작업을 기다릴 필요가 없는 Suspense 내부의 로딩 데이터가, 이후부터 fetch가 끝나면서 렌더링이 완료된 컴포넌트의 데이터를 하나씩 내려준다. 이는 Next.js 13과 리액트 18이 서버 컴포넌트의 렌더링과 이를 클라이언트에 제공하기 위해 스트리밍을 사용하고 있다는 증거다.
