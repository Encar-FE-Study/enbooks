# 08장. 사용 사례 시나리오 3: Valtio

# Valtio 살펴보기
- 자바스크립트 Proxy 객체 -> 자바스크립트의 특수한 객체로서 객체 연산을 감지하기 위한 핸들러를 만드는데 활용 가능

```js
const proxyObject = new Proxy({
    count:0,
    text:'hello'
}, {
    set:(target, prop, value) => {
        console.log('start setting', prop);
        target[prop] = value;
        console.log('end setting', prop);
    }
})
```

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기
- 프락시를 사용해 변경 가능한 객체에서 변경 불가능한 객체(스냅숏)를 생성
- snapshot은 Object.freeze로 동결된 상태

```js
import { proxy, snapshot } from 'valtio';

const state = proxy({count:0}); // 변경 가능한 객체
const snap1 = snapshot(state) // 변경 불가능한 객체(스냅숏)
```

## 프락시를 활용한 리렌더링 최적화
자동 렌더링 최적화 -> 상태 사용 추적이라는 기법 사용(객체, 배열 포함하는 객체도 지원)

```js
import {proxy, useSnapshot} from 'valtio';

const state = proxy({
    count1:0,
    count2:0
})

const Counter1 = () => {
    const snap = useSnapShot(state);
    const inc = () => ++state.count1;

    return (
        <>
        {snap.count1} <button onClick={inc}>+1</button>
        </>
    )
}

const Counter2 = () => {
    const snap = useSnapShot(state);
    const inc = () => ++state.count2;

    return (
        <>
            {snap.count2} <button onClick={inc}>+1</button>
        </>
    )
}
```

## 접근 방식의 장단점
- 변경 가능한 갱신과 불변 갱신 두가지 혼용
- 자동 렌더링 최적화 -> 디버깅 어려움

- [valtio 활용 앱](https://github.com/youngrove/client-state-packages)
