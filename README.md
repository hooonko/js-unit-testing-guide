# 자바스크립트 유닛 테스트 가이드 (A guide to unit testing in JavaScript)

### 번역 관련

> 이 문서는 https://github.com/mawrkus/js-unit-testing-guide 사이트의 문서를 fork해서 한글로 번역한 것입니다.
> 자바스크립트 테스팅 관련 자료를 찾아 공부하다가 번역을 해 두면 다른 사람들에게 도움이 될 것 같아서 번역을 하기로 했습니다.
> Github이나 Open Source와 관련된 지식이 풍부하지 않아서 문서 번역이나 fork, contribution에 관한 기초 지식이 없습니다.
> 혹시 문제가 된다면 알려주시면 삭제 하겠습니다.
> 영어도 잘 못하지만 국어도 잘 못해서 하다보니 발번역이 되었습니다. 오역이 있을 것 같아서 원문을 같이 달아 두었습니다. 도움이 되시길 바랍니다.
> (아직 섹션간 링크가 동작하지 않습니다. 영어가 아닌 경우 어떻게 하는지 알아내지 못했습니다. 방법을 알아내면 적용해 두겠습니다.)

## This is a living document. New ideas are always welcome. Contribute: fork, clone, branch, commit, push, pull request.

### Disclaimer

> All the information provided has been compiled & adapted from the references cited at the end of the document.
> The guidelines are illustrated by my own examples, fruit of my personal experience writing and reviewing unit tests.
> Many thanks to all of the sources of information & contributors.

## 목차 (Table of contents)

1. 일반적인 이론들 (General principles)
  + [유닛 테스트 (Unit tests)](#unit-tests)
  + [설계 이론들 (Design principles)](#design-principles)
2. 가이드라인 (Guidelines)
  + [가능한한 TDD를 사용해라 (Whenever possible, use TDD)](#whenever-possible-use-tdd)
  + [테스트들을 적절하게 구조화 해라 (Structure your tests properly)](#structure-your-tests-properly)
  + [테스트들의 이름을 적절하게 지어라 (Name your tests properly)](#name-your-tests-properly)
  + [테스트를 코맨트 해버리지 말아라 (Don't comment out tests)](#dont-comment-out-tests)
  + [테스트에 로직을 피해라 (Avoid logic in your tests)](#avoid-logic-in-your-tests)
  + [필요없는 expectation들을 작성하지 말아라 (Don't write unnecessary expectations)](#dont-write-unnecessary-expectations)
  + [포함된 모든 테스트에 적용할 액션들을 적절하게 setup해라 (Properly setup the actions that apply to all the tests involved)](#properly-setup-the-actions-that-apply-to-all-the-tests-involved)
  + [테스트들에서 팩토리 함수들을 사용하는 것을 고려해라 (Consider using factory functions in the tests)](#consider-using-factory-functions-in-the-tests)
  + [테스팅 프레임워크 API를 학습해라 (Know your testing framework API)](#know-your-testing-framework-api)
  + [같은 테스트에서 여러 가지를 관심 갖지 말아라 (Don't test multiple concerns in the same test)](#dont-test-multiple-concerns-in-the-same-test)
  + [일반적인 케이스와 엣지케이스를 모두 커버하라 (Cover the general case and the edge cases)](#cover-the-general-case-and-the-edge-cases)
  + [TDD를 적용할 때, 항상 가장 간단한 실패하는 테스트를 작성하는 것으로 부터 시작해라 (When applying TDD, always start by writing the simplest failing test)](#when-applying-tdd-always-start-by-writing-the-simplest-failing-test)
  + [TDD를 적용할 때, 각 테스트 우선 사이클의 단계는 항상 작게 만들어라 (When applying TDD, always make small steps in each test-first cycle)](#when-applying-tdd-always-make-small-steps-in-each-test-first-cycle)
  + [동작을 테스트하고, 내부 구현은 테스트 하지 말아라 (Test the behaviour, not the internal implementation)](#test-the-behaviour-not-the-internal-implementation)
  + [모든 것을 목킹하지 말아라 (Don't mock everything)](#dont-mock-everything)
  + [모든 결함에 대한 테스트를 작성하라 (Create new tests for every defect)](#create-new-tests-for-every-defect)
  + [복잡한 사용자의 상호작용에 대한 유닛테스트를 작성하지 마라 (Don't write unit tests for complex user interactions)](#dont-write-unit-tests-for-complex-user-interactions)
  + [간단한 사용자 액션을 테스트 해라 (Test simple user actions)](#test-simple-user-actions)
  + [테스트 코드를 먼저 리뷰해라 (Review test code first)](#review-test-code-first)
  + [Practice code katas, 짝 프로그래밍으로 배워라. (Practice code katas, learn with pair programming)](#practice-code-katas-learn-with-pair-programming)
3. [참고 (References)](#references)

## 일반적인 이론들 (General principles)

### 유닛 테스트 (또는 단위 테스트) (Unit tests)

**유닛 = 작업의 단위 (Unit = Unit of work)**

이 것은 몇 개의 공개된 API를 통해 실행되는 **다수의 메소드나 클래스**가 포함될 수 있는 것으로 다음과 같은 것을 할 수 있다. (This could involve **multiple methods and classes** invoked by some public API that can:)

+ 값을 반환하거나 예외를 발생시킬 수 있다. (Return a value or throw an exception)
+ 시스템의 상태를 바꿀 수 있다. (Change the state of the system)
+ 3rd party를 호출 할 수 있다. (Make 3rd party calls)

하나의 유닛 테스트는 하나의 작업 단위의 동작을 테스트 해야 한다. 주어진 입력에 대한 어떤 완결된 결과를 기대하며 위에 나열된 어떤 것도 될 수 있다. (A unit test should test the behaviour of a unit of work: for a given input, it expects an end result that can be any of the above.)

**유닛 테스트는 분리 가능하며 서로 독립적이다. (Unit tests are isolated and independent of each other)**

+ 모든 주어진 동작은 **한가지로 그리고 하나의 테스트로** 특징지어져야 한다. (Any given behaviour should be specified in **one and only one test**)
+ 하나의 테스트의 실행이나 실행의 순서는 **다른 테스트에 영향을 줄 수 없다**. (The execution/order of execution of one test **cannot affect the others**)

코드는 이 독립성을 보장하도록 설계 된다 (아래 "설계 이론들" 참조). (The code is designed to support this independence (see "Design principles" below).)

**유닛테스트는 가벼운 테스트이다 (Unit tests are lightweight tests)**

+ 반복가능하고 (Repeatable)
+ 빠르고 (Fast)
+ 일관되고 (Consistent)
+ 작성하기 쉽고 읽기 쉽다 (Easy to write and read)

**유닛테스트들도 역시 코드이다. (Unit tests are code too)**

유닛 테스트는 테스트 되는 코드와 같은 수준의 품질을 만족해야 한다. 유닛 테스트는 좀 더 유지보수가능하고 읽기 쉽도록 재구성 할 수 있다. (They must meet the same level of quality as the code being tested. They can be refactored as well to make them more maintainable and/or readable.)

• [Back to ToC](#user-content-table-of-contents) •

### 설계 이론들 (Design principles)

좋은 유닛 테스팅의 핵심은 테스트 가능한 코드를 작성하는 것이다. 간단한 설계이론을 적용하는 것이 도움이 될 수 있다. 특히: (The key to good unit testing is to write **testable code**. Applying simple design principles can help, in particular:)

+ 좋은 naming convention을 사용하고 당신의 코드에 코멘트 해라 ("어떻게?"가 아니라 "왜?"), 코멘트가 좋지 않은 이름이나 좋지 않은 설계를 대신하면 안된다는 것을 명심해라. (Use a **good naming** convention and **comment** your code (the "why?" not the "how"), keep in mind that comments are not a substitute for bad naming or bad design)
+ 중복 코드를 피해라. (**DRY**: Don't Repeat Yourself, avoid code duplication)
+ 단일 책임: 각 객체나 함수는 반드시 한가지 작업에만 집중해야 한다. (**Single responsibility**: each object/function must focus on a single task)
+ 같은 컴포넌트에는 한 단계의 추상화만 유지해라. (예를 들어, 같은 메소드 안에서 비즈니스 로직을 더 낮은 수준의 기술적인 상세와 섞지 말아라) (Keep a **single level of abstraction** in the same component (for example, do not mix business logic with lower-level technical details in the same method))
+ 컴포넌트 간의 의존성을 최소화 하라. 캡슐화 하고, 컴포넌트간에 주고받는 정보를 최소화 해라 (**Minimize dependencies** between components: encapsulate, interchange less information between components)
+ 하드코딩 보다는 설정 가능하게 해라. 이는 테스트시에 정확히 동일한 환경을 만들지 않아도 되도록 해준다. (**Support configurability** rather than hard-coding, this prevents having to replicate the exact same environment when testing (e.g.: markup))
+ 알맞은 설계 패턴을 적용해라. 특히 의존성 주입은 객체 생성의 책임을 비즈니스 로직으로 부터 분리시키는 것을 가능하게 해준다. (Apply adequate **design patterns**, especially **dependency injection** that allows separating an object's creation responsibility from business logic)
+ 상태를 전역에서 수정 가능하도록 하는 것은 피해라. (Avoid global mutable state)

• [Back to ToC](#user-content-table-of-contents) •

## 가이드라인들 (Guidelines)

The goal of these guidelines is to make your tests:

+ **Readable**
+ **Maintainable**
+ **Trustworthy**

These are the 3 pillars of good unit testing.

All the following examples assume the usage of the [Jasmine](http://jasmine.github.io) framework.

• [Back to ToC](#user-content-table-of-contents) •

---------------------------------------

### 가능한한 TDD를 사용해라 (Whenever possible, use TDD)

TDD는 설계 프로세스지 테스팅 프로세스가 아니다. TDD는 소프트웨어 컴포넌트들을 ("유닛들") 상호작용 통해서 설계하는 믿음직한 방법이다. 그래서 그 것들의 동작이 유닛테스트를 통해서 특징지어진다. (TDD is a _design process_, not a testing process. TDD is a robust way of designing software components ("units") interactively so that their behaviour is specified through unit tests.)

어떻게? 왜? (How? Why?)

#### 테스트 우선 사이클 (Test-first cycle)

1. 하나의 간단한 실패하는 테스트를 작성한다. (Write a simple failing test)
2. 그 테스트를 통과시켜줄 최소한의 코드를 작성한다. (Make the test pass by writing the minimum amount of code)
3. 설계 이론들과 패턴들을 적용해 코드를 재구성한다. (Refactor the code by applying design principles/patterns)

2번 단계에서는 품질에 신경 쓰지 말아라. (During phase 2, don't bother with quality.)

#### 테스트 우선 사이클의 결과 (Consequences of the test-first cycle)

+ 테스트를 먼저 작성하는 것은 코드 디자인을 좀 더 테스트 가능하도록 해준다. (Writing a test first makes the code design more testable)
+ 요구되는 기능을 구현하기 위해 필요한 양의 코드만 작성하는 것은 전체 코드를 작게 유지시켜주기 때문에 좀 더 유지보수하기 쉬워진다. (Writing just the amount of code needed to implement the required functionality makes the resulting codebase minimal, thus more maintainable)
+ 전체 코드는 리펙토링 기법을 사용해 개선될 수 있고, 테스트들은 당신에게 새로운 코드가 기존 기능들에 영향을 주지 않는 다는 것에 대한 확신을 준다. (The codebase can be enhanced using refactoring mechanisms, the tests give you confidence that the new code is not modifying the existing functionalities)
+ 사이클마다 코드를 정리하는 것은 전체 코드를 유지보수하기 쉽게 만들어 주고, 코드를 자주 조금 씩 바꾸는것이 쉬워진다. (Cleaning the code in each cycle makes the codebase more maintainable, it is much cheaper to change the code frequently and in small increments)
+ 개발자에 대한 빠른 피드백을 통해 당신이 아무것도 망가뜨리지 않았음을 알 수 있고, 당신이 시스템을 좋은 방향으로 진화시키고 있음을 알게 해준다. (Fast feedback for the developers, you know that you don't break anything and that you are evolving the system in a good direction)
+ 새로운 기능 추가나 버그 수정, 새로운 설계 적용하는데 자신감을 준다. (Generates confidence to add features, fix bugs, or explore new designs)

테스트 우선 접근이 없이 작성된 코드는 보통 테스트하기 힘들다. (Note that code written without a test-first approach is often very hard to test!)

• [Back to ToC](#user-content-table-of-contents) •

### 테스트들을 적절하게 구조화 해라 (Structure your tests properly)

테스트들을 논리적인 하위 셋으로 구조화하기 위해 테스트 슈트 내부에 서브 테스트 슈트를 생성하는 것을 망설이지 말아라. (Don't hesitate to nest your suites to structure logically your tests in subsets.)

**:(**

```js
describe('A set of functionalities', () => {
  it('a set of functionalities should do something nice', () => {
  });

  it('a subset of functionalities should do something great', () => {
  });

  it('a subset of functionalities should do something awesome', () => {
  });

  it('another subset of functionalities should also do something great', () => {
  });
});
```

**:)**

```js
describe('A set of functionalities', () => {
  it('should do something nice', () => {
  });

  describe('A subset of functionalities', () => {
    it('should do something great', () => {
    });

    it('should do something awesome', () => {
    });
  });

  describe('Another subset of functionalities', () => {
    it('should also do something great', () => {
    });
  });
});
```

• [Back to ToC](#user-content-table-of-contents) •

### 테스트들의 이름을 적절하게 지어라 (Name your tests properly)

테스트들의 이름은 간결하고 명백하고 서술적으로 올바른 영어로 적어야 한다. 스팩 러너의 output을 읽고, 이해 가능한지 확인해라! 다른 누군가도 그것을 읽는다는 것을 염두에 두어라. 테스트는 코드의 살아있는 문서가 될 수 있다. (Tests names should be concise, explicit, descriptive and in correct English. Read the output of the spec runner and verify that it is understandable! Keep in mind that someone else will read it too. Tests can be the live documentation of the code.)

**:(**

```js
describe('myGallery', () => {
  it('init set correct property when called (thumb size, thumbs count)', () => {
  });

  // ...
});
```

**:)**

```js
describe('The Gallery instance', () => {
  it('should properly calculate the thumb size when initialized', () => {
  });

  it('should properly calculate the thumbs count when initialized', () => {
  });

  // ...
});
```

당신이 테스트의 이름들을 절절하게 작성하는데 도움이 되기 위해, 당신은 "unit of work - scenario/context - expected behaviour" 패턴을 이용할 수 있다. (In order to help you write test names properly, you can use the **"unit of work - scenario/context - expected behaviour"** pattern:)

```js
describe('[unit of work]', () => {
  it('should [expected behaviour] when [scenario/context]', () => {
  });
});
```

같은 시나리오나 문맥에 여러 테스트가 있는 경우 다음과 같은 패턴을 사용할 수 있다. (Or whenever you have many tests that follow the same scenario or are related to the same context:)

```js
describe('[unit of work]', () => {
  describe('when [scenario/context]', () => {
    it('should [expected behaviour]', () => {
    });
  });
});
```

예를 들면, (For example:)

**:) :)**

```js
describe('The Gallery instance', () => {
  describe('when initialized', () => {
    it('should properly calculate the thumb size', () => {
    });

    it('should properly calculate the thumbs count', () => {
    });
  });

  // ...
});
```

• [Back to ToC](#user-content-table-of-contents) •

### 테스트를 코맨트 해버리지 말아라. (Don't comment out tests)

(테스트는 존재하는 이유가 있다. 아니면 지워라.) Never. Ever. Tests have a reason to be or not.

절대로 너무 느리디거나 복잡하다거나 false negatives 발생시킨 다는 이유로 코맨트 해버리지 말아라. 대신 빠르게 하거나 간단하게 만들거나 신뢰성 있게 만들어라. 그렇지 않으면 지워버려라. (Don't comment them out because they are too slow, too complex or produce false negatives. Instead, make them fast, simple and trustworthy. If not, remove them completely.)

• [Back to ToC](#user-content-table-of-contents) •

### 테스트에 로직을 피해라. (Avoid logic in your tests)

항상 단순한 문장만 사용하고, 루프나 조건문을 사용하지 마라. 만약 그렇다는 건 테스트 자체에 버그를 추가하는 것이다. (Always use simple statements. Loops and conditionals must not be used. If they are, you add a possible entry point for bugs in the test itself:)

+ 조건문: 당신은 테스트가 어떤 패스를 취할지 모른다. (Conditionals: you don't know which path the test will take)
+ 루프: 테스트들 간에 상태를 공유하는 것이다. (Loops: you could be sharing state between tests)

**:(**

```js
it('should properly sanitize strings', () => {
  let result;
  const testValues = {
    'Avion'         : 'Avi' + String.fromCharCode(243) + 'n',
    'The-space'     : 'The space',
    'Weird-chars-'  : 'Weird chars!!',
    'file-name.zip' : 'file name.zip',
    'my-name.zip'   : 'my.name.zip'
  };

  for (result in testValues) {
    expect( sanitizeString(testValues[result]) ).toEqual(result);
  }
});
```

**:)**

```js
it('should properly sanitize strings', () => {
  expect( sanitizeString('Avi'+String.fromCharCode(243)+'n') ).toEqual('Avion');
  expect( sanitizeString('The space') ).toEqual('The-space');
  expect( sanitizeString('Weird chars!!') ).toEqual('Weird-chars-');
  expect( sanitizeString('file name.zip') ).toEqual('file-name.zip');
  expect( sanitizeString('my.name.zip') ).toEqual('my-name.zip');
});
```

확인 사항 각각을 별도의 테스트로 작성하는 것이 좋다. 그러면 모든 가능한 상황에 대한 괜찮은 결과를 제공해 주고 가독성과 유지보수성을 개선해 줄 것이다. (Better: write a test for each type of sanitization. It will give a nice output of all possible cases, improving readability and maintainability.)

**:) :)**

```js
it('should sanitize a string containing non-ASCII chars', () => {
  expect( sanitizeString('Avi'+String.fromCharCode(243)+'n') ).toEqual('Avion');
});

it('should sanitize a string containing spaces', () => {
  expect( sanitizeString('The space') ).toEqual('The-space');
});

it('should sanitize a string containing exclamation signs', () => {
  expect( sanitizeString('Weird chars!!') ).toEqual('Weird-chars-');
});

it('should sanitize a filename containing spaces', () => {
  expect( sanitizeString('file name.zip') ).toEqual('file-name.zip');
});

it('should sanitize a filename containing more than one dot', () => {
  expect( sanitizeString('my.name.zip') ).toEqual('my-name.zip');
});
```

• [Back to ToC](#user-content-table-of-contents) •

### 필요없는 expectation들을 작성하지 말아라 (Don't write unnecessary expectations)

유닛테스트는 어떤 동작이 어떻게 작동해야 하는지에 대한 설계 스팩지, 코드가 어떤 동작하는 데 일어나는 모든 것을 관찰하는 것들의 목록은 아니라는 것을 기억해라. (Remember, unit tests are a design specification of how a certain *behaviour* should work, not a list of observations of everything the code happens to do.)

**:(**

```js
it('should multiply the number passed as parameter and subtract one', () => {
  const multiplySpy = spyOn(Calculator, 'multiple').and.callThrough();
  const subtractSpy = spyOn(Calculator, 'subtract').and.callThrough();

  const result = Calculator.compute(21.5);

  expect(multiplySpy).toHaveBeenCalledWith(21.5);
  expect(subtractSpy).toHaveBeenCalledWith(43, 1);
  expect(result).toBe(42);
});
```

**:)**

```js
it('should multiply the number passed as parameter and subtract one', () => {
  const result = Calculator.compute(21.5);
  expect(result).toBe(42);
});
```

이렇게 하면 유지보수하기 쉬워진다. 당신의 테스트는 더이상 상세 구현에 얽매이지 않는다. (This will improve maintainability. Your test is no longer tied to implementation details.)

• [Back to ToC](#user-content-table-of-contents) •

### 포함된 모든 테스트에 적용할 액션들을 적절하게 setup해라. (Properly setup the actions that apply to all the tests involved)

**:(**

```js
describe('Saving the user profile', () => {
  let profileModule;
  let notifyUserSpy;
  let onCompleteSpy;

  beforeEach(() => {
    profileModule = new ProfileModule();
    notifyUserSpy = spyOn(profileModule, 'notifyUser');
    onCompleteSpy = jasmine.createSpy();
  });

  it('should send the updated profile data to the server', () => {
    jasmine.Ajax.install();

    profileModule.save();

    const request = jasmine.Ajax.requests.mostRecent();

    expect(request.url).toBe('/profiles/1');
    expect(request.method).toBe('POST');
    expect(request.data()).toEqual({ username: 'mawrkus' });

    jasmine.Ajax.uninstall();
  });

  it('should notify the user', () => {
    jasmine.Ajax.install();

    profileModule.save();

    expect(notifyUserSpy).toHaveBeenCalled();

    jasmine.Ajax.uninstall();
  });

  it('should properly execute the callback passed as parameter', () => {
    jasmine.Ajax.install();

    profileModule.save(onCompleteSpy);

    jasmine.Ajax.uninstall();

    expect(onCompleteSpy).toHaveBeenCalled();
  });
});
```

셋업 코드는 모든 테스트에 적용되어야 한다. (The setup code should apply to all the tests:)

**:)**

```js
describe('Saving the user profile', () => {
  let profileModule;

  beforeEach(() => {
    jasmine.Ajax.install();
    profileModule = new ProfileModule();
  });

  afterEach( () => {
    jasmine.Ajax.uninstall();
  });

  it('should send the updated profile data to the server', () => {
    profileModule.save();

    const request = jasmine.Ajax.requests.mostRecent();

    expect(request.url).toBe('/profiles/1');
    expect(request.method).toBe('POST');

  });

  it('should notify the user', () => {
    spyOn(profileModule, 'notifyUser');

    profileModule.save();

    expect(profileModule.notifyUser).toHaveBeenCalled();
  });

  it('should properly execute the callback passed as parameter', () => {
    const onCompleteSpy = jasmine.createSpy();

    profileModule.save(onCompleteSpy);

    expect(onCompleteSpy).toHaveBeenCalled();
  });
});
```

읽기 쉽고 유지보수 가능하게 유지하기 위해서 setup 코드를 최소한으로 유지하는 것을 고려해라. (Consider keeping the setup code minimal to preserve readability and maintainability.)

• [Back to ToC](#user-content-table-of-contents) •

### 테스트들에서 팩토리 함수들을 사용하는 것을 고려해라. (Consider using factory functions in the tests)

팩토리들은 아래와 같은 이점이 있다. (Factories can:)

- 셋업 코드가 줄어들도록 도와준다. 특히 의존성 주입을 사용한다면 더 그렇다. (help reduce the setup code, especially if you use dependency injection)
- 셋업 대신에 테스트 자체메서 함수 호출 만으로 생성되도록 해줘서, 각 테스트들을 읽기 쉽도록 만들어준다. (make each test more readable, since the creation is a single function call that can be in the test itself instead of the setup)
- 새로운 객체를 생성할 때 유연함을 제공해준다. (예를 들면, 초기 상태를 설정하기) (provide flexibility when creating new instances (setting an initial state, for example))

There's a trade-off to find here between applying the DRY principle and readability.

**:(**

```js
describe('User profile module', () => {
  let profileModule;
  let pubSub;

  beforeEach(() => {
    let element = document.getElementById('my-profile');
    pubSub = new PubSub({ sync: true });

    profileModule = new ProfileModule({
      element,
      pubSub,
      likes: 0
    });
  });

  it('should publish a topic when a new "like" is given', () => {
    spyOn(pubSub, 'notify');
    profileModule.incLikes();
    expect(pubSub.notify).toHaveBeenCalledWith('likes:inc', { count: 1 });
  });

  it('should retrieve the correct number of likes', () => {
    profileModule.incLikes();
    profileModule.incLikes();
    expect(profileModule.getLikes()).toBe(2);
  });
});
```

**:)**

```js
describe('User profile module', () => {  
  function createProfileModule({
    element = document.getElementById('my-profile'),
    likes = 0,
    pubSub = new PubSub({ sync: true })
  })
  {
    return new ProfileModule({ element, likes, pubSub });
  }

  it('should publish a topic when a new "like" is given', () => {
    const pubSub = jasmine.createSpyObj('pubSub', ['notify']);
    const profileModule = createProfileModule({ pubSub });

    profileModule.incLikes();

    expect(pubSub.notify).toHaveBeenCalledWith('likes:inc');
  });

  it('should retrieve the correct number of likes', () => {
    const profileModule = createProfileModule({ likes: 40 });

    profileModule.incLikes();
    profileModule.incLikes();

    expect(profileModule.getLikes()).toBe(42);
  });
});
```

Factories are particularly useful when dealing with the DOM:

**:(**

```js
describe('The search component', () => {
  describe('when the search button is clicked', () => {
    let container;
    let form;
    let searchInput;
    let submitInput;

    beforeEach(() => {
      fixtures.inject(`<div id="container">
        <form class="js-form" action="/search">
          <input type="search">
          <input type="submit" value="Search">
        </form>
      </div>`);

      container = document.getElementById('container');
      form = container.getElementsByClassName('js-form')[0];
      searchInput = form.querySelector('input[type=search]');
      submitInput = form.querySelector('input[type=submith]');
    });

    it('should validate the text entered', () => {
      const search = new Search({ container });
      spyOn(search, 'validate');

      search.init();

      input(searchInput, 'peace');
      click(submitInput);

      expect(search.validate).toHaveBeenCalledWith('peace');
    });

    // ...
  });
});
```

**:)**

```js
function createHTMLFixture() {
  fixtures.inject(`<div id="container">
    <form class="js-form" action="/search">
      <input type="search">
      <input type="submit" value="Search">
    </form>
  </div>`);

  const container = document.getElementById('container');
  const form = container.getElementsByClassName('js-form')[0];
  const searchInput = form.querySelector('input[type=search]');
  const submitInput = form.querySelector('input[type=submith]');

  return {
    container,
    form,
    searchInput,
    submitInput
  };
}

describe('The search component', () => {
  describe('when the search button is clicked', () => {
    it('should validate the text entered', () => {
      const { container, form, searchInput, submitInput } = createHTMLFixture();
      const search = new Search({ container });
      spyOn(search, 'validate');

      search.init();

      input(searchInput, 'peace');
      click(submitInput);

      expect(search.validate).toHaveBeenCalledWith('peace');
    });

    // ...
  });
});
```

• [Back to ToC](#user-content-table-of-contents) •

### 테스팅 프레임워크 API를 학습해라. (Know your testing framework API)

테스팅 프레임워크/라이브러리의 API 문서는 가까이 두어라. (The API documentation of the testing framework/library should be your bedside book!)

API에 대한 좋은 지식을 갖는 것은 일반적으로 테스트 코드의 사이즈나 복잡도를 줄여주고 개발을 돕는다. (Having a good knowledge of the API can help you in reducing the size/complexity of your test code and, in general, help you during development. A simple example:)

**:(**

```js
it('should call a method with the proper arguments', () => {
  const foo = {
    bar: jasmine.createSpy(),
    baz: jasmine.createSpy()
  };

  foo.bar('qux');

  expect(foo.bar).toHaveBeenCalled();
  expect(foo.bar.calls.argsFor(0)).toEqual(['qux']);
});

/*it('should do more but not now', () => {
});

it('should do much more but not now', () => {
});*/
```

**:)**

```js
fit('should call once a method with the proper arguments', () => {
  const foo = jasmine.createSpyObj('foo', ['bar', 'baz']);

  foo.bar('baz');

  expect(foo.bar).toHaveBeenCalledWith('baz');
});

it('should do something else but not now', () => {
});

it('should do something else but not now', () => {
});
```

#### Note

The handy `fit` function used in the example above allows you to execute only one test without having to comment out all the tests below. `fdescribe` does the same for test suites. This could help save a lot of time when developing.

More information on the [Jasmine website](http://jasmine.github.io).

• [Back to ToC](#user-content-table-of-contents) •

### 같은 테스트에서 여러 가지를 관심 갖지 말아라. (Don't test multiple concerns in the same test)

어떤 메소드가 몇가지 종료된 결과를 가진다면, 각각은 분리해서 테스트해라. 버그가 발생할 때마다 문제가 발생한 코드의 위치를 찾는데 돔움이 될 것이다. (If a method has several end results, each one should be tested separately. Whenever a bug occurs, it will help you locate the source of the problem.)

**:(**

```js
it('should send the profile data to the server and update the profile view properly', () => {
  // expect(...)to(...);
  // expect(...)to(...);
});
```

**:)**

```js
it('should send the profile data to the server', () => {
  // expect(...)to(...);
});

it('should update the profile view properly', () => {
  // expect(...)to(...);
});
```

테스트 이름에 And 나 Or 가 들어가면.. 좋지 않은 징조이다. (Beware that writing "AND" or "OR" when naming your test smells bad...)

• [Back to ToC](#user-content-table-of-contents) •

### 일반적인 케이스와 엣지케이스를 모두 커버하라. (Cover the general case and the edge cases)

이상동작은 보통 엣지 케이스에서 발생한다. 당신의 테스트들은 당신의 코드에 대한 살아있는 문서가 될 수 있음을 기억해라. ("Strange behaviour" usually happens at the edges... Remember that your tests can be the live documentation of your code.)

**:(**

```js
it('should properly calculate a RPN expression', () => {
  const result = RPN('5 1 2 + 4 * - 10 /');
  expect(result).toBe(-0.7);
});
```

**:)**

```js
describe('The RPN expression evaluator', () => {
  it('should return null when the expression is an empty string', () => {
    const result = RPN('');
    expect(result).toBeNull();
  });

  it('should return the same value when the expression holds a single value', () => {
    const result = RPN('42');
    expect(result).toBe(42);
  });

  it('should properly calculate an expression', () => {
    const result = RPN('5 1 2 + 4 * - 10 /');
    expect(result).toBe(-0.7);
  });

  it('should throw an error whenever an invalid expression is passed', () => {
    const compute = () => RPN('1 + - 1');
    expect(compute).toThrow();
  });
});
```

• [Back to ToC](#user-content-table-of-contents) •

### TDD를 적용할 때, 항상 가장 간단한 실패하는 테스트를 작성하는 것으로 부터 시작해라. (When applying TDD, always start by writing the simplest failing test)

**:(**

```js
it('should suppress all chars that appear multiple times', () => {
  expect( keepUniqueChars('Hello Fostonic !!') ).toBe('HeFstnic');
});
```

**:)**

```js
it('should return an empty string when passed an empty string', () => {
  expect( keepUniqueChars('') ).toBe('');
});
```

거기서 부터 점차 기능들을 만들어라. (From there, start building the functionalities incrementally.)

• [Back to ToC](#user-content-table-of-contents) •

### TDD를 적용할 때, 각 테스트 우선 사이클의 단계는 항상 작게 만들어라. (When applying TDD, always make small steps in each test-first cycle)

테스트 슈트는 간단한 케이스로 부터 복잡한 케이스 순으로 만들어라. 점진적인 설계를 염두에 두어라. 소프트웨어를 빨리 제공하고, 점진적이고, 짧은 반복주기를 생각해라. (Build your tests suite from the simple case to the more complex ones. Keep in mind the incremental design. Deliver software fast, incrementally, and in short iterations.)

**:(**

```js
it('should return null when the expression is an empty string', () => {
  const result = RPN('');
  expect(result).toBeNull();
});

it('should properly calculate a RPN expression', () => {
  const result = RPN('5 1 2 + 4 * - 10 /');
  expect(result).toBe(-0.7);
});
```

**:)**

```js
describe('The RPN expression evaluator', () => {
  it('should return null when the expression is an empty string', () => {
    const result = RPN('');
    expect(result).toBeNull();
  });

  it('should return the same value when the expression holds a single value', () => {
    const result = RPN('42');
    expect(result).toBe(42);
  });

  describe('Additions-only expressions', () => {
    it('should properly calculate a simple addition', () => {
      const result = RPN('41 1 +');
      expect(result).toBe(42);
    });

    it('should properly calculate a complex addition', () => {
      const result = RPN('2 9 + 15 3 + + 7 6 + +');
      expect(result).toBe(42);
    });
  });

  // ...

  describe('Complex expressions', () => {
    it('should properly calculate an expression containing all 4 operators', () => {
      const result = RPN('5 1 2 + 4 * - 10 /');
      expect(result).toBe(-0.7);
    });
  });
});
```

• [Back to ToC](#user-content-table-of-contents) •

### 동작을 테스트하고, 내부 구현은 테스트 하지 말아라. (Test the behaviour, not the internal implementation)

**:(**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager._users[0].name).toBe('Dr. Falker');
  expect(userManager._users[0].password).toBe('Joshua');
});
```

같은 수준의 API를 테스트하는 것이 좋은 접근이다. (A better approach is to test at the same level of the API:)

**:)**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager.loginUser('Dr. Falker', 'Joshua')).toBe(true);
});
```

장점: (Advantage:)

+ 테스트를 재구성할 때 클래스나 개게의 내부 구현의 변경은 필요 없다. (Changing the internal implementation of a class/object will not necessarily force you to refactor the tests)

단점: (Disadvantage:)

+ 테스트가 실패했을 때, 수정하기 위해서 코드의 어느 부분이 문제인지 디버깅 해야 한다. (If a test is failing, we might have to debug to know which part of the code needs to be fixed)

균형을 찾아야 한다. (Here, a balance has to be found, unit-testing some key parts can be beneficial.)

• [Back to ToC](#user-content-table-of-contents) •

### 모든 것을 목킹하지 말아라. (Don't mock everything)

**:(**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = jasmine.createSpyObj('storage', ['setItem', 'getItem']);
      storage.getItem.and.returnValue('1'); // ouch.

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

This test fails, because the survey is considered disabled. Let's fix this:

**:)**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = jasmine.createSpyObj('storage', ['setItem', 'getItem']);
      storage.getItem.and.callFake(key => {
        switch (key) {
          case 'page-visited':
            return '1';

          case 'survey-disabled':
            return null;
        }

        return null;
      }); // ouch.

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

This will work... but needs a lot of code. Let's try a simpler approach:

**:(**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = window.localStorage; // ouch.
      storage.setItem('page-visited', '1');

      const surveyManager = new SurveyManager();
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

We created a permanent storage of data. What happens if we do not properly clean it?
We might affect the other tests. Let's fix this:

**:) :)**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      // Note: have a look at https://github.com/tatsuyaoiw/webstorage
      const storage = new MemoryStorage(); // yeah.
      storage.setItem('page-visited', '1');

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });
});
```

The `MemoryStorage` used here does not persist data. Nice and easy. Minimal. No side effects.

#### Takeaway

의존하는 것들은 여전히 실제 객체일 수 있다는 것을 염두에 두어라. 할 수 있다면 모든 것을 목킹하지 말아라. (The idea to keep in mind is that *dependencies can still be "real" objects*. Don't mock everything because you can.)
특히 이런 경우는 실제 객체를 사용해라. (In particular, consider using the "real" version of the objects if:)

- 테스트 설정이 간단하고 쉬운 경우. (it leads to a simple, nice and easy tests setup)
- 테스트들 간의 상태를 공유하지 않는 경우. (it does not create a shared state between the tests, causing unexpected side effects)
- Ajax 호출이나, API 호출, 브라우저 새로고침 같은 걸 하지 않는 경우 (the code being tested does not make AJAX requests, API calls or browser page reloads)
- 실행 속도가 느리지 않은 경우 (the speed of execution of the tests stays *within the limits you fixed*)

• [Back to ToC](#user-content-table-of-contents) •

### 모든 결함에 대한 테스트를 작성하라. (Create new tests for every defect)

결함이 발견된 경우 코드를 수정하기 전에 그 결함을 다시 재현할 수 있는 테스트를 작성해라. 거기서 부터 평소처럼 TDD를 이용해 수정해라. (Whenever a bug is found, create a test that replicates the problem **before touching any code**. From there, you can apply TDD as usual to fix it.)

• [Back to ToC](#user-content-table-of-contents) •

### 복잡한 사용자의 상호작용에 대한 유닛테스트를 작성하지 마라. (Don't write unit tests for complex user interactions)

Examples of complex user interactions:

+ Filling a form, drag and dropping some items then submitting the form
+ Clicking a tab, clicking an image thumbnail then navigating through a gallery of images previously loaded from a database
+ (...)

이런 상호 작용들은 많은 작업들의 단위가 포함될 것이고 좀더 높은 수준 기능 테스트에서 다루어 져야한다. 그 테스트는 실행에 많은 시간이 소요된다. 그 테스트는 깨지지 쉽고.. 매번 실패할 때마다 디버깅 해야 한다. (These interactions might involve many units of work and should be handled at a higher level by **functional tests**. They will take more time to execute. They could be flaky (false negatives) and they need debugging whenever a failure is reported.)

For functional testing, consider using a test automation framework ([Selenium](http://docs.seleniumhq.org/), ...) or QA manual testing.

• [Back to ToC](#user-content-table-of-contents) •

### 간단한 사용자 액션을 테스트 해라. (Test simple user actions)

Example of simple user actions:

+ Clicking on a link that toggles the visibility of a DOM element
+ Submitting a form that triggers the form validation
+ (...)

These actions can be easily tested **by simulating DOM events**, for example:

```js
describe('When clicking on the "Preview profile" link', () => {
  it('should show the profile preview if it is hidden', () => {
    const previewLink = document.createElement('a');
    const profileModule = createProfileModule({ previewLink, previewIsVisible: false });

    spyOn(profileModule, 'showPreview');

    click(previewLink);

    expect(profileModule.showPreview).toHaveBeenCalled();
  });

  it('should hide the profile preview if it is displayed', () => {
    const previewLink = document.createElement('a');
    const profileModule = createProfileModule({ previewLink, previewIsVisible: true });

    spyOn(profileModule, 'hidePreview');

    click(previewLink);

    expect(profileModule.hidePreview).toHaveBeenCalled();
  });
});
```

• [Back to ToC](#user-content-table-of-contents) •

### 테스트 코드를 먼저 리뷰해라 (Review test code first)

코드를 리뷰할 때 항상 코드의 테스트들을 읽는 것 부터 시작해라. 테스트는 당신이 파고 들 수 있는 코드의 작은 사용 케이스이다. (When reviewing code, always start by reading the code of the tests. Tests are mini use cases of the code that you can drill into.)

이 것은 개발자의 의도를 빠르게 이해할 수 있도록 도와준다 (테스트 이름만 봐도 알 수 있을지 모른다). (It will help you understand the intent of the developer very quickly (could be just by looking at the names of the tests).)

• [Back to ToC](#user-content-table-of-contents) •

### Practice code katas, 짝 프로그래밍으로 배워라. (Practice code katas, learn with pair programming)

경험 만이 선생님이기 때문에, 극단적으로, 탁월함은 연습에서 나온다. 이론을 적용하고 적용하고 또 적용해 봐라. 더 나아지기 위해 항상 피드백을 이용해라. (Because experience is the _only_ teacher. Ultimately, greatness comes from practicing; applying the theory over and over again, using feedback to get better every time.)

• [Back to ToC](#user-content-table-of-contents) •

## 참고 (References)

+ Roy Osherove - "JS Unit Testing Good Practices and Horrible Mistakes" : https://www.youtube.com/watch?v=iP0Vl-vU3XM
+ Enrique Amodeo - "Learning Behavior-driven Development with JavaScript" : https://www.packtpub.com/application-development/learning-behavior-driven-development-javascript
+ Steven Sanderson - "Writing Great Unit Tests: Best and Worst Practices" : http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/
+ Rebecca Murphy - "Writing Testable JavaScript" : http://alistapart.com/article/writing-testable-javascript
+ YUI Team - "Writing Effective JavaScript Unit Tests with YUI Test" : http://yuiblog.com/blog/2009/01/05/effective-tests/
+ Colin Snover - "Testable code best practices" : http://www.sitepen.com/blog/2014/07/11/testable-code-best-practices/
+ Miško Hevery - "The Clean Code Talks -- Unit Testing" : https://www.youtube.com/watch?v=wEhu57pih5w
+ Addy Osmani - "Learning JavaScript Design Patterns" : http://addyosmani.com/resources/essentialjsdesignpatterns/book/
+ José Armesto - "Unit Testing sucks (and it’s our fault) " : https://www.youtube.com/watch?v=GZ9iZsMAZFQ
+ Clean code cheat sheet: http://www.planetgeek.ch/2014/11/18/clean-code-cheat-sheet-v-2-4/

• [Back to ToC](#user-content-table-of-contents) •

## Contributors

Ruben Norte: https://github.com/rubennorte

• [Back to ToC](#user-content-table-of-contents) •
