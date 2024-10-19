# 07장. 사용 사례 시나리오 1: Zustand

> [Zustand](https://github.com/pmndrs/zustand)는 주로 리액트의 모듈 상태를 생성하도록 설계된 작은 라이브러이다.
> Zustand는 24.10.15 일자 기준으로 어제(24.10.14) 5버전이 나왔다.
> 
> [Zustand를 이용한 개인 프로젝트](https://github.com/simjieun/client-store-project/tree/main/zustand)

## 모듈 상태와 불변 상태 이해하기
- Zustand는 상태를 유지하는 store를 만드는 데 사용되는 라이브러리이다.
- Zustand는 주로 모듈 상태를 위해 설계됐으며 모듈에서 store를 정의하고 내보내는 것을 할 수 있다
- 상태 객체 속성을 갱신할 수 없는 불변 상태 모델을 기반으로 한다.
  - 상태를 변경하기 위해서는 새 객체를 생성해서 대체해야 하며, 수정하지 않은 객체는 재사용해야 한다.
- 불변 상태 모델의 장점은 상태 객체의 참조에 대한 동등성만 확인하면 변경 여부를 알 수 있으므로 객체의 값 전체를 확인할 필요가 없다.

```javascript
import create from "zustand";

export const store = create(() => ({ count: 0 }));

console.log(store.getState()); // { count: 0 }
store.setState({ count: 1 });
console.log(store.getState()); // { count: 1 }

const state1 = store.getState();
state1.count = 2; // 잘못됨
store.setState(state1); // 갱신되지 않음

store.setState((prev) => ({ count : prev.count + 1 })); // 함수갱신이라고하며, 이전상태를 이용해 상태를 변경할수있다.
```

```javascript
import create from "zustand";

export const store = create(() => ({
  count: 0,
  text: "hello",
}))

console.log(store.getState()); // { count: 0, text: 'hello' }
store.setState({
  count: 2,
})
console.log(store.getState()); // { count: 2, text: 'hello' } 이는 내부적으로 Object.assign()으로 구현된다.

store.subscribe(()=> {
    console.log('store state is changed');
})
store.setState({ count: 3 });
```
- store에 객체로 상태를 저장시에 count 속성만 가지고 사용하는 컴포넌트랑 text를 가지고 사용하는 컴포넌트에서 count만 갱신하면 text를 구독하는 컴포넌트는 리렌더링이 이뤄지지 않는가? (이부분 테스트필요)
- 

## 리렌더링 최적화를 위한 리액트 훅 추가하기
- 리액트에서 store를 사용하려면 사용자 정의 훅이 필요하다.
- Zustand의 create함수는 훅으로 사용할 수 있는 store를 생성한다. store 대신 useStore로 지정한다.

```javascript
import create from "zustand";

export const useStore = create(() => {
    count: 0,
    text: "hello",
}));

// 해당 컴포넌트는 store 상태가 변경될 때마다 리렌더링 된다. 관련 없는 text값을 변경하더라도 리렌더링이 된다.
const Component = () => {
    const { count, text } = useStore();
    return (
        <div>
            <p>{count}</p>
        </div>
    );
}

// 리렌더링을 피하기 위해서는 선택자 함수를 사용한다.
// 이 같은 선택자 기반 리렌더링 제어를 수동 렌더링 최적화라고 한다.
const Component = () => {
    const count = useStore((state) => state.count);
    return (
        <div>
            <p>{count}</p>
        </div>
    );
}

// 선택자함수가 새객체를 포함해 새로운 배열을 생성하기 때문에 
// count값이 변경되지 않은 경우에도 컴포넌트가 리렌더링된다.
const Component = () => {
    const [{ count }] = useStore((state) => [{ count: state.count }]);
    return (
        <div>
            <p>{count}</p>
        </div>
    );
}
```
- 선택자 기반 렌더링 최적화의 장점 : 동작을 정확히 예측할 수 있다
- 선택자 기반 렌더링 최적화의 단점 : 객체 참조에 대한 이해가 필요하다.

## 읽기 상태와 갱신 상태 사용하기

```javascript
import create from "zustand";

type StoreState = {
  count1: number;
  count2: number;
  inc1: () => void;
  inc2: () => void;
};

const useStore = create<StoreState>((set) => ({
  count1: 0,
  count2: 0,
  inc1: () => set((prev) => ({ count1: prev.count1 + 1 })),
  inc2: () => set((prev) => ({ count2: prev.count2 + 1 })),
}));

const selectCount1 = (state: StoreState) => state.count1;
const selectInc1 = (state: StoreState) => state.inc1;

const Counter1 = () => {
    const count1 = useStore(selectCount1);
    const inc1 = useStore(selectInc1);
    return (
        <div>
            count1: {count1} <button onClick={inc1}>+1</button>
        </div>
    );
};
```
## 구조화된 데이터 처리하기

## 이 접근 방식과 라이브러리의 장단점
- Zustand의 읽기 및 쓰기 상태는 다음과 같다.
  - 읽기 상태 : 리렌더링을 최적화하기 위해 선택자 함수를 사용한다
  - 쓰기 상태 : 불편 상태 모델을 기반으로 한다.
- 핵심은 리액트가 최적화를 위해 객체 불변성을 기반으로 하는데, Zustand도 역시 불변성을 기반으로 한다.
- Zustand는 리액트와 동일한 모델을 사용해 라이브러리의 단순성과 번들 크기가 작다는 점에 큰 이점이 있다.
- 반면, 선택자를 이용한 수동 렌더링 최적화를 해야하며, 객체 참조 동등성을 이해해야 하며, 선택자 코드를 위해 보일러플레이트 코드를 많이 작성해야 할 필요가 있다.

