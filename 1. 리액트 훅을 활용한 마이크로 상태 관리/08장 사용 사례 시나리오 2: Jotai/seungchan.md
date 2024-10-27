# Jotai

# 특징
- 사용이 간단(Provider 선언없이 사용 가능)
- Provider를 주어서 Scope 별로 상태를 분리 가능
- Provider를 명시적으로 선언하지 않아도 내부적으로 기본 전역 스토어를 사용하여 atom을 관리
- ex)useAtom은 useState와 비슷한 구조이지만 atom이 조타이 내부의 Provider 통해 관리되기에 전역 상태 관리 가능

# Atom이란
- 조타이의 가장 작은 상태 단위
- 상태 저장/공유/새로운 상태 생성(수정)이 가능
- 불변성을 가짐(직접 수정대신 setter함수 통해 수정가능)
- 하나의 아톰은 하나의 상태만 관리


```javascript
// 1. 상태 저장
const textAtom = atom('hello') // 아톰 객체를 반환

function TextInput() {
  const [text, setText] = useAtom(textAtom)
  // text: 현재 상태 값
  // setText: 상태 업데이트 함수
  
  return <input value={text} onChange={e => setText(e.target.value)} />
}

// 2. 상태 공유
function TextDisplay() {
  const [text] = useAtom(textAtom) // 같은 atom을 사용하여 상태 공유
  return <div>Current text: {text}</div>
}

// 3. 상태 파생
const uppercaseAtom = atom(
  (get) => get(textAtom).toUpperCase() // textAtom의 값을 기반으로 새로운 상태 생성
)
```
## Atoms-in-Atom
- 하나의 아톰 안에 여러 개의 아톰을 포함할 수 있는 패턴
- 복잡한 상태를 구조화하는 데에 좋다
``` javascript
// 개별 아톰들
const firstNameAtom = atom('John')
const lastNameAtom = atom('Doe')
const ageAtom = atom(25)

// 여러 아톰을 하나로 조합 - 파생아톰(기존아톰을 가지고 새로운 상태를 조합)
const userAtom = atom((get) => ({
  firstName: get(firstNameAtom),
  lastName: get(lastNameAtom),
  age: get(ageAtom),
  fullName: `${get(firstNameAtom)} ${get(lastNameAtom)}`
}))

// 컴포넌트에서 사용
function UserProfile() {
  const [user] = useAtom(userAtom)
  const [firstName, setFirstName] = useAtom(firstNameAtom)
  
  return (
    <div>
      <p>Full name: {user.fullName}</p>
      <input 
        value={firstName}
        onChange={(e) => setFirstName(e.target.value)}
      />
    </div>
  )
}
```
```javascript
// atoms
const todosAtom = atom([])
const filterAtom = atom('all')
const filteredTodosAtom = atom((get) => {
  const todos = get(todosAtom)
  const filter = get(filterAtom)
  return filterTodos(todos, filter)
})

// 컴포넌트들
function TodoList() {
  const [filteredTodos] = useAtom(filteredTodosAtom)
  console.log("TodoList 리렌더링")
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  )
}

function TodoFilter() {
  const [filter, setFilter] = useAtom(filterAtom)
  console.log("TodoFilter 리렌더링")
  
  return (
    <select value={filter} onChange={e => setFilter(e.target.value)}>
      <option value="all">All</option>
      <option value="active">Active</option>
      <option value="completed">Completed</option>
    </select>
  )
}

function TodoStats() {
  const [todos] = useAtom(todosAtom)
  console.log("TodoStats 리렌더링")
  
  return <div>Total todos: {todos.length}</div>
}
=> todosAtom이나 filterAtom의 값이 변경되면 자동으로 filteredTodosAtom도 재계산됨

1) setTodos([...todos, newTodo])
// 1. todosAtom 값 변경
// 2. filteredTodosAtom 재계산
// 3. 리렌더링되는 컴포넌트:
//    - TodoList (filteredTodosAtom 사용)
//    - TodoStats (todosAtom 사용)
//    - TodoFilter는 리렌더링 되지 않음 (filterAtom 값 변경 없음)

2) setFilter('active')
// 1. filterAtom 값 변경
// 2. filteredTodosAtom 재계산
// 3. 리렌더링되는 컴포넌트:
//    - TodoList (filteredTodosAtom 사용)
//    - TodoFilter (filterAtom 사용)
//    - TodoStats는 리렌더링 되지 않음 (todosAtom 값 변경 없음)


```

# Provider구현(간략하게)
```javscript
import React, { createContext, useContext, useEffect, useState } from 'react';

// Store와 Atom 타입 정의
type Atom<T> = {
  init: T;
};

type Listener = () => void;

export class Store {
  private values = new Map<Atom<any>, any>();
  private listeners = new Map<Atom<any>, Set<Listener>>();

  get<T>(atom: Atom<T>): T {
    if (!this.values.has(atom)) {
      this.values.set(atom, atom.init);
    }
    return this.values.get(atom);
  }

  set<T>(atom: Atom<T>, value: T): void {
    this.values.set(atom, value);
    this.notify(atom);
  }

  sub<T>(atom: Atom<T>, listener: Listener): () => void {
    if (!this.listeners.has(atom)) {
      this.listeners.set(atom, new Set());
    }
    this.listeners.get(atom)!.add(listener);

    return () => {
      this.listeners.get(atom)?.delete(listener);
      if (this.listeners.get(atom)?.size === 0) {
        this.listeners.delete(atom);
      }
    };
  }

  private notify<T>(atom: Atom<T>): void {
    this.listeners.get(atom)?.forEach(listener => listener());
  }
}

// Store를 위한 Context 생성
const StoreContext = createContext<Store | null>(null);

// Provider 컴포넌트
interface ProviderProps {
  store: Store;
  children: React.ReactNode;
}

export const Provider: React.FC<ProviderProps> = ({ store, children }) => {
  return (
    <StoreContext.Provider value={store}>
      {children}
    </StoreContext.Provider>
  );
};

// 편의를 위한 커스텀 훅
export const useStore = () => {
  const store = useContext(StoreContext);
  if (!store) {
    throw new Error('Store Provider not found');
  }
  return store;
};

// atom 생성 함수
export const atom = <T,>(initialValue: T): Atom<T> => ({
  init: initialValue,
});

// atom 값을 사용하기 위한 훅
export const useAtom = <T,>(atom: Atom<T>) => {
  const store = useStore();
  const [value, setValue] = useState(() => store.get(atom));

  useEffect(() => {
    const unsub = store.sub(atom, () => {
      setValue(store.get(atom));
    });
    return unsub;
  }, [store, atom]);

  const setAtom = (newValue: T) => {
    store.set(atom, newValue);
  };

  return [value, setAtom] as const;
};
```


## Provider 사용예시
```javascript
// 1. atom 생성
const countAtom = atom(0)
const nameAtom = atom('John')

// 2. Provider 없이 사용
function Counter1() {
  const [count, setCount] = useAtom(countAtom)
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count1: {count}
    </button>
  )
}

// 3. Privder와 함께
const myStore = createStore()

const countAtom = atom(0)
myStore.set(countAtom, 1)
const unsub = myStore.sub(countAtom, () => {
  console.log('countAtom value is changed to', myStore.get(countAtom))
})
// unsub() to unsubscribe

const Root = () => (
  <Provider store={myStore}>
    <App />
  </Provider>
)
``