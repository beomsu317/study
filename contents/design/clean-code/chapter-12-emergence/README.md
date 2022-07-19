# Emergence

## 창발적 설계로 깔끔한 코드를 구현하자

다음 단순한 설계 규칙 네 가지가 소프트웨어 설계 품질을 크게 높여준다.

* 모든 테스트를 실행한다.
* 중복을 없앤다.
* 프로그래머 의도를 표현한다.
* 클래스와 메서드 수를 최소로 줄인다.

위 목록은 중요도 순이다.

## 단순한 설계 규칙 1: 모든 테스트를 실행하라

설계는 의도한 대로 돌아가는 시스템을 내놓아야 한다. 검증이 불가능한 시스템은 절대 출시하면 안 된다. 

테스트가 가능한 시스템을 만들려고 노력하면 설계 품질이 더불어 높아진다. 크기가 작고 목적 하나만 수행하는 클래스가 나온다. SRP를 준수하는 클래스는 테스트가 더 쉽다. 

결합도가 높으면 테스트 케이스를 작성하기 어렵다. DIP 원칙을 적용하거나 DI, 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮춘다. 따라서 설계 품질은 더욱 높아진다.

"테스트 케이스를 만들고 계속 돌려라"라는 간단하고 단순한 규칙을 따르면 시스템은 낮은 결합도와 높은 응집력이라는, 객체 지향 방법론이 지향하는 목표를 저절로 달성한다.

## 단순한 설계 규칙 2 ~ 4: 리팩터링

테스트 케이스를 모두 작성했다면 이제 코드와 클래스를 정리해도 좋다. 코드를 점진적으로 리팩터링 해나간다. 테스트 케이스가 있으므로 코드를 정리하다 시스템이 깨질까 걱정할 필요가 없다. 

리팩터링 단계에서는 소프트웨어 설계 품질을 높이는 기법이라면 무엇이든 적용해도 괜찮다. 응집도를 높이고, 결합도를 낮추고, 관심사를 분리하고, 시스템 관심사를 모듈로 나누고, 함수와 클래스 크기를
줄이고, 더 나은 이름을 선택하는 등. 이 단계는 단순한 설계 규칙 중 나머지 3개를 적용해 중복을 제거하고, 의도를 표현하고, 클래스와 메서드 수를 최소로 줄이는 단계이다.

## 중복을 없애라

중복은 추가 작럽, 추가 위험, 불필요한 복잡도를 뜻한다. 

다음 두 메서드가 있다고 가정하자.

```java
int size() {} 
boolean isEmpty() {}
```

각 메서드를 따로 구현할 수도 있지만, `size` 메서드를 이용하면 코드를 중복해 구현할 필요가 없어진다.

```java
boolean isEmpty() { 
    return 0 == size();
}
```

또 다른 코드를 보자.

```java
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension) < errorThreshold) 
        return;
    float scalingFactor = desiredDimension / imageDimension; 
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    
    RenderedOp newImage = ImageUtilities.getScaledImage( image, scalingFactor, scalingFactor);
    image.dispose(); 
    System.gc(); image = newImage;
}

public synchronized void rotate(int degrees) {
    RenderedOp newImage = ImageUtilities.getRotatedImage( image, degrees);
    image.dispose(); 
    System.gc(); 
    image = newImage;
}
```

`scaleToOneDimension` 메서드와 `rotate` 메서드를 살펴보면 일부 코드가 동일하다. 다음과 같이 코드를 정리해 중복을 제거한다.

```java
public void scaleToOneDimension(float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension) < errorThreshold) 
        return;

    float scalingFactor = desiredDimension / imageDimension; 
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    replaceImage(ImageUtilities.getScaledImage( 
            image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
    replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

private void replaceImage(RenderedOp newImage) { 
    image.dispose();
    System.gc();
    image = newImage;
}
```

공통 코드를 뽑으니 클래스가 SRP를 위반한다. 그러므로 `replaceImage` 메서드를 다른 클래스로 옮겨도 좋다. 

TEMPLATE METHOD 패턴은 고차원 중복을 제거할 목적으로 자주 사용하는 기법이다. 다음 예제를 보자.

```java
public class VacationPolicy {
    public void accrueUSDivisionVacation() {
        // code to calculate vacation based on hours worked to date 
        // ...
        // code to ensure vacation meets US minimums
        // ...
        // code to apply vaction to payroll record
        // ... 
    }

    public void accrueEUDivisionVacation() {
        // code to calculate vacation based on hours worked to date
        // ...
        // code to ensure vacation meets EU minimums
        // ...
        // code to apply vaction to payroll record
        // ...
    }
}
```

최소 법정 일수를 계산하는 코드를 제외하면 두 메서드는 거의 동일하다. 여기에 TEMPLATE METHOD 패턴을 적용해 중복을 제거한다.

```java
abstract public class VacationPolicy {
    public void accrueVacation() {
        calculateBaseVacationHours();
        alterForLegalMinimums();
        applyToPayroll();
    }

    private void calculateBaseVacationHours() { /* ... */ }
    abstract protected void alterForLegalMinimums();
    private void applyToPayroll() { /* ... */ }
}

public class USVacationPolicy extends VacationPolicy {
    @Override
    protected void alterForLegalMinimums() {
        // US specific logic 
    }
}

public class EUVacationPolicy extends VacationPolicy {
    @Override
    protected void alterForLegalMinimums() {
        // EU specific logic 
    }
}
```

하위 클래스는 중복되지 않는 정보만 제공해 `accureVacation` 알고리즘에서 빠진 구멍을 메운다.

## 표현하라

시스템이 점차 복잡해지면서 개발자가 시스템을 이해하느라 보내는 시간은 점점 늘어나고 동시에 코드를 오해할 가능성도 커진다. 따라서 코드는 개발자의 의도를 분명히 표현해야 한다. 그래야
결함이 줄어들고 유지보수 비용이 적게 든다.

1. 좋은 이름을 선택한다. 
2. 함수와 클래스 크기를 가능한 줄인다. 작은 클래스와 작은 함수는 이름 짓기도 쉽고, 구현하기도 쉽고, 이해하기도 쉽다.
3. 표준 명칭을 사용한다. 예를 들어, 디자인 패턴은 의사소통과 표현력 강화가 주요 목적이다. 클래스가 COMMAND나 VISITOR 같은 표준 패턴을 사용해 구현된다면 클래스 이름에 패턴 이름을 넣어준다.
4. 유닛 테스트 케이스를 꼼꼼히 작성한다. 잘 만든 테스트 케이스를 읽어보면 클래스 기능이 한눈에 들어온다.

## 클래스와 메서드 수를 최소로 줄여라

중복을 제거하고, 의도를 표현하고, SRP를 준수한다는 기본적인 개념도 극단적으로 치달으면 득보다 실이 많아진다. 클래스와 메서드 크기를 줄이자고 조그만 클래스와 메서드를 수없이 만드는 사례도 없지 않다. 그래서
이 규칙은 함수와 클래스 수를 가능한 줄이라고 제안한다.

이 규칙은 간단한 설계 규칙 네 개 중 우선순위가 가장 낮다. 다시 말해, 클래스와 함수 수를 줄이는 작업도 중요하지만, 테스트 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요하다.

## 결론

단순한 설계 규칙을 따른다면 우수한 기법과 원칙을 단번에 활용할 수 있다.