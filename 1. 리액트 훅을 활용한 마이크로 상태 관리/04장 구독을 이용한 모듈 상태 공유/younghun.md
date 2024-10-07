# 04 구독을 이용한 모듈 상태 공유

## 모듈 상태 살펴보기
- 모듈 : ES 모듈 혹은 파일을 의미

## 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법
- 모듈 상태를 일치시키는 방법
컴포넌트 생명주기를 고려하며 컴포넌트 외부에 있는 모듈 수준에서 setState를 관리할 필요가 있음

## 기초적인 구독 추가하기
- subscribe를 통해 각 컴포넌트에서 상태 싱크
```js
const useStore = (store) => {
    const [state, setState] = useState(store.getState());

    useEffect(()=> {
        const unsubscribe = store.subscribe(()=> {
            setState(store.getState())
        })
        setState(store.getState())
        return unsubscribe;
    },[store])

    return [state, store.setState]
}
```

## 선택자와 useSubscription 사용하기
- 위의 useStore와 다르게 특정 값만 선택하는 useStoreSelector 훅
useEffect 실행 시점에 따라 재구독 될때까지는 갱신되기 이전 상태 값을 반환하는 이슈
use-subscription이라는 공식적인 훅 제공(https://www.npmjs.com/package/use-subscription)

