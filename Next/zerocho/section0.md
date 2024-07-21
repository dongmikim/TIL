# Section 0. Intro

## Next.js란?

- React: 화면 단 라이브러리
- Next: 프레임워크 하나로 별도의 패키지 설치없이 라우팅, 데이터 페칭 등 가능

## Next13 구성

- App Router
- Pages Router

Pages Router만 있다가 2023년에 App Router가 도입되었다.

> 💡 우선순위: App Router > Page Router

### App Router 도입 이유

- Pages Router의 문제를 기존 구조에서 해결하기 어려워 App Router 도입
- App Router 도입으로 아래 작업들이 편리해짐
  - 페이지 별 공통적으로 사용되는 layout 기능 추가
  - 미들웨어를 이용해 권한 별 페이지 제어가 쉬워짐
  - React 18 버전 사용으로 서버 컴포넌트 활용이 가능

### 서버 컴포넌트

- Next 서버에서 리액트를 미리 렌더링해서 클라이언트에게 완성된 html을 보내주는것
- 자바스크립트 용량도 줄고 여러모로 이점이 많다
- 사용자의 브라우저에서 했던 일을 Next 서버에서 하는거라 서버 부담이 증가됨
- 그래서 Next는 서버에 **캐시**를 적극적으로 사용하여 서버 부담을 줄이려고 함
