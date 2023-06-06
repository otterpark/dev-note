# Redux Toolkit

Redux의 기존 복잡성을 줄여주기 위해 나온 라이브러리이며, Redux 사용을 위해 작성한 ducks(패턴 요소)가 전체적인 코드량을 늘린다는 불만이 발생하였고 리덕스팀이 이를 수용하여 코드는 적게, 하지만 더 사용하기 편하게 만든 것이 Redux-Toolkit이다.

> 기존 리덕스와 구조, 패러다임 모두 동일

## 여기서 잠깐! ducks 패턴이란?

<https://github.com/erikras/ducks-modular-redux>

erikras 라는 개발자가 만든 패턴인데 쉽게 말해 `액션 타입`, `액션 생성 함수`, `리듀서`를 각각 별도의 파일(심지어는 별도의 폴더)에 분리하여 작성하기 보다는, 그 셋을 하나의 모듈처럼 한 파일 안에 작성하자는 제안하는 패턴이다.

기존의 아래의 방식을 지양하고자 하는 패턴이다.

```
|_ store
|_ constants
|_ reducers
|_ actions
```

규칙은 아래와 같다.

1. 반드시 리듀서 함수를 default export해야 한다.
2. 반드시 액션 생성 함수를 export해야 한다.
3. 반드시 접두사를 붙인 형태로 액션 타입을 정의해야 한다. (아래 예제 코드 참고)
4. 외부 리듀서가 모듈 내 액션 타입을 바라보고 있거나, 모듈이 재사용 가능한 라이브러리로 쓰이는 것이라면 액션 타입을 UPPER_SNAKE_CASE 형태로 이름 짓고 export 하면 된다. (필수 X)

### 1. counter 모듈(src/modules/counter.js)

```js
/* ----------------- 액션 타입 ------------------ */
const INCREASE = 'counter/INCREASE';
const DECREASE = 'counter/DECREASE';

/* ----------------- 액션 생성 함수 ------------------ */
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });

/* ----------------- 모듈의 초기 상태 ------------------ */
const initialState = {
  number: 0,
};

/* ----------------- 리듀서 ------------------ */
export default function counter(state = initialState, action) {
  switch (action.type) {
    case INCREASE:
      return {
        ...state,
        number: state.number + 1,
      };
    case DECREASE:
      return {
        ...state,
        number: state.number - 1,
      };
    default:
      return state;
  }
}
```

### 2. root reducer

```js
import { combineReducers } from 'redux';
import counter from './counter';

const rootReducer = combineReducers({
  counter,
  // 다른 모듈을 하나씩 추가
});

export default rootReducer;
```

## 특징

### 1. 불필요한 코드 감소

Redux의 Action 함수 및 생성 함수, reducer, store 등을 하나의 기능마다 일일히 작성해야하는 불필요한 코드들을 줄여준다.

### 2. 과도한 패키지 양 해결

Redux의 복잡성을 해결하기 위해 여러 패키지들을 사용하는데 각기 다른 패키지들의 이해들을 파악하는데에 진입장벽이 높아졌던 부분들이 해결

### 3. 과도한 보일러플레이트 양 감소

기존 작성한 코드에서 새로운 기능(Store를 활용하여)을 만든다고 하였을 때 세팅 자체가 어렵고 반복되는 작업들을 copy 후 paste 하는 것들이 많았던 부분을 해결

## 비교

### 기존 redux 방식

`counter.js`

```js
/* ----------------- 액션 타입 ------------------ */
const INCREASE = 'counter/INCREASE';
const DECREASE = 'counter/DECREASE';

/* ----------------- 액션 생성 함수 ------------------ */
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });

/* ----------------- 모듈의 초기 상태 ------------------ */
const initialState = {
  number: 0,
};

/* ----------------- 리듀서 ------------------ */
export default function counter(state = initialState, action) {
  switch (action.type) {
    case INCREASE:
      return {
        ...state,
        number: state.number + action.payload,
      };
    case DECREASE:
      return {
        ...state,
        number: state.number - action.payload,
      };
    default:
      return state;
  }
}
```

`rootReducer.js`

```js
/* ----------------- root configure ------------------ */
import { combineReducers } from 'redux';
import counter from './counter';

const rootReducer = combineReducers({
  counter,
  // 다른 모듈을 하나씩 추가
});

export default rootReducer;
```

### redux-toolkit 방식

`/store/counterSlice.js`

```js
/* ----------------- 모듈의 초기 상태 ------------------ */
const initialState = {
  number: 0,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increase: (state, action) => {
      state.number = state.number + action.payload;
    },

    decrease: (state, action) => {
      state.number = state.number - action.payload;
    },
  },
});

/* ----------------- 액션 생성 함수 ------------------ */
export const { increase, decrease } = counterSlice.actions;

/* ----------------- configStore 등록을 위한 export ------------------ */
export default counterSlice.reducer;

```

`rootStore.js`

```js
/* ----------------- root configure ------------------ */
import { configureStore } from "@reduxjs/toolkit";

import counter from "../store/counterSlice";

const rootStore = configureStore({
  reducer: {
    counter: counter,
  },
});

export default rootStore;
```

- 불변성을 지키는 immutable 관련 미들웨어가 자동 추가되어 있음
- createSlice 내부에 Immer 라는 라이브러리를 사용하여, 사용자의 변경 사항을 추적한 뒤 해당 변경 사항을 추적 후 변경 불가능한 업데이트를 가능하게 해줌

## createAsyncThunk()

보통 API 통신에 사용하며, payload creator는 무조건 promise를 반환해야하기 때문에 Promise 또는 async/await를 구문을 사용하여 작성

createAsyncThunk 생성시 액션 크리에이터, 액션 유형, 액션을 자동 디스패치하는 thunk 함수를 생성한다. 액션 크리에이터와 액션 유형은 아래와 같다.

- `pending` - paload creator를 실행 전 보류 중인 작업을 디스패치
- `fulfilled` - promise 성공 시 처리
- `rejected` - promise 실패 시 처리

```js
// counterSlice.js
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import axios from 'axios';

export const fetchCounter = createAsyncThunk('counter/fetchUserCounter', async () => {
  return axios
    .get('url')
    .then((res) => res.data)
    .catch((error) => error);
});

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    numbers: 0,
    loading: false,
    error: ''
  },
  reducers: {},
  extraReducers: {
    [fetchUser.pending]: (state) => {
      state.numbers = 0;
      state.loading = true;
      state.error = '';
    },
    [fetchUser.fulfilled]: (state, action) => {
      state.numbers = action.payload;
      state.loading = false;
      state.error = '';
    },
    [fetchUser.rejected]: (state, action) => {
      state.numbers = 0;
      state.loading = false;
      state.error = action.payload;
    }
  }
});

export default counterSlice.reducer;
```

`component에서 사용하기`

```js
import { useSelector, useDispatch } from 'react-redux';
import { fetchUserCounter } from './counterSlice';

export default function App() {
  const dispatch = useDispatch();
  const { counter, loading, error } = useSelector((state) => state.counter);

  if (error) {
    return <p>무언가 문제가 생겼으니 다시 시도해 주세요.</p>;
  }

  if (loading) {
    return <p>로딩중...</p>;
  }

  return (
    <div>
      <h1>받아온 User Count </h1>
      <button onClick={() => dispatch(fetchUserCounter())}>유저 카운터 얻기</button>
      <p>{counter}</p>
    </div>
  );
}
```

## createEntityAdapter()

createEntityAdapter 호출 시 미리 만들어진 reducer 함수가 포함된 Adapter를 반환한다.

- `addOne / addMany` - state에 새 항목 추가
- `upsertOne / upsertMany` - 새 항목을 추가하거나 기존 항목을 업데이트
- `updateOne / updateMany` - 부분 값을 제공하여 기존 항목 업데이트
- `removeOne / removeMany` - ID를 기준으로 항목 제거
- `setAll` - 모든 기존 항목 교체

또한 아래 부분들을 포함하고 있습니다.

- `getInitialState` - { ids: [], entities: {} }와 같은 객체를 반환하며, 모든 항목 ID의 배열과 함께 항목의 정규화된 상태를 저장
- `getSelectors` - 표준 선택자 함수 집합을 생성

```js
import {
  createEntityAdapter,
  createSlice,
  configureStore,
} from '@reduxjs/toolkit'

const booksAdapter = createEntityAdapter({
  // Assume IDs are stored in a field other than `book.id`
  selectId: (book) => book.bookId,
  // Keep the "all IDs" array sorted based on book titles
  sortComparer: (a, b) => a.title.localeCompare(b.title),
})

const booksSlice = createSlice({
  name: 'books',
  initialState: booksAdapter.getInitialState(),
  reducers: {
    // Can pass adapter functions directly as case reducers.  Because we're passing this
    // as a value, `createSlice` will auto-generate the `bookAdded` action type / creator
    bookAdded: booksAdapter.addOne,
    booksReceived(state, action) {
      // Or, call them as "mutating" helpers in a case reducer
      booksAdapter.setAll(state, action.payload.books)
    },
  },
})

const store = configureStore({
  reducer: {
    books: booksSlice.reducer,
  },
})

console.log(store.getState().books)
// { ids: [], entities: {} }

// Can create a set of memoized selectors based on the location of this entity state
const booksSelectors = booksAdapter.getSelectors((state) => state.books)

// And then use the selectors to retrieve values
const allBooks = booksSelectors.selectAll(store.getState())
```