# Jest

코드가 내가 의도한 대로 정상적으로 잘 동작하는지 확인하는 Test Case를 만드는 JavaScript 테스팅 라이브러리다.

> `Unit Test`를 위해 사용

## 1. describe ( name, fn )

여러 개의 `test()` 코드를 하나의 테스트 작업 단위로 묶어주는 API. 하나의 테스트 케이스를 `test()`라고 한다면 `describe()`는 여러 개의 테스트 케이스를 하나의 그룹으로 묶어주는 역할이다.

```ts
describe('Testing 1', () => {
  test('message equals to be React', () => {
    // ..
  });

  test('data equals to be Object', () => {
    // ..
  });
});
```

## 2. test(if) ( name, fn, timeout )

테스트 코드를 돌리기 위한 API. 하나의 테스트 케이스를 의미하며 `it()`과 같은 역할.

```ts
test('message equals to be Vue', () => {
  // ..
});
```

[describe, test 메소드는 여기 페이지에서 확인](/jest/globals.md)

## 3. expect()

테스트 할 대상을 넣는 API. `expect()`에는 주로 테스트 입력 값 또는 기대 값을 넣습니다.

```ts
const message = 'React';
test('message equals to be React', () => {
  expect(message).toBe('React');
});
```

## 4. matcher

Jest는 다른 방법으로 값을 테스트 하도록 `matcher`를 사용합니다.

[matcher 메소드는 여기 페이지에서 확인](/jest/expect.md)
