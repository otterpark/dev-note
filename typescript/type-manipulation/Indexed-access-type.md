# Indexed Access Types - Type[’a’] 구문을 사용한 유형의 하위 집합 엑세스

## `typeof` 타입 연산자

JavaScript 에서는 표현식 컨텍스트에서 사용할 수 있는 `typeof` 연산자가 있으며, TypeScript 에서는 타입 컨텍스트에서 변수나 프로퍼티 타입을 추론할 수 있는 `typeof` 연산자가 있습니다.

```tsx
// JavaScript
console.log(typeof "Hello world"); // string

// TypeScript
let s = "hello";
let n: typeof s; // type: string
```

보통 다른 타입 연산자와 함께 `typeof`를 사용하여 많은 패턴을 편리하게 표현할 수 있습니다.

```tsx
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<f> // Error: f는 값을 가르키지만 유형으로 쓰이고 있다
type P = ReturnType<typeof f>; // type: { x: number; y: number; }
```

### 제한

TypeScript는 `typeof`를 사용할 수 있는 표현식의 종류를 제한합니다.

변수 이름 혹은 프로퍼티에만 `typeof`를 사용할 수 있습니다.
