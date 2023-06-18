# Axios Interceptors

`request`나 `response`에서 then이나 catch 되기 전에 먼저 가로채서 데이터를 처리할 수 있다.

## Axios Interceptors로 token 체크 후 request 보내기

```ts
/*
  request 인터셉터 추가하기
  실제 사용했던 react-native 코드
*/
api.interceptors.request.use(async (config) => {
  // 요청이 전달되기 전에 작업 수행
  const user = await AsyncStorage.getItem('user');

  if (user) {
    const userData = JSON.parse(user) as User;
    // eslint-disable-next-line no-param-reassign
    config.headers.Authorization = `Bearer ${userData.accessToken}` ?? '';
  }

  return config;
});
```

## Axios Interceptors data response 받기

axios에서는 기본적으로 response의 데이터를 data로 넘겨주는데 이를 매번 체크하기 보다는 응답을 받을 때 response.data를 받는 형식으로 쓰는 방법이있다.

```ts
// 응답 인터셉터 추가하기
axios.interceptors.response.use(
  // 응답이 전달되기 전 실행
  (response) => response.data, // response.data return
  (error) => Promise.reject(error)
);
```
