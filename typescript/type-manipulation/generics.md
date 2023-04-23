# Generics - 매개변수를 받는 타입

`Generics` 같은 경우에는 TypeScript 주제 중 `More on Functions` 와 `Object Types` 에 예제로 나오기도 했지만, 개념적으로 접근해보면 설명하면 한 가지 타입보다 여러 가지 타입에서 동작하는 재사용성이 높은 컴포넌트를 만들 때 자주 사용됩니다.

```tsx
function identity<T>(arg: T): T{
  return arg;
}

let output = identity<string>("myString"); // 첫 번째 방법 - type: string
/*
 원래는 위와 방법도 호출이 가능하지만, 앞서 배운 타입 추론을 통해 두 번째 방법 같이 더 가독성 있게 간단히 사용할 수 있습니다. 
 여기서의 핵심은 TypeScript는 컴파일러 값인 "myString"을 보고 타입 추론을 하여 Type을 정합니다. 
 (더 복잡한 예제에서는 컴파일러가 타입을 유추하기 어려울 때는 첫 번째 방법처럼 명시적으로 타입을 지정해 줘야한다)
*/

let output = identity("myString"); // 두 번째 방법 -  type: string
```

위와 같이 우리는 타입 변수인 `Type`을 통해 전달하는 인수에 따라서 컴파일러가 `Type` 값을 자동으로 정하게 해준다.

## 제네릭 타입 변수 작업

제네릭을 사용하면 컴파일러가 함수 본문에 제네릭 타입화된 매개변수를 사용하도록 강요합니다.

```tsx
function loggingIdentity<T>(arg: T): T{
  console.log(arg.length); // Error: Type에는 length가 없습니다.
  return arg;
}
```

위 예제를 보면 `제네릭 타입`을 사용 시 `any`나 `모든 타입`을 의미하는데, `.length` 가 없는 `number` 를 대신 전달할 수도 있습니다. 그러한 경우 `number`에는 length가 없기 때문에 Type 에러가 납니다.

그럼 다른 예제를 보자.

```tsx
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length); // Success 배열 프로토타입에는 length가 있기 때문에
  return arg;
}

function loggingIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length); // Success: 배열 프로토타입에는 length가 있기 때문에
  return arg;
}
```

위 함수는 일반 Type이 아닌 Type의 배열에서 동작했다고 가정한 예시이다. `loggingIdentity`의 타입을 정리해보면 아래와 같다.

> 제네릭 함수 `loggingIdentity`는 타입 매개변수 `Type`과 `Type` 배열인 인수 `arg`를 가지며, `Type` 배열을 반환한다.
>

## 제네릭 타입

함수 자체의 타입과 제네릭 인터페이스를 만드는 법에 대해 알아보자.

```tsx
function identity<Type>(arg: T): T {
  return arg;
}

let myIdentityFirst: <Input>(arg: Input) => Input = identity; // 첫 번째 방법
let myIdentitySecond: { <T>(arg: T): T } = identity; // 두 번째 방법
```

위 코드를 아래 인터페이스 코드로 작성할 수 있습니다.

```tsx
interface GenericInfinityFn {
 <T>(arg: T): T;
}

function identity<T>(text: T>: T {
 return arg;
}

let myIndentity: GenericInfinityF = identity;
```

만약 컴파일러가 타입을 유추하기 어렵거나 인자 타입을 강조하고 싶다면 다시 인자 타입을 강조할 수 있습니다.

```tsx
interface GenericInfinityFn<T> {
 <T>(arg: T): T;
}

function identity<T>(text: T>: T {
 return arg;
}

let myIndentity: GenericInfinityFnn<Input> = identity;
```

이와 같은 방식으로 제네릭 인터페이스 뿐 아니라 클래스도 생성 가능합니다.

## 제네릭 클래스

제네릭 클래스도 마찬가지로 제네릭 인터페이스와 형태가 비슷합니다.

제네릭 클래스의 특징으로는 클래스 이름 뒤에 `<>` 안쪽에 제네릭 타입 매개변수 목록을 가집니다. 해당 클래스로 인스턴스 생성 시 타입에 어떤 값이 들어갈지 지정해 주면 됩니다.

```tsx
 class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
} // 제네릭 클래스
 
let myGenericNumber = new GenericNumber<number>(); // type: number로 지정
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

클래스는 두 가지 타입을 가집니다.

- `정적 측면`
- `인스턴스 측면`

제네릭 클래스는 `정적 측면`이 아닌 `인스턴스 측면`에서만 제네릭이므로, 클래스 작업 시 정적 멤버 클래스는 타입 매개변수에 쓸 수 없습니다.

## 제네릭 제약조건

제네릭 함수에서 타입 추론을 할 수 있게 힌트를 줄 수 있습니다.

```tsx
function loggingIdentity<T>(arg: T): T{
  console.log(arg.length); // Error: Type에는 length가 없습니다.
  return arg;
}
```

위 예제를 보면 `T` 가 어떤 타입인지 정의되지 않았기에 Error가 납니다. 이럴 경우 해당 타입을 정의하지 않고도 제약 조건을 걸어 타입에 제약을 걸 수 있습니다.

```tsx
interface Lengthwise {
  length: number;
}
 
function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Type: length 프로퍼티를 가진 Type
  return arg;
}

loggingIdentity(3); // Error: Number Type에는 'length'가 존재하지 않음
loggingIdentity({ length: 10, value: 3 }); // Success: 'arg.length' 코드는 속성 접근과 같이 동작하므로 성공
```

## 객체의 속성을 제약하는 방법

이름이 있는 객체에서 프로퍼티를 가져오고 싶은 경우, 다른 타입 매개변수로 제한된 타입 매개변수를 선언할 수 있습니다.

```jsx
function getProperty<T, Key extends keyof T>(obj: T, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 }; // type: 'a' | 'b' | 'c' | 'd'

getProperty(x, "a"); // Success: 타입 해당
getProperty(x, "m"); // Error: 타입에 'm'은 없습니다.
```