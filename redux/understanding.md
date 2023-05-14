# Redux Understanding

## 동기

프론트엔트 제품 개발에 있어서 새로 갖춰야할 요건들이 늘어나고 있습니다. 즉, 자바스크립트 싱글 페이지 애플리케이션이 갖추어야할 요건이 많아졌습니다. 이는 또한 상황들에 맞춰 많은 상태를 관리할 필요가 생겨났습니다.

하지만 항상 변하는 상태를 관리하기가 힘듦니다. 프로그래머조차 애플리케이션 안에 상태를 언제, 왜, 어떻게 업데이트할지 제어할 수 없는 지경에 이르고 맙니다. 낙관적 업데이트, 서버 렌더링, 라우트가 일어나기 전에 데이터를 가져온다든지 복잡한 상황들이 많이 나옵니다.  이런 복잡함은 `변화(mutation)`, `비동기(asynconicity)`의 개념을 섞어서 사용하며 `React`에서는 이 문제를 해결하기 위해 비동기와 DOM 조작들을 없애버렸습니다. 즉, **`React`는 데이터를 관리하는 일을 관여하지 않습니다.**

Redux는 아래 세 가지를 따라 상태 변화가 일어나는 시점에 제약을 두어 상태 변화를 예측 가능하게 만들고자 시도합니다.

- `Flux`
- `CQRS`
- `Event Sourcing`

이러한 제약은 또한 세가지 원칙에 반영됩니다.

## 3가지 원칙

Redux의 기초를 이루는 원칙들은 다음과 같습니다.

### 진실은 하나의 근원으로부터

**애플리케이션의 모든 상태는 하나의 저장소 안에 하나의 객체 트리 구조로 저장됩니다.**

위 개녕을 통해 범용적인 어플리케이션(universal application)을 만들기 쉽습니다. 서버로 가져온 상태는 `Serialized`되거나 `hydrated(수화)`되어 전달되며 클라이언트에서 추가적인 코딩 없이 사용할 수 있습니다.

```tsx
console.log(store.getState())

/* Prints
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
*/
```

### 상태는 읽기 전용이다

**`상태`를 변화시키는 유일한 방법은 무슨 일이 벌어지는 지를 묘사하는 `액션` 객체를 전달하는 방법 뿐이다**

위를 통해서 상태를 직접 바꾸지 못 한다는 것을 보장할 수 있습니다. 모든 상태 변화는 중앙에서 관리되며 모든 `액션`은 엄격한 순서에 맞춰 실행됩니다. `액션`은 기록을 남길 수 있고, `serializer` 할 수 있으며, 저장할 수 있고, 테스트나 디버깅을 재현하는 것도 가능한 단순한 `평범 객체`입니다.

```tsx
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

### 변화는 순수 함수로 작성되어야한다.

**`액션`에 의해 `상태` 트리가 어떻게 변화하는 지를 지정하기 위해 프로그래머는 `순수 리듀서`를 작성해야합니다**

리듀서는 이전 상태와 액션을 받아온 후 다음 상태를 반환하는 `순수 함수`이다. **이전 상태를 변경하는 대신 새로운 상태 객체를 생성해서 반환**해야한다.

```tsx
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

import { combineReducers, createStore } from 'redux'
const reducer = combineReducers({ visibilityFilter, todos })
const store = createStore(reducer)
```

## 용어집

### 상태(state)

```tsx
type State = any
```

`상태`(또는 `상태 트리`)는 넓은 의미의 단어지만 Redux API에서는 보통 저장소에 의해 관리되고 `getState()`에 의해 반환되는 하나의 상태값을 지칭합니다.

컨벤션에 따르면 최상위 상태는 객체나 Map과 같은 key-value 컬렉션이지만 기술적으로는 어떤 타입이든 가능합니다.

> 상태를 항상 직렬화 가능하게 하는 것이 좋다.(JSON으로 변환될 수 없으면 절대 넣지말아야한다.)

## 액션(Action)

```tsx
type Action = Object
```

`액션`은 상태를 변화시키려는 의도를 표현하는 평범한 객체입니다. 액션은 저장소에 데이터를 넣는 유일한 방법입니다.

액션은 어ㅓ떤 형태의 액션이 표시하는 `type` 필드를 가져야 합니다. 타입은 상수로 정의되고 다른 모듈에서도 마찬가지로 `import` 가능합니다.

> 문자열은 직렬화될 수 있기에 문자열을 권장합니다.

조금 어떤 식으로 쓸지 감을 잡기 위해 아래 링크를 참고하면 좋다.

[flux-standard-action](https://github.com/redux-utilities/flux-standard-action)

## 리듀서(reducer)

```tsx
type Reducer<S, A> = (state: S, action: A) => S
```

`리듀서`(또는 `리듀싱 함수`)는 **누적값과 값을 받아서 새로운 누적값을 반환**하는 함수입니다. 이들은 값들의 컬렉션을 받아서 하나의 값으로 줄이는데 사용됩니다.

`리듀서`는 기본 함수형 프로그래밍에서 왔으며, 자바스크립트(`Array.prototype.reduce()`)와 같은 함수가 아닌 프로그래밍 언어에도 리듀싱을 위한 내장 API가 있습니다.

`Redux`에서 누적값은 상태 객체이고, 누적될 값은 액션이며, **주어진 이전 상태와 액션에서 새로운 상태를 계산**합니다. 모든 출력은 `순수 함수`여야 합니다.

## 디스패치 함수(Dispatch)

```tsx
type BaseDispatch = (a: Action) => Action
type Dispatch = (a: Action | AsyncAction) => any
```

`디스패치 함수`는 `액션`이나 `비동기 액션`을 받는 함수입니다. 보통 `디스패치 함수`와 저장소 인스턴스가 `미들웨어`를 거치지 않고 제공하는 기본 `dispatch` 함수를 구분해야 합니다.

기본 `디스패치 함수`는 반드시 동기적으로 저장소의 리듀서에 `액션`을 보내야 합니다. 리듀서는 저장소가 반환하는 이전 상태와 새 상태를 계산합니다.

`미들웨어`는 기본 디스패치 함수를 감쌉니다. `미들워어`를 통해 `디스패치 함수`는 액션 뿐 아니라 `비동기 액션`을 처리할 수 있습니다. 미들웨어는 액션이나 비동기 액션을 다음 미들웨어에 넘기기 전에 변환하거나, 지연시키거나, 무시하거나, 해석하거나 할 수 있습니다.

## 액션 생산자(Action Creator)

```tsx
type ActionCreator<A, P extends any[] = any[]> = (...args: P) => Action | AsyncAction
```

`액션 생산자`는 단지 `액션`을 만드는 함수입니다. `액션`은 정보의 묶음이고, `액션 생산자`는 액션을 만드는 곳이다.

`액션 생산자`를 호출하면 `액션`만 만들어내고 `디스패치`하지 않습니다. 저장소를 변경하기 위해서는 `dispatch` 함수를 호출해야하며, `액션 생산자`를 호출해 그 결과 저장소 인스턴스로 바로 `디스패치`하는 함수를 `바인드된 액션 생산자`라고 부릅니다.

`액션 생산자`가 아래와 같은 상황이면 `비동기 액션`을 반환해야 합니다.

- 현재 상태를 읽어야하는 상황
- API 호출을 실행해야하는 상황
- 라우트 전환같은 부수효과를 일으켜야하는 상황

## 비동기 액션

```tsx
type AsyncAction = any
```

`비동기 액션`은 `디스패치 함수`로 보내는 값이지만, 리듀서에게 받아들여질 준비가 되어있지 않은 상태입니다. 비동기 액션은 기본 `dispatch()` 함수로 전달되기 전에 `미들웨어`를 통해 `액션`으로 바뀌어야 합니다. `비동기 액션`은 미들웨어에 따라 서로 다른 타입이 될 수 있습니다.

> Promise나 async와 같은 비동기 기본형으로, 작업이 완료되면 액션을 보냅니다.

## 미들웨어

```tsx
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```

`미들웨어`는 `디스패치 함수`를 결합해서 `새 디스패치 함수`를 반환하는 `고차함수`입니다. 이들은 종종 `비동기 액션`을 액션으로 전환합니다.

`미들웨어`는 함수 결합을 통해 서로 결합할 수 있습니다. 이는 `액션`을 로깅하거나, `라우팅`과 같은 부수효과를 일으키거나, `비동기 API 호출`을 일련의 `동기 액션`으로 바꾸는데 유용합니다.

## 저장소

```tsx
type Store = {
  dispatch: Dispatch,
  getState: () => State,
  subscribe: (listener: () => void) => () => void,
  replaceReducer: (reducer: Reducer) => void
}
```

`저장소`는 애플리케이션의 `상태 트리`를 가지고 있는 객체입니다. `리듀서` 수준에서 결합이 일어나기 때문에, `Redux` 앱에는 단 하나의 저장소만 있어야 합니다.

- `dispatch(action)` - 위에서 설명한 기본 디스패치 함수입니다.
- `getState()` - 저장소의 현재 상태를 반환합니다.
- `subscribe(listener)` - 상태가 바뀔 때 호출될 함수를 등록합니다.
- `replaceReducer(nextReducer)` - 핫 리로딩과 코드 분할을 구현할때 사용됩니다. (사용할 일 거의 없음)

## 저장소 생산자

```tsx
type StoreCreator = (reducer: Reducer, preloadedState: ?State) => Store
```

`저장소 생산자`는 Redux 저장소를 만드는 함수입니다. `디스패치 함수`와 마찬가지로, Redux 패키지에 들어있는 `기본 저장소 생산자`인 `createStore(reducer, preloadedState)`와 `저장소 인핸서`에서 반환되는 저장소 생산자는 구분해야 합니다.

## 저장소 인핸서

```tsx
type StoreEnhancer = (next: StoreCreator) => StoreCreator
```

`저장소 인핸서`는 저장소 생산자를 결합하여 강화된 새 저장소 생산자를 반환하는 고차함수입니다. 이는 미들웨어와 비슷하게 조합가능한 방식으로 저장소 인터페이스를 바꿀 수 있게 해줍니다.

> 저장소 인핸서는 React에서 "컴포넌트 인핸서"로 불리는 고차 컴포넌트와 같은 개념입니다.
