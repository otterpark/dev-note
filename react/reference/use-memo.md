# useMemo()

`useMemo()` 리렌더링 사이에 계산 결과를 캐시할 수 있는 React hook입니다.

## Reference

```tsx
const cachedValue = useMemo(calculateValue, dependencies)
```

- 컴포넌트 최상위에서 `useMemo`를 사용하여, 다시 렌더링하는 사이에 계산을 캐시합니다.

### 1. Parameters

#### `calculateValue`

캐시하려는 값을 계산하는 함수입니다. **순수해야 하고, 인수를 받지 않아야 하며, 모든 유형의 값을 반환**해야합니다. React는 초기 렌더링 중에 함수를 호출합니다. 다음 렌더링에서 React는 마지막 렌더링 이후 종속성이 변경되지 않은 경우 동일한 값을 다시 반환합니다. 그렇지 않으면 계산값을 호출하고 결과를 반환 후 나중에 재사용할 수 있도록 저장합니다.

#### 2. `dependencies`

`calculateValue` 코드 내에서 참조된 모든 반응하는 값의 목록입니다. 반응하는 값에는 `Props`, `State`, `컴포넌트 본문 내부에서 직접 선언된 모든 변수 및 함수`가 포함됩니다. 만약 `lint`가 `React Config`로 설정되어 있다면, 모든 반응형 값이 종속성으로 올바르게 지정되었는지 확인합니다. 의존성 목록에는 일정한 수의 항목이 있어야하고 `[dep1, dep2, dep3]`과 같이 인라인으로 작성해야 합니다. React는 `Object.is` 비교 알고리즘을 사용하여 각 의존성을 이전 값과 비교합니다.

### 2. Returns

초기 렌더링에서 `useMemo`는 인자 없이 `calculateValue`를 호출한 결과를 반환합니다.

다음 렌더링 중에는 마지막 렌더링에서 이미 저장된 값을 반환하거나(종속성이 변경되지 않은 경우) `calculateValue`를 다시 호출하여 `calculateValue`가 반환한 결과를 반환합니다.

### 3. 주의사항

#### 1. 💡 `useMemo`는 hook이므로 컴포넌트 최상위 레벨이나 자체 hook에서만 호출할 수 있다

루프나 조건 내부에서는 호출할 수 없습니다. 필요한 경우 새 컴포넌트를 추출하고 상태를 그 안으로 이동하세요.

#### 💡 2. `StrictMode` 사용 시 `calculation function`을 두번 호출

개발 모드에서 `StrictMode` 사용 시 실수로 발생한 불필요한 문제들을 찾기 위해 `calculation function`를 두 번 호출합니다. `calculation function`가 순수 함수라면 이러한 동작에 영향을 미치지 않습니다. 호출 중 하나의 결과는 무시됩니다.

#### 💡 3. React는 특별한 이유가 없는 한 캐시된 값을 버리지 않습니다

예를 들어, 개발 단계에서 컴포넌트의 파일을 편집할 때 React는 캐시를 버립니다. 개발과 프로덕션 모두에서 컴포넌트가 초기 마운트 중에 일시 중단되면 React는 캐시를 버립니다. 성능 최적화를 위해 `useMemo`에만 의존한다면 괜찮을 것입니다. 그렇지 않은 경우 상태 변수나 참조가 더 적합할 수 있습니다.

> React의 향후 방향
>
> - React는 캐시를 버리기를 활용하는 더 많은 기능을 추가할 수도 있습니다.
> - React에 가상화된 목록에 대한 기본 지원이 추가되면 가상화된 테이블 뷰포트에서 스크롤되는 항목에 대한 캐시도 버리는 것이 합리적일 것입니다.

캐싱된 반환값을 메모화라고도 하는데, 이 hook을 `useMemo`라고 부르는 이유입니다.

## 사용법

### 계산이 많은 것들 스킵하기

리렌더링 사이에 계산을 캐시하려면 컴포넌트의 최상위 수준에서 `useMemo` 호출로 계산을 래핑해야합니다.

```tsx
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

메모를 사용하려면 두 가지를 전달해야 합니다.

1. `() =>`와 같이 인수를 받지 않고 계산하고자 하는 값을 반환하는 계산 함수
2. 컴포넌트 내에서 계산에 사용되는 모든 값을 포함한 종속성 목록

초기 렌더링에서 사용메모에서 얻을 수 있는 값은 계산을 호출한 결과입니다.

이후 모든 렌더링에서 React는 **의존성을 마지막 렌더링에서 전달한 의존성과 비교**합니다. 종속성 중 어떤 것도 변경되지 않았다면(`Object.is`와 비교했을 때), `useMemo`는 이전에 이미 계산한 값을 반환합니다. 그렇지 않으면 React는 계산을 다시 실행하고 새 값을 반환합니다.

다시 말해, `useMemo`는 **의존성이 변경될 때까지 재렌더링 사이에 계산 결과를 캐시**합니다.

이 기능이 언제 유용한지 예시를 통해 살펴보겠습니다.

기본적으로 React는 컴포넌트를 다시 렌더링할 때마다 컴포넌트의 전체 바디를 다시 실행합니다. 예를 들어, 이 `TodoList`가 상태를 업데이트하거나 부모로부터 새로운 `Props`을 받으면 `filterTodos` 함수가 다시 실행됩니다.

```tsx
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

일반적으로 대부분의 계산은 매우 빠르기 때문에 문제가 되지 않습니다. 그러나 **큰 배열을 필터링하거나 변환하거나 `복잡한 계산`을 수행하는 경우 데이터가 변경되지 않았다면 계산을 다시 수행하지 않는 것이 좋습니다**. `todos`과 `tab`이 같고 마지막 렌더링 시, 앞서와 같이 계산을 `useMemo`로 감싸면 이전에 이미 계산한 `visibleTodos`를 재사용할 수 있습니다.

> `useMemo`는 성능 최적화를 위해서만 사용해야 합니다. `useMemo` 없이 코드가 작동하지 않는 경우 근본적인 문제를 찾아서 먼저 해결하세요. 그런 다음 `useMemo`를 추가하여 성능을 개선할 수 있습니다.

이러한 유형의 캐싱을 **`메모화`** 라고 합니다.

#### 여기서의 `복잡한 계산`이란?

일반적으로 수천 개의 개체를 만들거나 반복하는 경우가 아니라면 계산하는데 시간이 많이 들지 않을 것입니다. 좀 더 확신을 얻고 싶다면 콘솔 로그를 추가하여 코드에 소요된 시간을 측정할 수 있습니다

```tsx
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

측정하려는 상호작용하는 곳에 `console.time`과 `console.timeEnd`를 수행합니다. `filter array: 0.15ms`가 콘솔에 표시됩니다. 전체적으로 기록된 시간이 1ms 이상으로 합산되면 해당 계산을 `useMemo`해 두는 것이 좋습니다. 그런 다음 실험으로 해당 계산을 `useMemo`로 감싸서 해당 상호작용에 대해 총 로깅 시간이 감소했는지 여부를 확인할 수 있습니다.

```tsx
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // todos와 tab 변경되지 않은 경우 건너뛰기
}, [todos, tab]);
console.timeEnd('filter array');
```

`useMemo`는 첫 번째 렌더링을 더 빠르게 만들지 않습니다. **업데이트 시 불필요한 작업을 건너뛰는 데만 도움**이 됩니다.

컴퓨터가 사용자 컴퓨터보다 빠를 수 있으므로 인위적인 속도 저하로 성능을 테스트하는 것이 좋습니다. 예를 들어 Chrome은 이를 위해 CPU 스로틀링 옵션을 제공합니다.

또한 개발 환경에서 성능을 측정하는 것은 가장 정확한 결과를 얻지 못한다는 점에 유의하세요. (예를 들어 `StrictMode`가 켜져 있으면 각 컴포넌트가 한 번이 아닌 두 번 렌더링되는 것을 볼 수 있습니다.) 가장 정확한 결과를 얻으려면 프로덕션용 앱을 빌드하고 사용자가 사용하는 것과 같은 기기에서 테스트하세요.

### 컴포넌트 재렌더링 건너뛰기

경우에 따라 `useMemo`는 하위 컴포넌트를 다시 렌더링하는 성능을 최적화하는 데 도움이 될 수도 있습니다. 이를 설명하기 위해 `TodoList` 컴포넌트가 하위 `List` 컴포넌트에 `Props`로 `visibleTodos`를 전달한다고 가정해 보겠습니다.

```tsx
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

`theme` `Props`를 전환하면 앱이 잠시 멈추는 것을 볼 수 있다. JSX에서 `<List />`를 제거하면 빠르게 느껴집니다. 이는 `List` 컴포넌트를 최적화할 가치가 있다는 것을 알려줍니다.

기본적으로 컴포넌트가 다시 렌더링되면 React는 **모든 자식을 재귀적으로 다시 렌더링**합니다. 그렇기 때문에 `TodoList`가 다른 테마로 리렌더링될 때 `List` 컴포넌트도 리렌더링됩니다. 이는 리렌더링을 하는데 많은 계산이 필요하지 않은 컴포넌트에는 괜찮습니다. 그러나 리렌더링이 느리다는 것을 확인했다면 `List`를 `memo`로 감싸서 `Props`가 지난 렌더링과 동일할 때 리렌더링을 건너뛰도록 지시할 수 있습니다.

```tsx
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

이 변경 사항을 적용하면 모든 `Props`가 마지막 렌더링과 동일한 경우 `List`가 리렌더링을 건너뜁니다. 여기서 **계산을 캐싱하는 것이 중요하다!** `useMemo` 없이 `visibleTodos`를 계산했다고 상상해 보세요.

```tsx
export default function TodoList({ todos, tab, theme }) {
  // theme가 변경될 때마다 다른 배열이 표시된다
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* 
        따라서 List의 props는 동일하지 않으며 매번 리렌더링됩니다.
      */}
      <List items={visibleTodos} />
    </div>
  );
}
```

위의 예에서 `filterTodos` 함수는 `{} 객체 리터럴`이 항상 새 객체를 생성하는 방식과 유사하게 **항상 다른 배열을 생성**합니다. 일반적으로는 문제가 되지 않지만, `list` `Props`은 결코 동일하지 않으며 `memo` 최적화가 작동하지 않는다는 것을 의미합니다. 바로 이때 `useMemo`가 유용합니다.

```tsx
export default function TodoList({ todos, tab, theme }) {
  // 재렌더링 사이에 계산을 캐시하도록 React에 지시하기
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // 따라서 이러한 종속성이 변경되지 않는 한
  );
  return (
    <div className={theme}>
      {/*
        list에 동일한 Props이 제공되며 리렌더링을 생략할 수 있습니다.
      */}
      <List items={visibleTodos} />
    </div>
  );
}
```

`visibleTodos` 계산을 `useMemo`로 감싸면 종속성이 변경될 때까지 리렌더링 간에 동일한 값을 갖도록 할 수 있습니다. 특별한 이유가 없는 한 계산을 `useMemo`로 래핑할 필요는 없습니다. 이 예제에서는 `memo`로 래핑된 컴포넌트에 전달하면 리렌더링을 건너뛸 수 있기 때문입니다. 이 페이지에서 자세히 설명하는 몇 가지 다른 이유도 있습니다.

### 다른 hook의 종속성 메모화

컴포넌트 본문에서 직접 생성된 객체에 의존하는 계산이 있다고 가정해 보자.

```tsx
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 주의: 컴포넌트 본문에서 생성된 객체에 대한 종속성
  // ...
```

이렇게 객체에 의존하는 것은 **메모화의 취지를 무색**하게 합니다. 컴포넌트가 리렌더링되면 컴포넌트 본문 내부의 모든 코드가 다시 실행됩니다. `searchOptions` 객체를 생성하는 코드 줄도 리렌더링할 때마다 실행됩니다. `searchOptions`는 `useMemo` 호출의 종속성이고 매번 다르기 때문에 React는 종속성이 다르다는 것을 알고 매번 `searchItems`를 다시 계산합니다.

이 문제를 해결하려면 검색옵션 객체를 종속성으로 전달하기 전에 `searchOptions` 객체 자체를 메모화할 수 있습니다

```tsx
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ 텍스트가 바뀔때만 바뀐다

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ allItems과 searchOptions 가 바뀔때만 바뀐다.
  // ...
```

위의 예에서 텍스트가 변경되지 않았다면 `searchOptions` 객체도 변경되지 않습니다. 그러나 이보다 더 나은 수정 방법은 `searchOptions` 객체 선언을 `useMemo` 계산 함수 내부로 이동하는 것입니다.

```tsx
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ allItems과 text 가 바뀔때만 바뀐다.
  // ...
```

이제 계산은 텍스트에 직접적으로 의존합니다(`문자열이므로 '실수로' 달라질 수 없음`)

### 함수 메모화

`Form` 컴포넌트가 `memo`로 감싸져 있다고 가정해봅시다. 여기에 함수를 `Props`로 전달하려고 합니다.

```tsx
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

`function() {}`가 다른 객체를 생성하는 것처럼 `function() {}`와 같은 `함수 선언`과 `() => {}`와 같은 표현식은 리렌더링할 때마다 다른 함수를 생성합니다. 새 함수를 만드는 것 자체는 문제가 되지 않고 지양해야 할 일이 아닙니다. 하지만 폼 컴포넌트가 `메모화`되어 있다면 `Props`가 변경되지 않았을 때 리렌더링하는 것을 건너뛰고 싶을 것입니다. `Props`가 항상 달라지면 `메모화`의 취지가 무색해집니다.

`useMemo`로 함수를 `메모화`하려면 `calculation 함수`가 다른 함수를 반환해야 합니다.

```tsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

무언가 투박해 보입니다. 함수를 `메모화`하는 것은 흔한 일이며, React에서는 이를 위해 특별히 hook이 내장되어 있습니다. 중첩 함수를 추가로 작성할 필요가 없도록 함수를 `useMemo` 대신 `useCallback`으로 래핑해보자.

```tsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

위의 두 예제는 완전히 동일합니다. `useCallback`을 사용하면 내부에 중첩된 함수를 추가로 작성하지 않아도 된다는 장점이 있습니다. 그 외에는 아무것도 하지 않습니다.

## 문제 해결

### 렌더링할 때마다 계산이 두 번 실행됩니다

`StrictMode`에서 React는 일부 함수를 한 번이 아닌 두 번 호출합니다.

```tsx
function TodoList({ todos, tab }) {
  // 이 컴포넌트 함수는 렌더링할 때마다 두 번 실행됩니다.

  const visibleTodos = useMemo(() => {
    // 종속성 중 하나라도 변경되면 이 계산은 두 번 실행됩니다.
    return filterTodos(todos, tab);
  }, [todos, tab]);

  // ...
```

이는 예상되는 현상이며 코드를 손상시키지 않아야 합니다.

이 개발 모드 동작은 컴포넌트를 **순수하게 유지하는 데 도움**이 됩니다. React는 호출 중 하나의 결과를 사용하고 다른 호출의 결과는 무시합니다. 컴포넌트와 계산 함수가 순수하다면 로직에 영향을 주지 않아야 합니다. 하지만 **실수로 함수를 순수하지 않게 작성한 경우 실수를 알아채고 수정하는 데 도움**이 됩니다.

예를 들어, 이 순수하지 않은 함수는 `Props`로 받은 배열을 변경합니다.

```tsx
  const visibleTodos = useMemo(() => {
    // 🚩 실수: `Props` 변형
    todos.push({ id: 'last', text: 'Go for a walk!' });
    const filtered = filterTodos(todos, tab);
    return filtered;
  }, [todos, tab]);
```

React는 함수를 두 번 호출하므로 할 일이 두 번 추가되는 것을 알 수 있습니다. 계산이 기존 객체를 변경해서는 안 되지만 계산 중에 생성한 새 객체를 변경해도 괜찮습니다. 예를 들어 `filterTodos` 함수가 항상 다른 배열을 반환하는 경우 해당 배열을 변경할 수 있습니다.

```tsx
const visibleTodos = useMemo(() => {
    const filtered = filterTodos(todos, tab);
    // ✅ 계산 중에 생성한 객체를 변경합니다.
    filtered.push({ id: 'last', text: 'Go for a walk!' });
    return filtered;
  }, [todos, tab]);
```

### `useMemo` 호출이 객체를 반환해야 하지만 정의되지 않은 객체를 반환합니다

```tsx
 // 🔴 '() => {}'를 사용하는 화살표 함수에서는 객체를 반환할 수 없습니다.
  const searchOptions = useMemo(() => {
    matchMode: 'whole-word',
    text: text
  }, [text]);
```

JavaScript에서 `() => {`는 화살표 함수 본문을 시작하므로 `{` 중괄호는 객체의 일부가 아닙니다. 이 때문에 **객체를 반환하지 않고 실수가 발생**합니다. `({`, `})`와 같은 괄호를 추가하면 이 문제를 해결할 수 있습니다.

```tsx
// 아래 코드는 작동하지만 누군가가 다시 깨뜨리기 쉽습니다.
const searchOptions = useMemo(() => ({
  matchMode: 'whole-word',
  text: text
}), [text]);
```

하지만 이 방식은 여전히 헷갈리고 괄호를 제거하면 누군가 쉽게 실수할 수 있습니다.

이러한 실수를 방지하려면 `return` 문을 명시적으로 작성하세요.

```tsx
  // ✅ 이것은 작동하고 명시적입니다.
  const searchOptions = useMemo(() => {
    return {
      matchMode: 'whole-word',
      text: text
    };
  }, [text]);
```

### 컴포넌트가 렌더링될 때마다 `useMemo`의 계산이 다시 실행됩니다

**두 번째 인수로 종속성 배열을 지정했는지 확인**하세요!

종속성 배열을 잊어버렸을 경우, `useMemo`는 매번 계산을 다시 실행합니다.

```tsx
function TodoList({ todos, tab }) {
  // 🔴 종속성 배열이 없으므로, 매번 재계산한다.
  const visibleTodos = useMemo(() => filterTodos(todos, tab));
  // ...
```

아래는 종속성 배열을 두 번째 인수로 전달하는 수정된 버전입니다.

```tsx
function TodoList({ todos, tab }) {
  // ✅ 불필요하게 재계산하지 않습니다.
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
```

만약 도움이 되지 않는다면 종속성 중 하나 이상이 이전 렌더링과 다르다는 문제일 수 있습니다. 종속성을 콘솔에 수동으로 로깅하여 이 문제를 디버그할 수 있습니다.

```tsx
const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
console.log([todos, tab]);
```

그런 다음 콘솔에서 서로 다른 리렌더된 배열을 마우스 오른쪽 버튼으로 클릭하고 두 배열 모두에 대해 `"전역 변수로 저장"`을 선택할 수 있습니다. 첫 번째 배열이 `temp1`로 저장되고 두 번째 배열이 `temp2`로 저장되었다고 가정하면 브라우저 콘솔을 사용하여 두 배열의 각 종속성이 동일한지 확인할 수 있습니다

```tsx
Object.is(temp1[0], temp2[0]); // 배열 간에 첫 번째 종속성이 동일한지?
Object.is(temp1[1], temp2[1]); // 배열 간에 두 번째 종속성이 동일한지?
Object.is(temp1[2], temp2[2]); // 배열 간에 모든 종속성이 동일한지?
```

어떤 종속성이 메모화를 방해하는지 찾았다면 그 종속성을 제거할 방법을 찾거나 함께 메모화해 놓으세요.

### 루프에서 각 목록 항목에 대해 `useMemo`를 호출해야 하지만 허용되지 않습니다

`chart` 컴포넌트가 `memo`로 감싸져 있다고 가정해 봅시다. `ReportList` 컴포넌트가 리렌더링할 때 목록의 모든 `Chart`를 리렌더링하는 것을 건너뛰고 싶을 수 있습니다. 하지만 `useMemo`를 루프에서 호출할 수는 없습니다.

```tsx
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 다음과 같이 useMemo를 루프에서 호출할 수 없습니다.
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}
```

대신 각 항목에 대한 구성 요소를 추출하고 개별 항목에 대한 데이터를 메모화하세요.

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
  // ✅ 최상위 수준에서 사용 메모를 호출합니다.
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```

또는 `useMemo`를 제거하고 대신 `Report` 자체를 `memo`로 래핑할 수 있습니다. 항목 `Props`가 변경되지 않으면 `Report`가 리렌더링을 건너뛰므로 `chart`도 리렌더링을 건너뜁니다.

```tsx
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  const data = calculateReport(item);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
});
```
