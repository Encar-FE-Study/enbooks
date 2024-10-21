## Zustand에서 Store 렌더링

### Shallow Compare를 통해 스토어의 상태업데이트를 비교

### 상태값을 가져올때 useStore()와 같이 스토어 전체를 가져오는 것을 지양하자

### 값을 selector로 가져와야 렌더링 최적화가 된다

```javascirpt

// store.js
import create from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  user: {
    name: 'Alice',
    age: 30,
  },
  setUser: (newUser) => set({ user: { ...newUser } }),
}));

export default useStore;

```

```javascript
// Counter.js
import React from "react";
import useStore from "./store";

function Counter() {
  const { count, increment } = useStore();

  console.log("Counter component rendered");

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default Counter;
```

```javascript
// UserProfile.js
import React from "react";
import useStore from "./store";

function UserProfile() {
  const { user, setUser } = useStore(); // 이렇게 쓰면 전체 Store를 구독하는 꼴이다

  const updateUser = () => {
    setUser({ name: "Bob", age: 25 });
  };
  // updateUser함수 실행시 -> store 업데이트 -> Counter는 관련없는 속성값을 참고하고 있는 것처럼 보이나 Store 전체를 참고하고 있기에 Counter 컴포넌트도 다시 렌더링된다
  return (
    <div>
      <p>
        User: {user.name}, Age: {user.age}
      </p>
      <button onClick={updateUser}>Update User</button>
    </div>
  );
}

export default UserProfile;
```

## 해결

- 아래와 같이 selector를 사용하거나 shallow 비교를 해주자(해당 속성이 변할때만 리렌더가 일어나게 함)

```javascript
const { count, increment } = useStore((state) => ({
  count: state.count,
  increment: state.increment,
}));

or;

import shallow from "zustand/shallow";

const { count, increment } = useStore(
  (state) => ({
    count: state.count,
    increment: state.increment,
  }),
  shallow
);
```

## 중첩 객체 다루기

- IMMMER가 필수는 아니다.zustand에 Immer가 내장되어 있지는 X
- 같이 사용하면 편하다

```javascript
import create from "zustand";

// Define the store
const useStore = create((set) => ({
  user: {
    name: "Alice",
    address: {
      city: "Wonderland",
      zip: "12345",
    },
  },
  // Action to update the user's city
  setCity: (newCity) =>
    set((state) => ({
      user: {
        ...state.user,
        address: {
          ...state.user.address,
          city: newCity,
        },
      },
    })),
}));

function UpdateCity() {
  const setCity = useStore((state) => state.setCity);

  const handleChangeCity = () => {
    setCity("New Wonderland");
  };

  return <button onClick={handleChangeCity}>Change City</button>;
}

import create from "zustand";
import { immer } from "zustand/middleware/immer";
```

```javascript
const useStore = create(
  immer((set) => ({
    user: {
      name: "Alice",
      address: {
        city: "Wonderland",
        zip: "12345",
      },
    },
    setCity: (newCity) =>
      set((state) => {
        state.user.address.city = newCity; // Mutable
      }),
  }))
);

// Usage in a component
function UpdateCity() {
  const setCity = useStore((state) => state.setCity);

  const handleChangeCity = () => {
    setCity("New Wonderland");
  };

  return <button onClick={handleChangeCity}>Change City</button>;
}
```
