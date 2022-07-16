# Formatting

프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야 한다. 코드 형식을 맞추기 위한 간단한 규칙을 정하고 그 규칙을 따라야 한다. 필요하다면 규칙을 자동으로 적용하는 도구를 활용한다.

## 형식을 맞추는 목적

코드 형식은 중요하다! 너무 중요하므로 무시해선 안 되며 융통성 없이 맹목적으로 따르면 안 된다. 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다.

## 적절한 행 길이를 유지하라

대부분 200줄 정도인 파일로도 커다란 시스템 구축이 가능하다. 일반적으로 큰 파일보다 작은 파일이 이해하기 쉽다.

### 신문 기사처럼 작성하라

이름은 간단하면서도 설명이 가능하게 짓는다. 이름만 보고도 올바른 모듈을 살펴보고 있는지 아닌지를 판단할 정도로 신경 써서 짓는다. 소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명한다. 아래로 내려갈수록 의도를
세세하게 묘사한다. 마지막에는 가장 저차원 함수와 세부 내역이 나온다.

### 개념은 빈 행으로 분리하라

각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이는 빈 행을 넣어 분리해야 한다. 다음 코드를 보자.

```java
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
            Pattern.MULTILINE + Pattern.DOTALL);

    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }

    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

패키지 선언부, import 문, 각 함수 사이 빈 행이 들어간다. 간단한 규칙이지만 세로 레이아웃에 심오한 영향을 미친다. 다음은 빈 행을 뺀 코드이다. 코드 가독성이 현저히 떨어지는 것을 
확인할 수 있다. 

```java
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
            Pattern.MULTILINE + Pattern.DOTALL);
    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1)); }
    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

### 세로 밀집도

줄바꿈이 개념을 분리한다면 세로 밀집도는 연관성을 의미한다. 즉, 서로 밀접한 코드 행은 세로로 가까이 놓여야 한다는 의미이다.

다음은 의미없는 주석으로 두 인스턴스 변수를 떨어뜨려 놓았다.

```java
public class ReporterConfig {
    /**
     * The class name of the reporter listener */
    private String m_className;
    
    /**
     * The properties of the reporter listener */
    private List<Property> m_properties = new ArrayList<Property>();

    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

다음 코드가 훨씬 읽기 쉽다. 코드가 한 눈에 들어온다. 척 보면 변수 2개에 메서드가 1개인 클래스라는 사실이 드러난다.  

```java
public class ReporterConfig {
    private String m_className;
    private List<Property> m_properties = new ArrayList<Property>();

    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

### 수직 거리

서로 밀접한 개념은 세로로 가까이 두어야 한다. 물론 두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않는다. 하지만 타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다. 이것이 바로
`protected` 변수를 피해야 하는 이유 중 하나이다.

같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성을 표현한다. 연관성이란 한 개념을 이해하는 데 다른 개념이 중요한 정도이다. 연관성이 깊은 두 개념이 멀리 떨어져 있으면 
코드를 읽는 사람이 소스 파일과 클래스를 여기저기 뒤지게 된다.

**변수 선언.** 변수는 사용하는 위치에 최대한 가까이 선언한다. 우리가 만든 함수는 매우 짧으므로 지역 변수는 각 함수 맨 처음에 선언한다. 

```java
private static void readPreferences(){
    InputStream is = null;
    try {
        is = new FileInputStream(getPreferencesFile());
        setPreferences(new Properties(getPreferences()));
        getPreferences().load(is);
    } catch(IOException e){
        try {
            if(is!=null)
                is.close();
        } catch(IOException e1){
        }
    }
}
```

루프를 제어하는 변수는 흔히 루프 문 내부에 선언한다. 

```java
public int countTestCases() { 
    int count= 0;
    for (Test each : tests)
        count += each.countTestCases(); 
    return count;
}
```

드물지만 다소 긴 함수에서 블록 상단이나 루프 직전에 변수를 선언하는 사례도 있다. 다음은 아주 긴 함수에 속한다.

```java
// ...
for (XmlTest test : m_suite.getTests()) {
    TestRunner tr = m_runnerFactory.newTestRunner(this, test);
    tr.addListener(m_textReporter); 
    m_testRunners.add(tr);
    invoker = tr.getInvoker();
    for (ITestNGMethod m : tr.getBeforeSuiteMethods()) { 
        beforeSuiteMethods.put(m.getMethod(), m);
    }
    for (ITestNGMethod m : tr.getAfterSuiteMethods()) { 
        afterSuiteMethods.put(m.getMethod(), m);
    } 
}
// ...
```

**인스턴스 변수.** 인스턴스 변수는 클래스 맨 처음에 선언한다. 변수 간 세로로 거리를 두지 않는다. C++의 경우 모든 인스턴스를 클래스 마지막에 선언한다는 소위 가위 규칙(scissors rule)을 적용한다.
하지만 자바에서는 보통 클래스 맨 처음에 인스턴스 변수를 선언한다. 여기서 중요한 점은 변수를 모은다는 사실이다.

**종속 함수.** 한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다. 그러면 프로그램이 자연스럽게 읽힌다. 

```java
public class WikiPageResponder implements SecureResponder { 
    protected WikiPage page;
    protected PageData pageData;
    protected String pageTitle;
    protected Request request;
    protected PageCrawler crawler;
    
    public Response makeResponse(FitNesseContext context, Request request) throws Exception {
        String pageName = getPageNameOrDefault(request, "FrontPage");
        loadPage(pageName, context);
        if (page == null) 
            return notFoundResponse(context, request); 
        else return makePageResponse(context);
    }
    
    private String getPageNameOrDefault(Request request, String defaultPageName) {
        String pageName = request.getResource(); 
        if (StringUtil.isBlank(pageName))
            pageName = defaultPageName;
        return pageName; 
    }
    
    protected void loadPage(String resource, FitNesseContext context) throws Exception {
        WikiPagePath path = PathParser.parse(resource);
        crawler = context.root.getPageCrawler(); 
        crawler.setDeadEndStrategy(new VirtualEnabledPageCrawler()); 
        page = crawler.getPage(context.root, path);
        if (page != null)
            pageData = page.getData();
    }
    
    private Response notFoundResponse(FitNesseContext context, Request request) throws Exception {
        return new NotFoundResponder().makeResponse(context, request);
    }
    
    private SimpleResponse makePageResponse(FitNesseContext context) throws Exception {
        pageTitle = PathParser.render(crawler.getFullPath(page)); 
        String html = makeHtml(context);
        SimpleResponse response = new SimpleResponse(); 
        response.setMaxAge(0); 
        response.setContent(html);
        return response;
    } 
    // ...
}
```

위 코드는 상수를 적절한 수준에 두는 좋은 예다. `getPageNameOrDefault` 함수 안에서 `FrontPage` 상수를 사용하는 방법도 있지만, 그러면 기대와는 달리 잘 알려진 상수가
적절하지 않은 저차원 함수에 묻힌다. 상수를 알아야 마땅한 함수에서 실제 사용하는 함수로 넘겨주는 방법이 더 좋다.

**개념적 유사성.** 어떤 코드는 서로 끌어당긴다. 개념적인 친화도가 높기 때문이다. 친화도가 높은 요인은 여러 가지다. 함수를 호출해 직접적인 종속성이 생기거나, 변수와 그 변수를 사용하는 함수 등. 
그 외에도 비슷한 동작을 수행하는 일군의 함수가 있다. 

```java
public class Assert {
    static public void assertTrue(String message, boolean condition) {
        if (!condition) fail(message);
    }

    static public void assertTrue(boolean condition) {
        assertTrue(null, condition);
    }

    static public void assertFalse(String message, boolean condition) {
        assertTrue(message, !condition);
    }

    static public void assertFalse(boolean condition) {
        assertFalse(null, condition);
    }
// ...
}
```

위 함수들은 개념적 친화도가 매우 높다. 명명법이 똑같고 기본 기능이 유사하고 간단하다. 종속적인 관계가 없더라도 가까이 배치할 함수들이다.

### 세로 순서

일반적으로 함수 호출 종속성은 아래 방향으로 흐른다. 즉, 호출되는 함수를 호출하는 함수보다 나중에 배치한다. 그러면 소스 코드 모듈이 고차원에서 저차원으로 자연스럽게 내려간다. 

## 가로 형식 맞추기

짧은 행이 바람직하다. 필자는 120자 정도로 행 길이을 제한한다.

### 가로 공백과 밀집도

```java
private void measureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars += lineSize; 
    lineWidthHistogram.addLine(lineSize, lineCount); 
    recordWidestLine(lineSize);
}
```

할당 연산자를 강조하려고 앞뒤에 공백을 줬다. 공백을 넣으면 두 가지 주요 요소가 확실히 나뉜다는 사실이 분명해진다.

반면, 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않았다. 함수와 인수는 서로 밀접하기 때문이다. 공백을 넣으면 한 개념이 아니라 별개로 보인다. 함수를 호출하는 코드에서 괄호 안 인수는
공백으로 분리했다. 쉼표를 강조해 인수가 별개라는 사실을 보여주기 위해서다.

우선순위를 강조하기 위해서도 공백을 사용한다.

```java
public class Quadratic {
    public static double root1(double a, double b, double c) {
        double determinant = determinant(a, b, c);
        return (-b + Math.sqrt(determinant)) / (2*a);
    }

    public static double root2(int a, int b, int c) {
        double determinant = determinant(a, b, c);
        return (-b - Math.sqrt(determinant)) / (2*a);
    }

    private static double determinant(double a, double b, double c) {
        return b*b - 4*a*c;
    }
}
```

승수 사이는 공백이 없다. 항 사이 공백이 들어간다. 덧셈, 뺄셈은 우선순위가 곱셈보다 낮기 때문이다. 

### 가로 정렬

변수 이름이나 할당문의 오른쪽 피연산자를 다음과 같이 정렬했다.

```java

public class FitNesseExpediter implements ResponseSender {
    private     Socket              socket;
    private     InputStream         input;  
    private     OutputStream        output; 
    private     Request             request;
    private     Response            response;
    private     FitNesseContext     context;
    protected   long                requestParsingTimeLimit;
    private     long                requestProgress;
    private     long                requestParsingDeadline;
    private     boolean             hasError;

    public FitNesseExpediter(Socket s, 
                             FitNesseContext context) throws Exception
    {
        this.context =              context;
        socket =                    s;
        input =                     s.getInputStream();
        output =                    s.getOutputStream();
        requestParsingTimeLimit =   10000;
    }
}
```

위와 같은 정렬은 엉뚱한 부분을 강조하여 진짜 의도가 가려져 유용하지 못하다. 선언부를 읽다 보면 변수 타입은 무시하고 변수 이름부터 읽게 된다. 

선언문과 할당문을 별도로 정렬하지 않는다면, 오히려 중대한 결함을 찾기 쉽다. 

```java
public class FitNesseExpediter implements ResponseSender {
    private Socket socket;
    private InputStream input;
    private OutputStream output;
    private Request request;
    private Response response;
    private FitNesseContext context;
    protected long requestParsingTimeLimit;
    private long requestProgress;
    private long requestParsingDeadline;
    private boolean hasError;

    public FitNesseExpediter(Socket s, FitNesseContext context) throws Exception {
        this.context = context;
        socket = s;
        input = s.getInputStream();
        output = s.getOutputStream();
        requestParsingTimeLimit = 10000;
    }
    
    // ...
}
```

### 들여쓰기

범위(scope)로 이뤄진 계층을 표현하기 위해 코드를 들여쓴다. 들여쓰는 정도는 계층에서 코드가 자리잡은 수준에 비례한다. 프로그래머는 이런 들여쓰기 체계에 크게 의존한다. 왼쪽으로 코드를 맞춰
코드가 속하는 범위를 시각적으로 표현한다. 

즉, 들여쓰기한 파일은 구조가 한눈에 들어오지만, 들여쓰기 하지 않은 코드는 열심히 분석하지 않는 한 거의 이해할 수 없다.  

**들여쓰기 무시하기.** 때로는 간단한 `if` 문, `while` 문, 짧은 함수에서 들여쓰기를 무시하고픈 유혹이 생길 수 있다. 하지만 들여쓰기로 범위를 제대로 표현해야 한다.

### 가짜 범위

떄로는 빈 `while` 문이나 `for` 문을 접한다. 이럴 경우 빈 블록을 올바로 들여쓰고 괄호로 감싼다. 세미콜론(;)은 새 행에다 들여써서 넣어준다. 이렇게 하지 않으면 눈에 띄지 않는다.

```java
while (dis.read(buf, 0, readBufferSize) != -1)
;
```

## 팀 규칙

팀은 한 가지 규칙에 합의해야 한다. 긜고 모든 팀원은 그 규칙을 따라야 한다. 그래야 소프트웨어가 일관적인 스타일을 보인다.

## 밥 아저씨의 형식 규칙

```java
public class CodeAnalyzer implements JavaFileAnalysis {
    private int lineCount;
    private int maxLineWidth;
    private int widestLineNumber;
    private LineWidthHistogram lineWidthHistogram;
    private int totalChars;

    public CodeAnalyzer() {
        lineWidthHistogram = new LineWidthHistogram();
    }

    public static List<File> findJavaFiles(File parentDirectory) {
        List<File> files = new ArrayList<File>();
        findJavaFiles(parentDirectory, files);
        return files;
    }

    private static void findJavaFiles(File parentDirectory, List<File> files) {
        for (File file : parentDirectory.listFiles()) {
            if (file.getName().endsWith(".java")) 
                files.add(file);
            else if (file.isDirectory()) 
                findJavaFiles(file, files);
        }
    }

    public void analyzeFile(File javaFile) throws Exception {
        BufferedReader br = new BufferedReader(new FileReader(javaFile));
        String line;
        while ((line = br.readLine()) != null)
            measureLine(line);
    }

    private void measureLine(String line) {
        lineCount++;
        int lineSize = line.length();
        totalChars += lineSize;
        lineWidthHistogram.addLine(lineSize, lineCount);
        recordWidestLine(lineSize);
    }

    private void recordWidestLine(int lineSize) {
        if (lineSize > maxLineWidth) {
            maxLineWidth = lineSize;
            widestLineNumber = lineCount;
        }
    }

    public int getLineCount() {
        return lineCount;
    }

    public int getMaxLineWidth() {
        return maxLineWidth;
    }

    public int getWidestLineNumber() {
        return widestLineNumber;
    }

    public LineWidthHistogram getLineWidthHistogram() {
        return lineWidthHistogram;
    }

    public double getMeanLineWidth() {
        return (double) totalChars/lineCount;
    }

    public int getMedianLineWidth() {
        Integer[] sortedWidths = getSortedWidths();
        int cumulativeLineCount = 0;
        for (int width : sortedWidths) {
            cumulativeLineCount += lineCountForWidth(width);
            if (cumulativeLineCount > lineCount/2)
                return width;
        }
        throw new Error("Cannot get here");
    }

    private int lineCountForWidth(int width) {
        return lineWidthHistogram.getLinesforWidth(width).size();
    }

    private Integer[] getSortedWidths() {
        Set<Integer> widths = lineWidthHistogram.getWidths();
        Integer[] sortedWidths = (widths.toArray(new Integer[0]));
        Arrays.sort(sortedWidths);
        return sortedWidths;
    }
}
```