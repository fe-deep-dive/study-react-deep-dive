## Quiz

- 진행자 : 김민정
- 날짜 : Jun 25 2024 <!-- e.g. Aug 4 2023 -->

---

<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1.

<!--

-->

<br/>

> 2.

<!--

-->

<br/>

> 3. 다음 두 코드를 빌드했을 때의 차이점에 대해 Next.js의 동작방식을 기반으로 설명하세요.

```tsx
export default function Hello() {
  console.log(typeof window === 'undefined' ? '네이버' : '카카오');
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
  console.log(typeof window === 'undefined' ? '쿠팡' : '배민');
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
