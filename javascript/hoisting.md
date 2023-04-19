# 호이스팅(Hoisting)

`hoist`란 사전적으로 끌어올린다라는 뜻을 가지고 있으며, 주로 `var` 키워드를 사용 시 나타난다. JavaScript 실행 시 실행 컨텍스트가 생성되며 Runtime 시점 전 변수나 함수 선언문을 다른 구문보다 먼저 읽어 `최상위 Scope`로 위치 시키는 현상을 `Hoisting`이라고 한다.

> 변수 및 함수 선언이 물리적으로 작성한 코드의 상단으로 옮겨지는 건 아니다

**`Hoisting`은 총 세 가지 단계를 거쳐서 실행됨**

### 1. 선언 단계

모든(변수, 함수)선언문을 찾으면 변수 객체(Variable Object)에 변수를 등록하며 스코프가 참조하는 대상이됨

### 2. 초기화 단계

모든 변수 객체(Variable Objecct)에 메모리를 할당함과 동시에 모든 변수에 `undefined`로 초기화

### 3. 할당 단계

`undefined`로 초기화된 변수에 실제값을 할당

## 변수 호이스팅(Valiable Hoisting)

var 키워드로 선언된 변수는 위의 단계에서 선언 단계와 초기화 단계가 한번에 이루어짐

아래 예시로 살펴보자.

```javascript
console.log(score); // undefined
var score = 1;
console.log(score); // 1
```

위의 코드를 보면 score는 선언되지도 않았는데 `undefined`라는 값이 들어가 있는데 이러한 현상은 변수 호이스팅 때문

## 함수 호이스팅(Function Hoisting)

모든 함수는 위의 단계에서 선언 단계, 초기화 단계, 할당 단계가 한번에 이루어짐

아래 예시로 살펴보자.

 ```javascript
console.log(printHello()); // Hello
function printHello() {
    console.log("Hello");
}
 ```

위의 코드를 보면 `console.log(printHello());`는 선언 이전에 Hello 라는 결과 값을 출력하는데 이러한 현상은 함수 호이스팅 때문
