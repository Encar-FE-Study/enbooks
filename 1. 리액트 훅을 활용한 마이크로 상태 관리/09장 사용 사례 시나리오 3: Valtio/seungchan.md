## Valtio 특징
- useSnapShot(state) 실행시마다 새로운 불변객체를 생성함
- 이 값은 원본 객체와 완전히 분리됨
- 즉, 직접적인 상태 수정이 가능하며 내부적으로 불변성을 보장한다.
- redux나 immer 같은 추가 라이브러리 필요 X
- 이것은 Proxy의 사용 덕분이다.
  
```javascript
const state = proxy({
  count: 0,
  user: { name: 'John' }
})
function Component() {
  // 매 렌더링마다 새로운 객체(스냅샷)를 생성
  const snap = useSnapshot(state)
  
  // snap은 원본 state와 다른 새로운 참조를 가진 객체
  console.log(snap === state) // false
  console.log(snap.user === state.user) // false
}
```
## Proxy 패턴의 사용 이유
- 원본 객체의 동작을 Proxy로 감싸서 가로채어 원하는 로직을 추가하기 위한 용도
- 객체의 불변성을 보장하며, 객체 속성값 직접 수정이 가능

```javascript
// 기본적인 Proxy 동작 예시
const state = proxy({
  count: 0,
  nested: { value: 1 }
});

// Proxy의 set trap을 통해 count의 증가가감지됨
state.count++;

```


# 객체의 속성값에 접근해서 값을 바꿨을 뿐인데 어떻게 이 변경을 추적하고 새로운 객체가 스냅샷에 기록되나?

이는 Proxy api가 제공하는 deep proxy기능 덕분
## Deep proxy 동작방식

```javascript
// Proxy의 기본적인 동작을 보여주는 단순화된 예시
// 실제 Valtio 내부에서는 이런 식으로 동작
// 아래 동작들을 trap이라 부름 (객체의 특정 동작을 가로채기(trap))하기 때문
function createDeepProxy(target) {
  return new Proxy(target, {
    get(target, prop) {
      // 1. 객체의 속성에 접근할 때 호출됨
      const value = target[prop];
      
      // 2. 가져온 값이 객체라면 그것도 다시 proxy로 감싸줌
      if (typeof value === 'object' && value !== null) {
        return createDeepProxy(value);
      }
      
      return value;
    },
    
    set(target, prop, value) {
      // 3. 값을 설정할 때 호출됨
      target[prop] = value;
      // 4. 변경 사실을 알림
      notifyUpdate();
      return true;
    }
  });
}

```


## 실제동작과정
```javscript
// 1. 초기 상태 생성
const state = proxy({
  nested: { value: 1 }
});

// 내부적으로 이런 구조가 됨:
// state = Proxy({
//   nested: Proxy({ value: 1 })  // nested도 proxy로 감싸짐
// })

// 2. 값 변경 시의 동작
state.nested.value++;

// 실제로 일어나는 일:
// a. state.nested 접근 -> get trap 발동
// b. nested 객체에 대한 proxy 반환
// c. .value 접근 -> nested proxy의 get trap 발동
// d. value++ 실행 -> nested proxy의 set trap 발동

```


## 전체흐름

```javscript
// 실제 Valtio 내부 동작을 단순화한 예시
const subscriptions = new Set();

function createStateProxy(target) {
  // 이미 처리된 객체 추적
  const handled = new WeakMap();
  
  function makeHandler(path = []) {
    return {
      get(target, prop) {
        const value = target[prop];
        
        if (typeof value === 'object' && value !== null) {
          // 이미 proxy로 감싸진 객체인지 확인
          if (handled.has(value)) {
            return handled.get(value);
          }
          
          // 새로운 proxy 생성
          const newProxy = new Proxy(value, makeHandler([...path, prop]));
          handled.set(value, newProxy);
          return newProxy;
        }
        
        return value;
      },
      
      set(target, prop, value) {
        const oldValue = target[prop];
        target[prop] = value;
        
        // 값이 실제로 변경됐을 때만 알림
        if (oldValue !== value) {
          // 모든 구독자에게 변경 알림
          subscriptions.forEach(fn => fn());
        }
        
        return true;
      }
    };
  }
  
  return new Proxy(target, makeHandler());
}

// 사용 예시
const state = createStateProxy({
  nested: { value: 1 }
});

// 컴포넌트에서의 사용
function useMySnapshot(proxy) {
  const [, forceUpdate] = useState({});
  
  useEffect(() => {
    // 상태 변경 구독
    const callback = () => forceUpdate({});
    subscriptions.add(callback);
    
    return () => subscriptions.delete(callback);
  }, []);
  
  // 현재 상태의 스냅샷 생성
  return JSON.parse(JSON.stringify(proxy));
}
```

1. 즉, 모든 객체의 속성이 자동으로 Proxy로 감싸짐
2. 어떤 깊이에서든 속성 변경이 감지됨 (getTrap)
3. 변경이 감지되면 새로운 스냅샷이 생성됨 (setTrap)
4. 구독중인 컴포넌트들 업데이트 (setTrap+구독)

## Proxy 역할
- getTrap: 깊은 객체 구조에서도 모든 객체를 proxy로 감싸서 변경 감지 - 값에 접근시 사용 및 발동됨
- setTrap 실제 값 변경을 감지하고 구독자들에게 알림을 보냄 - 값을 할당시 사용 및 발동됨
- 구독 시스템: set trap에서 발생한 알림을 받아서 스냅샷을 생성하고 컴포넌트를 업데이트


### Trap 예시
```javascript
// 일반적인 객체
const obj = { value: 1 };
obj.value = 2; // 그냥 할당

// Proxy를 사용한 객체
const proxy = new Proxy({ value: 1 }, {
  set(target, prop, value) {  // set trap - 동작을 가로챔
    console.log(`값이 ${value}로 변경되려고 합니다!`);
    target[prop] = value;
    return true;
  }
});

proxy.value = 2; // "값이 2로 변경되려고 합니다!" 출력
```

```javascript
const handler = {
  // 1. get trap: 속성에 접근할 때
  get(target, prop) {
    console.log(`${prop} 속성에 접근`);
    return target[prop];
  },

  // 2. set trap: 속성에 값을 할당할 때
  set(target, prop, value) {
    console.log(`${prop}에 ${value} 할당`);
    target[prop] = value;
    return true;
  },

  // 3. deleteProperty trap: 속성 삭제할 때
  deleteProperty(target, prop) {
    console.log(`${prop} 속성 삭제`);
    delete target[prop];
    return true;
  },

  // 4. has trap: in 연산자 사용할 때
  has(target, prop) {
    console.log(`${prop} in 객체 확인`);
    return prop in target;
  }
};

const obj = { name: 'John' };
const proxy = new Proxy(obj, handler);

// 각 동작에 따른 trap 발동
proxy.name;          // get trap 발동: "name 속성에 접근"
proxy.name = 'Jane'; // set trap 발동: "name에 Jane 할당"
delete proxy.name;   // deleteProperty trap 발동: "name 속성 삭제"
'name' in proxy;     // has trap 발동: "name in 객체 확인"


const handler = {
  get(target, prop) {
    console.log(`get: ${prop}`);
    const value = target[prop];
    return value;
  },
  set(target, prop, value) {
    console.log(`set: ${prop} = ${value}`);
    target[prop] = value;
    return true;
  }
};

const state = new Proxy({ nested: { value: 1 } }, handler);

// 중첩된 속성 변경시 trap 발동 순서
state.nested.value = 2;
// 출력:
// 1. "get: nested" (nested 속성 접근)
// 2. "set: value = 2" (value 속성에 값 할당)


주의

const state = proxy({
  value: 1,
  nested: { deep: 2 }
});

// 이런 경우는 trap이 발동하지 않음
const { nested } = state;  // 구조분해 후에는 proxy 체인이 끊어짐
nested.deep = 3; 

// 대신 이렇게 해야 trap이 정상 발동
state.nested.deep = 3;    // proxy 체인 유지
```

