# this

`JavaScript` 에서 `this`는 현재 실행 컨텍스트에서 바인딩된 객체를 가리킵니다. `this`는 함수를 어떻게 호출했느냐에 따라 달라지며, 함수를 호출한 방식에 따라 `this`에 바인딩되는 값이 결정됩니다.

> `JavaScript`에서 함수는 호출될 때, 자동적으로 함수에 `arguments 객체`와 `this`가 암묵적으로 받는다.

일반적으로 함수 호출 방식에 따른 `this`의 값은 다음과 같습니다.

들어가기 전 기본적으로 아래 내용을 알고 가야한다. 전역 객체(Global Object)는 모든 객체의 유일한 최상위 객체를 의미한다.

```js
// Browser-side
this === window // true

// node 일 때: Server-side
this === global // true
```

## 1. 일반 함수 호출

전역 객체(window 또는 global - node.js)에 바인딩 됩니다.

```js
var ga = 'global variable';

console.log(ga); // global variable
console.log(window.ga); // global variable

function foo() {
  console.log('Hello');
}
window.foo(); // Hello
```

**`내부 함수`는 `일반 함수`, `메소드`, `콜백함수` 어디에서 선언되었든 관계없이 `this`는 전역객체에 바인딩한다.**

## 2. 메소드 호출

메소드가 속한 객체에 바인딩 됩니다.

```js
var obj1 = {
  name: 'Park',
  sayName: function() {
    console.log(this.name);
  }
}

var obj2 = {
  name: 'Jin'
}

obj2.sayName = obj1.sayName;

obj1.sayName(); // Park
obj2.sayName(); // Jin
```

## 3. 생성자 함수 호출

새로 생성되는 객체에 바인딩 됩니다.

```js
// 생성자 함수
function Person(name) {
  this.name = name;
}

var me = new Person('Park');
console.log(me); // Person {name: "Park"}
```

## 4. `call`, `apply`, `bind` 메소드 호출

`this`는 바인딩될 객체는 함수 호출 패턴에 의해 결정되므로 특정 객체에 명시적으로 바인딩하는 방법을 제공한다.(지정된 객체에 해당 메소드를 사용하여 바인딩)

