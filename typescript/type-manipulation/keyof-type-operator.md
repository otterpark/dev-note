# Keyof Type Operator - 키 타입 연산자를 사용하여 새 타입 만들기

## `Keyof` 타입 연산자

`keyof` 연산자는 객체 타입에서 객체의 키값들을 `숫자`나 `문자열 리터럴 유니온` 을 생성합니다.

```tsx
type Point = { x: number; y: number };
type P = keyof Point; // type: 'x' | 'y' 와 동일한 타입
```

만약 타입이 `string`이나 `number` 인덱스를 시그니쳐를 가지고 있다면, `keyof`는 해당 타입을 리턴합니다.

```tsx
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish; // type: number
 
type Mapish = { [k: string]: boolean };
type M = keyof Mapish; // type:  객체 키는 항상 문자열(toString())을 강제하기에 string | number | symbol
```
