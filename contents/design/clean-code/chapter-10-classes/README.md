# Classes

## 클래스 체계

표준 자바 관례에 따르면 가장 먼저 변수 목록이 나온다. `static public` 상수, `static private` 변수, `private` 인스턴스 변수 순으로 나온다. `public` 변수가 필요한 경우는 거의 없다.

변수 목록 다음에는 `public` 함수가 나온다. `private` 함수는 자신을 호출하는 공개 함수 직후에 넣는다. 즉, 추상화 단계가 순차적으로 내려간다. 

### 캡슐화

변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다. 떄로는 변수나 유틸리티 함수를 `protected`로 선언해 테스트 코드에 접근을 허용하기도 한다. 같은 패키지 안에서
테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면 그 함수나 변수를 `protected`로 선언하거나 패키지 전체로 공개한다. 하지만 그 전에 `private` 상태를 유지할 방법을 강구해야 한다. 캡슐화는 푸는 결정은
언제나 최후의 수단이다.

## 클래스는 작아야 한다!

클래스를 만들 때 가장 중요한 규칙은 크기다. 클래스는 작아야 한다.

다음은 `SuperDashboard`라는 클래스로, `public` 메서드 수가 대략 70개 정도다. 

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public String getCustomizerLanguagePath();
    public void setSystemConfigPath(String systemConfigPath);
    public String getSystemConfigDocument();
    public void setSystemConfigDocument(String systemConfigDocument);
    public boolean getGuruState();
    public boolean getNoviceState();
    public boolean getOpenSourceState();
    // ... many methods follow ...
}
```

클래스 이름은 해당 클래스 책임을 기술해야 한다. 이는 클래스 크기를 줄이는 첫 번째 관문이다. 간결한 이름이 떠오르지 않는다면 클래스 크기가 너무 커서 그렇다. 클래스 이름이 모호하다면
클래스 책임이 너무 많아서다. 

클래스 설명은 `if`, `and`, `or`, `but`을 사용하지 않고서 25단어 내외로 가능해야 한다. 

* `SuperDashboard`는 마지막으로 포커스를 얻었던 컴포넌트에 접근하는 방법을 제공하며, 버전과 빌드 번호를 추적하는 메커니즘을 제공한다.

첫 번째 `~하며`는 `SuperDashboard`에 책임이 너무 많다는 증거다.

### 단일 책임 원칙

SRP는 클래스나 모듈을 변경할 이유가 하나여야 한다는 원칙이다.

`SuperDashboard`는 변경할 이유가 두 가지다. 

1. `SuperDashboard`는 소프트웨어 버전 정보를 추적한다. 그런데 버전 정보는 소프트웨어를 출시할 때마다 달라진다.
2. `SuperDashboard`는 자바 스윙 컴포넌트를 관리한다. 즉, 스윙 코드를 변경할 때마다 버전 번호가 달라진다.

책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다. `SuperDashboard`에서 버전 정보를 다루는 메서드를 따로 빼내 `Version`이라는 독자적인 클래스를 만든다. 이 클래스는
다른 애플리케이션에서 재사용하기 아주 쉬운 구조다.

```java
public class Version { 
    public int getMajorVersionNumber();
    public int getMinorVersionNumber();
    public int getBuildNumber();
}
```

따라서 큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다. 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

### 응집도(Coheison)

클래스는 인스턴스 변수 수가 작아야 한다. 각 클래스 메서드는 클래스 인스턴스 변수 하나 이상 사용해야 한다. 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스 응집도는 더 높다. 이는 클래스에 속한
메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이다. 

다음은 `Stack`을 구현한 코드다. 이 클래스는 응집도가 매우 높다. 

```java
public class Stack {
    private int topOfStack = 0;
    List<Integer> elements = new LinkedList<Integer>();

    public int size() {
        return topOfStack;
    }

    public void push(int element) {
        topOfStack++;
        elements.add(element);
    }

    public int pop() throws PoppedWhenEmpty {
        if (topOfStack == 0)
            throw new PoppedWhenEmpty();
        int element = elements.get(--topOfStack);
        elements.remove(topOfStack);
        return element;
    }
}
```

`함수를 작게, 매개변수 목록을 짧게`라는 전략을 따르다 보면 때때로 몇몇 메서드만 사용하는 인스턴스 변수가 많아진다. 이는 새로운 클래스로 쪼개야 한다는 신호다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다

큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다. 

큰 함수 일부를 작은 함수 하나로 빼내려고 한다. 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다고 가정하자. 그렇다면 이 네 변수를 클래스 인스턴스로 변수로 가져간다면 새 작은 함수는 인수가 필요 없다.
하지만 이러한 방식은 인스턴스 변수가 늘어나므로 응집력을 잃는다. 그런데 몇몇 함수가 몇몇 변수만 사용한다면 독자적인 클래스로 분리가 가능하다. 즉, 클래스가 응집력을 잃는다면 쪼개라!

다음 예제를 통해 함수/클래스를 여럿으로 쪼개보자. 

```java
package literatePrimes;

public class PrintPrimes {
    public static void main(String[] args) {
        final int M = 1000;
        final int RR = 50;
        final int CC = 4;
        final int WW = 10;
        final int ORDMAX = 30;
        int P[] = new int[M + 1];
        int PAGENUMBER;
        int PAGEOFFSET;
        int ROWOFFSET;
        int C;
        int J;
        int K;
        boolean JPRIME;
        int ORD;
        int SQUARE;
        int N;
        int MULT[] = new int[ORDMAX + 1];
        J = 1;
        K = 1;
        P[1] = 2;
        ORD = 2;
        SQUARE = 9;
        while (K < M) {
            do {
                J = J + 2;
                if (J == SQUARE) {
                    ORD = ORD + 1;
                    SQUARE = P[ORD] * P[ORD];
                    MULT[ORD - 1] = J;
                }
                N = 2;
                JPRIME = true;
                while (N < ORD && JPRIME) {
                    while (MULT[N] < J) 
                        MULT[N] = MULT[N] + P[N] + P[N];
                    if (MULT[N] == J) 
                        JPRIME = false;
                    N = N + 1;
                }
            } while (!JPRIME); 
            K = K + 1;
            P[K] = J;
        } {
            PAGENUMBER = 1;
            PAGEOFFSET = 1;
            while (PAGEOFFSET <= M) {
                System.out.println("The First " + M +
                        " Prime Numbers --- Page " + PAGENUMBER);
                System.out.println("");
                for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
                    for (C = 0; C < CC; C++)
                        if (ROWOFFSET + C * RR <= M)
                            System.out.format("%10d", P[ROWOFFSET + C * RR]);
                    System.out.println("");
                }
                System.out.println("\f");
                PAGENUMBER = PAGENUMBER + 1;
                PAGEOFFSET = PAGEOFFSET + RR * CC;
            }
        }
    }
}
```

함수가 하나뿐인 위 프로그램은 엉망진창이다. 다음은 작은 함수와 클래스로 나눈 후 함수, 클래스, 변수에 의미 있는 이름을 부여한 결과다. 

```java
package literatePrimes;

public class PrimePrinter {
    public static void main(String[] args) {
        final int NUMBER_OF_PRIMES = 1000;
        int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);
        final int ROWS_PER_PAGE = 50;
        final int COLUMNS_PER_PAGE = 4;
        RowColumnPagePrinter tablePrinter =
                new RowColumnPagePrinter(ROWS_PER_PAGE, COLUMNS_PER_PAGE,
                        "The First " + NUMBER_OF_PRIMES + " Prime Numbers");
        tablePrinter.print(primes);
    }
}
```

```java
package literatePrimes;

import java.io.PrintStream;

public class RowColumnPagePrinter {
    private int rowsPerPage;
    private int columnsPerPage;
    private int numbersPerPage;
    private String pageHeader;
    private PrintStream printStream;

    public RowColumnPagePrinter(int rowsPerPage, int columnsPerPage,
                                String pageHeader) {
        this.rowsPerPage = rowsPerPage;
        this.columnsPerPage = columnsPerPage;
        this.pageHeader = pageHeader;
        numbersPerPage = rowsPerPage * columnsPerPage;
        printStream = System.out;
    }

    public void print(int data[]) {
        int pageNumber = 1;
        for (int firstIndexOnPage = 0;
             firstIndexOnPage < data.length;
             firstIndexOnPage += numbersPerPage) {
            int lastIndexOnPage =
                    Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
            printPageHeader(pageHeader, pageNumber);
            printPage(firstIndexOnPage, lastIndexOnPage, data);
            printStream.println("\f");
            pageNumber++;
        }
    }

    private void printPage(int firstIndexOnPage, int lastIndexOnPage,
                           int[] data) {
        int firstIndexOfLastRowOnPage =
                firstIndexOnPage + rowsPerPage - 1;
        for (int firstIndexInRow = firstIndexOnPage;
             firstIndexInRow <= firstIndexOfLastRowOnPage;
             firstIndexInRow++) {
            printRow(firstIndexInRow, lastIndexOnPage, data);
            printStream.println("");
        }
    }

    private void printRow(int firstIndexInRow, int lastIndexOnPage,
                          int[] data) {
        for (int column = 0; column < columnsPerPage; column++) {
            int index = firstIndexInRow + column * rowsPerPage;
            if (index <= lastIndexOnPage)
                printStream.format("%10d", data[index]);
        }
    }

    private void printPageHeader(String pageHeader, int pageNumber) {
        printStream.println(pageHeader + " --- Page " + pageNumber);
        printStream.println("");
    }

    public void setOutput(PrintStream printStream) {
        this.printStream = printStream;
    }
}
```

```java
package literatePrimes;

import java.util.ArrayList;

public class PrimeGenerator {
    private static int[] primes;
    private static ArrayList<Integer> multiplesOfPrimeFactors;

    protected static int[] generate(int n) {
        primes = new int[n];
        multiplesOfPrimeFactors = new ArrayList<Integer>();
        set2AsFirstPrime();
        checkOddNumbersForSubsequentPrimes();
        return primes;
    }

    private static void set2AsFirstPrime() {
        primes[0] = 2;
        multiplesOfPrimeFactors.add(2);
    }

    private static void checkOddNumbersForSubsequentPrimes() {
        int primeIndex = 1;
        for (int candidate = 3;
             primeIndex < primes.length;
             candidate += 2) {
            if (isPrime(candidate))
                primes[primeIndex++] = candidate;
        }
    }

    private static boolean isPrime(int candidate) {
        if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
            multiplesOfPrimeFactors.add(candidate);
            return false;
        }
        return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
    }

    private static boolean isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
        int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
        int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
        return candidate == leastRelevantMultiple;
    }

    private static boolean isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
        for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
            if (isMultipleOfNthPrimeFactor(candidate, n)) return false;
        }
        return true;
    }

    private static boolean isMultipleOfNthPrimeFactor(int candidate, int n) {
        return
                candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
    }

    private static int
    smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
        int multiple = multiplesOfPrimeFactors.get(n);
        while (multiple < candidate)
            multiple += 2 * primes[n];
        multiplesOfPrimeFactors.set(n, multiple);
        return multiple;
    }
}
```

가장 먼저 눈에 띄는 변화가 프로그램이 길어졌다는 사실이다. 길이가 늘어난 이유는 다음과 같다. 

1. 리팩터링한 프로그램은 좀 더 길고 서술적인 변수 이름을 사용한다. 
2. 리팩터링한 프로그램은 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용한다. 
3. 가독성을 높이고자 공백을 추가하고 포맷을 맞추었다.

* `PrimePrinter`: `main` 함수를 포함하며 실행 환경을 책임진다. 호출 방식이 다라지면 클래스도 바뀐다. 예를 들어, SOAP 서비스로 바꾸려면 `PrimePrinter` 클래스를 수정한다.
* `RowColumnPagePrinter`: 숫자 목록을 주어진 행과 열에 맞춰 페이지에 출력하는 방법을 안다. 출력하는 모양을 변경하려면 이 클래스를 수정한다.
* `PrimeGenerator`: 소수 목록을 생성하는 방법을 안다. 이는 객체로 인스턴스화하는 클래스가 아니며, 단순히 변수를 선언하고 감추려고 사용하는 공간일 뿐이다. 소수를 계산하는 알고리즘이 바뀐다면 이 클래스를 수정한다.

먼저 원래 프로그램의 정확한 동작을 검증하는 테스트 슈트를 작성했다. 그 다음, 한 번에 하나씩 수 차례에 걸쳐 코드를 변경했다. 코드를 변경할 때마다 원래 프로그램과 동일하게 동작하는지 확인했다. 

## 변경하기 쉬운 클래스

대부분의 시스템은 지속적인 변경이 가해진다. 그리고 변경할 때마다 의도되지 않은 위험이 따른다. 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

다음은 주어진 메타 데이터로 적절한 SQL 문자열을 만드는 `Sql` 클래스다. 아직 미완성이라 `update` 문과 같은 일부 SQL 기능을 지원하지 않는다. `update` 문을 지원할 시점이 오면
클래스에 `손대어` 고쳐야 한다. 문제는 코드에 `손대면` 위험이 생긴다는 사실이다. 어떤 변경이든 클래스에 손대면 다른 코드를 망가뜨릴 위험이 존재한다.

```java
public class Sql {
    public Sql(String table, Column[] columns);
    public String create();
    public String insert(Object[] fields);
    public String selectAll();
    public String findByKey(String keyColumn, String keyValue);
    public String select(Column column, String pattern);
    public String select(Criteria criteria);
    public String preparedInsert();
    private String columnList(Column[] columns);
    private String valuesList(Object[] fields, final Column[] columns);
    private String selectWithCriteria(String criteria);
    private String placeholderList(Column[] columns);
}
```

새로운 SQL 구문을 지원하려면 반드시 `Sql` 클래스에 손대야 한다. 또한 기존 SQL 문을 수정할 때도 반드시 `Sql` 클래스를 고쳐야 한다. 이렇듯 변경할 이유가 두 가지이므로 `Sql` 클래스는
SRP를 위반한다. 구조적인 관점에서도 SRP를 위반한다. `selectWithCriteria`라는 `private` 메서드가 있는데, 이 메서드는 `select` 문을 처리할 때 사용한다.

`public` 인터페이스를 각각 `Sql` 클래스에서 파생하는 클래스로 만들었다. `valuesList`와 같은 비공개 메서드는 해당하는 파생 클래스로 옮겼다. 모든 파생 클래스가 공통으로 사용하는 `private`
메서드는 `Where`과 `ColumnList`라는 두 유틸리티 클래스에 넣었다.

```java
abstract public class Sql {
    public Sql(String table, Column[] columns);
    abstract public String generate();
}

public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns);
    @Override public String generate();
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns); 
    @Override public String generate();
}

public class InsertSql extends Sql { 
    public InsertSql(String table, Column[] columns, Object[] fields); 
    @Override public String generate();
    private String valuesList(Object[] fields, final Column[] columns);
}

public class SelectWithCriteriaSql extends Sql { 
    public SelectWithCriteriaSql(
            String table, Column[] columns, Criteria criteria); 
    @Override public String generate();
}

public class SelectWithMatchSql extends Sql { 
    public SelectWithMatchSql(
            String table, Column[] columns, String pattern);
    @Override public String generate();
}

public class FindByKeySql extends Sql {
    public FindByKeySql(
            String table, Column[] columns, String keyColumn, String keyValue);
    @Override public String generate();
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns);
    @Override public String generate();
    private String placeholderList(Column[] columns);
}

public class Where {
    public Where(String criteria);
    public String generate();
}

public class ColumnList {
    public ColumnList(Column[] columns);
    public String generate();
}
```

각 클래스는 단순하다. 코드는 순식간에 이해된다. 함수 하나를 수정했다고 다른 함수가 망가질 위험도 없어졌다. 테스트 관점에서 모든 논리를 증명하기도 쉬워졌다. 클래스가 서로 분리되었기 때문이다.

`update` 문을 추가할 때 기존 클래스를 변경할 필요가 없다는 사실에 주목하자. `update` 문을 만드는 논리는 `Sql` 클래스를 상속받아 새 클래스 `UpdateSql`에 넣으면 그만이다.

따라서 재구성한 `Sql` 클래스는 SRP, OCP를 지원한다.

새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직하다. 이상적인 시스템은 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하지 않는다.

### 변경으로부터 격리

요구사항이 변하기 마련이므로, 코드도 변하기 마련이다. 상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다. 그래서 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

상세한 구현에 의존하는 코드는 테스트가 어렵다. 예를 들어, `Portfolio` 클래스를 만든다고 가정하자. `Portfolio` 클래스는 외부 `TokyoStockExchange` API를 사용해 값을 계산한다. 
따라서 테스트 코드는 시세 변화에 영향을 받는다. 

`Portfolio` 클래스에서 `TokyoStockExchange` API를 직접 호출하는 대신 `StockExchange`라는 인터페이스를 생성한 후 메서드 하나를 선언한다.

```java
public interface StockExchange {
    Money currentPrice(String symbol);
}
```

다음 `StockExchange` 인터페이스를 구현하는 `TokyoStockExchange` 클래스를 구현한다. 또한 `Portfolio` 생성자를 수정해 `StockExchange` 참조자를 인수로 받는다.

```java
public Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
        this.exchange = exchange; 
    }
    // ... 
}
```

이제 `TokyoStockExchange` 클래스를 흉내내는 테스트용 클래스를 만들 수 있다. 테스트용 클래스는 `StockExchange` 인터페이스를 구현하며 고정된 주가를 반환한다.

```java
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub();
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value());
    }
}
```

위와 같은 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 높아진다. 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.

이렇게 결합도를 줄이면 자연스럽게 DIP를 따르는 클래스가 나온다. DIP는 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙이다.

개선한 `Portfolio` 클래스는 `TokyoStockExchange`라는 상세한 구현 클래스가 아닌, `StockExchange` 인터페이스에 의존한다. 이와 같은 추상화로 실제 주가를 얻어오는
출처나 얻어오는 방식 등과 같은 구체적인 사실을 모두 숨긴다.