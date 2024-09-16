# Section 4. 백엔드 개발자에게 API를 받았다

## 1. 서버 세팅

- Node 설치
- PostgreSQL 설치
  - port: 5432
  - password 설정
- Redis 설치
  - window에서는 redis 지원이 되지 않아 memurai로 대신 설치
  - Mac: `brow i redis`
- [prisma repo](github.com/zerocho/nest-prisma) clone
- pgAdmin 실행
  1.  create - register server
  2.  General tab -> name: pg16 (아무거나)
  3.  `npx prisma migrate dev`

## 로그인과 회원가입 실제로 하기

`signup.ts`
회원가입 로직 설정에서 `credentials: 'include'` 작성을 해줘야 세션 쿠키가 브라우저에 전달된다.

주소를 rewrite 해야하는 경우, 다음과 같이 설정

```js
// next.config.js
const nextConfig = {
  async rewrites() {
    return [
      {
        source: '/upload/:slug', // source path를 가졌다면 destination으로 요청보냄
        destination: 'http://localhost:9090/upload/:slug',
      },
    ]
  },
}

module.exports = nextConfig
```

## 업로드 이미지 미리보기

`react-text-autosize`
textarea 사이즈 자동으로 증가시켜주는 라이브러리

새션 토큰: 프론트 로그인용
connect.sid 백엔드에 요청을 보내기 위한 토큰 (백엔드 로그인용)

jwt토큰 사용하는데, 그 토큰이 커넥트sid가 쿠키형태로 들어있다고 보면 된다

```ts
// auth.ts
let setCookie = authResponse.headers.get('Set-Cookie')
if (setCookie) {
  const parsed = cookie.parse(setCookie)
  cookies.set('connect.sid') // 브라우저에다가 쿠키를 심어주기
}
```

프론트 서버에는 쿠키를 심으면 개인정보 유출 문제가 발생할 수 있어서 절대 심으면 안된다

## useMutation 사용하기

게시글 업로드 시, optimistic update를 활용하면 바로 성공했다고 치고 화면에 올려준다.

## 주소에 해시가 들어가면 문제가 됩니다(encodeURIComponent)

주소에 특수문자 해시가 들어가면 서버에 전달이 되지 않는다.  
그러므로 인코딩을 해줘야 한다.

`encodeURIComponent('#hash')`

## 하트 누를 때 optimistic update 적용하기

팔로우, 좋아요 기능에 적용한다.
`immer` 라이브러리를 통해 불변성을 지키면서 값을 업데이트한다.

```ts
// immer 사용 O
shallow.pages[pageIndex][index].Hearts = [{ userId: sesson?.user?.email as string }]
shallow.pages[pageIndex][index]._count.Hearts += 1

// immer 사용 X
const pageIndex = value.pages.findIndex(page => page.includes(obj))
const index = value.pages[pageIndex].findIndex(v => v.postId === postId)
const shallow = { ...value }

value.pages = { ...value.pages }
value.pages[pageIndex] = [...value.pages[pageIndex]]
shallow.pages[pageIndex][index] = {
  ...shallow.pages[pageIndex][index],
  Hearts: shallow.pages[pageIndex][index].Hearts.filter(v => v.userId !== session?.user?.email),
  _count: {
    ...shallow.pages[pageIndex][index]._count,
    Hearts: shallow.pages[pageIndex][index]._count.Hearts - 1,
  },
}
```

## 서로 다른 컴포넌트간 query 일치하게 하기

추천에 표시된 사용자를 내가 팔로우한다면, 상대방에게도 싱크를 맞춰주는 작업이다.

## 완전히 로그아웃하기, 프론트서버에 쿠키보내기

### 캐시 초기화

`invalidateQueries` 를 사용하여 캐시를 초기화한다.

### 세션 초기화

fetch를 이용해 로그아웃 api를 호출하여 session을 초기화한다.

```ts
import { cookies } from 'next/headers'

headers: {
  Cookie: cookies().toString()
}
```

## 메타데이터 설정

기본적으로 서버 컴포넌트에서 실행된다.  
동적으로 메타 데이터를 생성해야 한다면 `generateMetadata`를 사용한다.

```tsx
// layout.tsx , page.tsx
export async function generateMetadata({searchParam}) {
    return {
        title: `${searchParam.q}`
        description: `${searchParam.q}`
    }
}
```

## SSR 적용 여부 판단 기준

- 페이지마다 새로고침을 통해 미리보기에서 정상 렌더링되는지 확인한다. SSR을 적용했을 때 이미지가 너무 크면 이미지 조절하고, 네트워크 탭의 미리보기를 통해서 SSR이 잘 되고 있는지 주기적인 확인이 필요하다.
- 로그인을 안해도 페이지에 접근할 수 있도록 하기 위해서 SSR 적용
- SSR의 경우 SSR하면 프론트 서버에 무리가 가므로 필요한 부분만 적용하는게 권장된다.

## ### 답글, 재게시글 (with. Zustand)

Parent, Original 필드에 맞는 값들을 확인해서 각 값들을 활용하여 필요한 내용을 화면에 표시할 수 있다.
다른 상태 라이브러리들과 비슷하게 setAction을 통해 불변성을 지키며 업데이트 가능한다.

```ts
import { Post } from '@/model/Post'
import { create } from 'zustand'

interface ModalState {
  mode: 'new' | 'comment'
  data: Post | null
  setMode(mode: 'new' | 'comment'): void
  setData(data: Post): void
  reset(): void
}

export const useModalStore = create<ModalState>((set, get) => ({
  mode: 'new',
  data: null,
  setMode(mode) {
    set({ mode })
  },
  setData(data) {
    set({ data })
  },
  reset() {
    set({
      mode: 'new',
      data: null,
    })
  },
}))
```
