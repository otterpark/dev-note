# useReduce()

```tsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

`useReduce()`는 컴포넌트에 `reducer`를 추가할 수 있는 React Hook입니다.

## 1. useReducer(reducer, initialArg, init?)

컴포넌트의 최상위 레벨에서 `useReduce()`를 호출하여 `Reduce`를 통해 상태를 관리하세요.

```tsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

### Parameter

- `reducer` - 상태가 업데이트되는 방식을 지정하는 `reducer` 함수입니다. 순수 함수여야 하며 `state`와 `action`을 인자로 받고 다음 상태를 반환해야 합니다. `state`와 `action`은 어떤 유형이든 가능합니다.
- `initialArg` - 초기 상태가 계산되는 값입니다. 모든 유형의 값일 수 있습니다. 초기 상태가 계산되는 방법은 다음 초기화 인자에 따라 달라집니다.
- `선택적 init`: 초기 상태를 반환해야 하는 이니셜라이저 함수입니다. 지정하지 않으면 초기 상태가 `initialArg`로 설정됩니다. 그렇지 않으면 초기 상태는 `init(initialArg)`를 호출한 결과로 설정됩니다.

### Return

`useReducer`는 정확히 두 개의 값을 가진 배열을 반환합니다.

1. 현재 `State`이며, 첫 번째 렌더링 중에는 `init(initialArg)` 또는 `초기값이 없는 경우 initialArg`로 설정됩니다.
2. `dispatch 함수`는 `State`를 다른 값으로 업데이트하고 리렌더링을 트리거할 수 있게 해줍니다.

### 주의사항

#### 💡 `useReducer`는 Hook이므로 컴포넌트의 최상위 수준이나 자체 Hook에서만 호출할 수 있습니다

**루프나 조건 내부에서는 호출할 수 없습니다**. 필요하다면 새 컴포넌트를 추출하고 상태를 그 안으로 옮기세요.

#### 💡 `StrictMode`에서 React는 실수로 발생한 불순물을 찾기 위해 `Reducer`와 `initializer`를 두 번 호출합니다

이는 개발 전용 동작이며 프로덕션에는 영향을 미치지 않습니다. `Reducer`와 `initializer`가 순수하다면 로직에 영향을 미치지 않습니다. 호출 중 하나의 결과는 무시됩니다.

## 2. dispatch function

`useReducer`가 반환하는 `dispatch 함수`를 사용하면 상태를 다른 값으로 업데이트하고 다시 렌더링을 트리거할 수 있습니다. `dispatch 함수`에 유일한 인수로 액션을 전달해야 합니다.

```tsx
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

React는 현재 상태와 `dispatch`에 전달한 액션과 함께 제공한 `reducer` 함수를 호출한 결과로 다음 상태를 설정합니다.

### Parameters

- `action` - 사용자가 수행한 작업입니다. 모든 유형의 값이 될 수 있습니다. 일반적으로 작업은 일반적으로 이를 식별하는 유형 속성이 있는 객체이며, 선택적으로 추가 정보가 있는 다른 속성을 포함할 수도 있습니다.

### Returns

반환 값이 없다.

### 주의사항

#### 💡 dispatch 함수는 다음 렌더링에 대한 상태 변수만 업데이트합니다

`dispatch 함수`를 호출한 후 상태 변수를 읽으면 호출 전 화면에 있던 이전 값을 그대로 가져옵니다.

#### 💡 만약 여러분이 제공한 새 값이 `Object.is` 비교에 의해 결정된 현재 상태와 동일하다면, React는 컴포넌트와 그 자식들을 리렌더링하는 과정을 생략합니다. 

이것은 최적화입니다. React는 여전히 결과를 무시하기 전에 컴포넌트를 호출해야 할 수도 있지만 코드에 영향을 미치지는 않습니다.

#### 💡 React는 상태 업데이트를 일괄 처리합니다

모든 이벤트 핸들러가 실행되고 설정된 함수를 호출한 후에 화면을 업데이트합니다. 이렇게 하면 단일 이벤트 중에 여러 번 리렌더링하는 것을 방지할 수 있습니다. 드물지만 `DOM`에 접근하기 위해 React가 화면을 더 일찍 업데이트하도록 강제해야 하는 경우, `flushSync`를 사용할 수 있습니다.

## 사용법

### 컴포넌트에 `reducer` 추가하기

```tsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

`useReducer`는 정확히 두 개의 항목이 있는 배열을 반환합니다.

1. 현재 상태가 처음에 제공한 초기 상태로 설정된다.
2. 상호작용에 반응하여 이를 변경할 수 있는 `dispatch 함수`.

화면의 내용을 업데이트하려면 사용자가 수행한 작업, 즉 액션을 나타내는 객체로 `dispatch를 호출`합니다.

```tsx
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

React는 현재 `state`와 `action`을 `reducer` 함수에 전달합니다. `reducer`는 다음 상태를 계산하고 반환합니다. React는 다음 상태를 저장하고, 컴포넌트를 렌더링하고, UI를 업데이트합니다.

`useReducer`는 `useState`와 매우 유사하지만 이벤트 핸들러의 상태 업데이트 로직을 컴포넌트 외부의 단일 함수로 옮길 수 있습니다.

### `reducer` 작성하기

`reducer` 선언 이후 다음 상태를 계산하고 반환할 코드를 입력해야 합니다. 관례상 스위치 문으로 작성하는 것이 일반적입니다. 스위치의 각 경우에 대해 다음 상태를 계산하고 반환합니다.

```tsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

`action`은 어떤 형태든 가질 수 있습니다. 일반적으로 액션을 식별하는 타입 프로퍼티가 있는 객체를 전달하는 것이 일반적입니다. 여기에는 `Reducer`가 다음 상태를 계산하는 데 필요한 최소한의 필수 정보가 포함되어야 합니다.

```tsx
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
  
  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

`Action` 유형 이름은 컴포넌트에 로컬로 지정됩니다. 각 액션은 데이터의 여러 변경으로 이어지더라도 하나의 상호작용을 설명합니다. 상태의 모양은 임의적이지만 일반적으로 객체나 배열이 됩니다.

> 💡 주의사항
>
> 상태는 읽기 전용이므로, 상태의 객체나 배열을 직접 수정하지마세요.
> ```tsx
> function reducer(state, action) {
>    switch (action.type) {
>      case 'incremented_age': {
>        // ✅ Instead, return a new object
>        return {
>          ...state,
>          age: state.age + 1
>        };
>      }
> ```

### `initial state` 다시 생성하지 않기

React는 초기 상태를 한 번 저장하고 다음 렌더링에서 이를 무시합니다.

```tsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

`createInitialState(username)`의 결과는 초기 렌더링에만 사용되지만 여전히 모든 렌더링에서 이 함수를 호출하고 있습니다. 이는 큰 배열을 생성하거나 복잡한 계산을 수행하는 경우 낭비가 될 수 있습니다.

이 문제를 해결하려면 이 함수를 `initializer 함수`로 전달하고 대신 세 번째 인수로 `useReducer`를 전달할 수 있습니다.

```tsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

함수를 호출한 결과인 `createInitialState()`가 아니라 함수 자체인 `createInitialState`를 전달하고 있음을 알 수 있습니다. 이렇게 하면 초기화 후 초기 상태가 다시 생성되지 않습니다.

위의 예제에서 `createInitialState`는 사용자 이름 인수를 받습니다. `initializer`가 초기 상태를 계산하는 데 아무런 정보가 필요하지 않은 경우, `useReducer`의 두 번째 인수로 null을 전달할 수 있습니다.

## 문제 해결

### 작업을 발송했지만 로깅하면 이전 상태 값이 표시됩니다

`dispatch 함수`를 호출해도 실행 중인 코드의 상태는 변경되지 않습니다.

```tsx
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // Request a re-render with 43
  console.log(state.age);  // Still 42!

  setTimeout(() => {
    console.log(state.age); // Also 42!
  }, 5000);
}
```

상태는 스냅샷처럼 동작하기 때문입니다. 상태를 업데이트하면 새 상태 값으로 다른 렌더링을 요청하지만 이미 실행 중인 이벤트 핸들러의 상태 JavaScript 변수에는 영향을 미치지 않습니다.

다음 상태 값을 추측해야 하는 경우 `reducer`를 직접 호출하여 수동으로 계산할 수 있습니다.

```tsx
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

### 작업을 발송했지만 화면이 업데이트되지 않습니다

`Object.is` 비교에 의해 결정된 대로 다음 상태가 이전 상태와 같으면 React는 업데이트를 무시합니다. 이는 보통 객체나 배열의 상태를 직접 변경할 때 발생합니다.

```tsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 잘못된 state 변경
      state.age++;
      return state;
    }
    case 'changed_name': {
      // 🚩 잘못된 state 변경
      state.name = action.nextName;
      return state;
    }
    // ...
  }
}
```

기존 state **객체를 변경하고 반환했기 때문에 React가 업데이트를 무시**했습니다. 이 문제를 해결하려면 항상 상태 객체를 업데이트하고 상태 배열을 변경하는 대신 상태 객체를 업데이트해야 합니다.

```tsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅
      return {
        ...state,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      // ✅
      return {
        ...state,
        name: action.nextName
      };
    }
    // ...
  }
}
```

### 내 `reducer` 상태의 일부가 디스패치 후 정의되지 않습니다

새 상태를 반환할 때 모든 `case` 브랜치가 기존 필드를 모두 복사하는지 확인하세요.

```tsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // 꼭 까먹지 말아야한다.
        age: state.age + 1
      };
    }
    // ...
```

### 디스패치 후 전체 `Reducer` 상태가 정의되지 않습니다

상태가 예기치 않게 정의되지 않은 경우, `case` 중 하나에서 상태를 반환하는 것을 잊었거나 `action` 유형이 케이스 문 중 어느 것과도 일치하지 않을 수 있습니다. 이유를 찾으려면 스위치 외부에서 오류를 발생시키세요.

```tsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ...
    }
    case 'edited_name': {
      // ...
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

> `TypeScript`와 같은 정적 유형 검사기를 사용하여 이러한 실수를 포착할 수도 있습니다.

### 오류가 발생했습니다: "재렌더링 횟수가 너무 많습니다"

다음과 같은 오류가 발생할 수 있습니다. `재렌더링 횟수가 너무 많습니다. React는 무한 루프를 방지하기 위해 렌더링 횟수를 제한합니다.` 일반적으로 이것은 렌더링 중에 무조건 `Action`을 `Dispatch`한다는 것을 의미하므로 컴포넌트는 렌더링, `Dispatch`(렌더링을 유발), 렌더링, `Dispatch`(렌더링을 유발) 등의 루프에 진입하게 됩니다. 이벤트 핸들러를 지정하는 과정에서 실수로 인해 발생하는 경우가 많습니다.

```tsx
// 🚩 렌더링 중에 핸들러를 호출합니다.
return <button onClick={handleClick()}>Click me</button>

// ✅ 이벤트 핸들러를 전달합니다.
return <button onClick={handleClick}>Click me</button>

// ✅ 이벤트 핸들러를 전달합니다.
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

이 오류의 원인을 찾을 수 없는 경우 콘솔에서 오류 옆에 있는 화살표를 클릭하고 자바스크립트 스택을 살펴보고 오류의 원인이 되는 특정 `Dispatch` 함수 호출을 찾아보세요.

