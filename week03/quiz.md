## Quiz

- 진행자 : 서범석
- 날짜 : Jun 19 2024 <!-- e.g. Aug 4 2023 -->

---

<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. 다음은 `count` 상태 값을 1씩 올리고 싶어 작성한 코드입니다. 이 코드가 어떻게 작동하는지 설명하고, 문제점과 해결 방안을 설명해주세요.

```tsx
// count를 1초마다 1씩 올리고 싶어
const Counter = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => setCount(count + 1), 1000);
    return () => clearInterval(id);
  }, []); // 뭔가 주황색 경고가 뜬다.

  return <h1>{count}</h1>;
};
```

<!--
답: 1에서 더 이상 올라가지 않는다. 정확히는 setInterval은 계속 동작하지만, setCount 안에 있는 count가 첫 렌더링 시의 상태 값을 기억해 계속 0으로 초기화되고, 여기에 1을 더해 지속적으로 1로 보인다.

(100점) setCount(count + 1) 을 setCount((prev) => prev + 1)로 바꾼다.
(50점) useEffect의 의존성 배열에 count를 추가한다. (매 렌더링마다 Interval가 생성, 소멸된다.)
-->

<br/>

> 2. 아래는 자식 컴포넌트를 `React.memo`를 통해 메모이제이션한 코드입니다. 그러나 상태 값이 하나라도 변경되면 두 컴포넌트 모두 리렌더링되고 있습니다. 이유는 무엇이고, 어떻게 해결해야 하는지 설명해주세요.

```tsx
const 자식_컴포넌트 = memo(({ name, value, onChange }) => {
  useEffect(() => {
    console.log(name, "렌더링 되는거에요?");
  });

  return (
    <>
      <h1>
        {name} {value ? "온" : "오프"}
      </h1>
      <button onClick={onChange}>눌러보셈</button>
    </>
  );
});

function App() {
  const [상태1, set상태1] = useState(false);
  const [상태2, set상태2] = useState(false);

  // 얘를 눌러도 자식 컴포넌트 2개가 전부 리렌더링되고,
  const 토글1 = () => set상태1(!상태1);
  // 얘도 자식 컴포넌트 2개가 전부 리렌더링된다.
  const 토글2 = () => set상태2(!상태2);

  return (
    <>
      <자식_컴포넌트 name="범" value={상태1} onChange={토글1} />
      <자식_컴포넌트 name="부" value={상태2} onChange={토글2} />
    </>
  );
}
```

<!--
답: 상태1이나 상태2가 바뀌면 App 컴포넌트가 리렌더링된다. 그렇게 되면 토글1(), 토글2() 함수가 재생성되어 참조가 바뀌게 된다. 자식 컴포넌트에 전달되는 onChange 함수의 참조가 바뀌기 때문에, 두 컴포넌트는 모두 리렌더링된다.

해결 방법: useCallback으로 감싼다.
const 토글2 = useCallback(() => {
  set상태2(!상태!);
}, [상태2]);
-->

<br/>

> 3. `useEffect`를 클래스 컴포넌트의 생명주기 메서드처럼 사용하면 안되는 이유, `useContext`가 상태 관리 라이브러리가 아닌 이유를 설명해주세요.

<!--
답:
useEffect의 clean up 함수는 이전 상태를 청소해주는 개념이다. 렌더링 시점에서, 의존성 배열에 따라 부수효과를 작동시키는 훅이기 때문에, 정말 빈 배열로 사용해야 한다면 해당 위치에서 호출하는게 맞는지 등을 생각해봐야 한다.

useContext는 특정 상태를 기반으로 다른 상태를 만들어낼 수도 없고, 사용한다고 해서 렌더링 최적화를 할 수도 없기 때문에, 위의 두 조건을 충족시키지 못하여 상태 관리 라이브러리라고 할 수 없다.
-->
