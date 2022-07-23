# Refactoring SerialDate

여기서는 `SerialDate` 클래스를 탐험한다. `SerialDate`는 날짜를 표현하는 자바 클래스다. 

## 첫째, 돌려보자

`SerialDateTests`라는 클래스는 유닛 테스트 케이스 몇 개를 포한한다. 하지만 테스트 케이스를 훑어보면 모든 경우를 점검하지 않는 사실이 드러난다. 예를 들어, `SerialDateTests` 클래스는
[monthCodeToQuarter](https://github.com/beomsu317/study/blob/main/contents/design/clean-code/chapter-16-refactoring-serial-date/code/SerialDate.java#L395) 
코드를 전혀 실행하지 않는다.

그래서 코드 커버리지 분석 도구인 클로버(Clover)를 이용해 유닛 테스트가 실행하는 코드와 실행하지 않는 코드를 조사했다. 대략 50% 정도였다. 그래서 독자적으로 유닛 테스트 케이스를 구현했다.
구현한 테스트 케이스에는 많은 코드가 주석으로 처리되었다. 이들은 실패한 테스트 케이스들이지만, 마땅히 통과해야 한다고 여겨지는 테스트 케이스다. 

[testStringToWeekdayCode](https://github.com/beomsu317/study/blob/main/contents/design/clean-code/chapter-16-refactoring-serial-date/code/BobsSerialDateTest.java#L17)
의 경우 대소문자 구문 없이 모두 통과해야 했다. `SerialDate.java`의 303행과 307행을 `s.equalsIgnoreCase`로 바꾸면 충분하다. 

`BobsSerialDateTest.java`의 33행과 46행은 주석으로 남겼는데, `tues`와 `thurs`라는 약어를 지원해야 할지가 분명치 않아서다.

`BobsSerialDateTest.java`의 154행과 155행은 통과하지 않는다. 이 코드는 통과해야 마땅하다. 고치기는 어렵지 않다.

`BobsSerialDateTest.java`의 163행에서 214행에 나오는 테스트도 고치기 쉽다. `stringToMonthCode` 메서드를 다음과 같이 수정한다.

```java
if ((result < 1) || (result > 12)) {
    for (int i = 0; i < monthNames.length; i++) {
        if (s.equalsIgnoreCase(shortMonthNames[i])) {
            result = i + 1;
            break;
        }
        if (s.equalsIgnoreCase(monthNames[i])) {
            result = i + 1;
            break;
        }
    }
}
```

`BobsSerialDateTest.java`의 321행은 `SerialDate.java` 698행 `getFollowingDayOfWeek`의 버그를 드러낸다. 2004년 12월 25일은 토요일이다. 다음 토요일은 2005년 1월 1일이다. 하지만
테스트를 돌리면 12월 25일을 다음 토요일로 반환한다. 전형적인 경계 조건 오류다. 올바른 코드는 다음과 같다.

```java
if (baseDOW >= targetWeekday) {
```

`SerialDate.java`의 729행 `getNearestDayOfWeek` 메서드를 테스트하는 `BobsSerialDateTest.java`의 332행 `testGetNearestDayOfWeek` 유닛 테스트 메서드는 처음부터 이렇게 길지 않았다. 처음
구현한 테스트 케이스가 실패하는 바람에 계속 추가하게 되었다.

`SerialDate.java`의 743행은 절대 실행되지 않는다. 즉, 743행에 있는 `if` 문이 항상 거짓이라는 의미이다. 올바른 알고리즘은 다음과 같다.

```java
int delta = targetDOW - base.getDayOfWeek(); 
int positiveDelta = delta + 7;
int adjust = positiveDelta % 7;
if (adjust > 3)
    adjust -= 7;

return SerialDate.addDays(adjust, base);
```

마지막으로 `BobsSerialDateTest.java`의 420행과 432행은 `weekInMonthToString`과 `relativeToString`에서 오류 문자열을 반환하는 대신 `IllegalArgumentException`을 발생시켜 테스트를
통과시켰다. 

위와 같이 변경한 `SerialDate`는 모든 테스트 케이스를 통과했다. 이제 `SerialDate` 코드를 올바로 고쳐보자.

## 둘째, 고쳐보자

`SerialDate` 코드를 처음부터 짚어가며 하나씩 고쳐보자. 코드를 고칠 때마다 유닛 테스트를 수행했다.

1행부터 살펴보자. 라이선스 정보, 저작권, 작성자, 변경 이력이 나온다. 법적인 정보는 필요하므로 라이선스 정보와 저작권을 보존한다. 변경 이력은 소스 코드 제어 도구를 사용하므로 제거한다.

60행에서 시작하는 `import` 문은 `java.text.*`과 `java.util.*`로 줄여도 된다.

67행부터 나오는 Javadoc 주석은 HTML 태그를 사용한다. 주석 전부를 `<pre>`로 감싸는 편이 좋다. 그러면 소스 코드에 보이는 형식이 Javadoc에 그대로 유지된다.

86행은 클래스 선언이다. 클래스 이름이 `SerialDate`인 이유는 일련번호(serial number)를 사용해 클래스를 구현했기 때문이다. 여기서는 1899년 12월 30일 기준으로
경과한 날짜 수를 사용한다. 856행 주석에서 확인할 수 있다.

두 가지가 맘에 들지 않는다.

1. 일련번호라는 용어는 정확하지 못하다. 일련번호보다 상대 오프셋(relative offset)이 더 정확하다. 그래서 `SerialDate`가 그다지 서술적인 이름이 아니라 생각하며, 좀 더 서술적인 용어로는 서수(ordinal)가 있다.
2. `SerialDate`라는 이름은 구현을 암시하는데 실상은 추상 클래스다. 구현을 암시할 필요가 없다. 따라서 이름의 추상화 수준이 올바르지 못하다고 생각한다. `Date`나 `Day`는 많이 사용되기 때문에 `DayDate`를 쓰기로 했다. 이제 `SerialDate` 클래스의 이름을 `DayDate`로 변경하여 사용한다.

`MonthConstants` 클래스는 달을 정의하는 `static final` 상수 모음에 불과하다. 상수 클래스를 상속하면 `MonthConstants.January`와 같은 표현을 사용할 필요가 없어진다. 예전에 많이 쓰던 기교지만, `enum`으로 정의해야 마땅하다.

```java
public abstract class DayDate implements Comparable, Serializable {
    public static enum Month {
        JANUARY(1), 
        FEBRUARY(2),
        MARCH(3),
        APRIL(4), 
        MAY(5),
        JUNE(6),
        JULY(7), 
        AUGUST(8), 
        SEPTEMBER(9), 
        OCTOBER(10), 
        NOVEMBER(11), 
        DECEMBER(12);

        Month(int index) {
            this.index = index;
        }

        public static Month make(int monthIndex) {
            for (Month m : Month.values()) {
                if (m.index == monthIndex) return m;
            }
            throw new IllegalArgumentException("Invalid month index " + monthIndex);
        }

        public final int index;
    }
}
```

`enum` 형식으로 변경하면 `DayDate` 클래스 코드에 상당한 변경이 필요하다. 하지만 달을 `int`로 받던 메서드는 이제 `Month`로 받는다. 즉, `BobsSerialDateTest.java`의 79행 `isValidMonthCode` 메서드를
없애도 된다는 뜻이다. 또한 394행 `monthCodeToQuarter`와 같은 오류 검사 코드도 더 이상 필요하지 않다.

92행 `serialVersionUID` 변수는 직렬화를 제어한다. 이 값을 변경하면 이전 버전에서 직렬화한 `DayDate`를 더 이상 인식하지 못한다. 즉, 이전 버전에서 직렬화한 `DayDate` 클래스를 복원하려 시도하면 `InvalidClassException`이
발생한다. `serialVersionUID` 변수를 선언하지 않으면 컴파일러가 자동으로 생성한다. 즉, 모듈을 변경할 때마다 변수 값이 달라진다. 문서는 직접 선언하라고 권하지만, 자동 제어가 안전하게 여겨진다. 따라서 해당 변수를 제거했다.

95행 주석은 불필요하다. 불필요한 주석은 거짓말과 잘못된 정보가 쌓이기 좋은 곳이다. 따라서 95행과 기타 유사한 주석을 제거했다.

103행, 108행은 일련번호를 언급한다. 두 변수는 `DayDate` 클래스가 표현할 수 있는 최초 날짜와 최후 날짜를 의미한다. 

```java
public static final int EARLIEST_DATE_ORDINAL = 2;      // 1/1/1900 
public static final int LATEST_DATE_ORDINAL = 2958465;  // 12/31/9999
```

`EARLIEST_DATE_ORDINAL`은 0이 아니라 2인 이유는 분명치 않다. 857헹 주석에 따르면 마이크로소프트 엑셀에서 날짜를 표현하는 방식과 관련이 있는 듯 보인다. `SpreadsheetDate.java`의 70행이 이를 잘 설명한다. 
하지만 이러한 사안은 `SpreadsheetDate`의 구현과 관련된 부분이므로, `EARLIEST_DATE_ORDINAL`, `LATEST_DATE_ORDINAL` 클래스로 옮겨져야 한다. 실제로 `SpreadsheetDate` 클래스만 두 변수를 사용한다.

다음은 `MINIMUM_YEAR_SUPPORTED`, `MAXIMUM_YEAR_SUPPORTED` 변수이다. `DayDate`는 추상 클래스로 구체적인 정보를 포함할 필요가 없다. 따라서 최소 년도와 최대 년도를 지정할 이유도 없다. 이 두 변수를  
`SpreadsheetDate`로 옮기려 했으나, `RelativeDayOfWeekRule.java`의 179행, 180행에서 넘어온 인수 `year`가 올바른지 확인할 목적으로 사용된다.

`DayDate`를 훼손하지 않으면서 구현 정보를 전달할 방법이 필요하다. 일반적으로 파생 클래스의 인스턴스로부터 구현 정보를 가져온다. 하지만 `getDate` 메서드로 넘어오는 인수는 `DayDate` 인스턴스가 아니다. 그런데 
`getDate`는 `DayDate` 인스턴스를 반환한다. 어디선가 `DayDate` 인스턴스를 생성한다는 말이다. 즉, `getPreviousDayOfWeek`, `getNearestDayOfWeek`, `getNearestDayOfWeek` 메서드 중 하나가 생성한다.
세 메서드 모두 `addDays`가 생성하는 `DayDate` 인스턴스를 반환한다. `DayDate.java`의 602행에서 `createInstance`를 호출해 `DayDate` 인스턴스를 생성한다. 837행을 보면 `createInstance`는 
`SpreadsheetDate` 인스턴스를 생성한다.

일반적으로 기반 클래스는 파생 클래스를 몰라야 한다. 그래서 ABSTRACT FACTORY 패턴을 적용해 `DayDateFactory`를 만들었다. `DayDate` 인스턴스를 생성하며 (최대 날짜와 최소 날짜 등) 구현 관련 질문에도 답한다.

```java
public abstract class DayDateFactory {
    private static DayDateFactory factory = new SpreadsheetDateFactory();

    public static void setInstance(DayDateFactory factory) {
        DayDateFactory.factory = factory;
    }

    protected abstract DayDate _makeDate(int ordinal);

    protected abstract DayDate _makeDate(int day, DayDate.Month month, int year);

    protected abstract DayDate _makeDate(int day, int month, int year);

    protected abstract DayDate _makeDate(java.util.Date date);

    protected abstract int _getMinimumYear();

    protected abstract int _getMaximumYear();

    public static DayDate makeDate(int ordinal) {
        return factory._makeDate(ordinal);
    }

    public static DayDate makeDate(int day, DayDate.Month month, int year) {
        return factory._makeDate(day, month, year);
    }

    public static DayDate makeDate(int day, int month, int year) {
        return factory._makeDate(day, month, year);
    }

    public static DayDate makeDate(java.util.Date date) {
        return factory._makeDate(date);
    }

    public static int getMinimumYear() {
        return factory._getMinimumYear();
    }

    public static int getMaximumYear() {
        return factory._getMaximumYear();
    }
}
```

위 클래스에서 `createInstance` 메서드를 `makeDate`로 변경했다. 이름이 좀 더 서술적이다. 기본적으로 `SpreadsheetDateFactory`를 사용하지만 필요하다면 변경해도 괜찮다. 추상 메서드로 위임하는 `static` 메서드는
SINGLETON, DECORATOR, ABSTRACT FACTORY 패턴 조합을 사용한다. `SpreadsheetDateFactory` 클래스는 다음과 같다.

```java
public class SpreadsheetDateFactory extends DayDateFactory {
    public DayDate _makeDate(int ordinal) {
        return new SpreadsheetDate(ordinal);
    }

    public DayDate _makeDate(int day, DayDate.Month month, int year) {
        return new SpreadsheetDate(day, month, year);
    }

    public DayDate _makeDate(int day, int month, int year) {
        return new SpreadsheetDate(day, month, year);
    }

    public DayDate _makeDate(Date date) {
        final GregorianCalendar calendar = new GregorianCalendar();
        calendar.setTime(date);
        return new SpreadsheetDate(
                calendar.get(Calendar.DATE), DayDate.Month.make(calendar.get(Calendar.MONTH) + 1), calendar.get(Calendar.YEAR));
    }

    protected int _getMinimumYear() {
        return SpreadsheetDate.MINIMUM_YEAR_SUPPORTED;
    }

    protected int _getMaximumYear() {
        return SpreadsheetDate.MAXIMUM_YEAR_SUPPORTED;
    }
}
```

위에서 보듯 `MINIMUM_YEAR_SUPPORTED`, `MAXIMUM_YEAR_SUPPORTED` 변수를 적절한 위치인 `SpreadsheetDate` 클래스로 옮겼다.

`DayDate`의 문제는 123행부터 나오는 요일 상수다. 이는 `enum`이여야 마땅하므로 변경해준다. 

다음으로 159행 `LAST_DAY_OF_MONTH`부터 시작해 일련의 배열이 나온다. 각 배열을 설명하는 주석이 거의 비슷하다. 변수 이름만으로 의미가 확실하므로 주석을 삭제한다. 또한 `LAST_DAY_OF_MONTH`는 `public` 변수일 필요가 없다.

다음으로 등장하는 `AGGREGATE_DAYS_TO_END_OF_MONTH` 배열은 어디서도 사용되지 않으므로 제거했다. `LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH` 변수도 마찬가지다.

다음으로 등장하는 배열 171행 `AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH`은 `SpreadsheetDate`에서만 사용한다. 이 변수는 특정 구현에 의존하지 않으므로 변수가 사용되는 위치에 가깝게 옮겼다. 일관성을 유지하려면
`LAST_DAY_OF_MONTH` 배열처럼 `private` 변수로 만들고 `julianDateOfLastDayOfMonth`와 같은 `static` 메서드로 노출해야 하겠다. 하지만 아무도 이런 메서드를 사용하지 않으므로 일단 `SpreadsheetDate`로 옮겼다.
`LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH` 배열도 마찬가지다.

190행 ~ 248행까지 나오는 상수는 `enum`으로 변환이 가능하다. 처음 다섯개의 상수는 달에서 주를 선택한다. 그래서 `WeekInMonth`라는 `enum`으로 변경했다. 

```java
public enum WeekInMonth {
    FIRST(1), SECOND(2), THIRD(3), FOURTH(4), LAST(0);
    public final int index;

    WeekInMonth(int index) {
        this.index = index;
    }
}
```

215행 ~ 230행에 나오는 상수는 다소 모호하다. 이들 상수는 범위 끝 날짜를 범위에 포함할지 여부를 결정한다. 수학적으로 개구간(open interval), 반개구간(half-open interval), 폐구간(closed interval)이라는 개념이다.
그래서 `enum` 이름을 `DateInterval`로 결정했으며, 각 값을 `CLOSED`, `CLOSED_LEFT`, `CLOSED_RIGHT`, `OPEN`이라 정의했다.

236행 ~ 248행은 주어진 날짜 기준으로 특정 요일을 계산할 때 사용한다. 각각 직전 요일, 다음 요일, 가장 가까운 요일을 의미한다. `enum` 이름은 `WeekdayRange`로 결정했다. 각 값은 `LAST`, `NEXT`, `NEAREST`로 정의했다.

253행의 `description` 변수는 사용하지 않으므로 변수와 관련된 메서드를 모두 제거했다. 

259행의 기본 생성자도 제거했다. 기본 생성자는 컴파일러가 기본적으로 생성한다.

270행 `isValidWeekdayCode` 메서드는 건너뛴다. `Day` `enum`을 생성할 때 삭제했다.

287행 ~ 313행의 유일한 의미있던 주석은 반환값 `-1`을 설명하는 행이지만, `Day` `enum`으로 변경했으므로 더 이상 유효하지 않아 주석 모두 제거한다. 메서드는 이제 `IllegalArgumentException`을 던진다. 

또한 인수와 변수 선언에서 `final` 키워드를 모두 없앴다. `final`을 제거한다는 결정은 일부 기존 관례에 어긋나지만, `final` 키워드는 `final` 상수 등 몇 군데를 제외하면 별다른 가치가 없으며 코드만 복잡하게 만든다. 

303행 ~ 307행에 `if` 문이 두 번 나오는데 `||`를 연산자를 사용해 `if` 문 하나로 만들었다. 또한 `Day` `enum`을 사용해 `for` 루프를 돌렸으며 몇 가지 미적인 변경도 가했다.

`stringToWeekdayCode` 메서드는 `DayDate` 클래스에 속하지 않는다. 사실 `Day`의 구문분석 메서드다. 그래서 이 메서드를 `Day`로 옮겼다. 실제로 `Day`라는 개념은 `DayDate`에 의존하지 않으므로
`Day`를 `DayDate` 클래스에서 빼낸 후 독자적인 소스 파일로 만들었다.

또한 315행에서 328행까지 `stringToWeekdayCode` 메서드도 `Day`로 옮기고 `toString`이라 명명했다.

```java
public enum Day {
    MONDAY(Calendar.MONDAY), 
    TUESDAY(Calendar.TUESDAY), 
    WEDNESDAY(Calendar.WEDNESDAY),
    THURSDAY(Calendar.THURSDAY), 
    FRIDAY(Calendar.FRIDAY),
    SATURDAY(Calendar.SATURDAY), 
    SUNDAY(Calendar.SUNDAY);
    
    public final int index;
    private static DateFormatSymbols dateSymbols = new DateFormatSymbols();

    Day(int day) {
        index = day;
    }

    public static Day make(int index) throws IllegalArgumentException {
        for (Day d : Day.values())
            if (d.index == index) return d;
        throw new IllegalArgumentException(String.format("Illegal day index: %d.", index));
    }

    public static Day parse(String s) throws IllegalArgumentException {
        String[] shortWeekdayNames =
                dateSymbols.getShortWeekdays();
        String[] weekDayNames =
                dateSymbols.getWeekdays();
        s = s.trim();
        for (Day day : Day.values()) {
            if (s.equalsIgnoreCase(shortWeekdayNames[day.index]) || s.equalsIgnoreCase(weekDayNames[day.index])) {
                return day;
            }
        }
        throw new IllegalArgumentException(
                String.format("%s is not a valid weekday string", s));
    }

    public String toString() {
        return dateSymbols.getWeekdays()[index];
    }
    }
```

330행 ~ 356행까지 `getMonths`라는 메서드 두 개가 나온다. 첫 번째 메서드는 두 번째 메서드를 호출한다. 두 번째 메서드를 호출하는 메서드는 첫 번째 뿐이다. 그래서 두 메서드를 하나로 합쳐 단순화했다.
그리고 이름을 좀 더 서술적으로 변경했다.

```java
public static String[] getMonthNames() { 
    return dateFormatSymbols.getMonths();
}
```

365행 ~ 385행까지 `isValidMonthCode` 메서드는 `Month` `enum`을 만들면서 필요가 없어졌다. 그래서 삭제했다.

387행 ~ 418행까지 `isValidMonthCode` 메서드는 기능 욕심(FEATRUE ENVY)으로 보인다. 대신 `Month`에 `quarter`라는 메서드를 추가했다.

```java
public int quarter() { 
    return 1 + (index - 1)/3;
}
```

그러고 나니 `Month`가 아주 커졌다. 그래서 `Day` `enum`과 일관성을 유지하도록 `DayDate`에서 분리해 독자적인 파일로 만들었다. 

429행 ~ 466행까지 `monthCodeToString` 메서드 두 개가 나온다. 앞서 나온 메서드와 마찬가지로, 한 메서드가 다른 메서드를 호출하며 플래그를 넘긴다. 일반적으로 메서드 인수로 플래그는 바람직하지 
않으므로, 두 메서드 이름을 변경하고, 단순화하고, `Month` `enum`으로 옮겼다. 

```java
public String toString() {
    return dateFormatSymbols.getMonths()[index - 1];
}
public String toShortString() {
    return dateFormatSymbols.getShortMonths()[index - 1];
}
```

468행 ~  510행까지 `stringToMonthCode` 메서드 역시 이름을 변경하고, `Month` `enum`으로 옮기고, 단순화했다.

```java

public static Month parse(String s) {
    s = s.trim();
    for (Month m : Month.values())
        if (m.matches(s)) return m;
    try {
        return make(Integer.parseInt(s));
    } catch (NumberFormatException e) {
    }
    throw new IllegalArgumentException("Invalid month " + s);
}

private boolean matches(String s) { 
    return s.equalsIgnoreCase(toString()) ||
        s.equalsIgnoreCase(toShortString());
} 
```

535행 ~ 553행까지 `isLeapYear` 메서드는 `DayDate`에 속하지 않는다. `SpreadsheetDate`에 있는 두 메서드를 제외하곤 호출하는 곳이 없다. 따라서 이를 아래로 이동했다.

573행 ~ 592행까지 `isLeapYear` 메서드는 `LAST_DAY_OF_MONTH` 배열을 사용한다. 그런데 이 배열은 `Month` `enum`에 속하므로 이 메서드도 옮기고, 좀 더 서술적인 표현으로 고쳤다.

```java
public static boolean isLeapYear(int year) { 
    boolean fourth = year % 4 == 0;
    boolean hundredth = year % 100 == 0;
    boolean fourHundredth = year % 400 == 0; 
    return fourth && (!hundredth || fourHundredth);
}
```

594행 ~ 607행까지 `addDays` 메서드가 나온다. 이 메서드는 온갖 `DayDate` 변수를 사용하므로 `static`이어서는 안 된다. 그래서 인스턴스 메서드로 변경했다. 또한 `toSerial` 메서드를 호출하는데,
이 메서드 이름을 `toOrdinal`로 변경했다. 

```java
public DayDate addDays(int days) {
    return DayDateFactory.makeDate(toOrdinal() + days);
}
```

609행 ~ 632행까지 `addMonths` 메서드도 마찬가지다. 또한 알고리즘도 다소 복잡하므로 임시 변수 설명(EXPLAINING TEMPORARY VARIABLES)를 사용해 좀 더 읽기 쉽게 고쳤다.

```java
public DayDate addMonths(int months) {
    int thisMonthAsOrdinal = 12 * getYear() + getMonth().index - 1; 
    int resultMonthAsOrdinal = thisMonthAsOrdinal + months; int resultYear = resultMonthAsOrdinal / 12;
    Month resultMonth = Month.make(resultMonthAsOrdinal % 12 + 1);
    
    int lastDayOfResultMonth = lastDayOfMonth(resultMonth, resultYear); 
    int resultDay = Math.min(getDayOfMonth(), lastDayOfResultMonth); 
    return DayDateFactory.makeDate(resultDay, resultMonth, resultYear);
}
```

634행 ~ 655행까지 `addYears` 메서드도 마찬가지다. 

```java
public DayDate plusYears(int years) {
    int resultYear = getYear() + years;
    int lastDayOfMonthInResultYear = lastDayOfMonth(getMonth(), resultYear); 
    int resultDay = Math.min(getDayOfMonth(), lastDayOfMonthInResultYear); 
    return DayDateFactory.makeDate(resultDay, getMonth(), resultYear);
}
```

인스턴스 메서드로 변경하며 뭔가 꺼림칙했다. `date.addDays(5)`라는 표현이 `date` 객체를 변경하지 않고 새 인스턴스를 반환한다는 사실이 드러날까? 다음 코드는 오해할 소지가 충분하다.

```java
DayDate date = DateFactory.makeDate(5, Month.DECEMBER, 1952); 
date.addDays(7); // bump date by one week.
```

코드를 읽는 사람은 `addDays`가 `date` 객체를 변경한다고 생각한다. 그러므로 이런 모호함을 해결할 이름이 필요하다. 그래서 `plusDays`와 `plusMonths`라는 이름을 선택했다.

```java
DayDate date = oldDate.plusDays(5);
```

반면 아래 코드는 단순히 `date` 객체가 변했다고 받아들이기 어렵다.

```java
date.plusDays(5);
```

657행 ~ 687행까지 `getPreviousDayOfWeek` 메서드는 올바로 동작하지만 복잡하다. 단순화하고 임시 변수 설명을 사용해 좀 더 읽기 쉽게 고쳤다. 또한 `static` 메서드를 인스턴스 메서드로
고쳤으며 1016행 ~ 1026행까지에 있는 중복된 인스턴스 메서드를 제거했다.

```java
public DayDate getPreviousDayOfWeek(Day targetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index-getDayOfWeek().index;
    if(offsetToTarget >= 0)
        offsetToTarget -= 7;
    return plusDays(offsetToTarget);
}
```

689행 ~ 718행까지 `getFollowingDayOfWeek` 메서드도 똑같은 원리를 적용해 고쳤다.

```java
public DayDate getFollowingDayOfWeek(Day targetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index; 
    if (offsetToTarget <= 0)
        offsetToTarget += 7;
    return plusDays(offsetToTarget);
}
```

720행 ~ 750행까지 `getNearestDayOfWeek` 메서드는 한 번 정정했으나, 앞서 가한 변경은 직전 두 메서드에서 가한 변경과 패턴이 다르다. 그래서 패턴을 맞추고 임시 변수 설명을 사용해 알고리즘을 
좀 더 명확하게 고쳤다.

```java
public DayDate getNearestDayOfWeek(final Day targetDay) {
    int offsetToThisWeeksTarget = targetDay.index - getDayOfWeek().index; 
    int offsetToFutureTarget = (offsetToThisWeeksTarget + 7) % 7;
    int offsetToPreviousTarget = offsetToFutureTarget - 7;
    if (offsetToFutureTarget > 3)
        return plusDays(offsetToPreviousTarget);
    else 
        return plusDays(offsetToFutureTarget);
}
```

752행 ~ 763행까지 `getEndOfCurrentMonth` 메서드는 조금 이상하다. `DayDate` 인스턴스를 인수로 받아 거기서 `DayDate` 클래스 메서드를 호출하는 인스턴스 메서드이기 때문이다. 진짜
인스턴스 메서드로 바꾸고 몇몇 이름도 고쳤다.

```java
public DayDate getEndOfMonth() {
    Month month = getMonth();
    int year = getYear();
    int lastDay = lastDayOfMonth(month, year);
    return DayDateFactory.makeDate(lastDay, month, year);
}
```

766행 ~ 790행까지 `weekInMonthToString`를 보자. 이전에 만든 `WeekInMonth` `enum`으로 옮긴 후 `toString`으로 변경하였다. 그 다음 `static` 메서드를 인스턴스 메서드로 고쳤다. 테스트를 돌리느 
모든 테스트를 통과했다. 

다음으로 메서드를 완전히 삭제했다. 그랬더니 `BobsSerialDateTest.java`의 413행에 있는 `testWeekInMonthToString` 테스트를 모두 실패했다. 그래서 테스트 케이스를 `FIRST`, `SECOND` 등 `enum` 값을
사용하도록 변경했다. 리팩토링 시 `weekInMonthToString` 메서드를 호출하던 코드를 모두 `weekInMonth.toString`으로 변경했다. 모든 `enum` 클래스는 `toString`을 사용해 이름을 반환하므로 테스트를 모두 통과한다.

실제로 이 메서드를 사용하는 코드는 방금 사용한 테스트 케이스가 유일했다. 그러므로 테스트 케이스를 삭제했다.

792행 ~ 813행까지 `relativeToString` 메서드 역시 테스트 케이스 외 아무도 호출하지 않았다. 그래서 메서드와 테스트 케이스를 모두 삭제했다.

856행 ~ 863행까지 `toSerial` 메서드는 앞서 `toOrdinal`로 변경했다. 메서드를 살펴본 후 `getOrdinalDay`로 바꿔야 맞다고 결정했다.


865행 ~ 871행까지 `toDate` 메서드는 `DayDate`를 `java.util.Date`로 변경한다. 구현한 `toDate` 메서드를 살펴보면 메서드는 `SpreadsheetDate`에 의존하지 않으므로 `DayDate` 메서드로 변경했다.

`getYYYY`, `getMonth`, `getDayOfMonth`는 제대로 된 추상 메서드다. 하지만 `getDayOfWeek` 메서드 역시 `SpreadsheetDate` 구현에 의존하지 않으므로 `DayDate`로 변경해도 괜찮다고 생각했다. 하지만 
`SpreadsheetDate.java`의 252행을 보면 알고리즘이 서수 날짜 시작일의 요일에 암시적으로 의존한다. 다시 말해, 0번째 날짜 요일에 의존한다. 따라서 `getDayOfWeek` 메서드는 `DayDate`로 옮기지 못한다. 
물리적 의존성은 없지만, 논리적 의존성이 존재하기 때문이다.

구현에 논리적으로 의존하면 물리적으로 의존해야 마땅하다. 그래서 `DayDate`에 `getDayOfWeekForOrdinalZero`라는 추상 메서드를 구현하고 `SpreadsheetDate`에서 `Day.SATURDAY`를 반환하도록 구현했다. 그 다음
`getDayOfWeek` 메서드를 `DayDate`로 옮긴 후 `getOrdinalDay`와 `getDayOfWeekForOrdinalZero`를 호출하도록 변경했다.

```java
public Day getDayOfWeek() {
    Day startingDay = getDayOfWeekForOrdinalZero();
    int startingOffset = startingDay.index - Day.SUNDAY.index; 
    return Day.make((getOrdinalDay() + startingOffset) % 7 + 1);
}
```

922행 ~ 926행까지의 주석은 반복되므로 제거한다. 

929행 ~ 939행까지 `compare` 메서드 역시 추상 메서드일 필요가 없다. 그래서 `SpreadsheetDate`에 있는 `compare` 메서드를 `DayDate`로 변경했다. 그래서 이름을 `daysSince`로 변경했다. 또한 테스트
케이스도 추가했다. 

991행 ~ 1000행까지 `isInRange` 역시 `DayDate`로 변경하였다. `if` 문 연쇄가 번거롭게 보여 `DateInterval` `enum`으로 옮기는 방법으로 연쇄를 없앴다.

```java
public enum DateInterval {
    OPEN {
        public boolean isIn(int d, int left, int right) {
            return d > left && d < right;
        }
    },
    CLOSED_LEFT {
        public boolean isIn(int d, int left, int right) {
            return d >= left && d < right;
        }
    }, CLOSED_RIGHT {
        public boolean isIn(int d, int left, int right) {
            return d > left && d <= right;
        }
    },
    CLOSED {
        public boolean isIn(int d, int left, int right) {
            return d >= left && d <= right;
        }
    };

    public abstract boolean isIn(int d, int left, int right); 
}
```

```java
public boolean isInRange(DayDate d1, DayDate d2, DateInterval interval) { 
    int left = Math.min(d1.getOrdinalDay(), d2.getOrdinalDay());
    int right = Math.max(d1.getOrdinalDay(), d2.getOrdinalDay());
    return interval.isIn(getOrdinalDay(), left, right);
}
```

이상으로 `DayDate` 클래스를 모두 살펴봤다. 다음은 지금까지 한 작업을 정리한다.

1. 처음에 나오는 주석은 너무 오래되어서, 간단하게 고치고 개선했다.
2. `enum`을 모두 독자적인 소스 파일로 옮겼다.
3. `static` 변수 `dateFormatSymboles`와 `static` 메서드 `getMonthNames`, `isLeapYear`, `lasyDayOfMonth`를 `DateUtil`이라는 새 클래스로 옮겼다.
4. 일부 추상 메서드를 `DayDate` 클래스로 변경했다.
5. `Month.make`를 `Month.fromInt`로 변경했다. 다른 `enum`도 똑같이 변경했다. 또한 모든 `enum`에 `toInt()` 접근자를 생성하고 `index` 필드를 `private`으로 정의했다.
6. `plusYears`와 `plusMonths`에 중복이 있어, `correctLastDayOfMonth`라는 새 메서드를 생성해 중복을 없앴다. 
7. 모든 곳에서 사용하던 숫자 1을 없앴다. 모두 `Month.JANUARY.toInt()`, `Day.SUNDAY.toInt()`로 적절히 변경했다. `SpreadsheetDate` 알고리즘을 일부 변경했다. 

## 결론

다시 한 번 보이스카우트 규칙을 따랐다. 체크아웃한 코드보다 좀 더 깨끗한 코드를 체크인했다. 테스트 커버리지가 증가했으며, 코드 크기가 줄었고, 코드가 명확해졌다. 