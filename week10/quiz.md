## Quiz

- 진행자 : 박지영
- 날짜 : Aug 20 2024 <!-- e.g. Aug 4 2023 -->

---

<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. React 17에서는 이전 버전과 달리, JSX를 사용할 때 import React from 'react' 구문을 작성할 필요가 없어졌다. 이 구문이 필요했던 이유와, React 17부터 이 구문이 필요 없어지게 된 이유를 아래의 JSX 변환 코드를 참고하여 설명하라.
 
```js
// 이 컴포넌트를 변환합니다.
const Component = (
  <div>
    <span>hello world</span>
  </div>
)
```

```js
// before
var Component = React.createElement(
    'div',
    null,
    React.createElement("span", null, "hello world")
)

// after
'use strict'

var _jsxRuntime = require('react/jsx-runtime')

var Component = (0, _jsxRuntime.jsx)('div', {
  children: (0, _jsxRuntime.jsx)('span', {
    children: 'hello world',
  }),
})
```

<!--
답:
JSX는 브라우저가 이해할 수 있는 코드가 아니므로 JSX를 자바스크립트로 변환하는 과정이 꼭 필요하다. 16 버전까지는 JSX 변환을 위해 코드 내에서 React를 사용하는 구문이 없더라도 import React from 'react' 구문이 필요했다.
하지만 리액트 17부터는 바벨과 협력해 이러한 import 구문 없이도 JSX를 변환할 수 있게 됐다. 기존에는 createElement를 수행할 때 import React from 'react'까지 추가해주지는 않았지만, 17 버전에서는 require() 구문을 이용해 JSX를 변환할 때 필요한 모듈인 react/jsx-runtime을 불러오기 때문이다.
-->

<br/>

> 2. 리액트 17의 변경 사항이 아닌 것을 선택하라.

1. 이벤트 위임이 리액트 컴포넌트 최상단 트리에서 수행된다.
2. 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 실행한다.
3. 컴포넌트 내부에서 undefined를 반환하면 항상 에러가 발생한다.

<!--
답 : 2번

리액트 17에서도 이벤트 핸들러 내부에서 자동 배치가 이루어진다. 그러나 이것은 리액트 17의 변경 사항이 아니다.
리액트 18부터는 Promise나 setTimeout 같은 비동기 이벤트에서도 자동 배치가 이루어진다.
-->

<br/>

> 3. 리액트 18에서 추가된 useTransition 훅은 UI 변경 없이 상태를 업데이트해줌으로써 더 나은 사용자 경험을 제공한다. useTransition을 사용할 때 주의할 점을 [빈칸]을 채워 설명하라.

- startTransition 내부는 반드시 [빈칸1]와 관련된 작업만 넘길 수 있다.
- startTransition으로 넘겨주는 상태 업데이트는 다른 모든 동기 상태 업데이트로 인해 실행이 지연될 수 있다.
- startTransition으로 넘겨주는 함수는 반드시 [빈칸2]여야 한다.

<!--
답 :
빈칸1: setState와 같은 상태를 업데이트하는 함수
빈칸2: 동기 함수
-->
