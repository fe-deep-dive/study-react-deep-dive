## Quiz

- 진행자 : 김민정
- 날짜 : Jun 25 2024 <!-- e.g. Aug 4 2023 -->

---

<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. 다음 Next.js 코드를 보고, 접속한 각 url에 따라 어떤 결과를 가지는지 설명하세요.

```tsx
// pages/hello/[...greeting].tsx
import { useRouter } from "next/router";
import { useEffect } from "react";
import { NextPageContext } from "next";

export default function Gretting({ props: serverProps }: { props: string[] }) {
  const {
    query: { props },
  } = useRouter();

  useEffect(() => {
    console.log(props);
  }, [props, serverProps]);

  return (
    <>
      <ul>
        {serverProps.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </>
  );
}

export const getServerSideProps = (context: NextPageContext) => {
  const {
    query: { props },
  } = context;

  return {
    props: {
      props,
    },
  };
};
```

- /hello/1/2/3
- /hello/go/gu/ma
- /hello/gamja

<!--
답:
- /hello/1/2/3
props는 ['1', '2', '3'], console.log에 ['1', '2', '3']이 출력됨.
<li>1</li>
<li>2</li>
<li>3</li>

- /hello/go/gu/ma
props는 ['go', 'gu', 'ma'], console.log에 ['go', 'gu', 'ma']이 출력됨.
<li>go</li>
<li>gu</li>
<li>ma</li>

- /hello/gamja
props는 ['gamja'], console.log에 ['gamja']가 출력됨.
<li>gamja</li>
-->

<br/>

> 2. 다음 html은 App.tsx를 서버 사이드에서 렌더링 한 결과입니다.  
>    어떤 리액트 API를 사용하여 렌더링 되었는지 설명하고 client.js를 실행한 결과에 대해 설명해주세요.

```tsx
// App.tsx
const App = () => {
  const handleClick = () => {
    console.log("서버 사이드 렌더링!");
  };

  return (
    <>
      <button onClick={handleClick}>서버 사이드 렌더링?</button>
      <button onClick={handleClick}>서버 사이드 양파링</button>
      <button onClick={handleClick}>서버 사이드 샐러드</button>
    </>
  );
};
```

```html
<!DOCTYPE html>
<head>
  <title>React App</title>
</head>
<body>
  <div id="root">
    <button>서버 사이드 렌더링?</button>
    <button>서버 사이드 양파링</button>
    <button>서버 사이드 샐러드</button>
  </div>
</body>
</html>
```

```tsx
// client.js
import * as ReactDOM from "react-dom";
import App from "./App";

const rootElement = document.getElementById("root");

ReactDOM.hydrate(<App />, rootElement);
```

<!--
답 :
html에 data-reactroot 속성이 없으므로 App.tsx는 renderToStaticMarkup으로 렌더링 되었습니다.
따라서 해당 html은 순수한 HTML 문자열이기 때문에 hydrate를 실행하면 내부적으로 다음과 같은 과정이 수행됩니다.

1. hydrate가 인자로 주어진 리액트 컴포넌트에 대해 렌더링을 수행한다.
2. hydrate가 수행한 렌더링 결과와 인수로 넘겨받은 HTML을 비교한다.
3. HTML을 비교하였을 때 차이점이 있으므로 (data-reactroot 속성) hydrate가 렌더링한 결과를 기준으로 웹페이지를 일치시킵니다.
4. 이후 이벤트 핸들러가 연결됩니다.

결론적으로 두 번 렌더링 된다.
-->

<br/>

> 3. 다음 두 코드를 빌드했을 때의 차이점에 대해 Next.js의 동작방식을 기반으로 설명하세요.

```tsx
export default function Hello() {
  console.log(typeof window === "undefined" ? "네이버" : "카카오");
  return <>hello</>;
}

export const getServerSideProps = () => {
  return {
    props: {},
  };
};
```

```tsx
export default function Hi() {
  console.log(typeof window === "undefined" ? "쿠팡" : "배민");
  return <>hi</>;
}
```

<!--
답 :
(70점)
Hello는 getServerSideProps를 가지고 빌드되어 서버 사이드 런타임 체크가 되고, 서버 사이드에서 렌더링 된다. 또한 console.log()가 서버에 기록되어 window가 undefined이므로 '네이버'라는 문자열이 기록된다.

Hi는 getServerSideProps가 없이 빌드되어 서버 사이드 렌더링이 필요없는 정적인 페이지로 분류되고, 빌드 결과물에서 애초에 typeof window === 'undefined' ? '쿠팡'  : '배민'이 단순히 '배민'으로 축약되어 있다.

(30점)
getServerSideProps가 없으면 서버에서 실행하지 않아도 되는 페이지로 처리하고 typeof window의 처리를 모두 object로 바꾼 다음, 빌드 시점에 미리 트리쉐이킹을 해버리기 때문이다.

트리쉐이킹이란 ?  JavaScript 번들링 과정에서 사용되지 않는 코드(데드 코드)를 제거하는 것
-->
