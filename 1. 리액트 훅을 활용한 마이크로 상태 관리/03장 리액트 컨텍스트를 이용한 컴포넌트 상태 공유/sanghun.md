# 03. 리액트 컨텍스트를 이용한 컴포넌트 상태 공유

- 리액트가 제공해주는 컨텍스트(Context)기능은 컴포넌트 상태와 결합하여 전역 상태로 사용할 수 있다.
- 하지만 컨텍스틑 전역상태를 위해 설계된 것이 아니라 상태가 갱신될 때 모든 컨텍스트 소비자가 리렌더링된다는 이슈가 있다.

## useState와 useContext 탐구하기

#### useContext없이 useState 사용하기

```javascript
const App = () => {
  const [count, setCount] = useState(0);

  return (
    <Parent count={count} setCount={setCount}> />
  )
};
const Parent = ({count, setCount}) => {
  ...
}
const Component1 = ({count, setCount}) => {
  ...
}
const Component2 = ({count, setCount}) => {
  ...
}
```

- 위와 같은 구조로 App컴포넌트에서 상태를 정의하고 prop drilling을 통해서 Component1, Component2로 상태를 전달한다면 중간에있는 Parent컴포넌트는 불필요하게 상태를 가지게 된다.

#### 정적 값을 이용해 usContext 사용하기

- 리액트 컨텍스트를 사용하면 props를 사용하지 않고도 컴포넌트 트리 아래에 있는 자식 컴포넌트로 값을 전달하는 것이 가능하다.

```javascript
const ColorContext = createContext('black');

const Component = () => {
  const color = useContext(ColorContext);
  return <div style={{ color }}>{color}</div>;
};

const App = () => {
  return (
    <>
      <Component />
      <ColorContext.Provider value="skyblue">
        <Component />
      </ColorContext.Provider>
      <ColorContext.Provider value="red">
        <Component />
      </ColorContext.Provider>
      <ColorContext.Provider value="blue">
        <Component />
      </ColorContext.Provider>
    </Component>
  )
}
```

- 첫번째 컴포넌트는 Provider가 없어서 기본값인 black을 보여주고, 나머지 컴포넌트들은 각각 Provider로 제공받은 값에따라 다른 색상이 보여진다.

#### useContext와 함께 useState 사용하기

```javascript
const CountStateContext = createContext({
  count: 0,
  setCount: () => {},
});

const App = () => {
  const [count, setCount] = useState(0);
  return (
    <CountStateContext.Provider value={{ count, setCount }}>
      <Parent />
    </CountStateContext.Provider>
  );
};
```

- 위와같이 전역 Contenxt를 하나 만들면 Parent컴포넌트는 따로 상태를 가지고 있지 않아도 된다.
- Component1, Component2에서 useContext를 통해서 Provider로 제공된 값들을 가지고 올 수 있다.

## 컨텍스트 이해하기

- 컨텍스트 공급자가 새로운 컨텍스트 값을 갖게 되면 모든 컨텍스트 소비자는 리렌더링된다. 이는 공급자의 값이 모든 소비자에게 전파된다는 것을 의미한다.

#### 컨텍스트 전파의 작동 방식

- memo를 사용해서 prop이 변경되지 않으면 리렌더링 되지 않는 컴포넌트를 만들 수는 있지만, 내부 컨텍스트 소비자가 리렌더링되는 것을 막지는 못한다.

#### 컨텍스트에 객체를 사용할 때의 한계점

- 컨텍스트에 객체를 사용할 경우 특정 컴포넌트에서 사용하지 않는 값이 변경이 되어도 해당 컴포넌트는 리렌더링이 되는 이슈가 있다.
- 예를들어 Component1에서는 count1관련된 값만 사용을 하고 Component2에서 count2를 사용할 경우 Component1에서는 count1관련된 값이 변경됐을 경우에만 리렌더링이 되어야하는데, count2가 변경된 경우에도 리렌더링이 발생한다.
  > 추가 리렌더링을 방지하는 것은 기술적으로 필해야할 연산이지만, 사용자가 추가 리렌더링을 느끼지 못하는 경우 성능에 크게 문제가 없다면 오히려 오버엔지니어링을 하지 않는 것이 좋다.

## 전역 상태를 위한 컨텍스트 만들기

- 리액트 컨텍스트의 동작 방식을 기반으로 전역 상태와 함께 컨텍스트를 사용하는 것과 관련해서 두가지 해결책이 있다.

1. 작은 상태 조각 만들기
2. useReducer로 하나의 상태를 만들고 여러 컨텍스트로 전파하기

#### 작은 상태조각 만들기

- 합쳐진 큰 객체를 사용하는 대신 각 조각에 대한 컨텍스트와 전역 상태를 만든다.

```typescript
type CountContextType = [number, Dispatch<SetStateAction<number>>];

const CountContext1 = createContext<CountContextType>([0, () => {}]);
const CountContext2 = createContext<CountContextType>([0, () => {}]);

const Counter1 = () => {
  const [count1, setCount1] = useContext(CountContext1);
  ...
}

const Counter2 = () => {
  const [count2, setCount2] = useContext(CountContext2);
  ...
}
```

> 이 방식을 사용하면 컴포넌트 내에서 useState를 사용하는 방식보다 어떤 장정이 있지?

```typescript
const CountProvider1 = ({ children: { children: ReactNode } }) => {
  const [count1, setCount1] = useState(0);
  return (
    <CountContext1.Provider value={[count1, setCount1]}>
      {children}
    </CountContext1.Provider>
  );
};

const CountProvider2 = ({ children: { children: ReactNode } }) => {
  const [count2, setCount2] = useState(0);
  return (
    <CountContext2.Provider value={[count2, setCount2]}>
      {children}
    </CountContext2.Provider>
  );
};

const App = () => {
  <CountProvider1>
    <CountProvider2>
      <Parent />
    </CountProvider2>
  </CountProvider1>;
};
```

- 위와 같은 방식으로 컴포넌트를 작성하면 불필요한 리렌더링을 줄일 수 있다.

#### useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기

- 두번째 해결책으로 단일 상태를 만들고 여러 컨텍스트를 사용해 상태 조각을 배포하는 방법이 있다. 이 경우에는 상태를 갱신하는 함수를 별도의 컨텍스트로 만들어야 한다.

```typescript
type Action = { type: 'INC1' } | { type: 'INC2' };

const CountContext1 = createContext<number>(0);
const CountContext2 = createContext<number>(0);
// 별도의 상태갱신 컨텍스트
const DispachContext = createContext<Dispatch<Action>>(() => {});

const Counter1 = () => {
  const count1 = useContext(CountContext1);
  const dispatch = useContext(DispatchContext);

  return (
    <div>
      Count: {count1}
      <button type="button" onClick={() => dispatch({ type: 'INC1' })} />
    </div>
  );
};

const Counter2 = () => {
  const count2 = useContext(CountContext2);
  const dispatch = useContext(DispatchContext);

  ...
}
```

- 위와 같이 Counter1, Counter2 컴포넌트에서 상태를 가져오는 컨텍스트만 달리하고 상태를 갱신하는 컴포넌트는 같은것을 사용하여 사용할 수 있다.

```typescript
const Provider = ({children: {children: ReactNode}}) => {
  const [state, dispatch] = useReducer((prev: {count1: number, count2: number}, action: Action) => {
    if (action.type === 'INC1') {
      return {...prev, count1: count1 + 1};
    }
    if (action.type === 'INC2') [
      return {...prev, count2: count2: + 1};
    ]
    throw new Error('no matching action!');
  },{ count1: 0, count2: 0});

  return(
    <DispatchContext.Provider value={dispatch}>
      <CountContext1.Provider value={state.count1}>
        <CountContext2.Provide value={state.count2}>
          {children}
        </CountContext1.Provider>
      </CountContext2.Provider>
    </DispatchContext.Provider>
  )
}

const App = () => {
  return (
    <Provider>
      <Parent />
    </Provider>
  )
}
```

- useReducer로 인해서 코드가 길어졌지만 App컴포넌트는 깔끔하게 정리가 된다.
- 여러 상태컨텍스트를 사용하는 것보다 단일 상태를 사용할 때의 장점은 하나의 액션으로 여러 조각을 갱신할 수 있다는 것이다.

```typescript
  ...
  if (action.type === 'INC_BOTH') {
    return {
      ...prev,
      count1: prev.count1 + 1,
      count2: prev.count2 + 1
    }
  }
  ...
```

- 위와 같은 방식으로 하나의 액션만 추가하면 두개의 조각을 갱신할 수 있다.

## 컨텍스트 사용을 위한 모범 사례

#### 사용자 정의 훅과 공급자 컴포넌트 만들기

- Context를 hook으로 감싸서 컴포넌트에서 사용하게 하는 방법이며 각각 컴포넌트는 hook내부에서 사용되는 숨겨진 컨텍스트에 대해서는 알지 못한다. (캡슐화)

#### 사용자정의 훅이 있는 팩토리 패턴

- 위에서 사용한 방식은 다소 반복적인 작업이며, 이 작업을 대신수행해주는 팩토리 패턴을 사용할 수 있다.
