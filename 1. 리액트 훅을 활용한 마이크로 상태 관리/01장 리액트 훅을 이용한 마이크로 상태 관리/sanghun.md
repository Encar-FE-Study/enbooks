# 01. 리액트 훅을 이용한 마이크로 상태 관리

## 전통적인 상태관리

- 상태 관리는 리액트 애플리케이션 개발에서 중요한 문제 중 하나이다.
- 리액트의 전통적인 상태 관리는 **중앙 집중적인 방식**으로, 상태 관리를 위한 범용 프레임워크를 사용해 개발자가 문제를 해결하는 방식이었다.

## 리액트 훅의 등장

**리액트 훅**(React Hook)의 등장 이후, 리액트 훅을 통해 상태 관리를 더 **경량화**하고 **마이크로화**할 수 있게 되었다.

- 기본 훅을 사용해 **재사용 가능**한 기능을 만들 수 있다.
- 전통적인 상태 관리는 **범용적**이지만, 마이크로 상태 관리는 더 **목적 지향적**이다.

## 전역 상태 관리의 어려움

리액트 훅은 **지역 상태** 관리에 탁월한 기능을 제공하지만, **전역 상태 관리**는 리액트에서 다루기 까다로우며, 전역 상태 관리는 **커뮤니티와 생태계**의 도움을 받아야 한다. (Zustand, Jotai, Recoil, Redux 등)

## 마이크로 상태관리 이해하기

- 리액트에서 상태는 UI를 나타내는 모든 데이터를 뜻한다.
- 리액트 훅 이전에는 중앙 집중형 상태 관리 라이브러리를 사용하는 것이 일번적이었지만, 필요없는 기능까지 포함될 수 있어 오버스펙의 단점도 있었다.
- 애플리케이션의 특성에 따라 범용적인 상태관리가 다르게 사용되며, 그렇기 때문에 범용적인 상태관리를 위한 방법은 가벼워야 한다. 이를 가리켜 **마이크로 상태관리**라고 한다.

## 리액트 훅 사용하기

- 리액트 훅의 장점은 UI컴포넌트에서 로직을 추출할 수 있다는 것이다.
- 마이크로 상태관리 관점에서 하나의 기능단위별로 hook을 분리하는 것은 중요하다.

```javascript
const useCount = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('count is changed to', count);
  }, []);

  return [count, setCount];
};
```

- count와 관련된 기능들은 컴포넌트를 건드리지 않고 useCount 훅안에서만 추가, 수정하면 된다.

  #### 데이터 불러오기를 위한 서스펜스와 동시성 렌더링

  - 서스펜스(Suspense)는 기본적으로 비동기 처리에 대한 걱정 없이 컴포넌트를 코딩할 수 있는 방법
  - 동시성 렌더링은 렌더링 프로세스를 청크(chunk)단위로 분할하여 CPU가 장시간 차단되는 것을 방지하는 방법
  - 기존 state 객체나 ref 객체를 직접 변경할 경우 예기치 못한 렌더링 이슈가 발생할 수 있음

## 전역 상태 탐구하기

- 리액트에서 컴포넌트에서 정의되고 컴포넌트 트리 내에서 사용되는 상태를 지역상태라고 한다.
- 애플리케이션 내 서로 떨어져 있는 여러 컴포넌트에서 사용하는 상태는 전역상태로 부르며, 공유상태라 부르기도 한다.
- 리액트는 컴포넌트 모델에 기반하며, 컴포넌트 모델에서는 **지역성**(locality)이 중요하다. 이는 컴포넌트가 서로 격리돼야 하고 재사용이 가능해야 한다는 것을 의미한다.

## useState 사용하기

#### 값으로 상태 갱신하기

```javascript
const Componenet = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      {count}
      <button onClick={() => setCount(1)}>Set Count to 1</button>
    </div>
  );
};
```

- 버튼을 두번 클릭하면 setCount가 동일한 값을 호출하기 때문에 '베일아웃'되어 컴포넌트가 렌더링 되지 않는다.
- **베일아웃**이란 리렌더링을 발생키지 않는것을 의미힌다.

```javascript
...
  <button onClick={() => setState({count: 1})}>
    Set Count to 1
  </button>
...
```

- 버튼의 동작을 위와같이 바꾸면 버튼을 2번이상 클릭해도 항상 리렌더링이 발생한다. 왜냐하면 버튼이 클릭될 때마다 항상 새로운 객체를 만들기 때문이다.

#### 함수로 상태 갱신하기

```javascript
...
  <button onClick={() => setCount((c) => c+1)}>
    Increment Count
  </button>
...
```

- 위와 같이 onClick함수를 변경하면 실제로 버튼을 클릭한 횟수를 센다.
- 이전 값을 기반으로 갱신할 경우 유용하다.

#### 지연 초기화

```javascript
  const init = () => 0;

  const Component = () => {
    const [count, setCount] = useState(init);
    ...
  }
```

- init함수가 무거운 계산을 포함할 수 있는 경우 느리게 평가되는 위의 초기화 방법이 유용하다.
- **느리게 평가**란 지연 평가라고도 하며, 필요한 시점에 계산되는 것을 말한다.

## useReducer 사용하기

#### 베일아웃

```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_TEXT':
      if (!action.text) {
        // 베일 아웃
        return state;
      }
      return { ...state, text: action.text };
    ...
  }
};
```

- action.text가 비어있을 때 새로운 객체를 반환하지 않고 state자체를 반환해서 베일아웃 되도록 처리가 가능하다.

## useState와 useReducer의 유사점과 차이점

#### useReducer를 이용한 useState 구현

```javascript
const useState = (initialState) => {
  const [state, dispatch] = useReducer(
    (prev, action) => (typeof action === 'function' ? action(prev) : action),
    initialState
  );
  return [state, dispatch];
};
```

- 단순화

```javascript
const reducer = (prev, action) =>
  typeof action === 'function' ? action(prev) : action;
const useState = (initialState) => useReducer(reducer, initialState);
```

#### 인라인 리듀서 사용하기

- 인라인 리듀서 함수는 외부변수에 의존할 수 있다.
- 일반적인 기능은 아니며 권장하지 않는다.
