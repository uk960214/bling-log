---
title: TDD와 E2E 테스트 작성하기
date: "2022-02-18T10:03:09.284Z"
---
## TDD **(Test Driven Development)**

> TDD는 테스트를 먼저 작성하고 그 후 실제 코드를 작성하는 방법입니다. TDD는 **켄트 백(Kent Beck)**이 개발 혹은 널리 알린 개념입니다. 흔히들 개발전에 테스트 코드를 짜는게 TDD라고만 생각할 수 있는데, 그보다 본질적인 의미는 **문제를 정의하고, 그 해답을 찾아가는 과정이**라는게 TDD의 기본 취지입니다. 그리고 테스트도구는 그 철학을 이행하는 도구입니다.


### 내가 이해한 내용

- 테스트 > 개발의 순서로 진행한다고 해서 모두 TDD인 것은 아니다.
- 인용에서 적혀있듯이 개발하기 이전에 어떠한 문제를 해결할 것인지 목표를 설정한 후 개발을 통해서 그에 대한 해결책을 찾아나가는 과정을 TDD라고 한다.
- 처음에 테스트를 작성하면 (당연히) 실패하고 개발을 통해서 이를 성공으로 바꾸어나가는 패턴으로 작업한다.
- 일단 테스트에 성공하는 (지저분할 수도 있는) 코드를 작성한 후 리팩터링을 통해서 클린 코드로 만들어간다.

## E2E(End to End) 테스트

> E2E(End to End) 테스트는 개발물을 사용자 관점에서 테스트 하는 방법이다. 페이지에서 원하는 텍스트가 제대로 출력이 되었는지, 버튼을 클릭 했을 때 올바른 동작을 수행하는 지 등을 테스트한다. ([출처](https://hyg4196.tistory.com/127))
> 

### 내가 이해한 내용

- E2E 테스트의 특징은 테스트가 사용자 관점에서 이루어진다는 점이다.
- 즉 사용자가 페이지에서 취할 수 있는 액션을 검증하고 의도한 대로 결과가 작동하는 지 확인하는 과정이다.
- 따라서 E2E 테스트의 명세를 작성할 때에는 개발자의 관점의 형식이 아닌, 사용자의 액션에 따른 예상 결과의 형식으로 구성해야 한다.

```jsx
// bad
it("잘못된 자동차 이름을 입력하면 alert이 호출되어야 한다.", () => {})

// good
it("올바른 자동차 이름과 횟수를 입력하면 게임이 진행되고 우승자를 확인할 수 있어야 한다", () => {})
```

- 또한 검증 과정에서 화면에 표시되지 않는, 코드의 구조를 알아야 확인 할 수 있는 로직으로는 테스트를 작성하지 않는다 ([잘못된 코드 링크](https://github.com/woowacourse/javascript-racingcar/pull/77/commits/77ef5d3f033c416a131900da0850cef531fbd502))

```jsx
Cypress.Commands.add('stubRandomReturn', (returnValues = []) => {
  const randomStub = cy.stub();

  // 이 명령에서 처리하는 과정은 자동차를 전진시키기 위해서 어떤 로직이 사용되는지 알아야만 작성할 수 있는 내용이다. => E2E 테스트라고 할 수 없다.

	// 전진시키기 위해서 특정 값을 반환할 것을 예측하고 있다.
  returnValues.forEach((value, index) => {
    randomStub.onCall(index).returns(value);
  });

  cy.visit(baseUrl, {
    onBeforeLoad: (window) => {
			// 대체 해야할 함수가 무엇인지 정확히 알고 있다.
      window.MissionUtils = {
        Random: {
          pickNumberInRange: randomStub,
        },
      };
    },
  });
});

// 어떤 값을 반환해야 전진하는지 정확히 알고 있다.
beforeEach(() => {
  cy.stubRandomReturn([5, 1]);
});

it('올바른 입력값을 입력하면 우승자가 표시되어야 한다.', () => {
  //given
  const nameInput = 'bling,juunz';
  const countInput = 1;
  const winner = 'bling';

  //when
  cy.get('#car-name-input').type(nameInput);
  cy.get('#car-name-btn').click();
  cy.get('#count-input').type(countInput);

  //then
  cy.get('#count-btn')
    .click()
    .then(() => {
      cy.get('#winner-name').should('have.text', winner);
    });
});
```

# 3. Cypress 테스트 코드 리팩터링 여정기

## 가장 처음 작성한 코드 ([링크](https://github.com/woowacourse/javascript-racingcar/pull/77/commits/77ef5d3f033c416a131900da0850cef531fbd502?diff=split&w=0))

```jsx
it('잘못된 자동차 이름을 입력하면 alert가 호출되어야 한다.', () => {
    //given
    const alertStub = cy.stub();
    const invalidInput = 'juunzzi';

    cy.on('window:alert', alertStub);

    //when
    cy.get('#car-name-input').type(invalidInput);

    //then
    cy.get('#car-name-btn')
      .click()
      .then(() => {
        expect(alertStub).to.be.called;
      });
  });

  it('횟수 입력란에 1 미만의 값이 주어지면 alert가 호출되어야 한다.', () => {
    //given
    const alertStub = cy.stub();
    const invalidInput = -1;

    cy.on('window:alert', alertStub);

    //when
    cy.get('#count-input').type(invalidInput);

    //then
    cy.get('#count-btn')
      .click()
      .then(() => {
        expect(alertStub).to.be.called;
      });
  });

  it('잘못된 자동차 이름을 입력하면 alert가 호출되어야 한다.', () => {
    //given
    const nameInput = 'bling,juunz';
    const countInput = 1;
    const winner = 'bling';

    //when
    cy.get('#car-name-input').type(nameInput);
    cy.get('#car-name-btn').click();
    cy.get('#count-input').type(countInput);

    //then
    cy.get('#count-btn')
      .click()
      .then(() => {
        cy.get('#winner-name').should('have.text', winner);
      });
  });
```

## 1차 수정 ([링크](https://github.com/woowacourse/javascript-racingcar/pull/77/commits/91be7c7e0cddc62eb2df81840f560eb14af42aea))

```jsx
it('게임을 완료하고 우승자를 확인할 수 있어야 한다.', () => {
  //given
  const nameInput = 'bling,juunz';
  const countInput = 1;
  const winner = 'bling';

  //when
  cy.get('#car-name-input').type(nameInput);
  cy.get('#car-name-btn').click();
  cy.get('#count-input').type(countInput);
  cy.get('#count-btn').click();

  //then
  cy.get('#winner-name').should('have.text', winner);
});
```

## 2차 수정 ([링크](https://github.com/woowacourse/javascript-racingcar/pull/77/commits/7ffa499ac8d387996b1b1060a2e134cd520523dd))

```jsx
const carNameInputProcess = (nameInput) => {
  cy.get(`#${DOM.CAR_NAME_INPUT_ID}`).type(nameInput);
  cy.get(`#${DOM.CAR_NAME_BTN_ID}`).click();
};

const countInputProcess = (countInput) => {
  cy.get(`#${DOM.COUNT_INPUT_ID}`).type(countInput);
  cy.get(`#${DOM.COUNT_BTN_ID}`).click();
};

const setAlertStub = () => {
  const alertStub = cy.stub();
  cy.on('window:alert', alertStub);
  return alertStub;
};

// then function
const doesShowAlert = ({ inputSelector, inputValue, btnSelector, alertStub }) => {
  // when
  cy.get(inputSelector).type(inputValue);

  // then
  cy.get(btnSelector)
    .click()
    .then(() => {
      expect(alertStub).to.be.called;
    });
};

const isInitialStatus = () => {
  cy.get(`#${DOM.CAR_NAME_INPUT_ID}`).should('not.have.text');
  cy.get(`.${DOM.CAR_PROGRESS_CLASS}`).should('not.exist');
  cy.get(`.${DOM.WINNER_CONTAINER_ID}`).should('not.exist');
  cy.get(`#${DOM.COUNT_INPUT_ID}`).should('not.be.visible');
};

it('게임을 완료하고 우승자를 확인할 수 있어야 한다.', () => {
  //given
  const nameInput = 'bling,juunz';
  const countInput = 1;

  //when
  carNameInputProcess(nameInput);
  countInputProcess(countInput);

  //then
  cy.get(`#${DOM.WINNER_NAME_ID}`).should('be.visible');
});

it('잘못된 자동차 이름을 입력하면 alert가 호출되어야 한다.', () => {
  //given
  const alertStub = setAlertStub();
  const invalidInput = 'juunzzi';

  // when - then
  doesShowAlert({
    inputSelector: `#${DOM.CAR_NAME_INPUT_ID}`,
    inputValue: invalidInput,
    btnSelector: `#${DOM.CAR_NAME_BTN_ID}`,
    alertStub,
  });
});

it('횟수 입력란에 1 미만의 값이 주어지면 alert가 호출되어야 한다.', () => {
  //given
  const alertStub = setAlertStub();
  const nameInput = 'bling,juunz';
  const invalidInput = -1;
  carNameInputProcess(nameInput);

  // when - then
  doesShowAlert({
    inputSelector: `#${DOM.COUNT_INPUT_ID}`,
    inputValue: invalidInput,
    btnSelector: `#${DOM.COUNT_BTN_ID}`,
    alertStub,
  });
});

it('재시작 버튼을 누르면 처음 상태로 돌아가야 한다.', () => {
	//given
  const nameInput = 'bling,juunz';
  const countInput = 2;

  //when
  carNameInputProcess(nameInput);
  countInputProcess(countInput);
  cy.get(`#${DOM.RESTART_BTN_ID}`).click();

  //then
  isInitialStatus();
});
```

## 최종 수정

```jsx
import { DOM } from '../../src/lib/constants.js';

describe('구현 결과가 요구사항과 일치해야 한다.', () => {
  const baseUrl = '../../index.html';

  beforeEach(() => {
    cy.visit(baseUrl);
  });

  // then function
  const isInitialStatus = () => {
    cy.get(`#${DOM.CAR_NAME_INPUT_ID}`).should('not.have.text');
    cy.get(`#${DOM.COUNT_INPUT_ID}`).should('not.be.visible');
    cy.get(`.${DOM.CAR_PROGRESS_CLASS}`).should('not.exist');
    cy.get(`.${DOM.WINNER_CONTAINER_ID}`).should('not.exist');
  };

  it('올바른 자동차 이름과 횟수를 입력하면 게임이 진행되고 우승자를 확인할 수 있어야 한다.', () => {
    //given
    const nameInput = 'bling,juunz';
    const countInput = 1;

    //when
    cy.inputNames({ nameInput });
    cy.inputCount({ countInput });

    //then
    cy.get(`#${DOM.WINNER_NAME_ID}`).should('be.visible');
  });

  it('잘못된 자동차 이름을 입력하면 게임이 진행되지 않고 실패 피드백을 전달한다.', () => {
    //given
    const invalidInput = 'juunzzi';

    //then - expect alert
    cy.inputNames({ nameInput: invalidInput, isInvalidName: true });
  });

  it('횟수 입력란에 1 미만의 값을 입력하면 게임이 진행되지 않고 실패 피드백을 전달한다.', () => {
    //given
    const nameInput = 'bling,juunz';
    const invalidCountInput = -1;

    //when
    cy.inputNames({ nameInput });

    //then - expect alert
    cy.inputCount({ countInput: invalidCountInput, isInvalidCount: true });
  });

  it('재시작 버튼을 누르면 진행된 게임 정보가 지워지고 다시 게임을 진행할 수 있는 상태가 되어야 한다.', () => {
    //given
    const nameInput = 'bling,juunz';
    const countInput = 2;

    //when
    cy.inputNames({ nameInput });
    cy.inputCount({ countInput });
    cy.get(`#${DOM.RESTART_BTN_ID}`).click();

    //then
    isInitialStatus();
  });
});
```

```jsx
import { DOM, ERROR_MESSAGE } from '../../src/lib/constants';

Cypress.Commands.add('clickShowsAlert', (btnSelector, errorMessage) => {
  const alertStub = cy.stub();
  cy.on('window:alert', alertStub);

  // then
  cy.get(btnSelector)
    .click()
    .then(() => {
      expect(alertStub).to.be.calledWith(errorMessage);
    });
});

Cypress.Commands.add(
  'processInput',
  ({ inputSelector, btnSelector, input, isInvalidInput, errorMessage }) => {
    cy.get(inputSelector).type(input);

    // show alert on invalid input
    if (isInvalidInput) {
      cy.clickShowsAlert(btnSelector, errorMessage);
      return;
    }

    cy.get(btnSelector).click();
  }
);

Cypress.Commands.add('inputNames', ({ nameInput, isInvalidName = false }) => {
  cy.processInput({
    inputSelector: `#${DOM.CAR_NAME_INPUT_ID}`,
    btnSelector: `#${DOM.CAR_NAME_BTN_ID}`,
    input: nameInput,
    isInvalidInput: isInvalidName,
    errorMessage: ERROR_MESSAGE.CAR_NAME_LENGTH_OVER,
  });
});

Cypress.Commands.add('inputCount', ({ countInput, isInvalidCount = false }) => {
  cy.processInput({
    inputSelector: `#${DOM.COUNT_INPUT_ID}`,
    btnSelector: `#${DOM.COUNT_BTN_ID}`,
    input: countInput,
    isInvalidInput: isInvalidCount,
    errorMessage: ERROR_MESSAGE.INVALID_COUNT,
  });
});
```

## 1차 수정 시 반영 내용

- [ ]  게임 실행 테스트는 alert가 trigger 될 때와는 다르게 chaining 하면 오류 발생 ⇒ chainging 제거

```jsx
//when
cy.get('#car-name-input').type(nameInput);
cy.get('#car-name-btn').click();
cy.get('#count-input').type(countInput);

//then
cy.get('#count-btn')
.click()
.then(() => {
  cy.get('#winner-name').should('have.text', winner);
});
```

```jsx
//when
cy.get('#car-name-input').type(nameInput);
cy.get('#car-name-btn').click();
cy.get('#count-input').type(countInput);
cy.get('#count-btn').click();

//then
cy.get('#winner-name').should('have.text', winner);
```

## 2차 수정 시 반영 내용

- [x]  E2E 테스트 반영, 랜덤 함수를 체크하지 않고 결과 요소가 표시되는지 만을 검증
- [x]  반복적으로 사용되는 함수 분리, 상수화 한 문자열 활용

```jsx
// 자동차 이름을 입력하는 로직
const carNameInputProcess = (nameInput) => {
  cy.get(`#${DOM.CAR_NAME_INPUT_ID}`).type(nameInput);
  cy.get(`#${DOM.CAR_NAME_BTN_ID}`).click();
};

// 횟수를 입력하는 로직
const countInputProcess = (countInput) => {
  cy.get(`#${DOM.COUNT_INPUT_ID}`).type(countInput);
  cy.get(`#${DOM.COUNT_BTN_ID}`).click();
};

// alertStub를 설정하고 반환하는 로직
const setAlertStub = () => {
  const alertStub = cy.stub();
  cy.on('window:alert', alertStub);
  return alertStub;
};

// alert를 표시하는 테스트의 로직
// then function
const doesShowAlert = ({ inputSelector, inputValue, btnSelector, alertStub }) => {
  // when
  cy.get(inputSelector).type(inputValue);

  // then
  cy.get(btnSelector)
    .click()
    .then(() => {
      expect(alertStub).to.be.called;
    });
};
```

## 최종 수정 시 반영한 내용

- [x]  전역에서 사용되는 테스트 코드를 Cypress의 `commands`로 분리
- [x]  입력이 alert를 발동하는 로직을 별도의 함수로 선언하지 않고 기본 입력 로직에 인자로 잘못된 입력 값인지 여부를 주입해서 해당 함수 내에서 분기할 수 있도록 명령 생성
- [x]  alertStub는 인자로 주입하지 않고 alert를 발동했을 때 처리하는 함수에서 자체 생성하도록 변경
- [x]  위 내용까지 포함, 이름 입력과 횟수 입력에서 반복되는 코드를 묶어서 하나의 명령으로 선언

```jsx
// 자동차 이름을 입력하는 로직
const carNameInputProcess = (nameInput) => {
  cy.get(`#${DOM.CAR_NAME_INPUT_ID}`).type(nameInput);
  cy.get(`#${DOM.CAR_NAME_BTN_ID}`).click();
};

// 횟수를 입력하는 로직
const countInputProcess = (countInput) => {
  cy.get(`#${DOM.COUNT_INPUT_ID}`).type(countInput);
  cy.get(`#${DOM.COUNT_BTN_ID}`).click();
};

// alertStub를 설정하고 반환하는 로직
const setAlertStub = () => {
  const alertStub = cy.stub();
  cy.on('window:alert', alertStub);
  return alertStub;
};

// alert를 표시하는 테스트의 로직
// then function
const doesShowAlert = ({ inputSelector, inputValue, btnSelector, alertStub }) => {
  // when
  cy.get(inputSelector).type(inputValue);

  // then
  cy.get(btnSelector)
    .click()
    .then(() => {
      expect(alertStub).to.be.called;
    });
};
```