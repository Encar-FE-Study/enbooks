# 03 리액트 컨텍스트를 이용한 컴포넌트 상태 공유

## useState와 useContext 탐구하기
- useContext는 props drilling의 제거하기 위한 수단으로 사용

## Context 이해하기
- Context 부모가 새로운 Context 값을 받으면 모든 Context 자식 컴포넌트 리렌더링
(원인 - 부모 컴포넌트, 컨텍스트)
- Context 값 변경되지 않음에도 리렌더링 발생 시 -> 내용 끌어올리기 혹은 memo 사용
- 객체의 일부 값만 사용하는 경우에도 리렌더링 발생

## 전역 상태를 위한 컨텍스트 만들기
- 작은 상태 조각 만들기
Provider 중첩 완화 필요

## useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기
reducer의 상태를 각 Context에 전파

## Context 사용을 위한 모범 사례
- 사용자 정의 훅과 공급자 컴포넌트 만들기
```js
<CountContext.Provider value={useState(0)}/>
```
- 사용자 정의 훅이 있는 팩토리 패턴
```js
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
const [Count2Provider, useCount2] = createStateContext(useNumberState);  
```


- reduceRight를 이용한 공급자 중첩 방지
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight
```js
const App = () => (
    <Count1Provider>
        <Count2Provider>
            <Count3Provider>
                <Parent>
            </Count3Provider>
        </Count2Provider>
    </Count1Provider>
)
```
```js
const App = () => {
  const providers = [
    [Count1Provider, { initialValue: 10 }],
    [Count2Provider, { initialValue: 20 }],
    [Count3Provider, { initialValue: 30 }],
  ] as const;
  return providers.reduceRight(
    (children, [Component, props]) => createElement(Component, props, children),
    <Parent />
  );
};
```

