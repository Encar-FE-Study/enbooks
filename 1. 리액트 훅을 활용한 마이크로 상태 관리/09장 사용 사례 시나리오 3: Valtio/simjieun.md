# 09장. 사용 사례 시나리오 3: Valtio
- Valtio는 Zustand나 Jotai와는 다르게 변경 가능한 갱신 모델을 기반으로 하는 전역 상태 관리 라이브러리다.
- Valtio는 프락시를 사용해 변경 불가능한 스냅숏을 가져온다.
- Valtio의 자동 렌더링 최적화는 상태 사용 추적 이라는 기법을 기반으로 한다.

## Valtio 살펴보기
- Valtio는 프락시를 활용해 상태 변경을 감지하는 라이브러리이다.
- 프락시는 자바스크립트의 특수한 객체로서 객체 연산을 감지하기 위한 핸들러를 만드는 데 활용할 수 있다.
```javascript
const proxyObject = new Proxy({
    count: 0,
    text: "hello",
}, {
    get: (target, prop, receiver) => {
        console.log("start getting", target, prop, receiver);
    },
    set: (target, prop, value) => {
        console.log("start setting", target, prop, value);
        target[prop] = value;
        console.log("end setting", target, prop, value);
    }
});
```
- 위처럼 선언하며, 첫번째 인수는 객체이고 두번째 인수는 핸들러는 담는 컬렉션 객체이다.
- javascript에 Proxy 객체는 ES6에서 등장하였다.
- 핸들러에 들어갈수있는 메소드는 링크를 통해 알수있다.(https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy)

### Proxy 객체를 사용해서 로컬스토리지 동기화하기
```javascript
const STORAGE_KEY = "index_userinfo_value";
const loadFromLocalStorage = () => {
    const storedData = localStorage.getItem(STORAGE_KEY);
    return storedData ? JSON.parse(storedData) : { age: "20", gender: "여성", location: "서울" };
};

const saveToLocalStorage = (data) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
};

// Proxy로 감싸는 객체 생성
const syncedObject = new Proxy(loadFromLocalStorage(), {
    get: (target, prop) => {
        return target[prop];
    },
    set: (target, prop, value) => {
        console.log(`Setting ${prop} to ${value}`);
        target[prop] = value;            
        saveToLocalStorage(target);
        return true;
    }
});

syncedObject.age = "30";            // age 값 변경 -> 로컬 스토리지에 자동 저장
syncedObject.location = "제주";      // location 값 변경 -> 로컬 스토리지에 자동 저장
```

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기
- Valtio는 프락시를 사용해 변경 가능한 객체에서 변경 불가능한 객체를 생성한다. 이 불변 객체를 스냅숏(snapshot)이라고 한다.
- 프락시 객체로 감싼 변경 가능한 객체를 생성할려면 Valtio를 통해 proxy 함수를 사용하면 된다.
- 불변 객체를 생성할려면 Valtio를 통해 snapshot 함수를 사용하면 된다.

```javascript
import { proxy, snapshot } from "valtio";

const state = proxy({ count: 0 });
const snap = snapshot(state);
```

## 프락시를 활용한 리렌더링 최적화
- Valtio는 리렌더링을 최적화하고 변경을 감지하기 위해 프락시를 사용한다.
- snapshot 프락시는 스냅숏 객체의 속성 접근을 감지하는 사용된다.
- 아래의 예제 에서는 state 프락시 객체를 변경하고 snap은 읽기만 한다. snap객체는 Object.freeze로 동결되며 기술적으로 변경할 수 없다.
- 접근은 useSnapshot 훅에 의해 추적 정보를 감지되며, 추적 정보를 기반으로 useSnapshot 훅은 필요한 경우에만 리렌더링을 감지한다.
```javascript
import { proxy, useSnapshot } from "valtio";

const state = proxy({
  count1: 0,
  count2: 0,
});

const Counter1 = () => {
  const snap = useSnapshot(state);
  const inc = () => ++state.count1;
  return (
    <>
      {snap.count1} <button onClick={inc}>+1</button>
    </>
  );
};

const Counter2 = () => {
  const snap = useSnapshot(state);
  const inc = () => ++state.count2;
  return (
    <>
      {snap.count2} <button onClick={inc}>+1</button>
    </>
  );
};

const App = () => (
  <>
    <div>
      <Counter1 />
    </div>
    <div>
      <Counter2 />
    </div>
  </>
);

export default App;
```

## 이 접근 방식의 장단점
- Valtio는 두 가지 상태 업데이트 모델이 있다.
- 하나는 불변 갱신이고 하나는 변경 가능한 객체이다. 자바스크립트 자체는 변경 가능한 갱신을 허용하지만 리액트는 불변 상태를 중심으로 만들어졌다. 두모델을 같이 사용하는 경우 혼동하지 않도록 주의해야 한다.
- Valtio는 변경 가능한 갱신으로 애플리케이션 코드를 줄이는데 도움이 된다. 또한 Valtio는 프락시 기반 렌더링 최적화를 사용해 애플리케이션 코드를 줄이는데 도움이 된다.



## 프락시를 활용한 리렌더링 최적화
- Valtio는 리렌더링을 최적화하고 변경을 감지하기 위해 프락시를 사용한다.
- snapshot 프락시는 스냅숏 객체의 속성 접근을 감지하는 사용된다.
- 아래의 예제 에서는 state 프락시 객체를 변경하고 snap은 읽기만 한다. snap객체는 Object.freeze로 동결되며 기술적으로 변경할 수 없다.
- 접근은 useSnapshot 훅에 의해 추적 정보를 감지되며, 추적 정보를 기반으로 useSnapshot 훅은 필요한 경우에만 리렌더링을 감지한다.
```javascript
import { proxy, useSnapshot } from "valtio";

const state = proxy({
    count1: 0,
    count2: 0,
});

const Counter1 = () => {
    const snap = useSnapshot(state);
    const inc = () => ++state.count1;
    return (
        <>
        {snap.count1} <button onClick={inc}>+1</button>
        </>
    );
};

const Counter2 = () => {
    const snap = useSnapshot(state);
    const inc = () => ++state.count2;
    return (
        <>
        {snap.count2} <button onClick={inc}>+1</button>
        </>
    );
};

const App = () => (
    <>
        <div>
            <Counter1 />
        </div>
        <div>
            <Counter2 />
        </div>
    </>
);

export default App;
```

## 이 접근 방식의 장단점
- Valtio는 두 가지 상태 업데이트 모델이 있다.
- 하나는 불변 갱신이고 하나는 변경 가능한 객체이다. 자바스크립트 자체는 변경 가능한 갱신을 허용하지만 리액트는 불변 상태를 중심으로 만들어졌다. 두모델을 같이 사용하는 경우 혼동하지 않도록 주의해야 한다.
- Valtio는 변경 가능한 갱신으로 애플리케이션 코드를 줄이는데 도움이 된다. 또한 Valtio는 프락시 기반 렌더링 최적화를 사용해 애플리케이션 코드를 줄이는데 도움이 된다.
