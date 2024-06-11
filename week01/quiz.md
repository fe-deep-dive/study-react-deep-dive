## Quiz
- 진행자 : 박진아
- 날짜 : July 4 2024  <!-- e.g. Aug 4 2023 -->
---
<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. 두 객체를 비교한 결과 값은 무엇인지 설명해주세요.

```jsx
var hello = {
  great: 'hello, world'
}

var hi = {
 great: 'hello, world'
}

console.log(hello === hi) 

console.log(hello.grat === hi.great) 
```

<!--
답: 
false, true
객체는 저장할 때마다 다른 참조를 바라보기 때문에 내용이 같더라도 false를 반환.
원시값은 값을 평가하므로 true를 반환
-->

> 2. useState에서 setState는 어떻게 상태 값을 업데이트 할 수 있을까요?

```jsx
function Component() {
  const [state, setState] = useState();

  function handleClick() {
    setState((prev) => prev + 1);
  }

  // ...
}
```
<!--
답: 
useState는 클로저의 원리를 사용하고 있다. setState는 클로저 내부에 접근해 상태값에 접근할 수 있다.
-->

> 3. 출력 결과와 마이크로 태스크 큐를 설명해주세요.

```jsx
console.log('a');

setTimeout(() => {
  console.log('b');
}, 0);

Promise.resolve().then(() => {
  console.log('c');
});
```

<!--
답:
a, c, d 

마이크로 태스크 큐가 처리하는 작업은 process.nextTick, Promises, queueMicroRask, MutationObserver가 있다. 이 태스크는 태스크 큐보다 더 높은 우선순위를 갖고 있다.
-->
