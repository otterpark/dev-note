# Mapped Types - 기존 유형의 각 속성을 매핑하여 유형 만들기

들어가기 전, 매핑된 유형(`Mapped Types`)이 어떤건지 알아보자.

매핑된 유형은 기존 유형에서 유형 정보를 매핑하여 새로운 유형을 만드는 프로세스입니다. 즉 다시 말해 `기존에 정의되어 있는 타입을 새로운 타입으로 변환해 주는 문법`입니다.

JavaScript로 map에 대해 `Array.prototype.map()`로 설명하면 아래와 같습니다.

```js
const arr = [{ id: 1, title: 'tomato'}, { id: 2, title: 'apple'}, { id: 3, title: 'orange'}];
const result = arr.map(function(item) {
  return item.title;
});
console.log(result); // ['tomato', 'apple', 'result'];
```

위 예제를 보면 상수 `arr`를 선언 후 `Array.prototype.map()` 함수를 돌려 return 값을 상수 `result`에 넣어서 `console.log()`에 찍는 코드이다. 위 과정은 배열에 각 요소를 순회하여 객체(id, title)에서 문자열(title)로 변환하였습니다. 이 과정들이 매핑을 하는 과정과 동일하다.

## Mapping Modifiers

매핑 중 추가로 할 수 있는 수정자로는 `readonly`와 `?(optional)`가 있습니다.

- `-` 또는 `+` : 둘을 `접두사`라고 부르며, 가변성에 영향을 미친다. 접두사를 사용하여 수정자를 추가하거나 제거할 수 있습니다.
- `?(optional)` : 선택성에 영향을 미친다.

> 접두사(`-`, `+`)를 추가하지 않으면 `+`로 간주합니다.

`-` 사용 시 해당 수정자를 제거해야한다.

예제로 예를 들어보자

```tsx
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
  // readonly에 접두사인 '-'가 붙었으므로, 유형 속성에서 'readonly'를 제거한다.
};
```

```tsx
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
  // '?(optional)'에 접두사인 '-'가 붙었으므로, 유형 속성에서 'optional' 속성을 제거한다.
};
```

`매핑된 유형 수정자(Mapping Modifiers)`는 매핑된 유형에 유연성을 추가하여 속성을 필수 및 쓰기 기능으로 만들어 줍니다.

## `as`를 통한 키 재매핑

TypeScript 4.1 이상에서 매핑된 타입에 `as`절을 사용해서 매핑된 키의 이름을 바꿀 수 있습니다.

- `Capitalize` - 유틸리티 type으로 key를 camelCase로 바꿔준다.
- `Exclude` - 유틸리티 type으로 key에서 제외 시킨다.

간단하게 User에 대한 예제로 한번 자세히 이해해보자.

```tsx
type User = {
    name: string;
    email: string;
    password: string;
};

type Permissions = 'read' | 'write';

type UserPermissions = {
    [Property in keyof User as `${Permissions}${Capitalize<Property>}`]: boolean;
};
```

**전체적으로 정리해보면**

- User의 각 속성에 대해 키를 가져온다.(name, age, email, password)
- 키를 카멜 케이스로 유지시킨다.(`Capitalize`)
- 템플릿 리터럴로 키 이름을 수정해 준다.(name -> readName);
- `Permissions`가`Union type`이니 각각 read와 write로 매핑하며 추가한다.

그럼 아래와 같은 결과가 나온다.

```tsx
type UserPermissions = {
    readName: boolean;
    writeName: boolean;
    readEmail: boolean;
    writeEmail: boolean;
    readPassword: boolean;
    writePassword: boolean;
};
```

## Further Exploration(추가 탐색)

조건부 타입을 사용한 매핑된 타입에 대해서 알아보자. 아래 예제에서는 `pii` 프로퍼티에 따라 `true` 또는 `false`를 반환하는 예제이다.

```tsx
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};

type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;

/*
  type ObjectsNeedingGDPRDeletion = {
    id: false;
    name: true;
  }
*/
```
