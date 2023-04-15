# Template Literal Types - 템플릿 리터럴 문자열을 통해 속성을 변경하는 매핑된 타입

탬플릿 리터럴 유형(`Template Literal Types`)은 리터럴 유형(`Literal Types`)을 기반으로 하여 유니온(`Union`)을 통해 여러 문자열로 확장할 수 있습니다.

> JavaScript 템플릿 리터럴 문자열과 구문은 **동일**하지만 `Type 위치에서 사용`됩니다.

```tsx
type Fruit = 'apple' | 'orange';
type Status = 'fresh' | 'rotted';

type AllFruitStatus = `${Fruit}-${Status}`;
// type: AllLocaleIDs = 'apple-fresh' | 'apple-rotted' | 'orange-fresh' | 'orange-rotted'
```

만약 `AllLocaleIDs`를 `camel case`로 쓰고 싶다면,
유틸리티 type인 `Capitalize`를 사용하면 된다.

```tsx
type AllFruitStatus = `${Fruit}${Capitalize<Status>}`;
// type: AllLocaleIDs = 'appleFresh' | 'appleRotted' | 'orangeFresh' | 'orangeRotted'
```

## Template Literal Types 활용하기

### 반복되는 타입 정의 없애기

#### **여기서부터는 토스 기술 블로그에서 가져온 내용들입니다**

```js
element.addEventListener('click', () => alert('clicked!'));
element.onClick = () => alert('clicked!');
```

이벤트에 대한 핸드러를 등록할 때, `addEventListener('event', handler)`와 `onEvent=handler`의 두 가지 형식을 모두 사용할 수 있는 `MyElement` 타입을 생각해 봅시다. 아래 두 가지 방법을 모두 사용할 수 있다.

아래의 문제 점이 있다고 생각해보자!

```tsx
// 이벤트 이름이 하나 추가될 때마다...
type EventNames = 'click' | 'doubleClick' | 'mouseDown' | 'mouseUp';

type MyElement = {
  addEventListener(eventName: EventNames, handler: (e: Event) => void): void;

  // onEvent() 도 하나씩 추가해 줘야한다.
  onClick(e: Event): void;
  onDoubleClick(e: Event): void;
  onMouseDown(e: Event): void;
  onMouseUp(e: Event): void;
}
```

> 이벤트 이름(`EventNames`)이 하나가 추가될 때마다 onEvent()도 하나씩 추가해 줘야하는 불편함이 있다.

이러한 경우에 탬플릿 리터럴 유형(`Template Literal Types`)을 사용하면 한 곳만 수정해도 모두 반영되게 수정할 수 있습니다.

```tsx
type EventNames = "Click" | "doubleClick" | "mouseDown" | "mouseUp";
type CapitalizedEventNames = Capitalize<EventNames>;
type HandlerNames = `on${CapitalizedEventNames}`;

type Handlers = {
  [H in HandlerNames]: (event: Event) => void;
};

type MyElement = Handlers & {
  addEventListener: (eventName: EventNames, handler: (event: Event) => void) => void;
};
```

## 내장 문자열 조작 유형

문자열 조작을 돕기 위해 문자열 조작에 사용할 수 있는 type 집합들이 있습니다.

### `Uppercase<StringType>` - 대문자 변환

```tsx
type Greeting = 'Hello, world'
type ShoutyGreeting = Uppercase<Greeting> // type: 'HELLO, WORLD'
```

### `Lowercase<StringType>` - 소문자 변환

```tsx
type Greeting = 'Hello, world'
type ShoutyGreeting = Lowercase<Greeting> // type: 'hello, world'
```

### `Capitalize<StringType>` - 첫 번째 문자 대문자로 변환

```tsx
type Greeting = 'Hello, world'
type ShoutyGreeting = Capitalize<Greeting> // type: 'Hello, world'
```

### `Uncapitalize<StringType>` - 첫 번째 문자 소문자로 변환

```tsx
type Greeting = 'Hello, world'
type ShoutyGreeting = Uncapitalize<Greeting> // type: 'hello, world'
```
