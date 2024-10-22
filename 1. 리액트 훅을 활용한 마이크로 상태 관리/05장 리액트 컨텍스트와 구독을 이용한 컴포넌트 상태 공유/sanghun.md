# 05. 리액트 컨텍스트와 구독을 이용한 컴포넌트 상태 공유

- 컨텍스트의 장점과 구독의 장점을 모두 합친 상태관리 방법은 규모가 큰 애플리케이션에 유용함

#### 모듈상태의 한계

```javascript
const store = createStore({ count: 0 });
const Counter = () => {
  const [state, setState] = useStore(store);
  const inc = () => {
    setState((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));
  };

  return (
    <div>
      {state.count} <button onClick={inc}>+1</button>
    </div>
  );
};

const store2 = createStroe({ count: 0 });
const Counter2 = () => {
  const [state, setState] = useStore(store2);
  ...
};
```

- 두개의 컴포넌트에서 유일하게 다른점은 store, store2의 참조인데 Counter컴포넌트는 모듈 상태를 사용하기 때문에 재사용이 불가능하다. 이것이 모듈 상태의 한계이다.
- 일반적으로 모듈상태는 컴포넌트 외부에 정의되기 때문에 코드는 동일하고 store만 다르게 참조하는 n개의 컴포넌트의 경우 서로 재사용할 수 없다.
- ContextAPI의 <Provider> 패턴을 활용하여 하나의 컴포넌트를 재사용하면서 상위트리에서 value로 store를 넘겨주면 하나의 컴포넌트로 재사용이 가능함
- 하지만 컴포넌트의 depth가 2단계정도로 낮을 경우 props로 전달하는것이 성능, 유지보수 측면에서 높다고 판단됨 (개인의견)

## 컨텍스트 사용이 필요한 시점

- 컨텍스트는 서로 다른 하위 트리에 서로 다른 값을 제공해야 하는 경우 필요함

## 컨텍스트 구독 패턴 사용하기

```typescript
type State = { count: number; text?: string };

export const StoreContext = createContext<Store<State>>(
  createStore<State>({ count: 0, text: 'hello' })
);

export const StoreProvider = ({
  initialState,
  children,
}: {
  initialState: State;
  children: ReactNode;
}) => {
  const storeRef = useRef<Store<State>>();
  if (!storeRef.current) {
    storeRef.current = createStore(initialState);
  }

  return (
    <StoreContext.Provider value={storeRef.current}>
      {children}
    </StoreContext.Provider>
  );
};
```

- 이 패턴의 핵심은 useContext와 useSubscription을 사용하는 것임
- 컨텍스트로 하위 트리에서 상태를 분리해서 제공할 수 있고, 구독을 통해 리렌더링을 피할 수 있어서 좋은 패턴임
- 결국 **컴포넌트의 불필요한 리렌더링**을 피하기 위해 다양한 패턴을 사용해서 상태를 관리하는 것임
> zustand도 동일하게 createContext를 활용하여 하위 트리에 상태를 분리해서 제공 가능 + selector를 활용한 구독으로 불필요한 리렌더링 감소
> 뒤에 결국 다 나오는 내용임