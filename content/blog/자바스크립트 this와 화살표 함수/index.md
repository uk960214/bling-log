---
title: 자바스크립트 this와 화살표 함수
date: "2022-02-24T15:43:30.284Z"
---

# 마주한 문제

이벤트 핸들러를 추가하는 과정에서 인스턴스의 매서드에 연결하는 과정에서 호출된 매서드의 `this`가 인스턴스가 아닌 이벤트를 호출 시킨 DOM 요소로 지정되어서 `this`를 사용해 인스턴스의 프로퍼티에 접근할 수 없었다!

# 해결책

매서드를 정의할 때, 일반 함수(매서드)로 선언하는 것이 아니라 화살표 함수로 정의한다!

그러면 this가 이벤트 객체인 요소를 가리키지 않고 인스턴스를 가리켜 제대로 작동한다.

# 원인 분석

## 자바스크립트에서 `this`

> 화살표 함수는 자신의 `this`가 없습니다.  대신 화살표 함수를 둘러싸는 렉시컬 범위(lexical scope)의 `this`가 사용됩니다; 화살표 함수는 일반 변수 조회 규칙(normal variable lookup rules)을 따릅니다. 때문에 현재 범위에서 존재하지 않는 `this`를 찾을 때, 화살표 함수는 바로 바깥 범위에서 `this`를 찾는 것으로 검색을 끝내게 됩니다.

ES6 이전에는 자바스크립트에서 `this`는 호출한 방법에 따라서만 좌우되었다.

전역에서 호출한 경우 그 함수 내부에서 `this`는 전역 객체를 참조하고, this의 문맥을 넘겨주기 위해서는 호출 시에 `call` 혹은 `apply`를 통해서 넘겨주어야 했다.

ES5에서는 새로 `bind`가 도입되었는데 호출 시에 함수에 `bind`를 한 경우 호출된 함수와 동일한 코드를 복사해서 `bind`의 첫 번째 매개 변수를 `this`로 고정하는 새로운 함수를 생성한다.

> 화살표 함수 도입에 영향을 준 두 요소: 보다 짧아진 함수 및  `바인딩하지 않은 this.`

ES6에서 화살표 함수가 도입되는 데 영향을 미친 두 요소 중 하나가 바로 `bind`이다.

> 현재 범위에서 존재하지 않는 `this`를 찾을 때, 화살표 함수는 바로 바깥 범위에서 `this`를 찾는 것으로 검색을 끝내게 됩니다.

객체의 매서드로는 아래와 같은 현상이 나타난다.

```jsx
var obj = { // does not create a new scope
  i: 10,
  b: () => console.log(this.i, this),
  c: function() {
    console.log(this.i, this)
  }
}
obj.b(); // prints undefined, Window {...} (or the global object)
obj.c(); // prints 10, Object {...}
```

위 예시에서 `obj`는 전역에서 정의되고 사용되어 블록 내부에서 `this`는 전역 객체인 `window`를 가리킨다. `b`와 `c`는 모두 `this`를 참조하도록 되어있지만 화살표 함수인 `b`에서 `this`는 `obj`의 `this`인 `window`를, 일반 매서드인 `c`에서 `this`는 해당 객체와 연결된다.

> 함수를 어떤 객체의 메서드로 호출하면 `this`의 값은 그 객체를 사용합니다.

## 다시 이벤트로 돌아와서

`addEventListener` 자체도 `EventTarget`이라는 인터페이스의 매서드로 이벤트를 제공하는 모든 객체에 대해서 리스너를 추가하는 기능을 한다. 그 말인 즉 `addEventListener`의 내부에서 `this`는 소속된 객체인 이벤트 객체라는 것을 의미한다.

원래 아래와 같이 클래스 내부에서 매서드를 이벤트 핸들러로 추가해줬었다.

```jsx
class Controller {
	start() {
		this.someDom = document.querySelector('.something');
		this.someDom.addEventListener('click', this.onDomClick);
	}
	
	onDomClick() {
		console.log(this) // <button class="something">...</button>
	}
}
```

위와 같이 구현된 코드에서는 `this`가 DOM 요소로 log된다.

```jsx
class Controller {
	start() {
		this.someDom = document.querySelector('.something');
		this.someDom.addEventListener('click', this.onDomClick);
	}
	
	onDomClick = () => {
		console.log(this) // Controller {...}
	}
}
```

# 결론

이벤트 핸들러 함수 안에서 인스턴스에 접근하기 위해 `this`를 사용하고 싶으면 매서드를 화살표 함수로 정의해야 한다.

---

출처

[https://developer.mozilla.org/ko/docs/Web/API/EventTarget/addEventListener](https://developer.mozilla.org/ko/docs/Web/API/EventTarget/addEventListener)

[https://developer.mozilla.org/ko/docs/Web/API/EventTarget](https://developer.mozilla.org/ko/docs/Web/API/EventTarget)

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions#%EB%A9%94%EC%86%8C%EB%93%9C%EB%A1%9C_%EC%82%AC%EC%9A%A9%EB%90%98%EB%8A%94_%ED%99%94%EC%82%B4%ED%91%9C_%ED%95%A8%EC%88%98)

[https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)