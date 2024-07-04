## Quiz

- 진행자 : 박지영
- 날짜 : July 4 2024 <!-- e.g. Aug 4 2023 -->

---

<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. Redux와 Recoil이 데이터를 저장하는 방법의 차이점을 설명하세요.

<!--
답:
Redux는 하나의 전역 스토어에 상태 객체를 저장해두고 이 상태를 하위 컴포넌트에 전파할 수 있다.
Recoil은 리덕스와 달리 atom이라는 단위로 상태를 관리한다.
-->

<br/>

> 2. 상태를 저장하고 가져오며 변경을 추적하여 리렌더링할 수 있는 Store를 만들기 위한 빈 칸을 채우세요.

```jsx
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never

type Store<State> = {
  get: () => State
  set: (action: Initializer<State>) => State
  subscribe: (callback: () => void) => () => void
}

export const createStore = <State extends unknwon>(
  initialState: Initializer<State>,
): Store<State> => {
  let state = typeof initialState !== 'function' ? initialState : initialState()

  const callbacks = new [빈칸]<() => void>()

  const get = () => state
  const set = (nextState: State | ((prev: State) => State)) => {
    state =
      typeof nextState === 'function'
        ? (nextState as (prev: State) => State)(state)
        : nextState

    [빈칸]

    return state
  }

  const subscribe = (callback: () => void) => {
    callbacks.[빈칸](callback)

    return () => {
      callbacks.delete(callback)
    }
  }

  return { get, set, subscribe }
}
```

<!--
답 :
1. Set
2. callbacks.forEach((callback) => callback())
3. add
-->

<br/>

> 3. 리액트에서 상태 관리 라이브러리의 필요성을 설명하세요.

<!--
답 :
리액트에서 상태가 변화하면 렌더링이 발생한다.
상태가 많아지고 복잡할수록 상태의 위치와 범위 관리, 상태 변화 감지, tearing 방지 등 상태 관리가 어려워진다.
그렇기 때문에 전역에서 상태에 접근할 수 있게 해주는 상태 관리 라이브러리가 필요하다.
-->
