# React Package 구성 요소 및 용어 정의

## 구성 요소

React 프레임워크는 생각보다 크고 복잡하며, 다양한 패키지들로 구성되어 있다.

### React Core

Component를 정의하며, 다른 패키지에 의존성이 없으므로 다양한 플랫폼(브라우저, 모바일)에 올려서 사용 가능하다.

> 올린다라는 이야기는 화면에 그려주는 어떤거가 있다는 이야기인데 바로 `rerender`이다.

### Renderer

호스트(브라우저, 모바일)와 react를 연결하는 역할을 한다. 또한 `reconciler`와 `event(legacy-events)` 패키지에 대한 의존성을 가지고 있다.

> react-dom, react-native-renderer 등 호스트 환경에 의존한다.

### Event(legacy-event)

react에서 react 내부적으로 Event를 사용하기 위해서 추가적인 기능이 필요했고, `SyntheticEvent`라는 이름으로 내부적으로 개발된 이벤트 시스템을 통해 기존 웹에서 Event를 wrapping 하여 추가적인 기능을 수행할 수 있게 하였다.

### Scheduler

React는 Task 단위로 비동기로 실행한다. 예를 들어 useState에 setState 호출했을 때 바로 console.log를 찍어보면 수정한 state 값이 바로 찍히지 않고 이전 state 값이 찍히는 것을 확인할 수 있다. 비동기로 Task가 정확히 실행되는 타이밍을 `Schedular`가 알고 있다.

### Reconciler

fiber architecture에서 VDOM 재조정을 담당하며, 컴포넌트를 호출하는 곳이다.

## 용어

### Rendering

컴포넌트를 호출하여 return react element -> VDOM에 적용(재조정)하는 과정

#### 전체 과정

1. 컴포넌트 호출 return react element
2. VDOM 재조정 작업 (여기까지 렌더링)
3. rerender가 컴포넌트 정보를 DOM에 삽입(mount)
4. mount가 끝난 DOM을 브라우저가 paint

### React Element

컴포넌트 호출 시 return 하는 것(JSX -> babel을 통해 react.createElement() 호출)

컴포넌트의 정보는 아래와 같다.

- type
- key
- props
- ref

### Fiber

VDOM 안에 포함된 노드 객체(아키텍쳐와 동일하지만 서로 다르다)이며, `react element`의 내용이 DOM에 반영되기 위해, 먼저 VDOM에 추가되어야 하는데 이를 위해 확장(컴포넌트의 상태, life cycle, hook이 관리)한 객체가 `fiber`이다.
