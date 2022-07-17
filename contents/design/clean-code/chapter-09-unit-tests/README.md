# Unit Tests

우리 분야는 지금까지 눈부신 성장을 이뤘지만 갈 길은 여전히 멀다. 애자일과 TDD 덕택에 단위 테스트를 자동화하는 프로그래머들이 많아졌으며 점점 더 늘어나는 추세다. 하지만 우리 분야에 테스트를 투가하려고 급하게
서두르는 와중에 많은 프로그래머들이 제대로 된 테스트 케이스 작성해야 한다는 좀 더 미묘한 (중요한) 사실을 놓쳐버렸다.

## TDD 법칙 세 가지

TDD는 실제 코드를 짜기 전 단위 테스트부터 짜라고 요구한다. 이 규칙은 빙산의 일각에 불과하다. 

* 첫째 법칙: 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
* 둘째 법칙: 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
* 셋째 법칙: 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

위 세 가지 규칙을 따르면 개발과 테스트가 대략 30초 주기로 묶인다. 테스트 코드와 실제 코드가 함께 나올뿐더러 테스트 코드가 실제 코드보다 불과 몇 초 전에 나온다.

이렇게 일하면 매일 수십 개, 매달 수백 개, 매년 수천 개에 달하는 테스트 케이스가 나온다. 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 나온다. 하지만 실제 코드와 맞먹을 정도로
방대한 테스트 코드는 심각한 관리 문제을 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기

테스트 코드가 복잡할수록 실제 코드를 짜는 시간보다 테스트 케이스를 추가하는 시간이 더 걸리기 십상이다. 실제 코드를 변경해 기존 테스트 케이스가 실패하기 시작하면, 지저분한 코드로 인해, 
실패하는 테스트 케이스를 점점 더 통과시키기 어려워진다. 그래서 테스트 코드는 계속해서 늘어나는 부담이 된다. 

따라서 테스트 코드는 실제 코드 못지 않게 중요하므로 깨끗하게 짜야 한다.

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다

테스트 코드를 깨끗하게 유지하지 않으면 결국은 잃어버린다. 그리고 테스트 케이스가 없으면 실제 코드를 유연하게 만드는 버팀목도 사라진다. 즉, 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이
바로 **단위 테스트**다. 테스트 케이스가 있다면 변경이 두렵지 않다. 테스트 케이스가 없다면 모든 변경이 잠정적인 버그다. 아키텍처가 아무리 유연하고, 설계를 아무리 잘 나눴더라도, 테스트 케이스가 없으면
개발자는 버그가 숨어들까 두렵기 때문에 변경을 주저한다.

테스트 코드가 지저분하면 코드를 변경하는 능력이 떨어지며 코드 구조를 개선하는 능력도 떨어진다. 테스트 코드가 지저분할수록 실제 코드도 지저분해진다. 결국 테스트 코드를 잃어버리고 실제 코드도 망가진다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만들려면 가독성이 필요하다. 테스트 코드는 최소의 표현으로 많은 것을 나타내야 한다.

다음 테스트 케이스 3개는 이해하기 어렵기에 개선할 여지가 충분하다. 

```java
public void testGetPageHieratchyAsXml() throws Exception {
    crawler.addPage(root, PathParser.parse("PageOne")); 
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne")); 
    crawler.addPage(root, PathParser.parse("PageTwo"));
    
    request.setResource("root"); request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder(); 
    SimpleResponse response = 
        (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType()); 
    assertSubString("<name>PageOne</name>", xml); 
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception
{
    WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne")); 
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne")); 
    crawler.addPage(root, PathParser.parse("PageTwo"));
    
    PageData data = pageOne.getData();
    WikiPageProperties properties = data.getProperties();
    WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
    symLinks.set("SymPage", "PageTwo");
    pageOne.commit(data);
    request.setResource("root"); 
    request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder(); 
    SimpleResponse response = 
        (SimpleResponse) responder.makeResponse(
                new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType()); 
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml); 
    assertSubString("<name>ChildOne</name>", xml);
    assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
    crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");
    
    request.setResource("TestPageOne"); 
    request.addInput("type", "data");
    Responder responder = new SerializedPageResponder(); 
    SimpleResponse response =
        (SimpleResponse) responder.makeResponse( new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType()); 
    assertSubString("test page", xml);
    assertSubString("<Test", xml);
}
```

`PathParser`는 문자열을 `pagePath` 인스턴스로 변환한다. `pagePath`는 crawler가 사용하는 객체다. 이 코드는 테스트와 무관하며 테스트 코드의 의도만 흐린다. `responder` 객체를 
생성하는 코드와 `response`를 수집해 변환하는 코드 역시 노이즈에 불과하다. 게다가 `resource`와 인수에서 요청 URL을 만드는 어설픈 코드도 보인다.

다음은 위 코드를 리팩터링한 결과이다.

```java
public void testGetPageHierarchyAsXml() throws Exception { 
    makePages("PageOne", "PageOne.ChildOne", "PageTwo");
    submitRequest("root", "type:pages");
    
    assertResponseIsXML(); 
    assertResponseContains(
            "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
        );
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception { 
    WikiPage page = makePage("PageOne");
    makePages("PageOne.ChildOne", "PageTwo");
    
    addLinkTo(page, "PageTwo", "SymPage");
    
    submitRequest("root", "type:pages");
    
    assertResponseIsXML(); 
    assertResponseContains(
            "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>" 
        );
    assertResponseDoesNotContain("SymPage"); 
}

public void testGetDataAsXml() throws Exception { 
    makePageWithContent("TestPageOne", "test page");     
    submitRequest("TestPageOne", "type:data");     
    
    assertResponseIsXML();
    assertResponseContains("test page", "<Test"); }
```

BUILD-OPERATE-CHECK 패턴이 위와 같은 테스트 구조에 적합하다. 각 테스트는 명확히 세 부분으로 나눠진다. 

1. 테스트 자료를 만든다.
2. 테스트 자료를 operate 한다.
3. operation 결과가 올바른지 확인한다.

노이즈 코드를 거의 다 없앴다는 사실에 주목한다. 테스트 코드는 진짜 필요한 자료 유형과 함수만 사용해야 한다. 그러므로 코드를 읽는 사람은 코드가 수행하는 기능을 재빨리 이해할 수 있다.

### 도메인에 특화된 언어

위 코드는 DSL로 테스트 코드를 구현하는 기법을 보여준다. 흔히 쓰는 시스템 조작 API를 사용하는 대신 API 위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용하므로 테스트 코드를 
짜기도 읽기도 쉬워진다. 이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다. 즉, 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 **언어**다.

이런 API는 처음부터 설계된 API가 아니다. 노이즈로 범벅된 코드를 리팩터링하다가 진화된 API다. 숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리팩터링해야 한다.

### 이중 표준

테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다. 단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다.

온도가 급격하게 떨어지면 경보, 온풍기, 송풍기가 모두 가동되는지 확인하는 코드인 다음 코드를 보자.

```java
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
    hw.setTemp(WAY_TOO_COLD);
    controller.tic(); 
    assertTrue(hw.heaterState()); 
    assertTrue(hw.blowerState()); 
    assertFalse(hw.coolerState()); 
    assertFalse(hw.hiTempAlarm()); 
    assertTrue(hw.loTempAlarm());
}
```

위 코드는 세세한 사항이 많으며, 코드에서 점검하는 상태 이름과 상태 값을 확인해야하므로 읽기가 어렵다. 

다음과 같이 변환해 코드 가독성을 크게 높였다.

```java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    wayTooCold();
    assertEquals("HBchL", hw.getState()); 
}
```

`tic` 함수는 `wayTooCold` 함수를 만들어 숨겼다. `assertEquals`에 들어있는 이상한 문자열에 주목한다. 대문자는 `on`이고, 소문자는 `off`를 뜻한다. 

이는 [그릇된 정보를 피하라](contents/design/clean-code/chapter-02-meaningful-names/README.md) 규칙 위반에 가깝지만 여기서는 적절해 보인다. 의미만 안다면 눈길이
문자열을 따라 움직이며 결과를 빨리 판단한다. 다음 테스트 코드를 보면 이해하기 쉽다는 사실이 분명해진다.

```java
@Test
public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
    tooHot();
    assertEquals("hBChl", hw.getState()); 
}

@Test
public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
    tooCold();
    assertEquals("HBchl", hw.getState()); 
}

@Test
public void turnOnHiTempAlarmAtThreshold() throws Exception {
    wayTooHot();
    assertEquals("hBCHl", hw.getState()); 
}

@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    wayTooCold();
    assertEquals("HBchL", hw.getState()); 
}
```

다음은 `getState` 함수이다. 효율을 높이려면 `StringBuffer`가 더 적합하다.

```java
public String getState() {
    String state = "";
    state += heater ? "H" : "h"; 
    state += blower ? "B" : "b"; 
    state += cooler ? "C" : "c";
    state += hiTempAlarm ? "H" : "h"; 
    state += loTempAlarm ? "L" : "l"; 
    return state;
}
```

필자는 실제 코드에서도 크게 무리가 아니라면 `StringBuffer`를 피한다. `StringBuffer`를 안 써서 치르는 대가가 미미하다. 하지만 애플리케이션은 컴퓨터 자원과 메모리가 제한적일 가능성이 높다.
하지만 테스트 환경은 자원이 제한적일 가능성이 낮다. 

이것이 이중 표본의 본질이다. 실제 환경에서는 절대 안 되지만 테스트 환경에서는 전혀 문제없는 방식이 있다. 대개 메모리나 CPU 효율과 관련이 있는 경우다. 코드의 깨끗함과는 철저히 무관하다.

## 테스트 당 assert 하나

Junit으로 테스트 코드를 짤 때는 함수마다 `assert` 문을 단 하나만 사용해야 한다고 주장하는 사람(Dave Astel)이 있다. 가혹한 규칙일 수 있지만 `assert` 문이 하나인 함수는 결론이
하나라서 코드를 이해하기 쉽고 빠르다.

**깨끗한 테스트 코드** 단락에서 리팩터링한 코드를 보면 "출력이 XML이다"라는 `assert` 문과 "특정 문자열을 포함한다"는 `assert` 문을 하나로 병합하는 방식이 불합리해 보인다.
다음과 같이 테스트를 두 개로 쪼개 각자가 `assert`를 수행하면 된다. 

```java
public void testGetPageHierarchyAsXml() throws Exception { 
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
    whenRequestIsIssued("root", "type:pages");
    thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
    whenRequestIsIssued("root", "type:pages");
    thenResponseShouldContain(
            "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
        ); 
}
```

위에서 함수 이름을 바꿔 `given-when-then`이라는 관례를 사용했다. 그러면 테스트 코드를 읽기 쉬워진다. 불행하게도, 위와 같이 테스트를 분리하면 중복되는 코드가 많아진다.

TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있다. `given/when` 부분을 부모 클래스에 두고 `then` 부분을 자식 클래스에 두면 된다. 아니면 완전히 독자적인 테스트 클래스를
만들어 `@Before` 함수에 `given/when` 부분을 넣고 `@Test` 함수에 `then` 부분을 넣어도 된다. 하지만 모두 배보다 배꼽이 더 크다. 이것저것 감안해 보면 결국 처음 리팩토링한 것 처럼
`assert` 문을 여럿 사용하는 편이 좋다고 생각한다.

`단일 assert 문` 규칙은 훌륭한 지침이다. 때로는 주저 없이 함수 하나에 여러 `assert` 문을 넣기도 한다. 단지 `assert` 문 개수는 최대한 줄여야 좋다고 생각한다.

### 테스트 당 개념 하나

"테스트 함수마다 한 개념만 테스트하라"는 규칙이 더 낫다.

다음 코드는 독자적인 개념 세 개를 테스트하는 코드이다. 세 개념을 한 함수에 넣으면 독자가 각 절이 존재하는 이유와 테스트하는 개념을 모두 이해해야 한다.

```java
/**
 * Miscellaneous tests for the addMonths() method.
 * */
public void testAddMonths() {
    SerialDate d1 = SerialDate.createInstance(31, 5, 2004);
    SerialDate d2 = SerialDate.addMonths(1, d1);
    assertEquals(30, d2.getDayOfMonth()); 
    assertEquals(6, d2.getMonth()); 
    assertEquals(2004, d2.getYYYY());
    
    SerialDate d3 = SerialDate.addMonths(2, d1); 
    assertEquals(31, d3.getDayOfMonth()); 
    assertEquals(7, d3.getMonth());
    assertEquals(2004, d3.getYYYY());
    
    SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1)); 
    assertEquals(30, d4.getDayOfMonth());
    assertEquals(7, d4.getMonth());
    assertEquals(2004, d4.getYYYY());
}
```

셋으로 분리한 테스트 함수는 각 다음 기능을 수행한다.
* (5월처럼) 31일로 끝나는 달의 마지막 날짜가 주어지는 경우
  1. (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안 된다.
  2. 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
* (6월처럼) 30일로 끝나는 달의 마지막 날짜가 주어지는 경우
  1. 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되면 안 된다.

이렇게 표현하면 장황한 테스트 코드 속에 감춰진 일반적인 규칙이 보인다. 

따라서 가장 좋은 규칙은 다음과 같다.

* 개념 당 `assert` 문 수를 최소로 줄여라
* 테스트 함수 하나는 개념 하나만 테스트하라

## F.I.R.S.T

깨끗한 코드는 다음 다섯 가지 규칙을 따르는데, 각 규칙에서 첫 글자를 따오면 `FIRST`가 된다.

* Fast: 테스트는 빨리 돌아야 한다. 테스트가 느리면 자주 돌릴 엄두를 못 낸다. 자주 돌리지 않으면 문제를 찾아내 고치지 못한다. 결국 코드 품질이 망가진다.
* Independent: 각 테스트는 서로 의존하면 안 된다. 각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 괜찮아야 한다.
* Repeatable: 테스트는 어떤 환경에서도 반복 가능해야 한다. 실제 환경, QA 환경 등에서도 실행할 수 있어야 한다. 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다. 게다가 환경이 지원되지 않기에 테스트를 수행하지 못하는 상황에 직면한다.
* Self-Validating: 테스트는 `bool` 값으로 결과를 내야 한다. 테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며 지루한 수작업 평가가 필요하게 된다.
* Timely: 테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다. 실제 코드를 구현한 다음 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다. 어떤 실제 코드는 테스트하기 너무 어렵다고 판명날지 모른다. 테스트가 불가능하도록 실제 코드를 설계할지도 모른다.

## 결론

깨끗한 테스트 코드는 실제 코드만큼이나 중요하다. 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다. 그러므로 테스트 코드는 지속적으로 깨끗하게 관리하자. 표현력을 높이고 간결하게 정리하자.
테스트 API를 구현해 DSL을 만들자. 그러면 그만큼 테스트 코드를 짜기 쉬워진다. 

