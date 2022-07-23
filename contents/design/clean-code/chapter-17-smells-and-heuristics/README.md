# Smells and Heuristics

## 주석

### C1: 부적절한 정보

다른 시스템(소스 코드 관리 시스템, 버그 추적 시스템 등)에 저장할 정보는 주석으로 적절하지 못하다. 일반적으로 작성자, 최종 수정일 SPR(Software Problem Report) 번호 등과 같은 메타 정보만 주석으로 넣는다.

### C2: 쓸모 없는 주석

오래된 주석, 엉뚱한 주석, 잘못된 주석은 더 이상 쓸모가 없다. 쓸모 없어질 주석은 아예 달지 않는 편이 좋으며, 쓸모 없어진 주석은 빨리 삭제하는 것이 좋다.

### C3: 중복된 주석

코드만으로 충분한데 구구절절 설명하는 주석이 중복된 주석이다. 주석은 코드만으로 다하지 못하는 설명을 부언한다.

```java

/**
 * @param sellRequest
 * @return
 * @throws ManagedComponentException */
public SellResponse beginSellItem(SellRequest sellRequest) 
        throws ManagedComponentException {
}
```

### C4: 성의 없는 주석

작성할 가치가 있는 주석은 시간을 들여 최대한 멋지게 작성한다.

### C5: 주석 처리된 코드

주석으로 처리된 코드가 나오면 얼마나 오래된 코드인지, 중요한 코드인지 아닌지 알 수 없다. 따라서 주석으로 처리된 코드를 발견하면 즉각 지워라. 소스 코드 관리 시스템이 기억하고 있다.

## 환경

### E1: 여러 단계로 빌드해야 한다

빌드는 간단히 한 단계로 끝나야 한다. 시스템에 필요한 파일을 찾느라 여기저기 뒤적일 필요가 없어야 한다. 한 명령으로 전체를 체크아웃해서 한 명령으로 빌드할 수 있어야 한다.

```shell
svn get mySystem
cd mySystem
ant all
```

### E2: 여러 단계로 테스트해야 한다

모든 유닛 테스트는 한 명령으로 돌려야 한다. 모든 테스트를 한 번에 실행하는 능력은 중요하다. 그 방법이 빠르고, 쉽고, 명백해야 한다.

## 함수

### F1: 너무 많은 인수

함수에서 인수 개수는 작을수록 좋다. 넷 이상은 그 가치가 아주 의심스러우므로 최대한 피한다.

### F2: 출력 인수

출력 인수는 직관을 정면으로 위배한다. 일반적으로 독자는 출력 인수를 입력으로 간주한다. 함수에서 뭔가의 상태를 변경해야 한다면 함수가 속한 객체의 상태를 변경한다.

### F3: 플래그 인수

`boolean` 인수는 함수가 여러 기능을 수행한다는 의미이다. 플래그 인수는 혼란을 초래하므로 피해야 마땅하다.

### F4: 죽은 함수

아무도 호출하지 않는 함수는 삭제한다. 죽은 코드는 낭비다. 

## 일반

### G1: 한 소스 파일에 여러 언어를 사용한다

오늘날 프로그래밍 환경은 한 소스 파일 내 여러 언어를 지원한다. 이상적으로 소스 파일 하나에 언어 하나만 사용하는 방식이 좋다. 현실적으로는 여러 언어가 불가피하며, 노력을 기울여
언어 수와 범위를 최대한 줄이도록 해야한다.

### G2: 당연한 동작을 구현하지 않는다

최소 놀람의 원칙(The Principle of Least Surprise)에 의거해 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다. 예를 들어, 요일 문자열에서
요일을 나타내는 `enum`으로 변환하는 함수를 보자.

```java
Day day = DayDate.StringToDay(String dayName);
```

함수가 `Monday`를 `Day.MONDAY`로 변환하리라 기대한다. 또한 일반적으로 쓰는 요일 약어도 올바로 변환하리라 기대한다. 대소문자는 당연히 구분하지 않으리라 기대한다.

당연한 동작을 구현하지 않으면 코드를 읽거나 사용하는 사람이 더 이상 함수 이름만으로 함수 기능을 직관적으로 예상하기 어렵다. 때문에 코드를 일일이 살펴야 한다.

### G3: 경계를 올바로 처리하지 않는다

직관에 의존하지 말아라. 모든 경계 조건을 찾아내고, 모든 경계 조건을 테스트하는 테스트 케이스를 만들어라.

### G4: 안전 절차 무시

안전 절차를 무시하면 위험하다. `serialVersionUID`를 직접 제어할 필요가 있을지 모르지만 그래도 직접 제어는 언제나 위험하다. 컴파일러 경고 일부를 꺼버리면 빌드가 
쉬워질지 모르지만 자칫하면 끝없는 디버깅에 시달린다. 실패하는 테스트 케이스를 나중으로 미루는 태도는 위험하다.

### G5: 중복

DRY(Don't Repeat Yourself) 원칙이라고도 부른다. 코드에서 중복을 발견할 때마다 추상화할 기회로 간주하라. 중복된 코드를 하위 루틴이나 다른 클래스로 분리하라. 추상화 수준을
높였으므로 구현이 빨라지고 오류가 적어진다.

1. 가장 뻔한 유형은 똑같은 코드가 여러 차례 나오는 중복이다. 이런 중복은 간단한 함수로 교체한다.
2. 좀 더 미묘한 유형은 여러 모듈에서 일련의 `switch/case`나 `if/else` 문으로 똑같은 조건을 거듭 확인하는 중복이다. 이런 중복은 다형성으로 대체해야 한다.
3. 더더욱 미묘한 유형은 알고리즘이 유사하나 코드가 서로 다른 중복이다. TEMPLATE METHOD 패턴이나 STRATEGY 패턴으로 중복을 제거한다.

### G6: 추상화 수준이 올바르지 못하다

추상화는 저차원 상세 개념에서 고차원 일반 개념을 분리한다. 때로 (고차원 개념을 표현하는) 추상 클래스와 (저차원 개념을 표현하는) 파생 클래스를 생성해 추상화를 수행한다.
추상화로 개념을 분리할 때는 철저해야 한다. 모든 저차원 개념은 파생 클래스에 넣고, 모든 고차원 개념은 기초 클래스에 넣는다.

예를 들어, 세부 구현과 관련한 상수, 변수, 유틸리티 함수는 기초 클래스에 넣으면 안 된다. 기초 클래스는 구현 정보에 무지해야 마땅하다.

다음 코드를 보자.

```java
public interface Stack {
    Object pop() throws EmptyException;

    void push(Object o) throws FullException;

    double percentFull();

    class EmptyException extends Exception {
    }

    class FullException extends Exception {
    }
}
```

`percentFull` 함수는 추상화 수준이 올바르지 못하다. 스택을 구현하는 방법은 다양하다. 어떤 구현은 "꽉 찬 정도"라는 개념이 타당하며, 어떤 구현은 알아낼 방법이 전혀 없다. 그러므로 함수는
`BoundedStack`과 같은 파생 인터페이스에 넣어야 마땅하다.

잘못된 추상화 수준은 거짓말이나 꼼수로 해결하지 못한다. 

### G7: 기초 클래스가 파생 클래스에 의존한다

개념을 기초 클래스와 파생 클래스로 나누는 흔한 이유는 고차원 기초 클래스 개념을 저차원 파생 클래스 개념으로부터 독립성을 보장하기 위해서다. 그러므로 기초 클래스가 파생 클래스를 사용한다면 뭔가 문제가 
있다는 의미이다.

간혹 파생 클래스 개수가 고정되어 있다면 기초 클래스에서 파생 클래스를 선택하는 코드가 들어간다. FSM(Finite State Machine) 구현에서 많이 본 사례다. FSM은 기초 클래스와 파생 클래스가 굉장히 
밀접하며 언제나 같은 JAR로 배포한다. 일반적으로는 기초 클래스와 파생 클래스를 다른 JAR 파일로 배포하는 편이 좋다.

기초 클래스와 파생 클래스를 다른 JAR 파일로 배포하면, 독립적인 개별 컴포넌트 단위로 시스템을 배치할 수 있다. 즉, 변경이 시스템에 미치는 영향이 아주 작아지므로 유지보수가 수월하게 된다.

### G8: 과도한 정보

잘 정의된 모듈은 인터페이스가 아주 작지만, 많은 동작이 가능하다. 잘 정의된 인터페이스는 많은 함수를 제공하지 않아 결합도(coupling)가 낮다.  

부실하게 정의된 모듈은 인터페이스가 구질구질하다. 부실하게 정의된 인터페이스는 반드시 호출해야 하는 온갖 함수를 제공한다. 그래서 결합도가 높다.

자료를 숨겨라. 유틸리티 함수를 숨겨라. 상수와 임시 변수를 숨겨라. 메서드나 인스턴스 변수가 넘치는 클래스를 피하라. 하위 클래스에서 필요하다는 이유로 `protected` 변수나 함수를 마구 생성하지 마라.
인터페이스를 매우 작게 그리고 깐깐하게 만들어라. 정보를 제한해 결합도를 낮춰라.

### G9: 죽은 코드

실행되지 않는 코드를 의미한다. 불가능한 조건을 확인하는 `if` 문과 `throw` 문이 없는 `try` 문에서 `catch` 블록이 좋은 예다. 또한 호출하지 않는 유틸리티 함수와
`switch/case` 문에서 불가능한 `case` 조건도 좋은 예다. 이들은 시스템에서 제거하라.

### G10: 수직 분리

변수와 함수는 사용되는 위치에 가깝게 정의한다. 지역 변수는 처음으로 사용하기 직전에 선언하며 수직으로 가까운 곳에 위치해야 한다. 

`private` 함수는 처음으로 호출한 직후에 정의한다. `private` 함수는 전체 클래스 범위에 속하지만 그래도 정의하는 위치와 호출하는 위치를 가깝게 유지한다. 

### G11: 일관성 부족

어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다. 앞서 언급한 최소 놀람의 법칙(The Principle of Least Surprise)에도 부합한다. 표기법은
신중히 선택하며, 일단 선택한 표기법은 신중하게 따른다. 

한 함수에서 `response`라는 변수에 `HttpServletResponse` 인스턴스를 저장했다면, 다른 함수에서도 일관성 있고 동일한 변수 이름을 사용한다.

이러한 일관성만으로도 코드를 읽고 수정하기가 쉬워진다.

### G12: 잡동사니

비어 있는 생성자는 쓸데없이 코드만 복잡하게 만든다. 따라서 제거하는게 마땅하다.

### G13: 인위적 결합

서로 무관한 개념을 인위적으로 결합하지 않는다. 예를 들어, 일반적인 `enum`은 특정 클래스에 속할 이유가 없다. `enum`이 클래스에 속한다면 `enum`을 사용하는 코드가 특정 클래스를 알아야 한다. 
범용 `static` 함수도 마찬가지로 특정 클래스에 속할 이유가 없다. 

일반적으로 인위적인 결합은 직접적인 상호작용이 없는 두 모듈 사이 일어난다. 뚜렸한 목적 없이 변수, 상수, 함수를 당장 편한 위치(잘못된)에 넣어버린 결과다.   

함수, 상수, 변수를 선언할 때는 시간을 들여 올바른 위치를 고민한다. 당장 편한 곳에 선언하고 내버려두면 안 된다.

### G14: 기능 욕심

클래스 메서드는 자신의 클래스 변수와 함수에 관심을 가져야지 다른 클래스의 변수와 함수에 관심을 가져선 안 된다. 메서드가 다른 객체의 참조자(accessor)와 변경자(mutator)를 사용해 그 객체 
내용을 조작한다면 메서드가 그 객체 클래스의 범위를 욕심내는 탓이다. 자신이 그 클래스에 속해 그 클래스 변수를 직접 조작하고 싶다는 뜻이다. 다음 예제를 보자.

```java
public class HourlyPayCalculator {
    public Money calculateWeeklyPay(HourlyEmployee e) {
        int tenthRate = e.getTenthRate().getPennies();
        int tenthsWorked = e.getTenthsWorked();
        int straightTime = Math.min(400, tenthsWorked);
        int overTime = Math.max(0, tenthsWorked - straightTime);
        int straightPay = straightTime * tenthRate;
        int overtimePay = (int) Math.round(overTime * tenthRate * 1.5);
        return new Money(straightPay + overtimePay);
    }
}
```

`calculateWeeklyPay` 메서드는 `HourlyEmployee` 객체에서 온갖 정보를 가져온다. 즉, `calculateWeeklyPay` 메서드는 `HourlyEmployee` 클래스의 범위를 욕심낸다. 

기능 욕심은 한 클래스의 속사정을 다른 클래스에 노출하므로 제거하는 편이 좋다. 하지만 때로는 어쩔 수 없는 경우도 생긴다. 다음 코드를 보자.

```java
public class HourlyEmployeeReport {
    private HourlyEmployee employee;

    public HourlyEmployeeReport(HourlyEmployee e) {
        this.employee = e;
    }

    String reportHours() {
        return String.format(
                "Name: %s\tHours:%d.%1d\n", employee.getName(), employee.getTenthsWorked() / 10, employee.getTenthsWorked() % 10);
    }
}
```

`reportHours` 메서드는 `HourlyEmployee` 클래스를 욕심낸다. 그렇다고 `HourlyEmployee` 클래스가 보고서 형식을 알 필요는 없다. 함수를 `HourlyEmployee` 클래스로 옮기면 여러 원칙을
위반하게 된다. `HourlyEmployee`가 보고서 형식과 결합되므로 보고서 형식이 바뀌면 클래스도 바뀐다. 

### G15: 선택자 인수

함수 호출 끝에 달리는 `false` 인수만큼 밉살스런 코드도 없다. 선택자(selector) 인수는 목적을 기억하기 어려울 뿐 아니라 각 선택자 인수가 여러 함수를 하나로 조합한다. 선택자 인수는 큰 함수를
작은 함수 여럿으로 쪼개지 않으려는 게으름의 증거다. 다음 코드를 보자.

```java
public int calculateWeeklyPay(boolean overtime) {
    int tenthRate = getTenthRate();
    int tenthsWorked = getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime); 
    int straightPay = straightTime * tenthRate;
    double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate; 
    int overtimePay = (int)Math.round(overTime*overtimeRate);
    return straightPay + overtimePay;
}
```

초과근무 수당을 1.5배로 지급하면 `true`이고 아니면 `false`다. 독자는 `calculateWeeklyPay(false)`라는 코드를 발견할 때마다 의미를 떠올려야 한다. 

```java
public int straightPay() {
    return getTenthsWorked() * getTenthRate();
}

public int overTimePay() {
    int overTimeTenths = Math.max(0, getTenthsWorked() - 400); 
    int overTimePay = overTimeBonus(overTimeTenths);
    return straightPay() + overTimePay;
}

private int overTimeBonus(int overTimeTenths) {
    double bonus = 0.5 * getTenthRate() * overTimeTenths; 
    return (int) Math.round(bonus);
}
```

일반적으로, 인수를 넘겨 동작을 선택하는 대신 새로운 함수를 만드는 편이 좋다.

### G16: 모호한 의도

코드를 짤 땐 의도를 최대한 밝힌다. 행을 바꾸지 않고 표현한 수식, 헝가리식 표기법, 매직 넘버 등은 의도를 흐린다. 

```java

public int m_otCalc() { 
    return iThsWkd * iThsRte +
        (int) Math.round(0.5 * iThsRte * 
        Math.max(0, iThsWkd - 400)
        ); 
}
```

### G17: 잘못된 책임

소프트웨어 개발자가 내리는 가장 중요한 결정 중 하나가 코드를 배치하는 위치다. 예를 들어, `PI` 상수는 어디에 위치시켜야 할까? `Math`? `Trigonometry`? `Circle`?

여기서도 최소 놀람의 법칙(The Principle of Least Surprise)를 적용한다. 코드는 독자가 자연스럽게 기대할 위치에 배치한다. `PI` 상수는 삼각함수를 선언한 클래스에 넣어야 맞다. 
`OVERTIME_RATE` 상수는 `HourlyPayCalculator` 클래스에 선언해야 맞다.

### G18: 부적절한 static 함수

`Math.max(double a, double b)`는 좋은 `static` 메서드다. 특정 인스턴스와 관련된 기능이 아니다. 결정적으로 `Math.max` 메서드를 오버라이드할 가능성은 전혀 없다. 

우리는 간혹 `static`으로 정의하면 안 되는 함수를 `static`으로 정의하곤 한다. 다음 예제를 보자.

```java
HourlyPayCalculator.calculatePay(employee, overtimeRate).
```

특정 객체와 관련이 없으며 모든 정보를 인수에서 가져오기 때문에 `static` 함수로 여겨도 적당하다고 생각할 수 있다. 하지만 함수를 재정의할 가능성이 존재한다. 수당을 계산하는 알고리즘이 여러
개일지도 모른다. 예를 들어, `OvertimeHourlyPayCalculator`와 `StraightTimeHourlyPayCalculator`를 분리하고 싶을지도 모른다. 그러므로 `Employee` 클래스에 속하는 인스턴스 함수여야 한다.

일반적으로 `static` 함수보다 인스턴스 함수가 더 좋다. 반드시 `static` 함수로 정의해야겠다면 오버라이드할 가능성은 없는지 꼼꼼히 확인한다.

### G19: 서술적 변수

프로그램 가독성을 높이는 가장 효과적인 방법 중 하나가 계산을 여러 단계로 나누고 중간 값으로 서술적인 변수 이름을 사용하는 방법이다. 다음 예제를 보자.

```java
Matcher match = headerPattern.matcher(line); 
if(match.find())
{
    String key = match.group(1);
    String value = match.group(2); 
    headers.put(key.toLowerCase(), value);
}
```

서술적인 변수 이름을 사용하여 첫 번째로 일치하는 그룹이 `key`이고 두 번째로 일치하는 그룹이 `value`라는 사실이 명확히 드러난다.

### G20: 이름과 기능이 일치하는 함수

다음 코드를 보자.

```java
Date newDate = date.add(5);
```

5를 더하는 함수인가? 아니면 5주? 5시간? `date` 인스턴스르 변경하는 함수인가? 아니면 새로운 `Date` 함수를 반환하는 함수인가? 코드만 봐서는 알 수 없다.

만약 5일을 더해 `date` 인스턴스를 변경하는 함수라면 `addDaysTo` 혹은 `increaseByDays`라는 이름이 좋다. 반면 `date` 인스턴스는 변경하지 않으면서 새 `Date`를 반환한다면
`daysLater`나 `daysSince`라는 이름이 좋다.

이름만으로 분명하지 않기에 구현을 살피거나 문서를 뒤져야 한다면 더 좋은 이름으로 바꾸거나 아니면 더 좋은 이름을 붙이기 쉽도록 기능을 정리해야 한다.

### G21: 알고리즘을 이해하라

구현이 끝났다고 선언하기 전 함수가 돌아가는 방식을 확실히 이해했는지 확인하라. 테스트 케이스를 모두 통과한다는 사실만으론 부족하다. 작성자가 알고리즘이 올바르다는 사실을 알아야 한다.

알고리즘이 올바르다는 사실을 확인하고 이해하려면 기능이 뻔히 보일 정도로 함수를 깔끔하고 명확하게 재구성하는 방법이 최고다.

### G22: 논리적 의존성은 물리적으로 드러내라

한 모듈이 다른 모듈에 의존한다면 물리적인 의존성도 있어야 한다. 의존하는 모듈이 상대 모듈에 대해 뭔가를 가정하면(논리적으로 의존하면) 안 된다. 의존하는 모든 정보를 명시적으로 요청하는 편이 좋다.

예를 들어, 근무시간 보고서를 가공되지 않은 상태로 출력하는 함수를 구현한다고 가정하자. `HourlyReporter`라는 클래스는 모든 정보를 모아 `HourlyReportFormatter`에 적당한 형태로 넘긴다. 
`HourlyReportFormatter`는 넘어온 정보를 출력한다.

```java
public class HourlyReporter {
    private HourlyReportFormatter formatter;
    private List<LineItem> page;
    private final int PAGE_SIZE = 55;

    public HourlyReporter(HourlyReporter formatter) {
        this.formatter = formatter;
        page = new ArrayList<LineItem>();
    }

    public void generateReport(List<HourlyEmployee> employees) {
        for (HourlyEmployee e : employees) {
            addLineItemToPage(e);
            if (page.size() == PAGE_SIZE)
                printAndClearItemList();
        }
        if (page.size() > 0) printAndClearItemList();
    }

    private void printAndClearItemList() {
        formatter.format(page);
        page.clear();
    }

    private void addLineItemToPage(HourlyEmployee e) {
        LineItem item = new LineItem();
        item.name = e.getName();
        item.hours = e.getTenthsWorked() / 10;
        item.tenths = e.getTenthsWorked() % 10;
        page.add(item);
    }

    public class LineItem {
        public String name;
        public int hours;
        public int tenths;
    }
}
```

위 코드는 `PAGE_SIZE` 상수에 논리적인 의존성이 있다. 페이지 크기는 `HourlyReportFormatter`가 책임질 정보다. 이는 `HourlyReporter`에 선언한 잘못된 책임에 해당한다. `HourlyReporter`
클래스는 `HourlyReportFormatter`가 페이지 크기를 알 거라고 가정한다. 이 가정이 논리적 의존성이다. `HourlyReporter` 클래스는 `HourlyReportFormatter` 클래스가 페이지 크기 55를 처리할
줄 안다는 사실에 의존한다. 만약 `HourlyReportFormatter` 구현 중 하나가 페이지 크기 55를 제대로 처리하지 못한다면 오류가 생긴다.

`HourlyReportFormatter`에 `getMaxPageSize()`라는 메서드를 추가하면 논리적 의존성이 물리적 의존성으로 변한다. `HourlyReporter` 클래스는 `PAGE_SIZE`를 사용하는 대신 `getMaxPageSize()`
함수를 호출하면 된다.

### G23: If/Else 혹은 Switch/Case 문보다 다형성을 사용하라

3장에서 새 함수를 추가할 확률이 높은 코드에서는 `switch` 문이 더 적합하다고 주장했다.

1. 대다수 개발자가 `switch` 문을 사용하는 이유는 그 상황에서 올바른 선택이기보다는 당장 손쉬운 선택이기 때문이다. 그러므로 `switch`를 선택하기 전 다형성을 먼저 고려하여야 한다.
2. 타입보다 함수가 더 쉽게 변하는 경우는 드물다. 그러므로 모든 `switch` 문을 의심해야 한다.

### G24: 표준 표기법을 따르라

팀은 업계 표준에 기반한 구현 표준을 따라야 한다. 구현 표준은 인스턴스 변수 이름을 선언하는 위치, 클래스/메서드/변수 이름을 정하는 방법, 괄호를 넣는 위치 등을 명시해야 한다. 표준을 설명하는
문서는 코드 자체로 충분해야 하며 별도 문서를 만들 필요는 없어야 한다.

### G25: 매직 넘버는 명명된 상수로 교체하라

일반적으로 코드에서 숫자를 사용하지 말라는 규칙이다. 숫자는 명명된 상수 뒤로 숨기라는 의미다. 예를 들어, 86,400이라는 숫자는 `SECONDS_PER_DAY`라는 상수 뒤로 숨긴다. 쪽 당 55줄을 인쇄한다면 
숫자 55는 `LINE_PER_PAGE` 상수 뒤로 숨긴다. 어떤 상수는 이해하기 쉬우므로, 코드 자체가 자명하다면, 상수 뒤로 숨길 필요가 없다. 다음 코드를 보자.

```java
double milesWalked = feetWalked/5280.0;  // 5280은 잘 알려진 고유한 숫자다.
int dailyPay = hourlyRate * 8;           
double circumference = radius * Math.PI * 2;
```

매직 넘버는 단지 숫자만 의미하지 않는다. 의미가 분명하지 않은 토큰을 모두 가리킨다.

```java
assertEquals(7777, Employee.find(“John Doe”).employeeNumber());
```

위 `assert` 코드에서 매직 넘버는 두 개다. 하나는 `7777`이고, 다른 하나는 `John Doe`다. 둘 다 의미가 분명하지 않다. `John Doe`는 시급 직원 #7777의 이름이다. 따라서 
위 코드는 다음과 같이 해석된다.

```java
assertEquals(
        HOURLY_EMPLOYEE_ID, 
        Employee.find(HOURLY_EMPLOYEE_NAME).employeeNumber()
        );
```

### G26: 정확하라

코드에서 뭔가 결정할 때는 정확히 결정한다. 호출하는 함수가 `null`을 반환할지도 모른다면 `null`을 반드시 점검한다. 조회 결과가 하나뿐이라 짐작한다면 하나인지 확실히 확인한다. 통화를 다뤄야 한다면 정수를 
사용하고 반올림을 올바로 처리한다. 코드에서 모호성과 부정확은 의견차나 게으름의 결과다. 

### G27: 관례보다 구조를 사용하라

설계 결정을 강제할 때는 규칙보다 관례를 사용한다. 명명 관례도 좋지만 구조 자체로 강제하면 더 좋다. 예를 들어, `enum` 변수가 멋진 `switch/case` 문보다 추상 메서드가 있는 기초 클래스가 더 좋다.
`switch/case` 문을 매번 똑같이 구현하게 강제하기는 어렵지만, 파생 클래스는 추상 메서드를 모두 구현하지 않으면 안 되기 때문이다.

### G28: 조건을 캡슐화하라

`boolean` 논리는 이해하기 어렵다. 따라서 조건의 의도를 밝히는 함수로 표현하라.

```java
if (shouldBeDeleted(timer))
```

위 코드보다 아래의 코드가 더 좋다.

```java
if (timer.hasExpired() && !timer.isRecurrent())
```

### G29: 부정 조건은 피하라

부정 조건은 긍정 조건보다 이해하기 어렵다. 가능하면 긍정 조건으로 표현한다.

```java
if (buffer.shouldCompact())
```

위 코드가 아래 코드보다 더 좋다.

```java
if (!buffer.shouldNotCompact())
```

### G30: 함수는 한 가지만 해야 한다

큰 함수는 한 가지만 수행하는 작은 함수 여럿으로 나눠야 한다. 다음 코드를 보자.

```java
public void pay() {
    for (Employee e : employees) {
        if (e.isPayday()) {
            Money pay = e.calculatePay();
            e.deliverPay(pay);
        } 
    }
}
```

위 코드는 세 가지 임무를 수행한다. 

1. 직원 목록을 루프로 돌며
2. 각 지원의 월급일을 확인하고
3. 해당 직원에게 월급을 지급한다.

위 함수는 다음과 같이 나누는게 좋다.

```java
public void pay() {
    for (Employee e : employees)
        payIfNecessary(e); 
}

private void payIfNecessary(Employee e) {
    if (e.isPayday())
        calculateAndDeliverPay(e); 
}

private void calculateAndDeliverPay(Employee e) {
    Money pay = e.calculatePay(); 
    e.deliverPay(pay);
}
```

### G31: 숨겨진 시간적인 결합

때로는 시간적인 결합이 필요하다. 하지만 이를 숨겨서는 안 된다. 함수를 짤 때 함수 인수를 적절히 배치해 함수가 호출되는 순서를 명백히 드러낸다. 다음 코드를 보자.

```java
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        saturateGradient();
        reticulateSplines();
        diveForMoog(reason);
    }
    // ... 
}
```

위 코드에서 세 함수가 실행되는 순서가 중요하다. 
1. `gradient`를 처리하기 위해 `saturateGradient()`를 호출
2. `splines`를 처리하기 위해 `reticulateSplines()`를 호출
3. 마지막으로 `diveForMoog()`를 수행

위 코드는 이런 시간적인 결합을 강제하지 않는다. 다음 코드가 더 좋다.

```java
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        Gradient gradient = saturateGradient();
        List<Spline> splines = reticulateSplines(gradient);
        diveForMoog(splines, reason);
    }
    // ... 
}
```

위 코드는 연결 소자를 생성해 시간적인 결합을 노출한다. 각 함수가 내놓는 결과는 다음 함수에 필요하다. 그러므로 순서를 바꿔 호출할 수 없다.

인스턴스 변수를 그래도 두었다는 점에 주목하자. 해당 클래스의 `private` 메서드에 필요한 변수일지도 모른다. 그렇다 하더라도 제자리를 찾은 변수들이 시간적인 결합을 좀 더 
명백히 드러낼 것이다.

### G32: 일관성을 유지하라

코드 구조를 잡을 때는 이유를 고민하라. 그리고 그 이유를 코드 구조로 명백히 표현하라. 시스템 전반에 걸쳐 구조가 일관성이 있다면 남들도 일관성을 따르고 보존한다. 다음 코드를 보자.

```java
public class AliasLinkWidget extends ParentWidget {
    public static class VariableExpandingWidgetRoot {
        // ...
    }
}
```

문제는 `VariableExpandingWidgetRoot` 클래스가 `AliasLinkWidget` 클래스 범위에 속할 필요가 없다는 것이다. 게다가 `AliasLinkWidget`과 무관한 클래스가 `AliasLinkWidget.VariableExpandingWidgetRoot`를
사용했다. 이들은 `AliasLinkWidget` 클래스를 알 필요가 없다. 

이는 일관성이 없어 보인다. 다른 클래스의 유틸리티가 아닌 `pulbic` 클래스는 자신이 아닌 클래스 범위 안에서 선언하면 안 된다. 패키지 최상위 수준에 `public` 클래스로 선언하는 관례가 일반적이다.

### G33: 경계 조건은 캡슐화하라

경계 조건은 빼먹기 십상이다. 경계 조건은 한 곳에서 별도로 처리한다. 즉, 코드 여기저기에 +1, -1을 흩어놓지 않는다. 다음 코드를 보자.

```java
if(level + 1 < tags.length) {
    parts = new Parse(body, tags, level + 1, offset + endTag);
    body = null; 
}
```

`level + 1`이 두 번이나 나온다. 이런 경계 조건은 변수로 캡슐화하는 편이 좋다. 변수 이름은 `nextLevel`이 적합하겠다.

```java
int nextLevel = level + 1; 
if(nextLevel < tags.length) {
    parts = new Parse(body, tags, nextLevel, offset + endTag);
    body = null; 
}
```

### G34: 함수는 추상화 수준을 한 단계만 내려가야 한다

함수 내 모든 문장은 추상화 수준이 동일해야 한다. 그리고 그 추상화 수준은 함수 이름이 의미하는 작업보다 한 단계만 낮아야 한다. 다음 예를 보자.

```java
public String render() throws Exception {
    StringBuffer html = new StringBuffer("<hr"); 
    if(size > 0)
        html.append(" size=\"").append(size + 1).append("\"");
    html.append(">");
        
    return html.toString(); 
}
```

위 함수는 수평자(`<hr>`)를 만드는 HTML 태그를 생성한다. 수평자 높이는 `size` 변수로 지정한다. 

함수에는 추상화 수준이 최소 두 개가 섞여있다. 

1. 수평선에 크기가 있다는 개념
2. HR 태그 자체의 문법

이 모듈은 네 개 이상 연이은 대시(-)를 감지해 HR 태그로 변환한다. 대시 수가 많을수록 크기는 커진다. 

위 코드를 다음과 같이 수정한다 `size` 변수 이름은 목적을 반영하게 적절히 변경했다. `size` 변수는 추가된 대시 개수를 저장한다.

```java
public String render() throws Exception {
    HtmlTag hr = new HtmlTag("hr");
    if (extraDashes > 0)
        hr.addAttribute("size", hrSize(extraDashes)); 
    return hr.html();
}

private String hrSize(int height) {
    int hrSize = height + 1;
    return String.format("%d", hrSize); 
}
```

위 코드는 뒤섞인 추상화 수준을 분리한다. `render` 함수는 HR 태그만 생성한다. HR 태그 문법은 전혀 상관하지 않는다. HTML 문법은 `HtmlTag` 모듈이 알아서 처리한다. 또한 원래 코드의 HR 태그에 슬래시(/)를
추가하지 않았다. 하지만 XHTML 표준은 슬래시를 요구한다. `HtmlTag` 모듈은 오래 전부터 XHTML 표준을 준수했다. 

### G35: 설정 정보는 최상위 레벨에 둬라

추상화 최상위 레벨에 둬야 할 기본값 상수나 설정 관련 상수를 저차원 함수에 숨겨서는 안 된다. 대신 고차원 함수에서 저차원 함수를 호출할 때 인수로 넘긴다. 다음 코드를 보자.

```java
public static void main(String[] args) throws Exception {
    Arguments arguments = parseCommandLine(args);
    // ... 
}

public class Arguments {
    public static final String DEFAULT_PATH = ".";
    public static final String DEFAULT_ROOT = "FitNesseRoot";
    public static final int DEFAULT_PORT = 80;
    public static final int DEFAULT_VERSION_DAYS = 14;
    // ...
}
```

첫 행은 명령행 인수의 구문을 분석한다. 각 인수 기본값은 `Arguments` 클래스 맨 처음에 나온다. 다음과 같은 코드를 찾으려 시스템의 저수준을 뒤질 필요가 없다.

```java
if (arguments.port == 0) // use 80 by default
```

설정 관련 상수는 최상위 레벨에 둔다. 그래야 변경하기도 쉽다. 설정 관련 변수는 나머지 코드에 인수로 넘긴다. 저차원 함수에 상수 값을 정의하면 안 된다.

### G36: 추이적 탐색을 피하라

일반적으로 한 모듈은 주변 모듈을 모를수록 좋다. A가 B를 사용하고 B가 C를 사용하더라도 A가 C를 알아야 할 필요는 없다는 뜻이다. 예를 들어 `a.getB().getC().doSomething();`은 바람직하지 않다.
이를 디미터의 법칙(Law of Demeter)라고 부른다. 즉, 자신이 직접 사용하는 모듈만 알아야 한다는 뜻이다. 

내가 사용하는 모듈이 내게 필요한 서비스를 모두 제공해야 한다. 원하는 메서드를 찾느라 객체 그래프를 따라 시스템을 탐색할 필요가 없어야 한다. 다음과 같은 간단한 코드로 충분해야 한다.

```java
myCollaborator.doSomething();
```

## 자바

### J1: 긴 import 목록을 피하고 와일드카드를 사용하라

패키지에서 클래스를 둘 이상 사용한다면 와일드카드를 사용해 패키지 전체를 가져와라.

```java
import package.*;
```

명시적인 `import` 문은 강한 의존성을 생성하지만 와일드카드는 그렇지 않다. 와일드카드로 패키지를 지정하면 특정 클래스가 존재할 필요는 없다. `import` 문은 패키지를 단순히 검색 경로에 
추가하므로 진정한 의존성이 생기지 않는다. 그러므로 모듈 간 결합성이 낮아진다.

와일드카드 `import` 문은 때로 이름 충돌이나 모호성을 초래한다. 이름이 같으나 패키지가 다른 클래스는 명시적인 `import` 문을 사용하거나 아니면 코드에서 클래스를 사용할 때 전체 경로를 명시한다. 
다소 번거롭지만 자주 발생하지 않는다.

### J2: 상수는 상속하지 않는다

어떤 프로그래머는 상수를 인터페이스에 넣고 그 인터페이스를 상속해 해당 상수를 사용한다. 다음 코드를 보자.

```java
public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    private double hourlyRate;

    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
        int overTime = tenthsWorked - straightTime;
        return new Money(
                hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime));
    }
    // ... 
}
```

`TENTHS_PER_WEEK`와 `OVERTIME_RATE` 상수는 `Employee` 클래스에서 왔을지 모른다. `Employee` 클래스를 보자.

```java
public abstract class Employee implements PayrollConstants {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
```

아니다. 상수가 없다. `PayrollConstants`라는 인터페이스를 구현한다. 

```java
public interface PayrollConstants {
    public static final int TENTHS_PER_WEEK = 400;
    public static final double OVERTIME_RATE = 1.5;
}
```

상수를 상속 계층 맨 위에 숨겨놨다. 상속은 이렇게 사용하면 안 된다. 언어의 범위 규칙을 속이는 행위다. 대신 `import static`을 사용하라.

```java
import static PayrollConstants.*;

public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    private double hourlyRate;

    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
        int overTime = tenthsWorked - straightTime;
        return new Money(
                hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime));
    }
    // ... 
}
```

### J3: 상수 vs Enum

자바 5는 `enum`을 제공한다. `public static final int`라는 옛날 기교를 더 이상 사용할 필요가 없다. `enum`은 메서드와 필드도 사용할 수 있다. `int`보다 더 유연하고 서술적인 강력한 도구다.

```java
public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    HourlyPayGrade grade;

    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
        int overTime = tenthsWorked - straightTime;
        return new Money(
                grade.rate() * (tenthsWorked + OVERTIME_RATE * overTime));
    }
    // ... 
}

public enum HourlyPayGrade {
    APPRENTICE {
        public double rate() {
            return 1.0;
        }
    },
    LEUTENANT_JOURNEYMAN {
        public double rate() {
            return 1.2;
        }
    },
    JOURNEYMAN {
        public double rate() {
            return 1.5;
        }
    },
    MASTER {
        public double rate() {
            return 2.0;
        }
    };

    public abstract double rate();
}
```

## 이름

### N1: 서술적인 이름을 사용하라

서술적인 이름을 신중하게 고른다. 소프트웨어가 진화하면 의미도 변하므로 선택한 이름이 적합한지 자주 되돌아본다. 소프트웨어의 가독성의 90%는 이름이 결정한다. 

다음 코드를 보자. 아래의 코드는 미지의 숫자와 기호가 뒤섞인 잡탕에 불과하다.

```java
public int x() { 
    int q = 0; 
    int z = 0;
    for (int kk = 0; kk < 10; kk++) { 
        if (l[z] == 10) {
            q += 10 + (l[z + 1] + l[z + 2]);
            z += 1; 
        }
        else if (l[z] + l[z + 1] == 10) {
            q += 10 + l[z + 2];
            z += 2;
        } else {
            q += l[z] + l[z + 1];
            z += 2; 
        }
    }
    return q; 
}
```

위 코드는 다음과 같이 짰어야 한다. 

```java
public int score() {
    int score = 0;
    int frame = 0;
    for (int frameNumber = 0; frameNumber < 10; frameNumber++) {
        if (isStrike(frame)) {
            score += 10 + nextTwoBallsForStrike(frame); 
            frame += 1;
        } else if (isSpare(frame)) {
            score += 10 + nextBallForSpare(frame);
            frame += 2;
        } else {
            score += twoBallsInFrame(frame); 
            frame += 2;
        } 
    }
    return score; 
}
```

신중하게 선택한 이름은 추가 설명을 포함한 코드보다 강력하다. `isStrike()` 함수를 찾아보면 예상한 대로 돌아간다.

```java

private boolean isStrike(int frame) { 
    return rolls[frame] == 10;
}
```

### N2: 적절한 추상화 수준에서 이름을 선택하라

구현을 드러내는 이름은 피하라. 작업 대상 클래스나 함수가 위치하는 추상화 수준을 반영하는 이름을 선택하라. 코드를 살펴볼 때마다 추상화 수준이 너무 낮은 변수 이름을 발견하리라. 발견할 때마다
기회를 잡아 바꿔놓아야 한다. 안정적인 코드를 만들려면 지속적인 개선과 노력이 필요하다. 

다음 `Modem` 인터페이스를 살펴보자.

```java
public interface Modem {
    boolean dial(String phoneNumber);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedPhoneNumber();
}
```

대다수 애플리케이션에 문제는 없다. 하지만 전화선에 연결되지 않는 일부 모뎀을 사용하는 애플리케이션을 생각해보자. 전용선을 사용하는 모뎀을 고려해보자. 일부는 USB로 연결된 스위치에 포트 번호를 보낼지도 모른다.
그렇다면 전화번호라는 개념은 추상화 수준이 틀렸다. 더 좋은 이름 선택 전략은 다음과 같다.

```java
public interface Modem {
    boolean connect(String connectionLocator);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedLocator();
}
```

위 코드는 전화번호로 제한하지 않으며, 다른 연결 방식에도 사용 가능하다.

### N3: 가능하다면 표준 명명법을 사용하라

기존 명명법을 사용하는 이름은 이해하기 더 쉽다. 예를 들어, DECORATOR 패턴을 활용한다면 decorate하는 클래스 이름에 `Decorator`라는 단어를 사용해야 한다. 예를 들어, `AutoHangupModemDecorator`는 세션
끝 무렵에 자동으로 연결을 해제하는 기능으로 `Modem`을 decorate하는 클래스 이름에 적합하다.

프로젝트에 유효한 의미가 담긴 이름을 많이 사용할수록 독자가 코드를 이해하기 쉬워진다.

### N4: 명확한 이름

함수나 변수의 목적을 명확히 밝히는 이름을 선택한다. 

다음 코드를 보자.

```java
private String doRename() throws Exception {
    if(refactorReferences) 
        renameReferences();
    renamePage();
    
    pathToRename.removeNameFromEnd(); 
    pathToRename.addNameToEnd(newName); 
    return PathParser.render(pathToRename);
}
```

이름만 봐서는 함수가 하는 일이 분명하지 않다. `renamePageAndOptionallyAllReferences`라는 이름이 더 좋다. 길지만 모듈에서 한 번만 호출된다. 길다는 단점을 서술성이 충분히 메꾼다.

### N5: 긴 범위는 긴 이름을 사용하라

이름 길이는 범위에 비례해야 한다. 

범위가 5줄 안팎이라면 `i`나 `j`와 같은 변수 이름도 괜찮다. 

다음 코드를 보자. 깔끔한 코드다.

```java
private void rollMany(int n, int pins) {
    for (int i=0; i<n; i++) 
        g.roll(pins);
}
```

이름이 짧은 변수나 함수는 범위가 길어지면 의미를 잃는다. 그러므로 이름 범위가 길수록 이름을 정확하고 길게 짓는다.

### N6: 인코딩을 피하라

이름에 타입 정보나 범위 정보를 넣어서는 안 된다. 오늘날 IDE에서는 이름 앞에 `m_`이나 `f`와 같은 접두어가 불필요하다. 중복된 정보이며 혼란스럽게 만든다.

### N7: 이름으로 사이드 이펙트를 설명하라

함수, 변수, 클래스가 하는 일을 모두 기술하는 이름을 사용한다. 이름에 사이드 이펙트를 숨기지 않는다. 실제로 여러 작업을 수행하는 함수에 동사 하나만 달랑 사용하면 곤란하다.

다음 코드를 보자.

```java
public ObjectOutputStream getOos() throws IOException { 
    if (m_oos == null) {
        m_oos = new ObjectOutputStream(m_socket.getOutputStream()); 
    }
    return m_oos; 
}
```

위 함수는 단순히 `oos`만 가져오지 않는다. 기존에 `oos`가 없으면 생략한다. 그러므로 `createOrReturnOos`라는 이름이 더 좋다.

## 테스트

### T1: 불충분한 테스트

테스트 케이스는 잠재적으로 깨질 만한 부분을 모두 테스트해야 한다. 테스트 케이스가 확인하지 않는 조건이나 검증하지 않는 계산이 있다면 그 테스트는 불완전하다.

### T2: 커버리지 도구를 사용하라!

커버리지 도구는 테스트가 빠뜨리는 공백을 알려준다. 이를 사용해 테스트가 불충분한 모듈, 클래스, 함수를 찾기 쉬워진다. 

### T3: 사소한 테스트를 건너뛰지 마라

사소한 테스트는 짜기 쉽다. 사소한 테스트가 제공하는 문서적 가치는 구현에 드는 비용을 넘어선다.

### T4: 무시한 테스트는 모호함을 뜻한다

불분명한 요구사항은 테스트 케이스를 주석으로 처리하거나 테스트 케이스에 `@Ignore`륿 붙여 표현한다. 선택 기준은 모호함이 존재하는 테스트 케이스가 컴파일이 가능한지 불가능한지에 달려있다.

### T5: 경계 조건을 테스트하라

경계 조건은 각별히 신경 써서 테스트한다. 알고리즘은 올바로 짜놓고 경계 조건에서 실수하는 경우가 흔하다.

### T6: 버그 주변은 철저히 테스트하라

버그는 서로 모이는 경향이 있다. 한 함수에서 버그를 발견했다면 그 함수를 철저히 테스트하는 편이 좋다. 

### T7: 실패 패턴을 살펴라

때로는 테스트 케이스가 실패하는 패턴으로 문제를 진단할 수 있다. 테스트 케이스를 최대한 꼼꼼히 짜라는 이유도 여기에 있다. 합리적인 순서로 정렬된 꼼꼼한 테스트 케이스는 실패 패턴을 드러낸다.

### T8: 테스트 커버리지 패턴을 살펴라

통과하는 테스트가 실행하거나 실행하지 않는 코드를 살펴보면 실패하는 테스트 케이스의 실패 원인이 드러난다.

### T9: 테스트는 빨라야 한다

느린 테스트 케이스는 실행하지 않게 된다. 그러므로 테스트 케이스 빨리 돌아가게 최대한 노력한다.

## 결론

가치 체계가 이 책의 주제이자 목표다. 일부 규칙만 따른다고 깨끗한 코드가 얻어지지 않는다. 휴리스틱 목록을 익힌다고 장인이 되지는 못한다. 전문가와 장인 정신은 가치에서 나온다. 그 가치에 기반한
규율과 절제가 필요하다.