# Jest 테스트 글로벌 세팅

## clearMocks

모든 테스트 전에 모의 함수 호출, 인스턴스, 컨텍스트 및 결과를 자동으로 지우며, 실행되었던 모의 함수 구현은 제거되지 않습니다.

> 각 테스트 전에 `jest.clearAllMocks()`를 호출하는 것과 동일합니다.

```tsx
jest.mock('.');

const { randomNumber } = require('.');

describe('tests', () => {
  randomNumber.mockReturnValue(42);

  it('should return 42', () => {
    const random = randomNumber();

    expect(random).toBe(42);
    expect(randomNumber).toBeCalledTimes(1);
  });

  it('should return same number', () => {
    const random1 = randomNumber();
    const random2 = randomNumber();

    expect(random1).toBe(42);
    expect(random2).toBe(42);

    expect(randomNumber).toBeCalledTimes(2);
  });
});
```

만약 위와 같이 테스트를 진행한다고 하였을 때 `expect(randomNumber).toBeCalledTimes(2)` 부분은 아래와 같이 에러를 띄웁니다.

```tsx
Error: expect(jest.fn()).toBeCalledTimes(expected)

Expected number of calls: 2
Received number of calls: 3
```

Jest설정에서 `clearMocks를 활성화하면 테스트 사이에 모든 모의 컨텍스트가 리셋된다.

> `jest.clearAllMocks()`를 `beforeEach()` 함수에 추가하는 것과 동일하나 일일이 테스트마다 추가해줘야 하는 번거로움이있다.

## resetMocks

모든 테스트 전에 모의 상태를 자동으로 재 설정합니다. 모의 구현이 제거가 되지만 처음의 구현 상태로 복원되지 않습니다.

> 각 테스트 전에 jest.resetAllMocks()를 호출하는 것과 동일하다.

이전에 알아봤던 `clearMocks`가 `resetMocks`의 모의 구현을 지움으로써 한단계 더 발전시킵니다.

그럼 다시 예제로 돌아가볼까요?

```tsx
describe('tests', () => {
  it('should return 42', () => {
    randomNumber.mockReturnValue(42);
    const random = randomNumber();

    expect(random).toBe(42);
    expect(randomNumber).toBeCalledTimes(1);
  });

  it('should return 42 twice', () => {
    const random1 = randomNumber();
    const random2 = randomNumber();

    expect(random1).toBe(42);
    expect(random2).toBe(42);

    expect(randomNumber).toBeCalledTimes(2);
  });
});
```

논리적으로는 실패할 것으로 예상하지만 코드가 통과하게 됩니다. Jest 모의 객체는 해당 파일 내에서 전역적으로 작동한다.

> `describe`, `it`, `test` 스코프를 사용하든 상관없다.

Jest 컨텍스트에서 `resetMocks`를 활성화하면 테스트 사이에 모든 모의 구현이 초기화된다.

> `beforeEach()`에 `jest.resetAllMocks()`를 추가하는 것과 동일합니다.

## restoreMocks

모든 테스트 전에 모의 상태와 구현을 자동으로 복원합니다. 모든 모의 구현을 제거하며 초기 구현을 복원합니다.

> 각 테스트 전에 `jest.restoreAllmocks()`를 호출하는 것과 동일합니다.

`restoreMocks`는 테스트 격리 및 안정성을 한 차원 더 높여준다.

이번엔 함수를 직접 모킹하지 않고 `Math.random()`을 모킹해보자.

```tsx
const { randomNumber } = require('.');

const spy = jest.spyOn(Math, 'random');

describe('tests', () => {
  it('should return 42', () => {
    spy.mockReturnValue(42);
    const random = randomNumber();

    expect(random).toBe(42);
    expect(spy).toBeCalledTimes(1);
  });

  it('should return 42 twice', () => {
    spy.mockReturnValue(42);

    const random1 = randomNumber();
    const random2 = randomNumber();

    expect(random1).toBe(42);
    expect(random2).toBe(42);

    expect(spy).toBeCalledTimes(2);
  });
});
```

`clearMocks` 및 `resetMocks`를 활성화하고 `restoreMocks`를 비활성화하면 위 테스트가 통과합니다. 하지만 `restoreMocks`를 활성화하면 아래와 같은 오류 메세지와 함께 모든 테스트가 실패한다.

```tsx
Error: expect(received).toBe(expected) // Object.is equality

Expected: 42
Received: 0.503533695686772
```

`restoreMocks`는 각 테스트 전에 `Math.random()`의 원래 구현을 복원했기 때문에 42가 아닌 임의의 숫자를 얻습니다. 이를 수정하기 위해서는 아래와 같이 작성해야합니다.

```tsx
describe('tests', () => {
  it('should return 42', () => {
    const spy = jest.spyOn(Math, 'random').mockReturnValue(42);
    const random = randomNumber();

    expect(random).toBe(42);
    expect(spy).toBeCalledTimes(1);
  });

  it('should return 42 twice', () => {
    const spy = jest.spyOn(Math, 'random').mockReturnValue(42);

    const random1 = randomNumber();
    const random2 = randomNumber();

    expect(random1).toBe(42);
    expect(random2).toBe(42);

    expect(spy).toBeCalledTimes(2);
  });
});
```

## resetModules

기본적으로 각 테스트 파일은 독립적인 모듈 레지스트리를 갖습니다. 이 기능은 로컬 모듈 상태가 테스트 간에 충돌하지 않도록 모든 테스트에 대해 모듈을 분리하는 데 유용ㅎ다.

> `resetModules`를 활성화하면 한 단계 더 나아가 각 개별 테스트를 실행하기 전에 모듈 레지스트리를 재설정합니다. 

이 모든 것은 `clearMocks`, `resetMocks` 그리고 `restoreMocks`를 기반으로 구축된다.

`resetModules`를 활성화하면 파일의 각 테스트에 대해 각 테스트에 대한 모듈의 새 버전이 제공된다.

> `beforeEach()`에서 `jest.resetAllModules()`를 추가하는 것이 더 나은 경우가 있다.

## 마지막으로

기본적으로 Jest 테스트는 파일 수준에서만 격리됩니다. 테스트를 안전하게 처리하기 위해서 위에서 알아보았던 설정을 추가해보자!

```json
{
  clearMocks: true,
  resetMocks: true,
  restoreMocks: true,
  resetModules: true // 상황에 맞춰 설정
}
```
