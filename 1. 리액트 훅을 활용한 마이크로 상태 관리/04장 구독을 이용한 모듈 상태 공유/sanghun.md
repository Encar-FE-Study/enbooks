# 04. 구독을 이용한 모듈 상태 공유

- 컨텍스트는 싱글턴 패턴을 위해 설계된 것이 아니라 싱글턴 패턴을 피하고 각 하위 트리에 서로 다른 값을 제공하기 위한 기능이다.
- 전역상태를 싱글턴 패턴으로 만들고 싶다면 모듈 상태를 사용하는 것이 좋다.
  > 모듈상태란, ECMAScript(ES) 모듈 스코프에 정의된 상수 또는 변수 이다.

```typescript
const createContainer = (initialState) => {
  let state = initialState;
  const getState = () => state;
  const setState = (nextState) => {
    state = typeof nextState === 'function' ? nextState(state) : nextState;
  };
  return { getState, setState };
};
```

## 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법

- 컴포넌트가 추가될 수록 중복코드가 계속 늘어나 좋은 해결책이 아님

## 기초적인 구독 추가하기

```typescript
export type Store<T> = {
  getState: () => T;
  setState: (action: T | ((prev: T) => T)) => void;
  subscribe: (callback: () => void) => () => void;
};

export const createStore = <T extends unknown>(initialState: T): Store<T> => {
  let state = initialState;
  const getState = () => state;
  const callbacks = new Set<() => void>();
  const setState = (nextState: T | ((prev: T) => T)) => {
    state =
      typeof nextState === 'function'
        ? (nextState as (prev: T) => T)(state)
        : nextState;

    callbacks.forEach((callback) => callback());
  };
  const subscribe = (callback: () => void) => {
    callbacks.add(callback);
    return () => {
      callbacks.delete(callback);
    };
  };

  return { getState, setState, subscribe };
};
```

- useSubscription 훅을 이용해서 좀더 공식적인 방법 사용 가능
