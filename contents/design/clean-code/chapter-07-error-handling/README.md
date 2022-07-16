# Error Handling

깨끗한 코드와 오류 처리는 연관성이 있다. 상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다. 좌우된다는 표현은 코드 기반이 오류만 처리한다는 의미가 아니다. 여기저기 흩어진 오류 처리 코드 때문에
실제 코드가 하는 일을 파악하기가 어렵다는 의미다. 오류 처리는 중요하다. 하지만 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다.

## 오류 코드보다 예외를 사용하라

얼마 전까지만 해도 예외를 지원하지 않는 프로그래밍 언어가 많았다. 예외를 지원하지 않는 언어는 오류를 처리하고 보고하는 방법이 제한적이었다. 

```java
public class DeviceController {
    //...

    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1); 
        // Check the state of the device
        if (handle != DeviceHandle.INVALID) {
            // Save the device status to the record field 
            retrieveDeviceRecord(handle);
            // If not suspended, shut down
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
            logger.log("Invalid handle for: " + DEV1.toString());
        }
    }
    // ...
}
```

위와 같은 방법을 사용하면 호출자 코드가 복잡해진다. 함수를 호출한 즉시 오류를 확인해야 하기 때문이다. 이 단계는 잊어버리기 쉽다. 따라서 오류가 발생하면 예외를 던지는 편이 낫다. 그럼 논리가 오류 처리 코드와
뒤섞이지 않아 호출자 코드가 더 깔끔해진다.

```java
public class DeviceController {
    // ...
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }

    private DeviceHandle getHandle(DeviceID id) {
        // ...
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
        // ... 
    }
    //...
}
```

단순히 보기만 좋아지지 않고 코드 품질도 나아졌다. 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리했기 때문이다. 

## Try-Catch-Finally 문부터 작성하라

`try-catch-finally` 문에서 `try` 블록에 들어가는 코드를 실행하면 어느 시점에서든 실행이 중단된 후 `try` 블록으로 넘어갈 수 있다. 따라서 `try` 블록은 트랜잭션과 비슷하다. `try`
블록에서 무슨 일이 생기는지 `catch` 블록은 프로그램 상태를 일관성 있게 유지해야 한다. 그러므로 예외가 발생할 코드를 짤 때는 `try-catch-finally` 문으로 시작하는 편이 낫다. 그러면
`try` 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

파일을 열어 직렬화된 객체 몇 개를 읽어 들이는 코드 예제를 살펴보자.

다음은 파일이 없으면 예외를 던지는지 알아보기 위한 유닛 테스트다.

```java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file"); 
}
```

유닛 테스트에 맞춰 다음 코드를 구현했다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) { 
    // dummy return until we have a real implementation
    return new ArrayList<RecordedGrip>();
}
```

코드가 예외를 던지지 않으므로 단위 테스트는 실패한다. 잘못된 파일 접근을 시도하게 구현을 변경한다. 다음 코드는 예외를 던진다.

```java

public List<RecordedGrip> retrieveSection(String sectionName) { 
    try {
        FileInputStream stream = new FileInputStream(sectionName) 
    } catch (Exception e) {
        throw new StorageException("retrieval error", e); 
    }
    return new ArrayList<RecordedGrip>(); 
}
```

이제 유닛 테스트가 성공한다. 이 시점에서 리팩터링이 가능하다. `catch` 블록에서 예외 타입을 좁혀 실제로 `FileInputStream` 생성자가 던지는 `FileNotFoundException`을 잡아낸다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) { 
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        stream.close();
    } catch (FileNotFoundException e) {
        throw new StorageException("retrieval error”, e); 
    }
    return new ArrayList<RecordedGrip>(); 
}
```

`try-catch` 구조로 범위를 정했으므로 TDD를 사용해 필요한 나머지 논리를 추가한다. 나머지 논리는 `FileInputStream`을 생성하는 코드와 `close` 호출문 사이에 넣으면 오류나 예외가 전혀 발생하지 않는다고 가정한다.

먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다. 그러면 자연스럽게 `try` 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

## 미확인(unchecked) 예외를 사용하라

확인된 예외를 선보인 자바 첫 버전에서는 메서드가 반환할 예외를 모두 열거했다. 반환하는 예외는 메서드 타입의 일부였다. 코드가 메서드를 사용하는 방식이 메서드 선언과 일치하지 않으면 컴파일도 되지 않았다. 

C++, C#은 확인된 예외를 지원하지 않는데도 불구하고, 안정적인 소프트웨어를 구현하기에 무리가 없다. 

확인된 예외는 OCP를 위반한다. 메서드에서 확인된 예외를 던졌는데 `catch` 블록이 세 단계 위에 있다면 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야 한다. 즉, 하위 단계에서 코드를 변경하면 상위 단계
메서드 선언부를 모두 고쳐야 한다는 의미이다. 모듈과 관련된 코드가 바뀌지 않았더라도 (선언부가 바뀌었으므로) 모듈을 다시 빌드한 다음 배포해야 한다는 말이다.

때로는 확인된 예외도 유용하다. 중요한 라이브러리를 작성한다면 모든 예외를 잡아야 한다. 하지만 일반적인 애플리케이션은 의존성이라는 비용해 이익보다 크다.

## 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙인다. 그러면 오류가 발생한 원인과 위치를 찾기 쉬워진다. 자바는 모든 예외에 호출 스택을 제공하지만, 실패한 코드의 의도를 파악하기 어렵다. 

오류 메시지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급한다. 로깅 기능을 사용한다면 `catch` 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.

## 호출자를 고려해 예외 클래스를 정의하라

오류를 분류하는 방법은 수없이 많다. 하지만 애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야 한다.

다음은 오류를 형편없이 분류한 사례이다. 외부 라이브러리를 호출하는 `try-catch-finally` 문을 포함한 코드로, 외부 라이브러리가 던질 예외를 모두 잡아낸다.

```java
ACMEPort port = new ACMEPort(12);
try { 
    port.open();
} catch (DeviceResponseException e) { 
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) { 
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) { 
    reportPortError(e);
    logger.log("Device response exception");
} finally { 
    // ...
}
```

대다수 상황에서 오류를 처리하는 방식은 비교적 일정하다. 

1. 오류를 기록한다.
2. 프로그램을 계속 수행해도 좋은지 확인한다.

위 경우 예외에 대응하는 방식이 거의 동일하다. 그러므로 코드를 간결하게 고치기 쉽다. 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환하면 된다.

```java
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    //... 
}
```

`LocalPort` 클래스는 단순히 `ACMEPort` 클래스가 던지는 예외를 잡아 변환하는 wrapper 클래스일 뿐이다.

```java

public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
    // ... 
}
```

`LocalPort` 클래스처럼 `ACMEPort`를 wrap하는 클래스는 매우 유용하다. 외부 API를 wrapping 하면 외부 라이브러리와 프로그램 사이 의존성이 크게 줄어든다. 나중에 다른 라이브러리로
갈아타도 비용이 적다. 또한 wrapper 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방식으로 테스트하기도 쉽다.

흔히 예외 클래스가 하나만 있어도 충분한 코드가 많다. 예외 클래스에 포함된 정보로 오류를 구분해도 괜찮은 경우가 그렇다. 한 예외는 잡아내고 다른 예외는 무시해도 괜찮은 경우라면 여러 예외 클래스를 사용한다.

## 정상 흐름을 정의하라

외부 API를 wrap하여 자체적인 예외를 던지고, 핸들러를 정의해 중단된 계산을 처리한다. 이는 대개 멋진 처리 방식이지만, 때로는 적합하지 않을 때도 있다.

다음은 비용 청구 애플리케이션에서 총계를 계산하는 허술한 코드다.

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) { 
    m_total += getMealPerDiem();
}
```

식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다. 식비를 비용으로 청구하지 않았다면 일일 기본 식비를 총계에 더한다. 그런데 예외가 논리를 따라가기 어렵게 만든다. 특수 상황을 처리할
필요가 없다면 더 좋을 것으로 보인다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID()); 
m_total += expenses.getTotal();
```

`ExpenseReportDAO`를 고쳐 언제나 `MealExpenses` 객체를 반환한다. 청구한 식비가 없다면 일일 기본 식비를 반환하는 `MealExpenses` 객체를 반환한다.

```java
public class PerDiemMealExpenses implements MealExpenses { 
    public int getTotal() {
        // return the per diem default
    }
}
```

이를 특수 사례 패턴(SPECIAL CASE PATTERN)이라 부른다. 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다. 그러면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다. 

## null을 반환하지 마라

흔히 저지르는 실수 중 하나가 `null`을 반환하는 습관이다. 다음 예제를 보자.

```java
public void registerItem(Item item) { 
    if (item != null) {
        ItemRegistry registry = peristentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item); 
            }
        }
    }
}
```

`null`을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 넘긴다. 만약 `peristentStore`가 `null`이라면 실행 시 `NullPointerException`이 발생할 것이다. 어느 쪽에서 이 예외를 
처리하든 나쁜 코드이다. 

메서드에서 `null`을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환한다. 외부 API가 `null`을 반환한다면 wrap 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다.

다음과 같은 코드가 있다고 가정하자.

```java
List<Employee> employees = getEmployees(); 
if (employees != null) {
    for(Employee e : employees) { 
        totalPay += e.getPay();
    } 
}
```

`getEmployees`는 `null`을 반환한다. `getEmployees`를 변경해 빈 리스트를 반환한다면 코드가 훨씬 깔끔해진다.

```java
List<Employee> employees = getEmployees(); 
for(Employee e : employees) {
    totalPay += e.getPay(); 
}
```

다행스럽게 자바에는 `Collections.emptyList()`가 있어 미리 정의된 읽기 전용 리스트를 반환한다. 우리 목적에 적합한 리스트다.

```java
public List<Employee> getEmployees() { 
    if( .. there are no employees .. )
        return Collections.emptyList(); 
}
```

이렇게 변경하면 코드도 깔끔해질뿐만 아니라 `NullPointerException`이 발생할 가능성도 줄어든다.

## null을 전달하지 마라

메서드에서 `null`을 반환하는 방식도 나쁘지만 메서드로 `null`을 전달하는 방식은 더 나쁘다. 

다음은 두 지점 사이 거리를 계산하는 메서드이다.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        return (p2.x – p1.x) * 1.5;
    }
// ... 
}
```

인자로 `null`을 전달하면 `NullPointerException`이 발생할 것이다. 다음과 같이 새로운 예외 유형을 만들어 던지는 방법이 있다.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        if (p1 == null || p2 == null) {
            throw InvalidArgumentException(
                    "Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x – p1.x) * 1.5;
    }
}
```

위 코드가 원래 코드보다 나아 보일수도 있으나 `InvalidArgumentException`을 잡아내는 핸들러가 필요하다. 

다음은 `assert` 문을 사용하는 방법이다.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        assert p1 != null : "p1 should not be null";
        assert p2 != null : "p2 should not be null";
        return (p2.x – p1.x) * 1.5;
    }
}
```

대다수 프로그래밍 언어는 호출자가 실수로 넘기는 `null`을 적절히 처리하는 방법이 없다. 그렇다면 애초에 `null`을 넘기지 못하도록 금지하는 정책이 합리적이다.

## 결론

깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 오류 처리 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다. 또한 독립적인 추론이
가능해지며 코드 유지보수성도 크게 높아진다.