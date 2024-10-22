# 06. 전역 상태 관리 라이브러리 소개

## 전역상태 관리 문제 해결하기

- 전역상태를 설계할 때 두가지 문제점이 있다.

#### 1. 전역상태를 읽는 방법

- 전역 상태는 여러가지 값을 가질 수 있고, 전역상태를 사용하는 컴포넌트는 모든 값이 필요하지 않을 수 있다. 따라서 변경된 값이 컴포넌트와 관련이 없어도 불필요한 리렌더링이 발생한다.

#### 2. 전역상태에 값을 넣거나 갱신하는 방법

- 전역 상태는 여러 값을 가질 수 있으며, 중첩된 객체일 수 있다. 전역 상태 변경을 감지하기 위해서는 전역 상태를 변경하는 함수를 제공해야 한다.

## 데이터 중심 접근 방식과 컴포넌트 중심 접근 방식 사용하기

#### 데이터 중심 접근 방식 이해하기

- 데이터 중심 접근 방식의 경우는 모듈상태가 리액트 외부의 자바스크립트 메모리에 있기 때문에 모듈 상태를 사용하는 편이 적합함
- 데이터 중심 접근 방식을 사용하는 전역 상태 라이브러리는 모듈 상태를 생성하고 모듈 상태를 리액트 컴포넡트에 연결하는 API를 제공함

#### 컴포넌트 중심 접근방식 이해하기

- 컴포넌트 중심 접근 방식을 사용하면 컴포넌트를 먼저 설계할 수 있으며 컴포넌트 생명 주기 내에서 전역 상태를 유지하는 것이 적합함
- 컴포넌트 중심 접근 방식을 사용하는 전역 상태 라이브러리는 팩토리 함수를 제공하고, 생성된 함수를 통해 전역 상태의 생명주기를 처리하도록 함

> 데이터 중심 접근방식은 상태나 UI에 집중하는 것이 아니라 데이터 자체의 변화를 관리하고, 필요한 곳에서 데이터를 사용하는 구조
```javascript
// 1. Zustand로 데이터 중심의 전역 상태 스토어 생성
const useStore = create((set) => ({
  todos: [],  // 전역 데이터 상태 (할 일 목록)
  addTodo: (todo) => set((state) => ({ todos: [...state.todos, todo] })),
  removeTodo: (id) => set((state) => ({
    todos: state.todos.filter((todo) => todo.id !== id)
  })),
}));

// 2. 할 일 추가를 위한 컴포넌트
const AddTodo = () => {
  const addTodo = useStore((state) => state.addTodo);
  
  const handleAdd = () => {
    const newTodo = { id: Math.random(), title: "새 할 일" };
    addTodo(newTodo);
  };

  return (
    <button onClick={handleAdd}>Add Todo</button>
  );
};

// 3. 할 일 목록을 보여주는 컴포넌트
const TodoList = () => {
  const todos = useStore((state) => state.todos);  // 전역 데이터 상태인 'todos'만 구독

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
};

// 4. 메인 앱 컴포넌트
const App = () => {
  return (
    <div>
      <h1>Todo App with Zustand (Data-Centric)</h1>
      <AddTodo />
      <TodoList />
    </div>
  );
};

export default App;
```

> 컴포넌트 중심 접근방식은 각 컴포넌트가 필요한 전역 상태만 선택적으로 구독하고 사용할 수 있도록 설계하는 구조
```javascript
// 1. Zustand로 전역 상태 스토어 생성
const useStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
  reset: () => set({ count: 0 }),
}));

// 2. 전역 상태를 사용하는 컴포넌트 A (카운트를 증가시키는 컴포넌트)
const CounterComponent = () => {
  const count = useStore((state) => state.count);  // count 상태를 구독
  const increase = useStore((state) => state.increase);  // increase 함수 구독

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increase}>Increase Count</button>
    </div>
  );
};

// 3. 전역 상태를 사용하는 컴포넌트 B (카운트를 리셋하는 컴포넌트)
const ResetComponent = () => {
  const reset = useStore((state) => state.reset);  // reset 함수만 구독

  return (
    <button onClick={reset}>Reset Count</button>
  );
};

// 4. 메인 앱 컴포넌트에서 전역 상태를 사용하는 여러 컴포넌트 구성
const App = () => {
  return (
    <div>
      <h1>Global State Management with Zustand</h1>
      <CounterComponent />
      <ResetComponent />
    </div>
  );
};

export default App;
```

## 리렌더링 최적화

- 리렌더링 최적화의 핵심은 컴포넌트에서 state의 어느 부분이 사용될지 지정하는 것임
- state의 일부분을 지정하는 3가지 방법은 **선택자 함수 사용, 속성 접근 감지, 아톰 사용** 등이 있음

#### 선택자 함수 사용

```javascript
const Component = () => {
  const value = useSelector((state) => state.b.c);
  ...
}
```

- 위와 같이 선택자함수를 사용해 어느 부분을 사용할지 명시적으로 지정하며 **수동 최적화** 시킬 수 있음

#### 속성 접근 감지

- 속성 접근을 감지하고 감지한 정보를 렌더링 최적화에 사용할 수 있는 상태 사용추적이 있음

```javascript
const Component = () => {
  const trackedState = useTrackedState();
  return (
    <>
      <p>{trackedState.b.c}</p>
    </>
  );
};
```

- 위와 같이 useTracedState 같이 속성접근을 감지하는 훅을 만들어 자동 최적화를 진행할 수 있음

#### 아톰사용

- 아톰은 리렌더링을 발생시키는 데 사용되는 최소 상태 단위
