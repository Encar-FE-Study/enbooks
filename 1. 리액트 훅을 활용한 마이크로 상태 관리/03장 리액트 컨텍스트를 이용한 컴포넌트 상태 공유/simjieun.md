# 03. 리액트 컨텍스트를 이용한 컴포넌트 상태 공유
- 리액트 16.3 버전부터 컨텍스트(Context)라는 기능을 제공하기 시작함
  - useContext와 useState를 이용하면 전역 상태를 위한 사용자 정의 훅을 만들 수 있다.

## useState와 useContext 탐구하기
### useContext 없이 useState 사용하기
- 상태끌어올리기 패턴으로 부모 컴포넌트에서 state를 정의하여 props로 전달하는 방식으로 구현
- 부모 컴포넌트에서 자식 컴포넌트로 props를 전달하는 작업을 **프로퍼티 내리꽂기(prop drilling)** 이라 한다.
- 애플리케이션의 규모가 커진다면 트리 아래로 props를 전달하는 것이 적절하지 않게 된다. -> 불필요한 props를 전달하게 되므로

### 정적 값을 이용해 useContext 사용하기
- 컨텍스트를 사용하면 props를 사용하지 않고도 부모 컴포넌트에서 트리 아래에 있는 자식 컴포넌트로 값을 전달하는 것이 가능하다.
- 리액트 컨텍스트에는 값을 제공하는 공급자(provider)와 컨텍스트를 소비하는 소비자 컴포넌트(useContext가 있는 컴포넌트)가 존재한다.
- 컴포넌트 트리 중에서 가장 가까운 공급자를 선택해 컨텍스트 값을 가져온다.

```javascript
const ColorContext = createContext('black');

const Component = () => {
  const color = useContext(ColorContext);
  return <div style={{ color }}>Hello {color}</div>;
};

const App = () => (
  <>
    <Component /> // Hello black
    <ColorContext.Provider value="red">
      <Component /> // Hello red
    </ColorContext.Provider>
    <ColorContext.Provider value="green">
      <Component /> // Hello green
    </ColorContext.Provider>
    <ColorContext.Provider value="blue">
      <Component /> // Hello blue
      <ColorContext.Provider value="skyblue">
        <Component /> // Hello skyblue
      </ColorContext.Provider>
    </ColorContext.Provider>
  </>
);
```
### useContext와 함께 useState 사용하기
- 컨텍스트에서 상태와 갱신 함수를 전달할 수 있다.

```javascript
const CountStateContext = createContext({ count: 0, setCount: (_: SetStateAction<number>) => {} });

const App = () => {
    const [count, setCount] = useState(0);
    return (
        <CountStateContext.Provider value={{ count, setCount }}>
            <Parent />
        </CountStateContext.Provider>
    );
};

const Parent = () => (
    <>
        <Component1 />
        <Component2 />
    </>
);

const Component1 = () => {
    const { count, setCount } = useContext(CountStateContext);
    return (
        <div>
            {count}
            <button onClick={() => setCount((c) => c + 1)}>+1</button>
        </div>
    );
};

const Component2 = () => {
    const { count, setCount } = useContext(CountStateContext);
    return (
        <div>
            {count}
            <button onClick={() => setCount((c) => c + 2)}>+2</button>
        </div>
    );
};

```

## 컨텍스트 이해하기
- 컨텍스트 전파의 작동 방식과 한계에 대해서 알아보자

### 컨텍스트 전파의 작동 방식
- 컨텍스트 공급자가 새로운 컨텍스트 값을 받으면 모든 컨텍스트 소비자 컴포넌트가 리렌더링 된다.
> 리액트에서 리렌더링이 이뤄질때
> 1. useState로 상태가 변경될 때
> 2. useReducer로 dispatch가 실행될 때 
> 3. Key, Props가 변경될 때
> 4. 부모컴포넌트가 렌더링 될 때
> 5. Context 값이 변경될 때
