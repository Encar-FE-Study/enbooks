# 08장. 사용 사례 시나리오 2: Jotai

- Jotai는 useState, useReducer와 함께 상태의 작은 조각인 아톰이라고 불리는 것을 모델로 삼는다.
- Zustand와 달리 컴포넌트 상태를 사용하며, **불변 상태 모델**이다.
  > 아톰은 원자라고 하며 최소 단위를 나타낸다.

## Jotai 이해하기

- 일반 컨텍스트 사용

```typescript
import { createContext, ReactNode, useContext, useState } from 'react';

type MyContextType = [number, React.Dispatch<React.SetStateAction<number>>];

const CounterContext = createContext<MyContextType | undefined>(undefined);

export const CounterProvider = ({ children }: { children: ReactNode }) => {
  return (
    <CounterContext.Provider value={useState(0)}>
      {children}
    </CounterContext.Provider>
  );
};

export const useCounterContext = () => {
  const context = useContext(CounterContext);
  if (!context) {
    throw new Error('no find context!');
  }

  return context;
};

function App() {
  return (
    <CounterProvider>
      <Counter1 />
      <Counter2 />
    </CounterProvider>
  );
}

function Counter1() {
  const [count, setCount] = useCounterContext();
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
    </div>
  );
}

... Counter2 동일
```

- Jotai 사용

```typescript
export const countAtom = atom(0);

function Counter3() {
  const [count, setCount] = useAtom(countAtom);
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
    </div>
  );
}
```

- atom으로 상태를 등록하고 useAtom으로 쉽게 사용할 수 있다.
- 컨텍스트를 사용해 전역상태를 하나 만드는 것은 공급자도 추가해야하고 보일러플레이트 코드가 많아진다.
- 하지만 jotai를 사용하면 atom만 간단히 추가하면 되기 때문에 문법 단순함의 이점이 있다.

## 렌더링 최적화

```typescript
export const firstNameAtom = atom('Woo');
export const lastNameAtom = atom('Sang hun');
export const ageAtom = atom(30);

function Person() {
  const [firstName] = useAtom(firstNameAtom);
  const [lastNa] = useAtom(lastNameAtom);
  return (
    <div>
      {firstName} + {lastNa}
    </div>
  );
}
```

- Jotai도 각 상태를 atom으로 분리해서 렌더링 최적화를 할 수 있다. 다만 상태가 여러개일 경우 조작해야할 아톰이 많아져 불편할 수 있기에 **파생 아톰**이라는 개념이 있다.

```typescript
export const personAtom = atom((get) => ({
  firstName: get(firstNameAtom),
  lastName: get(lastNameAtom),
  age: get(ageAtom),
}));

function Person() {
  const [person] = useAtom(personAtom);
  return (
    <div>
      {person.firstName} {person.lastName}
    </div>
  );
}
```

- 위와같이 파생아톰으로 객체를 만들어서 사용하면 편하지만, ageAtom이 변경되었을 때도 불필요한 리렌더링이 발생한다.
- 이를 피하기위해 사용하는 아톰만 결합하여 파생아톰을 별도로 만들어 관리해야하며, 이를 **상향식 접근법**이라고 한다.

```typescript
export const fullNameAtom = atom((get) => ({
  firstName: get(firstNameAtom),
  lastName: get(lastNameAtom),
}));
```

## Jotai가 아톰값을 저장하는 방식 이해하기

- 아톰값을 저장하는 store가 별도로 존재하며, 키가 아톰 구성객체이고 값이 아톰 값인 **WeakMap** 객체가 있다.
  > **메모리 효율성**: WeakMap에 저장된 아톰은, 더 이상 참조되지 않을 때 자동으로 메모리에서 해제되므로 메모리 누수를 방지
  > **불필요한 데이터 제거**: 특정 컴포넌트가 언마운트될 때 관련된 아톰도 더 이상 사용되지 않으면 가비지 컬렉터가 자동으로 제거
  > **안정성**: WeakMap은 객체 키를 이용하여 다양한 상태를 분리 관리하므로, 아톰 간 충돌 없이 안전하게 상태를 보관
- jotai는 아톰을 결합한 방식인 파생아톰이 핵심인데 그 파생아톰이 많아져도 효율적으로 메모리를 관리할 수 있게된다.
- Provider를 사용하면 jotai도 다른 store를 사용하게 할 수 있다. atom이 값을 가지고 있지 않기 때문에 같은 atom이라도 Provider에 따라 재사용이 가능하다.

## 배열구조 처리하기

```typescript
const [, setTodos] = useSetAtom(todosAtom);
```

- 상태갱신만 하고싶을 경우 useSetAtom을 사용
- Atoms-in-Atom 패턴
  - 아톰 구성은 문자열로 평가될 때 유일한 식별자로 반환하므로 key로 사용이 가능
  - 해당 패턴을 사용하면 리렌더링을 줄일 수 있음

https://github.com/sangsang-6914/micro_state_management/tree/main/jotai

#### 느낀점

- 아톰(원자)를 이용해 상태를 최대한 잘게 쪼개서 관리하는 것이 Jotai의 핵심원리
- 아톰들을 잘 결합하여 파생아톰을 만드는것이 불필요한 리렌더링을 최적화 시키는 것이기에 효율적으로 파생아톰을 만들고 관리하는 것이 중요해보임
- zustand에 비해서 사용법이 더 단순화 되어 있는듯
- 하지만 오히려 복잡한 상태를 관리하기 위해서는 아톰의 조합과정이 더 복잡하게 느껴지는 것 같음 + 보일러플레이트 코드가 많아지는 느낌
  - 큰규모 프로젝트에는 적합하지 않은듯?
