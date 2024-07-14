- JSX는 자바스크립트 표준 코드 ❌
  - 페이스북이 임의로 만든 새로운 문법
- 따라서 JSX는 반드시 트랜스파일러를 거쳐야 런타임이 이해할 수 있는 의미 있는 JS 코드로 변환된다.

**JSX 설계 목적**

- HTML이나 XML을 자바스크립트 내부에 표현하는 것이 유일한 목적은 아니다.
- 다양한 트랜스파일러에서
- 다양한 속성을 가진 **트리 구조**를
- **토큰화**해 **ECMAScript로 변환**하는데 초점을 두고 있다.

더 쉽게 말하자면,

- JSX 내부에 트리 구조로 표현하고 싶은 다양한 것들을 작성해 두고
- 이 JSX를 트랜스파일 과정을 거쳐 자바스크립트(ECMAScript)가 이해할 수 있는 코드로 변경하는 것이 목표

JSX는 자바스크립트 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는데 도움을 주는 새로운 문법이다.

## 구성요소

### JSXElements

- JSXElementName
  - JSXIdentifier

    - JSX 내부에서 사용할 수 있는 식별자
    - $와 \_만 가능

    ```js
    function Valid() {
      return <$></$>
    }

    function Valid2() {
      return <_></_>
    }
    ```

````
	- JSXNamespacedName
		- `JSXIdentifier:JSXIdentifier`의 조합
		- :을 통해 서로 다른 식별자를 이어준다
		- 연속은 묶을 수 없다.
		```js
		function Valid () {
			return <foo:bar></foo:bar>
		}

		// 불가능
		function iInvalid () {
			return <foo:bar:baz></foo:bar:bza>
		}
		```
	- JSXMemberExpression
		- `JSXIdentifier:JSXIdentifier`의 조합
		- .을 통해 서로 다른 식별자를 이어준다.
		- 여러 개 이어서 가능하다.
		```js
		function Valid () {
			return <foo.bar></foo.bar>
		}

		function Valid2 () {
			return <foo.bar.baz></foo.bar.bza>
		}
		```
### JSXAttributes
### JSXChildren
### JSXStrings
```js
/* \을 사용하는데 문제 없다. */
<button>\</button>

// Uncaught SyntaxError: Invalid or unexpected token
let excape1 = '\'

// Valid
let escape2 = '\\'
````

- \ 는 자바스크립트에서 특수문자를 처리할 때 사용되므로
- 몇 가지 제약사항이 있다
  - \ 를 표현하기 위해서는 \\ 로 이스케이프해야 한다.

## JSX는 어떻게 자바스크립트로 변환될까?

`@babel/plugin-transform-react-jsx` 플러그인을 통해 변환된다.

<JSX가 변환되는 특성을 활용한 리팩토링 예시>
JSX 반환값이 결국 `React.createElement`로 귀결된다는 사실을 파악한다면
아래처럼 리팩토링할 수 있다.

```ts
// ❌ props 여부에 따라 children 요쇼만 달라지는 경우
// 굳이 번거롭게 전체 내용을 삼항 연산자로 처리할 필요가 없다.
// 이 경우 불필요한 코드 중복이 일어난다.

import { PropWidthChildren } from 'react'

function TextOrHeading({
	isHeading,
	children
}: PropsWithChildren<{ isHeading: boolean }>) {
	return isHeading ? (
		<h1 className="text">{children}</h1>
		) : (
		<span className="text">{children}</span>
		)
	)
}
```

```ts
// ✅ JSX가 변환되는 특성을 활용한다면
// 다음과 같이 간결하게 처리할 수 있다.
import { createElement, PropWidthChildren } from 'react'

function TextOrHeading({
  isHeading,
  children,
}: PropsWithChildren<{ isHeading: boolean }>) {
  return createElement(
    isHeading ? 'h1' : 'span',
    { className: 'text' },
    children
  )
}
```

---

참고

- 모던 리액트 Deep Dive
