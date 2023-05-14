# React Testing Library API

## render

```tsx
function render(
  ui: React.ReactElement<any>,
  options?: {
    /* You won't often use this, expand below for docs on options */
  },
): RenderResult
```

`document.body`에 추가되는 컨테이너에 렌더링합니다.

```tsx
import {render} from '@testing-library/react'

render(<div />)
```

```tsx
import {render} from '@testing-library/react'
import '@testing-library/jest-dom'

test('renders a message', () => {
  const {asFragment, getByText} = render(<Greeting />)
  expect(getByText('Hello, world!')).toBeInTheDocument()
  expect(asFragment()).toMatchInlineSnapshot(`
    <h1>Hello, World!</h1>
  `)
})
```

## render Options

옵션을 지정해야 하는 경우 사용 가능한 옵션은 다음과 같습니다.

### `container`

기본적으로 `React Testing Library`는 `div`를 생성하고 해당 `div`를 `document.body`에 추가하며, 이 곳에서 `React 컴포넌트가 렌더링`됩니다. 이 옵션을 통해 고유한 `HTMLElement 컨테이너`를 제공하면 `document.body`에 자동으로 추가되지 않습니다.

예를 들어 `tablebody` 엘리먼트를 단위 테스트 하는 경우, 해당 엘리먼트는 `div`의 자식이 될 수 없습니다. 이 경우 `table`을 `render container`로 지정할 수 있습니다.

```ts
const table = document.createElement('table');

const {container} = render(<TableBody {...props} />, {
  container: document.body.appendChild(table),
});
```

### `baseElement`

`container`가 지정되어 있으면 기본값은 해당 컨테이너이고, 그렇지 않으면 기본값은 `document.body`입니다. 이것은 쿼리의 기본 요소로 사용되며 `debug()`를 사용할 때 출력되는 요소이기도 합니다.

### `hydrate`

`hydrate`가 `true`로 설정되어 있으면 `ReactDOM.hydrate`로 렌더링됩니다. 서버 측 렌더링을 사용하고 `ReactDOM.hydrate`를 사용하여 컴포넌트를 마운트하는 경우 유용할 수 있습니다.

### `legacyRoot`

기본적으로 동시 기능(예: `ReactDOMClient.createRoot`)을 지원하여 렌더링합니다. 그러나 `React 17`에서와 같이 렌더링이 필요한 `레거시 앱`(예: `ReactDOM.render`)을 다루는 경우 `legacyRoot: true`를 설정하여 이 옵션을 `활성화`해야 합니다.

### `wrapper`

`wrapper` 옵션으로 React 컴포넌트를 전달하여 내부 엘리먼트 주위에 렌더링되도록 합니다. 이 옵션은 일반적인 데이터 공급자를 위해 재사용 가능한 `custom render` 함수를 만들 때 가장 유용합니다.

### `queries`

바인딩할 쿼리이며, 병합하지 않는 한 `DOM Testing Library`의 기본 집합을 재정의합니다.

```ts
// 예: 테이블 내용을 가로지르는 함수
import * as tableQueries from 'my-table-query-library'
import {queries} from '@testing-library/react'

const {getByRowColumn, getByText} = render(<MyTable />, {
  queries: {...queries, ...tableQueries},
})
```

## render result

`render` 메서드는 몇 가지 프로퍼티를 가진 객체를 반환합니다.

### `...queries`

`render`의 가장 중요한 기능은 `DOM Testing Library`의 쿼리가 기본값이 `document.body`인 `baseElement`에 바인딩된 첫 번째 인수와 함께 자동으로 반환된다는 점입니다.

```tsx
const {getByLabelText, queryAllByTestId} = render(<Component />)
```

### `container`

렌더링된 React 엘리먼트의 포함 DOM 노드(`ReactDOM.render`를 사용하여 렌더링)`div`입니다. 이것은 일반 DOM 노드이므로 `container.querySelector` 등을 호출하여 자식들을 검사할 수 있습니다.

> 💡 렌더링된 엘리먼트의 루트 엘리먼트를 가져오려면 `container.firstChild`를 사용합니다.
>
> - 루트 엘리먼트가 `React Fragment`인 경우 `container.firstChild`는 해당 `Fragment`의 첫 번째 자식만 가져오고 `Fragment` 자체는 가져오지 않습니다.

🚨 하지만 컨테이너를 사용하여 렌더링된 엘리먼트를 쿼리하는 경우 다시 생각해봐야 합니다! 컨테이너를 사용하여 요소를 쿼리하지 마세요!

### `baseElement`

컨테이너에서 React 엘리먼트가 `render`되고 포함되는 `DOM 노드`입니다. 렌더링 옵션에서 `baseElement`를 지정하지 않으면 기본값은 `document.body`가 됩니다.

테스트하려는 컴포넌트가 컨테이너 `div` 외부에서 무언가를 렌더링할 때 유용합니다(예: 본문에서 직접 HTML을 렌더링하는 포털 컴포넌트를 스냅샷으로 테스트하려는 경우).

참고: `render`이 반환하는 쿼리는 `baseElement`를 조사하므로, 쿼리를 사용하여  `baseElement` 없이 포털 컴포넌트를 테스트할 수 있습니다.

### `debug`

이 메서드는 `console.log(prettyDOM(baseElement))`의 바로 가기입니다.

```tsx
import React from 'react'
import {render} from '@testing-library/react'

const HelloWorld = () => <h1>Hello World</h1>
const {debug} = render(<HelloWorld />)
debug()

// <div>
//   <h1>Hello World</h1>
// </div>
// you can also pass an element: debug(getByTestId('messages'))
// and you can pass all the same arguments to debug as you can
// to prettyDOM:
// const maxLengthToPrint = 10000
// debug(getByTestId('messages'), maxLengthToPrint, {highlight: false})
```

> `DOM Testing Library`에서 제공되는 `prettyDOM`을 둘러싼 간단한 랩퍼입니다.

### `rerender`

`Props` 업데이트를 수행하는 컴포넌트를 테스트하여 `Props`이 올바르게 업데이트되고 있는지 확인하는 것이 더 좋습니다. 하지만 테스트에서 렌더링된 컴포넌트의 `Props`을 업데이트하고 싶다면 이 함수를 사용하여 렌더링된 컴포넌트의 `Props`을 업데이트할 수 있습니다.

```tsx
import {render} from '@testing-library/react'

const {rerender} = render(<NumberDisplay number={1} />)

// 동일한 컴포넌트를 다른 Props로 렌더링하기
rerender(<NumberDisplay number={2} />)
```

### `unmount`

렌더링된 컴포넌트가 `unmount`됩니다. 페이지에서 컴포넌트가 제거되었을 때 어떤 일이 발생하는지 테스트할 때 유용합니다. (예: 이벤트 핸들러가 메모리 누수를 유발하지 않는지 테스트하는 경우)

> 이 메서드는 ReactDOM.unmountComponentAtNode에 대한 아주 작은 추상화입니다.

```tsx
import {render} from '@testing-library/react'

const {container, unmount} = render(<Login />)
unmount()
// 컴포넌트가 마운트 해제되었으며, container.innerHTML === ''
```

### `asFragment`

렌더링된 컴포넌트의 `DocumentFragment`를 반환합니다. `라이브 바인딩`을 피하고 컴포넌트가 이벤트에 어떻게 반응하는지 확인해야 할 때 유용할 수 있습니다.

```tsx
import React, {useState} from 'react'
import {render, fireEvent} from '@testing-library/react'

const TestComponent = () => {
  const [count, setCounter] = useState(0)

  return (
    <button onClick={() => setCounter(count => count + 1)}>
      Click to increase: {count}
    </button>
  )
}

const {getByText, asFragment} = render(<TestComponent />)
const firstRender = asFragment()

fireEvent.click(getByText(/Click to increase/))

// 이렇게 하면 첫 번째 렌더링과 클릭 이벤트 이후 DOM의 상태의 차이만 스냅샷합니다.
expect(firstRender).toMatchDiffSnapshot(asFragment())
```

### `cleanup`

렌더링으로 마운트된 React 트리를 마운트 해제합니다.

> 이 작업은 사용 중인 테스트 프레임워크가 `afterEach` 글로벌을 지원하고 테스트 환경에 주입된 경우 자동으로 수행됩니다. 그렇지 않은 경우 각 테스트 후에 수동으로 정리해야 합니다.

```tsx
import {cleanup, render} from '@testing-library/react'
import test from 'ava'

test.afterEach(cleanup)

test('renders into document', () => {
  render(<div />)
  // ...
})
```

> 🚨 `render`을 호출했을 때 `cleanup`을 호출하지 않으면 메모리 누수가 발생하고 테스트에서 디버깅하기 어려운 오류가 발생할 수 있습니다.

### `renderHook`

`custom test component`가 있는 렌더링에 대한 편리한 래퍼입니다. 이 API는 널리 사용되는 `테스트 패턴`에서 나온 것으로, 주로 `hooks`를 게시하는 라이브러리에서 유용하게 사용됩니다. `custom test component`를 사용하면 테스트하려는 항목이 추상화 뒤에 숨겨지지 않기 때문에 더 읽기 쉽고 강력한 테스트가 가능하므로 `render`을 선호해야 합니다.

```tsx
function renderHook<Result, Props>(
  render: (props: Props) => Result,
  options?: RenderHookOptions<Props>,
): RenderHookResult<Result, Props>
```

예)

```tsx
import {renderHook} from '@testing-library/react'

test('returns logged in user', () => {
  const {result} = renderHook(() => useLoggedInUser())
  expect(result.current).toEqual({name: 'Alice'})
})
```

#### `renderHook` Options

처음 호출될 때 `render` 콜백에 전달되는 프로퍼티를 선언합니다. 프로퍼티 없이 재렌더링을 호출하면 전달되지 않습니다.

```tsx
import {renderHook} from '@testing-library/react'

test('returns logged in user', () => {
  const {result, rerender} = renderHook((props = {}) => props, {
    initialProps: {name: 'Alice'},
  })
  expect(result.current).toEqual({name: 'Alice'})
  rerender()
  expect(result.current).toEqual({name: undefined})
})
```

> `renderHook`을 래퍼 및 `initialProps` 옵션과 함께 사용할 경우, `initialProps`는 래퍼 컴포넌트에 전달되지 않습니다. 래퍼 컴포넌트에 프로퍼티를 제공하려면 아래와 같이 사용해야 합니다.

```tsx
const createWrapper = (Wrapper, props) => {
  return function CreatedWrapper({ children }) {
    return <Wrapper {...props}>{children}</Wrapper>;
  };
};

...

{
  wrapper: createWrapper(Wrapper, { value: 'foo' }),
}
```

#### `renderHook` Result

`renderHook` 메서드는 프로퍼티가 있는 객체를 반환합니다.

**`result`**

렌더링 콜백의 가장 최근에 커밋된 반환 값의 값을 보유합니다.

```tsx
import {renderHook} from '@testing-library/react'

const {result} = renderHook(() => {
  const [name, setName] = useState('')
  React.useEffect(() => {
    setName('Alice')
  }, [])

  return name
})

expect(result.current).toBe('Alice')
```

값은 `result.current`에 보관됩니다. 결과는 가장 최근에 커밋된 값에 대한 참조라고 생각하면 됩니다.

**`rerender`**

이전에 렌더링된 렌더 콜백을 새 `Props`로 렌더링합니다.

```tsx
import {renderHook} from '@testing-library/react'

const {rerender} = renderHook(({name = 'Alice'} = {}) => name)

// 동일한 hooks를 다른 props로 다시 렌더링하기
rerender({name: 'Bob'})
```

