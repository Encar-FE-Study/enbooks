# 09장. 사용 사례 시나리오 4: React Tracked

## React Tracked 이해하기
- 상태 관리 기능을 제공하지는 않지만 상태 사용 추적(proxy)을 통한 렌더링 체적화 기능 제공
- proxy-compare 라이브러리 사용
- use-context-selector 라이브러리 사용

## 향후 전망
- React Tracked 사용법 두 가지
    - 리액트 컨텍스트에서 createContainer 사용 
        - use-context-selector & createTrackedSelector로 구현
        - 리액트 컨텍스트 사용 대체 목적 
    - React Redux에서 createTrackedSelector 사용
        - proxy-compare로 구현
