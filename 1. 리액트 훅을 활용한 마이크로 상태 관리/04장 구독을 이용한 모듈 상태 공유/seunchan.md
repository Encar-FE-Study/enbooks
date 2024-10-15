# 왜 context는 구독 패턴을 사용할까?

구독 패턴의 예시

```javascript
class Store {
  constructor() {
    this.state = { count: 0 };
    this.subscribers = [];
  }
  subscribe(callback) {
    this.subscribers.push(callback);
    return this;
  }
  unsubscribe(callback) {
    this.subscribers = this.subscribers.filter((sub) => sub !== callback);
    return this;
  }

  // Method to update state
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.notify();
  }

  notify() {
    this.subscribers.forEach((callback) => callback(this.state));
  }
}

const store = new Store();

store.setState({ count: 1 }); // Logs: State updated: { count: 1 }
store.setState({ count: 2 }); // Logs: State updated: { count: 2 }

store.setState({ count: 3 });
```

장점:
반응성: 상태가 변경되면 구성 요소가 자동으로 업데이트
디커플링: 구독을 통해 문제를 분리할 수 있고, 해당 변화를 수신 가능
유연성: 구독자를 쉽게 추가하거나 제거할 수 있으므로 유연한 변경이 가능
단점:
성능 오버헤드: 제대로 관리하지 않으면 잦은 업데이트로 인해 성능 문제가 발생할 수 있으며, 특히 많은 구성 요소가 변경 사항을 구독시 관리가 힘듦
디버깅 어려움: 데이터 흐름을 이해하고 구독과 관련된 문제를 추적하는 것이 어려움, 데이터가 단방향이 아니라면? 그래서 리덕스가 단방향 데이터 흐름을 가지는 구나.

React.context가 왜 구독 패턴을 쓰는지

- 변경에 기민하게 대응이 가능
- 유연한 값의 추가와 제거 가능
  단점
- 구독 패턴과 마찬가지로 최적화가 힘들다
