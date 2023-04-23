# The Basics

TypeScript는 타입 시스템이 알아서 올바른 타입을 어떻게든 잘 알아낼 수 있다면 타입 표기를 굳이 적지 않는 것이 가장 좋습니다.

## 1. 다운레벨링

템플릿 문자열은 ECMAScript 2015 라고 불리는 버전의 ECMAScript에서 등장한 기능입니다. TypeScript는 새 버전의 ECMAScript의 코드를 ECMAScript 3 또는 ECMAScript 5와 같은 보다 예전 버전의 것들로 다시 작성해 줍니다. 새로운 또는 “상위” 버전의 ECMAScript를 예전의 또는 “하위” 버전의 것으로 바꾸는 과정을 `다운레벨링` **이라 부르기도 합니다.

```jsx
tsc --target es2015 input.ts
```

- 타겟 기본 값은 ES3 이지만, 대다수의 브라우저들은 `ES2015`를 지원하고 있으므로 구버전 브라우저가 없어진 지금은 타겟을 `ES2015` 또는 그 이상으로 설정하면 된다.

## 2. 엄격도

기존은 개발 방향을 방해하지 않는 방향을 목표로 관대한 기준으로 이루어져 있는데, 잠재적으로 null/undefined 값에 대한 검사는 이루어 지지 않는다.

하지만 대다수의 사람들은 TypeScript가 최대한 타입 검사를 해주기를 선호하므로 최대한 `strictNullChecks` 을 옵션으로 설정하여 사용하는 것을 권장한다.

**설정 방법**

- CLI에서 `--strict` 플래그를 설정
- `tsconfing.json`에 `"strict": true` 를 추가한다.

### noImplicitAny

TypeScript는 값의 타입을 추론하지 않을 시 가장 관대한 타입인 `any`로 처리

### strictNullChecks

`null`과 `undefined`를 보다 명시적으로 처리
