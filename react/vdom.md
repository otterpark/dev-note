# VDOM

## DOM(Document Object Model) 이란?

브라우저 엔진에서 HTML 코드를 JavaScript 언어로 제어할 수 있게 만들어주는 `Object`

## VDOM(Virtual DOM) 이란?

VDOM을 설명하기 위해서는 개인적으로 근본적인 원인을 찾기 위해서는 브라우저 렌더링 방식을 알아야 한다고 생각합니다.

![dom](./img/dom.png)

1. `DOM Tree`: HTML을 파싱하여 DOM Node로 이루어진 트리를 만듭니다.
2. `Attachment`: CSS를 파싱하여 CSSOM을 만듭니다.
3. `Render Tree 생성` - DOM 트리와 CSSOM 트리가 결합하여 실제 페이지를 렌더링하는 데 필요한 노드를 가지고 Render Tree를 형성합니다.
4. `Layout`: 각 노드들은 스크린의 좌표가 주어지며, 정확히 어디에 나타나야 할 지 위치가 주어진다. (box 형태)
5. `printing`: Render Tree가 화면에 내용을 표시하기 위해 전체적으로 탐색을 거쳐 픽셀 형태로 렌더링 요소들에 색을 입히는 과정을 진행한다.

브라우저는 HTML을 전달받으면 아래와 같은 과정들을 거쳐 페이지를 렌더링한다. 기존 브라우저단에서 DOM 변화가 생길 시 **`이 모든 과정들을 다시 실행`** 하게 됩니다. 이 과정들을 반복하여 생성할 시 결국 브라우저에서 많이 연산을 해야하기 때문에 **`렌더링 성능`** 이 떨어집니다. 

이런 비효율적인 것을 `Virtual DOM(가상 돔)`으로 해결할 수 있다. DOM의 변화가 생길 시 실제 DOM에 접근하여 조작하는 것이 아닌 DOM 작업을 가상화하여 미리 처리한 후 최종 결과를 Browser DOM 과 비교 후, 실제 DOM에 Patch 합니다. 즉 가상 DOM은 최소한의 돔 조작을 통해 실행되게 도와줍니다.

### 가상 DOM 과정

1. 데이터가 변했다.
2. 변화된 버전의 전체 UI를 가상돔에 리렌더링한다.
3. 기존의 Browser DOM과 변화된 가상 DOM을 비교한다.
4. 바뀐 부분만 실제 DOM에 Patch 한다.

[![React VDOM 공식 문서 설명](https://scrap.kakaocdn.net/dn/bjflAd/hyNUteQ4qD/83OvTKyC1kygCrXBtUeO8K/img.jpg?width=1280&height=720&face=0_0_1280_720)](https://www.youtube.com/watch?v=BYbgopx44vo)

## DOM과 VDOM의 차이점은 무엇일까요?

위에서 이야기한대로 DOM은 DOM 변화가 생길 시 브라우저 렌더링 과정을 거쳐 다시 재 렌더링 하고 이로인해 브라우저에 `렌더링 성능`이 저하되는데, VDOM 같은 경우에 최소한의 DOM 조작을 통해 DOM 변화를 처리합니다.
