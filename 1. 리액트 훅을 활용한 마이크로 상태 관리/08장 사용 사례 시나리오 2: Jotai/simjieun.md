# 08장. 사용 사례 시나리오 2: Jotai

> [Jotai](https://github.com/pmndrs/jotai)는 전역 상태를 위한 작은 라이브러이다.
> useState, useReducer와 함께 상태의 작은 조각인 아톰이라고 불리는 것을 모델로 삼는다.

## Jotai 이해하기

### Jotai 문법은 단순하다.
#### 1. 컨텍스트를 사용한 예제
```javascript
const CountContext = createContext(
  (undefined as unknown) as [number, Dispatch<SetStateAction<number>>]
);

const CountProvider = ({ children }: { children: ReactNode }) => (
  <CountContext.Provider value={useState(0)}>{children}</CountContext.Provider>
);

const Counter1 = () => {
    const [count, setCount] = useContext(CountContext);
    const inc = () => setCount((c) => c + 1);
    return (
        <>
            {count} <button onClick={inc}>+1</button>
        </>
    );
};
const App = () => (
    <CountProvider>
        <div>
            <Counter1 />
        </div>
    </CountProvider>
);

export default App;
```

#### 2. Jotai를 사용한 예제
```javascript
import { atom, useAtom } from "jotai";

const countAtom = atom(0);

const Counter1 = () => {
    const [count, setCount] = useAtom(countAtom);
    const inc = () => setCount((c) => c + 1);
    return (
        <>
            {count} <button onClick={inc}>+1</button>
        </>
    );
};
const App = () => (
    <div>
        <Counter1 />
    </div>
);

export default App;
```

### 동적 아톰 생성
- Jotai의 스토어는 아톰 구성 객체와 아톰 값으로 이루어진 WeakMap이다. 
- 아톰 구성 객체는 atom 함수로 생성되며, 아톰 값은 useAtom 훅이 반환한다. 
- useAtom 훅은 스토어의 특정 아톰을 구독하며, 구독 기반이므로 불필요한 리렌더링을 방지할 수 있다.

## 렌더링 최적화
## Jotai가 아톰 값을 저장하는 방식 이해하기
## 배열 구조 추가하기
## Jotai의 다양한 기능 사용하기
