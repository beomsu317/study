# THE WEB IS A DETAIL

웹이 등장한 뒤로 모든 연산 능력을 서버 팜(server farm)에 위치할 것이고 브라우저는 멍청해질 것이라 생각했다. 그러다 브라우저에 애플릿(applet)을 추가하기 시작했다. 하지만 이 방식이 내키지 않았기에
동적인 내용은 다시 서버로 이동시켰다. 하지만 다시 이 방식이 마음에 들지 않아 웹 2.0을 고안했고, Ajax와 자바스크립트를 이용해 처리 과정의 많은 부분을 다시 브라우저로 옮겼다. 그리고 이제 노드(
node.js)를 이용해 자바스크립트를 다시 서버로 이동시키는 방식에 열광해 있다.

## 끝없이 반복하는 추

연산 능력을 중앙에 집중하는 방식과 분산하는 방식 사이에서 끊임없이 움직이고 있다.

Q사는 상당히 인기를 끈 개인용 재무 시스템을 개발했다. 이 시스템은 데스크톱 앱으로, 상당히 유용한 GUI를 제공했다. 어느날 웹이 등장한 후 Q사는 앱의 GUI가 마치 브라우저처럼 보이고 동작하도록 변경했다.
하지만 이를 좋아하는 사람은 많지 않았으며, 몇 차례의 출시를 거쳐 Q사는 브라우저스러운 느낌을 조금씩 없애고 다시 일반적인 데스크톱 GUI로 되돌렸다.

여기서 아키텍트가 애플리케이션을 보호하기 위해 업무 규칙을 UI로부터 분리했어야 한다.

## 요약

GUI는 세부사항이며, 웹은 GUI이다. 따라서 웹은 세부사항이다. 아키텍트라면 이러한 세부사항을 핵심 업무로부터 분리된 경계 바깥에 두어야 한다.

UI와 애플리케이션 사이 추상화가 가능한 경계가 존재한다. 업무 로직은 다수의 유스케이스로 구성되며, 각 유스케이스는 사용자를 대신해 일부 함수를 수행하는 것으로 볼 수 있다. 완전한 입력 데이터와 그에 따른 출력
데이터는 데이터 구조로 만들어서 유스케이스를 실행하는 처리 과정의 입력 갑소가 출력 값으로 사용할 수 있다. 이 방식을 따르면 각 유스케이스가 장치 독립적인 방식으로 UI라는 입출력 장치를 동작시킨다고 간주할 수
있다.

## 결론

이러한 종류의 추상화는 만들기 쉽지 않고, 제대로 만들려면 수차례의 반복 과정을 거쳐야 하지만, 가능하다. 이러한 추상화가 필요할 때가 많을 수 있다. 