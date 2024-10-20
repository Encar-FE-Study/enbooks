# 07. 사용 사례 시나리오1: Zustand

## 모듈 상태와 불변 상태 이해하기

- zustand는 상태 객체 속성을 갱신할 수 없는 **불변 상태 모델**을 기반으로 한다.
- 상태를 변경하기 위해서는 새 객체를 생성 후 대체하고 수정하지 않은 객체는 재사용한다.
  > 불변 상태모델은 상태 객체의 참조에 대한 **동등성**만 확인하면 객체 전체 값을 확인할 필요가 없음 (성능상 이점))
  > 깊은비교없이 얕은비교 만으로도 상태변화를 감지할 수 있어 성능비용을 감소시킬 수 있음
- state에 직접 접근을 해서 상태를 변경하는 것은 불변성을 위반하는 잘못된 사용법이다.

```javascript
import useStore from '../store/store';

function Main() {
  const { count, setState } = useStore();
  const handleIncrease = () => {
    count = count + 1;
    // setState({ count: count + 1 });
  };
  return (
    <>
      <div>{count}</div>
      <button onClick={handleIncrease}>+</button>
    </>
  );
}

export default Main;
```

- 위와 같이 상태를 변경하면 참조값이 변경되지 않아 리렌더링이 발생하지 않음

```javascript
  ...
  const handleIncrease = () => {
      setState({ count: count + 1 });
    };
  ...
```

- 위와 같이 새로운 객체를 통해 상태갱신을 해야한다.

```javascript
useStore.subscribe((newVal, prevVal) =>
  console.log(`${prevVal.count} => ${newVal.count}`)
);
```

- 스토어를 구독(subscribe)해 놓으면 구독한 상태가 변경되면 listenr가 실행된다.
  > subscribe를 활용하면 UI 업데이트, 로깅 및 디버깅, 비동기 작업 트리거 등에 유용하게 사용가능

## 리액트 훅을 이용한 리렌더링 최적화

```javascript
function Main() {
  console.log('Main rerendering');
  const count = useStore((state) => state.count);
  const inc = useStore((state) => state.inc);
  const handleIncrease = () => {
    inc();
  };

  return (
    <>
      <div>{count}</div>
      <button onClick={handleIncrease}>+</button>
    </>
  );
}
```

- useStore에 선택자 함수를 사용하면 'text'가 변화되어도 불필요한 리렌더링이 일어나지 않는다.
- 위와 같은 방식은 수동 렌더링 최적화임
  > 여러개의 state를 가져올 경우 코드가 너무 난잡해질 수 있어서 한번에 가져오는 방법이 없을까?

```javascript
  ...
  const { count, inc } = useStore(
    (state) => ({
      count: state.count,
      inc: state.inc,
    }),
    (prev, cur) => JSON.stringify(prev) === JSON.stringify(cur)
  );
  ...
```

- 위와 같이 selector 뒤에 옵셔널로되어있는 equalityFn에 메모리주소가 아닌 실제 값을 비교하는 함수를 넣어주면 된다. (+ custom selector 지정도 가능)
  https://velog.io/@2ast/React-Zustand-custom-selector%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EB%A0%8C%EB%8D%94%EB%A7%81-%EC%B5%9C%EC%A0%81%ED%99%94

## 읽기 상태와 갱신 상태 사용하기

- count1, count2의 합계를 보여주는 컴포넌트에서 count1은 증가하고 count2가 같은 양만큼 감소할 때 리렌더링이 발생하는 edge case를 피하기 위해 파생 상태에 대한 선택자 함수를 사용하면 됨

```javascript
function Total() {
  const total = useCountStore((state) => state.count1 + state.count2);
  console.log('rerendering!');
  return <div>{total}</div>;
}
```

## 구조화된 데이터 처리하기

```javascript
type Todo = {
  id: number,
  title: string,
  done: boolean,
};

type StoreState = {
  todos: Todo[],
  addTodo: (title: string) => void,
  removeTodo: (id: number) => void,
  toggleTodo?: (id: number) => void,
};

let nextId = 0;

const useTodoStore =
  create <
  StoreState >
  ((set) => ({
    todos: [],
    addTodo: (title) =>
      set((prev) => ({
        todos: [...prev.todos, { id: ++nextId, title, done: false }],
      })),
    removeTodo: (id) =>
      set((prev) => ({
        todos: prev.todos.map((todo) =>
          todo.id === id ? { ...todo, done: !todo.done } : todo
        ),
      })),
  }));

function TodoList() {
  const todos = useTodoStore((state) => state.todos);
  return (
    <div>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  );
}

function NewTodo() {
  const addTodo = useTodoStore((state) => state.addTodo);
  const [text, setText] = useState('');
  const onClick = () => {
    addTodo(text);
    setText('');
  };
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e) setText(e.target.value);
  };
  const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
    if (event.key === 'Enter') {
      onClick(); // Enter 키가 눌렸을 때 onClick 함수 실행
    }
  };
  return (
    <div>
      <input value={text} onChange={handleChange} onKeyDown={handleKeyDown} />
      <button onClick={onClick} disabled={!text}>
        Add
      </button>
    </div>
  );
}

function TodoItem({ todo }: { todo: Todo }) {
  console.log('TodoItem Renrender');
  const removeTodo = useTodoStore((state) => state.removeTodo);
  const toggleTodo = useTodoStore((state) => state.toggleTodo);
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => toggleTodo(todo.id)}
      />
      <span style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
        {todo.title}
      </span>
      <button onClick={() => removeTodo(todo.id)}>Delete</button>
    </div>
  );
}

export default memo(TodoItem);
```

## 이 접근 방식과 라이브러리의 장단점

- 읽기 상태 : 리렌더링을 최적화하기 위해 선택자 함수 사용
- 쓰기 상태 : 불변 상태 모델을 기반 (불변성 유지하며 상태 변경)
- Zustand 상태 모델은 객체 불변성 규칙과 완전히 일치
- 라이브러리 사용의 단순성과 번들크기가 작아 좋음 (1~2kb로 가장 작음)
- 선택자를 이용한 수동 렌더링 최적화가 단점 (custom Selector를 사용하면 어느정도 해결?)

## 느낀점

- Zustand 사용방법 자체가 정말 간단하고 쉬운듯
- 하지만 책에서도 나와있듯이 컴포넌트에서 사용해야 하는 상태가 여럿일 경우 수동 렌더링 최적화를 위해 추가 작업이 필요하다는 단점
- 사람들이 왜 많이 쓰는지는 알 것 같음
- subscribe기능을 실무에서 사용해보지 않았는데 잘 사용하면 좋을 것 같음
