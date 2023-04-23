# More on Functions

TypeScript 함수들이 어떻게 호출될 수 있는지를 설명하는 방법을 알아보자

## 1. Function Type Expressions(함수 타입 유형식)

```tsx
// (a: string)의 의미는 fn 함수는 a에 string Type인 파라미터를 가진 함수의 뜻이다. 
// Type을 지정하지 않을 시에는 암묵적으로 any Type이 지정된다.
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

// 물론 위 형식을 아래와 같이 'type alias'로 사용 가능하다.
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
 
function printToConsole(s: string) {
  console.log(s);
}
 
greeter(printToConsole);

```

## 2. **Call Signatures(호출 시그니처)**

```tsx
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

`함수 유형식`(Function type expression)은 `=>`를 사용하고 `개체 유형식`(Call Signatures)은 `:` 을 사용한다.

## 3. Construct Signatures(구성 시그니처)

생성자 함수 타입 지정 방식이다

```tsx
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

상수 유형식을 활용하여 아래와 같이 사용할 수 있다.

```tsx
interface CallOrConstruct {
  new (s: string): Date; // construct
  (n?: number): number; // call
}

// construct
let object: Date = new CallOrConstruct("optional string s");
// call
let myNumber: number = CallOrConstruct(/* n= */ 42);
```

하나의 함수 객체가 두 가지 목적을 수행하는 경우 `TypeScript`로 양쪽 모두 타입을 지정하여 결합시킬 수 있다.

## 4. 제네릭 함수

제네릭 문법이 두 값 사이에 상관관계를 표현하기 위해 사용된다.

```tsx
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}

// s는 "string" 타입
const s = firstElement(["a", "b", "c"]);
// n은 "number" 타입
const n = firstElement(ㅏ[1ㅈ, 2, 3]);
// u는 "undefined" 타입
const u = firstElement([]);
```

**추론**

`Type`을 특정하지 않았음에도 `Type`을 자동적으로 추론합니다.

```tsx
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
 
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
/*
 여기서의 매개변수 n은 type이 string으로 추론되며, 
 parsed의 type은 number[] type으로 추론한다.
*/
```

**타입 제한 조건**

두 값을 연관시키기 위해 특정한 값들의 부분집합에 한해 동작하게 타입을 제한할 수 있다.

```tsx
// 아래 같이 정의함으로써 a, b의 매개변수에 대해서 .length 프로퍼티에 접근할 수 있어야한다.
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
const longerArray = longest([1, 2], [1, 2, 3]); // type: number[]
const longerString = longest("alice", "bob"); // type: alice | bob
const notOK = longest(10, 100); // Error: Number object에는 length 프로퍼티를 가지고 있지 않음
```

**타입 인수를 명시하기**

TypeScript는 제네릭 호출에서 의도된 type을 대체로 추론해 내지만 예외인 부분도 있다.

```tsx
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}

const arr = combine([1, 2, 3], ["hello"]); // Error: Type 'string' is not assignable to type 'number'.

// 아래와 같이 의도 했다면 수동으로 type을 명시해야한다.
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

**좋은 제네릭 함수를 작성하기 위한 가이드라인**

- 타입 매개변수를 누르기

    ```tsx
    // A 방식
    function firstElement1<Type>(arr: Type[]) {
      return arr[0];
    }
    
    // B 방식
    function firstElement2<Type extends any[]>(arr: Type) {
      return arr[0];
    }
     
    // a: number (good)
    const a = firstElement1([1, 2, 3]);
    // b: any (bad)
    const b = firstElement2([1, 2, 3]);
    
    /*
     A 방식이 함수를 작성하는데 더 좋은 방법이며, 최대한 타입 매개변수를 제약하기
     보다는 타입 매개변수를 최대한 사용한다.
    */
    ```

- 더 적은 타입 매개변수를 사용하기

    ```tsx
    // A 방식
    function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
      return arr.filter(func);
    }
    
    // B 방식
    function filter2<Type, Func extends (arg: Type) => boolean>(
      arr: Type[],
      func: Func
    ): Type[] {
      return arr.filter(func);
    }
    
    /*
     A 방식이 함수를 작성하는데 더 좋은 방법이며, 
     항상 가능하다면 타입 매개변수를 최소로 사용한다. 
    */
    ```

- 타입 매개변수는 두 번 나타나야 한다.

    ```tsx
    // A 방식
    function greet<Str extends string>(s: Str) {
      console.log("Hello, " + s);
    }
    
    // B 방식
    function greet(s: string) {
      console.log("Hello, " + s);
    }
    
    greet("world");
    
    /*
     타입 매개변수는 여러 값의 타입을 연관시키는 용도로 사용함으로 B 방식인
     기존 함수 타입 유형식의 함수를 작성하는데 더 좋은 방법이다.
    */
    ```

## 5. 선택적 매개변수

`?` 로 표시함으로 선택적 매개변수를 만들 수 있다.

```tsx
function f(x?: number): void {
  // ...
}
f(); // OK
f(10); // OK
f(undefined); // OK

/*
 매개변수 타입이 number로 지정되어 있지만 명시되지 않은 매개변수는 undefiend가
 되기 때문에, x 매개변수는 실질적으로 number | undefiend 타입이 될 것이다.
*/
```

**콜백 함수에서 선택적 매개변수**

선택적 매개변수 및 함수 타입 표현식에 대해 알게 되면, 콜백을 호출하는 함수를 작성할 때 잦은 실수들이 발생한다.

```tsx
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // 오늘은 index를 제공하고 싶지 않아
    callback(arr[i]);
  }
}

myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));

/*
 위 함수의 의도는 위 두 호출 모두 유효하기를 바래서 사용했지만,
 의미하는 바는 callback이 하나의 인수로 호출될 수 있다 라는 이유가 되기 때문에
 TypeScript는 이러한 의미를 강제하여 실제로 일어나지 않게 에러를 발생시킨다.
*/
```

위 이야기는 즉 기존 JavaScript에서 매개변수로 지정한 것보다 많은 인수를 전달하여 호출 시 남은 인수들은 단순히 무시된다. TypeScript도 마찬가지 방식으로 동작한다고 보면 된다. 같은 타입을 가진 매개변수가 더 적은 함수는 더 많은 매개변수가 있는 함수를 대체할 수 있다.

> 💡 콜백에 대한 함수 타입을 작성할 때, 해당 인수 없이 호출할 의도가 없는 한, 절대로 선택적 매개변수를 사용하지 말자!

## 6. 함수 오버로드

동일한 이름에 매개변수만 다른 여러 버전의 함수를 만드는 것을 `함수 오버로드`라고 한다. 다시 말해 함수 이름은 같지만 매개변수 또는 반환값의 타입이 다른 함수라고 이야기할 수 있다. 함수 오버로드는 `선언부` 와 `구현부` 로 나뉘어 지는데 `선언부` 에서는 가능한 매개변수 타입만 지정하고, 함수 구현은 `구현부` 에서만 이루어진다. 순서: `선언부` → `구현부`

- 함수 본문을 작성하기 위해 사용된 시그니처는 외부에 노출되지 않으며, 오버로드된 함수 작성 시 두 개 이상의 시그니처를 함수 구현 위에 작성해야 한다.
- 구현 시그니처는 오버로드된 시그니처와 호환이 되어야 한다.
- `선언부` 는 최소 두 개 이상이어야 한다.

```tsx
// 선언부
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// 구현부
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3); // Error: 선언부에 의하면 매개변수는 1개 or 3개
```

문자열 혹은 배열의 길이를 반환하는 함수인 예시이다.

```tsx
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}

len('');
len([0]);
len(1); // Error: 선언부의 타입과 일치하지 않음
```

위 함수는 `string | array`를 통해서 호출할 수 있습니다. 하지만, `TypeScript`는 하나의 오버로드를 통해서 함수를 해석하기 때문에 이 함수는 `string | array`가 될 수 있는 값을 통해서 호출할 수 없습니다.

이렇게 열심히 설명해 놓고 결국 공식 문서에서는 오버로드 쓰지말고 유니온 타입 쓰라고 한다

```tsx
function len(x: any[] | string) {
  return x.length;
}
```

매우 훨씬 간결하고 명시적으로 바뀌었습니다. 하하.

### 함수 내에서 `this` 선언하기

TypeScripts는 함수 안에서 `this` 가 무엇이 되어야 할지, 코드 흐름 분석을 통해 추론한다. `this`를 사용 시 **첫 번째 매개 변수**로 선언한다.

- JavaScript에서 `this` 라는 이름의 매개변수를 가질 수 없지만, TypeScript 는 해당 문법 공간을 함수 본문에서 `this` 의 타입 정의가 가능하다.

    ```tsx
    interface DB {
      filterUsers(filter: (this: User) => boolean): User[];
    }
     
    const db = getDB();
    const admins = db.filterUsers(function (this: User) {
      return this.admin;
    });
    ```

- 일반적으로 다른 객체가 함수를 호출할 때 제어하는 콜백 스타일 API에서 흔히 사용된다. 이런 효과를 얻기 위해서는 arrow 함수가 아닌 function 키워드를 사용해야한다.

    ```tsx
    interface DB {
      filterUsers(filter: (this: User) => boolean): User[];
    }
     
    const db = getDB();
    const admins = db.filterUsers(() => this.admin);
    /*
     Error: arrow 함수는 this의 전역 값에 해당한다는게 포인트
     The containing arrow function captures the global value of 'this'.
     Element implicitly has an 'any' type because type 'typeof globalThis' 
     has no index signature.
    */
    ```

## 알아야 하는 다른 타입

### `void`

값을 반환하지 않는 함수의 반환 값을 의미합니다. 보통 `return` 문이 없거나 명시적으로 반환되지 않을 때 추론되는 타입이다.

```tsx
function noop() {
 return;
}
```

- JavaScript 에서는 아무것도 반환하지 않는 함수를 `undefiend` 값을 반환합니다. 하지만 TypeScript에서 `void` , `undefiend` 는 같은 것으로 간주되지 않습니다.(`void` ≠ `undefiend`)

### `object`

원시 값(`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, `undefined`)이 아닌 모든 값을 지칭하며, 빈 객체 타입 `{ }` 와는 다르고, 전역 타입 `Object` 와도 다릅니다. (`object` ≠ `Object`)

> 💡 JavaScript에서 함수 값은 객체이다. 프로퍼티, 프로토타입 체인, `Object Prototype`, `instanceof Object`, `Object.keys` 등등이 있는데 TypeScript에서 함수 타입은 `object` 로 간주됩니다.

### `unknown`

`unknown` 타입은 모든 값을 나타냅니다. `any` 타입과 유사하지만 `unknown` 타입에 어떤 것을 대입하는 것이 유효하지 않는다.

```tsx
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b(); // Error: 'a' is of type 'unknown'.
}
```

`unknown` 타입의 값을 반환하는 함수를 만들어보자.

```tsx
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// 'obj'를 사용할 때 조심해야 합니다!
const obj = safeParse(someRandomString);
```

### `never`

`never` 타입은 결코 관측될 수 없는 값을 의미하며, 보통 함수의 예외 발생 또는 프로그램 실행 종료함을 의미한다.

```tsx
function fail(msg: string): never {
  throw new Error(msg);
}

function fn(x: string | number) {
  if (typeof x === "string") {
    // type: string
  } else if (typeof x === "number") {
    // type: number
  } else {
    x; // type: never
  }
}
```

### `Function`

전역 타입 `Function`은 `bind`, `call`, `apply`그리고 JavaScript 함수 값에 있는 다른 프로퍼티를 설명하는 데에 사용됩니다. 또한 여기에는 `Function` 타입의 값은 언제나 호출될 수 있다는 값을 가지며, 이러한 호출은 `any`를 반환합니다.

```tsx
function doSomething(f: Function) {
  return f(1, 2, 3);
}

/*
  위의 경우 타입되지 않은 함수 호출이며, 안전하지 않은 any 타입을 반환하기 때문에
 최대한 () => void 타입을 사용하는게 좋다.
*/
```

## 나머지 매개변수와 인수

### 나머지 매개변수(Rest Parameter)

우리는 보통 선택적 매개변수와 오버로드를 사용하여 다양한 정해진 인수들을 받을 수 있지만, 우리가 정해지지 않은 수의 인수를 받아들이는 함수를 `나머지 매개변수`를 이용하여 정의할 수 있다.

```tsx
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}

const a = multiply(10, 1, 2, 3, 4); // a -> [10, 20, 30, 40]
```

TypeScript에서는 매개변수에 대한 타입 표기는 암묵적으로 `any[]` 를 사용하며, 타입 표현식은 `Array<T>` 또는 `T[]` 또는 튜플 타입으로 표현해야 한다.

### 나머지 인수(Rest Argument)

반대로 전개 구문을 사용하여 배열에서 제공되는 인수의 개수를 제공할 수 있다. TypeScript는 배열이 불변하다고 간주하지 않는다.

```tsx
// 에러
const args = [8, 5];
const angle = Math.atan2(...args);
// Error: A spread argument must either have a tuple type or be passed to a rest parameter.

// 해결책
const args = [8, 5] as const;
const angle = Math.atan2(...args); // OK
```

## 매개변수 구조 분해(**Parameter Destructing)**

구조분해한 변수의 뒤에 타입을 지정하지 않고 구조분해한 `{ }`뒤에 콜론을 찍고 타입을 지정

```tsx
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

위 예제를 타입 별칭으로도 변환 가능합니다.

```tsx
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

## 함수의 할당가능성

### `void` 반환 타입

함수의 `void` 반환 타입은 예측할 수 있는 동작을 발생시킬 수 있다.

`void` 반환 타입으로의 문맥적 타이핑은 함수를 **아무것도 반환하지 않도록 강제하지 않습니다.**

```tsx
type voidFunc = () => void;
 
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};
```

리터럴 함수 정의가 `void` 반환 값을 가지고 있다면 그 함수는 어떠한 것도 **반환해서는 안됩니다.**
