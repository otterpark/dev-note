# JSX란?

`JSX`는 `JavaScript` 파일 안에 `HTML`과 유사한 마크업을 작성할 수 있게 해주는 `JavaScript`용 구문 확장자이다.
> ECMAScript의 XML과 유사한 구문 확장

## JSX의 특징

### 1. 반드시 하나의 root Element로 반환해야한다

컴포넌트 여러 요소를 반환하기 위해서 단일 부모 태그로 요소를 래핑하여 반환해야합니다.

```jsx
<div>
  <h1>Hedy Lamarr's Todos</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```

여기서의 `div`는 `root Element`이며, `div`에 의미 없이 `root Element`를 넣고 싶을 때 `Fragment(<> </>)`로 감싸서 처리해도 된다.

```jsx
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

### 2. 모든 태그들은 닫아야한다

JSX에서는 태그가 명시적으로 닫혀야 한다.

```jsx
<>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
   />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```

### 3. camelCase로 작성되어야 한다.(ex - className)

JSX는 JavaScript로 바뀌며, JSX로 작성된 속성들은 JavaScript 객체의 키가 됩니다. 따라서 JavaScript의 변수 이름 제한에 따라 대시(`/`) 또는 예약어를 사용할 수 없습니다. React에서는 많은 HTML과 SVG 속성들을 모두 camelCase로 사용합니다.

```jsx
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo"
/>
```

## 왜 React에서는 JSX를 사용할까?

React에서는 JSX 사용이 필수는 아니지만 JavaScript 코드 안에서 **UI 관련 작업을 할때 시각적으로 더 도움**을 준다고 합니다. 컴포넌트를 작성하는 다른 방법들이 있지만 대부분의 리엑트 개발자들은 **`JSX`의 간결함**을 선호합니다.

## React에서 JSX는 어떻게 사용되나?

브라우저는 즉시 JSX를 이해하지 못하므로 Babel 또는 TypeScript와 같은 컴파일러를 사용하여 JSX 코드를 일반 JavaScript로 변환합니다.

JSX는 XML과 같이 작성된 부분을 React.createElement를 쓰는 JavaScript 코드로 변환합니다. 중괄호(`{}`)를 사용하여 JavaScript 코드를 그대로 쓸 수 있으며, JavaScript 코드와 1:1로 매칭됩니다.

JSX 코드

```JSX
<div>
  <p>Count: {count}!</p>
  <button type="button" onClick={() => setCount(count + 1)}>Increase</button>
</div>
```

변환된 JS 코드

```js
React.createElement("div", null, /
  React.createElement("p", null, "Count: ", count, "!"), 
  React.createElement("button", {
  type: "button",
  onClick: () => setCount(count + 1)
}, "Increase")
);
```

새로운 JSX 변환 컴파일 코드(`React 17버전 이후`)

```js
import { jsxs as _jsxs } from "react/jsx-runtime";
import { jsx as _jsx } from "react/jsx-runtime";
_jsxs("div", {
  children: [
    _jsxs("p", {
      children: ["Count: ", count, "!"]
    }),
    _jsx("button", {
      type: "button",
      onClick: () => setCount(count + 1),
      children: "Increase"
    })]
});
```

새로운 JSX 변환 컴파일 코드가 기존과 달라진 이유는 더 이상 `React를 가져올 필요가 없다는 점` 입니다.
