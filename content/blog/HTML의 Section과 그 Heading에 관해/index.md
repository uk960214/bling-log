---
title: HTML의 Section과 그 Heading에 관한 새로운 사실
date: "2022-02-17T17:18:29.284Z"
---

[관련 Review 코멘트](https://github.com/woowacourse/javascript-racingcar/pull/77#discussion_r805353358)

## HTML의 Section

[Section에 대한 MDN 문서](https://developer.mozilla.org/ko/docs/Web/HTML/Element/section)

[Section Tag의 사용일람](https://developer.mozilla.org/ko/docs/Web/HTML/Element/section#%EC%82%AC%EC%9A%A9_%EC%9D%BC%EB%9E%8C)

> **HTML `<section>` 요소**는 HTML 문서의 독립적인 구획을 나타내며, 더 적합한 의미를 가진 요소가 없을 때 사용합니다. 보통 `<section>`은 제목을 포함하지만, 항상 그런 것은 아닙니다.
> 

## 알고 있었던 내용

HTML5의 Semantic Tags를 적절하게 사용하려면 페이지를 구획하기 위해서 무분별하게 `div`를 사용하는 것보다 적절하게 `section`과 `article` 등을 사용해서 구분해주어야 한다.

## 새롭게 알게 된 내용


> 💡 섹션 요소에는 반드시 제목이 포함되어야 한다. ([링크](https://velog.io/@apricotsoul/HTML-section요소-main-요소))

### 이유

`section`은 페이지의 요소를 의미 있게 구분 짓는 요소이다. 실제로 페이지에 제목이 표시하지 않더라도 주제와 역할을 담은 제목은 반드시 포함되어야 한다. (제목을 달 수 없는 섹션은 존재하지 않아야 할지도 모른다.) 디자인 상 생략하는 것이 불가피하다면 `hidden` 속성을 추가한 후 `label`을 사용해서 [스크린 리더](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%81%AC%EB%A6%B0_%EB%A6%AC%EB%8D%94)를 통해서는 접근할 수 있도록 설정해 주어야 한다.

```html
<section aria-labeledby="a11y-markup-headline">
<h2 hidden id="a11y-markup-headline">정보 접근성과 HTML 마크업</h2>
...
</section>
```

cf. `a11y`는 accessibility의 준말로 접근성을 의미한다.