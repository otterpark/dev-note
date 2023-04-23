# **Narrowing**

TypeScript 에서 변수는 덜 정확한 타입에서 더 정확한 타입으로 변할 수 있는데 이러한 과정을 `type narrowing`이라고 합니다.

```tsx
interface Animal = {
    name: string;
    count?: number;
};

function addAnimal(animal: Animal){
  animal.count = animal.count + 1; // type 에러!! Object is possibly 'undefined'
}
```

위 예시를 보면 count 프로퍼티가 `undefined`가 될 수 있기 때문에 타입 에러가 발생할 수 있습니다.  `addAnimal`을 보면 inteface Animal의 count가 `number` 또는 `undefined`가 올 수도 있으므로 type 에러가 뜬다. 정리해 보면 현재 덜 정확한 타입에서 더 정확한 타입으로 변할 수 있게끔 하는 과정을 `type narrowing` 이라고 한다.

지금부터는 6가지 방법의 `type narrowing` 방법을 알아보자!

## 1. `typeof` type guards

JavaScript에서 연산자 유형을 제공하는데 `typeof` 를 통해서 `type narrowing`을 할 수 있다. 종류는 아래와 같다.

- String
- Number
- bigint
- boolean
- symbol
- undefined
- object
- function

```jsx
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    console.log(strs); // strs이 object 타입
  } else if (typeof strs === "string") {
    console.log(strs); // strs이 string 타입
  } else if (typeof strs === "null") {
    console.log(strs); // strs이 null 타입
  } else {
    // do nothing
  }
}
```

## 2. Truthiness narrowing

`boolean` 처럼 명령문의 조건이 항상 `true` or `false` 일 때 유용하게 사용되는 `type narrowing` 이다.

- `0`
- `NaN`
- `""` (the empty string)
- `0n` (the `bigint` version of zero)
- `null`
- `undefined`

```tsx
interface Animal = {
    name: string;
    count?: number;
};

/*
function addAnimal(animal: Animal){
  animal.count = animal.count + 1; // type 에러!! Object is possibly 'undefined'
}

이전 에러는 아래와 같이 수정하면 통과한다.
*/

function addAnimal(animal: Animal){
 if (animal.count) {
  animal.count = animal.count + 1; 
 } 
}
```

## 3. **Equality narrowing**

TypeScript는 스위치 문 또는 `===`, `!==`, `==`, `!=` 와 같은 등식 검사를 유형을 좁힌다.

```tsx
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // 위의 등식 검사를 통과하면 x, y는 String 이다.
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x); // string | number
    console.log(y); // string | boolean
  }
}

function printAll(strs: string | string[] | null) {
  if (strs !== null) {
  // 여기를 통과하면 strs는 string | string[]
    if (typeof strs === "object") {
   // 여기를 통과하면 strs는 string[]
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
   // 여기를 통과하면 strs는 string
      console.log(strs);
    }
  }
}

interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  if (container.value != null) {
  // 아래 등식을 통과 시 undefiend 와 null 타입이 제거된 number로 된다. 
    console.log(container.value);
    container.value *= factor;
  }
}
```

## 4. The `in` operator narrowing

TypeScripts는 `in` 연산자를 통해 이름의 속성을 이용하여 잠재적인 유형을 좁힐 수 있다.

```tsx
interface Person {
    firstName: string;
    lastName: string;
}
interface Organization {
    name: string;
}
type Contact = Person | Organisation;

function sayHello(contact: Contact) {
    if ("firstName" in contact) {
    /* 
     아래 등식을 통과 시 contact는 Person 타입으로 
     좁혀지고 타입 에러가 발생하지 않는다.
    */
        console.log("Hello " + contact.firstName);
    }
}
```

## 5. `instanceof` narrowing

`instanceof` 를 사용하여 해당 변수가 사용하고 있는 prototype chain을 통해 포함되어 있는지를 체크하여 타입을 좁힌다.

```tsx
class Person {
    constructor(
        public firstName: string,
        public lastName: string
    ) {}
}
class Organisation {
    constructor(public name: string) {}
}
type Contact = Person | Organisation;

function sayHello(contact: Contact) {
  /*
     console.log("Hello " + contact.firstName);
   만약 위와 같이 출력한다면 contact이 
  */
  if (contact instanceof Person) {
     console.log("Hello " + contact.firstName);
  }
}
```

## 6. **Assignments**

변수 할당을 통해 타입을 좁힌다.

```tsx
let x = Math.random() < 0.5 ? 10 : "hello world!"; // x 타입은 number | string

x = 1;
console.log(x); // x 타입은 number

x = "goodbye!";
console.log(x); // x 타입은 string
```

## 7. **Control flow analysis**

코드 분석을 통해 제어 흐름을 파악하여 유형을 좁힌다.

```tsx
function example() {
  let x: string | number | boolean;
  x = Math.random() < 0.5;
  console.log(x); // type 은 boolean
             
 
  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x); // type 은 string
  } else {
    x = 100;
    console.log(x); // type 은 string
  }
 
  return x;  // type 은 string | number
       
}
```

## 8. **Using type predicates**

TypeScript 컴파일러에 특정 값이 어떤 유형인지 알리는 특수 반환 유형이다.

```tsx
type Rating = 1 | 2 | 3 | 4 | 5;

async function getRating(productId: string) {
 const response = await fetch(
  /products/${productId}
 );
 const product = await response.json();
 const rating = product.rating;
 return rating;
}
```

위 예제에서 `getRating`의 리턴 타입은 `Promise<any>`로 추측 되는데, 이 함수의 리턴 값을 변수에 할당 시 `any` 타입을 가지게 된다.

이 말은 즉 아무런 타입 확인이 일어나지 않는다는 말이 된다.

이때 우리는 `type predicate` 를 가지게 함수를 만들어 더욱 코드 타입을 안정적이게 할 수 있다.

```tsx
function isValidRating(
  rating: any
): rating is Rating {
  if (!rating || typeof rating !== "number") {
    return false;
  }
  return (
    rating === 1 ||
    rating === 2 ||
    rating === 3 ||
    rating === 4 ||
    rating === 5
  );
}
```

`rating is Rating` 은 바로 `type predicate` 입니다. `type predicate` 를 사용 시에는 무조건 `boolean` 값을 리턴해야한다. 이제 위에 코드를 수정해보자.

```tsx
async function getRating(productId: string) {
  const response = await fetch(
    `/products/${productId}`
  );
  const product = await response.json();
  const rating = product.rating;
  if (isValidRating(rating)) {
    return rating; // type: Rating
  } else {
    return undefined;
  }
}
```

`isValidRating()`을 통해 if문 안의 `rating`의 타입은 Rating이 된다.

## 9. **Discriminated unions**

각 Union안에 동일한 키(key)를 갖지만 다른 값(value)을 갖도록 만들 수 있다.

### 1. `DisCriminated unions`을 사용 안한 경우

```tsx
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

### 2. `DisCriminated unions`을 사용한 경우

```tsx
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
  }
}
```

- `DisCriminated unions`을 사용 시 코드를 조금 더 직관적으로 작성 가능
- 공통적인 `property`를 통해 구분이 쉬움

## 10. The `never` type

`Narrowing`을 사용하여 범위를 계속 좁히다 보면 아무것도 남지 않는 수준으로 줄여질 수 있는데 이 경우는 `TypeScript`에서 존재해서는 안 된다는 것을 나타내기 위해 `never` type을 사용한다.

> `never` 유형은 모든 유형에 할당할 수 있지만 `never` 유형에 `never` 유형을 할당할 수 없다.
>

## 11. **Exhaustiveness checking**

예상치 못한 런타임 에러를 미연에 방지하기 위해 철저하게 검사한다.

```tsx
type Shape = Circle | Square;
 
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
   /* 
    _exhautiveCheck 까지 왔다면 circle 도 square 도 아니라 어떤 type인지
    체크가 불가능하다. 이러한 경우 never type을 사용한다.
   */
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

여기서 만약 새로운 Shape를 추가하면 어떻게 될까?

```tsx
interface Triangle {
  kind: "triangle";
  sideLength: number;
}
 
type Shape = Circle | Square | Triangle;
 
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      /* 
        아래와 같이 _exhaustiveCheck에 never type을 적용하면 에러가 뜬다.
        현재 Shape는 Circle | Square | Triangle 타입이 있는데, switch 문 중 Triangle 타입에 대한 케이스가 없기에 에러 메세지가 뜬다.
        Type 'Triangle' is not assignable to type 'never'.
      */
      const _exhaustiveCheck: never = shape;

      return _exhaustiveCheck;
  }
}
```