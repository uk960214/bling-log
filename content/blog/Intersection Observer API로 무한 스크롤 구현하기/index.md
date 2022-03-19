---
title: Intersection Observer API로 무한 스크롤 구현하기
date: "2022-03-15T15:17:11.284Z"
---

기존에 도전해봤던 스크롤 이벤트 관련된 기능들은 윈도우 자체에 scroll 이벤트를 추가해서 특정 높이가 되었을 때 핸들러를 실행하는 방식이었다. 이번 기회를 통해서 Intersection Observer API를 사용해봤는데 생각보다 너무 깔끔하고 간단하게 되어있어서 간단한 로직으로 무한 스크롤을 구현할 수 있었다.

## 기본적인 작동 방식

MDN 문서에 따르면 Intersection Observer API를 사용하면 아래 경우에 설정한 콜백 함수가 발동된다고 설명한다.

> - A **target** element intersects either the device's viewport or a specified element. That specified element is called the **root element** or **root** for the purposes of the Intersection Observer API.
> - The first time the observer is initially asked to watch a target element.
>
> 
>
> - 목표 요소가 기기의 viewport 혹은 특정한 요소와 교차하는 경우. 이 특정한 요소는 Intersection Observer API를 위한 **root element** 혹은 **root**라고 칭한다.
> -  처음으로 observer에게 목표 요소를 지켜보라고 요청했을 때.

특정 이벤트 시에 콜백 함수를 실행한다는 점에서 이벤트 리스너와 유사하지만 정의하고 사용하는 문법에는 차이가 있다.

먼저 이  observer를 사용하기 위해서는 새로 Intersection Observer를 정의해주어야 한다. 정의한 뒤, 지켜보고자 하는 요소를 지켜보도록 observer 매서드에 추가해주면 된다.

```js
const target = document.querySelector('.selector');
const observer = new IntersectionObserver(callback);
observer.observe(target)
```

위와 같이 구현하면 앞서서 설명한 것처럼 콜백 함수가 1회 무조건 실행된다. target을 지켜보라고 요청할 때 기본적으로 실행해주기 때문이다. 하지만 무한 스크롤을 구현하기 위해서는 처음에는 실행하지 않고 해당 요소가 viewport와 교차할 때만 실행하도록 콜백 함수를 구현해주어야 한다.

### Intersection Observer의 콜백 함수

Intersection Observer의 콜백 함수는 IntersectionObserverEntry라는 객체의 배열을 인자로 받는다. 이 중에서 현재 가장 중요한 것은 isIntersecting이라는 속성이다. 이 Observer의 root요소와 target 요소가 교차하고 있는 지를 계산해 boolean 값으로 반환한다. 앞서서 구현하고자 했던 'viewport와 교차할 때만 실행하도록'이라는 조건을 추가하기에 가장 적합한 속성이다. 그래서 콜백함수를 아래와 같이 기본 설정을 해줄 수 있다.

```js
const callback = (entries) => {
    if (entries[0].isIntersecting) {
     	// do something
    }
}
```

entries의 0번 요소를 참조하는 이유는 인자로 받는 것이 교차할 때마다 생성되는 entry의 배열이기 때문이다. (이 부분에 대해서는 내용적인 추후 보완이 필요하다.)

## threshold 설정하기

아무런 추가 설정도 해주지 않은 가장 기본적인 설정에서는 target 요소가 root 요소와 단 1%만 교차하더라도 콜백 함수가 실행된다. 하지만 경우에 따라 사용자가 확실히 그 컨텐츠를 확인했을 때, 다시 말해 target 요소가 50% 혹은 75%, 때로는 100% 다 교차했을 때 함수를 실행하기 원할 수 있다. 이를 위해서 추가적인 옵션을 설정해줄 수 있다.

```js
const observer = new IntersectionObserver(callback, { threshold: 0.5 });
```

위와 같은 설정으로 이제 target 요소가 50% 이상 교차하면 콜백함수가 실행되도록 설정해주었다.

## unobserve

무한 스크롤을 구현하는 데에 있어서 observer 콜백 함수 실행 이후에도 마지막 요소를 지켜보고 있으면 예상치 못한 결과가 발생할 수 있다. 다음 요소들이 그 뒤로 추가 로딩이 되어서 더 이상 그 요소는 마지막 요소가 아니게 될 것이기 때문이다. 만약 이 observer를 제거하지 않고 그대로 둔다면, 마지막 요소가 아닌 요소가 화면을 나가고 들어올 때에도 추가적인 요소가 로딩되는 결과를 초래할 것이다. 이를 방지하기 위해서 한 번 콜백 함수가 실행 될 때 지금 지켜보고 있는 요소에서 observer를 제거하는 과정이 필수적이다. 그 이후 함수가 실행된 이후에 새로 생성된 마지막 요소를 observer가 지켜보도록 설정해주면 추가적으로 로딩된 요소의 끝에 다다랐을 때만 그 다음을 로딩할 수 있게된다.

```js
const callback = (entries) => {
    if (entries[0].isIntersecting) {
    	observer.unobserve(entries[0].target)
     	// do something
    }
}
```

## 실제로 적용해보기

앞서 살펴본 모든 방식을 조합해서 구현한 무한 스크롤은 아래와 같은 방식이다.

```js
class View {
    ...
    constructor() {
        this.videoList = document.querySelector('.video-list');
        this.loadMoreResultObserver = this.#handleScrollToLastItem();
    }

    #handleSearch = async (event) => {
        event.preventDefault();
        const { value: keyword } = this.searchInputKeyword;
        this.#sendSearchRequest(keyword);
    };

    #handleScrollToLastItem() {
        return new IntersectionObserver(
            async (entries) => {
                if (entries[0].isIntersecting) {
                    this.loadMoreResultObserver.unobserve(entries[0].target);
                    this.#sendSearchRequest();
                }
            },
            { threshold: 0.5 }
        );
    }

	async #sendSearchRequest(keyword) {
        try {
          this.videoList.insertAdjacentHTML('beforeend', this.#skeletonTemplate());
          const { searchResultArray, hasNextPage } = await this.search.handleSearchRequest(keyword);
          this.#renderSearchResult({ searchResultArray, keyword, hasNextPage });
        } catch (error) {
          this.#renderError(error.message);
        }
      }

    #renderSearchResult({ searchResultArray, keyword, hasNextPage }) {
        const resultElementArray = searchResultArray.map((resultItem) =>
          this.#createVideoElement(resultItem)
        );

        this.videoList.append(...resultElementArray);
        if (hasNextPage) this.loadMoreResultObserver.observe(this.videoList.lastChild);
    }


    
}
```

우선 사용 이전에 IntersectionObserver를 생성해준다. 함수의 분리와 명명을 통해서 의미를 명확히 하기 위해 새 IntersectionObserver를 반환하는 매서드로 만들어서 실행시켜주었다.

이 로직에서 가장 처음 검색은 검색어를 입력하고 제출할 때 실행된다. 그 때까지 관찰할 대상이 없는 observer는 실행되지 않는다. 처음 가지고 온 결과를 랜더링 하는 과정에서 처음으로 검색 결과 중 마지막 요소를 관찰하도록 추가된다.

```js
#renderSearchResult({ searchResultArray, keyword, hasNextPage }) {
    ...
    if (hasNextPage) this.loadMoreResultObserver.observe(this.videoList.lastChild);
}
```

이후 마지막 검색 결과의 50% 이상이 화면에 표시되면 observer의 콜백 함수가 실행된다.

```js
async (entries) => {
    if (entries[0].isIntersecting) {
        this.loadMoreResultObserver.unobserve(entries[0].target);
        this.#sendSearchRequest();
    }
}
```

기존에 관찰하고 있던 요소는 unobserve하고 새로운 요청을 보낸 뒤 받아온 결과를 render한다. 이 과정에서 자연스럽게 마지막 요소에 다시 observer가 마지막 요소를 관찰하게 된다.