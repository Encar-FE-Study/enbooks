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

- codeSandBox 링크 : https://codesandbox.io/p/sandbox/cm3hk3?file=%2Fsrc%2FApp.tsx%3A49%2C31
- 컨텍스트가 변경되었기 때문에 리렌더링이 이뤄지고, memo는 props 변경을 기준으로만 리렌더링을 방지할 뿐, context 변경에 대해서는 방지하지 않는다는걸 알수 있다.

### 컨텍스트에 객체를 사용할 때의 한계점
- 컨텍스트 값에 원시값이 아닌 객체값을 사용할때에는 불필요한 리렌더링이 발생할수 있다.
- 객체의 하나의 키만 변경하고 다른 키를 변경하지 않았을때, 다른키를 구독하는 컴포넌트가 리렌더링이 된다.

## 전역 상태를 위한 컨텍스트 만들기
- 리액트 컨텍스트의 동작 방식을 기반으로 전역 상태와 함께 컨텍스트를 사용하는 것과 관련해서 두가지 해결책을 살펴보자
- codeSandBox 링크 : https://codesandbox.io/p/sandbox/cm3hk3?file=%2Fsrc%2FApp.tsx%3A42%2C25

### 작은 상태 조각 만들기
- 첫번째 해결책은 전역 상태를 여러 조각으로 나누는 것이다.
- 합쳐진 큰 객체를 사용하는 대신 각 조각에 대한 컨텍스트와 전역 상태를 만든다.

### useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기
- 두번째 해결책은 단일 상태를 만들고 여러 컨텍스트를 사용해 상태 조각을 배포하는 것이다.
- 이 경우 상태를 갱신하는 함수를 배포하는것은 별도의 컨텍스트로 해야 한다.
- useReducer를 기반으로 했을때, 상태 조각을 위한 컨텍스트와 디스패치(dispatch) 함수를 위한 컨텍스트가 존재한다.

> 불필요한 리렌더링을 피하기 위해 여러 컨텍스트를 사용하는 것이다.

## 컨텍스트 사용을 위한 모범 사례

### 사용자 정의 훅과 공급자 컴포넌트 만들기
- 사용자 정의 훅과 공급자 컴포넌트를 생성한다
- 기본 컨텍스트 값을 null로 만들고 사용자 정의 훅에서 값이 null인지 확인한다

```javascript
type CountContextType = [number, Dispatch<SetStateAction<number>>];

const Count1Context = createContext<CountContextType | null>(null);
export const Count1Provider = ({ children }: { children: ReactNode }) => (
  <Count1Context.Provider value={useState(0)}>
    {children}
  </Count1Context.Provider>
);
export const useCount1 = () => {
  const value = useContext(Count1Context);
  if (value === null) throw new Error("Provider missing");
  return value;
};
```
### 사용자 정의 훅이 있는 팩토리 패턴
- 사용자 정의 훅과 공급자 컴포넌트를 만드는 것은 다소 반복적인 작업이니, 이런 작업을 수행하는 팩토리 패턴으로 컨텍스트를 구현해본다.
- 생성 시 초깃값을 정의하는 대신 런타임에 상태의 초깃값을 설정할 수 있다는 장점이 있다.
- createStateContext의 핵심은 반복적인 코드를 피하면서 같은 기능을 제공하는 것이다.
```javascript
import { ReactNode, createContext, useContext, useState } from "react";

const createStateContext = <Value, State>(
  useValue: (init?: Value) => State
) => {
  const StateContext = createContext<State | null>(null);
  const StateProvider = ({
    initialValue,
    children,
  }: {
    initialValue?: Value;
    children?: ReactNode;
  }) => (
    <StateContext.Provider value={useValue(initialValue)}>
      {children}
    </StateContext.Provider>
  );
  const useContextState = () => {
    const value = useContext(StateContext);
    if (value === null) throw new Error("Provider missing");
    return value;
  };
  return [StateProvider, useContextState] as const;
};

const useNumberState = (init?: number) => useState(init || 0);

const [Count1Provider, useCount1] = createStateContext(useNumberState);  
```
### reduceRight를 이용한 공급자 중첩 방지
- Provider의 중첩이 많을때 코딩 스타일을 완화하기 위해 **reduceRight**를 사용할 수 있다.
```javascript
const App = () => {
  const providers = [
    [Count1Provider, { initialValue: 10 }],
    [Count2Provider, { initialValue: 20 }],
    [Count3Provider, { initialValue: 30 }],
    [Count4Provider, { initialValue: 40 }],
    [Count5Provider, { initialValue: 50 }],
  ] as const;
  return providers.reduceRight(
    (children, [Component, props]) => createElement(Component, props, children),
    <Parent />
  );
};
```
