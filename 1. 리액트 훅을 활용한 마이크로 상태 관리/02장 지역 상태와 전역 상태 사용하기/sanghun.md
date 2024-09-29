# 02. 지역상태와 전역 상태 사용하기

- 리액트 컴포넌트는 트리 구조를 구성하며, 트리구조의 하위 컴포넌트에서 상위 컴포넌트의 상태를 사용하기 위해서는 상위에서 만든 지역상태를 하위 컴포넌트에서 사용하기만 하면 된다.
- 멀리 떨어져 있는 컴포넌트들이 공통적인 상태를 사용하고 싶을 경우 전역상태를 사용해야 한다.

## 언제 지역 상태를 사용할까?

- 리액트 컴포넌트도 자바스크립트 함수이기 때문에 순수할 수 있지만 지역상태를 사용할 경우 비순수 함수가 된다.
- 그러나 상태가 컴포넌트 내에서만 사용된다면 다른 컴포넌트에 영향을 미치지 않는다.

#### 리액트와 props

```javascript
const AddOne = ({ number }) => {
  return <div>{number + 1}</div>;
};
```

- AddOne컴포넌트는 number props를 받아 number+1을 반환하는 순수 함수(컴포넌트)이다.

#### 지역상태에 대한 useState 이해하기

```javascript
const AddBase = ({ number }) => {
  const [base, changeBase] = useState(1);
  return <div>{number + base}</div>;
};
```

- AddBase컴포넌트는 인수에 포함되지 않은 base에 의존하기 때문에 순수하지 않은 컴포넌트이다.

#### 지역상태의 한계

- AddBase컴포넌트를 예로 base를 다른 코드영역에서 변경하고 싶을 때 전역상태가 필요하다.
- 전역 상태는 컴포넌트 외부에서 컴포넌트의 동작을 제어할 때 유용하게 사용가능 하지만, 컴포넌트 동작을 예측하기 어렵게 만든다는 점에서 과하게 사용하면 안된다.

## 지역 상태를 효과적으로 사용하는 방법

#### 상태 끌어올리기(Lifting State Up)

```javascript
const Parent = () => {
  const [count, setCount] = useState(0);
  return (
    <>
      <Component1 count={count} setCount={setCount}>
      <Component2 count={count} setCount={setCount}>
    </>
  )
}
const Component1 = ({count, setCount}) => {
  return (
    ...
    <button onClick={() => setCount((c) => c+1)}>
    ...
  )
};
const Component2 = ({count, setCount}) => {
  return (
    ...
    <button onClick={() => setCount((c) => c+1)}>
    ...
  )
}
```

- 위와같이 상태 끌어올리기 방식으로 상태관리를 구현할 경우, 하위 컴포넌트 어디에서든 상태가 변경되어 상위 컴포넌트로 전달되면 하위 모든 컴포넌트가 리렌더링 되는 성능상 이슈가 있다.

#### 내용 끌어올리기(Lifting Content Up)

- 복잡한 컴포넌트 트리의 경우 상위 컴포넌트의 상태에 의존하지 않는 컴포넌트가 있을 수 있음

```javascript
const AdditionalInfo = () => <p>info</p>;
const Component1 = ({count, setCount}) => {
  return (
    ...
    <AdditionalInfo />
    ...
  )
}
```

- 위와같이 컴포넌트 구조가 생서되면 count가 변경되었을 때, count와 관련없는 AdditionalInfo 컴포넌트도 함께 리렌더링 된다.

```javascript
const Component1 = ({count, setCount, additionalInfo}) => {
  return (
    ...
    {additionalInfo}
    ...
  )
}
```

- 위와 같이 내용 끌어올리기 방식으로 작업하면 count가 변경되어도 AdditionalInfo 컴포넌트는 리렌더링 되지않는다.

## 전역상태 사용하기

#### 전역상태란?

- 상태가 하나의 컴포넌트에 속하지 않고 여러 컴포넌트에서 사용할 수 있는 상태를 전역상태라고 한다.
- 모든 컴포넌트가 의존하는 애플리케이션 차원의 지역상태는 전역상태라고 할 수 있어서 명확하게 나누기는 힘들다.

#### 언제 전역상태를 사용할까?

1. prop을 전달하는 것이 적절하지 않을 때
   - 3depth 이상의 하위 컴포넌트로 prop을 전달할 경우 props drilling 문제를 해결하기 위해 사용
2. 이미 리액트 외부에 상태가 있을 때
