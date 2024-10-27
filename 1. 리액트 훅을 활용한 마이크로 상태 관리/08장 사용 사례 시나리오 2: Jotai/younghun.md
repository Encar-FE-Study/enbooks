# 08장. 사용 사례 시나리오 1: Jotai

## Context 비교한 두 가지 이점
- 구문 단순성
useAtom 훅 사용
- 동적 아톰 생성
Jotai 스토어는 아톰 구성 객체와 아톰 값으로 구성된 WeakMap 객체

## 렌더링 최적화
상향식 접근법
파생아톰

## 배열 구조 추가하기(Atoms in Atom)

## Jotai의 다양한 기능 사용하기
- 아톰의 write 함수 정의하기
```js
const anotherCountAtom = atom(
    (get) => get(countAtom),
    (get, set, arg) => {
        const nextCount.= typeof arg === 'function' ? 
            arg(get(countAtom)) : arg
        set(countAtom, nextCount)
        console.log('set count', nextCount)
    }
)
```

- 액션 아톰 사용하기
```js
const incrementCountAtom = atom(
    null,
    (get, set, arg) => set(countAtom, (c) => c+1)
)
```

- 아톰의 onMount 옵션 이해하기
```js
const countAtom = atom(0);
countAtom.onMount = (setCount) => {
    console.log('count atom 사용을 시작합니다');
    const onUnmount = () => {
        console.log('count atom 사용이 끝났습니다.)
    }
    return onUnmount;
}
```

- 이 외 기능들
jotai/utils, scope 개념, React Suspense 기능 지원

- [jotai 활용 앱](https://github.com/youngrove/client-state-packages)

