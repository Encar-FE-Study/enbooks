### 새로 알게 된 것 혹은 스스로 정리하고 싶은 것 위주로 적어보았습니다

## 리액트에서 상태 - UI를 나타내는 모든 데이터

- 시간이 지남에 따라 바뀔 수 있는 대상
- 보통 함수별(컴포넌트별)로 상태를 다룸
- 이와 별개로 복잡하거나 구분해서 다뤄야 하는 상태들이 존재
  ex) 폼 / 서버 데이터 혹은 캐시 상태
- 이를 편하게 다루기 위한 것이 react hook form, react query
- 왜 이들을 분리해서 다루는 게 좋은가?

### 상태 관리를 위해 필요한 기능

    - 상태읽기
    - 상태의 갱신(업데이트)
    - 상태가 변경되면 리렌더링

### 리액트 훅의 등장

- 장점
- 로직 - UI의 분리가 가능
- 로직의 재사용 및 명확한 이름 부여
- 이를 통해 캡슐화(추상화)가 가능해짐

## 컴포넌트 / 훅

- 재사용이 잘 될 수록 좋다
- 그렇다면 같은 입력에 대해 같은 출력을 반환하는 순수함수여야 한다
- 그렇다면 전역상태에 의존할수록 출력에 대한 추적과 순수성 유지가 힘들 수 있다.따라서 최대한 전역상태(변수)에 의지하지 않는 것이 좋다

### bailOut

- setState 실행시, 같은 상태값에 대해서는 리렌더링을 하지 않는 것

```javascript
const Component = ()=>{
   const [state,setState] = useState({ count : 0});
   return (
       ...
       <button onClick={()=>{
           state.count = 1;
           setState(state);

           // state의 메모리 주소가 그대로기에 새로운 상태로 인식 X -> 리렌더 X
           // 리액트는 shallow compare를 통해 객체의 메모리주소를 비교하여 원시 / 객체 상태값을 비교
           // deep compare (객체의 값을 하나씩 순회하며 비교하는 것은 중첩객체일 경우 큰 비용을 발생시키기에)
           // 따라서 불변성을 유지해줘야
       }}>
   )
   ...
}
```

## useState - useReducer는 기본적으로 동일하며 서로 상호 교환 가능하다

### useReducer를 이용현 useState구현

```javascript

const useState = (initialState)=> {
    const [state,dispatch] = useReducer((prev, action) => typeof action === 'function' ? action(prev) :  action, initialState));

    return [state,dispatch];
}

더 단순화해보면

const reducer = (prev, action) => typeof action === 'function' ? action(prev) :  action, initialState);

const useState = (initialState) => useReducer(reducer, initialState);
즉, useState가 할 수 있는 일은 useReducer로 대체 가능하다

```

### useState를 이용한 useReducer 구현

```javascript
const useReducer = (reducer, initialState) => {
  const [state, setState] = useState(initialState);
  const dispatch = (action) => setState((prev) => reducer(prev, action));
  return [state, dispatch];
};
```
