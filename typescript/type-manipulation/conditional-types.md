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

### infer 키워드

조건부 타입의 조건식이 참으로 평가될 때만 `infer` 키워드를 사용할 수 있다.
> 제약 조건 내에서 참조 또는 반환할 변수를 정의 가능

```tsx
T extends infer U ? X : Y
```

위 예제를 설명하면 U가 추론 가능한 타입이면 타입이면 X, 아니라면 Y이다.

> `infer` 키워드는 제약조건이 있는데, `extends`가 아닌 조건부 타입 `extends` 절에서만 사용 가능하다.

조금 더 이해를 높이기 위해 간단한 예제로 알아보자.

```tsx
type GetReturnType<T> = T extends (...args: never[]) => infer U ? U : never;

type Num = GetReturnType<() => number>; // type: number

type Str = GetReturnType<(x: string) => string>; // type: string

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>; // type: boolean[]
```

`infer` 키워드를 조건부 타입에서 사용하면 타입 매개변수`<U>`가 함수인지 여부를 확인 후 검사 과정에서 반환 타입을 변수로 만들어 R을 유추한 뒤 검사에 성공하면 반환하는 방식으로 수행됩니다.

## 분배법칙 조건부 타입

`Generic Type` 위에서 `Conditional Type`은 `Union Type`을 만나면 분배법칙이  동작합니다.

```tsx
type IsNever<T> = [T] extends [never] ? true : false;
```

위 내용의 `IsNever`는 실제 `never` 타입인지 아닌지를 판단하는 타입입니다.

그럼 다음 예제를 보자.

```tsx
type A = IsNever<never> // type: true
type B = IsNever<boolean> // type: false
```

타입 A에 `IsNever<Never>`를 넣으면 타입이 `never`이니, 당연히 타입은 `true`가 나올 것이다. 마찬가지로 타입 B에 `IsNever<Boolean>`를 넣으면 `never` 타입이 아니니 타입은 `false` 일 것이다.

그렇다면 위 `IsNever`를 아래 형식을 바꿔보자!

```tsx
type IsNever<T> = T extends never ? true : false;
```

그리고 난 후 똑같은 타입 A, B를 줬는데 결과가 다르다.

```tsx
type A = IsNever<never> // type: never
type B = IsNever<boolean> // type: false
```

타입 A에 `IsNever<Never>`를 넣었더니 타입은 `never`가 되었고, 타입 B에 `IsNever<Boolean>`를 넣었더니 타입은 `false`가 나왔다.

> 여기서 질문! 왜 그럼 우리가 생각한 대로 나오지 않았을까?

일단 타입 A의 `never`는 `공집합`임과 동시에 `유니온`인데, 타입 매개변수`<T>`와 `유니온`이 만나면 분배법칙이 실행된다.

어 그런데 왜? `never`가 `유니온`이야? 우리가 알던 방식이 아닌데? 라고 할 수 있겠지만, `never`는 그 자체로 `유니온`입니다.

다시 돌아와서,

```tsx
type IsNever<T> = T extends never ? true : false;
type A = IsNever<never>
```

위 예제는 앞서 말했듯이 분배법칙이 타입 매개변수`<T>`와 유니온이 만났으므로 분배법칙이 실행되며, 여기서 알아야 할건 `never extends never`는 `never`이다. `never`는 공집합이기 때문에 제네릭에 들어는 가지만 분배법칙이 일어나지 않는다. 그러므로 type A는 타입이 `never` 되는 것이다.

> 우리는 우리가 의도하지 않은 분배법칙을 아래와 같이 제한할 수 있다.

```tsx
type IsNever<T> = [T] extends [never] ? true : false;  // 방법 1
type IsNever<T> = { type: T } extends { type: never } ? true : false;  // 방법 2
```

다른 예제들도 알아보자!

```tsx
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>; // type: string[] | number[]



type Diff<T, U> = T extends U ? never : T;  // U에 할당할 수 있는 타입을 T에서 제거
type Filter<T, U> = T extends U ? T : never;  // U에 할당할 수 없는 타입을 T에서 제거

type DiffAlphabet = Diff<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>;  // Type: 'b' | 'd'
type FilterAlphabet = Filter<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>; // Type: 'a' | 'c'
```

