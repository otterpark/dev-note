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