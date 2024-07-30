## Quiz

- 진행자 : 김민정
- 날짜 : July 30 2024 <!-- e.g. Aug 4 2023 -->

---

<!--
1. 질문은 이해하기 쉽고 명확하게 적는다.
2. 문제는 아래의 예시를 참고해 작성한다.
3. 문제의 정답은 주석으로 표기한다.
-->

> 1. 다음 JSON은 `new Date()`를 AST 분석한 결과입니다. `new Date()`를 사용하지 않도록 규칙을 만들 때, 다음 빈칸을 채우세요.

```json
{
  "type": "Program",
  "start": 0,
  "end": 10,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 10,
      "expression": {
        "type": "NewExpression",
        "start": 0,
        "end": 10,
        "callee": {
          "type": "Identifier",
          "start": 4,
          "end": 8,
          "name": "Date"
        },
        "arguments": []
      }
    }
  ],
  "sourceType": "module"
}
```

```js
/**
 * @type {import('eslint').Rule.RuleModule}
 */
module.exports = {
  meta: {
    type: 'suggestion',
    docs: {
      description: 'disallow use of the new Date()',
      recommended: false,
    },
    fixable: 'code',
    schema: [],
    messages: {
      message:
        "'new Date()'는 클라이언트에서 실행 시 해당 기기의 시간에 의존적이라 정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해 주세요.",
    },
  },
  create: function (context) {
    return {
      <<<1>>>: function (node) {
        if (<<<2>>> && <<<3>>>) {
          context.report({
            node: node,
            messageId: 'message',
            fix: function (fixer) {
              return fixer.replaceText(node, 'ServerDate()');
            },
          });
        }
      },
    };
  },
};
```

<!--
답 :
1 : NewExpression
2 : node.callee.name === 'Date'
3: node.arguments.length === 0
-->

<br/>

> 2. React Testing Library의 getBy..., findBy..., queryBy... 의 차이에 대해 설명하세요.

<!--
답 :
getBy: 요소가 DOM에 존재한다고 가정할 때 사용, 요소가 없으면 즉시 에러
사용 시점: 테스트 중에 해당 요소가 반드시 있어야 하는 경우.

queryBy: 요소가 DOM에 존재하지 않을 수도 있을 때 사용, 요소가 없으면 null 반환
사용 시점: 요소가 없음을 확인하고 싶거나, 요소가 없을 가능성이 있는 경우.

findBy: 요소가 비동기적으로 나타날 때 사용, 요소가 나타날 때까지 기다리며, Promise를 반환. 요소가 특정 시간(기본값 1000ms) 안에 나타나지 않으면 에러
사용 시점: 요소가 비동기 작업 후에 나타날 것으로 예상되는 경우.

[요약]
getBy는 요소가 반드시 존재해야 하는 경우 사용합니다.
queryBy는 요소가 없을 수도 있을 때 사용합니다.
findBy는 비동기적으로 요소가 나타날 때까지 기다립니다.
-->

<br/>

> 3. userEvent.click과 fireEvent.click의 차이점을 설명하세요.

<!--
답 :
userEvent.click : 실제 사용자가 클릭하는 것과 유사하게 동작, 클릭 이벤트 외에도 pointer, mouse 이벤트를 포함하여 보다 현실적인 사용자 상호작용을 시뮬레이션, 내부적으로 fireEvent.click... 등을 호출

fireEvent.click : 단순히 클릭 이벤트를 트리거, 클릭 이벤트만 발생시키며, 다른 연관된 이벤트는 트리거되지 않음

-->
