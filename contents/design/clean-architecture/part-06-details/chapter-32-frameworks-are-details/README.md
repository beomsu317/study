# FRAMEWORKS ARE DETAILS

프레임워크는 아키텍처가 될 수 없다.

## 프레임워크 제작자

프레임워크 제작자는 자신의 해결해야 할 고유한 문제를 알고 있다. 그리고 그러한 문제를 해결하기 위해 프레임워크를 만든다. 당신의 문제를 해결하기 위해서가 아니다. 하지만 당신의 문제는 프레임워크가 풀려는 문제와 꽤
많이 겹칠 수 있다. 겹치는 영역이 크면 클수록 프레임워크는 실제로 더 유용하다.

## 혼인 관계의 비대칭성

프레임워크 제작자는 우리가 만들 소프트웨어와 프렘워크를 어떻게 통합할 수 있을지를 문서를 통해 알려준다. 대개의 경우 이들은 프레임워크를 중심에 두고 우리의 아키텍처는 그 바깥을 감싸야 한다고 말한다. 또한 이들은
프레임워크의 기반 클래스에서 직접 파생하거나, 프레임워크의 기능들을 업무 객체에 바로 임포트해서 사용하라고 권한다. 즉, 프레임워크 제작자는 당신의 애플리케이션이 가능하면 프레임워크에 결합될 것을 강하게 조언한다.

결국 프레임워크 제작자는 당신에게 프레임워크와 혼인하기를 요구하는 것이다. 즉, 프레임워크에 대해 장기간에 걸친 막대한 헌신을 요청하는 것이다. 하지만 프레임워크 제작자는 어떠한 경우에도 그에 상응하는 헌신을
당신에게 하지 않는다. 이 관계는 일방적이다.

## 위험 요인

* 업무 객체를 만들 때, 프레임워크 제작자는 자신의 코드를 상속할 것을 요구한다. 고유한 엔티티를 만들 때 말이다. 프레임워크 제작자는 프레임워크가 당신의 가장 안쪽 원과 결합되기를 원한다. 하지만 프레임워크가 한
  번 안으로 들어가면 다시 원 밖으로 나오지 않을 것이다.
* 프레임워크는 애플리케이션의 초기 기능을 만드는 데 도움이 될 것이다. 하지만 제품이 성숙해지면서 프레임워크가 제공하는 기능과 틀을 벗어나게 될 것이다. 이미 프레임워크와 결합되어 있다면, 시간이 지나면서
  프레임워크와 계속 싸우고 있는 자신을 발견하게 될 것이다.
* 프레임워크는 도움되지 않는 방향으로 진화할 수도 있다. 도움도 되지 않는 신규 버전으로 업그레이드하느라 다른 일을 못할 수도 있다. 심지어 사용 중인 기능이 사라지거나 반영하기 힘든 형태가 될 수 있다.
* 새롭고 더 나은 프레임워크가 등장해 갈아타고 싶을 수 있다.

## 해결책

> 프레임워크와 결합되지 말라.

프레임워크는 아키텍처 바깥쪽 원에 속하는 세부사항으로 취급해라.

업무 객체를 만들 때 프레임워크가 자신의 기반 클래스로부터 파생하기를 요구하면, 거절해라. 대신 프락시(proxy)를 만들고, 업무 규칙에 플러그인할 수 이쓴 컴포넌트에 이들 프락시를 위치시켜라. 핵심 코드에
플러그인할 수 있는 컴포넌트에 프레임워크를 통합하, 의존성 규칙을 준수해라.

## 이제 선언합니다. 

정말로 결합되야 하는 프레임워크도 있다. 예를 들어 C++은 STL과 결합되어야 할 가능성이 높다. 자바를 사용한다면 표준 라이브러리와 반드시 결합되어야 한다. 

이러한 관계는 정상이지만, 선택적이여야 한다. 프레임워크와 결합되고자 한다면 애플리케이션의 남은 생애 동안 그 프레임워크와 항상 함께해야 한다.

## 결론

프레임워크와 바로 결합하려고 하지 마라. 가급적 오랫동안 아키텍처 경계 너머에 둘 수 있도록 하자.
