# How to Write Good Tests

## TDD(Test Driven Development)

TDD는 기본적으로 테스트 주도 개발 스타일이다.

함수를 구현하기 전 테스트 케이스를 작성하는 것이 주 원칙이다. (유닛 테스트만)

1. 함수 시그니처를 작성
2. 함수에 대한 테스트 케이스 작성
3. 함수 로직을 구현하여 테스트 케이스 통과

* 하나의 테스트 케이스에 하나의 assertion만 존재해야 한다.
* 테스트 케이스에 실패하는 원인을 즉시 알아야 한다.
* 때때로 여러 assertions를 사용할 필요가 있다.
    * 예를 들어, 로딩바를 검증할 때 로딩 status에 대한 여러 assertion이 필요하다.

## What makes a good tests?

1. Scope
    * 테스트를 원하는 함수에 실제 코드가 테스트 케이스에서 다루어졌는지
2. Speed
    * 테스트 케이스가 얼마나 빠른지
3. Fidelity
    * 테스트 케이스가 실제 시나리오랑 얼마나 유사한지
4. Not a flaky test
    * flaky test란 간헐적으로 성공하거나 실패하는 것을 의미
5. 다른 테스트의 출력이 원하는 테스트에 의존되어서는 안된다.

## How many test cases should you write?

* 필요한 만큼 테스트 적게, 필요한 만큼 많이
* Equivalent class는 함수가 가져야 할 테스트의 양을 결정하는데 도움을 준다.

### Equivalent Class Example

Username, Password, Confirm Password를 입력받는 회원가입 폼이 있다고 하자. 하나의 Equivalent class를 통해 검증될 수 있다. 3가지 폼을 모두 입력해 성공하거나, 셋 중 한
가지를 입력하지 않아 실패하는 케이스도 하나의 Equivalent class를 통해 검증할 수 있다.

## References

* [How to Write Good Tests - Android Testing - Part 2](https://www.youtube.com/watch?v=rew86GST0g0&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=2)