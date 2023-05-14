# API Queries

쿼리는 페이지에서 요소를 찾기 위해 `Testing Library`에서 제공하는 메서드입니다.  요소를 선택한 후에는 `Event API` 또는 `user-event`를 사용하여 이벤트를 실행하고 페이지와의 사용자 상호 작용을 시뮬레이션하거나 `Jest` 및 `jest-dom`을 사용하여 요소에 대한 `assertions`을 만들 수 있습니다.

쿼리와 함께 작동하는 `Testing Library` 헬퍼 메서드가 있습니다. 요소는 동작에 대한 응답으로 나타나고 사라지므로 `waitFor` 또는 `findBy` 쿼리와 같은 비동기 API를 사용하여 DOM의 변경을 기다릴 수 있습니다. 특정 엘리먼트의 자식인 엘리먼트만 찾으려면 `within`를 사용할 수 있습니다. 필요한 경우 재시도 시간 제한 및 기본 `testID 속성`과 같은 몇 가지 옵션도 구성할 수 있습니다.

```tsx
import {render, screen} from '@testing-library/react';

test('should show login form', () => {
  render(<Login />)
  const input = screen.getByLabelText('Username')
  // Events and assertions...
})
```

`DOM Testing Library`의 기본 쿼리는 컨테이너를 첫 번째 인수로 전달해야 합니다. `Testing Library`는 컴포넌트를 렌더링할 때 이러한 쿼리의 사전 바인딩된 버전을 제공하므로 컨테이너를 제공할 필요가 없습니다.

> 최대한 `screen`을 사용하는 것을 권장한다.

## `queryOptions`

쿼리 유형과 함께 쿼리 옵션 객체를 전달할 수 있습니다.

## `screen`

`DOM Testing Library`에서 내보내는 모든 쿼리는 컨테이너를 첫 번째 인수로 받습니다. 문서 본문 전체를 쿼리하는 것이 매우 일반적이기 때문에, `DOM Testing Library`는 문서 본문으로 미리 바인딩된 모든 쿼리가 포함된 화면 객체도 내보냅니다(`within 함수 사용`). `React Testing Library`와 같은 래퍼는 화면을 다시 내보내므로 동일한 방식으로 사용할 수 있습니다.

`screen`도 디버그 방법들이 있습니다.

### `screen.debug()`

편의를 위해 화면에는 쿼리 외에 디버그 메서드도 노출됩니다. 이 메서드는 기본적으로 `console.log(prettyDOM())`의 단축키입니다. 이 메서드는 문서, 단일 요소 또는 요소 배열 디버깅을 지원합니다.

```tsx
import {screen} from '@testing-library/dom'

document.body.innerHTML = `
  <button>test</button>
  <span>multi-test</span>
  <div>multi-test</div>
`

// debug document
screen.debug()
// debug single element
screen.debug(screen.getByText('test'))
// debug multiple elements
screen.debug(screen.getAllByText('multi-test'))
```

### `screen.logTestingPlaygroundURL()`

`testing-playground`를 사용하여 디버깅하는 경우 브라우저에서 열 수 있는 URL을 로깅하고 반환하는 편리한 방법을 화면에 표시합니다.

```tsx
import {screen} from '@testing-library/dom'

document.body.innerHTML = `
  <button>test</button>
  <span>multi-test</span>
  <div>multi-test</div>
`

// log entire document to testing-playground
screen.logTestingPlaygroundURL()
// log a single element
screen.logTestingPlaygroundURL(screen.getByText('test'))
```

[testing-playground](https://testing-playground.com)
