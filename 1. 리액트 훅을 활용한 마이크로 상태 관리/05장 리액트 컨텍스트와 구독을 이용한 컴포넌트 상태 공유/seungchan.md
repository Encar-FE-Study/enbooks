# 05. 리액트 컨텍스트와 구독을 이용한 컴포넌트 상태 공유

## 1. 모듈 상태(module state)
```
정의 : 특정 파일(모듈)내에서 정의된 변수나 상태를 의미
특징: 모듈이 처음 로드될 때 한 번만 생성 및 평가된다.또한 모듈별로 Scope를 가진다(전역변수와의 차이)
과정: 모듈이 Import 될때, 모듈 로딩 -> 모듈 캐싱의 과정이 거침 이후 동일한 모듈이 다시 Import되어도 JS 엔진은 캐시된 모듈을 재사용
- 따라서 모듈안의 상태(변수)가 다시 실행되지 않는다
```


## 모듈 상태의 변수



```javascript
ex1)
import { createStore } from 'some-store-library';

const store = createStore({ count: 0 });

export const Counter = () => {
  // ...
};

```

```javascript
ex2)
// moduleA.js
export const store = { count: 0 };

// moduleB.js
import { store } from './moduleA.js';

store.count += 1;
console.log(store.count); // 1

// moduleC.js
import { store } from './moduleA.js';

console.log(store.count); // 1 (이미 moduleB에서 증가됨)

moduleA.js의 store는 한 번만 생성되고, moduleB.js와 moduleC.js에서 동일한 store를 공유

```

## 모듈 상태의 함수
```
 모듈 내에서 정의된 함수는 모듈이 로드될 때 한 번만 정의된다.
함수 자체는 한 번만 생성되지만, 해당 함수를 여러 번 호출할 수 있다.

```

```javascript
// storeModule.js
let count = 0;

export const increment = () => {
  count += 1;
  return count;
};



....


// componentA.js
import { increment } from './storeModule.js';

console.log(increment()); // 1
console.log(increment()); // 2

// componentB.js
import { increment } from './storeModule.js';

console.log(increment()); // 3

increment 함수는 모듈이 로드될 때 한 번만 정의되지만, 여러 번 호출되면서 count가 증가함. 
이는 count가 모듈 전체에서 공유되는 상태임을 보여줌

모듈 상태가 한 번만 평가되는 이유
모듈 캐싱: 자바스크립트 모듈 시스템은 모듈을 한 번만 로드하고 평가
동일한 모듈이 여러 번 임포트되더라도, 처음 로드된 결과가 캐시되어 재사용
모듈 스코프: 각 모듈은 자체적인 스코프를 가지므로, 전역 변수와는 달리 모듈 내부에서만 접근 가능한 변수를 정의
코드의 캡슐화(encapsulation)를 강화하여,변수 충돌을 방지

```


## 해결

문제상황
```javascript
// storeModule.js
let count = 0; -> 모듈 안에서 전역 변수의 역할
count는 모듈의 최상위 스코프에 정의된 변수로, 모든 Counter 컴포넌트 인스턴스가 동일한 count를 공유

export const Counter = () => {
  const inc = () => {
    count += 1;
    console.log(count);
  };

  return `<button onclick="inc()">+1</button>`;
};


```

해결
```javascript
// counterComponent.js
export const Counter = () => {
  let count = 0; // 함수 내부 변수

Counter 함수 호출 시마다 독립적인 count가 생성
따라서 여러 Counter 인스턴스가 있어도 각각의 count는 독립적으로 존재

  const inc = () => {
    count += 1;
    console.log(count);
  };

  return `<button onclick="inc()">+1</button>`;
};


```


## 요약
```
1. 모듈 변수는 일종의 전역 변수 역할을 하며, 이를 사용하는 곳에서 값이 공유되어 문제가 발생한다.
2. 이를 해결하기 위해 모듈 변수를 함수 내부로 옮기면, 함수 실행 시마다 새로운 실행 컨텍스트가 생성되어 각 함수에서 선언한 State(변수)가 Context를 가지게 되어 문제가 해결된다.
```

