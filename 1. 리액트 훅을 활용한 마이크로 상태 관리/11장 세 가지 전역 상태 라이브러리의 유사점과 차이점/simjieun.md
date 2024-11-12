# 11장. 세 가지 전역 상태 라이브러리 유사점과 차이점

## Zustand와 Redux의 차이점

#### 1. 리듀서 사용 여부
Redux: 리듀서를 사용하여 상태를 갱신해야 합니다.
Zustand: 리듀서를 사용하지 않고 상태와 업데이트 로직을 직접 정의할 수 있습니다.

Redux 예제:

```javascript
// Redux 리듀서 사용 예제
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    default:
      return state;
  }
};
```
Zustand 예제:

```javascript
// Zustand에서는 리듀서 없이 상태와 액션을 직접 정의할 수 있습니다.
import create from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));
```

#### 2. Store 설정 필요 여부
Redux: 스토어 설정이 필요합니다.
Zustand: 별도의 스토어 설정이 필요하지 않습니다. useStore로 상태를 바로 생성하고 사용할 수 있습니다.
Redux 예제:

```javascript
import { createStore } from 'redux';
const store = createStore(counterReducer); // Redux 스토어 설정
```

Zustand 예제:

```javascript
// Zustand에서는 create 함수를 사용하여 바로 스토어를 생성할 수 있습니다.
import create from 'zustand';
const useStore = create((set) => ({ count: 0 }));
```

#### 3. Provider 필요 여부
Redux: Provider가 필요하며, 컴포넌트를 Provider로 감싸서 상태를 전파합니다.
Zustand: Provider 없이 상태를 전파할 수 있습니다.
Redux 예제:

```javascript
import { Provider } from 'react-redux';
import store from './store';

function App() {
  return (
    <Provider store={store}>
      <MyComponent />
    </Provider>
  );
}
```
Zustand 예제:

```javascript
// Zustand에서는 Provider 없이 useStore를 바로 사용하여 상태에 접근할 수 있습니다.
function App() {
  return <MyComponent />;
}
```

#### 4. 훅 사용 방식
Redux: useSelector와 useDispatch 훅을 사용합니다.
Zustand: useStore 훅을 사용하여 상태와 액션을 직접 가져옵니다.

Redux 예제:

```javascript
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state) => state.count);
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch({ type: 'INCREMENT' })}>
      Count: {count}
    </button>
  );
}
```
Zustand 예제:

```javascript
import { useStore } from './store';

function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);

  return (
    <button onClick={increment}>
      Count: {count}
    </button>
  );
}
```
#### 5. Immer 사용 여부
Redux: Immer가 기본적으로 포함되어 있어, 불변성 관리를 쉽게 할 수 있습니다.

Zustand: Immer는 선택 사항으로, 필요 시 추가하여 사용할 수 있습니다.
#### 6. 상태 전파 방식
Redux: 컨텍스트(Context)를 사용하여 상태를 전파합니다.

Zustand: 모듈 임포트를 통해 상태를 전파합니다.
