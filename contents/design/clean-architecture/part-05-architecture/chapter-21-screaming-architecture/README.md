# SCREAMING ARCHITECTURE

건물의 아키텍처를 보고 집(정문, 거실, 현관 등)인지 도서관(사서를 위한 공간, 독서 공간, 작은 회의실 등)인지 알 수 있듯이, 애플리케이션의 아키텍처(최상위 디렉터리 구조, 최상위 패키지에 담긴 소스 파일)를
볼 때 어떤 시스템인지 알 수 있어야 한다.

## 아키텍처의 테마

주택이나 도서관의 계획서가 해당 건축물의 유스케이스에 대해 소리치는 것처럼, 소프트웨어 애플리케이션의 아키텍처도 애플리케이션의 유스케이스에 대해 소리쳐야 한다.

## 아키텍처의 목적

좋은 아키텍처는 프레임워크, 데이터베이스, 웹 서버, 그리고 여타 개발 환경 문제나 도구에 대해 결정을 미룰 수 있도록 한다. 좋은 아키텍처는 유스케이스에 중점을 두며, 지엽적인 관심사에 대한 결합은 분리시킨다.

## 하지만 웹은?

웹은 아키텍처가 아니고 전달 메커니즘(입출력 장치)이며, 애플리케이션 아키텍처에서도 이와 같이 다뤄야 한다. 애플리케이션이 웹을 통해 전달된다는 사실은 세부사항이며, 시스템 구조를 지배해선 안 된다.

## 프레임워크는 도구일 뿐, 삶의 방식은 아니다

프레임워크가 아키텍처의 중심을 차지아는 일을 막을 수 있는 전략을 개발하라.

## 테스트하기 쉬운 아키텍처

프레임워크를 전혀 준비하지 않더라도 필요한 유스케이스 전부에 대해 유닛 테스트를 할 수 있어야 한다. 테스트를 수행하는데 웹 서버 또는 데이터베이스가 연결되어 있어야만 수행가능해서는 안 된다.

## 결론

새로 합류한 프로그래머는 시스템이 어떻게 전달될지 알지 못한 상태에서도 시스템의 모든 유스케이스를 이해할 수 있어야 한다.