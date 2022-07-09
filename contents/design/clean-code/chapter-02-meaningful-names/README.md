# Meaningful Names

## 들어가면서

변수, 함수, 인수, 클래스, 패키지, 소스 파일 등에 이름을 붙이므로 이름을 잘 지으면 여러모로 편하다.

## 의도를 분명히 밝혀라

좋은 이름을 지으려면 시간이 걸리지만 좋은 이름으로 절약하는 시간이 훨씬 더 많다.

다음 코드에서 이름 `d`는 아무 의미도 드러나지 않는다.

```java
int d; // elapsed time in days
```

측정하려는 값과 단위를 표현하는 이름이 필요하다.

```java
int elapsedTimeInDays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;
```

다음 코드는 무엇을 의미할까?

```java
public List<int[]>getThem() {
    List<int[]>list1=new ArrayList<int[]>();
    for(int[]x:theList)
        if(x[0]==4)
            list1.add(x);
    return list1;
}
```

코드 맥락이 코드 자체에 명시적으로 드러나지 않는다. 다음과 같은 정보를 안다고 가정한다.

1. theList에 무엇이 들어있는가?
2. theList에서 0번째 값이 어째서 중요한가?
3. 값 4는 무슨 의미인가?
4. 함수가 반환하는 리스트 list1을 어떻게 사용하는가?

위 코드엔 이와 같은 정보를 충분히 제공할 수 있었지만 드러나지 않는다.

지뢰찾기 게임을 만든다고 가정하자. `theList`가 게임판인 사실을 안다. `theList`를 `gameBoard`로 변경해보자. 배열에서 0번째 값은 칸 상태를 뜻한다. 값 4는 깃발이 꽂힌 상태를 가리킨다.
각 개념에 이름만 붙여도 코드가 상당히 나아진다.

```java
public List<int[]>getFlaggedCells() {
    List<int[]>flaggedCells=new ArrayList<int[]>();
    for(int[]cell:gameBoard)
        if(cell[STATUS_VALUE]==FLAGGED)
            flaggedCells.add(cell);
    return flaggedCells;
}
```

int 배열을 사용하는 대신, 칸을 간단한 클래스로 만들고, `isFlagged`라는 좀 더 명시적인 함수를 사용해 `FLAGGED`라는 상수를 감춰보자.

```java
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells=new ArrayList<Cell>();
    for(Cell cell:gameBoard)
        if(cell.isFlagged())
            flaggedCells.add(cell);
    return flaggedCells;
}
```

단순히 이름만 고쳤는데도 함수가 하는 일을 이해하기 쉬워졌다.

## 그릇된 정보를 피하라

그릇된 단서는 코드의 의미를 흐린다. 널리 쓰이는 단어를 다른 의미로 사용해도 안 된다. 예를 들어, hp, six, sco는 변수 이름으로 적합하지 않다.

여러 계정을 그룹으로 묶을 때, 실제 `List`가 아니라면, `accountList`라 명명하지 않아야 한다. 대신 `accountGroup`, `bunchOfAccounts`, `accounts`라 명명하는게
좋다.

서로 흡사한 이름을 사용하지 않도록 주의한다. 한 모듈에서 `XYZControllerForEfficientHandlingOfStrings`를 사용하고, 다른
모듈에서 `XYZControllerForEfficientStorageOfStrings`를 사용한다고 가정하자. 유사한 개념은 유사한 표기법을 사용한다. 일관성이 떨어지는 표기법은 그릇된 정보다.

이름으로 그릇된 정보를 제공하는 끔찍한 예는 소문자 L이나 대문자 O 변수이다. 두 변수를 한 번에 사용하면 더욱 끔찍해진다.

```java
int a = l;
if ( O == l )
    a = O1; 
else
    l = 01;
```

이름만 바꾸면 이러한 문제가 깨끗이 풀린다.

## 의미 있게 구분하라

연속된 숫자를 붙이거나 불용어(noise/stop word: 의미가 없는 단어들)를 추가하는 방식은 적절하지 않다. 연속적인 숫자를 덧붙인 이름(a1, a2, ... aN)은 의도적인 
이름이 아니며, 아무런 정보를 제공하지 않는다. 다음 코드를 보자.

```java
public static void copyChars(char a1[], char a2[]) { 
    for (int i = 0; i < a1.length; i++) {
        a2[i] = a1[i]; 
    }
}
```

이 함수 인자로 `source`, `destination`을 사용한다면 코드 읽기가 훨씬 더 쉬워진다.

불용어를 추가한 이름도 역시 아무런 정보도 제공하지 못한다. `Porduct`라는 클래스가 있다고 가정하자. 다른 클래스를 `ProductInfo` 또는 `ProductData`라 한다면 구분이 어렵다. 
Info 또는 Data는 a, an, the와 마찬가지로 의미가 불분명한 불용어다.

불용어는 중복이다. `Customer` 클래스와 `CustomerObject`라는 클래스가 존재한다면 차이를 알 수 없다. 

명확한 관례가 없다면 변수 `moneyAmount`는 `money`와 구분이 안 된다. `customerInfo`는 `customer`와, `accountData`는 `account`와, `theMessage`는 `message`와 구분되지 않는다.
읽는 사람이 차이를 알도록 이름을 지어라.

## 발음하기 쉬운 이름을 사용해라.

사람들은 단어에 능숙하다. 그러므로 발음하기 쉬운 이름을 선택한다. 다음 두 클래스를 비교해보자.

```java
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102"; /* ... */
};
```

```java
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
}
```

두 번째 코드는 지적인 대화가 가능하다. 

## 검색하기 쉬운 이름을 사용하라

하나의 문자 또는 상수를 사용하면 텍스트 코드에서 쉽게 눈에 띄지 않는다. 긴 이름이 짧은 이름보다 좋다.

필자는 간단한 메서드에서 로컬 변수만 한 문자를 사용한다. 즉, **이름 길이는 범위 크기에 비례해야 한다.** 다음 두 코드를 비교해보자.

```java
for (int j=0; j<34; j++) { 
    s += (t[j]*4)/5;
}
```

```java
int realDaysPerIdealDay = 4;
    const int WORK_DAYS_PER_WEEK = 5;
    int sum = 0;
    for (int j=0; j < NUMBER_OF_TASKS; j++){
        int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
        int realTaskWeeks = (realdays / WORK_DAYS_PER_WEEK);
    sum+=realTaskWeeks;
}
```

두 번째 코드는 `sum`을 통해 검색이 가능하다. 이름을 의미있게 지으면 함수가 길어지지만 `WORK_DAYS_PER_WEEK` 등을 통해 찾기가 쉬워진다. 

## 인코딩을 피하라

문제 해결에 집중하는 개발자에게 인코딩은 부담이다. 

### 헝가리식 표기법

예전 언어들에는 첫 글자로 유형을 표현(헝가리식 표기법)했다. 

요즘 프로그래밍 언어는 훨씬 많은 타입을 지원하며, 컴파일러가 타입을 기억하고 강제한다. 게다가 클래스와 함수는 점차 작아지는 추세다. 즉, 변수를 선언한 위치와 사용하는 위치가 멀지 않다.

자바 프로그래머는 타입을 인코딩할 필요가 없다. 객체는 강한 타입(strongly-typed)이며, IDE는 코드를 컴파일하지 않고도 타입 오류를 감지할 정도로 발전했다.

```java
PhoneNumber phoneString;
// name not changed when type changed!
```

### 멤버 변수 접두어

이제 멤버 변수에 `m_`이라는 접두어를 붙일 필요도 없다. 클래스와 함수는 접두어가 필요없을 정도로 작아야 마땅하다. 또한 멤버 변수를 다른 색상으로 표시하거나 눈에 띄게 보여주는 IDE를 사용해야 한다.


```java
public class Part {
    private String m_dsc; // The textual description void setName(String name) {
    void setName(String name) {
        m_dsc = name; 
    }
}
```

```java
public class Part {
    String description;
    void setDescription(String description) {
        this.description = description; 
    }
}
```

사람들은 접두어(또는 접미어)를 무시하고 이름을 해독하는 방식을 빨리 익힌다. 즉, 접두어는 옛날에 작성한 구닥다리 코드라는 징표가 돼버린다.

### 인터페이스 클래스와 구현 클래스

때로는 인코딩이 필요한 경우도 있다. 예를 들어, 도형을 생성하는 ABSTRACT FACTORY를 구현한다고 가정하자. 이 팩토리는 인터페이스 클래스다. 구현은 구체 클래스(concrete class)에서 한다. 
두 클래스의 이름을 `IShapeFactory`와 `ShapeFactory`로 작성해보자. 인터페이스 클래스의 접두어 `I`는 주의를 흐트리고 과도한 정보를 제공한다. 다루는 클래스가 인터페이스라는 사실을
알리지 않고 사용자는 그냥 `ShapeFactory`라고 생각하게 하고 싶다면, 구현 클래스에 인코딩해야 한다. `ShapeFactoryImpl`나 `CShapeFactory`가 `IShapeFactory` 보다 좋다.

## 자신의 기억력을 자랑하지 마라

`r`이라는 변수가 호스트와 프로토콜을 제외한 소문자 URL이라는 사실을 기억한다면 똑똑한 프로그래머이다. 하지만 전문가 프로그래머는 **명료함이 최고**라는 사실을 이해하고 있다. 따라서 자신의 능력을
좋은 방향으로 사용해 남들이 이해하는 코드를 내놓는다.

## 클래스 이름

클래스 이름과 객체 이름은 명사나 명사구가 적합하다. `Customer`, `WikiPage`, `Account` 등. `Manager`, `Processor`, `Data`, `Info` 등과 같은 단어는 피하고, 동사는 사용하지 않는다.

## 메서드 이름

메서드 이름은 동사나 동사구가 적합하다. `postPayment`, `deletePage`, `save` 등이 좋은 예다. Accessor(접근자), Mutator(변경자), Predicate(조건자)는 자바빈 표준에 따라 앞에 `get`, `set`, `is`를 붙인다. 

```java
string name = employee.getName(); 
customer.setName("mike");
if (paycheck.isPosted())...
```

생성자를 오버로드할 때는 정적 팩토리 메서드를 사용한다. 메서드는 인수를 설명하는 이름을 사용한다. 

```java
Complex fulcrumPoint = Complex.FromRealNumber(23.0);
```

위 코드가 다음 코드보다 좋다.

```java
Complex fulcrumPoint = new Complex(23.0);
```

## 기발한 이름은 피하라

이름이 너무 기발하면 저자와 유머 감각이 비슷한 사람만, 농담을 기억하는 동안만 이름을 기억한다. 의도를 분명하고 솔직하게 표현해라.

## 한 개념에 한 단어를 사용해라

추상적인 개념 하나에 단어 하나를 선택해 이를 고수한다. 예를 들어, 똑같은 메서드를 클래스마다 `fetch`, `retrieve`, `get`으로 각각 정의하면 혼란스럽다. 메서드 이름은 독자적이고 일관적이어야, 
주석을 보지 않고도 프로그래머가 올바른 메서드를 선택한다.

## 말장난을 하지 마라

한 단어를 두 가지 목적으로 사용하지 마라. 다른 개념에 같은 단어를 사용한다면 그것은 말장난에 불과하다. 

예를 들어, 지금까지 구현한 `add` 메서드는 모두 기존 값 두 개를 더하거나 이어서 새로운 값을 만든다고 가정하자. 새로 작성하는 메서드는 집합에 값 하나를 추가한다. 이 메서드는 기존 `add` 메서드와
맥락이 다르므로 `insert`, `append`라는 이름이 적당하다. 새 메서드를 `add`라 부른다면 이는 말장난이다.

## 해법 영역에서 가져온 이름을 사용하라

코드를 읽는 사람도 프로그래머이므로 전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어 등을 사용해도 괜찮다. 모든 이름을 문제 영역(domain)에서 가져오게 되면, 매번 고객에에 의미를 물어야 한다.

## 문제 영역에서 가져온 이름을 사용하라

적절한 프로그래머 용어가 없다면 문제 영역에서 이름을 가져온다. 그러면 코드를 보수하는 프로그래머가 분야 전문가에게 의미를 물어 파악할 수 있다. 문제 영역 개념과 관련이 깊은 코드라면 문제 영역에서
이름을 가져와야 한다.

## 의미 있는 맥락을 추가하라

스스로 의미 있는 이름이 있지만 대부분은 그렇지 않다. 그래서 클래스, 함수 등에 맥락을 부여한다. 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다. 

예를 들어, `firstName`, `lastName`, `street`, `houseNumber`, `city`, `state`라는 변수들이 있다고 가정하자. 변수들을 보면 주소라는 사실을 알아챌 수 있다. 하지만 `state` 변수만을
사용하면 주소라는 사실을 알아채긴 어렵다. 

`addr` 접두어를 추가해 `addrState`로 정의하면 맥락이 더 분명해진다. `Address`라는 클래스를 생성하면 더 좋다.

다음 메서드를 보자. 함수 이름은 맥락 일부만 제공하며, 알고리즘이 나머지 맥락을 제공한다. 함수를 끝까지 읽어봐야 `number`, `verb`, `pluralModifier` 변수가 `통계 추측(guess statistics)` 메시지에
사용되는 사실을 알 수 있다. 독자가 맥락을 유추해야만 한다. 그냥 메서드만 훑어서는 세 변수의 의미가 불분명하다.

```java
private void printGuessStatistics(char candidate, int count) { 
    String number;
    String verb;
    String pluralModifier; 
    if (count == 0) {
        number = "no";
        verb = "are"; 
        pluralModifier = "s";
    } else if (count == 1) { 
        number = "1";
        verb = "is"; 
        pluralModifier = "";
    } else {
        number = Integer.toString(count); 
        verb = "are";
        pluralModifier = "s";
    }
    String guessMessage = String.format(
            "There %s %s %s%s", verb, number, candidate, pluralModifier
        );
    print(guessMessage); 
}
```

위 함수를 작은 조각으로 쪼개고자 `GuessStatisticsMessage` 클래스를 만든 후 세 변수를 클래스에 넣었다. 이제 세 변수의 맥락이 분명해진다. 즉, 세 변수는 확실하게 `GuessStatisticsMessage`에 속한다.

```java
public class GuessStatisticsMessage { 
    private String number;
    private String verb;
    private String pluralModifier;
    public String make(char candidate, int count) { 
        createPluralDependentMessageParts(count); 
        return String.format(
                "There %s %s %s%s", 
                verb, number, candidate, pluralModifier );
    }
    private void createPluralDependentMessageParts(int count) { 
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count); 
        }
    }
    private void thereAreManyLetters(int count) { 
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
    private void thereIsOneLetter() { 
        number = "1";
        verb = "is";
        pluralModifier = "";
    }
    private void thereAreNoLetters() { 
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } 
}
```

## 불필요한 맥락을 없애라

의미가 분명한 경우에 한해 일반적으로 짧은 이름이 긴 이름보다 좋다. 이름에 불필요한 맥락을 추가하지 않도록 주의한다. 

`accountAddress`와 `customerAddress`는 `Address` 클래스 인스턴스로는 좋은 이름이나 클래스 이름으로는 적합하지 못하다. `Address`는 클래스 이름으로 적합하다.
