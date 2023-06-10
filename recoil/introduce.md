# Recoil 소개

Recoil은 직교(orthogonal)하지만, 본질적인 방향 그래프를 정의하고 React 트리에 붙인다. 상태 변화는 이 그래프 뿌리(atoms)로 부터 순수함수인(selectors)를 거쳐 컴포넌트로 흐른다.

- 공유 상태(shared state)도 React의 내부상태(local state)처럼 간단한 get/set 인터페이스로 사용할 수 있게 API를 제공한다.
- 동시성 모드를 지원하고 있고 새로운 React의 기능들과 호환 가능성도 가진다.
- 상태 정의는 점진적으로 분산 가능하여, 코드 분할이 가능하다.
- 상태를 사용하는 컴포넌트를 수정하지 않고도 상태를 파생된 데이터로 대체할 수 있다.
- 파생된 데이터를 사용하는 컴포넌트를 수정하지 않아도 파생된 데이터는 동기식과 비동기식 간에 이동할 수 있다.
- 우리는 탐색을 일급 개념으로 취급할 수 있고 심지어 링크에서 상태 전환을 인코딩할 수 있다.
- 전체 어플리케이션 상태를 하위 호환하는 방식으로 유지하기 쉬우므로, 유지된 상태는 애플리케이션 변경에도 살아남을 수 있다.

## 주요 개념

Atoms에서 Selectors를 거쳐 React 컴포넌트로 내려가는 데이터 플로우 그래프를 만들 수 있다. Atoms는 컴포넌트가 구독할 수 있는 상태의 단위이며, Selectors는 atoms 상태 값을 동기 또는 비동기 방식을 통해 변환한다.

### Atoms

Atoms는 상태의 단위로 업데이트 및 구독이 가능하다.

- Atom이 업데이트되면 각 구독된 컴포넌트는 리렌더링
- 런타임에서 생성 가능
- Atoms는 React의 기본에 제공하는 state를 대신 사용할 수 있음
- 동일한 Atom이 여러 컴포넌트에서 사용되는 경우 모든 컴포넌트는 상태를 공유

```tsx
const fontSizeState = atom({
  key: 'fontSizeState',
  default: 14,
});
```

**여기서 중요 포인트!**

Atoms는 디버깅 지속성 및 모든 atoms의 map을 볼 수 있는 특정 고급 API에 사용되는 고유한 key가 필요한데 유니크 값이어야한다.

만약 atom을 읽고 사용하려면 `useRecoilState` 훅을 사용하여 가져온다.

```tsx
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return (
    <button onClick={() => setFontSize((size) => size + 1)} style={{fontSize}}>
      Click to Enlarge
    </button>
  );
}
```

추가적으로 `useSetRecoilState` 훅을 사용하면 setter 함수에 접근하여 새로운 값을 넣을 수있다.

```tsx
const setFontSize = useSetRecoilState(fontSizeState);
```

## Selectors

Seletor는 atoms나 다른 selectors를 입력으로 받아들이는 순수 함수이다. 

- 상위의 atoms 또는 selectors가 업데이트되면 하위의 selector 함수도 다시 실행
- 컴포넌트들은 selectors를 atoms처럼 구독할 수 있으며 selectors가 변경되면 컴포넌트들도 리렌더링
- 어떤 컴포넌트가 자신을 필요로 하는지 어떤 상태에 의존하지는지를 추적하여 함수적인 접근방식을 매우 효율적으로 만듦
- 상태 기반으로 파생 데이터를 계산하는데 사용되며, 쓸모없는 상태의 보존을 방지

Selector는 아래와 같이 정의한다.

```tsx
const fontSizeLabelState = selector({
  key: 'fontSizeLabelState',
  get: ({get}) => {
    const fontSize = get(fontSizeState);
    const unit = 'px';

    return `${fontSize}${unit}`;
  },
});
```

여기서의 `get` 속석은 계산될 함수이며, `get`을 통해 atoms와 selectors와 접근할 수 있다.

`useRecoilValue` 훅을 사용하여 호출할 수 있다.

```tsx
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  const fontSizeLabel = useRecoilValue(fontSizeLabelState);

  return (
    <>
      <div>Current font size: ${fontSizeLabel}</div>

      <button onClick={setFontSize(fontSize + 1)} style={{fontSize}}>
        Click to Enlarge
      </button>
    </>
  );
}
```
