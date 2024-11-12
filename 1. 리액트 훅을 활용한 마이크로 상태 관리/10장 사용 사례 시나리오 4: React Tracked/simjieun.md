# 10장. 사용 사례 시나리오 3: React Tracked

## React Tracked 이해하기
- React Tracked는 상태 관리 기능을 제공하지는 않지만 렌더링 최적화 기능을 제공한다. 이 기능을 **상태 사용 추적** 이라고 한다.
- 내부에서 상태 사용을 추적하고 렌더링을 자동으로 최적화한다.
- Valtio와 React Tracked 두 라이브러리는 동일하게 proxy-compare라는 내부 라이브러리를 사용한다.

## useState, useReducer와 함께 React Tracked 사용하기
- React Tracked는 주로 리액트 컨텍스트를 대체할 용도로 사용된다.

### useState와 함께 React Tracked 사용하기
- createContext 대신에 createContainer를 사용하고 useStateContext 대신에 useTracked를 사용한다.
- 그결과 리렌더링 최적화가 가능해졌다. 이것이 바로 상태 사용 추적 기능이다.

```javascript
import { useState } from "react";
import { createContainer } from "react-tracked";

const useValue = () => useState({ count: 0, text: "hello" });

const { Provider, useTracked } = createContainer(useValue);

const Counter = () => {
  const [state, setState] = useTracked();
  const inc = () => {
    setState((prev) => ({ ...prev, count: prev.count + 1 }));
  };
  return (
    <div>
      count: {state.count} <button onClick={inc}>+1</button>
    </div>
  );
};

const TextBox = () => {
  const [state, setState] = useTracked();
  const setText = (text: string) => {
    setState((prev) => ({ ...prev, text }));
  };
  return (
    <div>
      <input value={state.text} onChange={(e) => setText(e.target.value)} />
    </div>
  );
};

const App = () => (
  <Provider>
    <div>
      <Counter />
      <Counter />
      <TextBox />
      <TextBox />
    </div>
  </Provider>
);

export default App;
```

### useReducer와 함께 React Tracked 사용하기
- React Tracked가 리렌더링을 최적화할 수 있는 이유는 상태 사용 추적뿐만 아니라 use-context-selector 라이브러리를 사용해서이다.
- 이 라이브러리의 selector 함수는 사용해 컨텍스트 값을 구독할 수 있다.

### React Redux와 함께 React Tracked 사용하기

## 향후 전망
- React Tracked 구현은 두개의 내부 라이브러리에 의존한다
- proxy-compare : https://github.com/dai-shi/proxy-compare
- use-context-selector : https://github.com/dai-shi/use-context-selector
- useContextSelector ?? : React에서 제공하는 훅인데, 이걸 모방해서 만든게 use-context-selector 라이브러리라고 한다.

## 정리
- React Tracked는 상태관리 라이브러리보다는 useState, useReducer와 Redux와 함께 리렌더링 최적화를 하기 위한 기능을 가진 라이브러리인것같다.

