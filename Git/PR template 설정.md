# PR template 설정

## 하나의 템플릿만 사용하는 경우
- `/.github` 루트 경로에 `PULL_REQUEST_TEMPLATE.md` 파일을 생성해서 작성한다.

## 여러 템플릿을 사용하는 경우
- `/.github/PULL_REQUEST_TEMPLATE` 경로에 템플릿 별로 마크다운 문서를 추가한다.
  - 이 때 파일명은 단순한게 좋다. 
- default template은 루트 경로에 있는 템플릿이 설정된다.
- 다른 템플릿으로 변경하고 싶다면 url parameter 조작이 필요하다.
  - dropdown UI로 선택하는 gitlab과 다르게 번거롭다..
  
### 사용 가이드 
`/.github/PULL_REQUEST_TEMPLATE/bug.md` 를 사용하려 한다면

https://git.com/frontend/project/...어쩌고?`template=bug.md`

template 쿼리 파라미터를 이용해 불러올 수 있다.

수동으로 타이핑해줘야 해서 템플릿 파일명은 간단한게 좋다.

---

참고
[Github Docs - Using query parameters to create a pull request](https://docs.github.com/ko/enterprise-server@3.7/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/using-query-parameters-to-create-a-pull-request)
