# useRef()

`useRef`는 **렌더링에 필요하지 않은 값을 참조**할 수 있는 React Hook입니다.

```tsx
const ref = useRef(initialValue)
```

## Reference

### `useRef(initialValue)`

컴포넌트의 최상위 레벨에서 `useRef`를 호출하여 참조를 선언합니다.

```tsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

### Parameter

- `initialValue`: 참조 객체의 현재 프로퍼티를 처음에 설정할 값입니다. 모든 유형의 값일 수 있습니다. 이 인수는 초기 렌더링 이후에는 무시됩니다.

### Return

`useRef`는 단일 프로퍼티를 가진 객체를 반환합니다

- `current`: 처음에는 전달한 `initialValue`로 설정됩니다. 나중에 다른 값으로 설정할 수 있습니다. `ref` 객체를 JSX 노드에 대한 `ref` 어트리뷰트로 React에 전달하면 React가 현재 프로퍼티를 설정합니다.

다음 렌더링에서 useRef는 동일한 객체를 반환합니다.

### 주의사항

#### 💡 ref.current 속성을 변경할 수 있습니다

state와 달리 변경할 수 있습니다. 하지만 렌더링에 사용되는 객체(예: state의 일부)를 담고 있다면 해당 객체를 변경해서는 안 됩니다.

#### 💡 ref.current 프로퍼티를 변경해도 React는 컴포넌트를 다시 렌더링하지 않습니다

`ref`는 일반 자바스크립트 객체이기 때문에 React는 사용자가 언제 변경했는지 알지 못합니다.

#### 💡 초기화를 제외하고는 렌더링 중에 `ref.current`를 쓰거나 읽지 마세요

이렇게 하면 컴포넌트의 동작을 예측할 수 없게 됩니다

#### 💡 `StrictMode`에서는 실수로 발생한 불순물을 찾기 위해 React가 컴포넌트 함수를 두 번 호출합니다

이는 개발 전용 동작이며 프로덕션에는 영향을 미치지 않습니다. 각 `ref` 객체는 두 번 생성되지만 버전 중 하나는 버려집니다. 컴포넌트 함수가 순수하다면(그래야만 하는 것처럼) 동작에 영향을 미치지 않습니다.

## 사용법

### 참조로 값 참조하기

컴포넌트의 최상위 레벨에서 `useRef`를 호출하여 하나 이상의 참조를 선언하세요.

```tsx
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

`useRef`는 **처음에 제공한 초기 값**으로 설정된 단일 현재 프로퍼티가 있는 참조 객체를 반환합니다.

다음 렌더링에서 `useRef`는 동일한 객체를 반환합니다. 현재 프로퍼티를 변경하여 정보를 저장하고 나중에 읽을 수 있습니다. `state`를 떠올리게 할 수도 있지만 중요한 차이점이 있습니다.

**참조를 변경해도 재렌더링이 트리거**되지 않습니다. 즉, `ref`는 컴포넌트의 시각적 출력에 영향을 주지 않는 정보를 저장하는데 적합합니다. 예를 들어 `interval ID`를 저장했다가 나중에 검색해야 하는 경우 참조에 넣을 수 있습니다. 참조 내부의 값을 업데이트하려면 현재 프로퍼티를 수동으로 변경해야 합니다.

```tsx
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

나중에 해당 `interval` ID를 읽어서 `clear` 해줘야할 때  해당 `interval` ID 지우는 함수를 호출할 수 있습니다.

```tsx
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

참조를 사용하면 다음을 보장할 수 있습니다.

- 렌더링할 때마다 **재설정되는 일반 변수와 달리 리렌더링 사이에 정보를 저장**할 수 있습니다.
- 변경해도 **재렌더링이 트리거**되지 않습니다(재렌더링을 트리거하는 상태 변수와 달리).
- 이 정보는 공유되는 외부 변수와 달리 컴포넌트의 각 복사본에 로컬로 적용됩니다.

참조를 변경해도 리렌더링이 트리거되지 않으므로 화면에 표시하려는 정보를 저장하는 데는 참조가 적합하지 않습니다. 대신 state를 사용하세요. 사용 참조와 사용 상태 중에서 선택하는 방법에 대해 자세히 알아보세요.

### 함정

렌더링 중에는 `ref.current`를 쓰거나 읽지 마세요.

React는 컴포넌트의 본문이 순수한 함수처럼 동작할 것으로 기대합니다.

- `input(props, state, 컨텍스트)`이 동일하다면 정확히 동일한 JSX를 반환해야 합니다.
- 다른 순서로 호출하거나 다른 인수를 사용하여 호출해도 다른 호출 결과에 영향을 미치지 않아야 합니다.

렌더링 중에 참조를 읽거나 쓰면 이러한 기대가 깨집니다.

```tsx
function MyComponent() {
  // ...
  // 🚩 렌더링 중에 참조를 작성하지 마세요.
  myRef.current = 123;
  // ...
  // 🚩 렌더링 중에 참조를 작성하지 마세요.
  return <h1>{myOtherRef.current}</h1>;
}
```

대신 `이벤트 핸들러`나 `Effect`에서 레퍼런스를 읽거나 쓸 수 있습니다.

```tsx
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ 이펙트에서 레퍼런스를 읽거나 쓸 수 있습니다.
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ 이벤트 헨들러에서 레퍼런스를 읽거나 쓸 수 있습니다.
    doSomething(myOtherRef.current);
  }
  // ...
}
```

렌더링 중에 무언가를 읽거나 써야 하는 경우 state를 대신 사용하세요.

이러한 규칙을 어겨도 컴포넌트는 여전히 작동할 수 있지만, React에 추가되는 대부분의 새로운 기능은 이러한 기대치에 의존합니다.

### `ref`로 DOM 조작하기

특히 `ref`를 사용해 **DOM을 조작**하는 것이 일반적입니다. React는 이를 기본적으로 지원합니다.

먼저 초기값이 `null`인 참조 객체를 선언합니다.

```tsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

그런 다음 `ref` 객체를 참조 어트리뷰트로 조작하려는 DOM 노드의 JSX에 전달합니다.

```tsx
// ...
return <input ref={inputRef} />;
```

React가 DOM 노드를 생성하고 화면에 배치하면 React는 `ref` 객체의 현재 프로퍼티를 해당 DOM 노드로 설정합니다. 이제 `<input>`의 DOM 노드에 접근하여 `focus()`와 같은 메서드를 호출할 수 있습니다.

```tsx
function handleClick() {
  inputRef.current.focus();
}
```

React는 노드가 화면에서 제거되면 현재 프로퍼티를 다시 `null`로 설정합니다.

### `ref` 콘텐츠 다시 생성하지 않기

React는 초기 `ref` 값을 한 번 저장하고 다음 렌더링에서 이를 무시합니다.

```tsx
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

새로운 `VideoPlayer()`의 결과는 초기 렌더링에만 사용되지만, 여전히 모든 렌더링에서 이 함수를 호출하게 됩니다. 이는 복잡한 객체를 생성하는 경우 낭비일 수 있습니다.

이 문제를 해결하려면 대신 다음과 같이 참조를 초기화할 수 있습니다.

```tsx
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

일반적으로 렌더링 중에 `ref.current`를 쓰거나 읽는 것은 허용되지 않습니다. 하지만 이 경우에는 결과가 항상 동일하고 초기화 중에만 조건이 실행되므로 충분히 예측할 수 있으므로 괜찮습니다.

## 문제해결

### 사용자 정의 컴포넌트에 대한 `ref`를 얻을 수 없습니다

이렇게 자신의 컴포넌트에 `ref를 전달하려고 하면

```tsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

> Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

와 같은 에러를 맞이하게 된다.

기본적으로 자체 컴포넌트는 내부의 DOM 노드에 대한 `ref`를 노출하지 않습니다.

이 문제를 해결하려면 `ref`를 가져올 컴포넌트를 찾습니다.

```tsx
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}
```

그런 다음 다음과 같이 `forwardRef`로 래핑합니다.

```tsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

그러면 부모 컴포넌트가 이를 참조할 수 있습니다.
