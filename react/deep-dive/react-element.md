# React Element

## Managing the Instances

React에 처음 접한 사람이면 class형 컴포넌트와 instance를 접한적이 있다. 예를 들면 아래와 같이 클래스를 생성하여 `Button` 컴포넌트를 선언할 수 있습니다. 앱이 실행 중일 때 화면에 이 컴포넌트의 인스턴스가 여러개 표시될 수 있으며, 각 인스턴스에는 고유한 프로퍼티와 로컬 상태가 있습니다. 그럼 왜 `elements`를 도입할까?

`기존 UI model(OOP)`에서는 child component instance를 만들고 지우는 것은 개발자의 책임이다. 만약 `Form` 컴포넌트가 `Button` 컴포넌트를 render 하길 원한다면, `Form` 컴포넌트는 `Button` 컴포넌트를 instance를 만들고 새로운 정보 (new props, state)에 대응하여 수동적으로 최신 상태를 유지해줘야 한다.

### 기존 UI model(OOP)

```tsx
class Form extends TraditionalObjectOrientedView {
  render() {
    // Read some data passed to the view
    const { isSubmitted, buttonText } = this.attrs;

    if (!isSubmitted && !this.button) {
      // Form is not yet submitted. Create the button!
      this.button = new Button({
        children: buttonText,
        color: 'blue'
      });
      this.el.appendChild(this.button.el);
    }

    if (this.button) {
      // The button is visible. Update its text!
      this.button.attrs.children = buttonText;
      this.button.render();
    }

    if (isSubmitted && this.button) {
      // Form was submitted. Destroy the button!
      this.el.removeChild(this.button.el);
      this.button.destroy();
    }

    if (isSubmitted && !this.message) {
      // Form was submitted. Show the success message!
      this.message = new Message({ text: 'Success!' });
      this.el.appendChild(this.message.el);
    }
  }
}
```

- Button class component -> instance(각자 local state, own properties를 가짐)
- Form 이 render() 에서 new Button() 호출 instance 를 appendChild 호출하여 render
- isSubmitted 와 this.button, this.message 체크해서 각 children 을 create, update, destroy

### 한계점

각 component 는 DOM node 주소와, 자식 component 의 instance 주소를 보관해야하며, 시의 적절하게 각각을 create, update, destroy 해야 함

관리할 컴포넌트 늘어나면 component 코드의 복잡도 증가
자식 component 에게 부모 component 가 직접 접근하므로 decouple 하기 어려움

그럼 React는 어떻게 다른가?

## Elements describe the TREE

위 문제를 해결하기 위해 React에서는 `elements`를 사용한다.

- 컴포넌트 instance 혹은 DOM node와 그들의 속성들에 대한 정보를 가지고 있는 `(describing) plain object`
- `plain object`는 컴포넌트 타입 (예를 들면 Button 또는 Form), 해당 컴포넌트 인스턴스나 DOM node의 속성 (예: color 등)과 해당 컴포넌트 인스턴스나 DOM node의 자식 `elements`에 대한 정보만 가지고 있다.
- `element`는 실제 인스턴스는 아니다. 대신, `element`는 개발자가 화면에 표현하고 싶은 것을 **React에 전달하는 방식**이다.(메소드 실행 불가)
- `element`는 `type: (string | ReactClass)`와 `props: Object`라는 두개의 fields를 가진 불변성의 설명을 담고있는 객체이다.

### React DOM elements(type: string)

```tsx
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}

<button class='button button-blue'>
  <b>
    OK!
  </b>
</button>
```

여기서 제일 중요한 점은 child와 parents elements들은 하나의 묘사일 뿐 실제 instance는 아니다. 실제 child와 parent elements들을 만들 때 화면에 나타나는 것들과 무관하다는 이야기이다.

- `props` - 태그 속성
- 객체만 있으므로 DOM 요소보다 가볍다.(객체가 아니다)
- 구문 분석할 필요가 없고 파싱 과정이 없기 때문에 쉽게 순회하며 조작이 가능하다.(객체이기에 DOM element보다 훨씬 가볍다)

### React Component Elements(type: Function or Class)

```tsx
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

element 에 의해 기존의 DOM node 와 React component 를 mixed, nested 한 구조가 가능해졌으며, 그 결과 component 끼리 ducoupled가 된다

> 컴포넌트끼리 is-a, has-a 관계로 표현이 가능해짐

```tsx
const DeleteAccount = () => ({
  type: 'div',
  props: {
    children: [{
      type: 'p',
      props: {
        children: 'Are you sure?'
      }
    }, {
      type: DangerButton,
      props: {
        children: 'Yep'
      }
    }, {
      type: Button,
      props: {
        color: 'blue',
        children: 'Cancel'
      }
   }]
});
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color='blue'>Cancel</Button>
  </div>
);
```

### `elements`를 도입하면서 어떻게 `기존 UI model(OOP)`이 가지고 있던 한계를 극복했는가?

element는 다른 element를 children을 가질 수 있게 구조가 되어있어서 이 말은 즉, tree 구조가 된다는 말인데 `컴포넌트 element`와 `실제 DOM에 있는 element`가 **같은 위계에서 관리**가 될 수 있다.

## Components Encapsulate Element Trees

React가 함수 혹은 클래스 type의 element를 마주치면, 해당 component에 주어진 해당 props에 맞게 어떤 element가 렌더링 되는지 묻는다.

예를 들면 아래와 같이 element를 리엑트가 마주하면

```tsx
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

React는 `Button(React Component element)`에게 무엇을 렌더링하는지 묻는다. 그럼 `Button`은 다음 element를 반환한다.

```tsx
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

1. type 값을 보고, 해당 component 함수에게 element 를 return 받는다
2. return 받은 element 의 type 값이 tag name 인 element(DOM element)를 만날때까지 1번으로 돌아감

React는 페이지의 모든 컴포넌트와 일치하는 DOM tag element를 알아낼때까지 위의 과정을 반복한다.

결국 모든 elements가 가진 DOM elements를 알게 되며, React가 해당 DOM elements 들을 적절한 때에 create, update, destroy를 해준다.

따라서 기존의 Form UI modeling을 아래와 같이 간단하게 구현할 수 있다.

```tsx
const Form = ({ isSubmitted, buttonText }) => {
  if (isSubmitted) {
    // Form submitted! Return a message element.
    return {
      type: Message,
      props: {
        text: 'Success!'
      }
    };
  }
  // Form is still visible! Return a button element.
  return {
    type: Button,
    props: {
      children: buttonText,
      color: 'blue'
    }
  };
};
```

> React functional component 는 Props 를 받아서 element 를 return 하는 javascript function

React element를 활용하여, **기존에 DOM structure 을 모두 활용하지 않고, 필요한 정보만 독립적으로 UI 를 관리**할 수 있게 됨

이 말은 즉, Component 가 return 한 element 를 활용해서 DOM Trees 를 `encapsulate`

## Components Can Be Classes or Functions

이 부분은 생략(Functional component에 hooks 기능이 추가 되어서 Class component 와 동등해짐)

## Top-Down Reconciliation

```tsx
ReactDOM.render({
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}, document.getElementById('root'));
```

위 코드를 실행하면 어떤 일이 일어날까?

React는 `Form` 컴포넌트에 주어진 `props`에 대해 어떤 element tree를 반환할지 물어볼 것인지 요청

`Form` -> `Button` -> DOM node element(button) 순서로 return element 진행

```tsx
// 개발자가 React에 주어진 props와 함께 Form 컴포넌트를 그려달라고 함
{
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}

// Form 컴포넌트가 리액트에 return한 값
{
  type: Button,
  props: {
    children: 'OK!',
    color: 'blue'
  }
}

// Button 컴포넌트가 return한 값. 하위에 더 이상의 children이 없는 것으로 확인되어 React는 더 물어볼 것이 없음
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

이러한 과정을 `Top-Down Reconciliation` 이라고 부른다. 

- ReactDom.render(), setState() 호출 시 React는 reconcilation을 호출하며, reconcilation 이 끝나면 최종 DOM tree가 어떻게 구성되는지 알게 되고 react-dom, react-native 같은 renderer가 DOM nodes를 업데이트 하기 위해 필요한 최소한의 변화들을 적용한다.
- 점진적 변화 덕분에 쉽게 최적화가 가능하며, props 가 immutable 이면 이 정제과정(refining process)과 tree의 특정 부분을 비교하는 것을 건너뛰라고 이야기하여, change 계산이 더 빨라짐
  > React와 immutable은 함께 시너지를 발휘하고 최소한의 노력으로 큰 최적화를 제공할 수 있다.

## Summary

- element는 DOM Tree 생성에 필요한 정보를 담은 object이며, 종류는 React DOM node element, React Component element 이다. (한번 element가 만들어지면 **절대 불변**이다)
- element는 property로 다른 element를 가질 수 있다 -> DOM node 와 React component 를 mixed, nested 가능하게 함
- component는 props -> element하는 function
- parent component가 return한 element의 type이 child component 이름일 때, props는 해당 child component의 input(props)가 된다 -> props는 단방향 흐름
- instance는 class component 에서의 this
- instance를 직접 생성할 필요 X -> React가 해줌(IoC)
- React.createElement(), JSX, React.createFactory()은 element를 return 한다

React element는 `기존 OOP UI model`의 문제점을 개선하기 위해 나왔으며, element는 DOM Tree 생성에 필요한 정보를 담은 `plain object`이고 화면에 그려질 것들을 props와 type으로서 DOM node를 설명한다.(중요: 실제 DOM이 아님) 또한 reconcliiation을 통해 react는 모든 dom tree를 완벽하게 알게되고, react는 렌더링과 제거되는 부분을 적절하게 알고 따로 진행한다. 이러한 결과로 도달할 수 있는 부분은 component는 props를 input으로 받아 DOM Node를 출력하는, React로 만들어진 앱을 이루는 최소한의 단위라는 것이다.
