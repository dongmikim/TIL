# Section 2. UI 클론코딩

## Layout 클론

**요구사항. 좌우 section의 너비는 다르고, 가운데 정렬이 되야 한다.**

- 두 개의 flex 아이템에 각각 flex-grow: 1을 주면 컨테이너의 남은 여백도 나눠 갖는다.
- `position: fixed`인 요소에 `width: 100%`를 주면 부모의 너비 100%가 계산되지 않을 수도 있다. 이 경우, `width: inherit`으로 부모의 너비를 상속하는 편이 낫다.

## useSelectedLayoutSegment로 ActiveLink 만들기

**요구사항. 클릭된 탭 메뉴가 다른 스타일로 표시되도록 한다.**

- Active Link 현재 표시 중인 링크의 아이콘이 강조 표시되는 것이다. 즉, 현재 페이지의 주소를 파악한 뒤 이에 따라 스타일링을 다르게 하면 된다. 따라서 클라이언트 컴포넌트를 선언할 필요가 있다.
- Next.js에서는 위 기능을 제공한다.
  - useSelectedLayoutSegment(): 첫 번째 path를 반환
  - useSelectedLayoutSegments(): 모든 path를 배열로 반환

> **컴포넌트 분리**  
> 컴포넌트 분리 여부를 결정할 때, 서버 컴포넌트인지 클라이언트 컴포넌트인지를
> 고려하면 좋다.
> 상태 변화가 있거나 이벤트가 많을 경우 클라이언트 컴포넌트로 설정하도록 한다.
> 그러나 서버 컴포넌트로 설정하고, 이벤트만 별도의 컴포넌트로 분리할 수도 있다.

## 홈탭 만들면서 Context API 적용해보기

**요구사항1. 스크롤을 내리면, 상단 탭은 고정되고 blur 처리된다.**

- `backdrop-filter: blur` 설정을 통해 배경을 블러 처리할 수 있다.

**요구사항2. 추천, 팔로우 탭이 존재하고 클릭할 때마다 탭 별 데이터를 렌더링해야 한다.**

- 클라이언트 컴포넌트로 설정하고 useContext를 사용한다.

## /compose/tweet 만들기

**요구사항1. 게시하기 버튼을 클릭하면 트위터 작성 모달이 표시되어야 한다.**

**요구사항2. 홈 화면을 유지한 상태로 표시되고, 새로고침을 해도 그대로 모달이 표시되어야 한다.**

- Parallel Route가 필요하며 이를 위해 메인 레이아웃 컴포넌트에 modal 자리를 만들어 놔야 한다.
- /compose/tweet의 기본 레이아웃은 Home 화면을 띄워놓고 그 위에 Modal을 구성한다.

## usePathname과 /explore 페이지

**요구사항1. 탐색하기 페이지에서는 '나를 위한 트렌드' UI가 표시되지 않아야 한다. 팔로우 추천 UI만 표시**

**요구사항2. 검색했을 경우, 오른쪽에 검색 필터 UI 표시**

### usePathname

도메인 뒤의 path(/ 뒤부터 ? 앞까지)를 반환한다.
서버 컴포넌트는 자식으로 클라이언트 컴포넌트를 가질 때, children이나 props로 전달한다.

```tsx
// Parent.tsx
export default function Parent() {
  return <Child>// ...</Child>
}

// Child.tsx
export default function Child({ children }: { children: ReactNode }) {
  return <div>{children}</div>
}
```

## useSearchParams와 프로필, /search 페이지

### searchParams

searchParams는 `URLSearchParams` 인스턴스가 아닌 자바스크립트 객체를 반환한다.

| URL           | `searchParams`       |
| ------------- | -------------------- |
| /shop?a=1     | `{ a: '1' }`         |
| /shop?a=1&b=2 | `{ a: '1', b: '2' }` |
| /shop?a=1&a=2 | `a: ['1', '2']`      |

- Page 컴포넌트는 `params`, `searchParams`가 optional default props이다.
- 하위 컴포넌트에서 searchParams 전달을 받아야 한다면
  1.  props로 전달받아 사용
  2.  클라이언트 컴포넌트라면 `useSearchParams` hook 사용
      - useSearchParams.get() 메서드를 사용하면 searchParams 중 key에 해당하는 value를 가져올 수 있다.
      - useSearchParams.toString() 메서드를 사용하면 현재 searchParams를 그대로 가져온다. 새로운 param을 추가할 때 사용하기 좋다.

## 이벤트 캡처링

동일한 태그 하위에 클릭 이벤트가 여러개 있을 경우, 이벤트가 충돌이 난다.
이를 해결하기 위해 이벤트 캡처링을 사용한다.

```tsx
<article onClickCapture={onClick}>{children}</article>
```

## faker

- 패키지 @faker-js/faker
- 50% 확률 만들기: `Math.random() > 0.5`

## /messages

**요구사항1. 주고 받은 쪽지에 대한 UI(프로필, 내용, 날짜) 표시**

**요구사항2. 메시지를 클릭하면 상세 화면으로 이동**

- 서버에서 join해서 보내주는 필드는 대문자로 시작한다.
- `array.at(index)`로 마지막 요소를 가져오기 편해졌다.

## 반응형 만들기

- breakpoint 설계 필요
- breakpoint는 변수로 관리허여 유지보수성 증가

## 💡 IDE Tip

리팩토링해서 import 경로의 폴더 구조가 자주 변경될 때 사용하면 좋은 IDE 기능이다.
import 경로를 자동으로 변경해준다.

- Webstorm: 단축키 F6
- VScode: 우클릭 > refactor > move
