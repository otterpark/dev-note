# Conditional Types - 타입 시스템에서 if문처럼 작동하는 타입

`Conditional Types`를 사용하면 타입관계 검사로 표현된 조건에 따라 두 가지 타입 중 하나를 선택하여 정의할 수 있습니다. JavaScript의 삼항 연산자 조건문(`condition ? trueExoression : falseExpression`)의 형태와 비슷한 형태를 가졌습니다.

```tsx
type ConditionalType = SomeType extends OtherType ? TrueType : FalseType
```

위 내용을 글로 풀어서 쓰면 아래와 같습니다.

> SomeType 타입이 OtherType 타입에 할당 가능한 타입이면 TrueType이고, 그렇지 않으면 FalseType

```tsx
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string; 
// type: Dog 타입(void)이 Animal 타입(void)에 할당 가능한 타입인가? Yes 그러므로 number

type Example2 = RegExp extends Animal ? number : string;
// type: RegExp 타입(string)이 Animal 타입(void)에 할당 가능한 타입인가? No 그러므로 number
```

`Conditional Types`는 `Generic`과 함께 사용될 때 조금 더 강력하다.

```tsx
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

createLabel 함수의 오버로드 예시이다. 하지만 오버로드는 새로운 타입이 늘어날 수록 코드는 복잡해지고 코드의 량도 늘어납니다. `Object Types`에 언급했듯이 이럴 때는 `generic function`을 사용하면 조금 더 간결하게 수정할 수 있다. 그럼 `Conditional Types`과 `generic function`을  같이 사용해보자!

```tsx
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;

function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}

let a = createLabel("typescript"); // type: NameLabal

let b = createLabel(2.8); // type: IdLabal

let c = createLabel(Math.random() ? "hello" : 42); // type: NameLabel | Idlabel
```

## 조건부 타입으로 제안하기

`Conditional Type` 의 장점 중 하나는 일반 유형의 가능한 실제 유형을 좁힐 수 있다는 점입니다.

```tsx
/*
  Flattn에 아무 타입의 배열이 주어지면, number를 사용한 인덱스 접근을 통해 String[] 요소 타입을 가져올 수 있습니다.
  만약 배열이 아닐 시에는 주어진 타입을 반환합니다.
*/
type Flatten<T> = T extends any[] ? T[number] : T;

type Str = Flatten<string[]>;
/*
  type: string[] 타입이 any[] 타입에 할당 가능한 타입인가? Yes
  그러므로 string[] 타입 배열에 인덱스 접근을 할 시 type은 string
*/
type Num = Flatten<number>;
/*
  type: number 타입이 any[] 타입에 할당 가능한 타입인가? No
  그러므로 type은 주어진 타입 그대로인 number
*/
```

## 조건부 타입 내에서 추론하기

조건부 타입은 `infer` 키워드를 사용해서 `참`을 비교하는 타입을 추론할 수 있습니다.

```tsx
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>; // type: number;

type Str = GetReturnType<(x: string) => string>; // type: string;

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>; // type: boolean[]
```

### infer 키워드

조건부 타입의 조건식이 참으로 평가될 때만 `infer` 키워드를 사용할 수 있다.

```tsx
T extends infer U ? X : Y
```

위 예제를 설명하면 U가 추론 가능한 타입이면 타입이면 X, 아니라면 Y이다.

> `infer` 키워드는 제약조건이 있는데, `extends`가 아닌 조건부 타입 `extends` 절에서만 사용 가능하다.

조금 더 이해를 높이기 위해 간단한 예제로 알아보자.

```tsx
type WhatType<T> = T extends Array<infer U> ? U : never;

const num: WhatType<number[]> // type: number
```

타입 변수 `U`는 `WhatType<number[]>` 으로 인해 Array 중 `any` 타입에 속하므로 타입은 `number`가 되고, `any` 타입에 속하지 않는 다른 타입은 절대 들어갈 수 없다.

## 분산적인 조건부 타입

`Generic Type` 위에서 `Conditional Type`은 `Union Type`을 만나면 분산적으로 동작합니다.

```tsx
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>; // type: string[] | number[]
```

조금 더 실용적인 예제를 알아보자!

```tsx
type Diff<T, U> = T extends U ? never : T;  // U에 할당할 수 있는 타입을 T에서 제거

type Filter<T, U> = T extends U ? T : never;  // U에 할당할 수 없는 타입을 T에서 제거

type DiffAlphabet = Diff<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>;  // Type: 'b' | 'd'
type FilterAlphabet = Filter<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>; // Type: 'a' | 'c'
```

```tsx
type Apple = {};
type Banana = {};
type Orange = {}

type Fruits = Apple | Banana | Orange;
type Box<T> = T extends any ? T[] : never;
type FruitBox = Box<Fruits>; // type: Apple[] | Banana[] | Orange[]
```
