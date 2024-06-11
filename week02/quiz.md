## Quiz
- 진행자 : 박지영
- 날짜 : June 11 2024  <!-- e.g. Aug 4 2023 -->
---
<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. 가상DOM이 왜 필요한지 리액트 컴포넌트의 렌더링 과정과 연관지어 설명해주세요.

<!--
답:
리액트 컴포넌트에 대한 정보를 1:1로 가지고 있는 것이 파이버이며, 이 파이버는 리액트 아키텍처 내부에서 비동기로 이뤄진다.
하지만 실제 브라우저 구조인 DOM에 반영하는 것은 동기적으로 일어나야 하고, 또 처리하는 작업이 많아 화면에 불완전하게 표시될 수 있는 가능성이 높으므로 이러한 작업을 가상에서, 즉 메모리 상에서 먼저 수행해서 최종적인 결과물만 실제 브라우저 DOM에 적용하는 것이 가상 DOM이다.
가상 DOM과 리액트의 핵심은 브라우저의 DOM을 더욱 빠르게 그리고 반영하는 것이 아니라, 값으로 UI를 표현하는 것이다. 화면에 표시되는 UI를 값으로 관리하고 이러한 흐름을 효율적으로 관리하기 위한 매커니즘이 바로 리액트의 핵심이다.
-->

> 2. 클래스 컴포넌트의 3가지 생명주기 시점을 간단하게 설명해주세요.
<!--
- 마운트(mount): 컴포넌트가 마운팅(생성)되는 시점
- 업데이트(update): 이미 생성된 컴포넌트의 내용이 변경(업데이트)되는 시점
- 언마운트(unmount): 컴포넌트가 더 이상 존재하지 않는 시점
-->

> 3. 아래 코드의 ClassComponent는 handleClick을 클릭하면 3초 뒤에 props에 있는 user를 alert로 띄워준다. 만약 3초 사이에 props를 변경하면 어떻게 될지 설명하세요.
```jsx
import React from 'react'

interface Props {
  user: string
}

export class ClassComponent extends React.Component<Props, {}> {
  private showMessage = () => {
    alert('Hello ' + this.props.user)
  }

  private handleClick = () => {
    setTimeout(this.showMessage, 3000)
  }

  public render() {
    return ‹button onClick={this.handleClick}>Follow</button>
  }
}
```


<!--
답:
ClassComponent의 경우에는 3초 뒤에 변경된 props를 기준으로 메시지가 뜨고, FunctionalComponent는 클릭했던 시점의 props 값을 기준으로 메시지가 뜬다. 

클래스 컴포넌트는 props의 값을 항상 this로부터 가져온다. 클래스 컴포넌트의 props는 외부에서 변경되지 않는 이상 불변 값이지만 this가 가리키는 객체, 즉 컴포넌트의 인스턴스의 멤버는 변경 가능한(mutable) 값이다. 따라서 render 메서드를 비롯한 리액트의 생명주기 메서드가 변경된 값을 읽을 수 있게 된다. 따라서 이 경우 부모 컴포넌트가 props를 변경해 컴포넌트가 다시 리렌더링됐다는 것은 this.props의 값이 변경된 것이다. 따라서 showMessage는 새로운 props의 값을 읽을 수 있게 된다.
-->
