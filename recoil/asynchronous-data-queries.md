# Asynchronous Data Queries

Recoil에서 `data-flow graph`를 이용하여 함수가 비동기적으로 작동하게 할 수 있습니다. 방법은 아주 간단하다. `Selector`에 `get` 콜백에서 값 자체 대산 값에 대한 `Promise`를 반환하기만 하면 됩니다. 이는 결국 `Selector` 이기 때문에 다른 `Selector`에서 비동기 `Selector`에 의존하여 데이터를 추가로 변환할 수 있다.

`Selector`는 항상 동일한 결과를 생성해야한다. `Selector` 결과가 캐시되거나, 다시 시작되거나, 여러 번 실행될 수 있기 때문에 중요합니다. 일반적으로 `Selector`는 읽기 전용 DB 쿼리를 모델링하는 데 좋은 방법입니다. 변경 가능한 데이터의 경우 쿼리 새로 고침을 사용할 수 있습니다. 또는 변경 가능한 상태, 지속 상태 또는 기타 부작용을 동기화하려면 `Atom Effects API` 또는 `Recoil 동기화` 라이브러리를 찾아봐야한다.

## Synchronous Example

사용자 이름을 가져오는 동기식 `Atom` 및 `Selector`

```tsx
const currentUserIDState = atom({
  key: 'CurrentUserID',
  default: 1,
});

const currentUserNameState = selector({
  key: 'CurrentUserName',
  get: ({get}) => {
    return tableOfUsers[get(currentUserIDState)].name;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameState);
  return <div>{userName}</div>;
}

function MyApp() {
  return (
    <RecoilRoot>
      <CurrentUserInfo />
    </RecoilRoot>
  );
}
```

## Asynchronous Example

쿼리해야 하는 사용자 이름이 데이터베이스에 저장되어 이는 경우 `Promise`를 반환하거나 `Async` 함수를 사용하면 된다. 종속성이 변경되면 선택기가 다시 재 평가되고 새 쿼리가 실행한다. 결과는 캐시되므로 쿼리는 고유 입력당 **한 번** 실행된다.

```tsx
const currentUserNameQuery = selector({
  key: 'CurrentUserName',
  get: async ({get}) => {
    const response = await myDBQuery({
      userID: get(currentUserIDState),
    });
    return response.name;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameQuery);
  return <div>{userName}</div>;
}
```

`Selector`의 인터페이스는 동일하므로 이 `Selector`를 사용하는 컴포넌트는 동기식 `Atom` 상태, 파생된 `Selector` 상태 또는 비동기식 쿼리로 뒷받침되었는지 신경 쓸 필요가 없다.

하지만 React `render` 함수는 동기식이기 때문에, `Promise`가 해결되기 전에 React 18버전부터 추가된 보류 중인 데이터를 처리하기 위해 `React Suspense`와 함께 작동하도록 설계되었습니다. 컴포넌트를 `Suspense 바운더리`로 감싸면 아직 보류 중인 모든 하위 컴포넌트를 잡아내고 `fallback UI`를 렌더링합니다.

```tsx
function MyApp() {
  return (
    <RecoilRoot>
      <React.Suspense fallback={<div>Loading...</div>}>
        <CurrentUserInfo />
      </React.Suspense>
    </RecoilRoot>
  );
}
```

## Async Queries Without React Suspense

보류 중인 비동기 `Selector`를 처리하기 위해 `React Suspense`를 사용할 필요는 없습니다. 렌더링 중 현재 상태를 확인하기 위해 `useRecoilValueLoadable()` 훅을 사용할 수도 있습니다.

```tsx
function UserInfo({userID}) {
  const userNameLoadable = useRecoilValueLoadable(userNameQuery(userID));
  switch (userNameLoadable.state) {
    case 'hasValue':
      return <div>{userNameLoadable.contents}</div>;
    case 'loading':
      return <div>Loading...</div>;
    case 'hasError':
      throw userNameLoadable.contents;
  }
}
```

## Queries with Parameters

파생된 상태뿐만 아니라 매개변수를 기반으로 쿼리하고 싶을 때가 있습니다. 예를 들어 컴포넌트 `Props`를 기반으로 쿼리하고 싶을 수 있습니다. `selectorFamily()` 헬퍼를 사용하면 됩니다.

```tsx
const userNameQuery = selectorFamily({
  key: 'UserName',
  get: userID => async () => {
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response.name;
  },
});

function UserInfo({userID}) {
  const userName = useRecoilValue(userNameQuery(userID));
  return <div>{userName}</div>;
}

function MyApp() {
  return (
    <RecoilRoot>
      <ErrorBoundary>
        <React.Suspense fallback={<div>Loading...</div>}>
          <UserInfo userID={1}/>
          <UserInfo userID={2}/>
          <UserInfo userID={3}/>
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

## Data-Flow Graph

쿼리를 `Selector`로 모델링하면 상태, 파생된 상태, 쿼리가 혼합된 데이터 흐름 그래프를 만들 수 있습니다. 이 그래프는 상태가 업데이트되면 자동으로 업데이트되고 React 컴포넌트를 리렌더링합니다.

다음은 현재 사용자의 이름과 친구 목록을 렌더링하는 예시입니다. 친구의 이름을 클릭하면 현재 사용자가 되고 이름과 목록이 자동으로 업데이트됩니다.

```tsx
const currentUserIDState = atom({
  key: 'CurrentUserID',
  default: null,
});

const userInfoQuery = selectorFamily({
  key: 'UserInfoQuery',
  get: userID => async () => {
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response;
  },
});

const currentUserInfoQuery = selector({
  key: 'CurrentUserInfoQuery',
  get: ({get}) => get(userInfoQuery(get(currentUserIDState))),
});

const friendsInfoQuery = selector({
  key: 'FriendsInfoQuery',
  get: ({get}) => {
    const {friendList} = get(currentUserInfoQuery);
    return friendList.map(friendID => get(userInfoQuery(friendID)));
  },
});

function CurrentUserInfo() {
  const currentUser = useRecoilValue(currentUserInfoQuery);
  const friends = useRecoilValue(friendsInfoQuery);
  const setCurrentUserID = useSetRecoilState(currentUserIDState);
  return (
    <div>
      <h1>{currentUser.name}</h1>
      <ul>
        {friends.map(friend =>
          <li key={friend.id} onClick={() => setCurrentUserID(friend.id)}>
            {friend.name}
          </li>
        )}
      </ul>
    </div>
  );
}

function MyApp() {
  return (
    <RecoilRoot>
      <ErrorBoundary>
        <React.Suspense fallback={<div>Loading...</div>}>
          <CurrentUserInfo />
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

## Concurrent Requests(동시 요청)

위의 예에서 알 수 있듯이 `friendInfoQuery`는 쿼리를 사용하여 각 친구에 대한 정보를 가져옵니다. 하지만 이 작업을 루프에서 수행하면 기본적으로 직렬화됩니다. 조회가 빠르다면 괜찮을 수도 있습니다. 만약 오래 걸린다고 했을 때 `waitForAll`과 같은 동시성 헬퍼를 사용하여 병렬로 실행할 수 있습니다.

```tsx
const friendsInfoQuery = selector({
  key: 'FriendsInfoQuery',
  get: ({get}) => {
    const {friendList} = get(currentUserInfoQuery);
    const friends = get(waitForAll(
      friendList.map(friendID => userInfoQuery(friendID))
    ));
    return friends;
  },
});
```

`waitForNone`을 사용하여 부분 데이터로 UI의 점진적 업데이트를 처리할 수 있습니다.

```tsx
const friendsInfoQuery = selector({
  key: 'FriendsInfoQuery',
  get: ({get}) => {
    const {friendList} = get(currentUserInfoQuery);
    const friendLoadables = get(waitForNone(
      friendList.map(friendID => userInfoQuery(friendID))
    ));
    return friendLoadables
      .filter(({state}) => state === 'hasValue')
      .map(({contents}) => contents);
  },
});
```

## Pre-Fetching

성능상의 이유로 렌더링 전에 가져오기를 시작하는 것이 좋습니다. 이렇게 하면 렌더링을 시작하는 동안 쿼리를 진행할 수 있습니다. React 문서에 몇 가지 예가 나와 있으며, 리코일에서도 동일하게 작동합니다.

위의 예시를 사용자가 사용자 변경 버튼을 클릭하자마자 다음 사용자 정보에 대한 가져오기를 시작하도록 변경해 보겠습니다.

```tsx
function CurrentUserInfo() {
  const currentUser = useRecoilValue(currentUserInfoQuery);
  const friends = useRecoilValue(friendsInfoQuery);

  const changeUser = useRecoilCallback(({snapshot, set}) => userID => {
    snapshot.getLoadable(userInfoQuery(userID)); // pre-fetch
    set(currentUserIDState, userID); // change current user to start new render
  });

  return (
    <div>
      <h1>{currentUser.name}</h1>
      <ul>
        {friends.map(friend =>
          <li key={friend.id} onClick={() => changeUser(friend.id)}>
            {friend.name}
          </li>
        )}
      </ul>
    </div>
  );
```

이 `Pre-Fetching`은 `selectorFamily()`를 트리거하여 비동기 쿼리를 시작하고 `Selector`의 캐시를 채우는 방식으로 작동한다는 점에 유의하세요. `Atom`을 설정하거나 `Atom` 효과에 의존하여 초기화하기 위해 `atomFamily()`를 대신 사용하는 경우, 제공된 스냅샷의 상태를 설정하려고 하면 호스트 `<RecoilRoot>`의 라이브 상태에 아무런 영향을 미치지 않으므로 `useRecoilCallback()` 대신 `useRecoilTransaction_UNSTABLE()`을 사용해야 합니다.

## Query Default Atom Values

`Atom`을 사용하여 로컬 편집 가능한 상태를 표현하되, 기본값을 쿼리하는 데는 프로미스를 사용하는 것이다.

```tsx
const currentUserIDState = atom({
  key: 'CurrentUserID',
  default: myFetchCurrentUserID(),
});
```

`Selector`를 사용하여 쿼리를 지연시키거나 다른 상태에 따라 달라집니다. `Selector`를 사용할 때 기본 `Atom` 값은 사용자가 명시적으로 `Atom`을 설정할 때까지 동적으로 유지되며 `Selector` 업데이트와 함께 업데이트된다는 점에 유의해야한다.

```tsx
const UserInfoState = atom({
  key: 'UserInfo',
  default: selector({
    key: 'UserInfo/Default',
    get: ({get}) => myFetchUserInfo(get(currentUserIDState)),
  }),
});
```

그리고 당연히 Atom families로 표현할 수 있다.

```tsx
const userInfoState = atomFamily({
  key: 'UserInfo',
  default: id  => myFetchUserInfo(id),
});

const userInfoState = atomFamily({
  key: 'UserInfo',
  default: selectorFamily({
    key: 'UserInfo/Default',
    get: id => ({get}) => myFetchUserInfo(id, get(paramsState)),
  }),
});
```

## Query Refresh

`Selector`를 사용하여 데이터 쿼리를 모델링할 때 `Selector` 결과는 항상 주어진 상태에 대해 일관된 값을 제공해야 합니다. `Selector`는 다른 `Atom` 및 `Selector` 상태로부터 파생된 상태를 나타냅니다. 따라서 `Selector` 결과 함수는 캐시되거나 여러 번 실행될 수 있으므로 주어진 입력에 대해 `idempotent(여러번 실행되어도 결과는 동일해야한다)`이어야 합니다. 그러나 `Selector`가  데이터 쿼리에서 데이터를 가져오는 경우 최신 데이터로 새로 고치거나 실패 후 재시도하기 위해 다시 쿼리하는 것이 유용할 수 있습니다. 이를 위한 몇 가지 방법이 있습니다.

### useRecoilRefresher()

`useRecoilRefresher_UNSTABLE()` 훅을 사용하면 콜백을 가져와서 캐시를 지우고 강제로 재평가하도록 호출할 수 있습니다.

```tsx
const userInfoQuery = selectorFamily({
  key: 'UserInfoQuery',
  get: userID => async () => {
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response.data;
  }
})

function CurrentUserInfo() {
  const currentUserID = useRecoilValue(currentUserIDState);
  const currentUserInfo = useRecoilValue(userInfoQuery(currentUserID));
  const refreshUserInfo = useRecoilRefresher_UNSTABLE(userInfoQuery(currentUserID));

  return (
    <div>
      <h1>{currentUserInfo.name}</h1>
      <button onClick={() => refreshUserInfo()}>Refresh</button>
    </div>
  );
}
```

### Use a Request ID

`Selector` 평가는 입력(종속 상태 또는 패밀리 매개변수)을 기반으로 주어진 상태에 대해 일관된 값을 제공해야 합니다. 따라서 쿼리에 요청 ID를 패밀리 매개변수 또는 종속성으로 추가할 수 있습니다.

```tsx
const userInfoQueryRequestIDState = atomFamily({
  key: 'UserInfoQueryRequestID',
  default: 0,
});

const userInfoQuery = selectorFamily({
  key: 'UserInfoQuery',
  get: userID => async ({get}) => {
    get(userInfoQueryRequestIDState(userID)); // Add request ID as a dependency
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response.data;
  },
});

function useRefreshUserInfo(userID) {
  const setUserInfoQueryRequestID = useSetRecoilState(userInfoQueryRequestIDState(userID));
  return () => {
    setUserInfoQueryRequestID(requestID => requestID + 1);
  };
}

function CurrentUserInfo() {
  const currentUserID = useRecoilValue(currentUserIDState);
  const currentUserInfo = useRecoilValue(userInfoQuery(currentUserID));
  const refreshUserInfo = useRefreshUserInfo(currentUserID);

  return (
    <div>
      <h1>{currentUserInfo.name}</h1>
      <button onClick={refreshUserInfo}>Refresh</button>
    </div>
  );
}
```

## Use an Atom

`Selector` 대신 `Atom`을 사용하여 쿼리 결과를 모델링하는 것입니다. 새로 고침 정책에 따라 새로운 쿼리 결과로 `Atom` 상태를 필수적으로 업데이트할 수 있습니다.

```tsx
const userInfoState = atomFamily({
  key: 'UserInfo',
  default: userID => fetch(userInfoURL(userID)),
});

// React component to refresh query
function RefreshUserInfo({userID}) {
  const refreshUserInfo = useRecoilCallback(({set}) => async id => {
    const userInfo = await myDBQuery({userID});
    set(userInfoState(userID), userInfo);
  }, [userID]);

  // Refresh user info every second
  useEffect(() => {
    const intervalID = setInterval(refreshUserInfo, 1000);
    return () => clearInterval(intervalID);
  }, [refreshUserInfo]);

  return null;
}
```

`Atom`은 Promise를 새 값으로 받아들이는 것을 지원하지 않는다는 점에 유의하세요. 따라서 쿼리 새로고침이 보류 중인 동안에는 원하는 동작을 위해 현재 `React Suspense`를 위해 `Atom`을 보류 상태로 둘 수 없습니다. 하지만 현재 로딩 상태와 실제 결과를 수동으로 인코딩하는 객체를 저장하여 이를 명시적으로 처리할 수 있습니다.

## Retry query from error message

`<ErrorBoundary>`에서 던져지고 잡힌 오류를 기반으로 쿼리를 찾아 다시 시도하는 예제입니다.

```tsx
function QueryErrorMessage({error}) {
  const snapshot = useRecoilSnapshot();
  const selectors = useMemo(() => {
    const ret = [];
    for (const node of snapshot.getNodes_UNSTABLE({isInitialized: true})) {
      const {loadable, type} = snapshot.getInfo_UNSTABLE(node);
      if (loadable != null && loadable.state === 'hasError' && loadable.contents === error) {
        ret.push(node);
      }
    }
    return ret;
  }, [snapshot, error]);
  const retry = useRecoilCallback(({refresh}) =>
    () => selectors.forEach(refresh),
    [selectors],
  );

  return selectors.length > 0 && (
    <div>
      Error: {error.toString()}
      Query: {selectors[0].key}
      <button onClick={retry}>Retry</button>
    </div>
  );
}
```
