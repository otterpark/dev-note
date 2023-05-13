# Jest Globals

이번에 테스트 코드 작성을 해보면서 기본적인 개념들에 대해서는 공부하고 들어갔지만 내용들을 한번 다 짚고 넘어가면 좋을 것 같아 내용을 한번 적어보고자 한다.

먼저 공식문서를 토대로 작성될 예정이며, 이번 기회를 통해 자주 쓰이는 함수들을 제외한 나머지 새로운 함수들을 배울 수 있어 좋았다.

그럼 시작해보자!

테스트 파일에서 Jest는 각 메서드와 객체 전역 환경을 배치합니다. Jest 각 메서드와 객체를 사용하고 싶다면 `import`를 통해 사용할 수 있다.

> `TypeScript`를 사용한다면 아래 세팅이 필요하다!
>
>[https://jestjs.io/docs/getting-started#using-typescript]

## beforeAll(fn, timeout)

`beforeAll`은 테스트 코드가 실행 전 함수를 **딱 한번** 실행합니다.

> 많은 테스트를 진행 시 사용할 전역 상태를 설정하는 경우에 용이합니다.

## afterAll(fn, timeout)

`afterAll`은 테스트 파일의 **테스트가 모두 완료된 이후** 함수를 **딱 한번**  실행합니다.

> 테스트 간에 공유되는 전역 설정 상태를 정리하려는 경우에 유용합니다.

## beforeEach(fn, timeout)

`beforeEach`는 **각 테스트가 실행 전** 함수를 실행합니다.

> 많은 테스트에서 사용되는 전역 설정 상태를 재설정하는 경우에 유용합니다.

## afterEach(fn, timeout)

`afterEach`는 **각 테스트가 완료된 이후** 함수를 실행합니다.

> 각 테스트에서 생성되는 상태를 정리하려는 경우에 유용합니다.

## beforeAll, afterAll, beforeEach, afterEach 정리

실제 코드에서 어떤 라이프사이클을 가지는지 딱 와닿게 예제로 알아보면 좋을 것 같아 정리해보았다.

```ts
beforeAll(() => console.log('outer - beforeAll'));
afterAll(() => console.log('outer - afterAll'));
beforeEach(() => console.log('outer - beforeEach'));
afterEach(() => console.log('outer - afterEach'));

test('Outer test', () => console.log('outer - test'));

describe('Scoped / Nested block', () => { // 1
  beforeAll(() => console.log('inner - beforeAll'));
  afterAll(() => console.log('inner - afterAll'));
  beforeEach(() => console.log('inner - beforeEach'));
  afterEach(() => console.log('inner - afterEach'));

  test('Inner test', () => console.log('inner - test'));
});

// outer - beforeAll
// outer - beforeEach
//    outer - test
// outer - afterEach
//    inner - beforeAll
// outer - beforeEach
//    inner - beforeEach
//    inner - test
//    inner - afterEach
// outer - afterEach
//    inner - afterAll
// outer - afterAll
```

예제를 정리해보면서 재미있었던 점은 `outer의 beforeEach`와 `inner의 beforeAll`, `inner의 beforeEach` 이다.

`beforeAll`은 위에서 설명했듯이 테스트 코드가 실행 전 딱 한번 실행하고, `beforeEach`는 각 테스트 함수가 실행될 때마다 실행됩니다.

`1번` 테스트가 실행될 때, `outer의 beforeEach`가 먼저 실행되고 `inner의 beforeAll`이 실행될 것 같았지만,`inner의 beforeAll`이 먼저 실행되는 것을 볼 수 있다.

정리해보면

> `outer의 beforeEach`의 우선순위가 `inner의 beforeAll` 보다 낮다.

## describe(name, fn)

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

### describe.each(table)(name, fn, timeout)

`describe.each`를 사용하면 `동일한 목적의 테스트`를 여러번 테스를 실행할 때 유용하게 쓸 수 있다.

```ts
describe.each([
  ["Cat", "Act"],
  ["Save", "Vase"],
  ["Elbow", "Below"],
])("areAnagrams(%s, %s)", (first, second) => {
  it("return true with ignoreCase option", () => {
    expect(areAnagrams(first, second, { ignoreCase: true })).toBe(true);
  });

  it("return false without ignoreCase option", () => {
    expect(areAnagrams(first, second)).toBe(false);
  });
});
```

### describe.only(name, fn)

`describe.only`는 설명 블록을 하나만 실행하려는 경우에만 사용할 수 있습니다.

```ts
describe.only('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});
```

> 마찬가지로 `describe.only.each`를 사용할 수 있다.

### describe.skip(name, fn)

특정 `describe 블록`의 테스트를 실행하지 않으려면 `describe.skip`을 사용할 수 있습니다.

보통은 테스트를 임시로 작성 후 이후에 처리하고자 할 때 유용하게 쓰일 수 있습니다.

```ts
describe.skip('my other beverage', () => {
  // 여기는 스킵된다.
});
```

> 마찬가지로 `describe.skip.each`를 사용할 수 있다.

## `test(name, fn, timeout)`

별칭으로 `it(name, fn, timeout)`로 사용할 수 있으며, 테스트 파일에는 테스트를 실행하는 테스트 메서드만 있어야 됩니다.

`test` 메소드 같은 경우에는 `describe`에서 사용했던 메소드들과 동일하게 작동하며, `test`에만 있는 특별한 함수만 다루어보려고 한다.

### test.failing(name, fn, timeout)

테스트를 작성하고 실패할 것으로 예상되는 경우 `test.failing`을 사용하면 된다. 

`test.failing`은 일반 테스트와 다른 방식으로 동작합니다. **테스트 실패 시 에러가 발생하면 통과**합니다. 반대로 에러가 발생하지 않으면 실패합니다.

```ts
test.failing('it is not equal', () => {
  expect(5).toBe(6); // 이 테스트는 통과된다.
});
```

> `failing.each, skip, skip.each, only, only.each`를 사용할 수 있다.

### test.todo(name)

테스트를 작성 예정이라면 `test.todo`를 사용하면 됩니다. `test.todo(name)`은  마지막에 요약 출력에 강조 표시되므로 테스트 작성해야할 수를 알 수 있습니다.

```ts
const add = (a, b) => a + b;

test.todo('add should be associative');
```
