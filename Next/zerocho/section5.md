# Section 5. 보너스: 백엔드 개발자가 퇴사했다

next 13 App Router부터는 서버 컴포넌트 도입으로 프론트엔드 서버 부담이 크게 증가되었다.  
그래서 Next는 4가지의 캐싱 메커니즘을 통해 성능을 최적화한다.

참고: [Document](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)

### 1. Request Memoization

- 동일한 API 요청을 사용처에서 여러번해도, 동일한 API 요청은 내부에서 자동으로 요청을 1번으로 줄여준다.
- Duration: 다른 페이지로 이동하는 경우 다시 요청된다.

### 2. Data Cache ⭐️

- 프론트에서 백엔드 서버로 보낸 요청들을 얼마나 캐시할 것인가, 한 번 보낸 요청을 얼마동안 기억할 것인가
- 데이터 캐시는 Request Memoization과 다르게 기본적으로 계속 유지가 된다.
- 그래서 revalidate나 opt-out 설정을 잘해줘야 한다. 그렇지 않으면 사용자들이 데이터를 평생 못 볼수도 있다.
- Duration: 기본적으로 백엔드 캐시가 설정된다.

**데이터 캐시를 사용하지 않는 방법**

- `cache: no-store`
- 캐시하지 않고, 매번 데이터를 새로 받아오겠다는 의미

**라이브러리 캐싱 끄기**

- `export const dynamic = 'force-dynamic'`
- 라이브러리에서 백엔드 서버로 보내는 요청도 캐싱이 된다.  
  이를 막기 위해 `page.js`에 다음 설정을 추가하면 페이지에서 사용하는 요청 모두를 캐싱하지 않게 된다.

### 3. Full Route Cache

- 컨텐츠가 업데이트되면 새로 페이지를 렌더링한다. (캐시를 사용하지 않는다)
- cookie, searchParams를 사용하는 순간 full route cache는 동작하지 않는다.
  위 요소들은 자꾸 바뀌는 데이터들이고, 변경될 때마다 페이지를 다시 그려야 하므로 캐싱을 하는 의미가 없다
- 즉, 뉴스기사 또는 블로그 같은 정적인 페이지일 때만 의미가 있다.

### 4. Router Cache

- 유일하게 클라이언트에서 동작하고, 컴포넌트별로 기억하는 캐시
- 한 번 받아온 layout은 당연히 재사용되는 컴포넌트이므로 서버에 굳이 캐싱할 필요가 없다
- 기본적으로 라우터 캐시를 off할 수는 없으며, 30초 캐싱이 유지된다.
- Duration: 새로고침하기 전까지는 계속 유지된다.

**30초 설정으로 인해 해로그인하고 프로필 사진이 안 바뀌는 경우 해결법**  
30초 캐시로 인해 `router.replace()`로 초기화를 해줘야 한다.

## Static(SSG), ISR 모드

Next에는 2가지 배포 모드가 존재한다.

- Static Mode
- Dynamic Mode

```js
// next.config.js
const nextConfig = {
  output: 'export', // 있으면 static, 없으면 dynamic 모드
  async rewrites() {
    return [ //... ]
  }
}
```

### Static 모드

- next 서버없이 정적인 html 페이지로 구성되어 있는 정적 사이트
- 빌드 시점에 모든 컨텐츠가 결정된다.
- 적용 사이트: 컨텐츠 변경이 없는 회사 소개 사이트 등

### ISR 모드

- Page Router에 있던 모드로 현재 App Router에서 사라졌다.
- SSG처럼 서버가 없다가, 컨텐츠에 변경이 있다면 html을 다시 업데이트하는 방식
- 뉴스나 블로그처럼 변경이 많지 않은 컨텐츠에 사용한다.
- App Router에서 ISR 모드를 사용하고 싶다면?
  - fetch, cache, revalidating 전략을 통해 사용할 . 수있다.

## 배포

배포해야 하는 서버

- Frontend Server
- Backend Server
- DB

실 프로덕트의 동작을 위해 3개의 서버를 배포해야 한다. 그러나 프론트엔드 개발자의 역할은 실제로 배포까지는 하지 않는다.

보통 프론트엔드 개발자의 역할은 서버 담당자에게 프론트엔드 서버를 전달해서 설정한 URL에서 돌도록 요청하면 된다.  
그러나 요즘은 프론트 서버도 프론트 개발자가 직접 배포하는 트렌드이다..!

### AWS에 배포 실습

1. git에 업로드
2. aws -> EC2 -> 인스턴스 시작

## Websocket

지금 강의에서는 socket.io4 버전으로 설치한다.
항상 클라이언트와 서버 버전이 일치해야 한다!

```
npm i socket.io-client@4
```

socket.io를 서버 컴포넌트에서 쓰면 메모리 누수가 발생한다. 주의!

웹소켓이 없는 구형 브라우저는 http 폴링을 지원

> **웹소켓**  
> 서버에서 클라이언트에게 데이터를 실시간으로 보내줄 수 있음
> 웹소켓이 없던 시절에는 클라에서 주기적으로 서버한테 새로운 데이터를 물어봐서 응답을 받아야 했다

커넥션 두번이상 방지

**커스텀 훅을 쓸 때 주의점**  
useSocket훅을 호출할 때마다 state가 다시 생성되기 때문에 socket state가 공유되지 않는다.

그래서 공유할 데이터는 `useState`를 사용하지 않고 함수 외부에서 `let` 변수로 작성한다.  
안티패턴이지만 커스텀훅간 공유할 데이터는 이렇게 작성해야 한다.

## Vanilla Extract 적용

> 상세 코드는 /lecture 폴더에서 확인

### 미디어 쿼리에서 변수 설정 방법

```css
/* as is */

@medit (prefers-color-scheme: dark) {
  :root {
    --foreground: 255, 255, 255;
  }
}
```

```ts
/* to be */

// 테마의 이름을 정하는 글로벌 객체
const global = createGlobalThemeContract({
  foreground: {
    color: 'foreground-rgb',
  },
  // ...
})

// 테마별 컬러에 대한 설정부
const lightGlobalTheme = {
  foreground: {
    color: 'rgb(255,255,255)',
  },
}
const darkGlobalTheme = {
  foreground: {
    color: 'rgb(255,255,255)',
  },
}

// 라이트 테마를 먼저 적용
createGlobalTheme(':root', global, lightGlobalTheme)

// 루트가 다크모드일 땐 darkGlobalTheme를 적용한다
globalStyle(':root', {
  '@media': {
    '(prefers-color-scheme: dark)': {
      vars: assignVars(global, darkGlobalTheme),
    },
  },
})
```
