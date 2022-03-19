---
title: 3월 2주차 - 공부 조각 모음
date: "2022-03-15T15:08:03.284Z"
---

## 1. JavaScript에 새로운 기능은 어떻게 추가될까?

### ECMAScript와 TC39

JavaScript의 출시 이후 이 언어에 대한 표준 정립의 필요성이 대두되었고 정보통신시스템의 표준화를 추구하는 Ecma International에 의해서 [ECMAScript](https://tc39.es/ecma262/)로 표준화되었다. 이 표준은 필요에 따라서 수정, 보완되는데 이를 평가하고 판단하는 기구가 ECMA 산하의 [TC39](https://tc39.es/)이다. TC39에는 정해진 절차에 따라서 기능의 추가, 수정, 삭제 등에 대해서 논의하고 의결하는 기구이다. 2015년 이후로는 매년 이 기구에서 정하는 내용에 따라서 매년 개정된 ECMAScript를 발표하고 있다. 어떠한 논의들이 진행되는 지는 공개된 [Github Repository](https://github.com/tc39/ecma262)를 통해서 확인할 수 있다.

### ECMAScript와 Javascript의 관계

ECMAScript가 JavaScript를 표준화하기 위해서 탄생한 것은 맞지만 JavaScript는 현재 ECMAScript의 표준을 따르고 있다.  JavaScript는 이 표준을 따르면서 그 외의 기능까지 제공하는 확장 프로그램 격으로 생각할 수 있다.

참고: [What’s the difference between JavaScript and ECMAScript?](https://www.freecodecamp.org/news/whats-the-difference-between-javascript-and-ecmascript-cba48c73a2b5)

## 2. Class의 private 식별자의 사용법에 대해서 놓치고 있던 부분

지금까지 매서드에 대해서는 private을 잘 사용하고 있었지만 그 외의 멤버 변수에 대해서는 사용하지 않았다. 사용해보려고 여러가지 시도를 해보던 중, private 멤버에 대해서는 선언 시에 할당을 할 수 없다는 사실을 발견했다.

> Private fields can only be declared up-front in a field declaration.
>
> Private 필드는 반드시 먼저 필드 선언에서 선언되야 합니다.

MDN의 한국어 해석에서는 선언 시 할당이 불가능하다고 되어있는데, 영어 원문에는 할당에 대한 내용이라기보다는 사용과 동시에 선언하는 방식이 아닌, 클래스의 필드에 먼저 선언되어야 한다는 방식으로 서술되어있다. 조금 더 풀어서 설명하자면 사용할 때 필요한 경우 `this.프로퍼티` 로 바로 선언해 사용하는 일반 프로퍼티와는 구별되어 클래스 내부에 이미 선언이 된 상태로만 사용하고 할당할 수 있다는 것이다. 그래서 아래와 같은 예시대로 사용해야한다.

``` js
class Rectangle {
  #height = 0;
  #width;
  constructor(height, width) {
    this.#height = height;
    this.#width = width;
  }
}
```

## 3. `Object.freeze()`는 언제 사용해야 할까?

기존에 미션을 진행하면서 상수로 따로 분리해 놓은 파일에서 같은 맥락에서 사용해서 객체로 묶은 상수에 대해서는 `Object.freeze()`를 활용해서 객체가 불변하도록 설정해준 뒤 사용했다. [MDN 문서](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)는 아래와 같이 설명하고 있다.

> **`Object.freeze()`** 메서드는 객체를 **동결**합니다. 동결된 객체는 더 이상 변경될 수 없습니다. 즉, 동결된 객체는 새로운 속성을 추가하거나 존재하는 속성을 제거하는 것을 방지하며 존재하는 속성의 불변성, 설정 가능성(configurability), 작성 가능성이 변경되는 것을 방지하고, 존재하는 속성의 값이 변경되는 것도 방지합니다. 또한, 동결 객체는 그 프로토타입이 변경되는것도 방지합니다.

객체가 `const`를 이용해서 선언되었다고 해도, 내부 속성에 대해서는 변경이 가능했었는데, 이 매서드를 사용하면 그 모든 행위(추가, 수정, 삭제)를 원천적으로 막을 수 있는 것이다.

이 매서드에 대한 설명을 읽으면서 분명 분리된 상수를 고정하는 것 외에 쓰임이 있을 것이라고 생각했다. Snake Case로 너무나도 명확하게 선언된 상수에 대해서 새로운 값을 할당하려는 사람은 없을 것이라고 판단했기 때문이다.

그렇게 해서 검색해본 결과, 가장 설득력 있는 사용처는 [한 질문](https://stackoverflow.com/questions/14791302/why-would-i-need-to-freeze-an-object-in-javascript)에서 찾을 수 있었다.

> As an API author, this may be exactly the behavior you want. For example, you may have an internally cached structure that represents a canonical server response that you provide to the user of your API by reference but still use internally for a variety of purposes. Your users can reference this structure, but altering it may result in your API having undefined behavior. In this case, you want an exception to be thrown if your users attempt to modify it.
>
> API의 작성자로서, 이 작동 방식이 당신이 원하는 것일 수도 있다. 예를 들어, 당신은 캐노니컬 서버 응답을 나타내는 구조를 내부적으로 캐시해서 당신의 API의 사용자에게 참조 값으로 제공하는 동시에 여러가지 사유로 내부적으로 이를 사용할 수도 있다. 당신의 사용자는 이 구조를 참조할 수 있지만, 이를 변경하는 것은 당신의  API가 의도하지 않은 행동을 하도록 만들 것이다. 이 경우에 당신은 당신의 사용자가 이를 수정하려는 시도를 할 때 예외를 throw할 수 있다.

> When you're writing a library/framework in JS and you don't want some developer to break your dynamic language creation by re-assigning "internal" or public properties. This is the most obvious use case for immutability.
>
> 당신이 자바스크립트로 라이브러리/프레임워크를 작성하면서 다른 개발자가 "내부" 혹은 public한 속성에 재할당하는 것을 통해서당신의 동적 언어 생성을 망가뜨리는 것을 원치 않을 때. 이것이 불변성의 가장 당연한 사용처라고 할 수 있다.

즉 API나 Library, Framework 등 작성자가 사용자에게 객체에 대해서 참조 값으로 접근권을 제공하지만, 내부 작동 상의 예상치 못한 오류를 방지하기 위해서 변경할 수 없는 객체를 고정해서 예외처리 할 수 있도록 객체를 동결하는 것이다.

위와 같은 상황을 고려했을 때, 상수인 것이 너무도 당연한 현재의 케이스에는 객체를 동결하는 것이 불필요한 것이라는 결론을 내릴 수 있었다.

## 4. async, await의 목적

> **Note:**The purpose of `async`/`await` is to simplify the syntax necessary to consume promise-based APIs.
>
> `async`/`await`의 목적은 Promise 기반의 API를 소비하는 데 필요한 문법을 단순화하는 것이다. ([출처](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function))

비동기 문법을 동기처럼 사용하기 위해서 사용하는 문법이라고는 생각하고 있었지만 MDN에서는 생각보다 명확하게 API를 사용할 때 활용하는 것을 목적으로 표시하고 있었다.