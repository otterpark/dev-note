# Object Types

JavaScript에서는 객체를 통해서 데이터를 그룹화하여 전달한다. TypeScript에서 어떻게 객체 유형을 표현하는지에 대해 알아보자

1. **`interface version`**

    ```tsx
    interface Persion {
      name: string;
      age: number;
    }

    function greet(person: Person){
      return "Hello " + person.name;
    }
    ```

2. **`type version`**`

    ```tsx
    type Persion {
      name: string;
      age: number;
    }

    function greet(person: Person){
      return "Hello " + person.name;
    }
    ```

## 1. Property Modifiers

객체 유형에는 아래와 같이 3가지 경우가 있습니다

### 옵션 프로퍼티(`Optional Properties`)

속성 선택 사항을 `?` 로 표시할 수 있습니다

```tsx
interface PaintOptions {
  shape: Shape;
  xPos?: number; // 선택 사항 - type: number | undefined
  yPos?: number;  // 선택 사항 - type: number | undefined
}
 
function paintShape(opts: PaintOptions) {
 let xPos = opts.xPos === undefined ? 0 : opts.xPos; // type: number
 let yPos = opts.yPos === undefined ? 0 : opts.yPos; // type: number
}
 
const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

위의 `paintShape()`는 `xPos`와 `yPos`의 초기값을 설정해 줌으로써 타입을 `number`로 속성을 설정할 수 있습니다. 위 방법 같은 경우에 매개변수에 `구조 분해 할당` 을 사용하여 기본값을 줌으로써 코드를 더 간단하고 명시적이게 줄일 수 있습니다.

```tsx
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
 // xPos, yPos type: number
}
```

### 읽기 프로퍼티(`readonly Properties`)

속성에 `readonly` 로 표시하면 유형 검사 중 쓸 수 없습니다.

```tsx
interface SomeType {
  readonly prop: string;
}
 
function doSomething(obj: SomeType) {
  console.log(`prop has the value '${obj.prop}'.`);
  obj.prop = "hello"; // Error : Cannot assign to 'prop' because it is a read-only property

 /*
  obj.prop 에 대해서 읽을 수 있지만, 재 할당은 불가능하다. 
 */
}
```

`readonly`는 내부 내용을 변경할 수 없다는 의미는 아니며, 속성 자체를 다시 쓸 수 없음을 의미합니다.

```tsx
interface Home {
  readonly resident: { name: string; age: number };
}
 
function visitForBirthday(home: Home) {
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++; // 업데이트 가능
}
 
function evict(home: Home) {
 // Error: Cannot assign to 'resident' because it is a read-only property.
  home.resident = {
    name: "Victor the Evictor",
    age: 42,
  };
}
```

위 예제에서 보면 `readonly` 사용 시,

`visitForBirthday()` 에서 home.resident 속성을 읽은 후 업데이트가 가능하지만

`evict()` 에서는 속성 자체에 다시 쓸 수 없기 때문에 에러가 나는 것을 볼 수 있습니다.

추가적으로 `readonly` 는 생각보다 얕게 동작하여 참조하고 있는 값 자체는 변경될 수 없지만 그 안에 있는 속성들이 모두 동일 접근 제어자를 가지고 있는 것이 아니다.

```tsx
type readonlyA = {
 readonly fooA: string
};
type readonlyB = {
 readonly barB: {bob: string};
}

const x: readonlyA = {fooA: 'foo'};
x.fooA = 'boo'; // Error: 변경 불가

const y: readonlyB = {barB: {bob: 'rice'}};
x.barB.bob = 'pizza' // 변경 가능
```

정리하면 `readonly` 는

- 값의 속성을 읽기 전용으로 설정해준다.
- 함수가 매개변수로 받는 값을 변경없이 사용할 때 사용하기 적합하다.
- 개발자 관점에서 읽기 쉬운 코드로 만들어내기 용이하다.

### Index Signatures

가끔 우리는 작업을 하다보면 type의 모든 이름을 알지 못하지만 값의 형태가 어떻게 되는지 알고 있을 때가 있습니다. 이를 위해 `Index Signatures`는 <key, value> 형식으로 key와 value를 명시적으로 사용하는 경우 사용됩니다.

```tsx
interface StringArray {
  [index: number]: string;
}
 
const myArray: StringArray = getStringArray();
const secondItem = myArray[1]; // type: string
const secondItem = myArray[1.toSring()]; // type: string
```

> 💡 **JavaScript 기초 리마인딩**
JavaScript가 객체 인덱스 서명에 사용된 객체에 대해 `toString()` 을 호출한다.

```tsx
let obj = {
  toString(){
    return 'Hello'
  }
}

let foo: any = {};

foo[obj.toString()] = 'World'; // TypeScript는 명시적으로 호출하게 강제함
foo[obj] = 'World'; // Error: 인덱스 서명은 string, number 이어야 한다.
```

즉, TypeScript에서도 마찬가지로 인덱스 서명은 반드시 `string`, `number`, `symbols` 이여야 합니다.

`String index signature` 는 `index signature 구조` 를 따라야한다. 이 말은 즉, 모든 속성이 반환 유형과 일치하도록 강제합니다.

```tsx
interface Foo {
  [key: string]: number;
  x: number;
  y: number;
}

interface Bar {
  [key: string]: number;
  x: number;
  y: string; // Error: Property `y` must of of type number
}

/*
 Index Signatures 타입인 [key: string]: number를 사용하였으면,
 모든 멤버들은 key: string, value: number를 일치해야합니다.
 그래서 Bar['y']는 string type을 지정했으므로, 에러가 납니다.
 아래와 같이 union 타입으로 수정 시 에러는 해결됩니다.
*/

interface Bar {
  [key: string]: number | string;
  x: number;
  y: string; // Error: Property `y` must of of type number
}
```

## Extending Types

클래스처럼 객체의 속성 및 구조로 다른 타입과 합치거나, 확장시킬 수 있다. 이로 인해 재 사용성 높은 컴포넌트로 나눌 때, 유연함을 제공해 줍니다.

```tsx
interface Animal {
  kind: string;
 age: number;
}

interface Pig extends Animal {
  name: string;
}

const pig: Pig = {
 kind: 'Duroc',
 age: 3,
 name: 'piki'
} 
```

## ****Intersection Types****

`interface` 를 사용하면 다른 유형을 확장하여 새로운 유형을 만들 수 있습니다. 기존 객체 유형을 결합하는데 사용되는 `교차 유형`으로 보면 되며 `&` 연산자를 사용하여 정의합니다. (**`교집합 타입 설정`**)

```tsx
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;

function draw(circle: ColorfulCircle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}

draw({ color: "blue", radius: 42 }); // Success
```

## `Extending Types` vs ****`Intersection Types`****

우리는 `interface`를 사용하면 `extends` 절을 사용하여 다른 유형을 확장하는 법과, `Intersections` 의 교차 유형의 `&` 연산자를 사용하여 확장 후 새로운 유형을 만들었던 것을 알아보았다. 둘은 그럼 어떤게 다른 것일까? 결론적으로 둘의 차이점은 두 유형에 모두 `동일한 속성의 키를 가진 멤버가 처리되는 방식`의 차이이다.

### 1. `Extending Types`

```tsx
interface Parent {
  x: string | number;
}

interface Child1 extends Parent {
  x: string; // Success
}

interface Child2 extends Parent {
  x: string | number | boolean; // Error: boolean 유형
}
```

- `interface` extends 할 `interface` 가 유사한 유형을 제한된다

#### `extends` 다른 예제를 살펴보자

```tsx

interface NumberToStringConverter {
  convert: (value: number) => string;
}

interface BidirectionalStringNumberConverter extends NumberToStringConverter {
  convert: (value: string) => number;
}
```

위 코드는 에러가 난다. 위에서 설명했듯이 그 이유는 `NumberToStringConverter` 가 `BidirectionalStringNumberConverter` 와 동일한 키(key)를 사용하지만 호환되지 않는 값(value) 속성 유형이 호환되지 않기 때문이다. 값(value)과 값(value) 매개변수 유형이 호환되지 않는다는 메세지를 전달합니다. 추가적으로 정리하면 **정적으로 알려진 멤버가 있는 유형에만 사용할 수 있다**.

### 2. `Intersection Types`

```tsx
interface Parent {
 x: string | number;
}

type Child1 = Parent & {
 x: string; // Success
}

type Child2 = Parent & {
 x: string | number | boolean; // Success
}
```

- `type` 은 여러 유형을 확장하여 새로운 유형을 만들 수 있다(`교차 유형`)

#### `intersection types` 다른 예제를 보자

```tsx
type NumberToStringConverter = {
  convert: (value: number) => string;
}

type BidirectionalStringNumberConverter = NumberToStringConverter & {
  convert: (value: string) => number;
}
```

`type`은 여러 유형을 확장하여 새로운 유형을 만들 수 있기 때문에 에러가 나지 않는다.

## 일반 객체 유형(**Generic Object Types)**

일반 객체 유형은 포함된 요소 유형과 독립적으로 작동하는 일종의 컨테이너 유형입니다. 때문에 다양한 데이터 유형에서 재 사용할 수 있도록 도와줍니다.

```tsx
interface Box<Type> {
  contents: Type;
}

interface StringBox {
  contents: string;
}

let boxA: Box<string> = { contents: "hello" };
boxA.contents; // type: string
 
let boxB: StringBox = { contents: "world" };
boxB.contents; // type: string
```

`Box<string>`과 `StringBox`는 동일하게 작동합니다.

```tsx
interface Box<Type> {
  contents: Type;
}

interface Apple {
  color: string;
}

type AppleBox = Box<Apple>;

const appleBox: AppleBox = {
  contents: {
    color: 'red'
  }
}
```

`box`의 `Type`은 무엇이든 대체할 수 있는 점에서 재 사용이 가능하다.

`generic functions`를 활용하여, 오버로드를 완전히 피할 수 있다.

```tsx
// 함수 오버로드 유형 사용 예시
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}

```

```tsx
// 일반 개체 유형 사용 예시
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

코드가 오버로드를 사용했을 때와 `generic functions`를 사용했을 때가 훨씬 더 간결해 진것을 느낄 수 있다.

물론 `인터페이스`가 아닌 `유형 별칭`으로도 만들 수 있습니다.

```tsx
type Box<Type> = {
  contents: Type;
};
```

`인터페이스` 와 달리 `유형 별칭`을 사용하면 더 많은 `object types`를 묘사할 수 있습니다.

```tsx
type OrNull<Type> = Type | null; // type: Type | null
 
type OneOrMany<Type> = Type | Type[]; // type: Type | Type[]
 
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>; // type: OneOrMany<Type> | null
 
type OneOrManyOrNullStrings = OneOrManyOrNull<string>; // type: OneOrMany<string> | null
```

## 배열 타입 유형(Array Type)

일반 객체 유형를 이해했으면 개념적인 부분들은 똑같으므로 예제로 대체한다.

```tsx
interface Array<Type> {
  length: number;
  pop(): Type | undefined;
  push(...items: Type[]): number;
}
```

## `ReadonlyArray` 유형

`ReadonlyArray`는 변경되어서는 안 되는 배열을 설명하는 특수 유형이다.

```tsx
function doStuff(values: ReadonlyArray<string>) {
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`); // Success

  values.push("hello!"); // Error: 값 변경 불가
}
```

`ReadonlyArray`의 특징으로는 형식만 가리킨다. 만약 값과 함께 쓰였다면 에러가 납니다.

```tsx
new ReadonlyArray("red", "green", "blue"); // Error: ReadOnlyArray인데 값으로 사용되고 있다.
```

아래와 같이 형식만 가리키면 사용 가능합니다.

```tsx
const colorArr: ReadonlyArray<string> = ["red", "green", "blue"];
```

여기서 제일 주의해야할 점은 `Array` 와 `ReadonlyArray` 간에 양방향이 아니다라는 점이다.

```tsx
let x: readonly string[] = [];
let y: string[] = [];
 
x = y;
y = x; // Error: 'readonly String[]'은 'readonly' 이라 string[] 변수 유형에 할당 불가
```

`readonly` 대한 개념은 아래로 대체한다.

[읽기 프로퍼티(`readonly Properties`)](https://www.notion.so/readonly-Properties-5367d404353c498794c7e964af52290a)

## Tuple 유형

Tuple 유형은 포함된 요소의 수, 특정 위치에 포함된 유형을 정확하게 파악하기 사용되는 또 다른  `Array` 유형입니다.

```tsx
type StringNumberPair = [string, number];

function doSomething(pair: StringNumberPair) {
  const a = pair[0]; // type: string
  const b = pair[1]; // type: number
  // ...
}
 
doSomething(["hello", 42]);
```

위 예제를 보면 요소의 수는  `pair[0]`, `pair[1]` 이고, 특정 위치로는 `pair[0]` 의 유형은 string이며, `pair[1]`의 유형은 number 입니다.

```tsx
function doSomething(pair: [string, number]) {
  const c = pair[2]; // Error: 튜플 타입은 [string, number] 이고 length는 2인데 index 2는 없다.
}
```

Tuple 유형은 아래 interface 유형과 동일합니다.

```tsx
type StringNumberPair = [string, number];
```

```tsx
interface StringNumberPair {
  length: 2;
  0: string;
  1: number;
}
```

추가적으로 JavaScript의 배열 구조 분해 할당을 사용하여 `Tuple 유형`을 구조화 할 수 있습니다.

```tsx
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;
 
  console.log(inputString);
  console.log(hash);
}
```

Tuple 유형에 당연히 선택적 속성과 나머지 요소를 가질 수 있습니다.

```tsx
type Either2dOr3d = [number, number, number?];
 
function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
 
  console.log(`Provided coordinates had ${coord.length} dimensions`); // length: 2 | 3
}
```

```tsx
/*
 첫 번째 요소는 string 타입
 두 번째 요소는 number 타입
 세 번째 요소는 boolean 타입의 배열 나머지 요소
*/
type StringNumberBooleans = [string, number, ...boolean[]];

/*
 첫 번째 요소는 string 타입
 두 번째 요소는 boolean 타입의 배열 나머지 요소
 세 번째 요소는 number 타입
*/
type StringBooleansNumber = [string, ...boolean[], number];

/*
 첫 번째 요소는 boolean 타입의 배열 나머지 요소 
 두 번째 요소는 string 타입
 세 번째 요소는 number 타입
*/
type BooleansStringNumber = [...boolean[], string, number];
```

나머지 매개변수 및 인수를 활용하여 기존에 배웠던 `함수 유형`을 보다 명시적으로 표현할 수 있습니다.

```tsx
// 함수 유형
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}

// Tuple 형식
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

> 💡 Tuple 유형은 **“명백한” 규약 기반 API**에 사용하기에 유용합니다.

## `readonly` Tuple 유형

Tuple 유형의 `readonly`는 `as const` 를 사용하면 똑같이 `readonly` Tuple 유형으로 유추된다는 특징이 있습니다.

```tsx
// 기본 readolny tuple 유형
function doSomething(pair: readonly [string, number]) {
  // ...
}

// as const를 사용하여 readonly tuple 유형을 구현
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) { 
  return Math.sqrt(x ** 2 + y ** 2);
}
 
// readonly를 사용하지 않았지만, point 유형이 readonly[3, 4]로 유추 
distanceFromOrigin(point); 
/*
 Error: 
 point 유형은 [3, 4] 형태로 변하지 않는다.
 tuple 유형으로 자유로운 [number, number] 포인트 형태로 올 수 있기에
 요소가 변형되지 않는다는 것을 보장할 수 없기 때문에 에러가 난다.
*/
```
