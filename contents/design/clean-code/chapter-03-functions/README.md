# Functions

FitNesse의 [HtmlUtil.java](https://github.com/unclebob/fitnesse/blob/57da8b53ca0a78c85b47473abea2ec01dd439b6d/src/fitnesse/html/HtmlUtil.java)
함수를 보자.

```java
public static String testableHtml(PageData pageData, boolean includeSuiteSetup) throws Exception
{
	WikiPage wikiPage = pageData.getWikiPage();
	StringBuffer buffer = new StringBuffer();
	if(pageData.hasAttribute("Test"))
	{
		if (includeSuiteSetup) 
		{
			WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_SETUP_NAME, wikiPage);
			if (suiteSetup != null) {
				WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
				String pagePathName = PathParser.render(pagePath);
				buffer.append("!include -setup .").append(pagePathName).append("\n");
			}
		}
		WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
		if (setup != null)
		{
			WikiPagePath setupPath = wikiPage.getPageCrawler().getFullPath(setup);
			String setupPathName = PathParser.render(setupPath);
			buffer.append("!include -setup .").append(setupPathName).append("\n");
		}
	}
	buffer.append(pageData.getContent());
	if(pageData.hasAttribute("Test"))
	{
        WikiPage teardown = PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
		if(teardown != null)
		{
			WikiPagePath tearDownPath = wikiPage.getPageCrawler().getFullPath(teardown);
			String tearDownPathName = PathParser.render(tearDownPath);
			buffer.append("\n").append("!include -teardown .").append(tearDownPathName).append("\n");
		}
        if (includeSuiteSetup)
        {
           WikiPage suiteTeardown = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage);
           if (suiteTeardown != null)
           {
              WikiPagePath pagePath = suiteTeardown.getPageCrawler().getFullPath(suiteTeardown);
              String pagePathName = PathParser.render(pagePath);
              buffer.append("!include -teardown .").append(pagePathName).append("\n");
           }
        }
    }
	pageData.setContent(buffer.toString());
	return pageData.getHtml();
}
```

위 코드는 추상화 수준이 너무 다양할 뿐더러, 코드도 너무 길어 이해하기 어렵다. 메서드 몇 개를 추출하고, 이름 몇 개를 변경하고, 구조를 변경해 다음과 같이 만들었다.

```java
public static String renderPageWithSetupsAndTeardowns(
        PageData pageData, boolean isSuite) throws Exception {
    boolean isTestPage = pageData.hasAttribute("Test"); 
    if (isTestPage) {
        WikiPage testPage = pageData.getWikiPage(); 
        StringBuffer newPageContent = new StringBuffer(); 
        includeSetupPages(testPage, newPageContent, isSuite);
        newPageContent.append(pageData.getContent()); 
        includeTeardownPages(testPage, newPageContent, isSuite); 
        pageData.setContent(newPageContent.toString());
    }
    return pageData.getHtml(); 
}
```

이제 함수가 설정(setup) 페이지와 해제(teardown) 페이지를 테스트 페이지에 넣은 후 해단 테스트 페이지를 HTML로 렌더링한다는 사실은 짐작할 수 있다.

## 작게 만들어라

함수를 만드는 첫째 규칙은 '작게!'다. 함수를 만드는 둘째 규칙은 '더 작게!'다. 일반적으로 함수는 리팩터링한 위 코드보다 짧아야 한다. 즉, 다음과 같이 변경할 수 있다.

```java
public static String renderPageWithSetupsAndTeardowns( 
        PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData))
        includeSetupAndTeardownPages(pageData, isSuite); 
    return pageData.getHtml();
}
```

### 블록과 들여쓰기

다시 말해, if/else/while 문에 들어가는 블록은 한 줄이여야 한다는 의미이다. 대게 거기서 함수를 호출한다. 그러면 바깥을 감싸는 함수(enclosing function)가 작아질 뿐만 아니라,
블록 안에서 호출하는 함수 이름을 적절히 지으면 코드를 이해하기도 쉬워진다.

즉, 중첩 구조가 생길만큼 함수가 커져서는 안 된다는 뜻이다. 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안 된다.

## 한 가지만 해라!

> 함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.

위에 소개된 코드는 세 가지를 한다고 주장할 수도 있다.

1. 페이지가 테스트 페이지인지 판단한다.
2. 그렇다면 설정 페이지와 해제 페이지를 넣는다.
3. 페이지를 HTML로 렌더링한다.

위에서 언급하는 세 단계는 지정된 함수 이름 아래에서 추상화 수준이 하나다. 

> TO `renderPageWithSetupsAndTeardowns`, 페이지가 테스트 페이지인지 확인한 후 테스트 페이지라면 설정 페이지와 해제 페이지를 넣는다. 테스트 페이지든 아니든 페이지를 HTML로 렌더링한다.

지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다. 

함수가 한 가지만 하는지 판단하는 방법이 더 있다. 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.

### 함수 내 섹션

함수에서 한 가지 작업만 하는 함수는 자연스럽게 섹션으로 나누기 어렵다.

## 함수 당 추상화 수준은 하나로!

함수가 확실히 한 가지 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다. `testableHtml`에선 이 규칙을 위반한다. `getHtml()`은 추상화 수준이 아주 높다. 
`String pagePathName = PathParser.render(pagePath);`는 추상화 수준이 중간이다. `append("\n")`와 같은 코드는 추상화 수준이 아주 낮다.

한 함수 내 추상화 수준을 섞으면 코드를 읽는 사람이 헷갈린다. 특정 표현이 근본 개념인지 아니면 세부사항인지 구분하기
어려운 탓이다. 하지만 문제는 근본 개념과 세부사항을 뒤섞기 시작하면, 깨어진 창문처럼 사람들이 함수에 세부사항을 점점 더 추가한다.

### 위에서 아래로 코드 읽기: **내려가기** 규칙

코드는 위에서 아래로 이야기처럼 읽혀야 좋다. 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다. 즉, 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 번에 한 단계씩
낮아진다. 이를 **내려가기** 규칙이라 부른다.

다르게 표현하면, TO 문단을 읽듯이 프로그램이 읽혀야 한다는 의미다. 이렇게 구현하면 추상화 수준을 일관되게 유지하기가 쉬워진다.

```
TO 설정 페이지와 해제 페이지를 포함하려면, 설정 페이지를 포함하고, 테스트 페이지 내용을 포함하고, 해제 페이지를 포함한다.
    TO 설정 페이지를 포함하려면, 슈트이면 슈트 설정 페이지를 포함한 후 일반 설정 페이지를 포함한다.
    TO 슈트 페이지를 포함하려면, 부모 계층에서 "SuiteSetUp" 페이지를 찾아 include 문과 페이지 경로를 추가한다.
    TO 부모 계층을 검색하려면, ...
```

## Switch 문

`switch` 문은 작게 만들기 어렵다. 또한 한 가지 작업만 하는 `switch` 문도 만들기 어렵다. 본질적으로 `switch` 문은 N가지를 처리한다. 하지만 각 `switch` 문을 저차원 클래스에 숨기고 절대로
반복하지 않는 방법은 있다. 다형성(polymorphism)을 이용한다. 

다음은 직원 유형에 따라 다른 값을 계산해 반환하는 함수이다.

```java
public Money calculatePay(Employee e) 
        throws InvalidEmployeeType {
    switch (e.type) { 
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e); 
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type); 
    }
}
```

위 함수에는 다음과 같은 문제가 있다.

1. 함수가 길다. 새 직원 타입을 추가하면 더 길어진다.
2. 한 가지 작업만 수행하지 않는다.
3. SRP를 위반한다.
4. OCP를 위반한다. 새 직원 타입을 추가할 때마다 코드를 변경하기 때문이다. 

가장 심각한 문제는 위 함수와 구조가 동일한 함수가 무한정 존재할 수 있다는 것이다. 예를 들어, `isPayday(Employee e, Date date)` 등. 

다음은 이를 해결한 코드이다. `switch`를 추상 팩토리(ABSTRACT FACTORY)에 숨긴다. 아무에게도 보여주지 않는다. 팩토리는 `switch` 문을 사용해 적절한 `Employee` 파생 클래스의 인스턴스를 생성한다. 
`calculatePay()`, `isPayday()`, `deliverPay()` 등과 같은 함수는 `Employee` 인터페이스를 거쳐 호출된다. 그러면 다형성으로 인해 실제 파생 클래스의 함수가 실행된다.

```java
public abstract class Employee {
    public abstract boolean isPayday(); 
    public abstract Money calculatePay(); 
    public abstract void deliverPay(Money pay);
}
// ...
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
// ...
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType { 
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r) ;
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmploye(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}
```

이렇게 상속 관계로 숨긴 후 절대로 다른 코드에 노출하지 않는다. 물론 불가피한 상황도 발생한다.

## 서술적인 이름을 사용하라!

> 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다. 

한 가지만 하는 작은 함수에 좋은 이름을 붙인다면 이런 원칙을 달성함에 있어 이미 절반은 성공했다. 함수가 작고 단순할수록 서술적인 이름을 고르기 쉬워진다.

이름이 길어도 괜찮다. 길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 길고 서술적인 이름이 길고 서술적인 주석보다 좋다. 그런 다음, 여러 단어를 사용해 함수 기능을 잘 표현하는 이름을 선택한다.

## 함수 인수

함수에서 이상적인 인수 개수는 0개다. 다음은 1개, 다음은 2개이며, 3개는 가능한 피하는 편이 좋다. 4개 이상은 특별한 이유가 필요하다. 특별한 이유가 있어도 사용하면 안 된다.

인수는 개념을 이해하기 어렵게 만든다. 테스트 관점에서도 인수가 3개를 넘어가면 인수마다 유효한 값으로 모든 조합을 구성해 테스트해야 하므로 부담스러워 진다. 

출력 인수는 입력 인수보다 이해하기 어렵다. 일반적으로 함수에 인수를 입력으로 넘기고 반환값으로 출력을 받는다는 개념에 익숙하다. 그래서 출력 인수는 독자가 코드를 재차 확인하게 만든다.

### 많이 쓰는 단항 형식

함수에 인수 1개를 넘기는 이유로 가장 흔한 경우는 두 가지이다.

1. 하나는 인수에 질문을 던지는 경우
   * ex) `boolean fileExists("MyFile")`
2. 인수를 뭔가로 변환해 결과를 반환하는 경우
   * ex) `InputStream fileOpen("MyFile")`

다소 드물게 사용하지만 유용한 단항 함수 형식이 이벤트다. 이벤트 함수는 입력 인수만 있고, 출력 인수는 없다. `passwordAttemptFailedNtimes(int attempts)`가 좋은 예다. 이벤트 함수는 조심해서 
사용해야 한다. 이벤트라는 사실이 코드에 명확히 드러나야 한다.

지금까지 설명한 경우가 아니라면 단항 함수는 가급적 피한다.

### 플래그 인수

함수로 불 값을 넣기는 관례는 대놓고 여러 가지를 처리한다고 공표하는 셈이다.

### 이항 함수

인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다. `writeField(name)`은 `writeField(outputStream, name)`보다 이해하기 쉽다. 후자는 첫 인수를 무시해야 한다는 사실을 깨닫는 
시간이 필요하다. 

`Point p = new Point(0, 0)`과 같이 이항 함수가 적절한 경우도 있다. 직교 좌표계 점은 일반적으로 인수 2개를 취한다. 이 두 요소에는 자연적인 순서도 있다.

이항 함수를 사용하려면 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항 함수로 바꾸도록 애써야 한다. 예를 들어, `writeField` 메서드를 `outputStream` 클래스 구성원으로 만들어
`outputStream.writeField(name)`으로 호출하는 것처럼.

### 삼항 함수

인수가 3개이 함수는 2개인 함수보다 훨씬 더 이해하기 어렵다. 따라서 삼항 함수를 만들 때는 신중히 고려해야 한다.  

예를 들어, `assertEquals(message, expected, actual)`이라는 함수를 보자. 첫 인수가 `expected`라고 예상하고 있었으며, 매번 함수를 볼 때마다 주춤햇다가 `message`를 무시해야 한다는
사실을 상기한다.

반면 `assertEquals(1.0, amount, .001)`은 주춤하게 되지만 그만한 가치가 충분한 삼항 함수이다. 부동소수점 비교가 상대적이라는 사실은 언제든 주지할 중요한 사항이다.

### 인수 객체

인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어 본다. 다음 두 함수를 보자.

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

위 예제에서 `x`와 `y`를 묶었듯이 변수를 묶어 넘기려면 이름을 붙여야 하므로 결국 개념을 표현하게 된다.

### 인수 목록

때로는 인수 개수가 가변적인 함수도 필요하다. `String.format` 메서드가 좋은 예다.

```java
String.format("%s worked %.2f hours.", name, hours);
```

위 예제처럼 가변 인수 전부 동등하게 취급하면 `List` 형 인수 하나로 취급할 수 있다. 따져보면 `String.format`은 사실상 이항 함수이다. 

가변 인수를 취하는 함수는 단항, 이항, 삼항 함수로 취급할 수 있다. 이를 넘어서 인수를 사용하는 경우에는 문제가 있다.

### 동사와 키워드

함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다. 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. 예를 들어, `write(name)`은 바로 이해할 수 있다.

마지막 예제는 함수 이름에 키워드를 추가하는 형식이다. 즉, 함수 이름에 인수 이름을 넣는다. `assertEquals`보다 `assertExpectedEqualsActual(expect, actual)`이 더 좋다. 그러면 인수
순서를 기억할 필요가 없어진다.

## 부수 효과(Side Effects)를 일으키지 마라

부수 효과는 함수에서 한 가지를 하겠다고 약속하고 남몰래 다른 짓을 하는 것을 의미한다. 예상치 못하게 클래스 변수를 수정하거나 때로는 함수로 넘어온 인수나 시스템 전역 변수를 수정한다. 
많은 경우 시간적인 결합(temporal coupling)이나 순서 종속성(order dependency)을 초래한다.

다음 함수를 보자. 두 인수가 올바르면 `true`를 반환하고 아니면 `false`를 반환한다. 하지만 함수는 부수 효과를 일으킨다.

```java
public class UserValidator {
   private Cryptographer cryptographer;
   public boolean checkPassword(String userName, String password) {
       User user = UserGateway.findByName(userName);
       if (user != User.NULL) {
           String codedPhrase = user.getPhraseEncodedByPassword(); 
           String phrase = cryptographer.decrypt(codedPhrase, password); 
           if ("Valid Password".equals(phrase)) {
               Session.initialize();
               return true; 
           }
       }
       return false; 
   }
}
```

여기서 함수가 일으키는 부수 효과는 `Session.initialize();` 호출이다. `checkPassword()` 함수는 이름 그대로 암호를 확인하지만 세션을 초기화한다는 사실이 드러나지 않는다. 그래서 함수
이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다. 

이런 부수 효과는 시간적인 결합을 초래한다. 즉, `checkPassword` 함수는 특정 상황에서만 호출이 가능하다. 즉, 세션을 초기화해도 괜찮은 경우에만 호출할 수 있다.

위 함수는 `checkPasswordAndInitializeSession`이라는 이름이 훨씬 좋다. 물론 함수가 한 가지만 한다는 규칙을 위반하지만.

### 출력 인수

일반적으로 인수를 함수 입력으로 해석한다. 다음 함수를 보자.

```java
appendFooter(s);
```

이 함수는 `s`를 바닥글로 첨부할까? 아니면 `s`에 바닥글을 첨부할까? 인수 `s`는 입력일까 출력일까? 함수 선언부를 보면 분명해진다.

```java
public void appendFooter(StringBuffer report)
```

인수 `s`가 출력 인수라는 사실은 함수 선언부를 보고 나서야 알 수 있다. 이러한 행위는 코드를 보다 주춤하는 행위와 동급이다. 

객체 지향 프로그래밍이 나오기 전에는 출력 인수가 불가피한 경우도 있었지만, 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다. 출력 인수로 사용하라고 설계한 변수가 `this`이기 때문이다.
즉, `appendFooter`는 다음과 같이 호출하는 방식이 좋다.

```java
report.appendFooter();
```

일반적으로 출력 인수는 피하고, 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.

## 명령과 조회를 분리하라!

함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다. 객체 상태를 변경하거나 객체 정보를 반환하거나 둘 중 하나이다. 둘 다 하면 혼란을 초래한다. 다음 함수를 보자.

```java
public boolean set(String attribute, String value);
```

이 함수는 이름이 `attribute`인 속성을 찾아 `value`로 설정한 후 성공하면 `true`를 반환하고 실패하면 `false`를 반환한다. 그래서 다음과 같이 괴상한 코드가 나온다.

```java
if (set("username", "unclebob")) // ...
```

위 함수를 구현한 개발자는 `set`을 동사로 의도했다. 하지만 `if` 문에 넣으면 형용사로 느껴진다. 그래서 `if` 문은 "username 속성이 unclebob으로 설정되어 있다면"으로 읽힌다.

해결책은 명령과 조회를 분리해 혼란을 애초에 뿌리뽑는 방법이다.

```java
if (attributeExists("username")) { 
    setAttribute("username", "unclebob");
    // ...
}
```

## 오류 코드보다 예외를 사용하라!

명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다. 자칫하면 `if` 문에서 명령을 표현식으로 사용하기 쉬운 탓이다.

```java
if (deletePage(page) == E_OK)
```

위 코드는 동사/형용사 혼란을 일으키지 않지만 여러 단계로 중첩되는 코드를 야기한다. 오류 코드를 반환하면 곧바로 처리해야 한다는 문제에 부딪힌다.

```java

if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK){ 
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed"); }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
```

반면 오류 코드 대신 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.

```java
try {
    deletePage(page); 
    registry.deleteReference(page.name); 
    configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
    logger.log(e.getMessage()); 
}
```

### Try/Catch 블록 뽑아내기

try/catch 블록은 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞는다. 그러므로 try/catch 블록을 별도 함수로 뽑아내는 편이 좋다.

```java
public void delete(Page page) { 
    try {
        deletePageAndAllReferences(page); 
    }
    catch (Exception e) { 
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception { 
    deletePage(page);
    registry.deleteReference(page.name); 
    configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
    logger.log(e.getMessage());
}
```

위에서 `delete` 함수는 모든 오류를 처리한다. 그래서 코드를 이해하기 쉽다. 실제 페이지를 제거하는 함수는 `deletePageAndAllReferences`이다. 이렇게 정상 동작과 오류 처리 동작을
분리하면 코드를 이해하고 수정하기 쉬워진다.

### 오류 처리도 한 가지 작업이다.

오류 처리도 한 가지 작업에 속한다. 그러므로 오류를 처리하는 함수는 오류만 처리해야 마땅하다. 

### Error.java 의존성 자석

오류 코드를 반환한다는 이야기는, 클래스든, 열거형 변수든, 어디선가 오류 코드를 정의한다는 뜻이다.

```java
public enum Error { 
    OK, 
   INVALID, 
   NO_SUCH, 
   LOCKED, 
   OUT_OF_RESOURCES,
   WAITING_FOR_EVENT;
}
```

위와 같은 클래스는 의존성 자석(magnet)이다. 다른 클래스에서 `Error`를 import해 사용해야 하므로, `Error`이 변경되면 `Error`를 포함하는 클래스 전부를 다시 컴파일하고 배치해야 한다. 
따라서 `Error` 변경이 어려워진다. 

오류 코드 대신 예외를 사용하면 새 예외는 `Exception` 클래스에서 파생되므로 재컴파일/재배치 없이 추가할 수 있다.

## 반복하지 마라!

처음에 소개된 코드를 보면 네 번이나 반복되는 알고리즘이 보인다. 알고리즘 하나가 `SetUp`, `SuiteSetUp`, `TearDown`, `SuiteTearDown`에서 반복된다. 
코드 길이가 늘어날 뿐 아니라 알고리즘이 변경되면 네 곳을 손봐야 한다. 게다가 어느 한 곳이라도 빠뜨리면 오류가 발생할 확률도 네 배나 높다.

결론의 수정된 코드는 `include` 방법으로 중복을 없앤다. 중복을 없앴더니 모듈 가독성이 크게 높아졌다는 사실을 깨닫을 수 있다.

객체 지향 프로그래밍은 코드를 부모 클래스로 몰아 중복을 없앤다. 구조적 프로그래밍, AOP(Aspect Oriented Programming), COP(Component Oriented Programming) 모두 어떤 면에서 중복 제거 전략이다.

## 구조적 프로그래밍

다음은 데이크스트라의 구조적 프로그래밍 원칙이다.

1. 모든 함수와 함수 내 모든 블록에 입구(entry)와 출구(exit)가 하나만 존재해야 한다. 즉, 함수는 `return` 문이 하나여야 한다는 말이다.
2. 루프 안에서 `break`나 `continue`를 사용해선 안 되며 `goto`는 절대 안 된다.

함수가 작다면 위 규칙은 별 이익을 제공하지 못한다. 함수가 아주 클 때만 상당한 이익을 제공한다. 따라서 함수를 작게 만든다면 간혹 `return`, `break`, `continue`를 여러 번 사용해도 괜찮다. 
때로는 오히려 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다. 반면 `goto` 문은 큰 함수에서만 의미가 있으므로, 작은 함수에서는 피해야 한다.

## 결론

함수는 동사며, 클래스는 명사다. 대가 프로그래머는 시스템을 **구현할** 프로그램이 아니라 **풀어갈** 이야기로 여긴다. 프로그래밍 언어라는 수단을 사용해 좀 더 풍부하고 표현력이 강한 언어를 만들어 이야기를 풀어간다.
시스템에서 발생하는 모든 동작을 설명하는 함수 계층이 바로 그 언어에 속한다. 각 동작은 DSL을 사용해 자신만의 이야기를 풀어간다. 

```java
package fitnesse.html;

import fitnesse.responders.run.SuiteResponder; 
import fitnesse.wiki.*;


public class SetupTeardownIncluder { 
    private PageData pageData;
    private boolean isSuite;
    private WikiPage testPage;
    private StringBuffer newPageContent;
    private PageCrawler pageCrawler;
    public static String render(PageData pageData) throws Exception { 
        return render(pageData, false);
    }

    public static String render(PageData pageData, boolean isSuite) throws Exception {
        return new SetupTeardownIncluder(pageData).render(isSuite);
    }
    
    private SetupTeardownIncluder(PageData pageData) { this.pageData = pageData;
        testPage = pageData.getWikiPage();
        pageCrawler = testPage.getPageCrawler();
        newPageContent = new StringBuffer();
    }
    
    private String render(boolean isSuite) throws Exception { 
        this.isSuite = isSuite;
        if (isTestPage()) 
            includeSetupAndTeardownPages(); return pageData.getHtml();
    }
    
    private boolean isTestPage() throws Exception { 
        return pageData.hasAttribute("Test");
    }

    private void includeSetupAndTeardownPages() throws Exception { 
        includeSetupPages();
        includePageContent();
        includeTeardownPages();
        updatePageContent(); 
    }
    
    private void includeSetupPages() throws Exception { 
        if (isSuite) 
            includeSuiteSetupPage(); includeSetupPage();
    }
    
    private void includeSuiteSetupPage() throws Exception { 
        include(SuiteResponder.SUITE_SETUP_NAME, "-setup");
    }
    
    private void includeSetupPage() throws Exception { 
        include("SetUp", "-setup");
    }

    private void includePageContent() throws Exception {
        newPageContent.append(pageData.getContent());
    }

    private void includeTeardownPages() throws Exception { 
        includeTeardownPage();
        if (isSuite) 
            includeSuiteTeardownPage(); 
    }
    
    private void includeTeardownPage() throws Exception { 
        include("TearDown", "-teardown");
    }

    private void includeSuiteTeardownPage() throws Exception { 
        include(SuiteResponder.SUITE_TEARDOWN_NAME, "-teardown");
    }
    
    private void updatePageContent() throws Exception { 
        pageData.setContent(newPageContent.toString());
    }
    
    private void include(String pageName, String arg) throws Exception { 
        WikiPage inheritedPage = findInheritedPage(pageName);
        if (inheritedPage != null) {
            String pagePathName = getPathNameForPage(inheritedPage);
            buildIncludeDirective(pagePathName, arg); 
        }
    }
    
    private WikiPage findInheritedPage(String pageName) throws Exception { 
        return PageCrawlerImpl.getInheritedPage(pageName, testPage);
    }
    
    private String getPathNameForPage(WikiPage page) throws Exception { 
        WikiPagePath pagePath = pageCrawler.getFullPath(page);
        return PathParser.render(pagePath);
    }
    
    private void buildIncludeDirective(String pagePathName, String arg) { 
        newPageContent
                .append("\n!include ")
                .append(arg) 
                .append(" .") 
                .append(pagePathName) 
                .append("\n");
    }
}
```

