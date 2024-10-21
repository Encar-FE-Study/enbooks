# 데이터 중심 접근 방식과 컴포넌트 중심 접근 방식 사용하기
## 둘의 구분
구분기준: "중앙화된 스토어를 사용하는가, 각 컴포넌트가 개별적으로 상태를 관리하는가"가 핵심. 하지만 실제 애플리케이션에서는 이 두 접근 방식을 혼합하여 사용

```
데이터 중심 접근 방식: 중앙 집중식 스토어(하나의 Store)를 사용하여 전역 상태를 관리, 여러 컴포넌트가 동일한 데이터를 공유하고 접근가능
컴포넌트 중심 접근 방식: 개별 컴포넌트가 자신의 로컬 상태를 관리, 필요 시 props를 통해 데이터를 전달

React.ContextAPI는 이 둘의 중간 형태로 상태 공유 가능
```

### 차이점
1. 중앙 집중화 vs. 분산화:
데이터 중심 접근 방식은 중앙 집중식으로 상태를 관리하여 모든 컴포넌트가 동일한 상태를 공유하고 접근가능
컴포넌트 중심 접근 방식은 분산화된 방식으로, 각 컴포넌트가 독립적으로 자신의 상태를 관리

2. 상태의 범위:

데이터 중심 접근 방식: 글로벌 상태를 관리하여 애플리케이션 전체에서 필요한 데이터를 쉽게 접근하고 공유할 수 있습니다.
컴포넌트 중심 접근 방식: 로컬 상태를 관리하여 특정 컴포넌트 내에서만 상태를 사용하고 변경합니다.
3. 복잡성 및 확장성:
데이터 중심 접근 방식: 대규모 애플리케이션에 적합하며, 상태 관리가 복잡해도 중앙에서 일관되게 관리가능 (하나의 데이터 소스)
컴포넌트 중심 접근 방식: 소규모 또는 중간 규모의 애플리케이션에 적합하며, 단순한 상태 관리에 효과적

4. 코드 구조:
데이터 중심 접근 방식: 리듀서, 액션, 스토어 등의 구조가 필요하여 코드가 더 구조화되고 일관적
컴포넌트 중심 접근 방식: 각 컴포넌트 내에서 상태를 관리하므로, 별도의 상태 관리 구조가 필요하지 않다.


## useTrackedState 훅 구현 예시
### 요구사항
기능: 
1. useTrackedState 훅은 trackedState의 .b.c 속성에 접근했음을 감지할 수 있으며
2. .b.c 속성값이 변경될 때만 해당 훅을 사용하는 컴포넌트가 리렌더링되야

다음을 고려해야
1) 중앙 상태 저장소: 전역 상태를 관리할 중앙 저장소를 만듭니다. 여기서는 간단히 useState와 useEffect를 사용

2) 프로퍼티 접근 추적: Proxy 객체를 사용하여 trackedState.b.c에 접근할 때를 감지합니다.

3) 구독 시스템: 특정 속성이 변경될 때만 컴포넌트를 리렌더링하도록 구독 시스템을 구현합니다.

아래는 이러한 요구사항을 충족하는 useTrackedState 훅의 구현 예시


```
1. 중앙 상태 저장소 구현
// trackedStore.js
import { useState, useEffect } from 'react';

let globalState = {
  a: 1,
  b: {
    c: 2,
    d: 3,
  },
};

// 변경 구독자 리스트
const subscribers = {
  'b.c': new Set(),
};

// 상태 변경 함수
export function setState(path, value) {
  const keys = path.split('.');
  let obj = globalState;
  for (let i = 0; i < keys.length - 1; i++) {
    obj = obj[keys[i]];
  }
  obj[keys[keys.length - 1]] = value;
  // 구독자에게 알림
  if (subscribers[path]) {
    subscribers[path].forEach((callback) => callback());
  }
}

// Proxy를 사용하여 상태 변경 감지
export const proxyState = new Proxy(globalState, {
  get(target, prop) {
    const value = target[prop];
    if (typeof value === 'object' && value !== null) {
      return new Proxy(value, this);
    }
    return value;
  },
  set(target, prop, value) {
    target[prop] = value;
    // 변경된 경로에 따라 구독자에게 알림
    const path = prop.toString();
    if (subscribers[path]) {
      subscribers[path].forEach((callback) => callback());
    }
    return true;
  },
});


```


```
2. useTrackedState 훅 구현

// useTrackedState.js
import { useState, useEffect } from 'react';
import { proxyState, subscribers, setState } from './trackedStore';

export function useTrackedState() {
  // 내부적으로 .b.c 값을 상태로 관리
  const [value, setValue] = useState(proxyState.b.c);

  useEffect(() => {
    // 변경을 감지할 경로
    const path = 'b.c';

    // 변경 시 호출될 콜백
    const callback = () => {
      setValue(proxyState.b.c);
    };

    // 구독자에 콜백 추가
    subscribers[path].add(callback);

    // 정리 시 구독자에서 콜백 제거
    return () => {
      subscribers[path].delete(callback);
    };
  }, []);

  return [value, (newValue) => setState('b.c', newValue)];
}


```


```
3. 컴포넌트에서 useTrackedState 사용하기

// TrackedComponent.js
import React from 'react';
import { useTrackedState } from './useTrackedState';

function TrackedComponent() {
  const [cValue, setCValue] = useTrackedState();

  const incrementC = () => {
    setCValue(cValue + 1);
  };

  return (
    <div>
      <h2>Tracked Component</h2>
      <p>b.c 값: {cValue}</p>
      <button onClick={incrementC}>b.c 값 증가</button>
    </div>
  );
}

export default TrackedComponent;


```

## 설명

```
1.중앙 상태 저장소 (trackedStore.js):

globalState는 애플리케이션의 전역 상태를 나타냅니다.
subscribers 객체는 특정 경로(b.c)에 대한 구독자 콜백들을 관리합니다.
setState 함수는 특정 경로의 값을 업데이트하고 해당 경로의 구독자들에게 변경을 알립니다.
proxyState는 Proxy 객체를 사용하여 상태의 변경을 감지하고, 변경 시 구독자에게 알립니다.


2. useTrackedState 훅 (useTrackedState.js):
훅은 proxyState.b.c 값을 로컬 상태로 관리합니다.
useEffect를 사용하여 b.c 경로의 변경을 구독(subscribe)하고, 변경 시 로컬 상태를 업데이트하여 컴포넌트를 리렌더링합니다.
훅은 현재 b.c 값과 이를 업데이트할 수 있는 함수를 반환합니다.


3.컴포넌트 (TrackedComponent.js):
useTrackedState 훅을 사용하여 b.c 값을 가져오고, 이를 화면에 표시합니다.
버튼을 클릭하면 b.c 값을 증가시키는 함수를 호출하여 상태를 업데이트합니다.


4. 애플리케이션 (App.js):  
TrackedComponent를 렌더링하여 useTrackedState 훅의 동작을 확인할 수 있습니다.
결과:
TrackedComponent는 b.c 값에만 관심이 있으며, b.c가 변경될 때만 리렌더링됩니다.
다른 상태(a나 b.d)가 변경되어도 TrackedComponent는 리렌더링되지 않습니다.
```



## 추가 고려사항
- 확장성: 현재 구현은 특정 경로(b.c)에 한정되어 있습니다. 여러 경로를 추적하려면 훅의 인자로 경로를 받아 처리하도록 확장할 수 있습니다.
```
// useTrackedState.js 수정
export function useTrackedState(path) {
  const [value, setValue] = useState(getValueByPath(proxyState, path));

  useEffect(() => {
    const callback = () => {
      setValue(getValueByPath(proxyState, path));
    };

    if (!subscribers[path]) {
      subscribers[path] = new Set();
    }
    subscribers[path].add(callback);

    return () => {
      subscribers[path].delete(callback);
    };
  }, [path]);

  return [value, (newValue) => setState(path, newValue)];
}

// helper 함수 추가
function getValueByPath(obj, path) {
  return path.split('.').reduce((acc, key) => acc[key], obj);
}

```
- 최적화: 상태 업데이트 시 실제로 값이 변경되었는지 확인하여 불필요한 리렌더링을 방지할 수 있습니다.
```
// setState 함수 수정
export function setState(path, value) {
  const keys = path.split('.');
  let obj = globalState;
  for (let i = 0; i < keys.length - 1; i++) {
    obj = obj[keys[i]];
  }
  if (obj[keys[keys.length - 1]] === value) return; // 값이 동일하면 무시
  obj[keys[keys.length - 1]] = value;
  if (subscribers[path]) {
    subscribers[path].forEach((callback) => callback());
  }
}
```


## Zustand에서 유사한 기능구현
### 정의
```
// store.js
import create from 'zustand';

const useStore = create((set) => ({
  b: {
    c: 2,
    d: 3,
  },
  setC: (newC) => set((state) => ({ b: { ...state.b, c: newC } })),
}));

export default useStore;

```

###  사용
```
// TrackedComponent.js
import React from 'react';
import useStore from './store';

function TrackedComponent() {
  const c = useStore((state) => state.b.c);
  const setC = useStore((state) => state.setC);

  const incrementC = () => {
    setC(c + 1);
  };

  return (
    <div>
      <h2>Tracked Component with Zustand</h2>
      <p>b.c 값: {c}</p>
      <button onClick={incrementC}>b.c 값 증가</button>
    </div>
  );
}

export default TrackedComponent;

```