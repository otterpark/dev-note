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

### toBe()

테스트의 결과를 확인하는 API. 테스트의 예상 결과 값을 넣습니다.

```ts
const message = 'React';
test('message equals to be React', () => {
  expect(message).toBe('React');
});
```

### toEqual()

객체가 일치한지 검증한다. 왠만한 일치를 비교하려할때 이 메소드를 쓰면 된다.

### toBeDefined()

변수가 정의되었는지 여부를 테스트

### toBeTruthy() / toBeFalsy()

느슨한 타입 기반 언어인 `JavaScript`는, 자바같은 강한 타입 기반 언어처럼 `true`와 `false`가 boolean 타입에 한정되지 않는다.

따라서 숫자 1이 `true`로 간주되고, 숫자 0이 `false`로 간주되는 것과 같이,

모든 타입의 값들을 `true` 아니면 `false` 간주하는 규칙이 있는데,

`toBeTruthy()` 는 검증 대상이 이 규칙에 따라 true로 간주되면 테스트 통과이고,

`toBeFalsy()` 는 반대로 false로 간주되는 경우 테스트가 통과된다.

```ts
test('number 0 is falsy but string 0 is truthy', () => {
  expect(0).toBeFalsy();
  expect('0').toBeTruthy();
});
```

### toBeCalled() / toHaveBeenCalled()

함수가 호출되었는지 여부를 확인하며 위에 두 `Matcher`는 같은 역할을 하는 함수다.

### toHaveLength() / toContain()

배열의 경우에는 배열이 길이를 체크하거나 특정 원소가 존재 여부를 테스트하는 경우가 많다.

- `toHaveLength()` 배열의 길이를 체크할 때 쓰이고,
- `toContain()` 특정 원소가 배열에 들어있는지를 테스트할 때 쓰인다.

```ts
test('array', () => {
  const colors = ['Red', 'Yellow', 'Blue'];

  expect(colors).toHaveLength(3); // 배열 길이 3 체크
  expect(colors).toContain('Yellow'); // Yellow 문자열 원소를 가지고 있는지 체크
  expect(colors).not.toContain('Green'); // Green 문자열 원소를 가지고 있지 않은지 체크
});
```

### toMatch()

문자열의 경우에는 단순히 `toBe()`를 사용해서 문자열이 정확히 일치하는지를 체크하지만,
종종 정규식 기반의 테스트가 필요할 떄가 있는데 `toMatch()` 함수를 사용하면 된다.

```ts
test('string', () => {
  expect(getUser(1).email).toBe('user1@test.com'); // 단순 문자열 비교
  expect(getUser(2).email).toMatch(/.*test.com$/); // 정규식 비교
});
```

### toThrow()

예외 발생 여부를 테스트해야할 때는 `toThrow()` 함수를 사용한다.

`toThrow()` 함수는 인자도 받는데, 문자열을 넘기면 예외 메세지를 비교하고 정규식을 넘기면 정규식 체크를 해준다.

> 💡 `toThrow()` 함수를 사용할 때 반드시 `expect()` 함수에 넘기는 검증 대상을 함수로 한 번 감싸줘야 한다.

### toHaveProperty()

객체에 해당 `key : value` 값이 있는지 검사한다.

```ts
test('find user property', async () => {
   const user = {
      id : 1,
      name : 'Jinwoo Park'
   }

   expect(user).toHaveProperty('id', 1);
   expect(user).toHaveProperty('name', 'Jinwoo Park');
 });
```

### toBeCalledTimes() / toBeCalledWith()

- `toBeCalledTimes()` - 함수가 몇번 호출되었는지 검증
- `toBeCalledWith()` - 함수가 설정한 인자로 호출 되었는지 검증

```ts
drink('milk');
drink('water');

expect(drink).toHaveBeenCalledTimes(2);
expect(drink).toHaveBeenCalledWith('milk');
```

### toReturn() / toHaveReturned()

함수가 오류없이 **반환**되는지 테스트

### toReturnTimes() / toHaveReturnedTimes()

함수가 지정한 횟수만큼 오류없이 반환되는지 테스트하는데 호출 횟수는 포함하지 않는다.

```ts
test('drink returns twice', () => {
  const drink = jest.fn(() => true);

  drink();
  drink();

  expect(drink).toHaveReturnedTimes(2);
});
```

### toReturnWith() / toHaveReturnedWith(value)

함수가 **지정한 값을 반환**하는지 테스트

```ts
test('drink returns milk', () => {
  const bottle = {name: 'milk'};
  const drink = jest.fn(bottle => bottle.name);

  drink(bottle);

  expect(drink).toHaveReturnedWith('milk');
});
```
