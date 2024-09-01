# Section 3. 백엔드 개발자에게 API 받기 전

## 1. MSW로 API 모킹하기

### 1.1 MSW 설정

```bash
npx msw init public/ --save
npm i msw -D
```

설치를 하면 public 폴더에 `mockServiceWorker.js` 파일이 생성된다.

**mocks 폴더 추가**

```
mocks
 ┣ browser.ts // 설정부
 ┣ handlers.ts
 ┗ http.ts // 설정부
```

`handlers.ts`에서 모킹할 API를 작성한다.

## 2. Server Actions 사용하기

Server Action은 React에서 제공하는 Server Components의 확장된 기능이다.  
클라이언트에서 서버로 요청을 보내지 않고도 서버에서 직접 비동기 로직이 돌아가게 해준다.

### 2.1 서버 컴포넌트

```tsx
const submit = async (formData: FormData) => {
  'use server'
  let shouldRedirect = false

  try {
    const response = await fetch(`${process.env.NEXT_PUBLIC_BASE_URL}/api/users`, {
      method: 'post',
      body: formData,
      credentials: 'include', // 이 속성이 있어야 쿠키 전달 가능
    })
    shouldRedirect = true
  } catch (err) {
    console.error(err)
    shouldRedirect = false
  }

  if (shouldRedirect) {
    redirect('/home')
  }
}
```

- 쿠키를 전달하기 위해 `credentials: 'include` 설정 추가
- `redirect` 함수는 try-catch 구문에서 사용이 불가능하기 때문에 `shouldRedirect` 플래그 변수를 이용한다.

### 2.2 클라이언트 컴포넌트

서버 컴포넌트에서 Server Actions을 사용할 경우, 클라이언트에서 응답에 따라 화면이 달라지는 상황을 대응하기 어려운 문제가 있다.

이를 해결하기 위해 form의 action 함수를 모듈로 분리하고, 나머지 form에서는 `react-dom`의 `useFormState`, `useFormStatus`를 이용한다.

```tsx
import { useFormState, useFormStatus } from 'react-dom'
const [state, formAction] = useFormState(onSubmit, { message: null })
```

## 3. 미들웨어, API라우트, catch-all 라우트

### 3.1 next-auth

`next-auth`는 Next에서 인증을 쉽게 구현할 수 있도록 도와주는 라이브러리이다.
카카오, 구글 등 다양한 인증 제공자를 지원하여 인증 기능을 빠르게 구현할 수 있게 해준다.

**설치 방법**  
`npm i next-auth@beta @auth/core`

**설정**

```ts
// auth.ts
import NextAuth from 'next-auth'
import GitHub from 'next-auth/providers/github'
export const { auth, handlers } = NextAuth({ providers: [GitHub] })

// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/auth'
export const { GET, POST } = handlers
```

### 3.2 Middleware

Next에서 제공하는 미들웨어는 HTTP 요청을 처리하기 전에 실행되는 코드를 작성할 수 있게 해주는 기능이다.
Route에 접근하기 전 단계에서 요청과 응답을 가로채서 인증 체크, 리다이렉션, 로그 기록 등의 작업을 할 수 있도록 해준다.

**파일 위치**
루트 디렉토리에 `middleware.ts` 파일을 생성한다.

```ts
// middleware.ts
export { auth as middleware } from '@/auth'
```

**미들웨어 설정**
match를 통해 특정 경로에서 실행되도록 미들웨어 설정을 분기처리할 수 있다.

```ts
export const config = {
  matcher: '/about/:path*',
}
```

### 3.3. API Route

- 동적 세그먼트: `[...folderName]`
- `app/api/auth/[…nextAuth]/route.ts`를 만들어 auth.ts의 GET, POST를 import하면, `/api/auth`를 통해 들어오는 GET, POST 요청은 모두 next auth가 관리하게 된다.

## 4. next-auth로 로그인하기

```ts
import NextAuth from 'next-auth'
import Credentials from 'next-auth/providers/credentials'

export const { signIn, signOut, auth } = NextAuth({
  providers: [
    Credentials({
      credentials: {
        username: { label: 'Username' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize({ request }) {
        const response = await fetch(request)

        if (!response.ok) return null

        return (await response.json()) ?? null
      },
    }),
  ],
})
```

## 5. 인증 여부에 따른 렌더링 분기 처리

### 5.1 서버 컴포넌트

```tsx
import { auth } from '@/auth'

export default async function UserAvatar() {
  const session = await auth()

  if (!session.user) return null

  return (
    <div>
      <img src={session.user.image} alt='User Avatar' />
    </div>
  )
}
```

### 5.2 클라이언트 컴포넌트

- 상위 컴포넌트에 `SessionProvider`로 감싸야 한다.
- `useSession` 훅으로 사용자 정보를 가져올 수 있다.

```tsx
import { useSession } from 'next-auth/react'
import { UserAvatar } from '@/components/UserAvatar'

export default function Dashboard() {
  const session = useSession()

  return (
    <nav>
      <UserAvatar session={session} />
    </nav>
  )
}
```

## 6. React-query SSR 설정

- fetch 관련 내용을 전역에서 사용하기 위해 `_lib` 폴더로 이동

### 6.1 설치하기

```bash
npm i @tanstack/react-query@5
npm i @tanstack/react-query-devtools@5 -D #devtools 설치
```

### 6.2 Provider 설정

```tsx
function RQProvider({ children }: Props) {
  const [client] = useState(
    new QueryClient({
      defaultOptions: {
        queries: {
          refetchOnWindowFocus: false,
          retryOnMount: true,
          refetchOnReconnect: false,
          retry: false,
        },
      },
    })
  )

  return (
    <QueryClientProvider client={client}>
      {children}
      {/* 로컬에서만 Devtool 사용 설정 */}
      <ReactQueryDevtools initialIsOpen={process.env.NEXT_PUBLIC_MODE === 'local'} />
    </QueryClientProvider>
  )
}
```

데이터를 사용할 컴포넌트 외부에 `RQProvider`를 감싼다.

### 6.3 Data Fetching

- Next에서는 기본적으로 fetch를 캐싱한다.
- 캐싱을 원하지 않을 경우, `cache: 'no-store'` 설정을 추가한다.

```ts
async function getPostRecommends() {
  const res = await fetch('http://localhost:9090/api/postRecommends', {
    next: {
      tags: ['posts', 'recommends'], // Query key?
    },
    cache: 'no-store',
  })

  if (!res.ok) {
    throw new Error('Failed to fetch data')
  }

  return res.json()
}
```

## 7. React Query 사용 이유와 상태값

### 7.1 React Query를 왜 쓰나요?

Redux, RTK, Redux-saga를 많이 쓰던 예전과 달리 요즘은 React Query가 지배적이다.

대표적인 차이점은 React query는 **서버에서 데이터를 가져오는 것**이고, Redux는 **컴포넌트 간에 데이터를 공유**하는 것이다.
또 React query는 데이터 캐싱이 편리하도록 설정되어 있고, Loading, Success, Failed 상태에 대한 인터페이스가 표준화되어 있다.

React query로 API 통신을 한다면 컴포넌트 간 상태 공유는 Context API나 Zustand를 사용하는 것이 좋다.

> Zustand는 지원 중단 되었으니 앞으로 Context API..?

### 7.2 React query 상태

- fresh: 최신 데이터
- stale: 서버에서 다시 데이터를 가져와야 하는 상태
  - refetchOnWindowFocus: 탭 이동 시 클릭할 때 발생
  - retryOnMount: 페이지 이동, 컴포넌트가 상태 때문에 잠깐 unmount 됐다가 mount 됐을 때.
  - refetchOnReconnect: 네트워크 연결이 끊겼다가 다시 연결됬을 때
  - retry
- staleTime: default 값은 0초. Infinity로도 설정 가능
- inactive: 화면에서 해당 데이터를 사용하지 않으면 inactive가 됨. cache랑 관계 없음.
- gcTime: inactive된 데이터를 cache memory에서 언제 제거할 지 결정하는 옵션
- fetching: 데이터 가져오는 중
- paused: 데이터를 가져오다가 일시정지

**항상 staleTime이 gcTime보다 짧아야 한다.**

### 7.3 refetch, invalidate, reset의 차이

**refetch**
데이터를 새로 가져와서 최신화한다.

**invalidate**
refetch와 유사하지만 특정 쿼리를 더 이상 쓰지 말라는 액션이다.
inactive 상태일 때는 데이터를 새로 가져오지 않는다. 그렇지만 `Observers`가 1이일 경우에는 서버에 데이터를 새로 요청한다.

**reset**
초기 데이터로 리셋된다. 처음에 설정된 초기 데이터가 없다면 데이터를 새로 가져온다.

```ts
const { data } = useQuery<IPost[]>({
  queryKey: ['posts', 'recommends'],
  queryFn: getPostRecommends,
  staleTime: 60 * 1000,
  initialData: () => {},
})
```

## 8. 팔로잉 게시글, 검색 결과 불러오기

### 8.1 key 설정

- `queryKey`는 문자열 뿐만 아니라 객체도 가능하다.
- 그러나 next tag 설정부에는 객체는 사용할 수 없다.

## 9. 조건부 쿼리 & 쿼리 재사용

### 9.1 조건부 쿼리

트렌드 페이지는 로그인했을 때만 데이터를 새로 가져오기 위해 `enabled` 옵션 설정을 해준다.

```tsx
export default function TrendSection() {
  const { data: session } = useSession();
  const { data } = useQuery<Hashtag[]>({
    queryKey: ["trends"],
    queryFn: getTrends,
    staleTime: 60 * 1000,
    gcTime: 300 * 1000,
    enabled: !!session?.user, // 세션에 유저 정보가 있을 때만 가져오기
  });
```

### 9.2 쿼리 재사용하기

`queryKey`가 같다면 동일한 데이터로 간주하여 다른 컴포넌트에섯 사용하는 경우에도
새로 불러 오지 않고 캐싱된 데이터를 재사용한다.

## 10. 에러 상황 처리

검색 페이지에 노출되어야 하는 경우 적용하면 좋다.

## 11. 게시글 상세 페이지, 답글, 포토모달

cache: `no-store`를 제거하면 tags를 update할 때 까지 사용.

## 12. 미흡한 부분 구현하기

`searchParams`는 Readonly이므로 newSearchParams 객체를 새로 생성해서, 해당 객체에 delete, set 메서드를 통해 추가 및 삭제한다.

```ts
const onChangeFollow = () => {
  const newSearchParams = new URLSearchParams(searchParams)
  newSearchParams.set('pf', 'on')
  //newSearchParams.delete("pf");
  router.replace(`/search?${newSearchParams.toString()}`)
}
```

## 13. 인피니트 스크롤링

```tsx
// Before
const { data } = useQuery<IPost[]>({
  queryKey: ['posts', 'recommends'],
  queryFn: getPostRecommends,
  staleTime: 60 * 1000, // fresh -> stale, 5분이라는 기준
  gcTime: 300 * 1000,
})

// After
const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetching,
  isPending,
  isLoading, // isPending && isFetching
  isError,
} = useInfiniteQuery<IPost[], Object, InfiniteData<IPost[]>, [_1: string, _2: string], number>({
  queryKey: ['posts', 'recommends'],
  queryFn: getPostRecommends,
  initialPageParam: 0, // 1,2,3,4,5 postId 불러옴 (다음 커서는 5)
  getNextPageParam: lastPage => lastPage.at(-1)?.postId,
  staleTime: 60 * 1000, // fresh -> stale, 5분이라는 기준
  gcTime: 300 * 1000,
})
```

## 14. react-intersection-observer로 불러오기

스크롤을 최하단으로 내렸을 때, 다음 페이지를 불러오는 함수를 호출해야 한다.

스크롤 최하단 감지하기

1. DOM의 스크롤 높이 계산 비교로 감지 (옛 발식..)
2. `intersection-observer` 이용

- 가상 태그를 밑에 두고, 그 태그에 옵저버를 걸어둔다.
- 화면에 보이지 않아도 닿으면 옵저버를 통해 이벤트를 호출한다.
  `fetchNextPage`로 페이지를 부른다.

### 14.1 intersection-observer 사용

`useInView` 훅을 통해 화면에 보이지 않는 ref가 감지될 경우 함수를 호출한다.

```tsx
const { data, fetchNextPage, hasNextPage, isFetching } = useInfiniteQuery<
  IPost[],
  Object,
  InfiniteData<IPost[]>,
  [_1: string, _2: string],
  number
>({
  queryKey: ['posts', 'recommends'],
  queryFn: getPostRecommends,
  initialPageParam: 0,
  getNextPageParam: lastPage => lastPage.at(-1)?.postId, // lastPage: 가장 최근에 불러왔던 페이지들
  staleTime: 60 * 1000, // fresh -> stale로 변환되는 시간(ms)
  gcTime: 300 * 1000,
})

const { ref, inView } = useInView({
  threshold: 0,
  delay: 0,
})

useEffect(() => {
  if (inView) {
    // 현재 데이터를 가져오고 있지 않고, 다음 페이지가 있다면
    // 다음 페이지를 패칭해
    !isFetching && hasNextPage && fetchNextPage()
  }
}, [inView, !isFetching, fetchNextPage, hasNextPage])
```

## 15. Suspense로 Streaming하여 최적화하기(feat. loading.tsx, error.tsx)

### 15.1 서버 컴포넌트

Next는 기본적으로 서버에서 페이지를 완성한 뒤 화면으로 보내 준다. 따라서 처음 페이지에 접근했을 때는 loading.tsx가 작동되지 않으나, 클라이언트에서 페이지를 이동하는 경우 작동한다.

### 15.2 클라이언트 컴포넌트

- isPending: 데이터를 아예 불러오지 않았을 때 true 반환 (구.isLoading)
- isFetching: 요청 내부의 비동기 함수 처리 여부.

초기 데이터를 불러오는 경우는 `isPending`  
데이터를 다시 가져와야 된다면 `isFetching`

### 15.3 직접 Suspense를 사용하면 좋은 이유

suspense를 사용한 부분만 lazy loading이 가능하다. (=Streaming 방식)

(lazy loading을 적용하면 초기 loadTime 감소)
