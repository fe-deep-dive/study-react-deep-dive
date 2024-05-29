## Commit Message Format
---
각 커밋 메세지는 **header** , **body**, **footer** 로 구성된다.

```
<header>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

`header`는 필수이며, [Commit Message Header](#commit-header)와 같은 형식을 따른다.

`body`는 선택 사항이며, 긴 설명이 필요한 경우에 작성한다. 최대 75자를 넘기지 않도록 [Commit Message Body](#commit-body) 형식을 따라 작성한다.

`footer`도 선택 사항이며, Issue tracker ID 등을 명시하고 싶은 경우에 [Commit Message Footer](#commit-footer) 형식을 따라 작성한다.

</br>

#### <a name="commit-header"></a>Commit Message Header

```
<type>: <short summary>
  │            │
  │            └─⫸ 현재 Commit에 대한 내용을 요약
  │
  └─⫸ Commit Type: feat|fix|refactor|comment|docs...
```

`<type>` 과 `<short summary>` 모두 필수로 작성한다.

#### Type

|Tag Name|Description|
|---|---|
|feat|새로운 기능 추가|
|fix|버그 수정|
|refactor|코드 리팩토링|
|comment|주석 추가 및 변경|
|docs|문서 수정|
|test|테스트 코드 작성 혹은 변경|
|rename|파일, 폴더를 이동 혹은 이름 변경|
|remove|파일 삭제|
|chore|그 외 기타|

#### Summary

`<summary>` 에는 현재 Commit의 변경 사항에 대한 간단한 설명을 작성한다.

- 명령형 현재 시제를 사용한다.
	- e.g. add login button, 로그인 버튼 추가
- 끝에 점(.)을 붙이지 않는다.

</br>

#### <a name="commit-body"></a>Commit Message Body

`Header` 에 작성하기에는 긴 내용의 경우, `Body` 에 작성한다.

- `Summary` 와 마찬가지로 명령형 현재 시제를 사용한다.
- 최대 75자를 넘기지 않도록 작성한다.
- **어떻게** 했는지가 아니라, **무엇을 왜** 했는지 작성한다.

</br>

#### <a name="commit-footer"></a>Commit Message Footer

`Footer` 에는 Issue link 및 관련 PR을 참조하는 경우 작성한다.
Issue Tracker의 유형은 다음과 같다.
- Fixes: 이슈 수정 중 (아직 해결되지 않은 경우)
- Resolves: 이슈를 해결했을 때 사용
- Ref: 참고할 이슈가 있을 때 사용
- Related to: 해당 커밋에 관련된 이슈 번호 (아직 해결되지 않은 경우)

예를 들어
```
Fixes #<issue number> Related to: #<issue number>, #<issue number>
```

</br>

#### Example

```
feat: 18장 요약본, 퀴즈 추가

18장 함수와 일급 객체
- scope 관련 내용 작성중
- 퀴즈 3개 추가

Related to: #17
```
