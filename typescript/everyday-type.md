# Everyday Types


TypeScript 코드를 작성할 때 가장 기본적이면서 흔이 만날 수 있는 타입들에 대해 알아보자

## 1. 원시 타입

`string`

문자열 값은 String

`number`

int, float 같은 건 존재하지 않고 모든 수는 Number

`boolean`

True false 값은 boolean

`bigint`

아주 큰 정수를 다루기 위한 원시 타입

```jsx
// BigInt 함수를 통하여 bigint 값을 생성
const oneHundred: bigint = BigInt(100);
 
// 리터럴 구문을 통하여 bigint 값을 생성
const anotherHundred: bigint = 100n;
```

`symbol`

전역적으로 고유한 참조값을 생성하는 데에 사용할 수 있는 원시 타입

```jsx
const firstName = Symbol("name");
const secondName = Symbol("name");
 
if (firstName === secondName) {.
  // 절대로 일어날 수 없습니다
}
```

## 2. 배열

`[1, 2, 3]`과 같은 배열의 타입을 지정할 때 `number[]` 또는 `Array<number>` 처럼 지정 가능

## 3. any

어떤 타입을 할당해야 할지 모를 때 사용하는 타입

- 변수 선언 후 초기화 과정에서 아무런 값을 할당하지 않을 시 `any` 타입이 자동 지정

    ```jsx
    noImplicityAny를 설정 시 any로 간주하는 모든 경우에 오류 발생을 시키게 할 수 있음
    ```

## 4. 변수에 대한 타입 표기

`const`, `var`, `let` 등을 사용하여 변수를 선언할 때 타입을 명시적으로 지정하기 위해 타입을 표기할 수 있다

```jsx
let myName: string = 'Park';

let yourName = 'Ho';
/* 
 TypeScript에서는 자동으로 초깃값을 바탕으로 타입 추론
 let yourName: string = 'Ho';
*/
```

가능하면 최대한 타입 표기를 적게 사용하게끔 타입 지정

## 5. 함수

```jsx
// 매개변수 타입 표기
function myName(name: string) {
 console.log(`Hello, my name is ${name}`);
}

// 반환 타입 표기
function getFavoriteNumber(): number {
 return 7;
}
```

- 함수도 변수와 마찬가지로 반환 타입을 표기하지 않아도 되며, `return` 문을 바탕으로 반환 타입을 추론함

## 6. 익명함수

```jsx
const names = ['Park', 'Hoo'];

names.forEach((name) => {
 console.log(name.toUppercase());
});
```

- TypeScript는 함수가 실행되는 문맥을 통해 해당 함수가 가져야하는 타입을 알 수 있다. 이러한 과정은 `문맥적 타입 부여` 라고 칭한다.

## 7. 객체 타입

```jsx
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });

// 옵셔널 프로퍼티 예시
function printName(obj: { first: string; last?: string }) {
 
 /*
  console.log(obj.last.toUpperCase()); 

  위와 같이 사용하면 아래와 같은 에러가 뜬다. 존재하지 않은 프로퍼티에 접근 시
  런타임 오류가 아닌 'undefiend` 값을 얻게 된다.

  Error: 'obj.last' is possibly 'undefiend'.
 */

 // 그래서 오류가 안 뜨게 하려면 아래와 같이 체크가 필요하다.
 if (obj.last !== undefined) {
    console.log(obj.last.toUpperCase());
  }
 
  // 최신 JavaScript 문법을 사용하였을 때 또 다른 안전한 코드
  console.log(obj.last?.toUpperCase());

 return `${first} ${last}`;
}
printName({ first: "Bob" }); // last는 옵셔널 프로퍼티이므로 선택적으로 넣거나 빼도 상관없다
printName({ first: "Alice", last: "Alisson" });
```

- 프로퍼티 구분 시 `,` 또는 `;` 으로 구분이 가능하며 타입을 지정하지 않을 시에는 `any` 타입으로 간주한다.
- 객체 타입에 `옵셔널 프로퍼티` 를 지정할 수 있는데 선택적 타입으로 `?` 를 붙이면 된다.

## 8. 유니언(의미: 합집합) 타입

서로 다른 두 개 이상의 타입들을 사용하여 만드는 것으로 조합에 사용된 타입 중 무엇이든 하나의 타입을 가질 수 있습니다.

```jsx
function printId(id: number | string) {
  console.log("Your ID is: " + id);

 /*
  위에서 유니언 타입으로 number와 string을 지정하였는데,
  string에서만 유효한 메서드를 사용할 수 없습니다.
 */
 console.log(id.toUpperCase()); // 오류

 /*
  만약 억지로 쓰고 싶다면 typeof 연산을 사용하여 체크 후 사용할 수 있습니다.
 */
 if (typeof id === "string") {
    // 이 분기에서 id는 'string' 타입을 가집니다
    console.log(id.toUpperCase());
  } else {
    // 여기에서 id는 'number' 타입을 가집니다
    console.log(id);
  }
}

printId(101); // 성공
printId("202"); // 성공
printId({ myID: 22342 }); // 오류
```

## 9. 타입 별칭

똑같은 타입을 한 번이 아닌 `여러 번 재 사용하기 위함` 또는 `다른 이름으로 부르고 싶은 경우`를 위해 타입 별칭을 사용합니다.

```jsx
type ID = number | string; // 유니언 타입: number, string
 
/* 
 function printCoord(pt: { x: number; y: number }) {
   console.log("The coordinate's x value is " + pt.x);
   console.log("The coordinate's y value is " + pt.y);
 }
 위 코드와 같은 코드입니다.
*/

type Point = {
  x: number;
  y: number;
}; // Point 타입은 x: number, y: number

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });

// 다른 이름으로 부르고 싶은 경우
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
```

## 10. 인터페이스

시용 방법은 이전에 배운 `타입 별칭` 과 동일하다.

```jsx
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

### 💡 여기서 잠깐!

그럼 **타입과 인터페이스의 차이점** 은 무엇일까?

#### 1. `interface 확장하기`

```jsx
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear()
bear.name
bear.honey
```

기존 interface에 새 필드 추가가 `가능`하다.

```jsx
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

#### 2. `type 확장하기`

```jsx
type Animal = {
  name: string
}

type Bear = Animal & {
  honey: Boolean
}

const bear = getBear();
bear.name;
bear.honey;
```

type은 생성 된 이후 확장이 `불가능`하다.

```jsx
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

// Error: Duplicate identifier 'Window'.
```

**타입과 인터페이스의 차이점**

- 타입 별칭은 선언 병합에 포함될 수 있지만, 인터페이스는 포함될 수 있습니다.
- 인터페이스는 오직 객체의 모양을 선언하는 데에만 사용되며, 기존의 원시 타입에 별칭을 부여하는 데에는 사용할 수 없습니다.
- 인터페이스 이름은 항상 있는 그대로 오류 메시지에 나타납니다.
(인터페이스가 이름으로 사용되었을 때)

> 개인적인 선호에 따라 `interface` 또는 `type` 을 선택할 수 있지만, 코드가 조금 더 명시적이고 유연한 `interface` 를 사용하고 이후 문제가 발생 했을 시 `type` 을 사용하면 좋다.
>

## 11. 타입 단언

TypeScript 보다 당신이 어떤 값의 타입에 대한 정보를 더 잘 아는 경우 구체적으로 명시하기 위해 서용된다.

```jsx
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

/*
 위 코드에서 getElementById를 사용 시 반환 값을 TypeScript는 HTMLElement 중 
 무언가 반환된다는 것을 알지만 우리는 HTMLCanvasElement가 반환된다는 사실을 알고 있으므로
 위와 같이 타입 단언을 할 수 있다.
*/

// 아래와 같이도 사용 가능하다.
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

 타입 단언은 컴파일러에 의하여 제거되며 코드의 런타임 동작에는 영향을 끼치지 않는다.

## 12. 리터럴 타입

일반적인 타입(`string`, `number`)과 다르게 구체적인 문자열과 숫자 값을 타입 위치에 지정할 수 있다.

```jsx
/*
 문맥적 타입으로 인해 
 let changingString = 'Park'; 같은 경우에 타입은 string으로 모든 문자열을 입력이 가능
 
 const constantString = 'Park' 같은 경우에는 const로 선언 시 재 할당이 불가능 하므로
 'Park` 이라는 타입으로 고정
*/

let changingString = "Park";
changingString = "Hoo";

changingString; // let changingString: string
      
let changingString: string
 
const constantString = "Park";
constantString;  // let changingString: 'Park'

// 구체적인 문자열을 사용한 예
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "center");

// 구체적인 숫자를 사용한 예
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

### 리터럴 추론

객체를 사용하여 변수를 초기화하면, TypeScript는 해당 객체의 프로퍼티는 이 후에 값이 변화할 수 있다고 가정합니다.

```jsx
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);

// Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.

// 수정 1:
const req = { url: "https://example.com", method: "GET" as "GET" };

// 수정 2
handleRequest(req.url, req.method as "GET");

// as const로 객체 전체를 리터럴 타입으로 변환 가능
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

## 13. null과 undefined

JavaScript에는 빈 값 또는 초기화되지 않는 값을 가리키는 두 가지 원시값이 존재합니다.

TypeScript에는 각 값에 대응하는 동일한 이름의 두 가지 타입이 존재하는데 이는 `strictNullChecks` 옵션에 따라 설정이 달라진다.

### `strictNullChecks` 설정 시

어떤 값이 `null` 또는 `undefined` 일 때, 해당 값과 함께 메서드 또는 프로퍼티를 사용하기에 앞서 해당 값을 테스트 해야한다.

```jsx
function doSomething(x: string | undefined) {
  if (x === undefined) {
    // 아무 것도 하지 않는다
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}

// Null 아님 단언 연산자
function liveDangerously(x?: number | undefined) {
  /* 
  명시적 검사 없이 표션식 뒤에 '!'를 작성하면 해당 값이 'null' 또는 'undefiend'가
  아니라고 타입을 단언할 수 있으며 반드시 해당 값이 'null' 또는 'undefiend'가 아닌 
  경우에만 사용해야 합니다.
 */
  console.log(x!.toFixed());
}
```

### `strictNullChecks` 설정되지 않았을 때

`null`, `undefined`를 모든 타입 변수에 대입할 수 있으며, `Null` 검사를 하지 않는 언어의 동작 방식과 유사하다. `Null` 검사의 부재는 버그의 주요 원인이 되기도 하여 별다른 이유가 없다면 `strictNullChecks` 옵션 설정을 권장한다.
