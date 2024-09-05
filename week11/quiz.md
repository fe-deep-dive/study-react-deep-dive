## Quiz
- 진행자 : 박진아
- 날짜 : Sep 5 2024  <!-- e.g. Aug 4 2023 -->
---
<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. 서버 컴포넌트 특징을 모두 고르기

1. 서버에서 컴포넌트를 렌더링 할 수 있다.
2. useState 훅을 사용할 수 있다.
3. 단 한 번 렌더링되면 그걸로 끝이다.
4. 데이터베이스에 접근할 수 있다.

<!--
답: 1, 3, 4
-->
<br/>

> 2. 리액트 컴포넌트 트리는 클라이언트 및 서버 컴포넌트가 혼재되어 있습니다. 클라이언트 컴포넌트 하위에 서버 컴포넌트를 구성할 수 있는 방법과 그것이 가능한 이유는 무엇일까요?
![스크린샷 2024-09-05 오후 7 32 08](https://github.com/user-attachments/assets/ecf8be05-5228-4499-bea8-9335db41a6c3)

<!--
서버 컴포넌트를 클라이언트 컴포넌트의 children또는 props로 넘겨준다. 서버 컴포넌트는 이미 서버에서 만들어진 트리(직렬화된 데이터)를 삽입해서 보여주면 되기 때문이다.
-->
<br/>

> 3. 데이터를 캐싱하지 않도록 '?' 채우고 틀린 부분이 있으면 고쳐주세요.
```jsx
// app/page.tsx
async function fetchData() {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts`,
    { ? },
  )
  const data = await res.json()
  return data;
}

export default function Page() {
  const data: Array<any> = fetchData()
  
  return (
    <ul>
      {data.map((item, key) => (
        <li key={key}>{item.id}</li>
      ))}
    </ul>
  )
}
```

<!--
- { cache: 'no-cache' } 또는 { next: { revalidate: 0 }}
- 서버 컴포넌트 사용하기 위한 aync, await
-->
