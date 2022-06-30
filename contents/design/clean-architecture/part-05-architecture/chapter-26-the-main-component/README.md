# THE MAIN COMPONENT

모든 시스템엔 최소한 하나의 컴포넌트가 존재하고, 이 컴포넌트에서 나머지 컴포넌트를 생성하고, 조정하며, 관리한다. 이러한 컴포넌트를 메인(Main)이라 부른다.

## 궁극적인 세부사항

메인 컴포넌트는 궁극적인 세부사항으로, 가장 낮은 수준의 정책이다. 운영체제를 제외하면 어떤 것도 메인에 의존하지 않는다. 메인은 모든 팩토리(Factory)와 전략(Strategy), 그리고 시스템 전반을 담당하는
나머지 기반 설비를 생성한 후, 시스템에서 더 높은 수준을 담당하는 부분으로 제어권을 넘기는 역할을 수행한다.

의존성 주입 프레임워크를 이용해 의존성을 주입하는 일은 메인 컴포넌트에서 이루어져야 한다. 메인에 의존성이 주입되고 나면, 메인은 의존성 주입 프레임워크를 사용하지 않고도 일반적인 방식으로 의존성을 분배할 수 있어야
한다.

움퍼스 사냥 게임의 메인 컴포넌트를 보자.

```java
public class Main implements HtwMessageReceiver {
    // ...
    public static void main(String[] args) throws IOException {
        game = HtwFactory.makeGame("htw.game.HuntTheWumpusFacade", // 클래스를 직접 참조하지 않도록 클래스 이름을 문자열 형태로 전달 -> 클래스를 직접 생성하지 않아 의존성이 없으므로 재컴파일/재배포하지 않아도 된다.
                new Main());
        createMap();
        BufferedReader br =
                new BufferedReader(new InputStreamReader(System.in));
        game.makeRestCommand().execute();
        while (true) {
            System.out.println(game.getPlayerCavern());
            System.out.println("Health: " + hitPoints + " arrows: " +
                    game.getQuiver());
            HuntTheWumpus.Command c = game.makeRestCommand();
            System.out.println(">");
            String command = br.readLine();
            if (command.equalsIgnoreCase("e"))
                c = game.makeMoveCommand(EAST);
            else if (command.equalsIgnoreCase("w"))
                c = game.makeMoveCommand(WEST);
            else if (command.equalsIgnoreCase("n"))
                c = game.makeMoveCommand(NORTH);
            else if (command.equalsIgnoreCase("s"))
                c = game.makeMoveCommand(SOUTH);
            else if (command.equalsIgnoreCase("r"))
                c = game.makeRestCommand();
            else if (command.equalsIgnoreCase("sw"))
                c = game.makeShootCommand(WEST);
            else if (command.equalsIgnoreCase("se"))
                c = game.makeShootCommand(EAST);
            else if (command.equalsIgnoreCase("sn"))
                c = game.makeShootCommand(NORTH);
            else if (command.equalsIgnoreCase("ss"))
                c = game.makeShootCommand(SOUTH);
            else if (command.equalsIgnoreCase("q"))
                return;
            c.execute();
        }
    }
    // ...
}
```

입력 스트림 생성, 메인 루프 처리, 간단한 입력 명령어 해석 등은 모두 main 함수에서 처리하지만, 명령을 실제 처리하는 일은 고수준 컴포넌트로 위임한다. 즉, 메인은 고수준의 시스템을 위한 모든 것을 로드한
후, 제어권을 고수준의 시스템에게 넘긴다.

## 결론

메인은 초기 조건과 설정을 구성하고, 외부 자원을 모두 수집한 후, 제어권을 고수준 정책으로 넘기는 플러그인이다. 메인은 플러그인이므로 메인을 애플리케이션 설정별로 하나씩 두어 둘 이상의 메인 컴포넌트를 만들 수
있다. 예를 들어 개발용 메인 플러그인, 테스트용 메인 플러그인 등등. 메인을 플러그인 컴포넌트로 여기고, 아키텍처 경계 바깥에 위치하게 하면 설정 관련 문제를 훨씬 쉽게 해결할 수 있다.
