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

#### Jotai에서 왜 WeakMap 객체를 사용 했을까?
```javascript
let object = { name: 'Garbage' };
let collector = [object];
object = null; // 참조를 null로 덮어씀

console.log(JSON.stringify(collector[0])); // {"name":"Garbage"}

let obj2 = { name: 'Trash' };
let weakMap = new WeakMap();

weakMap.set(obj2, 'die');
obj2 = null; // 참조를 덮어씀

console.log(typeof weakMap.get(obj2)); // undefined
```
- 위의 예제를 보면 배열은 각 키와 각 값에 대한 참조가 무기한 유지되도록 보장하기 때문인데, 이 때문에 다른 곳에서 객체를 참조하지 않더라도 키가 가비지 컬렉션 대상이 되지 못하지만
- WeakMap은 해당 객체에 대한 약한 참조만 가지기 때문에, 다른 곳에서 객체를 참조하지 않으면 객체는 메모리에서 해제된다.
- obj2는 이미 null로 덮어써졌고, 이전에 저장된 객체는 가비지 컬렉션으로 제거되었기 때문에 undefined가 반환된다.

##### WeakMap 사용의 장점
1. 메모리 관리: 객체가 더 이상 필요 없을 때 자동으로 메모리에서 해제되어, 메모리 누수(memory leak)를 방지한다.
2. 자동 메모리 해제: 참조가 끊기면 가비지 컬렉션이 발생하여 메모리를 효율적으로 관리할 수 있다.

#### 결론, 이런 사유로 Jotai에서 WeakMap을 사용한게 아닐까 싶다. [Jotai WeakMap을 이용한 Store 소스링크](https://github.com/pmndrs/jotai/blob/main/src/vanilla/store.ts)

## 렌더링 최적화
- 아톰을 원시 값처럼 원하는 만큼 작게 만들어서 리렌더링을 제어할 수 있다.

1. 하향식(top-down) 접근법
- 아래의 store와 선택자 접근하는 방식을 하향식 접근법이라고 한다.
```javascript
const personStore = createStore({
    firstName: "React",
    lassName: "Hooks",
    age:3,
})

const selectFirstName = (state) => state.firstName;
const selectLastName = (state) => state.lastName;

const Person = () => {
    const firtstName = useStoreSelector(store, selectFirstName);
    const lastName = useStoreSelector(store, selectLastName);
    return <>{firtstName} {lastName}</>;
}
```
2. 상향식(bottom-up) 접근법
- 아래 처럼 작은 아톰을 만들고 이를 결합해서 더 큰 아톰을 만드는 방식을 상향식 접근법이라 할 수 있다.
```javascript
const firstNameAtom = atom("React");
const lastNameAtom = atom("Hooks");
const ageAtom = atom(3);

const fullNameAtom = atom((get) => ({
    firstName: get(firstNameAtom),
    lastName: get(lastNameAtom),
}));

const Person = () => {
    const person = useAtom(fullNameAtom);
    return <>{person.firstName} {person.lastName}</>
}
```
## Jotai가 아톰 값을 저장하는 방식 이해하기
- 원시 아톰은 useState처럼 동작하도록 설계 되어있다.
- 아톰 값을 저장하는 store가 따로 있다. store에는 키가 아톰 구성 객체이고 값이 아톰 값이 WeakMap객체가 있다.
- 
## 배열 구조 추가하기
- 아톰 속 아톰들(Atoms-in-Atom) 이라고 부르는 새로운 패턴을 이용하여 리렌더링을 최적화 할수 있다.
- 해당 내용은 프로젝트로 https://github.com/simjieun/client-store-project/tree/main/jotai

## Jotai의 다양한 기능 사용하기
### 아톰의 write 함수 정의하기
```javascript
const countAtom = atom(0);

const doubledCountAtom = atom(
    (get) => get(countAtom) * 2,
    (get, set, arg) => set(countAtom, arg/2)
);
```
- get은 아톰의 값을 반환하는 함수다.
- set은 아톰의 값을 설정하는 함수다
- arg는 아톰을 갱신할 때 받을 임의의 값이다.(이 경우에는 doubledCountAtom을 말한다)

### 액션 아톰 사용하기
```javascript
const countAtom = atom(0);

const incrementCountAtom = atom(
    null, 
    (get, set, arg) => set(countAtom, (c) => c + 1)
);
```

### 아톰의 onMount 옵션 이해하기
- 아톰이 사용되기 시작할 때 특정 로직을 실행하고 싶을때는 onMount 옵션을 사용할 수 있다.
```javascript
const countAtom = atom(0);
countAtom.onMount = (setCount) => {
    console.log("count atom 사용을 시작합니다.");
    const onUnmount = () => {
        console.log("count atom 사용이 끝났습니다.")
    }
    return onUnmount;
}
```
