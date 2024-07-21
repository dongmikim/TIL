# Section 1. 기획자와 디자이너가 기획서를 던져주었다

## Layout

- 13 버전에서 도입되었다.
- 레이아웃 페이지는 리렌더링되지 않는다.

### RootLayout

- 모든 페이지의 공통 레이아웃
- app 경로 바로 하위(`app/layout.tsx)에 생성하는 최상위 레이아웃
- 필수이며, `html`, `body` 태그는 필수

```tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang='en'>
      <body className={inter.className}>{children}</body>
    </html>
  )
}
```

**현재 x.com router 구조**

- 메인(로그인 전): /
- 로그인: i/flow/login
- 회원가입: i/flow/signup
- 메인(로그인 후): /home
- 탐색하기: /explore
- 쪽지: /messages
- /compose/tweet
- 트윗 상세: [username]/status/[id]
  - 다이나믹 라우팅 slug

페이지별 레이아웃: 만들고 싶은 해당 폴더 하위에 `layout.tsx` 생성

**폴더 구조**

```
app
 ┣ (afterLogin)
 ┃ ┣ [username]
 ┃ ┃ ┣ status
 ┃ ┃ ┃ ┗ [id]
 ┃ ┃ ┃ ┃ ┗ page.tsx
 ┃ ┃ ┗ page.tsx
 ┃ ┣ compose
 ┃ ┃ ┗ tweet
 ┃ ┃ ┃ ┗ page.tsx
 ┃ ┣ explore
 ┃ ┃ ┗ page.tsx
 ┃ ┣ home
 ┃ ┃ ┣ layout.tsx
 ┃ ┃ ┗ page.tsx
 ┃ ┣ messages
 ┃ ┣ search
 ┃ ┗ layout.tsx
 ┣ (beforeLogin)
 ┃ ┣ @modal
 ┃ ┃ ┣ (.)i
 ┃ ┃ ┃ ┗ flow
 ┃ ┃ ┃ ┃ ┣ login
 ┃ ┃ ┃ ┃ ┃ ┗ page.tsx
 ┃ ┃ ┃ ┃ ┗ signup
 ┃ ┃ ┗ default.tsx
 ┃ ┣ _component
 ┃ ┃ ┣ LoginModal.tsx
 ┃ ┃ ┗ login.module.css
 ┃ ┣ i
 ┃ ┃ ┗ flow
 ┃ ┃ ┃ ┣ login
 ┃ ┃ ┃ ┃ ┗ page.tsx
 ┃ ┃ ┃ ┗ signup
 ┃ ┣ login
 ┃ ┃ ┗ page.tsx
 ┃ ┣ layout.tsx
 ┃ ┗ page.tsx
 ┣ favicon.ico
 ┣ globals.css
 ┣ layout.tsx
 ┣ not-found.tsx
 ┗ page.module.css
```

## Template

template 사용 상황

- 페이지를 이동하며 기록해야 될때, 리렌더링 개념보다 새롭게 마운트 되는 개념
- e.g. 구글 애널리틱스

## layout과 template 차이

|          | layout | template |
| -------- | ------ | -------- |
| 리렌더링 | X      | O        |

---

> **css module 선택 이유**  
> 다음과 같은 이유로 인해 클론코딩이므로 간단하게 css 모듈로 선택
>
> - tailwind: 호불호 심하고 가독성 X
> - styled component -> 서버 컴포넌트에서 SSR 문제
> - sass
> - css module
> - vanilla extract -> windows와 문제

## Route

> 💡 요구사항
>
> 1. 로그인은 /login으로 라우팅 된 후 i/flow/login으로 리다이렉트 처리
> 2. 메인 페이지에서 로그인 모달은 메인 배경을 뒤로 두고 표시되어야 한다. (Parallel Route 이용)
> 3. 새로고침할 경우 배경 페이지 없이 모달만 표시되어야 한다. (Intercepting Route 이용)

### Parallel Route

동일한 레이아웃 내에서 하나 이상의 페이지를 동시에 또는 조건부로 렌더링할 수 있다.

2번 요구사항 내용은 로그인 페이지 위에 로그인 모달을 띄우는 것이다.  
즉, `(beforeLogin)/page.tsx` 가 떠 있고, 그 위에 `(beforeLogin)/i/flow/login/page.tsx`가 떠 있어야 함

```tsx
// (beforeLogin)/page.tsx
import { ReactNode } from 'react'
import styles from '@/app/page.module.css'

type Props = { children: ReactNode; modal: ReactNode }

export default function Layout({ children, modal }: Props) {
  return (
    <div className={styles.container}>
      Before login Layout
      {children}
      {modal}
    </div>
  )
}
```

```
// (beforeLogin)/@modal/i/flow/login/page.tsx

export default function Page() {
  return (
    <>
      로그인 모달
    </>
  )
}
```

**Slot**
`@modal` 슬롯은 props로 부모 레이아웃에게 전달된다.

그러나 위와 같이 설정할 경우 발생하는 문제가 있다.
초기 로드 시, @modal의 페이지를 찾지 못하고 404가 뜬다.

그래서 이를 해결하기 위해 패러렐 라우트의 기본 페이지가 있다.

#### default.js

초기 로드 또는 새로고침 중에 일치하지 않는 슬롯에 대한 폴백으로 렌더링할 default.js 파일을 정의할 수 있다.  
즉, default 페이지를 정의하면 초기 로드 또는 새로고침할 때 `@modal/default.tsx`를 읽으므로 에러가 해결된다.

```
// @modal/default.tsx
export default function Default() {
  return null
}
```

```
(beforeLogin)
 ┣ @modal
 ┃ ┣ (.)i
 ┃ ┃ ┗ flow
 ┃ ┃ ┃ ┣ login
 ┃ ┃ ┃ ┃ ┣ login.module.css
 ┃ ┃ ┃ ┃ ┗ page.tsx // 로그인 페이지에서 i/flow/login으로 접속할 경우 뜨는 페이지
 ┃ ┗ default.tsx
 ┣ i
 ┃ ┗ flow
 ┃ ┃ ┣ login
 ┃ ┃ ┃ ┣ login.module.css
 ┃ ┃ ┃ ┗ page.tsx // 새로고침할 경우 뜨는 페이지 (로그인 페이지는 표시되지 않고, 모달만 뜬다)
 ┃ ┃ ┗ signup
 ┗ page.tsx
```

## Navigation

**Soft Navigation**  
클라이언트 측 탐색 중에 Next.js는 부분 렌더링을 수행하여 슬롯 내의 하위 페이지를 변경하는 동시에 다른 슬롯의 활성 하위 페이지는 현재 URL과 일치하지 않더라도 유지한다.

**Hard Navigation**  
새로고침 후 Next.js는 현재 URL과 일치하지 않는 슬롯의 활성 상태를 확인할 수 없다. 대신 일치하지 않는 슬롯에 대해 default.js 파일을 렌더링하거나 default.js가 존재하지 않는 경우 404를 렌더링한다.

## private folder

- 위 로그인 페이지 코드들이 중복되므로 private folder에 공통 파일로 정의할 수 있다.
- private folder는 폴더명 앞에 `_` prefix를 붙인다. -> `_component`
- 정의된 공통 파일은 사용처 에서 import하여 사용할 수 있다
