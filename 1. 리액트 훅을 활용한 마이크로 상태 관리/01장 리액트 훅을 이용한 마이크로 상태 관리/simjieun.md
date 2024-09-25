# 01. 리액트 훅을 이용한 마이크로 상태 관리

> 상태 관리는 리액트 애플리케이션을 개발할 때 중요한 문제 중 하나이다. 리액트 훅이 등장한 이후로 상태 관리를 경량화, 마이크로화 할 수 있게 되었다.

# 마이크로 상태 관리 이해하기
- 리액트에서 상태는 사용자 인터페이스(UI)를 나타내는 모든 데이터를 말한다.
- 상태는 시간이 지남에 따라 변할 수 있으며 리액트는 상태와 함께 렌더링할 컴포넌트를 처리한다.
- 범용적인 상태 관리를 위한 방법은 가벼워야 하며, 개발자는 요구사항에 따라 적절한 방법을 선택할 수 있어야한다. 이를 가리켜 **마이크로 상태 관리**라고 한다.
- 리액트의 가벼운 상태 관리라고 할 수 있으며, 각 상태 관리 방법마다 서로 다른 기능을 가지며, 개발자의 애플리케이션 요구사항에 따라 적합한 방법을 선택할 수 있다.
- 마이크로 상태 관리의 필수적인 기능 :
  - 상태 읽기
  - 상태 갱신
  - 상태 기반 렌더링

# 리액트 훅 사용하기
- useState : 지역 상태를 생성하는 기본적인 함수로, 로직을 캡슐화하고 재사용 가능하다는 리액트 훅의 특징이 있다.
- useReducer : 지역 상태를 생성 할 수 있으며, useState를 대체하는 용도로 자주 사용된다.
- useEffect : 리액트 렌더링 프로세스 바깥에서 로직을 실행할 수 있다. 리액트 컴포넌트 생명주기와 함께 작동하는 기능을 구현할수있다.

    ### 데이터 불러오기를 위한 서스펜스와 동시성 렌더링
    - 데이터 불러오기를 위한 서스펜스 : 기본적으로 비동기 처리에 대한 걱정 없이 컴포넌트를 코딩할 수 있는 방법이다.
    - 동시성 렌더링 : 렌더링 프로세스를 청크 단위로 분할해서 중앙 처리 장치(CPU)가 장시간 차단되는것을 방지하는 방법이다.
    - 순수해야 하는 규칙

# 전역 상태 탐구하기
- 전역 상태는 애플리케이션 내 서로 멀리 떨어져 있는 여러 컴포넌트에서 사용하는 상태를 말한다.
- 전역 상태가 싱글턴일 필요는 없으며, 싱글턴이 아니라는 점을 명확히 하기 위해 전역 상태를 공유 상태라 부르기도 한다.

# useState 사용하기
> useState의 기본 사용법부터 고급 사용법까지 알아본다.

## 값으로 상태 갱신하기
- useState로 상태 값을 갱신하는 한가지 방법은 새로운 값을 제공하는 것이다.
```javascript
const Component = () => {
  const [count, setCount] = useState(0);
  return (
          <div>
            {count}
            <button onClick={() => setCount(1)}>Set Count to 1</button>
          </div>
  );
};
```
- **베일아웃** : 리렌더링을 발생시키지 않는 것

## 함수로 상태 갱신하기
```javascript
const Component = () => {
  const [count, setCount] = useState(0);
  return (
          <div>
            {count}
            <button onClick={() => setCount((c) => c + 1)}>
              Increment Count
            </button>
          </div>
  );
};
```

## 지연 초기화
- useState가 호출되기 전까지 init함수는 평가되지 않고 느리게 평가된다. 즉, 컴포넌트가 마운트될때 한번만 호출된다.
```javascript
const init = () => 0;

const Component = () => {
  const [count, setCount] = useState(init);
  return (
          <div>
            {count}
            <button onClick={() => setCount((c) => c + 1)}>
              Increment Count
            </button>
          </div>
  );
};
```

# useReducer 사용하기
## 기본 사용법
```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_TEXT':
      return { ...state, text: action.text };
    default:
      throw new Error('unknown action type');
  }
};

const Component = () => {
  const [state, dispatch] = useReducer(
          reducer,
          { count: 0, text: 'hi' },
  );
  return (
          <div>
            {state.count}
            <button onClick={() => dispatch({ type: 'INCREMENT' })}>
              Increment count
            </button>
            <input value={state.text} onChange={(e) => dispatch({ type: 'SET_TEXT', text: e.target.value })} />
          </div>
  );
};
```
## 베일아웃
```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_TEXT':
      if (!action.text) {
        // bail out
        return state
      }
      return { ...state, text: action.text };
    default:
      throw new Error('unknown action type');
  }
};
```
## 원시값
- useReducer는 객체가 아닌 값, 즉 숫자나 문자열 같은 원시 값에 대해 작동한다.
```javascript
const Component = () => {
  const [isOpen, toggleOpen] = useReducer((state) => !state, false);
  return (
          <div>
            <button onClick={toggleOpen}>
              열기
            </button>
            {isOpen ? "open" : "close"}
          </div>
  );
};
```
## 지연초기화
- init 함수는 컴포넌트가 마운트될 때 한번만 호출되므로 무거운 연산을 포함할 수 있다. 
- useState와 달리 init함수는 useReducer의 두번째 인수인 initialArg를 받는다.
```javascript
const init = (count) => ({ count, text: 'hi' });

const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_TEXT':
      return { ...state, text: action.text };
    default:
      throw new Error('unknown action type');
  }
};

const Component = () => {
  const [state, dispatch] = useReducer(reducer, 0, init);
  return (
          <div>
            {state.count}
            <button
                    onClick={() => dispatch({ type: 'INCREMENT' })}
            >
              Increment count
            </button>
            <input
                    value={state.text}
                    onChange={(e) =>
                            dispatch({ type: 'SET_TEXT', text: e.target.value })}
            />
          </div>
  );
};
```

# useState와 useReducer의 유사점과 차이점

## useReducer를 이용한 useState 구현
## useState를 이용한 useReducer 구현
## 초기화 함수 사용하기
## 인라인 리듀서 사용하기

