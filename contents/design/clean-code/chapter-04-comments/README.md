# Comments

> 나쁜 코드에 주석을 달지 마라. 새로 짜라.

잘 달린 주석은 그 어떤 정보보다 유용하다. 경솔하고 근거 없는 주석은 코드를 이해하기 어렵게 만든다. 오래되고 조잡한 주석은 거짓과 잘못된 정보를 퍼뜨려 해악을 미친다.

우리는 코드로 의도를 표현하지 못해, 그러니까 실패를 만회하기 위해 주석을 사용한다. 즉, 주석은 언제나 실패를 의미한다. 그러므로 주석이 필요한 상황에 처하면 상황을 역전해 코드로 의도를
표현할 수 있는 방법이 없는지 확인하자.

프로그래머가 주석을 유지보수 하지 않으므로, 주석은 오래될수록 코드에서 멀어진다. 오래될수록 그릇될 가능성도 커진다.

부정확한 주석은 아예 없는 주석보다 훨씬 더 나쁘다. 부정확한 주석은 독자를 현혹하고 오도한다. 

진실은 코드 한곳에만 존재한다. 코드만이 자기가 하는 일을 진싱되게 말한다. 그러므로 우리는 주석을 가능한 줄이도록 꾸준히 노력해야 한다.

## 주석은 나쁜 코드를 보완하지 못한다

코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다. 표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 주석이 많이 달린 코드보다 훨씬 좋다. 따라서 자신이 저지른
난장판을 주석으로 설명하기보다 그 난장판을 치우는 데 시간을 보내야 한다.

## 코드로 의도를 표현하라!

다음 두 예제를 보자.

```java
// Check to see if the employee is eligible for full benefits 
if ((employee.flags & HOURLY_FLAG) && 
        (employee.age > 65))
```

```java
if (employee.isEligibleForFullBenefits())
```

몇 초만 더 생각하면 코드로 대다수 의도를 표현할 수 있다. 많은 경우 주석으로 달려는 설명을 함수로 만들어 표현해도 충분하다.

## 좋은 주석

### 법적인 주석

때로는 회사가 정립한 구현 표준에 맞춰 법적인 이유로 특정 주석을 넣으라고 명시한다. 각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고도 타당하다.

### 정보를 제공하는 주석

때로는 기본적인 정보를 주석으로 제공하면 편리하다. 예를 들어, 다음 주석은 추상 메서드가 반환할 값을 설명한다. 

```java
// Returns an instance of the Responder being tested.
protected abstract Responder responderInstance();
```

때때로 위와 같은 주석이 유용할지라도, 가능하면 함수 이름에 정보를 담는 편이 더 좋다. 예를 들어, 위 코드는 함수 이름을 `responderBeingTested`로 변경하면 주석이 필요없다.

### 의도를 설명하는 주석

때때로 주석은 구현을 이해하게 도와주는 선을 넘어 결정에 깔린 의도까지 설명한다. 다음은 주석으로 흥미로운 결정을 기록한 예제다.

```java
public int compareTo(Object o) {
    if (o instanceof WikiPagePath) {
        WikiPagePath p = (WikiPagePath) o;
        String compressedName = StringUtil.join(names, ""); 
        String compressedArgumentName = StringUtil.join(p.names, "");
        return compressedName.compareTo(compressedArgumentName);
    }
    return 1; // we are greater because we are the right type.
}
```

다음은 다른 예제이다. 저자의 의도는 분명히 드러난다.

```java
public void testConcurrentAddWidgets() throws Exception { 
    WidgetBuilder widgetBuilder = 
        new WidgetBuilder(new Class[]{BoldWidget.class});
    String text = "'''bold text'''";
    ParentWidget parent = new BoldWidget(new MockWidgetRoot(), "'''bold text'''"); 
    AtomicBoolean failFlag = new AtomicBoolean();
    failFlag.set(false);
    
    //This is our best attempt to get a race condition
    //by creating large number of threads.
    for (int i = 0; i < 25000; i++) {
        WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
        Thread thread = new Thread(widgetBuilderThread);
        thread.start(); 
    }
    assertEquals(false, failFlag.get()); 
}
```

### 의미를 명료하게 밝히는 주석

때때로 모호한 인수나 반환값은 그 의미를 읽기 좋게 표현하면 이해하기 쉬워진다. 일반적으로 인수나 반환값 자체를 명확하게 만들면 더 좋지만, 인수나 반환값이 표준 라이브러리나 변경하지 못하는 코드에
속한다면 의미를 명료하게 밝히는 주석이 유용하다.

```java

public void testCompareTo() throws Exception {
    WikiPagePath a = PathParser.parse("PageA"); 
    WikiPagePath ab = PathParser.parse("PageA.PageB");
    WikiPagePath b = PathParser.parse("PageB"); 
    WikiPagePath aa = PathParser.parse("PageA.PageA");
    WikiPagePath bb = PathParser.parse("PageB.PageB"); 
    WikiPagePath ba = PathParser.parse("PageB.PageA");
    
    assertTrue(a.compareTo(a) == 0);        // a == a 
    assertTrue(a.compareTo(b) != 0);        // a != b
    assertTrue(ab.compareTo(ab) == 0);      // ab == ab
    assertTrue(a.compareTo(b) == -1);       // a < b 
    assertTrue(aa.compareTo(ab) == -1);     // aa < ab
    assertTrue(ba.compareTo(bb) == -1);     // ba < bb
    assertTrue(b.compareTo(a) == 1);        // b > a
    assertTrue(ab.compareTo(aa) == 1);      // ab > aa
    assertTrue(bb.compareTo(ba) == 1);      // bb > ba
}
```

물론 그릇된 주석을 달아놓을 위험이 있다. 위 예제를 보면, 주석이 올바른지 검증하기 쉽지 않다. 그러므로 위와 같은 주석을 달 때는 더 나은 방법이 없는지 고민하고 정확히 달도록 주의한다.

### 결과를 경고하는 주석

때로 다른 프로그래머에게 결과를 경고할 목적으로 주석을 사용한다. 예를 들어, 다음은 특정 테스트 케이스를 꺼야하는 이유를 설명하는 주석이다.

```java
// Don't run unless you have some time to kill. (여유롭지 않다면 실행하지 마라) 
public void _testWithReallyBigFile() {
    writeLinesToFile(10000000);
    
    response.setBody(testFile);
    response.readyToSend(this);
    String responseString = output.toString(); 
    assertSubString("Content-Length: 1000000000", responseString);
    assertTrue(bytesSent > 1000000000);
}
```

요즘은 `@Ignore` 속성을 이용해 테스트 케이스를 꺼버린다. 구체적인 설명은 `@Ignore` 속성에 문자열로 넣어준다. 하지만 JUnit4가 나오기 전 메서드 이름 앞에 `_` 기호를 붙이는 방법이 일반적인 관례였다.

다음 주석이 적절한 예제이다.

```java
public static SimpleDateFormat makeStandardHttpDateFormat() {
    //SimpleDateFormat is not thread safe,
    //so we need to create each instance independently.
    SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z"); 
    df.setTimeZone(TimeZone.getTimeZone("GMT"));
    return df;
}
```

프로그램 효율을 높이기 위해 정적 초기화 함수를 사용하려던 프로그래머가 주석 때문에 실수를 면한다.

### TODO 주석

때로는 앞으로 할 일을 `//TODO` 주석으로 남겨두면 편하다. 

```java

//TODO-MdM these are not needed
// We expect this to go away when we do the checkout model protected VersionInfo makeVersion() throws Exception
{
    return null; 
}
```

`TODO` 주석은 프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술한다. 

### 중요성을 강조하는 주석

자칫 대수롭지 않다고 여겨질 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.

```java
String listItemContent = match.group(3).trim();
// the trim is real important. It removes the starting 
// spaces that could cause the item to be recognized
// as another list.
new ListItemWidget(this, listItemContent, this.level + 1); 
return buildList(text.substring(match.end()));
```

### 공개 API에서 Javadocs

설명이 잘 된 공개 API는 유용하다. 표준 자바 라이브러리에서 사용한 Javadocs가 좋은 예다.

공개 API를 구현한다면 반드시 훌륭한 Javadocs를 작성한다. 주의할 점은 여느 주석과 마찬가지로 Javadocs 역시 독자를 오도하거나, 그릇된 정보를 전달할 가능성이 존재한다.

## 나쁜 주석

대다수 주석이 이 범주에 속한다. 

### 주절거리는 주석

의무감 혹은 프로세스에서 하라고 하여 마지못해 주석을 달면 전적으로 시간낭비이다. 주석을 달기로 결정했다면 충분한 시간을 들여 최고의 주석을 달도록 노력한다.

다음은 FitNess의 코드로 주석이 제대로 달리지 않은 예제이다.

```java
public void loadProperties() {
    try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE; 
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
    }
    catch(IOException e) {
        // No properties files means all defaults are loaded
    } 
}
```

`catch` 블록에 있는 주석은 `IOException`이 발생하면 속성 파일이 없다는 뜻이고, 그러면 기본값을 메모리로 읽어 들인 상태라고 한다. 

1. `loadedProperties.load`를 호출하기 전에 읽어 들이나?
2. `loadedProperties.load`가 예외를 잡아 기본값을 읽어 들인 후 예외를 던지나?
3. `loadedProperties.load`가 파일을 읽어 들이기 전 모든 기본값부터 읽어 들이는가?
4. 저자가 `catch` 블록을 비워놓기 뭐해 몇 마디 덧붙였을 뿐인가?
5. 아니면 나중에 돌아와서 기본값을 읽어 들이는 코드를 구현하려 했는가?

답을 알아내려면 다른 코드를 뒤져보는 수밖에 없다. 이러한 주석은 바이트만 낭비할 뿐이다.

### 같은 이야기를 중복하는 주석

다음은 간단한 함수로, 헤더에 달린 주석이 같은 코드 내용을 그대로 중복한다.

```java
// Utility method that returns when this.closed is true. Throws an exception 
// if the timeout is reached.
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if(!closed) {
        wait(timeoutMillis);
        if(!closed)
            throw new Exception("MockResponseSender could not be closed"); 
    }
}
```

위 주석은 코드보다 더 많은 정보를 제공하지 못한다. 

다음은 Tomcat에서 가져온 코드다. 쓸모없고 중복된 Javadocs가 매우 많다. 아래 주석은 코드만 지저분하고 정신 없게 만든다.

```java
public abstract class ContainerBase implements Container, Lifecycle, Pipeline, MBeanRegistration, Serializable {
    
    /**
     * The processor delay for this component. */
    protected int backgroundProcessorDelay = -1;
    
    /**
     * The lifecycle event support for this component. */
    protected LifecycleSupport lifecycle = new LifecycleSupport(this);
    
    /**
     * The container event listeners for this Container. */
    protected ArrayList listeners = new ArrayList();
    
    /**
     * The Loader implementation with which this Container is * associated.
     */
    protected Loader loader = null;
    
    /**
     * The Logger implementation with which this Container is
     * associated.
     */
    protected Log logger = null;
    
    // ...
}
```

### 오해할 여지가 있는 주석

위 `waitForClose` 함수 예제를 보면 주석에 오해의 여지가 있다. `this.closed`가 `true`로 변하는 순간 메서드는 반환되지 않는다. `this.closed`가 `true`여야 메서드는 반환된다.
아니면 무조건 타임아웃을 기다렸다 `this.closed`가 그래도 `true`가 아니면 예외를 던진다.

주석에 담긴 살짝 잘못된 정보로 인해 `this.closed`가 `true`로 변하는 순간에 함수가 반환되리라는 생각으로 어느 프로그래머가 경솔하게 함수를 호출할지도 모른다.

### 의무적으로 다는 주석

모든 함수에 Javadocs를 달거나 모든 변수에 주석을 다는 규칙은 어리석기 그지없다. 이는 코드를 복잡하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다. 

다음은 모든 함수에 Javadocs를 넣으라는 규칙이 낳은 괴물이다. 아래의 주석은 아무 가치도 없으며, 잘못된 정보를 제공할 여지만 만든다.

```java
/** *
* @param title The title of the CD
* @param author The author of the CD
* @param tracks The number of tracks on the CD
* @param durationInMinutes The duration of the CD in minutes */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
    CD cd = new CD(); 
    cd.title = title; 
    cd.author = author; 
    cd.tracks = tracks; 
    cd.duration = duration; 
    cdList.add(cd);
}
```

### 이력을 기록하는 주석

때때로 사람은 모듈을 편집할 때마다 모듈 첫머리에 주석을 추가한다. 그리하여 모듈 첫머리 주석은 지금까지 모듈에 가한 변경을 모두 기록하는 일종의 로그가 된다.

```java
* Changes (from 11-Oct-2001)
* --------------------------
* 11-Oct-2001 : Re-organised the class and moved it to new package com.jrefinery.date (DG);
* 05-Nov-2001 : Added a getDescription() method, and eliminated NotableDate class (DG);
* 12-Nov-2001 : IBD requires setDescription() method, now that NotableDate class is gone (DG); Changed getPreviousDayOfWeek(), getFollowingDayOfWeek() and getNearestDayOfWeek() to correct bugs (DG);
* 05-Dec-2001 : Fixed bug in SpreadsheetDate class (DG);
* 29-May-2002 : Moved the month constants into a separate interface (MonthConstants) (DG);
```

예전에는 모듈 첫머리에 변경 이력을 기록하고 관리하는 관례가 바람직했지만, 현재는 소스 코드 관리 시스템이 존재하므로 이는 혼란만 가중할 뿐이다.

### 있으나 마나 한 주석

때떄로 있으나 마나 한 주석을 접한다. 즉, 너무 당연한 사실을 언급하며 새로운 정보를 제공하지 못하는 주석이다.

```java
/**
* Default constructor. */
protected AnnualDateRule() {
}
```

```java
/** The day of the month. */
private int dayOfMonth;
```

```java
/**
* Returns the day of the month. *
* @return the day of the month. */
public int getDayOfMonth() { 
    return dayOfMonth;
}
```

다음 코드에서 첫 번째 주석은 적절해 보인다. `catch` 블록을 무시해도 괜찮은 이유를 설명하는 주석이다. 하지만 두 번째 주석(짜증났기에 적은 주석)은 전혀 쓸모가 없다. 

```java
private void startSending() {
    try{
        doSending();
    }
    catch(SocketException e) {
        // normal. someone stopped the request.
    }
    catch(Exception e){
        try{
            response.add(ErrorResponder.makeExceptionString(e));
            response.closeAll();}
        catch(Exception e1) {
            //Give me a break!
        }
    }
}
```

있으나 마나 한 주석으로 분풀이를 하는 대신 프로그래머가 구조를 개선했더라면 짜증낼 필요가 없었을 것이다.

```java
private void startSending() {
    try {
        doSending();
    }
    catch(SocketException e) {
        // normal. someone stopped the request. 
    }
    catch(Exception e) {
        addExceptionAndCloseResponse(e); 
    }
}
    
private void addExceptionAndCloseResponse(Exception e) {
    try {
        response.add(ErrorResponder.makeExceptionString(e));
        response.closeAll();
    }
    catch(Exception e1) {
    }
}
```

### 무서운 잡음

때로는 Javadocs도 잡음이다. 다음 Javadocs는 단지 문서를 제공해야 한다는 잘못된 욕심으로 탄생한 잡음일 뿐이다. 다음 코드에서 붙여넣기 오류가 보인다. 독자는 여기서 주석을 
통해 어떤 이익도 얻을 수 없다.

```java
/** The name. */ 
private String name;

/** The version. */ 
private String version;
/** The licenceName. */
private String licenceName;

/** The version. */
private String info;
```

### 함수나 변수로 표현할 수 있다면 주석을 달지 말라

다음 코드를 보자.

```java
// does the module from the global list <mod> depend on the
// subsystem we are part of?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

이 코드에서 주석을 없애고 다시 표현하면 다음과 같다. 다음과 같이 주석이 필요하지 않도록 코드를 개선하는 편이 더 좋다.

```java
ArrayList moduleDependees = smodule.getDependSubsystems(); 
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

### 위치를 표시하는 주석

때떄로 프로그래머는 소스 파일에서 특정 위치를 표시하려 다음과 같이 주석을 사용한다. 

```java
// Actions //////////////////////////////////
```

극히 드물지만 위와 같은 배너 아래 특정 기능을 모아놓으면 유용하기도 하지만, 일반적으로 위와 같은 주석은 가독성만 낮추므로 제거해야 한다.

### 닫는 괄호에 다는 주석

때로는 프로그래머들이 닫는 괄호에 특수한 주석을 달아놓는다. 중첩이 심하고 장황한 함수라면 의미가 있을지 모르지만, 작고 캡슐화된 함수에는 잡음일 뿐이다. 그러므로 닫는 괄호에
주석을 다는 대신 함수를 줄이려 시도하자.

```java
public class wc {
    public static void main(String[] args) {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        String line;
        int lineCount = 0;
        int charCount = 0;
        int wordCount = 0; 
        try {
            while ((line = in.readLine()) != null) { 
                lineCount++;
                charCount += line.length();
                String words[] = line.split("\\W"); 
                wordCount += words.length;
            } //while
            System.out.println("wordCount = " + wordCount); 
            System.out.println("lineCount = " + lineCount); 
            System.out.println("charCount = " + charCount);
        } // try
        catch (IOException e) { 
            System.err.println("Error:" + e.getMessage());
        } //catch
    }
} //main
```

### 공로를 돌리거나 저자를 표시하는 주석

소스 관리 시스템은 누가 언제 무엇을 추가했는지 귀신처럼 기억한다. 저자 이름으로 코드를 오염시킬 필요가 없다. 

### 주석으로 처리한 코드

주석으로 처리한 코드만큼 밉살스러운 관행도 드물다. 다음과 같은 코드는 작성하지 마라.

```java
InputStreamResponse response = new InputStreamResponse();
response.setBody(formatter.getResultStream(), formatter.getByteCount());
// InputStream resultsStream = formatter.getResultStream();
// StreamReader reader = new StreamReader(resultsStream);
// response.setContent(reader.read(formatter.getByteCount()));
```

1960년대 즈음에는 주석으로 처리한 코드가 유용했지만, 이제는 소스 코드 관리 시스템을 사용하기 때문에 이와 같이 주석으로 처리한 코드는 필요가 없다.

### HTML 주석

소스코드에 HTML 주석은 혐오 그 자체다. HTML 주석은 IDE에서조차 읽기가 어렵다. (Javadocs와 같은) 도구로 주석을 뽑아 웹 페이지에 올릴 작정이라면 주석에 HTML 태그를 삽입해야
하는 책임은 프로그래머가 아니라 도구가 가져가야 한다.

```java

/**
 * Task to run fit tests.
 * This task runs fitnesse tests and publishes the results.
 * <p/>
 * <pre>
 * Usage:
 * &lt;taskdef name=&quot;execute-fitnesse-tests&quot;
 *           classname=&quot;fitnesse.ant.ExecuteFitnesseTestsTask&quot*
 *           classpathref=&quot;classpath&quot; /&gt; 
 * 
 * *OR
 * &lt;taskdef classpathref=&quot;classpath&quot;
 *               resource=&quot;tasks.properties&quot; /&gt;
 * <p/>
 * &lt;execute-fitnesse-tests
 *     suitepage=&quot;FitNesse.SuiteAcceptanceTests&quot;
 *     fitnesseport=&quot;8082&quot;
 *     resultsdir=&quot;${results.dir}&quot;
 *     resultshtmlpage=&quot;fit-results.html&quot;
 *     classpathref=&quot;classpath&quot; /&gt;
 * </pre>
 */
```

### 전역 정보

주석을 달아야 한다면 근처에 있는 코드만 기술하라. 코드 일부에 주석을 달면서 시스템의 전반적인 정보를 기술하지 마라.

다음 주석은 아래의 함수가 아니라 시스템 어딘가의 다른 함수를 설명한다. 즉, 포트 기본값을 설정하는 코드가 변해도 아래 주석이 변하리라는 보장이 없다.

```java
/**
 * Port on which fitnesse would run. Defaults to <b>8082</b>. *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnessePort = fitnessePort; 
}
```

### 너무 많은 정보

주석에 관련 없는 정보를 장황하게 늘어놓지 마라. 다음은 base64를 인코딩/디코딩하는 함수를 테스트하는 모듈에서 가져온 주석이다. RFC 번호를 제외하면 독자에게 불필요한 정보들 뿐이다.

```java
/*
RFC 2045 - Multipurpose Internet Mail Extensions (MIME)
Part One: Format of Internet Message Bodies
section 6.8. Base64 Content-Transfer-Encoding
The encoding process represents 24-bit groups of input bits as output 
strings of 4 encoded characters. Proceeding from left to right, a 
24-bit input group is formed by concatenating 3 8-bit input groups. 
These 24 bits are then treated as 4 concatenated 6-bit groups, each 
of which is translated into a single digit in the base64 alphabet. 
When encoding a bit stream via the base64 encoding, the bit stream 
must be presumed to be ordered with the most-significant-bit first. 
That is, the first bit in the stream will be the high-order bit in 
the first 8-bit byte, and the eighth bit will be the low-order bit in 
the first 8-bit byte, and so on.
*/
```

### 모호한 관계

주석과 주석이 설명하는 코드는 둘 사이 관계가 명확해야 한다. 

다음은 아파치 `commmons`에서 가져온 주석이다.

```java

/*
 * start with an array that is big enough to hold all the pixels 
 * (plus filter bytes), and an extra 200 bytes for header info 
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
```

여기서 필터 바이트는 무엇인가? `+1`과 관련이 있을까? 아니면 `*3`과 관련이 있을까? 아니면 둘 다? 주석을 다는 목적은 코드만으로 설명이 부족해서이다. 위 주석은 주석 자체가 다시 설명을 요구한다.

### 함수 헤더

짧은 함수는 긴 설명이 필요 없다. 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 좋다.

### 비공개 코드에서 Javadocs

공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다. Javadocs 주석이 요구하는 형식으로 인해 코드만 보기 싫고 산만해질 뿐이다.

### 예제

다음은 필자가 XP 몰입(Immersion) 강의에서 짰던 모듈이다. 바람직하지 못한 주석을 직접 찾아보자.

```java
/**
* This class Generates prime numbers up to a user specified
* maximum. The algorithm used is the Sieve of Eratosthenes.
* <p>
* Eratosthenes of Cyrene, b. c. 276 BC, Cyrene, Libya --
* d. c. 194, Alexandria. The first man to calculate the
* circumference of the Earth. Also known for working on
* calendars with leap years and ran the library at Alexandria.
* <p>
* The algorithm is quite simple. Given an array of integers
* starting at 2. Cross out all multiples of 2. Find the next
* uncrossed integer, and cross out all of its multiples.
* Repeat untilyou have passed the square root of the maximum
* value. *
* @author Alphonse
* @version 13 Feb 2002 atp */
import java.util.*;

public class GeneratePrimes {
    /**
     * @param maxValue is the generation limit. 
     */
    public static int[] generatePrimes(int maxValue) {
        if (maxValue >= 2) // the only valid case
        {
            // declarations
            int s = maxValue + 1; // size of array 
            boolean[] f = new boolean[s];
            int i;

            // initialize array to true. 
            for (i = 0; i < s; i++)
                f[i] = true;
            // get rid of known non-primes
            f[0] = f[1] = false;
            // sieve
            int j;
            for (i = 2; i < Math.sqrt(s) + 1; i++) {
                if (f[i]) { // if i is uncrossed, cross its multiples. 
                    for (j = 2 * i; j < s; j += i)
                        f[j] = false; // multiple is not prime
                }
            }
            // how many primes are there? 
            int count = 0;
            for (i = 0; i < s; i++) {
                if (f[i])
                    count++; // bump count.
            }
            int[] primes = new int[count];
            // move the primes into the result 
            for (i = 0, j = 0; i < s; i++) {
                if (f[i]) primes[j++] = i;
            }

            return primes; // return the primes
        } else // maxValue < 2
            return new int[0]; // return null array if bad input.
    }
}
```

다음은 이를 리팩터링한 결과이다. 주석 양이 상당히 줄었다는 사실에 주목하자. 전체 모듈에서 주석은 두 개뿐이다. 두 주석 모두 뭔가를 설명한다.

```java
/**
 * This class Generates prime numbers up to a user specified
 * maximum. The algorithm used is the Sieve of Eratosthenes. 
 * Given an array of integers starting at 2:
 * Find the first uncrossed integer, and cross out all its
 * multiples. Repeat until there are no more multiples 
 * in the array.
 */

public class PrimeGenerator {
    
    private static boolean[] crossedOut; 
    private static int[] result;
    
    public static int[] generatePrimes(int maxValue) {
        if (maxValue < 2) return new int[0];
        else {
            uncrossIntegersUpTo(maxValue); 
            crossOutMultiples(); 
            putUncrossedIntegersIntoResult(); 
            return result;
        }
    } 

    private static void uncrossIntegersUpTo(int maxValue) {
        crossedOut = new boolean[maxValue + 1];
        for (int i = 2; i < crossedOut.length; i++) 
            crossedOut[i] = false;
    }
    
    private static void crossOutMultiples() {
        int limit = determineIterationLimit(); 
        for (int i = 2; i <= limit; i++) 
            if (notCrossed(i)) crossOutMultiplesOf(i);
    }
    
    private static int determineIterationLimit() {
        // Every multiple in the array has a prime factor that 
        // is less than or equal to the root of the array size, 
        // so we don't have to cross out multiples of numbers 
        // larger than that root.
        double iterationLimit = Math.sqrt(crossedOut.length);
        return (int) iterationLimit; 
    }
    
    private static void crossOutMultiplesOf(int i) {
        for (int multiple = 2*i; multiple < crossedOut.length; multiple += i)
            crossedOut[multiple] = true; 
    }

    private static boolean notCrossed(int i) {
        return crossedOut[i] == false; 
    }
    
    private static void putUncrossedIntegersIntoResult() {
        result = new int[numberOfUncrossedIntegers()]; 
        for (int j = 0, i = 2; i < crossedOut.length; i++) 
            if (notCrossed(i)) result[j++] = i;
    }
    
    private static int numberOfUncrossedIntegers() {
        int count = 0;
        for (int i = 2; i < crossedOut.length; i++) 
            if (notCrossed(i)) count++;
        return count; 
    }
}
```

첫 번째 주석이 `generatePrimes` 함수와 흡사하여 중복이라 주장할 수 있다. 그래도 주석이 있어 알고리즘을 이해하기 쉬워진다고 생각하여 남겨 두는 편을 택했다.

두 번째 주석은 확실히 필요하다. 루프 한계값으로 제곱근을 사용한 이유를 설명한다. 