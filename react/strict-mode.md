# React StrictMode

`JavaScript`에서 엄격 모드(`'use strict'`)가 있다. 엄격 모드로 실행 시에 `JavaScript`를 실행 시 조금 더 엄격하게 코드를 검사합니다. `React`에서도 유사한 목적으로 사용되는 `<StrictMode />`가 있습니다.
> `StrictMode`는 애플리케이션 내의 잠재적 문제를 알아내기 위한 도구이다.

리엑트 공식 문서에 따라 코드를 작성하는 것이 좋지만 바쁘거나 프로젝트 이슈로 인해 우리는 우리 방식대로 결과물을 어떻게든 빠르게 만들어 보려고 하는데 이는 느리고 불안정할 수 있다.

`StrictMode` 모드를 사용하면 리엑트가 자식 컴포넌트를 검사하고 잘못 사용된 부분들을 경고 메세지를 통해 알려준다. 경고를 해결하다 보면 우리는 `React`의 어플리케이션에 잠재된 문제를 해결할 수 있다.

- 불완전한 렌더링으로 인한 버를 찾기 추가 시간을 들여 다시 렌더링 합니다.
- 누락된 이펙트들과 발생한 버그를 찾기위해 한번 더 이펙트에 대한 검사를 실행합니다.
- 더 이상 사용하지 않은 API를 사용하는지 확인합니다.
- 모든 검사는 개발 전용으로 빌드 시 영향을 미치지 않습니다.

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

## 검사항목

### 1. 잘못 사용한 생명주기 메서드

`componentWillMount` 같이 이전 클래스 컴포넌트에서 사용했던 API 같이 `React`에서 지원 종료한 메서드를 사용하면 경고한다.

### 2. 레거시 문자열 ref 사용 여부

레거시 문자열 ref API는 여러 단점들이 있는데 이를 경고한다.

### 3. findDOMNode 사용 여부

컴포넌트 자식 중 특정 엘리먼트를 찾는 함수이다. 자식 엘리먼트는 구현 방식에 따라 달라질 수 있기 때문에 유지보수면으로 힘들어져 이를 경고한다. (직접 `ref`를 지정해서 사용 권장)

### 4. 예상치 못한 부작용(side effect)

React는 `렌더링 단계(변경사항 탐지)`와 `커밋 단계(실제 변경사항 반영)`으로 나뉘어진다.

- 커밋 단계는 빠르게 진행되며, 렌더링 단계는 느릴 수 있다.
- 렌더링 단계의 여러 생명주기 메서드들이 여러번 호출되기 때문에 메모리 누수와 같은 부작용이 없는 것이 중요하다.
- 일부 함수를 이중으로 호출하여 부작용을 예측하고 문제 되는 부분을 찾을 수 있게 돕는다.

> react는 아래 함수들을 의도적으로 이중으로 호출한다.
>
> - constructor, render, shouldComponentUpdate, getDerivedStateFromProps
> - State updater
> - useState, useMemo, useReducer

### 5. 레거시 컨텍스트 사용 여부

레거시 context API는 오류가 발생하기 쉽기 때문에 이를 경고한다.
