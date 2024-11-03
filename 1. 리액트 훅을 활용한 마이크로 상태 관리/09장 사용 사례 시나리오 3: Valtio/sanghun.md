# 09장. 사용 사례 시나리오 3: Valtio

- valtio는 **변경 가능한 갱신모델을 기반**으로하는 전역상태 관리 라이브러리
- 모듈상태를 사용하며, proxy를 사용해 변경 불가능한 스냅숏을 가져온다.
- 자동으로 리렌더링을 최적화하며, **상태 사용 추적** 기법을 사용한다.

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기

- valtio는 proxy를 통해 변경 가능한 객체에서 변경 불가능한 객체를 생성한다. 이 객체를 **스냅숏**이라고 한다.
  > proxy를 활용해 불변객체를 만들 수 있는 프락시 객체를 생성할 수 있고, snapshot을 통해 불변객체(스냅숏)을 만들 수 있음
  > proxy객체와 snapshot은 다른 참조를 가지며 **snapshot은 readonly로 선언**된다.

```javascript
// state, snap1은 다른 참조를 가진다
// snap1은 readonly (불변)
export const state = proxy({ count: 0 });
const snap1 = snapshot(state);

++state.count;
const snap2 = snapshot(state);
```

- snap1과 snap2는 다른 참조를 가진다.
- proxy와 snapshot함수는 중첩 객체에 대해서도 작동하며 스냅샷 생성을 최적화 한다.

> valtio의 최적화는 이전 스냅숏에 대한 캐싱을 기반으로 동작한다.

## 프락시를 활용한 리렌더링 최적화

- 프락시 객체를 변경하고 snapshot은 읽기만 가능하며, snap은 Object.freeze로 동결된다.
- useSnapshot 훅에 의해 추적 정보로 감지되어 자동으로 리렌더링이 최적화 된다.

```javascript
export const state = proxy({
  count1: 0,
  count2: 0,
});

function Counter1() {
  const snap = useSnapshot(state);
  const inc = () => ++state.count1;
  return (
    <div>
      {snap.count1} <button onClick={inc}>+1</button>
    </div>
  );
}

function Counter2() {
  const snap = useSnapshot(state);
  const inc = () => {
    ++state.count2;
  };
  return (
    <div>
      {snap.count2}
      <button onClick={inc}>+1</button>
    </div>
  );
}
```

- Counter1과 Counter2는 동일한 스냅샷 객체를 바라보고 있는데, 각 컴포넌트의 useSnapshot 훅이 추적정보를 기억하고 알아낼 수 있어 Counter1 컴포넌트는 count2가 변경되어도 리렌더링되지 않는다.

## 작은 앱 만들어보기

https://github.com/sangsang-6914/micro_state_management/tree/main/valtio

## 이 접근방식의 장단점

- 변경 가능한 갱신의 가장 큰 장점은 자바스크립트 함수를 사용할 수 있다는 것
- 직관적으로 데이터를 변경할 수 있음
- 자동 렌더링 최적화를 통해 코드량을 줄일 수 있음
- 단점은 복잡한 구조에서 예측 가능성이 떨어진다는 것

#### 느낀점

- valtio는 proxy로 객체를 감싸 직접 객체를 수정하는 것처럼 다루면서도, 자동으로 불변성을 유지하고 리렌더링을 최적화 시켜줌
- 중첩객체나 배열같은 복잡한 데이터를 수정할 때 용이할 것 같음
- immer랑 비슷
- 이거쓸거면 그냥 jotai 쓸거 같음 (개인프로젝트 기준)
