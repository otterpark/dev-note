# useEffect()

`useEffect()`는 컴포넌트를 외부 시스템과 동기화할 수 있는 React hook입니다.

## Reference

```tsx
useEffect(setup, dependencies?)
```

- 컴포넌트 최상위에서 `useEffect`를 사용하여, 이펙트를 선언합니다.

### 1. Parameters

### `setup`

이펙트의 로직이 포함된 함수입니다. `setup` 함수는 선택적으로 `cleanup` 함수를 반환할 수도 있습니다. 컴포넌트가 DOM에서 처음 추가되면 React는 `setup` 함수를 실행합니다. 변경된 종속성으로 리렌더링 할때마다 React는 먼저 이전 값으로 `cleanup` 함수를 실행한 다음 새 값으로 `setup`함수를 실행합니다. 컴포넌트가 DOM에서 제거된 후 React는 마지막 `cleanup` 함수를 실행합니다.

### 선택적 의존성

`setup` 코드 내에서 참조된 모든 `reactive values`의 목록입니다. `reactive values`에는 `Props`, `state`, `컴포넌트 본문 내부에서 직접 선언된 모든 변수와 함수`가 포함됩니다. `lint`가 `React Config`로 구성된 경우, 모든 `reactive values`이 종속성으로 올바르게 지정되었는지 확인합니다. 의존성 목록에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`과 같이 인라인으로 작성해야 합니다. React는 `Object.is` 비교를 사용하여 각 의존성을 이전 값과 비교합니다. 이 인수를 생략하면 컴포넌트를 다시 렌더링할 때마다 `Effect`가 다시 실행됩니다. 의존성 배열을 전달할 때와 빈 배열을 전달할 때, 그리고 의존성을 전혀 전달하지 않을 때의 차이를 확인해 보세요.

### 2. Return

useEffect는 `undefined`를 리턴한다.

### 3. 주의사항

#### 1. 💡 `useEffect()`은 hook이므로, 컴포넌트의 최상위 레벨이나 자체 hook에서만 호출할 수 있다

루프나 조건 내부에서는 호출할 수 없습니다. 필요한 경우 새 컴포넌트를 추출하고 상태를 그 안으로 옮겨야합니다.

#### 2. 💡 외부 시스템과 동기화하려는 것이 아니라면 `Effect`가 필요하지 않을 수 있습니다

#### 3. 💡 `StrictMode`가 켜져 있으면 React는 첫 번째 실제 설정 전에 개발 전용 `setup`+ `cleanup` 사이클을 한 번 더 실행합니다

`cleanup` 로직이 `setup` 로직을 `미러링`하고 설정이 수행 중인 모든 작업을 중지하거나 취소하는지 확인하기 위한 `스트레스 테스트(신중하고 면밀한 테스트)`입니다. 문제가 발생하면 `cleanup` 기능을 구현하세요.

#### 4. 💡 의존성 중 일부가 컴포넌트 내부에 정의된 객체 또는 함수인 경우, `Effect`가 필요 이상으로 자주 다시 실행될 위험이 있습니다

 이 문제를 해결하려면 불필요한 객체 및 함수 의존성을 제거하세요. 또한 `상태 업데이트(extract state updates)`와 `비반응형 로직(non-reactive logic)`을 이펙트 외부로 추출할 수도 있습니다.

#### 5. 💡 `Effect` 가 시각적인 작업을 하고 있는 중 지연이 눈에 띄는 경우(깜박임 현상) `useEffect` => `useLayoutEffect`로 교체하세요

클릭과 같은 상호작용으로 인해 이펙트가 발생한 것이 아니라면, React는 `Effect`를 실행하기 전에 브라우저가 업데이트된 화면을 먼저 그리도록 할 것입니다.

#### 6. 💡 브라우저가 화면을 다시 그리는 것을 차단해야 하는 경우 `useEffect`를 `useLayoutEffect`로 교체하세요

클릭과 같은 상호작용으로 인해 이펙트가 발생했더라도 브라우저는 이펙트 내부의 상태 업데이트를 처리하기 전에 화면을 다시 그릴 수 있습니다.

#### 7. 💡 `Effect`는 클라이언트에서만 실행되며 서버 렌더링에서는 실행이 불가합니다

## 사용법

### 외부 시스템에 연결

일부 컴포넌트는 페이지에 표시되는 동안 `네트워크`, `일부 브라우저 API` 또는 `타사 라이브러리`에 연결된 상태를 유지해야 합니다. 이러한 시스템은 React에서 제어되지 않으므로 `외부(external)`라고 합니다.

컴포넌트를 외부 시스템에 연결하려면 컴포넌트의 최상위 수준에서 `useEffect`를 호출하세요.

```tsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
   const connection = createConnection(serverUrl, roomId);
    connection.connect();
   return () => {
      connection.disconnect();
   };
  }, [serverUrl, roomId]);
  // ...
}
```

useEffect에 두 개의 인수를 전달해야 합니다.

1. 해당 시스템에 연결하는 `setup` 코드가 포함된 설정 함수.
  
    > 해당 시스템과의 연결을 끊는 정리 코드가 포함된 `cleanup` 함수를 반환해야 합니다.
  
2. 해당 함수 내부에서 사용된 컴포넌트의 모든 값을 포함한 의존성 목록.

**React는 필요할 때마다 `setup` 및 `cleanup` 함수를 호출하며, 이는 여러 번 발생할 수 있습니다.**

1. 컴포넌트가 페이지에 추가(마운트)될 때 설정 코드가 실행됩니다.

2. 의존성이 변경된 컴포넌트를 다시 렌더링할 때마다 실행됩니다.
    - 먼저 정리 코드가 이전 프로퍼티와 상태로 실행됩니다.
    - 그런 다음 설정 코드가 새 프로퍼티와 상태로 실행됩니다.

3. 컴포넌트가 페이지에서 제거(마운트 해제)된 후 `cleanup` 코드가 마지막으로 한 번 실행됩니다.

위 순서에 대해 설명해 보겠습니다.

위의 `ChatRoom` 컴포넌트가 페이지에 추가되면 초기 `serverUrl` 및 `roomId`를 사용하여 채팅방에 연결됩니다. 다시 렌더링한 결과 `serverUrl` 또는 `roomId`가 변경되면(예: 사용자가 드롭다운에서 다른 채팅방을 선택하는 경우) `Effect`는 이전 채팅방과의 연결을 끊고 다음 채팅방에 연결합니다. 채팅방 컴포넌트가 페이지에서 제거되면 `Effect`가 마지막으로 연결이 끊어집니다.

버그를 찾는 데 도움을 주기 위해 개발 단계에서 React는 설정 전에 `setup` 및 `cleanup`을 한 번 더 실행합니다. 이는 `Effect`의 로직이 올바르게 구현되었는지 확인하는 `스트레스 테스트(신중하고 면밀한 테스트)`입니다. 이로 인해 눈에 보이는 문제가 발생한다면 `cleanup` 함수에 일부 로직이 누락된 것입니다. `cleanup` 기능은 `setup` 기능이 수행하던 작업을 중지하거나 실행 취소해야 합니다. 경험상 사용자가 프로덕션에서와 같이 한 번 호출되는 설정과 개발에서와 같이 `setup` → `cleanup` → `setup` 순서를 구분할 수 없어야 합니다. 일반적인 해결 방법을 참조하세요.

모든 `Effect`를 독립적인 프로세스로 작성하고 한 번에 하나의`setup`/`cleanup` 주기를 생각하세요. 컴포넌트가 `마운트`, `업데이트`, `마운트 해제 중인지 여부`는 중요하지 않아야 합니다. `cleanup` 로직이 `setup` 로직을 올바르게 `미러링`하면 필요한 만큼 자주 `setup`과 `cleanup`를 실행할 수 있는 탄력적인 효과가 됩니다.

`Effect`를 사용하면 **구성 요소를 일부 외부 시스템(예: 채팅 서비스)과 동기화** 할 수 있습니다. 여기서 외부 시스템은 다음과 같이 React에 의해 제어되지 않는 모든 코드를 의미합니다.

- `setInterval()` 및 `clearInterval()`로 관리되는 타이머.
- `window.addEventListener()` 및 `window.removeEventListener()`를 사용하는 이벤트 구독.
- `animation.start()` 및 `animation.reset()`과 같은 API가 있는 타사 애니메이션 라이브러리.

외부 시스템에 연결하지 않는 경우 Effect가 필요하지 않을 수 있습니다.

## 커스텀 훅의 래핑 효과

`Effect`는 `탈출구`입니다. `"React를 벗어나야 할 때"`, 그리고 `사용 사례에 더 나은 내장 솔루션이 없을 때 사용`합니다. `Effects`를 수동으로 작성해야 하는 경우가 자주 발생한다면, 일반적으로 컴포넌트가 의존하는 일반적인 동작에 대한 사용자 정의 hook을 추출해야 한다는 신호입니다.

예를 들어, 이 `useChatRoom` 커스텀 hook은 보다 선언적인 API 뒤에 `Effect`의 로직을 `"숨깁니다"`:

```tsx
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

그런 다음 모든 컴포넌트에서 이와 같이 사용할 수 있습니다.

```tsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

또한 `React 생태계`에는 모든 용도에 맞는 훌륭한 커스텀 Hook이 많이 있습니다.

## React가 아닌 외부 위젯 제어하기

때로는 외부 시스템을 컴포넌트의 일부 `Props`나 `State`를 동기화시키고 싶을 때가 있습니다.

예를 들어, React 없이 작성된 타사 맵 위젯이나 비디오 플레이어 컴포넌트가 있다면 `Effect`를 사용해 그 상태를 React 컴포넌트의 현재 상태와 일치시키는 메서드를 호출할 수 있습니다. 이 `Effect`는 `map-widget.js`에 정의된 `MapWidget` 클래스의 인스턴스를 생성합니다. `Map` 컴포넌트의 `zoomLevel` 프로퍼티를 변경하면 `Effect`는 클래스 인스턴스에서 `setZoom()`을 호출하여 **동기화 상태**를 유지합니다

### App.js

```tsx
import { useState } from 'react';
import Map from './Map.js';

export default function App() {
  const [zoomLevel, setZoomLevel] = useState(0);
  return (
    <>
      Zoom level: {zoomLevel}x
      <button onClick={() => setZoomLevel(zoomLevel + 1)}>+</button>
      <button onClick={() => setZoomLevel(zoomLevel - 1)}>-</button>
      <hr />
      <Map zoomLevel={zoomLevel} />
    </>
  );
}
```

### Map.js

```tsx
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return (
    <div
      style={{ width: 200, height: 200 }}
      ref={containerRef}
    />
  );
}
```

### map-widget.js

```tsx
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return (
    <div
      style={{ width: 200, height: 200 }}
      ref={containerRef}
    />
  );
}
```

이 예제에서는 `MapWidget` 클래스가 자신에게 전달된 DOM 노드만 관리하기 때문에 `cleanup` 함수가 필요하지 않습니다. `Map` React 컴포넌트가 트리에서 제거된 후, DOM 노드와 `MapWidget` 클래스 인스턴스는 브라우저 자바스크립트 엔진에 의해 **자동으로 가비지 수집**됩니다.

## Effect 데이터 가져오기

`Effect`를 사용하여 컴포넌트에 대한 데이터를 가져올 수 있습니다. 프레임워크를 사용하는 경우 프레임워크의 데이터 불러오기 메커니즘을 사용하는 것이 효과를 수동으로 작성하는 것보다 훨씬 효율적이라는 점에 유의하세요.

`Effect`에서 수동으로 데이터를 가져오려면 다음과 같은 코드가 필요합니다.

```tsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  // ...
```

`ignore` 변수는 거짓으로 초기화되고 정리 중에 참으로 설정됩니다. 이렇게 하면 네트워크 응답이 보낸 순서와 다른 순서로 도착할 수 있는 `'경합 조건(race conditions)'`이 코드에 발생하지 않습니다.

```tsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

`async / await` 구문을 사용하여 다시 작성할 수도 있지만 여전히 `cleanup` 함수를 제공해야 합니다.

`Effects`에서 **직접 데이터를 가져오는 작업을 반복적으로 작성하면 나중에 캐싱 및 서버 렌더링과 같은 최적화를 추가하기가 어려워**집니다. 직접 만들거나 커뮤니티에서 유지 관리하는 `커스텀 Hook`을 사용하는 것이 더 쉽습니다.

### 💡 Effects에서 데이터 가져오기를 대체할 수 있는 좋은 대안은 무엇인가요?

클라이언트 측 앱에서 데이터를 가져오기 위해 `Effects` 내에서 가져오기 호출을 작성하는 것은 널리 사용되는 방법입니다. 하지만 이는 **매우 수동적인 접근 방식**이며 **상당한 단점**이 있습니다.

- `Effects`는 서버에서 실행되지 않습니다. 즉, 초기 서버에서 렌더링되는 HTML에는 데이터가 없는 **로딩 상태만 포함**됩니다. 클라이언트 컴퓨터는 모든 JavaScript를 다운로드하고 앱을 렌더링해야만 이제 데이터를 로드해야 한다는 것을 알게 됩니다. 이는 매우 효율적이지 않습니다.
- `Effects`에서 직접 가져오기를 사용하면 `"네트워크 워터폴 - Waterfall"`를 쉽게 만들 수 있습니다. 부모 컴포넌트를 렌더링하면 부모 컴포넌트가 일부 데이터를 가져오고, 자식 컴포넌트를 렌더링하면 자식 컴포넌트가 데이터를 가져오기 시작합니다.
- 네트워크가 매우 빠르지 않은 경우 모든 데이터를 병렬로 가져오는 것보다 훨씬 느립니다.
효과에서 직접 가져오는 것은 일반적으로 데이터를 미리 로드하거나 캐시하지 않는다는 것을 의미합니다. 예를 들어 컴포넌트가 마운트를 해제했다가 다시 마운트하면 데이터를 다시 가져와야 합니다.
- `'경합 조건(race conditions)'` `fetch`와 같은 버그가 발생하지 않는 방식으로 패치 호출을 작성하려면 상용구 코드가 상당히 많이 필요합니다.

이러한 단점들은 React에만 국한된 것이 아닙니다. 모든 라이브러리를 사용한 마운트에서 데이터를 가져올 때 적용됩니다. **라우팅과 마찬가지로 데이터 불러오기도 제대로 수행하기가 쉽지 않으므로** 다음과 같은 접근 방식을 권장합니다:

1. 프레임워크를 사용하는 경우, 프레임워크에 내장된 데이터 불러오기 메커니즘을 사용하세요. 최신 React 프레임워크에는 효율적이고 위의 함정이 발생하지 않는 통합 데이터 불러오기 메커니즘이 있습니다.
2. 그렇지 않으면 클라이언트 측 캐시를 사용하거나 빌드하는 것을 고려하세요. 인기 있는 오픈 소스 솔루션으로는 `React Query`, `useSWR`, `React Router 6.4+` 등이 있습니다. 자체 솔루션을 구축할 수도 있는데, 이 경우 내부적으로 `Effects`를 사용하되 `요청 중복 제거`, `응답 캐싱`, `네트워크 워터폴 방지(데이터를 미리 로드하거나 데이터 요구 사항을 경로에 올려서)`를 위한 로직을 추가할 수 있습니다.

이 두 가지 접근 방식이 모두 적합하지 않은 경우 `Effects`에서 직접 데이터를 계속 가져올 수 있습니다.

### 반응형 의존성 지정

`Effect`의 의존성을 `"선택"`할 수 없다는 점에 유의하세요. `Effect`의 코드에서 사용되는 모든 반응형 값은 의존성으로 선언해야 합니다. `Effect`의 의존성 목록은 주변 코드에 의해 결정됩니다.

```tsx
function ChatRoom({ roomId }) { // 반응형 값
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // 반응형 값

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Effect는 두 반응형 값을 읽는다.
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // ✅ 따라서 Effect의 의존성으로 지정해야 합니다.
  // ...
}
```

`serverUrl` 또는 `roomId`가 변경되면 `Effect`가 새 값을 사용하여 채팅에 다시 연결됩니다.

`반응형 값`에는 `Props`와 `컴포넌트 내부에서 직접 선언된 모든 변수 및 함수가 포함`됩니다. `roomId`와 `serverUrl`은 `반응형 값`이므로 의존성에서 제거할 수 없습니다. 이 값을 생략하려고 시도할 때 `linter`가 React에 대해 올바르게 구성되면, `linter`는 이를 수정해야 하는 실수로 플래그를 지정합니다.

```tsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React hook useEffect에 누락된 종속성: 'roomId'와 'serverUrl'
  // ...
}
```

의존성을 제거하려면 `linter`가 의존성이 될 필요가 없다는 것을 `"증명"`해야 합니다. 예를 들어 `serverUrl`을 컴포넌트 밖으로 이동하여 `반응형이 아니며 재렌더링 시 변경되지 않음`을 증명할 수 있습니다.

```tsx
const serverUrl = 'https://localhost:1234'; // 더 이상 반응형 값이 아닙니다.

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 선언된 모든 의존성
  // ...
}
```

이제 `serverUrl`은 반응형 값이 아니므로(그리고 다시 렌더링할 때 변경할 수 없으므로) 의존성이 될 필요가 없습니다. `Effect`의 코드가 반응형 값을 사용하지 않는다면 의존성 목록은 비어 있어야 합니다(`[]`)

```tsx
const serverUrl = 'https://localhost:1234'; // 더 이상 반응형 값이 아닙니다.
const roomId = 'music'; // 더 이상 반응형 값이 아닙니다.

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 선언된 모든 의존성
  // ...
}
```

빈 의존성이 있는 `Effect`는 컴포넌트의 `Props`나 `State`가 변경되어도 다시 실행되지 않습니다.

### `Effect`의 이전 상태를 기반으로 상태 업데이트하기

`Effect`의 이전 상태를 기반으로 상태를 업데이트하려는 경우 문제가 발생할 수 있습니다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // 매초마다 카운터를 증가시키려면...
    }, 1000)
    return () => clearInterval(intervalId);
  }, [count]); // 🚩 ... 하지만 의존성으로 'count'를 지정하면 항상 간격이 재설정됩니다.
  // ...
}
```

`count`는 반응형 값이므로 의존성 목록에 지정해야 합니다. 하지만 이렇게 하면 카운트가 변경될 때마다 `Effect`를 정리하고 다시 설정해야 합니다. 이는 이상적이지 않습니다.

이 문제를 해결하려면 `c => c + 1` 상태 업데이터를 `setCount`에 전달하세요.

```tsx
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1); // ✅ state 업데이터가 통과
    }, 1000);
    return () => clearInterval(intervalId);
  }, []); // ✅ count는 의존성이 없다.

  return <h1>{count}</h1>;
}
```

이제 `count + 1` 대신 `c => c + 1`을 전달하므로 `Effect`는 더 이상 `count`에 의존할 필요가 없습니다. 이 수정으로 `count`가 변경될 때마다 간격을 다시 정리하고 설정할 필요가 없습니다.

### 불필요한 객체 종속성 제거하기

`Effect`가 렌더링 중에 생성된 `객체 또는 함수에 의존하는 경우` 너무 자주 실행될 수 있습니다. 예를 들어, 이 `Effect`는 **렌더링할 때마다 옵션 객체가 다르기 때문에 렌더링할 때마다 다시 연결**됩니다.

```tsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 이 object는 리렌더링할 때마다 처음부터 새로 생성
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // Effect 내부에서 사용
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 결과적으로 이러한 의존성은 리렌더링할 때마다 항상 달라집니다.
  // ...
```

렌더링 중에 생성된 오브젝트를 의존성으로 사용하지 마세요. 대신 `Effect` 내에서 오브젝트를 생성하세요.

#### App.js

```tsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

#### Chart.js

```tsx
export function createConnection({ serverUrl, roomId }) {
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

이제 `Effect` 내에서 `createOptions` 함수를 정의하면 `Effect` 자체는 `roomId` 문자열에만 의존합니다. 이 수정으로 입력을 입력해도 채팅이 다시 연결되지 않습니다. 다시 생성되는 함수와 달리 `roomId`와 같은 문자열은 다른 값으로 설정하지 않는 한 변경되지 않습니다. (의존성 제거 방법 체크)

### `Effect`에서 최신 `Props` 및 `State` 읽기

> React에 출시되지 않은 실험적인 API에 대해 설명합니다.

기본적으로 `Effect`에서 반응형 값을 읽을 때는 의존성으로 추가해야 합니다. 이렇게 하면 `Effect`가 해당 값의 모든 변경에 "반응"하도록 할 수 있습니다. 대부분의 의존성에서 원하는 동작입니다.

그러나 때로는 '반응'하지 않고 `Effect`에서 최신 `Props`과 `State`를 읽고 싶을 때가 있습니다. 예를 들어 페이지 방문 시마다 장바구니에 있는 품목의 수를 기록한다고 가정해 보겠습니다.

```tsx
function Page({ url, shoppingCart }) {
  useEffect(() => {
    logVisit(url, shoppingCart.length);
  }, [url, shoppingCart]); // ✅ 선언된 모든 의존성
  // ...
}
```

URL이 변경될 때마다 새 페이지 방문을 기록하되 `shoppingCart`가 변경되는 경우는 기록하지 않으려면 어떻게 해야 하나요? 반응성 규칙을 위반하지 않고 `shoppingCart`를 의존성에서 제외할 수는 없습니다. 그러나 코드가 `Effect` 내부에서 호출되더라도 변경 사항에 '반응'하지 않도록 표현할 수 있습니다. `useEffectEvent Hook`을 사용하여 `Effect Event`를 선언하고 그 안에 `shoppingCart`를 읽는 코드를 이동합니다.

```tsx
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, shoppingCart.length)
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ 선언된 모든 의존성
  // ...
}
```

`Effect event`는 반응형이 아니므로 `Effect`의 의존성에서 항상 생략해야 합니다. 이를 통해 비반응성 코드(일부 `Props` 및 `State`의 최신 값을 읽을 수 있는 코드)를 그 안에 넣을 수 있습니다. `onVisit` 내부에서 `shoppingCart`를 읽으면 `shoppingCart`가 `Effect`를 다시 실행하지 않도록 할 수 있습니다.

### 서버와 클라이언트에 서로 다른 콘텐츠 표시하기

앱에서 서버 렌더링을 사용하는 경우(직접 또는 프레임워크를 통해) 컴포넌트는 두 가지 다른 환경에서 렌더링됩니다. 서버에서는 초기 HTML을 생성하기 위해 렌더링됩니다. 클라이언트에서 React는 렌더링 코드를 다시 실행하여 이벤트 핸들러를 해당 HTML에 첨부할 수 있도록 합니다. 그렇기 때문에 `하이드레이션(서버의 초기 HTML 스냅샷을 브라우저에서 실행되는 완전한 인터랙티브 앱으로 전환)`이 작동하려면 클라이언트와 서버에서 초기 렌더링 출력이 동일해야 합니다.

드물지만 클라이언트에 다른 콘텐츠를 표시해야 하는 경우가 있을 수 있습니다. 예를 들어 앱이 `localStorage`에서 일부 데이터를 읽는 경우 서버에서는 이를 수행할 수 없습니다. 이를 구현하는 방법은 다음과 같습니다.

```tsx
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... 클라이언트 전용 JSX 반환 ...
  }  else {
    // ... 초기 JSX 반환 ...
  }
}
```

앱이 로드되는 동안 사용자는 **초기 렌더링 출력**을 볼 수 있습니다. 그런 다음 앱이 로드되고 `하이드레이션(서버의 초기 HTML 스냅샷을 브라우저에서 실행되는 완전한 인터랙티브 앱으로 전환)`되면 `Effect`가 실행되고 `didMount`를 `true`로 설정하여 리렌더링을 트리거합니다. 그러면 클라이언트 전용 렌더링 출력으로 전환됩니다. `Effect`는 서버에서 실행되지 않으므로 초기 서버 렌더링 중에 `didMount`가 `false`인 이유입니다.

이 패턴은 아껴서 사용하세요. 연결 속도가 느린 사용자는 초기 콘텐츠를 꽤 오랜 시간(수 초) 동안 보게 되므로 컴포넌트의 모양을 갑작스럽게 변경하고 싶지 않을 것입니다. 대부분의 경우 CSS를 사용하여 조건부로 다른 것을 표시함으로써 이러한 필요성을 피할 수 있습니다.
