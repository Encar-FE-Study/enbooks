# 04. 구독을 이용한 모듈 상태 공유
- 리액트에서 모듈 상태를 사용하는 방법을 알아보자. 컨텍스트보다 덜 알려진 패턴이지만 기존 모듈 상태를 통합하는데 자주 사용된다.

> 모듈 상태란?
> 모듈 상태의 엄격한 정의는 ECMAScript(ES) 모듈 스코프에 정의된 상수 또는 변수다.
> 이 책에서는 엄격한 정의를 따르지는 않을 것이다. 여기서는 단순하게 모듈 상태는 전역적이거나 파일의 스코프 내에서 정의된 변수라고 가정한다.

## 모듈 상태 살펴보기
- 모듈 상태는 모듈 수준에서 정의된 변수다.
- 여기서 모듈은 ES 모듈 또는 파일을 의미한다.

```javascript
export const createContainer = (initialState) => {
    let state = initialState;
    const getState = () => state;
    const setState = (newState) => {
        state = typeof nextState === 'function' ? newState(state) : newState;
    }
    
    return { getState, setState };
}

import { createContainer } from './createContainer';

const { getState, setState } = createContainer({ count: 0 });
```

## 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법
- 리액트 컴포넌트에서 모듈 상태를 사용하려면 리렌더링 최적화를 직접 처리 해야한다.
- codeSandbox 링크 : https://codesandbox.io/p/devbox/9rn63t?file=%2Fsrc%2F04.naive_solution_to_module_state.tsx%3A41%2C11
- count를 구독하는 컴포넌트에 최신 모듈 상태를 표시 하기 위해서 컴포넌트 외부에서 setState를 관리하도록 컴포넌트 생명주기에 따라 useEffect을 이용해 setState를 리액트 외부에 Set과 같은 별도의 자료구조에 추가해서 관리하도록 하였다.
- 좋은 해결책은 아닌것 같다. 중복 코드가 있고 컴포넌트가 추가될수록 계속해서 늘어난다.

## 기초적인 구독 추가하기
- 구독은 갱신에 대한 알림을 받기 위한 방법이다.

```javascript
const unsubscribe = store.subscribe(() => {
    console.log('store is updated');
});
```
- store에 subscribe 메서드가 있고, 이 메서드는 callback 함수를 받고 unsubscribe 함수를 반환한다.
- 
