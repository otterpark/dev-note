# useContext()

`useContext()`은 컴포넌트에서 컨텍스트를 읽고 구독할 수 있는 React Hook입니다.

> 즉, `props drilling`을 해결하고자 나온 Hook

## Reference

```tsx
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

- 컴포넌트의 최상위 수준에서 `useContext`를 호출하여 컨텍스트를 읽고 구독한다.

### 1. Parameters

#### SomeContext

이전에 `createContext`로 생성한 컨텍스트입니다. 컨텍스트 자체는 정보를 보유하지 않으며, **컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류**를 나타낼 뿐입니다.

### 2. Return

`useContext`는 호출하는 컴포넌트에 대한 컨텍스트 값을 반환합니다. 이 값은 **트리에서 호출 컴포넌트 위에 있는 가장 가까운 일부 컨텍스트 `Provider`에 전달된 값으로 결정**됩니다. 이러한 `Provider`가 없는 경우 반환되는 값은 해당 컨텍스트에 대해 `createContext`에 전달한 `defaultValue`가 됩니다. 반환된 값은 항상 **최신 값**입니다. **컨텍스트가 변경**되면 React는 **일부 컨텍스트를 읽는 컴포넌트를 자동으로 다시 렌더링**합니다.

### 3. 주의 사항

#### 💡 컴포넌트의 `useContext()` 호출은 동일한 컴포넌트에서 반환된 `Provider`의 영향을 받지 않는다

해당 `<Context.Provider>`는 `useContext()` 호출을 수행하는 컴포넌트 위에 있어야 합니다.

#### 💡 React는 다른 값을 받는 `Provider`로부터 시작해서 특정 컨텍스트를 사용하는 모든 자식들을 자동으로 `리렌더링`합니다

이전 값과 다음 값은 `Object.is` 비교를 통해 비교됩니다. **`memo`로 리렌더링을 건너뛰어도 자식들이 새로운 컨텍스트 값을 받는 것을 막지는 못합니다**.

#### 💡 빌드 시스템이 출력에 중복 모듈을 생성하는 경우(`symlinks`에서 발생할 수 있음) 컨텍스트가 손상될 수 있습니다

컨텍스트를 통해 무언가를 전달하는 것은 컨텍스트를 제공하는 데 사용하는 일부 컨텍스트와 컨텍스트를 읽는 데 사용하는 일부 컨텍스트가 `===` 비교에 의해 결정되는 정확히 동일한 객체인 경우에만 작동합니다.

## 사용법

### 트리에 깊이 있는 데이터 전달

컴포넌트의 최상위 수준에서 `useContext`를 호출하여 컨텍스트를 읽고 구독하세요.

```tsx
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

`useContext`는 전달한 컨텍스트에 대한 컨텍스트 값을 반환합니다. 컨텍스트 값을 결정하기 위해 React는 **컴포넌트 트리를 검색하고 특정 컨텍스트에 대해 위에서 가장 가까운 컨텍스트 `Provider`를 찾습니다**.

컨텍스트를 버튼에 전달하려면 버튼 또는 해당 상위 컴포넌트 중 하나를 해당 컨텍스트 제공업체로 래핑합니다.

```tsx
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... renders buttons inside ...
}
```

`Provider`와 Button 사이에 얼마나 많은 컴포넌트 레이어가 있는지는 중요하지 않습니다. `Form` 내부의 버튼이 `useContext(ThemeContext)`를 호출하면 `"dark"`를 값으로 받습니다.

> 💡 TIP
>
> `useContext()`는 항상 이를 호출하는 컴포넌트 위에서 가장 가까운 `Provider`를 찾습니다. 위쪽으로 검색하며, `useContext()`를 호출하는 컴포넌트 내의 `Provider`는 고려하지 않습니다.

### 컨텍스트를 통해 전달된 데이터 업데이트

시간이 지남에 따라 컨텍스트가 변경되기를 원하는 경우가 종종 있습니다. 컨텍스트를 업데이트하려면 컨텍스트를 상태와 결합하세요. 부모 컴포넌트에서 `state` 변수를 선언하고 현재 상태를 컨텍스트 값으로 `Provider`에게 전달합니다.

```tsx
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
        Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

이제 `Provider` 내부의 모든 `Button`이 현재 `theme` 값을 받게 됩니다. `Provider`에게 전달한 `theme` 값을 업데이트하기 위해 `setTheme`를 호출하면 모든 `Button` 컴포넌트가 새로운 `'light'` 값으로 리렌더링됩니다.

### 대체 기본값 지정

React가 부모 트리에서 특정 컨텍스트의 `Provider`를 찾을 수 없는 경우, `useContext()`가 반환하는 컨텍스트 값은 해당 컨텍스트를 생성할 때 지정한 기본값과 동일합니다.

```tsx
const ThemeContext = createContext(null);
```

기본값은 **절대 변경되지 않습니다**. 컨텍스트를 업데이트하려면 위에서 설명한 대로 `state`와 함께 사용하세요.

종종 `null` 대신에 기본값으로 사용할 수 있는 더 의미 있는 값이 있을 수 있습니다.

```tsx
const ThemeContext = createContext('light');
```

이렇게 하면 실수로 해당 `Provider` 없이 일부 컴포넌트를 렌더링해도 깨지지 않습니다. 또한 테스트 환경에서 많은 `Provider`를 설정하지 않고도 컴포넌트가 테스트 환경에서 잘 작동하는 데 도움이 됩니다.

### 트리의 일부에 대한 컨텍스트 재정의

트리의 일부분을 다른 값의 공급자로 래핑하여 해당 부분에 대한 컨텍스트를 재정의할 수 있습니다.

```tsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

필요한 만큼 공급자를 중첩하고 재정의할 수 있습니다.

> 보통 의미없는 리렌더링을 제한하기 위해 감써서 많이 사용한다.

### `Object`와 `Function`을 전달할 때 리렌더링 최적화하기

```tsx
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

여기서 컨텍스트 값은 두 개의 프로퍼티를 하나는 객체이며 또 다른 하나는 함수입니다. `MyApp`이 리렌더링할 때마다(예를 들어 경로 업데이트 시) 다른 함수를 가리키는 다른 객체가 될 것이므로 React는 `useContext(AuthContext)`를 호출하는 **트리 깊숙한 곳에 있는 모든 컴포넌트도 리렌더링**해야 합니다.

작은 앱에서는 문제가 되지 않습니다. 그러나 `currentUser`와 같은 기본 데이터가 변경되지 않았다면 리렌더링할 필요가 없습니다. React가 이 사실을 활용할 수 있도록 `login` 함수를 `useCallback`으로 감싸고 객체 생성을 `useMemo`로 감쌀 수 있습니다. 이것은 성능 최적화를 위한 것입니다.

```tsx
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

이 변경으로 인해 `MyApp`이 리렌더링해야 하는 경우에도 `currentUser`가 변경되지 않는 한 `useContext(AuthContext)`를 호출하는 컴포넌트는 리렌더링할 필요가 없습니다.

## 문제 해결

### 내 컴포넌트에 `Provider`의 값이 표시되지 않습니다

이런 일이 발생하는 몇 가지 일반적인 방법이 있습니다

1. `useContext()`를 호출하는 컴포넌트와 동일한 컴포넌트(또는 그 아래)에서 `<SomeContext.Provider>`를 렌더링하는 경우. `<SomeContext.Provider>`를 `useContext()`를 호출하는 컴포넌트의 위와 바깥으로 이동하세요.
2. 컴포넌트를 `<SomeContext.Provider>`로 감싸는 것을 잊었거나 생각했던 것과 다른 트리의 다른 부분에 배치했을 수 있습니다. React 개발자 도구를 사용하여 계층 구조가 올바른지 확인하세요.
3. 툴링에서 빌드 문제가 발생하여 제공하는 컴포넌트에서 보는 일부 컨텍스트와 읽는 컴포넌트에서 보는 일부 컨텍스트가 두 개의 다른 객체가 될 수 있습니다. 예를 들어 `심볼릭 링크`를 사용하는 경우 이런 문제가 발생할 수 있습니다.

    이를 확인하려면 `window.SomeContext1` 및 `window.SomeContext2`와 같은 전역에 할당하고 콘솔에서 `window.SomeContext1` === `window.SomeContext2`인지 확인하면 됩니다. 동일하지 않은 경우 빌드 도구 수준에서 해당 문제를 수정하세요.

### 기본값이 다른데도 항상 내 컨텍스트에서 정의되지 않은 값을 얻습니다

```tsx
// ✅ value를 사용하여 프로퍼티에 값을 지정
<ThemeContext.Provider value={theme}>
   <Button />
</ThemeContext.Provider>
```

`createContext(defaultValue)` 호출의 기본값은 위에 일치하는 `Provider`가 전혀 없는 경우에만 사용된다는 점에 유의하세요. 부모 트리 어딘가에 `<SomeContext.Provider value={undefined}>` 컴포넌트가 있는 경우, `useContext(SomeContext)`를 호출하는 컴포넌트는 정의되지 않은 값을 컨텍스트 값으로 받습니다.
