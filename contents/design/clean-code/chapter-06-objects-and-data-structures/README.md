# Objects and Data Structures

변수를 `private`으로 정의하는 이유는 외부에서 변수에 의존하지 않게 만들고 싶어서다. 그렇다면 수많은 프로그래머가 `get` 함수와 `set` 함수를 당연하게 `public`으로 노출하는 이유는 무엇일까?

## 자료 추상화

다음 2개의 코드를 보자. 첫 번째 코드는 구현을 외부로 노출하고 두 번째 코드는 구현을 숨긴다.

```java
public class Point { 
    public double x; 
    public double y;
}
```

```java
public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```

두 번째 코드의 경우 직교좌표계를 사용하는지 극좌표계를 사용하는지 알 수 없다. 그러나 인터페이스는 자료구조를 명백하게 표현한다. 좌표를 읽을 때는 각 값을 개별적으로 읽어야 하며, 설정할 때는 
두 값을 한번에 설정해야 한다.

첫 번째 코드는 확실히 직교좌표계를 사용한다. 또한 개별적으로 좌표값을 읽고 설정하게 강제한다. 이 코드는 구현을 노출하며 `private`으로 선언하더라도 `get`, `set` 함수를 제공한다면 구현을 외부로
노출하는 셈이다.

변수 사이 함수 계층을 넣는다고 구현이 감춰지지 않는다. 구현을 감추려면 추상화가 필요하다. 추상화를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.

다음 두 코드를 보자. 첫 번째 코드는 자동차 연료 상태를 구체적인 숫자 값으로 알려준다. 두 번째 코드는 연료 상태를 백분율이라는 추상적인 개념으로 알려준다. 첫 번째 코드는 두 함수가 변수값을 읽어
반환할 뿐이라는 사실이 거의 확실하지만, 두 번째 코드는 정보가 어디서 오는지 전혀 드러나지 않는다.

```java
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}
```

```java
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다. 

## 자료/객체 비대칭

객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다. 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다. 두 정의는 본질적으로 상반된다. 

다음은 절차적인 도형 클래스이다. 

```java
public class Square {
    public Point topLeft;
    public double side;
}

public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.141592653589793;

    public double area(Object shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.height * r.width;
        } else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```

만약 `Geometry` 클래스에 둘레 길이를 구하는 `perimeter()` 함수를 추가하는 경우, 도형 클래스는 아무 영향도 받지 않는다. 반대로 새 도형을 추가하고 싶다면 `Geometry` 클래스에
속한 함수를 모두 고쳐야 한다. 

다음은 객체 지향적인 도형 클래스이다. `area()`는 다형 메서드다. 새 도형을 추가해도 기존 함수에 아무런 영향이 없다. 하지만 새 함수를 추가하는 경우 도형 클래스의 전부를 고쳐야 한다.

```java
public class Square implements Shape {
    private Point topLeft;
    private double side;

    public double area() {
        return side * side;
    }
}

public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;

    public double area() {
        return height * width;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;

    public double area() {
        return PI * radius * radius;
    }
}
```

위 소개된 두 코드는 상호 보완적인 특질이 있다. 

> (자료 구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다.

새로운 자료 타입이 필요한 경우 갹체 지향 기법을 사용하는 것이 적합하며, 새로운 함수가 필요ㅏㄴ 경우 절차적인 코드와 자료 구조가 더 적합하다.

## 디미터 법칙

디미터 법칙은 잘 알려진 휴리스틱(heuristic)으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다. 좀 더 정확히 표현하자면, "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.

* 클래스 C
* f가 생성한 객체
* f 인수로 넘어온 객체
* C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안 된다. 

다음 코드는 디미터 법칙을 어기는 듯이 보인다.

```java
final String outputDir=ctxt.getOptions().getScratchDir().getAbsolutePath();
```

### 기차 충돌

위와 같은 코드를 기차 충돌(train wreck)이라 부른다. 여러 객차가 한 줄로 이어진 기차처럼 보이기 떄문이다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 것이 좋다. 다음과 같이 나눌 수 있다. 

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

위 예제가 디미터 법칙을 위반하는지 여부는 `ctxt`, `Options`, `ScratchDir`이 객체인지 아니면 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 디미터 법칙을 위반하지만, 자료 구조라면
당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

그런데 위 예제는 `get` 함수를 사용하는 바람에 혼란을 일으킨다. 코드를 다음과 같이 구현했다면 디미터 법칙을 거론할 필요가 없어진다.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

자료 구조는 무조건 함수 없이 `public` 변수만 퐇마하고 객체는 `private` 변수와 `public` 함수를 포함한다면, 문제는 훨씬 간단해진다. 하지만 단순한 자료 구조에도 `get` 함수와 `set` 함수를 정의하라
요구하는 프레임워크와 표준(ex: bean)이 존재한다.

### 잡종 구조

이러한 혼란으로 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다. 이런 잡종 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다. 그러므로 잡종 구조는 되도록 피하는 편이 좋다.

### 구조체 감추기

만약 `ctxt`, `Options`, `ScratchDir`이 객체라면, 예제처럼 줄줄이 엮어서는 안 된다. 그렇다면 임시 디렉토리의 절대 경로는 어떻게 얻어야 좋을까? 다음 두 코드를 보자.

```java
ctxt.getAbsolutePathOfScratchDirectoryOption();
```

```java
ctx.getScratchDirectoryOption().getAbsolutePath()
```

첫 번째 방법은 `ctxt` 객체에 공개해야 하는 메서드가 너무 많아진다. 두 번째 방법은 `getScratchDirectoryOption()`이 객체가 아니라 자료 구조를 반환한다고 가정한다. 어느 방법도 썩 좋지 않아 보인다.

`ctxt`가 객체라면 속을 드러내라고 하면 안 된다. 임시 디렉터리의 절대 경로가 왜 필요하는지를 알아보자. 다음은 같은 모듈에서 가져온 코드이다.

```java

String outFile = outputDir + "/" + className.replace('.', '/') + ".class"; 
FileOutputStream fout = new FileOutputStream(outFile); 
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

추상화 수준을 뒤섞어 놓아 다소 불편하다. 위 코드를 살펴보면, 임시 디렉터리의 절대 경로를 얻으려는 이유는 임시 파일을 생성하기 위함이다. 

그렇다면 `ctxt` 객체에 임시 파일을 생성하라고 시키는 것이 객체에게 맡기기에 적당한 임무로 보인다. `ctxt`는 내부 구조를 드러내지 않으며, 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다. 따라서
디미터 원칙을 위반하지 않는다.

## 자료 전달 객체

자료 구조체의 전형적인 형태는 `public` 변수만 있고 함수가 없는 클래스다. 이런 자료 구조체를 때로 DTO(Data Transfer Object)라 한다. DTO는 데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때
유용하다. 

좀 더 일반적인 형태는 `bean` 구조다. `bean`은 `private` 변수를 `get`, `set` 함수로 조작한다. 일종의 사이비 캡슐화로, 일부 OO 순수주의자나 만족시킬 뿐 별다른 이익을 제공하지 않는다.

```java
public class Address {
    private String street;
    private String streetExtra;
    private String city;
    private String state;
    private String zip;

    public Address(String street, String streetExtra, String city, String state, String zip) {
        this.street = street;
        this.streetExtra = streetExtra;
        this.city = city;
        this.state = state;
        this.zip = zip;
    }

    public String getStreet() {
        return street;
    }

    public String getStreetExtra() {
        return streetExtra;
    }

    public String getCity() {
        return city;
    }

    public String getState() {
        return state;
    }

    public String getZip() {
        return zip;
    }
}
```

### Active Record

활성 레코드는 DTO의 특수한 형태이다. `public` 변수가 있거나 `private` 변수에 `get`, `set` 함수가 있는 자료 구조지만, 대게 `save`나 `find` 같은 탐색 함수도 제공한다. 활성 레코드는
데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과이다.

불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔하다. 이는 잡종 구조이므로 바람직하지 않다. 따라서 활성 레코드는 자료 구조로 취급하며, 비즈니스 규칙을 담으며
내부 자료를 숨기는 객체는 따로 생성한다.

## 결론

* 객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하는 것은 어렵다.
* 자료 구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.

따라서 새로운 자료 타입을 추가하는 유연성이 필요하면 객체로, 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 적합하다.

