---
title: innerHTML은 쓰기 싫지만 선언형으로 쓰고 싶어
date: "2022-03-07T17:01:32.284Z"
---

지난 미션을 거치면서 앞으로는 이왕이면 innerHTML을 사용하지 않고 append, appendChild로  사용하는 방향으로 코드를 구현하기로  마음을 먹었다. 

지난 미션에서 아래와 같은 문제가 있었기 때문이다.

### 초기 코드 예시

```jsx
const progressNameTemplate = `<div>${name}</div>`
this.progressField.innerHTML = progressNameTemplate;
```

name 변수를 사용자의 입력 값을 그대로 받아와서 사용한다고 했을 때 사용자가 악의적으로 `<div>`라는 이름을 입력하면 마크업 자체를 망가뜨릴 수 있다.

### 그래서 수정했다

```jsx
// utils.js
export const createDivWithClassName = (className) => {
  const element = document.createElement('div');
  element.className = className;
  return element;
};

// view.js
const nameElement = createDivWithClassName(DOM.CAR_NAME_CLASS);
nameElement.textContent = name;
this.progressElement.append(nameElement)
```

각 HTML 요소를 만드는 함수만 유틸 함수로 따로 제공하면 코드를 몇 줄 추가하지 않더라도 충분히 append를 사용해서 사용자의 악의적인 입력 값을 방지할 수 있는 좋은 방법이라고 생각했다.

### 그러나 돌아온 새로운 피드백?

> 명령형으로 UI를 만드신 이유가 있나요?
> 

> innerHTML과 insertAdjacentHTML을 실무에서는 사용하지 않지만 지금 프로젝트에서 많이 사용하는 이유는 js로 동적인 값을 UI에 넣어줄수 있고 선언형이기 때문입니다. 읽는 사람으로서는 당연히 선언형으로 작성된 코드가 훨씬 빠르게 읽히죠.
> 

그렇다. JS에서 요소를 생성해주면 HTML과는 전혀 다른 형태로 구현이 되어야 하고 어떤 요소가 어디에 들어가는 지 복잡해질 수록 알기가 어려워진다는 단점이 있었다. 이것이 현재 코드의 상태처럼 명령형으로 구현된 코드의 단점이다. 이에 대해서는 프롤로그에 참고하기에 아주 좋은 자료가 있었다. (****[명령형과 선언형](https://prolog.techcourse.co.kr/studylogs/1860) by 소피아)**

이 시점에서 innerHTML로 돌아가는 선택을 내릴 수 있었지만 고집을 부려보고 싶었다. 지금과 같은 작동 방식으로 append를 사용하지만 좀 더 선언형으로 구현할 수 있는 방법을 찾아보겠다고 마음을 먹었다.

> 좋긴하지만 어려운길이라 예상합니다^_ㅠ react 공식문서의 첫 소개문구가 왜 선언형이며, jsx를 사용하게되었는지 생각해보면 도움될것 같네요
> 

이 선택에 대해서 리뷰어님의 조언을 참고해서 리액트의 JSX부터 고민해보기로 했다.

### 리액트와 선언형

리액트는 실제로 가장 [공식 홈페이지](https://ko.reactjs.org/)의 가장 첫 페이지에 아래와 리액트의 특징을 설명하고 있다.

> 선언형
....
선언형 뷰는 코드를 예측 가능하고 디버그하기 쉽게 만들어 줍니다.
> 

리액트는 이를 위해서 JSX를 사용하고 있다.

### JSX?

[리액트 공식 문서](https://ko.reactjs.org/docs/introducing-jsx.html)는 JSX를 아래와 같이 설명하고 있다.

```jsx
const element = <h1>Hello, world!</h1>;
```

> 위에 희한한 태그 문법은 문자열도, HTML도 아닙니다.

JSX라 하며 JavaScript를 확장한 문법입니다. UI가 어떻게 생겨야 하는지 설명하기 위해 React와 함께 사용할 것을 권장합니다. JSX라고 하면 템플릿 언어가 떠오를 수도 있지만, JavaScript의 모든 기능이 포함되어 있습니다.

JSX는 React “엘리먼트(element)” 를 생성합니다. [다음 섹션](https://ko.reactjs.org/docs/rendering-elements.html)에서는 DOM에 어떻게 렌더링하는지 알아보겠습니다. 아래를 보면 JSX를 시작하기 위해 필요한 기본사항을 찾으실 수 있습니다.
> 

리액트는 JS의 문법을 모두 포함하면서 HTML의 태그와 유사한 형태로 HTML 요소를 만들 수 있는 문법을 제공하는 확장된 파일 형식을 사용하는 것이다. 이것을 일반 브라우저에서 JS로 인식시킬 수 있도록 트랜스파일러인 babel을 사용한다. 즉 JSX 파일을 babel로 트랜스파일해서 순수한 JS로 만들어주는 것이다. 이  과정을 거치면 위의 코드는 아래처럼 해석된다. ([온라인 트랜스파일러](https://babeljs.io/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=&version=7.17.6&externalPlugins=&assumptions=%7B%7D)를 통해서 직접 실험해 볼 수도 있다.)

```jsx
function hello() {
  return /*#__PURE__*/React.createElement("div", null, "Hello world!");
} 
```

즉 트랜스파일을 거친 후에 보면 리액트도 리액트만의 방식으로 HTML요소를 만드는 함수를 가지고 있다는 것을 알 수 있다. 

### 시작은 createElement부터

`React.createElement`는 `React.createElement(component, props, ...children)`의 형식으로 구현되어있다. 첫 파라미터는 컴포넌트의 타입이다. (리액트에서는 태그 이름일 수도 있고 리액트 컴포넌트일 수도 있다). 두 번째는 요소의 각종 속성들을 받는다. 마지막으로 받는 것은 자녀 요소들이다. 

이 함수에서 착안해서 이번 프로젝트에 사용할 createElement 함수를 만들어보기 시작했다. 현재 프로젝트에서는 `component`의 위치에 들어갈 것은 태그의 이름 밖에 없고 props로 속성을 받아야 하는 것도 id나 class로 제한해서 생각했다. 그래서 아래와 같은 코드를 구현했다.  

```jsx
const createCustomElement = ({ tag, className, id, children }) => {
  const element = document.createElement(tag);
  Object.assign(element, className && { className }, id && { id });
  if (Array.isArray(children)) {
    element.append(...children);
    return element;
  }
  element.append(children);
  return element;
};
```

태그와 class, id를 받는 것까지는 큰 어려움이 없었다. children이 생각보다 조금 까다로웠는데 자녀 요소를 일괄적으로 배열로 받으면 문자열이 있을 때는 spread 연산자를 사용하면 문자열 그대로를 표시해주는 것이 아니라 큰 따옴표가 붙기 때문이다. 그래서 위에서처럼 문자열이 단독으로 자녀일 때는 자녀 요소 자체를 append 하도록 했고 그 외의 경우에만 spread 연산자를 사용해서 append 하도록 했다. 이 부분은 하지만 아직 미완성인데 만약 p 태그 안의 span 태그 같은 상황에서는 큰 따옴표 문제를 해결하지 못했기 때문이다.

이제 아래와 같이 코드를 작성해서 요소를 만들 수 있었다.

```jsx
createCustomElement ({
	tag: 'div',
  className: 'lotto',
  children: [
    createCustomElement({ tag: 'p', className: 'lotto-image', children: '🎟️' }),
    createCustomElement({ 
			tag: 'p',
      className: 'lotto-numbers',
      children: Array.from(lottoNumberSet).join(', '),
    }),
  ],
});
```

전보다는 훨씬 HTML스러워졌다고 생각한다. 하지만 조금 아쉬운 부분이 있었는데 createCustomElement라는 함수 명이 길면서도 많이 노출이 되어있어서 가독성이 떨어진다는 느낌을 받았다. 그래서 아래와 같이 추가적으로 함수를 만들어서 조금 더 해소해보고자 했다.

```jsx
//createElement.js
const createCustomElement = ({ tag, className, id, children }) => {
  const element = document.createElement(tag);
  Object.assign(element, className && { className }, id && { id });
  if (Array.isArray(children)) {
    element.append(...children);
    return element;
  }
  element.append(children);
  return element;
};

export const div = function createCustomDiv({ className, id, children }) {
  return createCustomElement({ tag: 'div', className, id, children });
};

export const p = function createCustomP({ className, id, children }) {
  return createCustomElement({ tag: 'p', className, id, children });
};

export const label = function createCustomLabel({ className, id, children }) {
  return createCustomElement({ tag: 'label', className, id, children });
};

// view.js
div({
  className: 'lotto',
  children: [
    p({ className: 'lotto-image', children: '🎟️' }),
    p({
      className: 'lotto-numbers',
      children: Array.from(lottoNumberSet).join(', '),
    }),
  ],
});
```

### 그래도 발전이 있었다.

아직은 개선할 부분이 많이 보인다. 앞서서 설명한 span 등의 태그의 구현 문제도 있고, 지금 방식으로는 사용할 모든 코드에 대해서 함수를 따로 만들어주어야 한다는 단점이 있다. 하지만 가장 큰 문제점은 지금 구조로도 부모-자녀 관계가 복잡해질 수록 코드의 가독성이 떨어진다는 것이었다. 다른 경우에 사용해봤는데 depth가 세 개만 넘어가도 코드를 작성하는 것부터 어려움이 있었다.

### 다음은 JSX?

기회가 된다면 다음에는 babel 이용해서 JSX의 형태를 JS로 트랜스파일 한 뒤에 createElement 함수를 내가 만든 함수로 대체하는 방식으로 발전시켜보고 싶다.

> 이왕 react를 모티브로 하고있으니 해당 아티클을 참고해도 좋을것 같습니다. [Build your own React](https://pomb.us/build-your-own-react/)
> 

리뷰어님이 위와 같은 자료도 추천해주셨는데 결국 이 길이 나만의 리액트를 만드는 경험을 하는 여정 같다는 느낌이 들었다.

---

관련 PR

1. [로또 미션 1 단계 PR](https://github.com/woowacourse/javascript-lotto/pull/88)
2. [로또 미션 2 단계 PR](https://github.com/woowacourse/javascript-lotto/pull/129) 

참고자료

1. [https://www.daleseo.com/react-jsx/](https://www.daleseo.com/react-jsx/)
2. [https://velog.io/@hanei100/React-React.createElement](https://velog.io/@hanei100/React-React.createElement)
3. [https://ko.reactjs.org/docs/introducing-jsx.html](https://ko.reactjs.org/docs/introducing-jsx.html)
4. [https://ko.reactjs.org/docs/react-without-jsx.html](https://ko.reactjs.org/docs/react-without-jsx.html)