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

## Jotai와 Recoil의 차이점

#### 1. 상태 선언
Recoil : 각 atom과 selector는 고유한 Key값이 필요하다.
```javascript
import { atom, selector } from 'recoil';

const countState = atom({
  key: 'countState', // 고유한 key 필요
  default: 0,
});

const doubleCountState = selector({
  key: 'doubleCountState',
  get: ({ get }) => get(countState) * 2,
});
```
Jotai : 상태 선언시 key값이 필요하지 않다. 이는 Jotai가 상태를 WeakMap을 통해 괸라하며, 상태 객체가 자동으로 고유하게 식별되기 때문이다.
```javascript
import { atom } from 'jotai';

const countAtom = atom(0);

const doubleCountAtom = atom((get) => get(countAtom) * 2);
```

#### 2. Provider 필요 여부
Recoil : RecoilRoot가 필요하다
```javascript
import { RecoilRoot } from 'recoil';

const App = () => (
    <RecoilRoot>
        <YourComponent />
    </RecoilRoot>
);
```
Jotai : 별도 Provider 없이 간단한 상태 공유 가능하다.

#### 3. 라이브러리 크기
Recoil은 2.2MB 이고, Jotai는 421KB 이다 보니 Jotai가 Recoil보다 약 4배정도 더 가벼운 라이브러리이다.

## Zustand, Jotai, Valtio 비교

- 규모가 있는 프로젝트에서는 Zustand를, 
- 개인 프로젝트 또는 규모가 작은 프로젝트에서는 Jotai를 사용하고,
- Valtio의 Proxy객체를 사용하여 상태관리하는 방안은 참신하지만, Proxy 객체만 사용해볼것같고
- React Tracked는 Context API의 최적화를 위해서 사용해볼것 같다.

### 혹시 누군가 이렇게 물어본다면,
### React에서 상태 관리가 복잡해졌을 때, 어떤 방식을 사용해 상태 관리를 단순화하겠습니까?
- 이 책을 읽고 나서 할수 있는 말
> React에서 상태 관리가 복잡해졌을 때,
> 저는 애플리케이션의 요구사항에 따라 데이터 중심 접근(예: Redux, Zustand)과 컴포넌트 중심 접근(예: Jotai, Context API)을 적절히 혼합해 사용할 것입니다. 
> 또한, 선택자 함수, 속성 접근 감지, 아톰과 같은 기법을 활용해 리렌더링을 최소화하겠습니다. 
> 소규모 프로젝트에서는 Jotai와 Context API를 사용해 경량 상태 관리를, 
> 대규모 프로젝트에서는 Redux와 Zustand와 같은 강력한 도구를 선택해 체계적으로 관리할 것입니다. 
> 전역 상태와 지역 상태를 명확히 분리하고, 간단한 상태는 useState와 useReducer를 통해 효과적으로 처리하겠습니다. 
> 무엇보다 상황에 맞는 최적화 도구를 활용해 상태 관리를 단순화하는 데 집중해보겠습니다.
