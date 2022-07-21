# JUnit Internals

JUnit은 자바 프레임워크 중 가장 유명하다. 여기서는 JUnit 프레임워크에서 가져온 코드를 평가한다.

## JUnit 프레임워크

살펴볼 코드는 `ComparisonCompactor`라는 모듈로 문자열 비교 오류를 파악할 때 유용한 코드다. `ComparisonCompactor`는 두 문자열을 받아 차이를 반환한다. 예를 들어, `ABCDE`와 `ABXDE`를 받아
`<...B[X]D...>`를 반환한다.

테스트 코드를 보자. 모듈에 필요한 기능이 상세히 드러난다. 

```java
package junit.tests.framework;

import junit.framework.ComparisonCompactor;
import junit.framework.TestCase;

public class ComparisonCompactorTest extends TestCase {
    public void testMessage() {
        String failure = new ComparisonCompactor(0, "b", "c").compact("a");
        assertTrue("a expected:<[b]> but was:<[c]>".equals(failure));
    }

    public void testStartSame() {
        String failure = new ComparisonCompactor(1, "ba", "bc").compact(null);
        assertEquals("expected:<b[a]> but was:<b[c]>", failure);
    }

    public void testEndSame() {
        String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
        assertEquals("expected:<[a]b> but was:<[c]b>", failure);
    }

    public void testSame() {
        String failure = new ComparisonCompactor(1, "ab", "ab").compact(null);
        assertEquals("expected:<ab> but was:<ab>", failure);
    }

    public void testNoContextStartAndEndSame() {
        String failure = new ComparisonCompactor(0, "abc", "adc").compact(null);
        assertEquals("expected:<...[b]...> but was:<...[d]...>", failure);
    }

    public void testStartAndEndContext() {
        String failure = new ComparisonCompactor(1, "abc", "adc").compact(null);
        assertEquals("expected:<a[b]c> but was:<a[d]c>", failure);
    }

    public void testStartAndEndContextWithEllipses() {
        String failure =
                new ComparisonCompactor(1, "abcde", "abfde").compact(null);
        assertEquals("expected:<...b[c]d...> but was:<...b[f]d...>", failure);
    }

    public void testComparisonErrorStartSameComplete() {
        String failure = new ComparisonCompactor(2, "ab", "abc").compact(null);
        assertEquals("expected:<ab[]> but was:<ab[c]>", failure);
    }

    public void testComparisonErrorEndSameComplete() {
        String failure = new ComparisonCompactor(0, "bc", "abc").compact(null);
        assertEquals("expected:<[]...> but was:<[a]...>", failure);
    }

    public void testComparisonErrorEndSameCompleteContext() {
        String failure = new ComparisonCompactor(2, "bc", "abc").compact(null);
        assertEquals("expected:<[]bc> but was:<[a]bc>", failure);
    }

    public void testComparisonErrorOverlapingMatches() {
        String failure = new ComparisonCompactor(0, "abc", "abbc").compact(null);
        assertEquals("expected:<...[]...> but was:<...[b]...>", failure);
    }

    public void testComparisonErrorOverlapingMatchesContext() {
        String failure = new ComparisonCompactor(2, "abc", "abbc").compact(null);
        assertEquals("expected:<ab[]c> but was:<ab[b]c>", failure);
    }

    public void testComparisonErrorOverlapingMatches2() {
        String failure = new ComparisonCompactor(0, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...[d]...> but was:<...[]...>", failure);
    }

    public void testComparisonErrorOverlapingMatches2Context() {
        String failure =
                new ComparisonCompactor(2, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...cd[d]e> but was:<...cd[]e>", failure);
    }

    public void testComparisonErrorWithActualNull() {
        String failure = new ComparisonCompactor(0, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }

    public void testComparisonErrorWithActualNullContext() {
        String failure = new ComparisonCompactor(2, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }

    public void testComparisonErrorWithExpectedNull() {
        String failure = new ComparisonCompactor(0, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }

    public void testComparisonErrorWithExpectedNullContext() {
        String failure = new ComparisonCompactor(2, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }

    public void testBug609972() {
        String failure = new ComparisonCompactor(10, "S&P500", "0").compact(null);
        assertEquals("expected:<[S&P50]0> but was:<[]0>", failure);
    }
}
```

위 테스트 케이스로 `ComparisonCompactor` 모듈에 대한 코드 커버리지가 100%가 나왔다고 한다.

다음은 `ComparisonCompactor` 모듈이다. 코드는 잘 분리되었고, 표현이 적절하며, 구조가 단순하다.

```java
package junit.framework;

public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;

    public ComparisonCompactor(int contextLength, String expected,
                               String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }

    public String compact(String message) {
        if (fExpected == null || fActual == null || areStringsEqual())
            return Assert.format(message, fExpected, fActual);
        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(fExpected);
        String actual = compactString(fActual);
        return Assert.format(message, expected, actual);
    }

    private String compactString(String source) {
        String result = DELTA_START +
                source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
        if (fPrefix > 0)
            result = computeCommonPrefix() + result;
        if (fSuffix > 0)
            result = result + computeCommonSuffix();
        return result;
    }

    private void findCommonPrefix() {
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());
        for (; fPrefix < end; fPrefix++) {
            if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix)) 
                break;
        }
    }

    private void findCommonSuffix() {
        int expectedSuffix = fExpected.length() - 1;
        int actualSuffix = fActual.length() - 1;
        for (;
             actualSuffix >= fPrefix && expectedSuffix >= fPrefix; 
             actualSuffix--, expectedSuffix--) {
            if (fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix)) 
                break;
        }
        fSuffix = fExpected.length() - expectedSuffix;
    }

    private String computeCommonPrefix() {
        return (fPrefix > fContextLength ? ELLIPSIS : "") +
                fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
    }

    private String computeCommonSuffix() {
        int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, 
                fExpected.length());
        return fExpected.substring(fExpected.length() - fSuffix + 1, end) + 
                (fExpected.length() - fSuffix + 1 < fExpected.length() - 
                        fContextLength ? ELLIPSIS : "");
    }

    private boolean areStringsEqual() {
        return fExpected.equals(fActual);
    }
}
```

긴 표현식 몇 개와 이상한 `+1` 등이 눈에 띄긴 하지만 전반적으로 훌륭한 코드다. 

위 코드는 다음과 같이 디팩터링된 코드처럼 짰을 수도 있다.

```java
package junit.framework;

public class ComparisonCompactor {
    private int ctxt;
    private String s1;
    private String s2;
    private int pfx;
    private int sfx;

    public ComparisonCompactor(int ctxt, String s1, String s2) {
        this.ctxt = ctxt;
        this.s1 = s1;
        this.s2 = s2;
    }

    public String compact(String msg) {
        if (s1 == null || s2 == null || s1.equals(s2))
            return Assert.format(msg, s1, s2);
        pfx = 0;
        for (; pfx < Math.min(s1.length(), s2.length()); pfx++) {
            if (s1.charAt(pfx) != s2.charAt(pfx)) break;
        }
        int sfx1 = s1.length() - 1;
        int sfx2 = s2.length() - 1;
        for (; sfx2 >= pfx && sfx1 >= pfx; sfx2--, sfx1--) {
            if (s1.charAt(sfx1) != s2.charAt(sfx2)) break;
        }
        sfx = s1.length() - sfx1;
        String cmp1 = compactString(s1);
        String cmp2 = compactString(s2);
        return Assert.format(msg, cmp1, cmp2);
    }

    private String compactString(String s) {
        String result =
                "[" + s.substring(pfx, s.length() - sfx + 1) + "]";
        if (pfx > 0)
            result = (pfx > ctxt ? "..." : "") + s1.substring(Math.max(0, pfx - ctxt), pfx) + result;
        if (sfx > 0) {
            int end = Math.min(s1.length() - sfx + 1 + ctxt, s1.length());
            result = result + (s1.substring(s1.length() - sfx + 1, end) +
                    (s1.length() - sfx + 1 < s1.length() - ctxt ? "..." : ""));
        }
        return result;
    }
}
```

보이스카우트 규칙에 따르면 우리는 처음 왔을 때보다 더 깨끗하게 해놓고 떠나야 한다. 원래의 코드를 어떻게 개선할까?

가장 먼저 거슬리는 부분은 멤버 변수 앞 `f`다. 오늘날 IDE에서는 이처럼 변수 이름에 스코프를 명시할 필요가 없다. `f`는 중복되는 정보이므로 모두 제거한다.

```java
private int contextLength;
private String expected;
private String actual;
private int prefix; 
private int suffix;
```

다음 `compact` 함수 시작부에 캡슐화되지 않은 조건문이 보인다.  

```java
if (fExpected == null || fActual == null || areStringsEqual())
```

의도를 정확히 표현하려면 조건문을 캡슐화해야 한다. 즉, 조건문을 메서드로 뽑아 적절한 이름을 붙인다.

```java
public String compact(String message) { 
    if (shouldNotCompact())
        return Assert.format(message, expected, actual);
    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(this.expected); 
    String actual = compactString(this.actual); 
    return Assert.format(message, expected, actual);
}

private boolean shouldNotCompact() {
    return expected == null || actual == null || areStringsEqual();
}
```

`compact` 함수에서 사용하는 `this.expected`와 `this.actual`도 눈에 거슬린다. 함수에 이미 `expected`라는 지역 변수가 있기 때문이다. 멤버 변수와 다른 의미이므로 이름을 명확하게 붙인다.

```java
String compactExpected = compactString(expected); 
String compactActual = compactString(actual);
```

부정문은 긍정문보다 이해하기 약간 더 어렵다. 그러므로 첫 문장 `if`를 긍정으로 만들어 조건문을 반전한다.

```java

public String compact(String message) { 
    if (canBeCompacted()) {
        findCommonPrefix();
        findCommonSuffix();
        String compactExpected = compactString(expected); 
        String compactActual = compactString(actual);
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    } 
}

private boolean canBeCompacted() {
    return expected != null && actual != null && !areStringsEqual();
}
```

문자열을 압축하는 함수지만 실제로 `canBeCompacted`가 `false`라면 압축하지 않으르모 함수 이름이 적절하지 않다. 또한 함수는 단순히 압축된 문자열이 아니라 형식이 갖춰진 문자열을 반환한다. 
따라서 `formatCompactedComparison`라는 이름이 적합하다. 새 이름에 인수를 고려하면 가독성이 훨씬 좋아진다.  

```java
public String formatCompactedComparison(String message) {
```

`if` 문 안에서 예상 문자열과 실제 문자열을 압축한다. 이 부분을 빼내 `compactExpectedAndActual`이라는 메서드로 만든다. 형식을 맞추는 작업은 `formatCompactedComparison`에 맡긴다.

```java
// ...
private String compactExpected; 
private String `compactActual`;
// ...
public String formatCompactedComparison(String message) { 
    if (canBeCompacted()) { 
        compactExpectedAndActual();
        return Assert.format(message, compactExpected, compactActual); 
    } else {
        return Assert.format(message, expected, actual);
    }
}

private void compactExpectedAndActual() { 
    findCommonPrefix();
    findCommonSuffix();
    compactExpected = compactString(expected); 
    compactActual = compactString(actual);
}
```

`compactExpected`와 `compactExpected`을 멤버 변수로 변경했다. `formatCompactedComparison` 함수에서 마지막 두 줄은 변수를 반환하지만 첫째 줄과 둘째 줄은 반환값이 없다. 
함수 사용방식이 일관적이지 못하다. 그러므로 `findCommonPrefix`와 `findCommonSuffix`를 변경해 접두어 값과 접미어 값을 반환한다.

```java
private void compactExpectedAndActual() { 
    prefixIndex = findCommonPrefix();
    suffixIndex = findCommonSuffix(); 
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}

private int findCommonPrefix() {
    int prefixIndex = 0;
    int end = Math.min(expected.length(), actual.length()); 
    for (; prefixIndex < end; prefixIndex++) {
        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) 
            break;
    }
    return prefixIndex;
}

private int findCommonSuffix() {
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex;
        actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
            break; 
    }
    return expected.length() - expectedSuffix; 
}
```

멤버 변수 이름도 좀 더 정확하게 바꿨다. 결국 둘 다 색인 위치를 나타내기 때문이다.

`findCommonSuffix`를 주의 깊게 보면, 숨겨진 시간적인 결합(hidden temporal coupling)이 존재한다. 즉, `findCommonSuffix`는 `findCommonPrefix`가 `prefixIndex`를 
계산한다는 사실에 의존한다. 그래서 시간 결합을 외부에 노출하고자 `findCommonSuffix`를 고쳐 `prefixIndex`를 인수로 넘겼다.

```java
private void compactExpectedAndActual() { 
    prefixIndex = findCommonPrefix(); 
    suffixIndex = findCommonSuffix(prefixIndex);
    compactExpected = compactString(expected);
    compactActual = compactString(actual); 
}

private int findCommonSuffix(int prefixIndex) {
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex;
        actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
            break; 
    }
    return expected.length() - expectedSuffix; 
}
```

이 방법이 썩 내키진 않는다. `prefixIndex`를 인수로 전달하는 방식은 다소 자의적이다. 함수 호출 순서는 정해지지만 `prefixIndex`가 필요한 이유는 설명하지 못한다. 그러므로 다른 방식을 고안하자.

```java
private void compactExpectedAndActual() { 
    findCommonPrefixAndSuffix(); 
    compactExpected = compactString(expected); 
    compactActual = compactString(actual);
}

private void findCommonPrefixAndSuffix() {
    findCommonPrefix();
    int expectedSuffix = expected.length() - 1; 
    int actualSuffix = actual.length() - 1; 
    for (;actualSuffix >= prefixIndex && 
        expectedSuffix >= prefixIndex;
        actualSuffix--, expectedSuffix-- ) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) break;
    }
    suffixIndex = expected.length() - expectedSuffix; 
}

private void findCommonPrefix() {
    prefixIndex = 0;
    int end = Math.min(expected.length(), actual.length());
    for (; prefixIndex < end; prefixIndex++)
        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) 
            break;
}
```

`findCommonPrefix`와 `findCommonSuffix`를 원래대로 되돌리고, `findCommonSuffix`라는 이름을 `findCommonPrefixAndSuffix`로 바꾸고, `findCommonPrefixAndSuffix`에서 가장
먼저 `findCommonPrefix`를 호출한다. 그러면 두 함수를 호출하는 순서가 앞서 고친 코드보다 분명해진다. 또한 `findCommonPrefixAndSuffix` 함수가 얼마나 지저분한지도 드러난다. 이를 정리해보자.

```java
private void findCommonPrefixAndSuffix() { 
    findCommonPrefix();
    int suffixLength = 1;
    for (; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
        if (charFromEnd(expected, suffixLength) !=
        charFromEnd(actual, suffixLength)) 
            break;
    }
    suffixIndex = suffixLength; 
}

private char charFromEnd(String s, int i) { 
    return s.charAt(s.length()-i);
}

private boolean suffixOverlapsPrefix(int suffixLength) { 
    return actual.length() - suffixLength < prefixLength ||
        expected.length() - suffixLength < prefixLength;
}
```

코드를 고치고 나니 `suffixIndex`가 실제로는 접미어 길이라는 사실이 드러난다. 이름이 적절하지 않다. `prefixIndex`도 마찬가지다. `index`와 `length`가 동의어지만 그렇다 하더라도 `length` 더 합당하다.
`suffixIndex`는 0에서 시작하지 않는다. 1에서 시작하므로 진정한 길이가 아니다. `computeCommonSuffix`에 `+1`이 곳곳에 등장하는 이유다. 이를 정리해보자.

```java
public class ComparisonCompactor {
    // ...
    private int suffixLength;

    // ...
    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;
        for (; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
            if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength))
                break;
        }
    }

    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }

    private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() - suffixLength <= prefixLength ||
                expected.length() - suffixLength <= prefixLength;
    }

    // ...
    private String compactString(String source) {
        String result =
                DELTA_START +
                        source.substring(prefixLength, source.length() - suffixLength) + DELTA_END;
        if (prefixLength > 0)
            result = computeCommonPrefix() + result;
        if (suffixLength > 0)
            result = result + computeCommonSuffix();
        return result;
    }

    // ...
    private String computeCommonSuffix() {
        int end = Math.min(expected.length() - suffixLength + 
                contextLength, expected.length());
        return expected.substring(expected.length() - suffixLength, end) + 
                (expected.length() - suffixLength < 
                        expected.length() - contextLength ? 
                        ELLIPSIS : "");
    }
}
```

`computeCommonSuffix`에서 `+1`을 없애고 `charFormEnd`dp `-1`을 추가하고 `suffixOverlapsPrefix`에 `<=`를 사용했다. 그 다음 `suffixIndex`를 `suffixLength`로 바꿨다.
이로써 가독성이 크게 높아졌다.

그런데 `+1`을 제거하던 중 `compactString`에서 다음 행을 발견했다.

```java
if (suffixLength > 0)
```

`suffixLength`가 1씩 감소했으므로 `>` 연산자를 `>=` 연산자로 고쳐야 한다. 하지만 `>=` 연산자는 말이 안 된다. 지금 그대로 `>` 연산자가 맞다. 즉, 원래 코드가 틀렸으며 버그라는 말이다. 엄밀하게
버그는 아니다. 코드를 좀 더 보면 `if` 문은 길이가 0인 접미어를 걸러낸다. 원래 코드는 `suffixIndex`가 언제나 1 이상이였으므로 `if` 문 자체가 있으나마나였다.

`compactString`에 있는 `if` 문 둘 다 필요 없어 보인다. 불필요한 `if` 문을 제거하고 `compactString` 구조를 다듬어 좀 더 깔끔하게 만들자.

```java

private String compactString(String source) { 
    return 
        computeCommonPrefix() +
        DELTA_START +
        source.substring(prefixLength, source.length() - suffixLength) + DELTA_END +
        computeCommonSuffix();
}
```

`compactString`은 단순히 문자열 조각만 결합한다. 사소하게 손 볼 곳이 아직 많다. 다음은 최종 코드이다.

```java
package junit.framework;
public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    
    private int contextLength; 
    private String expected; 
    private String actual;
    private int prefixLength;
    private int suffixLength;
    
    public ComparisonCompactor( 
            int contextLength, String expected, String actual
    ) {
        this.contextLength = contextLength;
        this.expected = expected;
        this.actual = actual;
    }

    public String formatCompactedComparison(String message) {
        String compactExpected = expected;
        String compactActual = actual;
        if (shouldBeCompacted()) {
            findCommonPrefixAndSuffix();
            compactExpected = compact(expected);
            compactActual = compact(actual);
        }
        return Assert.format(message, compactExpected, compactActual);
    }

    private boolean shouldBeCompacted() {
        return !shouldNotBeCompacted();
    }

    private boolean shouldNotBeCompacted() {
        return expected == null ||
                actual == null || expected.equals(actual);
    }

    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;
        for (; !suffixOverlapsPrefix(); suffixLength++) {
            if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength)
            )
                break;
        }
    }

    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }

    private boolean suffixOverlapsPrefix() {
        return actual.length() - suffixLength <= prefixLength ||
                expected.length() - suffixLength <= prefixLength;
    }

    private void findCommonPrefix() {
        prefixLength = 0;
        int end = Math.min(expected.length(), actual.length());
        for (; prefixLength < end; prefixLength++)
            if (expected.charAt(prefixLength) != actual.charAt(prefixLength)) break;
    }

    private String compact(String s) {
        return new StringBuilder()
                .append(startingEllipsis()).append(startingContext()).append(DELTA_START).append(delta(s)).append(DELTA_END).append(endingContext()).append(endingEllipsis()).toString();
    }

    private String startingEllipsis() {
        return prefixLength > contextLength ? ELLIPSIS : "";
    }

    private String startingContext() {
        int contextStart = Math.max(0, prefixLength - contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }

    private String delta(String s) {
        int deltaStart = prefixLength;
        int deltaEnd = s.length() - suffixLength;
        return s.substring(deltaStart, deltaEnd);
    }

    private String endingContext() {
        int contextStart = expected.length() - suffixLength;
        int contextEnd =
                Math.min(contextStart + contextLength, expected.length());
        return expected.substring(contextStart, contextEnd);
    }

    private String endingEllipsis() {
        return (suffixLength > contextLength ? ELLIPSIS : "");
    }
}
```

모듈은 일련의 분석 함수와 일련의 조합 함수로 나뉜다. 전체 함수는 위상적으로 정렬했으므로 각 함수가 사용된 직후에 정의된다. 분석 함수가 먼저 나오고 조합 함수가 그 뒤를 이어서 나온다.

## 결론

모듈은 처음보다 더 깨끗해졌다. 원래 깨끗하지 못했다는 말은 아니다. 저자들은 우수한 모듈을 만들었다. 하지만 세상에 개선이 불필요한 모듈은 없다. 코드를 처음보다 더 깨끗하게 만드는 책임은
우리 모두에게 있다.