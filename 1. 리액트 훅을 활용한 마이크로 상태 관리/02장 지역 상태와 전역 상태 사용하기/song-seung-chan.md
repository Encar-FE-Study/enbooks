## 함수와 인수

```javascript
const addOne = (n) => n + 1;
```

- 자바스크립트에서 함수는 인수를 받아서 값을 반환한다.
- 이 함수는 같은 인수에 대해 항상 같은 값을 반환하는 순수 함수이다. 순수함수는 같은 입력 - 같은 출력을 보장하고, 외부 변수를 변경하지 않기에 멱등성을 보장한다
- 멱등성 : 같은 연산을 여러번 수행해도 결과가 달라지지 X

```javascript
const base = 10;
const addBase = (n) => n + base;
```

위는 전역변수(base)에 따라 순수함수가 되지 않을 수 있는 위험이 있다.
이런 코드들이 산재하면 실행 흐름의 파악이 어려워진다.
따라서 아래처럼 변수의 scope를 제한하여서 외부에서는 변경이 불가하여 안정성을 보장시킬 수 있다.

```javascript
const createContainer = () => {
  let base = 1;
  const addBase = (n) => n + base;
  const changeBase = (b) => {
    base = b;
  };
  return { addBase, changeBase };
};

const { addBase, changeBase } = createContainer();
```

즉, 전역 변수의 잦은 사용은 애플리케이션 코드의 파악을 힘들게 한다.
따라서 리액트에서도 마찬가지로 지역상태를 기본으로 하고 전역상태를 보조적으로 사용해야 한다.

## React에서의 전역 상태

- 여러컴포넌트에서 사용되는 상태값인 경우
- 인증 상태, 다크모드 등 리액트 애플리케이션 바깥의 값인 경우
- 이럴때 Zustand,Redux를 사용한다.
- React.Context로도 대체가 가능하긴 하지만 단점이 있다.
- 단점
- 리렌더링 최적화가 안됨 : React.context는 value 변경시 useContext를 통해 해당 context를 사용하는 모든 컴포넌트를 리렌더링한다

```javascript
import React, { createContext, useContext, useState } from "react";

// Create the Context
const AppContext = createContext();

const AppProvider = ({ children }) => {
  const [user, setUser] = useState({ name: "John" });
  const [theme, setTheme] = useState({ color: "dark" });

  return (
    <AppContext.Provider value={{ user, theme }}>
      {children}
    </AppContext.Provider>
  );
};

const UserComponent = () => {
  const { user } = useContext(AppContext);
  console.log("UserComponent Rendered");
  return <div>User: {user.name}</div>;
};

const ThemeComponent = () => {
  const { theme } = useContext(AppContext);
  console.log("ThemeComponent Rendered");
  return <div>Theme: {theme.color}</div>;
};

const App = () => {
  return (
    <AppProvider>
      <UserComponent />
      <ThemeComponent />
      {/* Button to update user */}
      <button onClick={() => setUser({ name: "Jane" })}>
        Change User Name
      </button>
    </AppProvider>
  );
};

export default App;
```

버튼 클릭시 value중 name만 변경되었음에도 ThemeComponent의 콘솔도 출력된다.
이는 AppProvider의 value값이 재평가되어 발생하는 것이기에, 이를 방지하려면 아래와 같은 Context의 분리가 필요하다.

```javascript
    <AppProvider> // value로 name만 가짐
        <ThemeProvider> // value로 color만 가짐
            <UserComponent />
            <ThemeComponent />
            {/* Button to update user */}
            <button onClick={() => setUser({ name: "Jane" })}>
                Change User Name
            </button>
        </ThemeProvider>
    <AppProvider>
```

반면에 reudx나 zustand는 해당 값을 참조하고 있는 컴포넌트만 리렌더링된다(Selective ReRendering)
리덕스를 예시로 들어보자

```javascript


1.store.js
// Create Store
import { createStore } from "redux";
import { Provider } from "react-redux";

// Initial State
const initialState = {
  user: { name: "John" },
  theme: { color: "dark" }
};

// Reducer
const reducer = (state = initialState, action) => {
  switch (action.type) {
    case "SET_USER":
      return { ...state, user: action.payload };
    default:
      return state;
  }
};


2.사용하는 컴포넌트
import { useSelector, useDispatch } from "react-redux";

const UserComponent = () => {
  const user = useSelector((state) => state.user);
  console.log("UserComponent Rendered");
  return <div>User: {user.name}</div>;
};

const ThemeComponent = () => {
  const theme = useSelector((state) => state.theme);
  console.log("ThemeComponent Rendered");
  return <div>Theme: {theme.color}</div>;
};

const App = () => {
  const dispatch = useDispatch();

  return (
    <div>
      <UserComponent />
      <ThemeComponent />
      {/* Button to update user */}
      <button onClick={() => dispatch({ type: "SET_USER", payload: { name: "Jane" } })}>
        Change User Name
      </button>
    </div>
  );
};


3. App.js
import React from "react";
import {createRoot} from "react-dom/client";
import { Provider } from "react-redux";
import { store } from "./store";

const root = createRoot(document.getElementById("root"));

root.render(
    <Provider store={store}>
    <App />
  </Provider>,)

```

버튼 클릭시 상태값 중 name이 변경되고, UserComponent의 콘솔만 한 번 더 찍히게 된다.
useSelector가 이 과정에서 shallow compare를 통해 user객체의 변경만 감지하기 때문이다
