# Classes

TypeScript는 ES5에 도입된 `Class`를 지원합니다.

TypeScript도 타입 주석들과 기타 구문들을 사용하여 클래스와 다른 타입간의 관계를 표현할 수 있습니다.

## Class Memebers

가장 기본 적인 클래스를 정의하는 방법입니다.

```tsx
class Point { }
```

## Fields

필드 선언은 클래스에 쓰기 가능한 공용 속성을 만듭니다.

```tsx
class Point {
  x: number; // Error: 'x'는 초기화가 이루어 지지 않았으며, 생성자 할당이 이루어지지 않았다.
  y: number; // Error: 'y'는 초기화가 이루어 지지 않았으며, 생성자 할당이 이루어지지 않았다.
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

위 구문에서 클래스 내에 `Fields`에 값을 할당하지 않거나 생성자 함수에 값을 할당하지 않을 경우 에러를 띄운다. 그 이유는 아래 옵션 때문이다.

### --strictPropertyInitialization

클래스 필드 여부를 설정 하는 옵션인데 옵션 활성화 시 생성자에서 `초기화`를 해줘야한다.

#### 방법 1 - 생성자에 직접 초기화

```tsx
class Point {
  x: number;
  y: number;
  constructor() {
    this.x = 0;
    this.y = 0;
  }
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

#### 방법 2 - definite assignment assertion operator인 `!`를 사용

```tsx
class Point {
  x!: string;
}
```

### `readonly`

셍성자 외부 필드애 대한 할당을 방지하려면 `readonly`를 사용하면 됩니다.

```tsx
class Point {
  readonly x: number = 0;
  constructor(pointX?: number) {
    if (pointX) {
      this.x = pointX;
    }
  }
}

const pt = new Point();
pt.x = 0; // Error: `readonly` 프로퍼티라 값 할당이 불가능
```

## Constructors

클래스 생성자는 함수와 매우 유사합니다. 타입 주석(`변수에 할당되는 값의 유형`), 기본값 및 오버로드를 사용하여 매개 변수를 추가할 수 있습니다.

```tsx
class Point {
  x: number;
  y: number;

  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```tsx
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) { }
}
```

클래스 생성자 시그니처들과 함수 시그니처 사이에는 차이점이 있습니다.

- 생성자는 타입 파라미터를 받을 수 없다(외부 클래스 선언에 속함)
- 생성자는 리턴 타입 주석(`type annotation(let num: number)`)을 가질 수 없습니다. (클래스 인스턴스 유형은 어떤 것이든 항상 반환됨)

## Super

기본 Class 문법과 동일하게 상속 시에 상속 받은 클래스에 super()를 불러 온 후 `this`로 접근할 수 있습니다.

```tsx
class Person {
  name: string;
  location: string;
  constructor() {
    this.name = '';
    this.location = '';
  }
}

class Park extends Person {
  computer: number;
  constructor() {
    this.computer = 2; // Error: 자식 클래스 생성자에서는 super() 먼저 쓴 이후 this를 호출할 수 있다.
    super();
  }
}
```

## Method

메서드는 함수 및 행성자와 동일한 `type annotations`를 사용할 수 있습니다.

```tsx
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

`type annotations` 외에 TypeScript 에서는 다른 새로운 것을 추가하지 않습니다.

추가적으로 메서드내에서 변수 접근 시 꼭 `this`를 사용해야한다.

```jsx
let x: number = 0;

class C {
  x: string = "hello";
  m() {
    /*
      Error:
      클래스 프로퍼티가 아닌 전역 변수 x를 가리킨다.
      그러므로 x는 number인데, string을 넣으려고 하니 number를 에러
    */
    x = "world";
  }
}
```

## Getter / Setter

TypeScript 에서는 특별한 엑세스 권한 규칙이 존재합니다.

- 만약 `get`이 존재하지만 `set`이 존재하지 않을 경우 프로퍼티는 자동적으로 `readonly`가 됩니다.
- 만약 `setter` 파라미터가 지정되지 않았을 경우, `getter`에서 반환 유형이 유추됩니다.
- `getter`와 `setter`는 반드시 동일한 `Member Visibility`를 가져야 한다.

> 추가적으로 로직에 타입 추론 상 추가 논리가 필요없는 경우에는 `getter`, `setter`를 사용 안 해도 된다.

```tsx
class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

`TypeScript 4.3` 부터는 `getter`와 `setter` 설정 시 다른 접근자를 가질 수 있습니다.

```tsx
class Thing {
  _size = 0;

  get size(): number {
    return this._size;
  }

  set size(value: string | number | boolean) {
    let num = Number(value);

    if (!Number.isFinite(num)) { // NaN, Infinity 등을 허용하지 않음
      this._size = 0;
      return;
    }
    this._size = num;
  }
}
```

## Index Signatures

클래스는 인덱스 서명(Index Signatures)을 사용할 수 있습니다.

```tsx
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);

  check(s: string) {
    return this[s] as boolean;
  }
}
```

하지만 클래스 인덱스 서명 같은 경우에는 메소드 유형도 맞춰야 하므로 유용하게 사용하기 쉽지 않습니다.
> 인덱싱된 데이터를 클래스 인스턴스가 아닌 다른 위치에 저장 시키는게 더 낫다.

## Class Heritage

객체 지향 기능이 있는 다른언어와 마찬가지로 상속이 가능하다.

### `implements` Clauses

`implements` 를 사용해서 클래스가 특정 interface를 충족하는데 체크할 수 있습니다.

```tsx
interface Pingable {
  ping(): void;
}

interface Pongable {
  pong(): void;
}

// 아래처럼 합쳐서 상속할 수 있다.
class Sonar implements Pingable, Pongable {
  ping() {
    console.log('ping!');
  }
  pong() {
    console.log('pong');
  }
}
```

#### 주의 사항

`implements`는 클래스의 인터페이스 유형으로 취급될 수 있는지 체크하는 의도일 뿐 클래스 `type`이나 `method` 를 변경하지 않습니다. `implements` 를 사용하면서 주요 오류 원인은 클래스 `type`을 바꾸려다가 에러가 많이 발생합니다.

```tsx
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) { // Error: 매개변수 's'는 암시적으로 'any' 유형을 갖습니다.
    return s.toLowerCase() === "ok";
  }
}
```

`implements`을 사용 시 클래스 본문이 검사되는 방식이나 `type`이 추론되는 방식을 변경하지 않습니다.

```tsx
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10; // Error: 'y'는 type 'c'에 존재하지 않는다.
```

### `extends` Clauses

클래스는 기본 클래스를 확장시킬 수 있습니다. 파생된 클래스는 기본 클래스의 모든 프로피티와 메서드를 가지며, 추가 멤버를 정의할 수 있습니다.

```tsx
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
d.move(); // Animal Class 메서드 사용 가능
d.woof(3); // Dog Class 메서드 사용 가능
```

### Overriding Methods

`Derived` 클래스는 `Base` 클래스 필드 또는 속성을 재 정의할 수 있습니다. 구문을 사용하여 `Base` 클래스 메서드에 엑세스 할 수 있으며, `JavaScript` 클래스는 단순한 조회 객체이므로 `super field` 라는 개념이 없다는 것을 인지해야 합니다.

`TypeScript`는 `Derived` 클래스가 항상 `기본 클래스의 하위 유형`임을 강제합니다.

```tsx
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

`Derived` 클래스는 `Base` 클래스를 따르는 것이 중요합니다. `Base 클래스 참조를 통해 Derived 클래스 인스턴스를 참조한다` 라는 것을 꼭 기억해야합니다.

```tsx
// `Base` 클래스 참조를 통해 `Derived` 인스턴스의 별칭을 지정한다.
const b: Base = d;
b.greet(); // Success
```

### `type` 전용 필드 선언

필드 선언에 대한 런타임 효과가 없어야 함을 `TypeScript` 에서 나타내기 위해서는 `declare`을 작성할 수 있습니다.

> target >= ES2022 또는 `useDefineForClassFields` 값이 `True`인 경우 클래스 필드는 상위 클래스 생성자가 완료된 후 초기화 되어 상위 클래스에서 설정한 값을 덮어씁니다.

```tsx
interface Animal {
  dateOfBirth: any;
}

interface Dog extends Animal {
  breed: any;
}

class AnimalHouse {
  resident: Animal;
  constructor(animal: Animal) {
    this.resident = animal;
  }
}

class DogHouse extends AnimalHouse {
// 자바스크립트 코드를 내보내지 않으며, type이 올바른지만 체크합니다.
  declare resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}
```

### Initilization Order

JavaScript 클래스가 초기화 하는 순서는 경우에 따라 의외가 생길 수 있습니다.

```tsx
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}

class Derived extends Base {
  name = "derived";
}

const d = new Derived(); // base가 출력
```

JavaScript에서 클래스 초기화 순서는 아래와 같습니다.

1. Base 클래스 필드가 초기화됩니다.
2. Base 클래스 생성자가 실행됩니다.
3. Derived 클래스 필드가 초기화됩니다.
4. Derived 클래스 생성자가 실행됩니다.

이 말은 즉, Derived 클래스 필드 초기화가 아직 실행되지 않았기 때문에 Base 클래스 생성자 중 이름에 대한 자체 값을 확인했다는 의미입니다.

### Built-in Types 상속(ES6 / ES2015 이상으로 설정되어 있어 패스)

## Member Visibility

TypeScript를 사용하여 특정 메서드 또는 클래스 외부의 코드에 표시되는지 여부를 제어할 수 있습니다.

### `Public`

클래스 맴버의 기본 가시성은 어디에서나 엑세스 가능한 `public` 입니다.

```tsx
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```

### `Protected`

`protected` 멤버는 선언된 클래스의 하위 클래스만 볼 수 있다.

```tsx
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // 여기에서 보호 된 구성원에 액세스 할 수 있습니다.
    console.log("Howdy, " + this.getName());
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
```

#### Exposure of protected members

상속된 하위 클래스는 상위 클래스를 따라야하지만 기본 클래스의 하위 유형을 노출할 수 있도록 따로 설정도 가능합니다. `protected`된 멤버도 `public`처럼 엑세스 할 수 있습니다.

```tsx
class Base {
  protected m = 10;
}
class Derived extends Base {
  m = 15; // 기본 설정은 public
}
const d = new Derived();
console.log(d.m); // Success: 15
```

Derived 클래스는 기본으로 설정되는 `public` 설정으로 인해 `protected`가 의미 없게 변경될 수 있습니다. Derived 클래스에서 `protected`가 의도적으로 노출되지 않도록 주의해야합니다.

#### Cross-hierarchy protected access

`Base` 클래스 참조를 통해 보호된 `x`에 엑세스 하는 것이 가능한지 여부에 따라 다른 OOP 언어가 동의하지 않습니다.

```tsx
class Base {
  protected x: number = 1;
}
class Derived1 extends Base {
  protected x: number = 5;
}
class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10;
  }
  f2(other: Base) {
    other.x = 10; 
    /*
      Error:
      속성 'x'는 'protected' 있으며 'Derived2' 클래스의 인스턴스를 통해서만 액세스할 수 있습니다. 'x'는 'Base' 클래스의 인스턴스입니다.
    */
  }
}
```

`Java`는 이를 합법적인 것으로 간주하지만, `C#`과 `C++`은 이 코드를 불법으로 간주합니다.

`Derived2`에서 `x`에 엑세스 하는 것은 `Derived2`의 서브 클래스부터만 합법적이어야 하고 `Derived1`은 서브 클래스에 속하지 않기 때문에 `TypeScript`는 `C#`과 `C++` 처럼 불법으로 간주합니다. `Derived1` 참조를 통해 `x`에 엑세스하는 것이 불법이라면, `Base` 클래스 참조를 통해 엑세스 한다고 해서 상황이 개선되는 건 아닙니다.

### `Private`

`Protected`와 같지만, 하위 클래스에서도 멤버에 대한 엑세스를 허용하지 않습니다.

```tsx
class Base {
  private x = 0;
}
const b = new Base();

console.log(b.x);
// Error: 'private' 속성으로 인해 'Base' 클래스 안에서만 접근이 가능하다.

class Derived extends Base {
  showX() {
    console.log(this.x); 
    // Error: 서브 클래스에서도 접근이 불가능하다.
  }
}
```

`private` 멤버가 Derived 클래스에 표시되지 않으면 Derived 클래스는 가시성을 높일 수 없습니다.

```tsx
class Base {
  private x = 0;
}
class Derived extends Base {
  x = 1;
  // Error: Base Class에 멤버변수 'x'를 'private'로 설정해 놓고 여기서는 'public'으로 설정하면 안 된다는 에러
}
```

### Cross-instance private access

같은 클래스의 서로 다른 인스턴스가 서로의 `private` 멤버에 엑세스 할 수 있을지에 대해서 객체지향 언어마다 의견이 다릅니다.`Java`, `C##`, `C++`, `Swift`, `PHP`와 같은 언어에서는 허용하지만 `Ruby`에서는 허용허지 않습니다.

`TypeScript`는 인스턴스 간 비공개 엑세스를 허용합니다.

```tsx
class A {
  private x = 10;

  public sameAs(other: A) {
    return other.x === this.x; // Success
  }
}
```

### 주의사항

TypeScript 타입 시스템의 다른 측면과 마찬가지로 `private`와 `protected`는 타입 검사 중에만 적용됩니다.

이 말은 즉, `in` 또는 단순 속성 조회와 같은 JavaScript 런타임 구문은 `private`와 `protected` 멤버에 엑세스 할 수 있습니다.

#### `JavaScript`

```js
class MySafe {
  private secretKey = 12345;
}

const s = new MySafe();
console.log(s.secretKey); // Success: 12345
```

`private`는 type 검사 중 대괄호(`[]`) 기법을 사용하여 엑세스를 허용할 수 있습니다. 이렇게 되면 `private`로 선언된 필드가 단위 테스트 같은 항복에 대해 잠재적으로 쉽게 엑세스할 순 있으나 `soft private`이고 개인 정보를 엄격하게 다루지는 못합니다.

```tsx
class MySafe {
  private secretKey = 12345;
}
const s = new MySafe();

console.log(s.secretKey); // Error: private으로 접근 불가능
console.log(s['secretKey']); // Success: 대괄호 기법으로 엑세스 가능
```

> `private`와 달리 `JavaScript`에서  `private`는 컴파일 후 비공개로 유지되며, 방금 언급한 대괄호 표기법(`[]`) 엑세스와 같이 `soft private`로 만들기 위한 어떠한 것도 제공하지 않으므로 `hard private` 입니다.

## `Static` 클래스의 정적 블록

클래스는 정적 멤버가 있을 수 있습니다. 이러한 정적 멤버는 클래스의 특성 인스턴스와 연관 되지 않습니다. `클래스 생성자 객체 자체를 통해 엑세스` 할 수 있습니다.

```tsx
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

`Static`도 마찬가지로 `public`, `protected`, `private` 접근 제어자를 사용할 수 있습니다.

```tsx
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
// Error: 'x'는 'MyClass' 내에서만 엑세스 할 수 있습니다.
```

당연히 상속도 가능합니다.

```tsx
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

### 특수 `Static` 이름들

일반적으로 함수 프로토타입에서 프로퍼티를 덮어쓰는 것은 매우 안 좋은 행동입니다. 클래스는 그 자체적으로 함수를 호출할 수 있는 함수이기 때문에 `new` 또는 특정 `static` 이름들에는 사용할 수 없습니다. `name`, `length`, `call`과 같은 함수 속성은 `Static` 멤버에 정의할 수 없습니다.

```tsx
class S {
  static name = "S!";
  // Error: Static 속성 'name'이 생성자 함수 'S'의 내장 속성 'name'과 충돌한다라는 에러
```

### Static 클래스가 없는 이유는 무엇일까요?

`TypeScript`에는 `C#`과 같은 방식으로 `static class` 라는 구성이 없습니다.

이러한 구조는 해당 언어가 **모든 데이터와 함수를 클래스 내부에 강제로 포함하기 때문**입니다. `TypeScript`에는 해당 제한 사항이 없으므로 필요하지 않습니다. 인스턴스가 하나인 클래스는 일반적으로 `JavaScript/TypeScript`에서 일반 객체로 표현됩니다.

예를 들어 일반 객체로도 충분히 작업을 수행할 수 있습니다. 그러므로 TypeScript는 `static class`가 필요하지 않습니다.

```tsx
// 불필요한 '정적' 클래스
class MyStaticClass {
  static doSomething() {}
}

// 방법 1
function doSomething() {}

// 방법 2
const MyHelperObject = {
  dosomething() {},
};
```

## `static` 클래스의 블록

`Static block`을 사용하면 클래스 내의 비공개 필드에 엑세스 할 수 있는 자체 범위가 있는 일련의 명령문을 작성할 수 있습니다. 이 명령문은 작성할 수 있는 모든 기능을 갖춘 초기화 코드를 작성할 수 있고, 변수가 노출되지 않으며, 클래스 내부에 대한 전체 엑세스 권한이 있습니다.

```tsx
class Foo {
    static #count = 0;

    get count() {
        return Foo.#count;
    }

    static {
        try {
            const lastInstances = loadLastInstances();
            Foo.#count += lastInstances.length;
        }
        catch {}
    }
}
```

## Generic Classes

`interface`와 마찬가지로 클래스도 `Generic`이 될 수 있다. `Generic Class`는 `new`를 사용하여 인스턴스화되면 해당 type 매개변수는 함수 호출에서만 동일한 방식으로 추론됩니다.

```tsx
class Box<Type> {
  contents: Type;
  constructor(value: Type) {
    this.contents = value;
  }
}

const b = new Box("hello!");
```

`class`는 `interface`와 동일한 방식으로 `generic constraints(일반 제약 조건)` 또는 기본 값을 사용할 수 있습니다.

## 정적 멤버의 type 매개 변수

아래 코드는 합법적이지 않으며, 그 이유가 명확하지 않습니다.

```tsx
class Box<Type> {
  static defaultValue: Type;
  // Error: static 멤버는 클래스 유형 매개 변수를 참조할 수 없습니다.
}
```

type은 항상 완전히 지워집니다. 런타임에서 `Box.defaultValue` 속성 슬롯이 하나만 있습니다. 이는 `Box<string>.defaultValue`를 설정하면 `Box<number>.defaultValue`도 변경 된다는 것을 의미(`매우 좋지 않음`)합니다. `Generic Class`의 `static` 멤버는 클래스 형식의 매개 변수를 참조할 수 없습니다.

## 클래스 런타임에 `this`

`TypeScript`는 `JavaScript`의 런타임 동작을 변경하지 않으며, `JavaScript`는 몇 가지 특이한 런타임 동작이 있는 것으로 유명하다는 점을 기억하는 것이 좋습니다.

`JavaScript`는 이를 처리하는 방식은 특이합니다.

```js
class MyClass {
  name = 'MyClass';
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: 'obj',
  getName: c.getName,
};

console.log(obj.getName()); // obj
```

`this`는 함수 호출 방식에 따라 결정됩니다. `obj` 참조를 통해 함수가 호출되었으므로 `this`는 클래스 인스턴스가 아닌 `obj`입니다.

TypeScript는 이러한 종류의 오류를 완화하거나 방지할 수 있는 몇 가지 방법을 제공합니다.

### 1. Arrow Functions

`this` context를 잃는 방식으로 자주 호출되는 함수가 있을 경우 메서드 정의 대신 `Arrow Functions` 속성을 사용하는 것이 합리적이다.

```tsx
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  };
}
const c = new MyClass();
const g = c.getName;

console.log(g()); // MyClass
```

여기에는 몇 가지 단점이 있습니다.

- `this` 값은 `TypeScript`로 검사되지 않은 코드의 경우 런타임에서 괜찮다고 생각합니다.
- 각 클래스 인스턴스는 이런 식으로 정의된 각 함수의 자체 복사본을 갖기 때문에 더 많은 메모리를 사용합니다.
- `prototype chain`을 통해 기본 클래스에서 메서드를 가져올 수 없기 때문에 전달된 클래스에서 `super.getName`을 사용할 수 없습니다.

#### `this` 파라미터

매서드 또는 함수 정의에서 초기 매개변수 `this`는 `TypeScript`에서 특별한 의미를 갖습니다. 이러한 매개변수는 `JavaScript` 컴파일 중에 사라집니다.

```tsx
// TypeScript 'this' 파라미터를 넣었을 경우
function fn(this: SomeType, x: number) {
  /* ... */
}
```

```js
// JavaScript에서는 컴파일 중에 사라집니다.
function fn(x) {
  /* ... */
}
```

`TypeScript`는 `this` 매개변수가 있는 함수를 호출할 때 올바른 컨텍스트에서 호출되는지를 확인합니다. 화살표 함수를 사용하는 대신 메서드 정의에 `this` 매개변수를 추가하여 메서드가 올바르게 호출되도록 정적으로 강제할 수 있습니다.

```tsx
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// Success
c.getName();

// Error, would crash
const g = c.getName;
console.log(g());
// Error: 'void' 타입의 `this` 컨텍스트는 'MyClass' 타입의 메서드 'this'에 할당할 수 없습니다.
```

이 방법은 `Arrow function` 방식과 반대되는 장단점을 가지고 있습니다.

- `JavaScript` 호출자가 인식하지 못한 채 클래스 메서드를 잘못 사용할 수 있습니다.
- 클래스 인스턴스당 함수가 하나씩 할당되는 것이 아닌 클래스 정의당 하나의 함수만 할당 됩니다.
- 기본 베서드 정의는 여전히 `super`를 통해 호출할 수 있습니다.

## `this` Types

특별한 타입인 `this`는 현재 클래스 타입을 동적으로 참조합니다. 이것을 어떻게 활용할지 살펴보자!

```tsx
class Box {
  contents: string = '';
  // (method) Box.set(value: string): this
  set(value: string) {
    this.contents = value;
    return this;
  }
}
```

여기서 `TypeScript`는 집합의 return 유형을 `Box`가 아닌 `this`로 추론 했습니다. `box`의 서브클래스를 만들어 봅시다.

```tsx
class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}

const a = new ClearableBox();
const b = a.set("hello"); // b type: ClearableBox
```

`parameter type annotation`에서도 사용할 수 있습니다.

```tsx
class Box {
  content: string = '';
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

이것은 `other: Box`과 작성하는 것과 다릅니다. 만약 파생 클래스가 있는 경우 메서드는 이제 **동일한 파생 클래스의 다른 인스턴스만 허용**합니다.

```tsx
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
/*
  Error: 'Box' 타입의 인수를 'DerivedBox' 타입의 매개변수 유형에 할당할 수 없습니다.
  'Box' 타입에는 'DerivedBox' 타입의 'otherContent' 속성이 필요합니다.
*/
```

### `this` 기본 타입 가드

`Class` 및 `Interface`의 메서드 반환 위치에서 `this is Type`을 사용할 수 있습니다. `Type Narrowing`과 혼합하면 대상 객체의 타입이 지정된 타입으로 좁혀 집니다.

```tsx
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content; // type: FileRep

} else if (fso.isDirectory()) {
  fso.children; // type: Directory

} else if (fso.isNetworked()) {
  fso.host; // type: Networked & FileSystemObject
}
```

이 기반의 타입 가드의 일반적인 사용 사례는 특정 필드에 대한 지연 유효성을 검사를 허용하는 것이다. 예를 들어 `hasValue`가 참인 것으로 확인되면 **상자 안에 있는 값에서 정의되지 않은 값을 제거**합니다.

```tsx
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}
const box = new Box();
box.value = "Gameboy";
box.value; // (property) Box<unknown>.value?: unknown

if (box.hasValue()) {
  box.value; // (property) value: unknown
}
```

## 매개변수 속성들

`TypeScript`는 생성자 매개변수를 동일한 이름과 값을 가진 클래스 속성으로 변환하는 특수 구문을 제공합니다. 이를 `parameter propertis`라고 부르며, 생성자 인수 앞에 `public`, `private`, `protected`, `readonly` 중 하나를 생성자 인수에 접두사로 지정하여 생성합니다. 결과 필드는 해당 수정자를 가져옵니다.

```tsx
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) { }
}
const a = new Params(1, 2, 3);
console.log(a.x); // (property) Params.x: number
console.log(a.z); // Error: 'z'는 'private 속성으로 클래스 'Params'에서만 접근이 가능하다.
```

## 클래스 표현식

클래스 표현식은 클래스 선언과 매우 유사하다. 유일한 차이점은 클래스 표현식에는 이름을 필요로 하지 않지만, 바인딩된 식별자를 통해 참조할 수 있습니다.

```tsx
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};

const m = new someClass("Hello, world"); // type: someClass<string>
```

## `abstract` 클래스들과 맴버들

TypeScript`의 클래스, 메서드 및 필드는 추상적일 수 있습니다.

추상 메서드나 추상 필드는 구현이 제공되지 않은 메서드나 필드입니다. 이러한 멤버는 직접 인스턴스화할 수 없는 추상 클래스 내부에 존재해야 합니다.

추상 클래스의 역할은 모든 추상 멤버를 구현하는 서브클래스의 기본 클래스 역할을 하는 것입니다. 클래스에 추상 멤버가 없는 경우 `구상(Concrete)` 클래스이다.

> 구상 클래스: new 키워드로 객체를 생성할 수 있는 클래스

```tsx
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base(); // Error: 아무 인스턴스 없이 추상 클래스를 생성할 수 없다.
```

`Base`는 추상적이기 대문에 `new`로 인스턴스화 할 수 없습니다. 방법은 파생 클래스를 만들고 추상 멤버를 구현해야합니다.

```tsx
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

기본 클래스의 추상 멤버를 구현하는 것을 잊어버리면 오류가 발생합니다.

```tsx
class Derived extends Base {
  /*
    만약 여기에 추상 멤버를 구현하지 않는다면!!!

    Error: 추상적이지 않은 클래스 'Derived'는 'Base' 클래스에서 상속된 추상 멤버 'getName'을 구현하지 않았습니다.
  */
}
```

## 추상 Construct Signatures

어떤 추상 클래스에서 파생된 클래스의 인스턴스를 생성하는 클래스 생성자 함수를 받아들이고 싶을 때가 있습니다.

```tsx
function greet(ctor: typeof Base) {
  const instance = new ctor();

  // Error: 추상 클래스에 아무런 인스턴스를 생성하지 않았습니다.
  instance.printName();
}
```

`TypeScript`는 추상 클래스를 인스턴스화 하려고 한다는 것을 올바르게 알려줍니다. 즉, `greet`의 정의를 고려할 때 이 코드를 작성하는 것은 맞는 방법이며, 결국 추상 클래스를 구성하게 됩니다.

```tsx
// Bad!
greet(Base);
```

`construct signature`을 가진 무언가를 받아들이는 함수를 작성하려고 합니다.

```tsx
function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base); 
/*
  Error: 'type Base' 타입의 인수는 'new () => Base' 타입의 매개 변수에 할당할 수 없습니다. 추상 생성자 타입을 추상적이지 않은 생성자 타입에 할당이 불가능합니다.
*/
```

`TypeScript`는 어떤 클래스 생성자 함수를 호출할 수 있는지 정확하게 알려줍니다. `Derived`는 구체적이기 때문에 호출할 수 있지만 `Base`는 호출할 수 없습니다.

## 클래스들 사이 관계

대부분의 경우 `TypeScript`의 클래스는 다른 타입들과 동일하게 구조적으로 비교됩니다.

예를 들어 아래 두 클래스는 동일하기 때문에 서로 대신 사용할 수 있습니다.

```tsx
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

const p: Point1 = new Point2(); // Success
```

마찬가지로 명시적인 상속이 없더라도 클래스 간의 하위 타입 관계를 존재합니다.

```tsx
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

const p: Person = new Employee(); // Success
```

간단해 보일 수 있지만 매우 낯설게 느껴지는 경우가 몇 가지 있습니다.

- 빈 클래스에는 멤버가 없습니다.
- 구조적 타입 시스템에서 멤버가 없는 타입은 일반적으로 다른 타입의 상위 타입이다.

따라서 빈 클래스를 작성하지마세요!

```tsx
class Empty {}

function fn(x: Empty) {
  // 이런거 작성하지 마세요!
}

// All OK!
fn(window);
fn({});
fn(fn);
```
