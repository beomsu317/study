# STRUCTURED PROGRAMMING

순차(sequence), 분기(selection), 반복(iteration) 세 가지 구조만으로 표형할 수 있다.

* 데이크스트라는 단순한 열거법을 이용해 순차 구문이 올바름을 입증하였다.
* 데이크스트라는 열거법을 재적용하는 방식으로 처리했다. 결과적으로 두 경로가 수학적으로 적절한 결과를 만들어내면, 증명은 신뢰할 수 있다.
* 데이크스트라는 반복의 올바름을 증명하기 위해 귀납법을 사용했다. 이 경우에도 시작 조건과 종료 조건에도 열거법을 사용해 증명했다.

이러한 증명은 복잡했지만, 프로그램에서도 정리에 대한 유클리드 계층구조를 만들 수 있을 거라고 생각했다.

## 기능적 분해

구조적 프로그래밍을 통해 모듈을 증명 가능한 더 작은 단위로 재귀적으로 분해할 수 있게 되었다. 즉, 대규모 시스템을 모듈과 컴포넌트로 나눌 수 있었고, 더 나아가 모듈과 컴포넌트는 입증할 수 있는 아주 작은
기능들로 세분화할 수 있었다.

## 엄밀한 증명은 없었다

프로그램 관점에서 유클리드 계층 구조는 만들어지지 않았다. 하지만 무언가 올바른지 입증할 때 유클리드 방식 같이 수학적인 증명이 아닌 과학적인 방법으로도 가능하다.

## 과학이 구출하다

과학은 서술된 내용이 사실임을 증명하는 것이 아닌 서술이 틀렸음을 증명하는 방식으로 동작한다. 각고의 노력으로도 반례를 들 수 없는 서술이 있다면, 목표에 부합할 만큼은 참이라고 본다. 즉, 수학은 증명 가능한
서술이 참임을 입증하는 원리라면 과학은 증명 가능한 서술이 거짓임을 입증하는 원리라 할 수 있다.

## 테스트

프로그램이 잘못되었음을 테스트를 통해 증명할 수 있지만, 프로그램이 맞다고는 증명할 수 없다. 테스트에 충분히 노력을 들였다면 테스트가 보장할 수 있는 것은 프로그램이 목표에 부합할 만큼 충분히 참이라고 여길 수
있게 해주는 것이 전부다.

## 결론

구조적 프로그래밍이 현재까지 가치있는 이유는 프로그래밍에서 반증 가능한 단위를 만들어 낼 수 있는 능력 때문이다. 또한 아키텍처 관점에서 기능적 분해를 최고의 실천법 중 하나로 여기기 때문이기도 하다. 소프트웨어
아키텍트는 모듈, 컴포넌트, 서비스가 쉽게 반증 가능하도록(테스트하기 쉽도록) 만들어야 한다.