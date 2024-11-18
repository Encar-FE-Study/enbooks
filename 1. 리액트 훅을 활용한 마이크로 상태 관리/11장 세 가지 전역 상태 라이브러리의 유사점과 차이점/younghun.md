# 세 가지 전역 상태 라이브러리의 유사점과 차이점

## Zustand와 Redux의 차이점
- 상태 갱신 방법
    - 리듀서 사용 : Redux
- 디렉토리 구조
    - feature 디렉터리 사용 유무 : Redux
- Immer 사용(Redux)
- 상태 전파 방법
    - 모듈 임포트 : Zustand
    - Context 사용 : Redux
- 데이터 흐름
    - 단방향 : Redux
    - 개발자 선택 : Zustand

## Jotai와 Recoil을 사용하는 시점
- key 문자열 
    - Key 문자열 사용 : Recoil, 직렬화 가능
    - WeakMap 사용 : Jotai
- atom 함수
    - 통합된 atom 함수(atom, selector 대체) : Jotai
- provider-less mode

## Valtio와 Mobx 사용하기
- 렌더링 최적화하는 방법의 차이
    - Hooks : Valtio, 동시성 렌더링에 친화적
    - HoC : Mobx, 옵저버 방식, 예측 가능성
- 갱신 방식
    - 객체 기반 : Valtio
    - 클래스 기반 : Mobx

## Zustand, Jotai, Valtio 비교하기
- Poimandres
    - 작은 API 제공
    
- 상태의 위치
    - 모듈 상태 : Zustand, Valtio
    - 컴포넌트 상태 : Jotai
- 상태 갱신 스타일
    - 불변 상태 모델 기반 : Zustand 
    - 변경 가능 상태 모델 기반: Valtio