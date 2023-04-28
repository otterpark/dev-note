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

기본적으로 React는 컴포넌트를 다시 렌더링할 때마다 컴포넌트의 전체 바디를 다시 실행합니다. 예를 들어, 이 `TodoList`가 상태를 업데이트하거나 부모로부터 새로운 `Props`을 받으면 `filterTodos` 함수가 다시 실행됩니다:

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
