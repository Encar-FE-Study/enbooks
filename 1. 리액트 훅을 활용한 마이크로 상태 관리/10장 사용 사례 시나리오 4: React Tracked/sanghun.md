# 10장. 사용 사례 시나리오 3: React Tracked

- React Tracked는 **속성감지**를 기반으로 자동으로 렌더링 최적화를 수행하는 상태사용 추적 라이브러리다.
- React Tracked는 상태관리 기능을 제공하지 않고 렌더링 최적화 기능을 제공함
  - 이 기능을 **상태 사용 추적**이라고 함
- Valtio와 React-tracked는 동일하게 내부적으로 **proxy-compare**라는 라이브러리를 사용해서 상태추적을 추적해 리렌더링을 최적화 시킴

## useState, useReducer + React Tracked

- Context API -> React Tracked로 교체 시 리렌더링 발생.. (최적화 안됨)
  https://github.com/sangsang-6914/micro_state_management/blob/main/react-tracked/src/context/StateContext.tsx
- useTrackedState, useUpdate로 분리해도 리렌더링 발생
  - 구글링 정보가 없음..
  - 1.7.14v 도 안됨

## useReducer + React Tracked

- React Tracked가 리렌더링을 최적화할 수 있는 이유는 상태 사용 추적뿐 아니라 use-context-selector라는 내부 라이브러리 덕분임
- useReducer 사용해서 구현해도 리렌더링 최적화 안됨..
  https://github.com/sangsang-6914/micro_state_management/blob/main/react-tracked/src/context/StateContext2.tsx

## React Redux + React Tracked

- 이 조합으로 사용하면 리렌더링 최적화가 잘됨..
  https://github.com/sangsang-6914/micro_state_management/blob/main/react-tracked/src/context/StateContext3.tsx

## 향후 전망

- 향후 리액트에서 useContextSelector 또는 유사한 형태의 구현체가 등장할 가능성이 있는데, 이경우 쉽게 마이그레이션이 가능함

## 정리

- React Tracked는 전역상태 라이브러리는 아니며, useState, useReducer 혹은 Redux와 함께 사용해야함
- 자동 리렌더링 최적화 기능을 추가한다고 생각하면 됨 (근데 잘 안되는듯?)
