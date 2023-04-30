# useCallback()

`useCallback()`은 리렌더링 사이에 함수 정의를 캐시할 수 있는 React hook입니다.

## Reference

```tsx
const cachedFn = useCallback(fn, dependencies)
```

- 컴포넌트 최상위에서 `useCallback`를 사용하여, 상태 변수를 선언합니다.

### 1. Parameters

#### `fn`

캐시하려는 함수의 값입니다. 인자도 받을 수 있으며, 값 반환할 수 있습니다. React는 초기 렌더링 중에 함수를 반환(호출 X)합니다. 다음 렌더링에서 React는 마지막 렌더링 이후 의존성이 변경되지 않은 경우 동일한 함수를 다시 제공합니다. 그렇지 않으면 현재 렌더링 중에 전달한 함수를 제공하고 나중에 재사용할 수 있도록 저장합니다. React는 함수를 호출하지 않습니다. 함수는 사용자에게 반환되므로 언제 호출할지 결정할 수 있습니다.

#### `dependencies`

`fn` 코드 내에서 참조된 모든 반응하는 값의 목록입니다. 반응하는 값에는 `Props`, `State`, `컴포넌트 본문 내부에서 직접 선언된 모든 변수 및 함수`가 포함됩니다. 만약 `lint`가 `React Config`로 설정되어 있다면, 모든 반응형 값이 의존성으로 올바르게 지정되었는지 확인합니다. 의존성 목록에는 일정한 수의 항목이 있어야하고 `[dep1, dep2, dep3]`과 같이 인라인으로 작성해야 합니다. React는 `Object.is` 비교 알고리즘을 사용하여 각 의존성을 이전 값과 비교합니다.

### 2. Returns

초기 렌더링에서 `useCallback`은 전달한 `fn` 함수를 반환합니다.

이후 렌더링 중에는 마지막 렌더링에서 이미 저장된 `fn` 함수를 반환하거나(의존성이 변경되지 않은 경우), 이 렌더링 중에 전달한 `fn` 함수를 반환합니다.

### 3. 주의사항

#### 1. 💡 `useCallback()`은 hook이므로, 컴포넌트의 최상위 레벨이나 자체 hook에서만 호출할 수 있다

루프나 조건 내부에서는 호출할 수 없습니다. 필요한 경우 새 컴포넌트를 추출하고 상태를 그 안으로 옮겨야합니다.

#### 2. 💡 React는 특별한 이유가 없을 시 캐시된 함수를 버리지 않습니다

예를 들어, 개발 단계에서 컴포넌트의 파일을 편집할 때 React는 캐시를 버립니다. 또한 개발 및 프로덕션 단계에서 컴포넌트가 초기 마운트 중에 일시 중단되면 React는 캐시를 버립니다. 성능 최적화를 위해 `useCallback()`에 의존하는 경우 기대에 부합할 것입니다. 그렇지 않은 경우 상태 변수나 참조가 더 적합할 수 있습니다.

> React의 향후 방향
>
> - React는 캐시를 버리기를 활용하는 더 많은 기능을 추가할 수도 있습니다.
> - React에 가상화된 목록에 대한 기본 지원이 추가되면 가상화된 테이블 뷰포트에서 스크롤되는 항목에 대한 캐시도 버리는 것이 합리적일 것입니다.

## 사용법

### 컴포넌트 재랜더링 스킵하기

렌더링 성능을 최적화할 때 자식 컴포넌트에 전달하는 함수를 캐시해야 할 때가 있습니다. 먼저 이를 수행하는 방법에 대한 구문을 살펴본 다음 어떤 경우에 유용한지 알아보겠습니다.

컴포넌트를 다시 렌더링하는 사이에 함수를 캐시하려면 해당 함수의 정의를 `useCallback` hook에 래핑해보자

```tsx
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
}
```

콜백을 사용하려면 두 가지를 전달해야 합니다

- 리렌더링 사이에 캐시하려는 함수 정의
- 컴포넌트 내에서 함수 내부에 사용되는 모든 값을 포함한 `의존성 목록(list of dependencies)`

초기 렌더링에서 `useCallback`에서 반환되는 함수는 전달한 함수가 될 것입니다.

다음 렌더링에서 React는 **의존성을 이전 렌더링 중에 전달한 의존성과 비교**합니다. 의존성 중 변경된 것이 없다면(`Object.is`와 비교했을 때), `useCallback`은 이전과 동일한 함수를 반환합니다. 그렇지 않으면 `useCallback`은 이 렌더링에서 전달한 함수를 반환합니다.

다시 말해, `useCallback`은 **의존성이 변경될 때까지 리렌더링 사이에 함수를 캐시**합니다.

그럼 이 기능이 어디에 유용한지 예제로 알아보자

`ProductPage`에서 `ShippingForm` 컴포넌트로 `handleSubmit` 함수를 전달한다고 가정해 보겠습니다

```tsx
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

`theme` `Props`를 전환하면 앱이 잠시 멈추는 것을 볼 수 있습니다. 만약 JSX에서 `<ShippingForm />`을 제거하면 조금 더 빠르게 느껴질 수 있습니다. 이는 `ShippingForm` 컴포넌트를 최적화할 가치가 있다는 것을 알려줍니다.

기본적으로 컴포넌트가 리렌더링되면 React는 모든 자식을 재귀적으로 리렌더링(`리렌더링 조건`)합니다. 그렇기 때문에 `ProductPage`가 다른 테마로 리렌더링 시 `ShippingForm` 컴포넌트도 리렌더링됩니다. 이는 리렌더링하는 데 많은 계산이 필요하지 않은 컴포넌트에는 괜찮지만 어느정도 복잡한 계산이 필요한 컴포넌트의 경우 리렌더링 속도가 느리다는 것을 확인할 수 있습니다. `ShippingForm`을 `memo`로 레핑하여 `Props`가 지난 렌더링과 동일한 경우 리렌더링을 건너뛰도록 할 수 있습니다.

```tsx
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

이 변경 사항으로 `ShippingForm`은 모든 `Props`가 마지막 렌더링과 동일한 경우 리렌더링을 건너뜁니다. 이때 `함수 캐싱`이 중요합니다. `useCallback` 없이 `handleSubmit`을 정의했다고 가정해 봅시다.

```tsx
function ProductPage({ productId, referrer, theme }) {
  // 매번 theme이 바뀔때마다, 이 함수는 다른 함수가 될 것이다.
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      {/* 
        따라서 'ShippingForm'의 Props는 동일하지 않으며 매번 리렌더링합니다.
      */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

JavaScript에서 `function() {}` 또는 `() => {}`는 `{} 객체 리터럴`이 항상 새 객체를 생성하는 것과 유사하게 항상 다른 함수를 생성합니다. 일반적으로는 문제가 되지 않지만 `ShippingForm` `Props`는 동일하지 않으며 `memo` 최적화가 작동하지 않는다는 의미입니다. 바로 이 지점에서 `useCallback`이 유용합니다

```tsx
function ProductPage({ productId, referrer, theme }) {
  // 리렌더링 사이에 함수를 캐시하도록 React에 지시
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // 의존성이 변하지 않는 한

  return (
    <div className={theme}>
      {/* 
        'ShippingForm'은 동일한 Props를 수신하며 리렌더링을 건너뛸 수 있습니다.
      */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

`handleSubmit`을 `useCallback`으로 감싸면 **의존성이 변경될 때까지 리렌더링 간에 동일한 함수가 되도록** 할 수 있습니다. 특별한 이유가 없는 한 함수를 `useCallback`으로 래핑할 필요는 없습니다. 이 예제에서는 `memo`로 래핑된 컴포넌트에 함수를 전달하면 리렌더링을 건너뛸 수 있기 때문입니다. 이 페이지에서 더 자세히 설명하는 다른 이유도 있을 수 있습니다.

> `useCallback`은 **성능 최적화를 위해서만 사용**해야 합니다.
>
> 코드가 `useCallback` 없이 작동하지 않는다면 근본적인 문제를 찾아서 먼저 해결하세요. 그런 다음 `useCallback`을 다시 추가할 수 있습니다.

## 특이사항

### `Memoization(메모이제이션)`된 콜백에서 상태 업데이트하기

때로는 `Memoization(메모이제이션)`된 콜백의 이전 상태를 기반으로 상태를 업데이트해야 할 수도 있습니다. `handleAddTodo` 함수는 할일을 의존성으로 지정하여 다음 할일을 계산하기 때문입니다

```tsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

일반적으로 `Memoization(메모이제이션)`된 함수는 가능한 한 적은 의존성을 갖기를 원할 것입니다. 다음 상태를 계산하기 위해 일부 상태만 읽어야 하는 경우, 대신 업데이터 함수를 전달하여 해당 의존성을 제거할 수 있습니다.

```tsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // todos 의존성이 필요 없음
  // ...
```

여기서는 할 일들을 의존성으로 만들고 내부에서 읽는 대신, 상태를 업데이트하는 방법에 대한 명령어(`todos => [...todos, newTodo]`)를 React에 전달합니다.

> useState 업데이터 함수에 대해 자세히 읽어보세요!

## 이펙트가 너무 자주 발생하지 않도록 하기

가끔 `Effect` 내부에서 함수를 호출하고 싶을 때가 있습니다.

```tsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    // ...
```

모든 반응형 값은 `Effect`의 의존성으로 선언해야 합니다. 그러나 `createOptions`를 의존성으로 선언하면 `Effect`가 채팅방에 계속 다시 연결하게 됩니다.

```tsx
 useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 이 의존성은 리렌더링 시 매번 바뀐다.
  // ...
```

이 문제를 해결하려면 `Effect`에서 호출해야 하는 함수를 `useCallback`으로 래핑하면 됩니다

```tsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // roomId가 바뀔 때마다 change

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // createOptions 바뀔 때마다 change
  // ...
```

`roomId`가 동일한 경우 리렌더링할 때마다 `createOptions` 함수가 동일하게 유지됩니다. 하지만 함수 의존성을 없애는 것이 더 좋습니다. 함수를 `Effect` 내부로 이동합니다.

```tsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // useCallback이나 함수 의존성이 필요 없다.
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // roomId가 바뀔 때마다 change
  // ...
```

### 커스텀 훅 최적화하기

커스텀 Hook을 작성하는 경우 반환하는 모든 함수를 `useCallback`으로 감싸는 것이 좋습니다

```tsx
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

## 문제 해결

컴포넌트가 렌더링될 때마다 `useCallback`은 다른 함수를 반환합니다.

두 번째 인수로 의존성 배열을 지정했는지 확인해야합니다. 의존성 배열을 잊어버린 경우, `useCallback`은 매번 새로운 함수를 반환합니다

```tsx
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 매번 새로운 함수 반환: 의존성 배열 없음
  // ...
```

다음은 의존성 배열을 두 번째 인수로 전달하는 수정된 버전입니다.

```tsx
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // 불필요하게 새 함수를 반환하지 않는다.
  // ...
```

그래도 도움이 되지 않는다면 의존성 중 하나 이상이 이전 렌더링과 다르다는 문제일 수 있습니다. 의존성을 콘솔에 수동으로 로깅하여 이 문제를 디버그할 수 있습니다

```tsx
const handleSubmit = useCallback((orderDetails) => {
  // ..
}, [productId, referrer]);

console.log([productId, referrer]);
```

그런 다음 콘솔에서 서로 다른 리렌더된 배열을 마우스 오른쪽 버튼으로 클릭하고 두 배열 모두에 대해 `"전역 변수로 저장"`을 선택할 수 있습니다. 첫 번째 배열이 `temp1`로 저장되고 두 번째 배열이 `temp2`로 저장되었다고 가정하면 브라우저 콘솔을 사용하여 두 배열의 각 의존성이 동일한지 확인할 수 있습니다

```tsx
Object.is(temp1[0], temp2[0]); // 배열 간에 첫 번째 의존성이 동일한지?
Object.is(temp1[1], temp2[1]); // 배열 간에 두 번째 의존성이 동일한지?
Object.is(temp1[2], temp2[2]); // 배열 간에 모든 의존성이 동일한지?
```

어떤 의존성이 `Memoization(메모이제이션)`를 방해하는지 찾았다면 그 의존성을 제거할 방법을 찾거나 함께 `Memoization(메모이제이션)`해 놓으세요.

### 루프에서 각 목록 항목에 대해 `useCallback`을 호출해야하지만 허용하지 않습니다

`chart` 컴포넌트가 메모로 감싸져 있다고 가정해 봅시다. `ReportList` 컴포넌트가 리렌더링할 때 목록의 모든 차트를 리렌더링하는 것을 건너뛰고 싶을 수 있습니다. 하지만 루프에서 `useCallback`을 호출할 수는 없습니다

```tsx
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 이와 같은 루프에서는 useCallback을 호출할 수 없습니다
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

대신 개별 항목에 대한 컴포넌트를 추출하고 거기에 `useCallback`을 넣습니다

```tsx
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // 최상위 수준에서 사용 콜백 호출
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```

또는 마지막 코드에서 `useCallback`을 제거하고 대신 `Report` 자체를 `memo`로 감싸는 방법도 있습니다. 항목 `Props`가 변경되지 않으면 `Report`가 리렌더링을 건너뛰므로 `Chart`도 리렌더링을 건너뜁니다

```tsx
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```
