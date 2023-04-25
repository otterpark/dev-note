# Think in React

React는 자신이 보는 디자인 방식과 자신이 만드는 앱에 대한 생각들을 바꿀 수 있습니다. 생각의 흐름은 아래와 같이 흘러갑니다.

1. React로 사용자 인터페이스를 빌드할 때 컴포넌트라고 하는 조각으로 분해합니다.
2. 각 컴포넌트에 대해 서로 다른 시각적 상태를 설명합니다.
3. 컴포넌트를 서로 연결해 데이터가 흐르도록 합니다.

이러한 생각들을 React로 검색 가능한 제품 데이터 테이블로 구축하는 예로 설명합니다.

## 목업으로부터 시작하기

이미 JSON API와 디자이너의 목업이 있다고 가정합니다.

JSON API는 아래과 같은 데이터를 반환합니다.

```jsx
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

목업은 아래와 같습니다.

![React-mockup-ui](https://react.dev/images/docs/s_thinking-in-react_ui.png)

이미 JSON API와 디자이너의 목업이 있다고 가정해 보겠습니다.

React에서 UI를 구현하려면 일반적으로 동일한 5단계를 따릅니다.

## 1단계: UI 컴포넌트 계층으로 나누기

먼저 목업의 모든 컴포넌트와 하위 컴포넌트 주위에 상자를 그리고, 각 이름을 지정합니다.

자신의 배경 지식에 따라 여러 가지 방식으로 컴포넌트를 분할 할 수 있습니다.

- `프로그래밍` - 새 함수나 객체를 어떻게 생성할지 여부를 결정할 때와 동일한 기법을 사용합니다. `단일 책임 원칙`으로 컴포넌트는 이상적으로 한 가지 일만 수행해야 합니다. 만약 컴포넌트가 늘어나면 더 작은 하위 컴포넌트로 분해해야 합니다.
- `CSS` - 클래스 선택자를 만들 때, 무엇을 위해 만들 것인지 생각해야 합니다.
- `디자인` - 디자인의 레이어를 어떻게 구상할 것인지 고려해야 합니다. (컴포넌트는 조금 덜 세분화되어 있다.)

JSON이 잘 구조화되어 있다면, UI의 컴포넌트 구조에 자연스럽게 매핑되는 것을 발견할 수 있습니다. UI와 데이터 모델의 정보 아키텍처가 동일한 경우가 많기 때문입니다.(즉, 동일한 `Shape`이다.) UI를 컴포넌트로 분히하고, 각 컴포넌트가 데이터 모델의 한 부분과 일치하도록 합니다.

아래에는 5가지 컴포넌트 구성 요소가 있습니다.

![React-mockup-component](https://react.dev/images/docs/s_thinking-in-react_ui_outline.png)

1. `FilterableProductTable (grey)` - 전체 앱이 포함됩니다.
2. `SearchBar (blue)` - 사용자의 입력을 받습니다.
3. `ProductTable (lavender)` - 사용자 입력에 따라 목록을 표시하고 필터링합니다.
4. `ProductCategoryRow (green)` - 각 카테고리에 대한 제목을 표시합니다.
5. `ProductRow (yellow)` - 각 제품에 대한 행을 표시합니다.

`ProductTable (lavender)`를 보면 `ProductCategoryRow (green)`, `ProductRow (yellow)`과 합쳐진 자체 구성이 아닌 것을 확인할 수 있습니다. 이 부분 같은 경우에는 선호도 문제이며, 어떻게 구성을 하든 사용할 수 있습니다. 이 예제에서는 제품 테이블의 목록 안에 표시되므로 제품 테이블의 일부로 표시하였습니다. 그러나, 테이블 헤더가 복잡해지면(예: 정렬을 추가한다라는 가정) 자체 `ProductTable (lavender)` 컴포넌트로 이동할 수 있습니다.

목업에서 컴포넌트를 식별했으므로 이제 계층 구조로 정렬합니다. 목업의 다른 컴포넌트안에 표시되는 컴포넌트는 계층 구조에서 하위로 표시되어야 합니다.

- `FilterableProductTable (grey)`
  - `ProductSearchBar (blue)`
  - `ProductTable (lavender)`
    - `ProductCategoryRow (green)`
    - `ProductRow (yellow)`

## 2. React에서 정적버전 만들기

컴포넌트 계층 구조가 완성되었으니 앱으로 구현해보자. 가장 간단한 접근 방식으로 인터렉티브한 것들은 추가하지 않고 데이터 모델에서 UI를 렌더링하는 버전을 만들 것이다. 정적버전을 먼저 빌드하고 나중에 인터렉티브를 추가하는 경우가 조금 더 쉬울 수 있다.

- `정적버전을 만든다면` - 많은 생각은 필요없고 많은 타이핑이 필요하다.
- `인터렉티브를 추가한다면` - 많은 타이핑이 필요없고 많은 생각이 필요하다.

데이터 모델을 렌더링 하는 정적버전의 앱을 만들려면 다른 컴포넌트를 재 사용하고 프로퍼티를 사용하여 데이터를 전달하는 컴포넌트를 만들어야 합니다. 프로퍼티는 부모에서 자식으로 전달하는 방법(`단방향`)입니다.

데이터 모델을 렌더링하는 정적 버전의 앱을 빌드하려면 다른 컴포넌트를 재사용하고 프로퍼티를 사용하여 데이터를 전달하는 컴포넌트를 빌드해야 합니다. 프로퍼티는 부모에서 자식으로 데이터를 전달하는 방법입니다.
> `State(상태)`의 개념에 익숙하다면, 정적버전을 만들 때 `State(상태)`를 전혀 사용하지 마세요. `State(상태)`는 상호작용, 즉 시간이 지남에 따라 변경되는 데이터에만 사용됩니다. 이 앱은 정적버전이므로 필요하지 않습니다.

계층 구조에서 상위 컴포넌트로부터 만드는 `하향식(부모 -> 자식)` 빌드나 하위 컴포넌트부터 작업하는 `상향식(자식 -> 부모)` 빌드 중 하나를 선택할 수 있습니다. **`간단한 예제`에는 하향식으로 작성하는 것이 더 쉽고, `대규모 프로젝트`에서는 상향식으로 작성하는 것이 더 쉽습니다**.

```jsx
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

컴포넌트를 만들고 나면 데이터 모델을 렌더링하는 재 사용 가능한 컴포넌트 라이브러리를 가지게 됩니다. 이 앱은 Static 앱이므로 컴포넌트는 JSX만 반환합니다. 계층 구조의 맨 위에 있는 `FilterableProductTable` 컴포넌트는 데이터 모델을 `Props`로 사용합니다. 데이터가 최상위 컴포넌트에서 트리 하단에 있는 컴포넌트로 흘러내리기 때문에 이를 React의 특징인 `단방향 데이터 흐름`이라고 합니다.

> 다음 단계에서 자세히 다루기 위해, 여기에서는 절대 `state`를 사용하지 마세요.

## 3. 최소한의 완전한 UI 상태 표현 찾기

UI를 인터렉티브하게 만드려면 사용자가 기본 데이터 모델을 변경할 수 있도록 해야합니다. 이를 위해 `상태(state)`를 사용합니다.

`상태(state)`는 앱이 기억해야 하는 최소한의 변경 데이터 집합이라고 생각하면 됩니다. 상태를 구조화할 때 가장 중요한 원칙은 `반복하지 않는 것`입니다. 애플리케이션에 필요한 최소한의 상태 표현을 파악하고 그 외에 모든 것은 `on-demand` 방식으로 계산해야 합니다. 예를 들어 쇼핑 목록을 작성하는 경우 항목을 `상태(state)` 배열로 저장할 수 있습니다. 만약 목록에 있는 항목의 개수도 표시한다고 하면, 항목의 개수를 다른 `상태(state)` 값으로 저장하지 않고 배열의 길이를 반환하면 됩니다.

자 그럼 예제의 모든 데이터 조각들을 생각해보자!

1. 제품 목록
2. 사용자가 입력한 검색 텍스트
3. 체크박스의 값
4. 필터링된 제품 목록

다음 중 어떤 것이 상태일까요? 상태가 아닌 것들을 체크해 봅시다.

- 시간이 지나도 변하지 않나요? 그렇다면 상태가 아닙니다.
- 부모로부터 `props`를 통해 전달되나요? 그렇다면 상태가 아닙니다.
- 컴포넌트를 기존 `state` 또는 `props`를 기반으로 계산할 수 있나요? 그렇다면 상태가 아닙니다.

위에서 남은 것은 아마도 우리는 `상태(state)`라고 생각할 수 있습니다.

- 원래 제품 목록은 `props`로 전달되었으므로 `state`가 아닙니다.
- 검색 텍스트는 시간이 지남에 따라 변경되고 아무것도 계산할 수 없기 때문에 `state`입니다.
- 체크박스의 값은 시간이 지남에 따라 변경되고 아무것도 계산할 수 없기 때문에 `state`입니다.
- 필터링된 제품 목록은 원래 제품 목록을 가져와서 검색 텍스트 및 확인하는 값에 따라 필터링하여 계산할 수 있으므로 `state`가 아닙니다.

즉, 두 번째(검색 텍스트), 세 번째(체크박스의 값) 값만 상태입니다.

### `Props` vs `State`

React에는 두 가지 유형의 `props`와 `state` 데이터가 있습니다

- `props`는 함수에 전달하는 인자와 같습니다. 부모 컴포넌트가 자식 컴포넌트에 데이터를 전달하고 그 모양을 사용자 정의할 수 있게 해줍니다.
ex) form은 버튼에 색상 prop을 전달할 수 있습니다.

- `state`는 컴포넌트의 메모리와 같습니다. 컴포넌트가 일부 정보를 추적하고 상호작용에 반응하여 변경할 수 있게 해줍니다. 
ex) button은 isHovered 상태를 추적할 수 있습니다.

`props`와 `state`는 서로 다르지만 함께 작동합니다. 부모 컴포넌트는 종종 일부 정보를 state에 보관하고(변경할 수 있도록), 이를 자식 컴포넌트에 props로 전달합니다.

## 4. state를 정의할 위치 파악하기

앱의 최소 `state data`를 식별한 후 이 `state` 변경을 담당하는 컴포넌트, 즉 상태를 소유하는 컴포넌트를 식별해야 합니다. React는 단방향 데이터 흐름을 사용하며, 부모 컴포넌트에서 자식 컴포넌트로 컴포넌트의 계층 구조에 따라 데이터를 전달합니다. 어떤 컴포넌트가 어떤 상태를 소유해야 하는지 명확하지 않을 수 있습니다. 이 개념을 처음 접하면 어려울 수도 있지만 다음 단계를 따라하면 충분히 이해할 수 있습니다.

앱의 각각의 `state`에 대해:

1. 해당 `state`를 기반으로 무언가를 렌더링하는 모든 컴포넌트를 식별합니다.
2. 가장 가까운 공통 상위 컴포넌트, 즉 계층 구조에서 모든 컴포넌트 위에 있는 컴포넌트를 찾습니다.
3. `state`가 어디에 위치할지 결정합니다.

    - `state`를 공통의 부모 컴포넌트에 직접 넣을 수 있다.
    - `state`를 공통의 부모 위에 있는 컴포넌트에 넣을 수 있다.
    - `state`를 소유하기 위해 적합한 컴포넌트를 찾을 수 없을 경우, `state`를 소유하기 위한 컴포넌트를 새로 만들어 공통 부모 컴포넌트 위의 계층 구조 어딘가에 추가할 수 있다.

이전 단계에서는 이 앱에서 두 가지 상태(검색 입력 텍스트 값, 체크박스의 값)를 발견했습니다. 이 예제에서는 항상 함께 표시되므로 같은 위치에 `state`를 배치하는 것이 좋습니다.

1. **`state`를 사용하는 컴퍼넌트를 식별합니다**

    - `ProductTable`은 해당 `state(검색 입력 텍스트 값, 체크박스의 값)`을 기준으로 제품 목록을 필터링해야 합니다.
    - `SearchBar`는 해당 `state(검색 입력 텍스트 값, 체크박스의 값)`를 표시해야 합니다.

2. **공통 상위 컴포넌트 찾기**: 두 컴포넌트가 공유하는 첫 번째 상위 컴포넌트는 `FilterableProductTable` 이다.

3. **상태가 어디에 있는지 결정**: 필터 텍스트와 체크된 상태 값은 `FilterableProductTable`에 유지합니다.

따라서 state 값은 `FilterableProductTable`에 저장합니다.

`useState()` `Hook`으로 컴포넌트에 `state`를 추가합니다. `Hook`은 React에 "훅"할 수 있는 특수 함수입니다. `FilterableProductTable`의 상단에 `state` 변수 두 개를 추가하고 초기 상태를 지정합니다.

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);
```

그런 다음 `filterText` 및 `inStockOnly`를 `ProductTable` 및 `SearchBar`에 `props`로 전달합니다.

```jsx
<div>
  <SearchBar 
    filterText={filterText} 
    inStockOnly={inStockOnly} />
  <ProductTable 
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

이제 앱이 어떻게 동작하는지 확인할 수 있습니다. 아래 코드에서 `filterText` 초기값을 `useState('')`에서 `useState('과일')`로 편집합니다. 검색 입력 텍스트와 테이블 업데이트가 모두 표시됩니다.

```jsx
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} />
      <ProductTable 
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

위 예제를 입력하면 에러가 발생합니다. 아직 폼 작성을 작동할 수 있게 구현을 하지 않은 상태라 에러가 뜹니다.

`ProductTable`과 `SearchBar`는 `filterText` 및 `inStockOnly` `props`을 읽어 테이블, 입력 및 체크박스를 렌더링합니다. 예를 들어 `SearchBar`가 입력 값을 채우는 방식은 다음과 같습니다.

```jsx
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
```

아직 입력과 같은 사용자 작업에 응답하는 코드를 추가하지 않았습니다. 다음 단계에서 진행하며, 마지막 step이다.

## 5. 역 데이터 흐름 추가

현재 앱은 `props`와 `state`가 계층 구조 아래로 흐르면서 올바르게 렌더링이 됩니다. 하지만 사용자 입력에 따라 상태를 변경하려면 계층 구조의 깊은 곳에 있는 form 컴포넌트가 `FilterableProductTable`의 상태를 업데이트해야 하는 등 데이터가 다른 방향으로 흐르도록 지원해야 합니다.

React는 이 데이터 흐름을 명시적으로 만들지만, 양방향 데이터 바인딩보다 조금 더 많은 타이핑이 필요합니다. 위의 예시에서 입력하거나 확인란을 선택하려고 하면 React가 입력을 무시하는 것을 볼 수 있습니다. 이는 React에서 의도한 것입니다. `<input value={filterText} />`를 작성함으로써, 입력 값 `prop`이 항상 `FilterableProductTable`에서 전달된 `filterText` `state`와 같도록 설정했습니다. `filterText` `state`가 설정되지 않으므로 `input`은 절대 변경되지 않습니다.

사용자가 form input을 변경할 때마다 해당 변경 사항이 반영되어 `state`가 업데이트 되도록 만들고자하면 어떻게 해야할까? `state`는 `FilterableProductTable
`이 소유하고 있으므로 `FilterableProductTable`만 `setFilterText`과`setInStockOnly`를 호출할 수 있습니다. `SearchBar`가 `FilterableProductTable`의 상태를 업데이트하도록 하려면 `setFilterText`과`setInStockOnly`를 전달해야 합니다.

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
...
```

`SearchBar`안에 `onChange` 이벤트 헨들러를 추가하고 이 핸들러에서 상위 상태를 설정합니다.

```jsx
<input 
  type="text" 
  value={filterText} 
  placeholder="Search..." 
  onChange={(e) => onFilterTextChange(e.target.value)} />
```

이렇게 적용하면 전체적인 어플리케이션이 완성이 됩니다.

```jsx
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Search..." 
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

## 마지막 정리

### 첫 번째, UI 컴포넌트 계층으로 나누기

새 함수나 객체들이 어떻게 생성할지 생각 후 컴포넌트는 이상적으로 한 가지 일만 수행할 수 있게 컴포넌트를 계층으로 나눈다.

### 두 번째, 정적 버전으로 만들기

인터렉티브한 것들을 제외하고 데이터 모델을 렌더링하는 정적버전을 만들기 위해 다른 컴포넌트를 재 사용하고 프로퍼티를 사용하여 데이터를 전달하는 컴포넌트를 만들어야 합니다.(부모 -> 자식으로 전달하는 방법 - `단방향`)

#### `하향식(부모 -> 자식)`

  `간단한 예제`에는 하향식으로 작성하는 것이 쉽다.

#### `상향식(자식 -> 부모)`

  `대규모 프로젝트`에서는 상향식으로 작성하는 것이 쉽다.

### 세 번째, 최소한의 완전한 UI 상태 표현 찾기

앱이 기억해야 하는 최소한의 변경 데이터 집합인 `state`를 사용하여 사용자가 기본 데이터 모델을 변경할 수 있게 해준다.

- 시간이 지나도 변하지 않나요? 그렇다면 상태가 아닙니다.
- 부모로부터 `props`를 통해 전달되나요? 그렇다면 상태가 아닙니다.
- 컴포넌트를 기존 `state` 또는 `props`를 기반으로 계산할 수 있나요? 그렇다면 상태가 아닙니다.

### 네 번째, `state`를 정의할 위치 파악하기

`state` 변경을 담당하는 컴포넌트, 즉 `state`를 소유하는 컴포넌트를 식별해야합니다.

- `state`를 공통의 부모 컴포넌트에 직접 넣을 수 있다.
- `state`를 공통의 부모 위에 있는 컴포넌트에 넣을 수 있다.
- `state`를 소유하기 위해 적합한 컴포넌트를 찾을 수 없을 경우, state를 소유하기 위한 컴포넌트를 새로 만들어 공통 부모 컴포넌트 위의 계층 구조 어딘가에 추가할 수 있다.

### 다섯 번째, 역 데이터 흐름 추가

React는 단방향 데이터 흐름을 가지고 있기에 `props`와 `state`가 계층 구조 아래로 흐르면서 올바르게 렌더링됩니다. 계층 구조안에 데이터가 역으로 흐를 수 있게 무언가를 만들어 줘야합니다.

`state`를 관리하고 있는 컴포넌트에서 함수를 만들어 하위 컴포넌트에서 사용할 수 있게 `props`로 전달하여 하위 컴포넌트에서는 이벤트 핸들러를 통해 사용할 수 있게 해준다.

