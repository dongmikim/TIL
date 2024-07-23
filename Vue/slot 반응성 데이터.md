
```vue
// Parent Component
// current는 날짜 데이터
<TextButton>
  {{ current.format('YYYY') }}
</TextButton>
<IconButton @click={addMonth} />


// Child Component
<TextButton>
 //...
 <slot />
</TextButton>
```

- 문제: slot 컨텐츠 데이터가 변경되었지만 리렌더링 되지 않음
- 원인: slot 내부에서 변경되는 객체나 배열 속성의 값을 Vue에서 감지하지 못함 
(props로 전달할 경우에는 정상 리렌더링)
- 해결: 재할당
    ```vue
    addMonth (add) {
      const cloneCurrent = this.current.clone().add(add, 'months')
      this.current = cloneCurrent
      // ...
    },
    ```
