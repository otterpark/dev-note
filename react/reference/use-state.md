# useState()

`useState()`는 컴포넌트에 상태 변수를 추가할 수 있는 React hook 입니다.

## Reference

```tsx
const [state, setState] = useState(initialState);
```

- 컴포넌트 최상위에서 `State`를 사용하여, 상태 변수를 선언합니다.
- `Destructuring assignment`를 사용하여, `[something, setSomething]`과 같은 상태 변수의 이름을 지정합니다.

### 1. Parameters

`Parameters`에 `initialState`는 처음 상태의 초기값을 설정하며, 모든 타입의 값일 수 있지만 `함수`에 대한 특별한 동작이 있습니다.

> `initialState`는 초기 렌더링 이후에는 무시됩니다.

#### `initialState`의 값이 `함수`라면?

`initialState`로 전달하면 초기화 함수로 취급합니다. 이 함수는 순수 함수여야하고 인수를 받지 않아야 하며, 어떤 타입의 값도 반환해야 합니다. React는 컴포넌트를 초기화할 때 초기화 함수를 호출하고 그 반환값을 초기 상태로 저장합니다.

```tsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
}
```

만약 위와 같이 호출하면 `createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만 **모든 렌더링에서 이 함수를 호출**하게 됩니다. 따라서 이는 큰 배열을 생성하거나 복잡한 계산을 수행하는 경우 낭비가 될 수 있습니다.

아래와 같이 이것을 해결할 수 있습니다.

```tsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
}
```

여기서 함수를 호출한 결과인 `createInitialTodo()`가 아니라 함수 자체인 `createInitialTodos`를 전달하고 있는 것을 주목해야 합니다. 함수 자체를 `useState`에 전달하면 React는 **초기화 중에만 함수**를 호출합니다.

### 2. Return

`useState`는 정확히 두 개의 값을 가진 배열을 반환합니다.

- 첫 번째 렌더링 시 전달한 `initialState`와 일치합니다.
- `set 함수`를 사용하면 상태를 다른 값으로 업데이트하고 재 렌더링을 트리거 할 수 있습니다.

### 3. 주의사항

#### 💡 1. `useState`는 Hook이므로 **컴포넌트의 최상위 수준이나 자체 Hook**에서만 호출할 수 있습니다

`loop(루프)`나 `conditions(조건)` 내부에는 호출할 수 없으며, 필요 시 새 컴포넌트를 추출하고 상태를 그 안에 옮겨야합니다.

#### 💡 2. `StrictMode` 사용 시 `initializer function`을 두번 호출

개발 모드에서 `StrictMode` 사용 시 실수로 발생한 불필요한 문제들을 찾기 위해 `initializer function`를 두 번 호출합니다. `initializer function`가 순수 함수라면 이러한 동작에 영향을 미치지 않습니다. 호출 중 하나의 결과는 무시됩니다.

```tsx
function TodoList() {
  // StrictMode에서는 이 컴포넌트는 매번 2번씩 렌더링됩니다.

  const [todos, setTodos] = useState(() => {
    // initialization 함수는 두번 실행된다.
    return createTodos();
  });

  function handleClick() {
    setTodos(prevTodos => {
      // 업데이터 함수도 클릭 때마다 두번 실행된다.
      return [...prevTodos, createTodo()];
    });
  }
```

React는 호출 중 **하나의 결과를 사용하고 다른 호출의 결과는 무시**합니다. `컴포넌트`, `이니셜라이저`, `업데이터 함수`가 순수하다면 이 동작이 로직에 영향을 미치지 않습니다. React가 두 번 호출함으로써 실수를 찾는데 도움이 됩니다.

## `set` functions, like `setSomething(nextState)`

`useState`가 반환하는 `set`함수를 사용하면 상태를 다른 값으로 업데이트하고 렌더링을 다시 트리거할 수 있습니다. 다음 상태를 직접 전달하거나 이전 상태에서 계산하는 함수를 전달할 수 있습니다.

```tsx
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
}
```

### 1. Parameters

- `nextState` - 상태를 원하는 값입니다. 모든 유형의 값이 될 수 있지만 함수에 대한 특별한 동작이 있습니다.

#### `nextState`가 함수라면?

함수를 `nextState`로 전달하면 `업데이터 함수`로 취급됩니다. 이 함수는 순수 함수여야 하고, **보류 중인 상태를 유일한 인수로 사용**해야 하며, 다음 상태를 반환해야 합니다. React는 업데이터 함수를 대기열에 넣고 컴포넌트를 다시 렌더링합니다. 다음 렌더링 중에 React는 대기열에 있는 모든 업데이터를 이전 `state`에 적용하여 다음 `state`를 계산합니다.

**이전 상태를 기반으로 상태 업데이트**

```tsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

나이가 42세라고 가정한 후 `setAge(age + 1)`를 3번 호출한다고 가정을 해보자. 우리가 예상을 해보면 42세에서 3번을 1씩 더 했으니까 45세가 나온다고 생각하는데 43세가 나온다. 이는 set 함수를 호출해도 **이미 실행 중인 코드에서 나이 상태 변수가 업데이트 되지 않기 때문**입니다.

이러한 문제를 해결하기 위해서는 업데이터 함수를 넣어 해결할 수 있습니다..

```tsx
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

여기서 `a => a + 1`은 `업데이터 함수`입니다. 이 함수는 **보류 중인 상태를 가져와서 다음 상태를 계산**합니다.

React는 `업데이터 함수`를 대기열에 넣고 다음 렌더링 중 같은 순서로 호출합니다.

1. a => a + 1은 42를 보류 상태로 받고 다음 상태로 43을 반환
2. a => a + 1은 43을 보류 중인 상태로 수신하고 다음 상태로 44를 반환
3. a => a + 1은 보류 중인 상태로 44를 수신하고 다음 상태로 45를 반환

이후 대기열에 다른 업데이트가 없으므로, 45를 현재 상태로 저장합니다.

컨벤션에 따라 **보유 중인 상태 인수의 이름을 state 변수 이름의 첫 글자**로 정하는 것이 일반적입니다. 하지만 더 명확하다고 생각되는 다른 이름으로 사용해도 됩니다.

React는 개발 과정에서 순수 함수인지 확인하기 위해 `업데이터 함수`를 두 번 호출할 수 있습니다.

### 2. Returns

`set` 함수에는 반환 값이 없습니다.

### 3. 주의사항

#### 1. 💡 `set` 함수는 다음 렌더링에 대한 상태 변수만 업데이트합니다

`set`함수를 호출한 후 상태 변수를 읽으면 호출 전 화면에 있던 이전 값을 계속 가져옵니다.

```jsx
function handleClick() {
  console.log(count);  // 0

  setCount(count + 1); // Request a re-render with 1
  console.log(count);  // Still 0!

  setTimeout(() => {
    console.log(count); // Also 0!
  }, 5000);

}
```

상태는 `snapshot`처럼 동작하기 때문에, 상태를 업데이트하면 새 상태 값으로 다른 렌더링을 요청하지만 **이미 실행 중인 이벤트 핸들러의 `count`와 `JavaScript 변수`에는 영향을 미치지 않습니다.**

아래와 같이 미리 함수에 전달하기 전 상태를 변수에 저장 후 사용할 수 있습니다.

```jsx
const nextCount = count + 1;
setCount(nextCount);

console.log(count);     // 0
console.log(nextCount); // 1
```

#### 2. 💡 `Object.is` 비교로 사용자가 제공한 새 값이 현재 상태와 동일 시 컴포넌트와 그 자식들을 재렌더링 하지 않는다

`Object.is()` 비교에 의해 결정된 대로 사용자가 제공한 새 값이 현재 상태와 동일하다면, React는 컴포넌트와 그 자식들을 다시 렌더링하지 않습니다. 이것은 **`최적화`** 입니다. 경우에 따라 React가 자식을 건너뛰기 전에 컴포넌트를 호출해야 할 수도 있지만, 코드에 영향을 미치지는 않습니다.

보통 객체나 배열의 상태를 직접 변경할 때 발생합니다. 예를 들어 기존 상태 `obj`에 `key`는 `x`, `value`는 `10`을 추가한다고 가정해보자.

```tsx
obj.x = 10;  // Error: 기존 개체 변경
setObj(obj); // 아무일도 이러나지 않는다.
```

위와 같이 기존 객체를 변경 후 다시 `setObj`로 전달했기에 React가 업데이트를 무시했습니다. 이를 해결하기 위해 아래와 같이 기존 객체와 배열을 변경하지 않고 항상 상태의 객체와 배열을 교체해야합니다.

```tsx
setObj({
  ...obj,
  x: 10
}); // Success
```

#### 3. 💡 React는 배치 상태가 업데이트되고 모든 이벤트가 실행되고 설정된 함수를 호출 후 화면을 업데이트한다

이렇게 하면 단일 이벤트 중에 여러 번 다시 렌더링하는 것을 방지할 수 있습니다. 드물지만 DOM에 접근하기 위해 React가 화면을 더 일찍 업데이트하도록 강제해야 하는 경우, `flushSync`를 사용하여 해결할 수 있습니다.

#### 4. 💡 렌더링 도중 `set` 함수 호출 시 현재 렌더링 중인 컴포넌트 내에서만 허용한다

React는 해당 출력을 버리고 즉시 새로운 상태로 다시 렌더링을 시도합니다. 이 패턴은 거의 필요하지 않지만 **이전 렌더링의 정보를 저장**하는 데 사용할 수 있습니다.

#### 5. 💡 `StrictMode` 사용 시 `업데이터`를 두번 호출

개발모드에서 `StrictMode` 사용 시 실수로 발생한 불필요한 문제들을 찾기 위해 `업데이터` 함수를 두번 호출한다. 만약 업데이터 함수가 순수하다면 다른 동작에 영향을 미치지 않습니다.

## 이외 특이사항들

### 상태를 함수로 설정 시에 발생하는 오류

상태를 함수로 설정하려고 하는데 함수가 대신 호출되는 경우가 종종 발생한다.

```tsx
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

위 코드를 분석하면 함수를 전달하기 때문에 React는 `someFunction`이 초기화 함수고 `someOtherFunction`이 업데이터 함수라고 가정하고 이를 호출 후 결과를 저장하려고 시도합니다. 만약 전달한 함수를 저장하게 하려면 두 경우 모두 앞에 `() =>`를 넣어야 합니다.

```tsx
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```

### 재랜더링 횟수 초과 오류

React는 무한 루프를 방지하기 위해 **렌더링 횟수를 암묵적으로 제한**합니다. 이는 렌더링 중 무조건 상태를 설정하므로 컴포넌트는 렌더링, 상태 설정(렌더링 발생), 렌더링 등의 루프에 진입하게 됩니다. 보통 아래와 같이 이벤트 핸들러를 지정하는 과정에서 실수로 발생하는 경우가 많습니다.

```tsx
// Error: 렌더링이 되는 동안 handler를 호출한다.
return <button onClick={handleClick()}>Click me</button>

// Success: 이벤트 헨들러를 전달한다.
return <button onClick={handleClick}>Click me</button>

// Suceess: 인라인 함수를 전달한다.
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

> 💡 만약 오류의 원인을 찾을 수 없는 경우
>
> 콘솔에서 오류 옆에 있는 화살표를 클릭하고 자바스크립트 스택을 살펴보고 오류의 원인이 되는 특정 `set` 함수 호출
