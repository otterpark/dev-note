# About React Testing Library

`React Testing Library`는 기존의 테스팅 도구보다 `DOM Testing Library`를 제공해주어 `React 컴포넌트`를 보다 편하고 가볍게 테스트 할 수 있게 해주는 라이브러리이다.

`React Testing Library`는 `React 컴포넌트`를 테스트하기 위한 매우 가벼운 솔루션입니다. 더 나은 테스트 관행을 장려하는 방식으로 `react-dom`과 `react-dom/test-utils`로 가벼운 유틸리티 기능을 제공합니다.

기본 원칙은 아래와 같습니다.

> 테스트가 소프트웨어 사용 방식과 유사할수록 더 많은 신뢰를 얻을 수 있습니다.

테스트는 렌더링된 `React 컴포넌트의 인스턴스`를 다루는 대신 `실제 DOM 노드`에서 작동합니다. 이 라이브러리가 제공하는 유틸리티는 **사용자가 사용하는 것과 동일한 방식**으로 `DOM을 쿼리`할 수 있도록 도와줍니다.

즉, `React Testing Library`를 사용하면 애플리케이션의 접근성을 높이고 사용자가 구성 요소를 사용하는 방식에 더 가깝게 테스트할 수 있습니다(이 라이브러리는 `Enzyme`을 대체합니다).
