# Successive Refinement

여기서는 확장성이 부족했던 모듈을 소개한다. 그런 다음, 모듈을 개선하고 정리하는 단계를 살펴본다.

명령행 인수의 구문을 북석할 새롭게 만들 유틸리티를 `Args`라 하겠다. 

`Args` 생성자에 입력으로 들어온 인수 문자열과 형식 문자열을 넘겨 `Args` 인스턴스를 생성한 후 `Args` 인스턴스에 인수 값을 질의한다. 다음 예를 보자.

```java
public static void main(String[] args) { 
    try {
        Args arg = new Args("l,p#,d*", args);
        boolean logging = arg.getBoolean('l');
        int port = arg.getInt('p');
        String directory = arg.getString('d');
        executeApplication(logging, port, directory);
    } catch (ArgsException e) {
        System.out.printf("Argument error: %s\n", e.errorMessage());
    } 
}
```

매개변수 두 개로 `Args` 클래스의 인스턴스를 만들었다. 첫째 매개변수는 형식 또는 스키마를 지정하는 `l,p#,d*`다. 이 문자열은 명령행 인수 세 개를 정의한다. 

1. -l: 부울 인수.
2. -p: 정수 인수.
3. -d: 문자열 인수.

둘째 매개변수는 `main`으로 넘어온 명령행 인수 배열 자체다.

`ArgsException`이 발생하지 않는다면 명령행 인수의 구문을 성공적으로 분석했고 `Args` 인스턴스에 질의를 던져도 좋다는 의미이다. 

## Args 구현

다음은 `Args` 클래스다. 

```java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

import java.util.*;

public class Args {
    private Map<Character, ArgumentMarshaler> marshalers;

    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;

    public Args(String schema, String[] args) throws ArgsException {
        marshalers = new HashMap<Character, ArgumentMarshaler>();
        argsFound = new HashSet<Character>();
        parseSchema(schema);
        parseArgumentStrings(Arrays.asList(args));
    }

    private void parseSchema(String schema) throws ArgsException {
        for (String element : schema.split(","))
            if (element.length() > 0) 
                parseSchemaElement(element.trim());
    }

    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (elementTail.length() == 0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (elementTail.equals("*")) 
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if (elementTail.equals("##")) 
            marshalers.put(elementId, new DoubleArgumentMarshaler());
        else if (elementTail.equals("[*]"))
            marshalers.put(elementId, new StringArrayArgumentMarshaler());
        else
            throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
    }

    private void validateSchemaElementId(char elementId) throws ArgsException {
        if (!Character.isLetter(elementId))
            throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null);
    }

    private void parseArgumentStrings(List<String> argsList) throws ArgsException {
        for (currentArgument = argsList.listIterator(); currentArgument.hasNext(); ) {
            String argString = currentArgument.next();
            if (argString.startsWith("-")) {
                parseArgumentCharacters(argString.substring(1));
            } else {
                currentArgument.previous();
                break;
            }
        }
    }

    private void parseArgumentCharacters(String argChars) throws ArgsException {
        for (int i = 0; i < argChars.length(); i++)
            parseArgumentCharacter(argChars.charAt(i));
    }

    private void parseArgumentCharacter(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null) {
            throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null);
        } else {
            argsFound.add(argChar);
            try {
                m.set(currentArgument);
            } catch (ArgsException e) {
                e.setErrorArgumentId(argChar);
                throw e;
            }
        }
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }

    public int nextArgument() {
        return currentArgument.nextIndex();
    }

    public boolean getBoolean(char arg) {
        return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public String getString(char arg) {
        return StringArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public int getInt(char arg) {
        return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public double getDouble(char arg) {
        return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public String[] getStringArray(char arg) {
        return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
    }
}
```

다음은 `ArgumentMarshaler` 인터페이스와 파생 클래스들이다.

```java
public interface ArgumentMarshaler { 
    void set(Iterator<String> currentArgument) throws ArgsException;
}
```

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler {
    private boolean booleanValue = false;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        booleanValue = true;
    }

    public static boolean getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof BooleanArgumentMarshaler)
            return ((BooleanArgumentMarshaler) am).booleanValue;
        else
            return false;
    }
}
```

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_STRING);
        }
    }

    public static String getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof StringArgumentMarshaler)
            return ((StringArgumentMarshaler) am).stringValue;
        else
            return "";
    }
}
```

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class IntegerArgumentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            throw new ArgsException(INVALID_INTEGER, parameter);
        }
    }

    public static int getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof IntegerArgumentMarshaler)
            return ((IntegerArgumentMarshaler) am).intValue;
        else
            return 0;
    }
}
```

나머지 `DoubleArgumentMarshaler`와 `StringArrayArgumentMarshaler`는 생략한다. 

다음은 오류 코드 상수를 정의하는 부분이다.

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;

    public ArgsException() {
    }

    public ArgsException(String message) {
        super(message);
    }

    public ArgsException(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }

    public ArgsException(ErrorCode errorCode, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
    }

    public ArgsException(ErrorCode errorCode,
                         char errorArgumentId, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
        this.errorArgumentId = errorArgumentId;
    }

    public char getErrorArgumentId() {
        return errorArgumentId;
    }

    public void setErrorArgumentId(char errorArgumentId) {
        this.errorArgumentId = errorArgumentId;
    }

    public String getErrorParameter() {
        return errorParameter;
    }

    public void setErrorParameter(String errorParameter) {
        this.errorParameter = errorParameter;
    }

    public ErrorCode getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }

    public String errorMessage() {
        switch (errorCode) {
            case OK:
                return "TILT: Should not get here.";
            case UNEXPECTED_ARGUMENT:
                return String.format("Argument -%c unexpected.", errorArgumentId);
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.",
                        errorArgumentId);
            case INVALID_INTEGER:
                return String.format("Argument -%c expects an integer but was '%s'.",
                        errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format("Could not find integer parameter for -%c.",
                        errorArgumentId);
            case INVALID_DOUBLE:
                return String.format("Argument -%c expects a double but was '%s'.",
                        errorArgumentId, errorParameter);
            case MISSING_DOUBLE:
                return String.format("Could not find double parameter for -%c.",
                        errorArgumentId);
            case INVALID_ARGUMENT_NAME:
                return String.format("'%c' is not a valid argument name.", errorArgumentId);
            case INVALID_ARGUMENT_FORMAT:
                return String.format("'%s' is not a valid argument format.",
                        errorParameter);
        }
        return "";
    }

    public enum ErrorCode {
        OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, 
        INVALID_ARGUMENT_NAME, 
        MISSING_STRING,
        MISSING_INTEGER, INVALID_INTEGER,
        MISSING_DOUBLE, INVALID_DOUBLE
    }
}
```

자바는 정적 타입 언어라서 타입 시스템을 만족하려면 많은 단어가 필요하므로, 코드가 길어질 수 있다.

이름을 붙인 방법, 함수 크기, 코드 형식에 주목해 코드를 보자. 전반적으로 깔끔한 구조에 잘 짜인 프로그램인 것을 알 수 있다.

예를 들어, 새로운 인수 유형을 추가하는 방법은 `ArgumentMarshaler`에서 새 클래스를 파생해 `getXXX` 함수를 추가한 후 `parseSchemaElement` 함수에 새 `case` 문만 추가하면 끝이다.

### 어떻게 짰느냐고?

깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤 정리해야 한다.

## Args: 1차 초안

필자가 맨 처음 짠 `Args` 코드이다. 코드는 돌아가지만 엉망이다.

```java
import java.text.ParseException;
import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, Boolean> booleanArgs =
            new HashMap<Character, Boolean>();
    private Map<Character, String> stringArgs = new HashMap<Character, String>();
    private Map<Character, Integer> intArgs = new HashMap<Character, Integer>();
    private Set<Character> argsFound = new HashSet<Character>();
    private int currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;

    private enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT
    }

    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }

    private boolean parse() throws ParseException {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }

    private boolean parseSchema() throws ParseException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                String trimmedElement = element.trim();
                parseSchemaElement(trimmedElement);
            }
            return true;
        }
    }

    private void parseSchemaElement(String element) throws ParseException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (isBooleanSchemaElement(elementTail)) 
            parseBooleanSchemaElement(elementId);
        else if (isStringSchemaElement(elementTail)) 
            parseStringSchemaElement(elementId);
        else if (isIntegerSchemaElement(elementTail)) {
            parseIntegerSchemaElement(elementId);
        } else {
            throw new ParseException(
                    String.format("Argument: %c has invalid format: %s.", elementId, elementTail), 0);
        }
    }

    private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
            throw new ParseException(
                    "Bad character:" + elementId + "in Args format: " + schema, 0);
        }
    }

    private void parseBooleanSchemaElement(char elementId) {
        booleanArgs.put(elementId, false);
    }

    private void parseIntegerSchemaElement(char elementId) {
        intArgs.put(elementId, 0);
    }

    private void parseStringSchemaElement(char elementId) {
        stringArgs.put(elementId, "");
    }

    private boolean isStringSchemaElement(String elementTail) {
        return elementTail.equals("*");
    }

    private boolean isBooleanSchemaElement(String elementTail) {
        return elementTail.length() == 0;
    }

    private boolean isIntegerSchemaElement(String elementTail) {
        return elementTail.equals("#");
    }

    private boolean parseArguments() throws ArgsException {
        for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
            String arg = args[currentArgument];
            parseArgument(arg);
        }
        return true;
    }

    private void parseArgument(String arg) throws ArgsException {
        if (arg.startsWith("-"))
            parseElements(arg);
    }

    private void parseElements(String arg) throws ArgsException {
        for (int i = 1; i < arg.length(); i++)
            parseElement(arg.charAt(i));
    }

    private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            unexpectedArguments.add(argChar);
            errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
            valid = false;
        }
    }

    private boolean setArgument(char argChar) throws ArgsException {
        if (isBooleanArg(argChar))
            setBooleanArg(argChar, true);
        else if (isStringArg(argChar))
            setStringArg(argChar);
        else if (isIntArg(argChar))
            setIntArg(argChar);
        else
            return false;
        return true;
    }

    private boolean isIntArg(char argChar) {
        return intArgs.containsKey(argChar);
    }

    private void setIntArg(char argChar) throws ArgsException {
        currentArgument++;
        String parameter = null;
        try {
            parameter = args[currentArgument];
            intArgs.put(argChar, new Integer(parameter));
        } catch (ArrayIndexOutOfBoundsException e) {
            valid = false;
            errorArgumentId = argChar;
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            valid = false;
            errorArgumentId = argChar;
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw new ArgsException();
        }
    }

    private void setStringArg(char argChar) throws ArgsException {
        currentArgument++;
        try {
            stringArgs.put(argChar, args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
            valid = false;
            errorArgumentId = argChar;
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }

    private boolean isStringArg(char argChar) {
        return stringArgs.containsKey(argChar);
    }

    private void setBooleanArg(char argChar, boolean value) {
        booleanArgs.put(argChar, value);
    }

    private boolean isBooleanArg(char argChar) {
        return booleanArgs.containsKey(argChar);
    }

    public int cardinality() {
        return argsFound.size();
    }

    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }

    public String errorMessage() throws Exception {
        switch (errorCode) {
            case OK:
                throw new Exception("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
                return unexpectedArgumentMessage();
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.",
                        errorArgumentId);
            case INVALID_INTEGER:
                return String.format("Argument -%c expects an integer but was '%s'.",
                        errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format("Could not find integer parameter for -%c.",
                        errorArgumentId);
        }
        return "";
    }

    private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Argument(s) -");
        for (char c : unexpectedArguments) {
            message.append(c);
        }
        message.append(" unexpected.");
        return message.toString();
    }

    private boolean falseIfNull(Boolean b) {
        return b != null && b;
    }

    private int zeroIfNull(Integer i) {
        return i == null ? 0 : i;
    }

    private String blankIfNull(String s) {
        return s == null ? "" : s;
    }

    public String getString(char arg) {
        return blankIfNull(stringArgs.get(arg));
    }

    public int getInt(char arg) {
        return zeroIfNull(intArgs.get(arg));
    }

    public boolean getBoolean(char arg) {
        return falseIfNull(booleanArgs.get(arg));
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }

    public boolean isValid() {
        return valid;
    }

    private class ArgsException extends Exception {
    }
}
```

인스턴스 변수 개수도 압도적이며, `TILT`와 같은 문자열, `HashSets`와 `TreeSets`, `try-catch-catch` 블록 등 모두가 지저분한 코드에 기여하는 요인이다.

첫 버전은 이만큼 엉망이지 않았다. 다음은 `Boolean` 인수만 지원하던 초기 버전이다.

```java
package com.objectmentor.utilities.getopts;

import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
    private int numberOfArguments = 0;

    public Args(String schema, String[] args) {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }

    public boolean isValid() {
        return valid;
    }

    private boolean parse() {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        parseArguments();
        return unexpectedArguments.size() == 0;
    }

    private boolean parseSchema() {
        for (String element : schema.split(",")) {
            parseSchemaElement(element);
        }
        return true;
    }

    private void parseSchemaElement(String element) {
        if (element.length() == 1) {
            parseBooleanSchemaElement(element);
        }
    }

    private void parseBooleanSchemaElement(String element) {
        char c = element.charAt(0);
        if (Character.isLetter(c)) {
            booleanArgs.put(c, false);
        }
    }

    private boolean parseArguments() {
        for (String arg : args)
            parseArgument(arg);
        return true;
    }

    private void parseArgument(String arg) {
        if (arg.startsWith("-"))
            parseElements(arg);
    }

    private void parseElements(String arg) {
        for (int i = 1; i < arg.length(); i++)
            parseElement(arg.charAt(i));
    }

    private void parseElement(char argChar) {
        if (isBoolean(argChar)) {
            numberOfArguments++;
            setBooleanArg(argChar, true);
        } else
            unexpectedArguments.add(argChar);
    }

    private void setBooleanArg(char argChar, boolean value) {
        booleanArgs.put(argChar, value);
    }

    private boolean isBoolean(char argChar) {
        return booleanArgs.containsKey(argChar);
    }

    public int cardinality() {
        return numberOfArguments;
    }

    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }
    
    public String errorMessage() {
        if (unexpectedArguments.size() > 0) {
            return unexpectedArgumentMessage();
        } else return "";
    }

    private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Argument(s) -");
        for (char c : unexpectedArguments) {
            message.append(c);
        }
        message.append(" unexpected.");
        return message.toString();
    }

    public boolean getBoolean(char arg) {
        return booleanArgs.get(arg);
    }
}
```

위 코드도 불평거리가 많지만 나름대로 괜찮은 코드다. 간결하고 단순하며 이해하기 쉽다. 하지만 코드를 잘 살펴보면 나중에 엉망으로 변해갈 씨앗이 보인다. 

위 코드에 `String`과 `Integer`라는 인수 타입을 추가했을 뿐인데, 코드가 엄청 지저분해졌다. 먼저 `String` 인수 유형을 추가했다.

```java
package com.objectmentor.utilities.getopts;

import java.text.ParseException;
import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, Boolean> booleanArgs =
            new HashMap<Character, Boolean>();
    private Map<Character, String> stringArgs = new HashMap<Character, String>();
    private Set<Character> argsFound = new HashSet<Character>();
    private int currentArgument;
    private char errorArgument = '\0';

    enum ErrorCode {
        OK, MISSING_STRING
    }

    private ErrorCode errorCode = ErrorCode.OK;

    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }

    private boolean parse() throws ParseException {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        parseArguments();
        return valid;
    }

    private boolean parseSchema() throws ParseException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                String trimmedElement = element.trim();
                parseSchemaElement(trimmedElement);
            }
        }
        return true;
    }

    private void parseSchemaElement(String element) throws ParseException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (isBooleanSchemaElement(elementTail)) 
            parseBooleanSchemaElement(elementId);
        else if (isStringSchemaElement(elementTail)) 
            parseStringSchemaElement(elementId);
    }

    private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
            throw new ParseException(
                    "Bad character:" + elementId + "in Args format: " + schema, 0);
        }
    }

    private void parseStringSchemaElement(char elementId) {
        stringArgs.put(elementId, "");
    }

    private boolean isStringSchemaElement(String elementTail) {
        return elementTail.equals("*");
    }

    private boolean isBooleanSchemaElement(String elementTail) {
        return elementTail.length() == 0;
    }

    private void parseBooleanSchemaElement(char elementId) {
        booleanArgs.put(elementId, false);
    }

    private boolean parseArguments() {
        for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
            String arg = args[currentArgument];
            parseArgument(arg);
        }
        return true;
    }

    private void parseArgument(String arg) {
        if (arg.startsWith("-"))
            parseElements(arg);
    }

    private void parseElements(String arg) {
        for (int i = 1; i < arg.length(); i++)
            parseElement(arg.charAt(i));
    }

    private void parseElement(char argChar) {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            unexpectedArguments.add(argChar);
            valid = false;
        }
    }

    private boolean setArgument(char argChar) {
        boolean set = true;
        if (isBoolean(argChar))
            setBooleanArg(argChar, true);
        else if (isString(argChar))
            setStringArg(argChar, "");
        else
            set = false;
        return set;
    }

    private void setStringArg(char argChar, String s) {
        currentArgument++;
        try {
            stringArgs.put(argChar, args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
            valid = false;
            errorArgument = argChar;
            errorCode = ErrorCode.MISSING_STRING;
        }
    }
    
    private boolean isString(char argChar) {
        return stringArgs.containsKey(argChar);
    }

    private void setBooleanArg(char argChar, boolean value) {
        booleanArgs.put(argChar, value);
    }

    private boolean isBoolean(char argChar) {
        return booleanArgs.containsKey(argChar);
    }

    public int cardinality() {
        return argsFound.size();
    }

    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }

    public String errorMessage() throws Exception {
        if (unexpectedArguments.size() > 0) {
            return unexpectedArgumentMessage();
        } else
            switch (errorCode) {
                case MISSING_STRING:
                    return String.format("Could not find string parameter for -%c.",
                            errorArgument);
                case OK:
                    throw new Exception("TILT: Should not get here.");
            }
        return "";
    }

    private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Argument(s) -");
        for (char c : unexpectedArguments) {
            message.append(c);
        }
        message.append(" unexpected.");
        return message.toString();
    }

    public boolean getBoolean(char arg) {
        return falseIfNull(booleanArgs.get(arg));
    }

    private boolean falseIfNull(Boolean b) {
        return b == null ? false : b;
    }

    public String getString(char arg) {
        return blankIfNull(stringArgs.get(arg));
    }

    private String blankIfNull(String s) {
        return s == null ? "" : s;
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }

    public boolean isValid() {
        return valid;
    }
}
```

이제 코드는 통제를 벗어나기 시작했다. `Integer` 인수 유형을 추가하니 코드는 완전히 엉망이 되었다.

### 그래서 멈췄다

추가할 유형이 두 개는 더 있었는데 그러면 코드가 더 나빠지리라는 사실이 자명했다. 그래서 기능을 더 이상 추가하지 않고 리팩터링을 시작했다. `String`과 `Integer` 유형을 추가한 경험에서 새 인수 유형을
추가하려면 주요 지점 세 곳에 코드를 추가해야 한다는 사실을 깨달았다. 

1. 인수 유형에 해당하는 `HashMap`을 선택하기 위해 스키마 요소의 구문을 분석한다.
2. 명령행 인수에서 인수 유형을 분석해 진짜 유형으로 변환한다.
3. `getXXX` 메서드를 구현해 호출자에게 진짜 유형을 반환한다.

인수 유형은 다양하지만 모두가 유사한 메서드를 제공하므로 클래스 하나가 적합하다 판단했다. 그래서 `ArgumentMarshaler`라는 개념이 탄생했다.

### 점진적으로 개선하다

TDD 기법을 이용해 시스템이 변경 전과 똑같이 돌아가도록 만들었다. 이미 유닛 테스트와 인수 테스트를 만들어 놓았다. 유닛 테스트는 JUnit 프레임워크에서 자바로, 인수 테스트는
FitNess에서 위키 페이지로 작성했다. 두 테스트 모두 언제든 실행이 가능했으며, 시스템이 두 테스트를 모두 통과하면 올바로 동작한다고 봐도 좋았다.

가장 먼저 기존 코드 끝에 `ArgumentMarshaler` 클래스의 골격을 추가했다.

```java
private class ArgumentMarshaler {
    private boolean booleanValue = false;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
}
```

다음으로 코드를 최소로 건드리는, 가장 단순한 변경을 헀다. `Boolean` 인수를 저장하는 `HashMap`에서 `Boolean` 인수 유형을 `ArgumentMarshaler` 유형으로 바꿨다.

```java
private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<Character, ArgumentMarshaler>();
```

그러면 코드 일부가 깨지므로 다음과 같이 수정했다.

```java
// ...
private void parseBooleanSchemaElement(char elementId) {
    booleanArgs.put(elementId, new BooleanArgumentMarshaler()); 
}
// ...
private void setBooleanArg(char argChar, boolean value) { 
    booleanArgs.get(argChar).setBoolean(value);
} 
// ...
public boolean getBoolean(char arg) {
    return falseIfNull(booleanArgs.get(arg).getBoolean());
}
```

앞서 새 인수 유형을 추가하려면 세 곳(`parse`, `get`, `set`)을 변경해야 한다고 했는데, 위에서 수정한 부분과 정확히 일치한다. 이는 사소한 변경이었으나 몇몇 테스트 케이스가 실패했다.
`getBoolean` 함수를 보면 `y` 인수가 없는데 args로 `y`를 넘긴다면 `booleanArgs.get('y')`는 `null`을 반환하고 함수는 `NullPointerException`을 발생시킨다. 이전 코드는 
`falseIfNull` 함수로 `NullPointerException`를 막았으나 고친 코드는 `falseIfNull` 함수가 아무런 효력을 발휘하지 못했다.

이를 수정하는 것은 단순히 `null` 위치만 바꿔주면 충분했다. 이제 `null`인지 확인할 객체는 `boolean`이 아니라 `ArgumentMarshaler`였다.

먼저 `getBoolean` 함수에서 `falseIfNull`을 제거했다. 쓸모가 없으므로 해당 함수를 없앴다. 

```java
public boolean getBoolean(char arg) { 
    return booleanArgs.get(arg).getBoolean();
}
```

다음으로 함수를 두 행으로 쪼갠 후 `ArgumentMarshaler`를 `argumentMarshaler`라는 독자적인 변수에 저장했다. 변수의 이름이 길고, 유형 이름과 중복이 심했으며, 함수가 길어져 변수 이름을 `am`으로 줄였다.

```java
public boolean getBoolean(char arg) { 
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am.getBoolean();
}
```

그 다음 `null`을 점검했다.

```java
public boolean getBoolean(char arg) { 
    Args.ArgumentMarshaler am = booleanArgs.get(arg); 
    return am != null && am.getBoolean();
}
```

## String 인수

`String` 인수를 추가하는 과정은 `boolean`과 매우 유사하다. `HashMap`을 변경한 후 `parse`, `get`, `set` 함수를 고쳤다. 

```java
private Map<Character, ArgumentMarshaler> stringArgs = 
        new HashMap<Character, ArgumentMarshaler>();
// ...

private void parseStringSchemaElement(char elementId) { 
    stringArgs.put(elementId, new StringArgumentMarshaler());
} 
// ...
private void setStringArg(char argChar) throws ArgsException {
    currentArgument++;
    try {
        stringArgs.get(argChar).setString(args[currentArgument]); 
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_STRING; 
        throw new ArgsException();
    } 
}
// ...
public String getString(char arg) {
    Args.ArgumentMarshaler am = stringArgs.get(arg);
    return am == null ? "" : am.getString(); 
}
// ...
private class ArgumentMarshaler {
    private boolean booleanValue = false;
    private String stringValue;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }

    public void setString(String s) {
        stringValue = s;
    }

    public String getString() {
        return stringValue == null ? "" : stringValue;
    }
}
```

각 인수 유형을 처리하는 코드를 모두 `ArgumentMarshaler` 클래스에 넣고 `ArgumentMarshaler` 파생 클래스를 만들어 코드를 분리할 작정이다. 그러면 프로그램 구조를 조금씩 변경하는
동안에도 시스템의 정상 동작을 유지하기 쉬워진다. 

다음으로 `int` 인수를 `ArgumentMarshaler`로 옮겼다. 

```java
private Map<Character, ArgumentMarshaler> intArgs = 
        new HashMap<Character, ArgumentMarshaler>();
// ...
private void parseIntegerSchemaElement(char elementId) { 
    intArgs.put(elementId, new IntegerArgumentMarshaler());
}
// ...
private void setIntArg(char argChar) throws ArgsException { 
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        intArgs.get(argChar).setInteger(Integer.parseInt(parameter));
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_INTEGER; 
        throw new ArgsException();
    } catch (NumberFormatException e) { 
        valid = false;
        errorArgumentId = argChar; 
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER; 
        throw new ArgsException();
    } 
}
// ...
public int getInt(char arg) {
    Args.ArgumentMarshaler am = intArgs.get(arg);
    return am == null ? 0 : am.getInteger(); 
}
// ...
private class ArgumentMarshaler {
    private boolean booleanValue = false;
    private String stringValue;
    private int integerValue;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }

    public void setString(String s) {
        stringValue = s;
    }

    public String getString() {
        return stringValue == null ? "" : stringValue;
    }

    public void setInteger(int i) {
        integerValue = i;
    }

    public int getInteger() {
        return integerValue;
    }
}
```

이제 모든 논리를 `ArgumentMarshaler`로 옮겼으니 파생 클래스를 만들어 분산할 차례다. `setBoolean` 함수를 `BooleanArgumentMarshaler`로 옮긴 후 함수가 올바르게 호출되는지 확인한다. 
우선 `ArgumentMarshaler` 클래스에 추상 메서드 `set`을 만들었다.

```java

private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;
    private String stringValue;
    private int integerValue;

    public void setBoolean(boolean value) {
        booleanValue = value;
    }

    public boolean getBoolean() {
        return booleanValue;
    }

    public void setString(String s) {
        stringValue = s;
    }

    public String getString() {
        return stringValue == null ? "" : stringValue;
    }

    public void setInteger(int i) {
        integerValue = i;
    }

    public int getInteger() {
        return integerValue;
    }

    public abstract void set(String s);
}
```

그리고 `BooleanArgumentMarshaller` 클래스에 `set` 메서드를 구현했다.

```java
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }
}
```

마지막으로 `setBoolean` 호출을 `set` 호출로 변경했다.

```java
private void setBooleanArg(char argChar, boolean value) { 
    booleanArgs.get(argChar).set("true");
}
```

이제 `set` 기능을 `BooleanArgumentMarshaler`로 옮겼으므로 `ArgumentMarshaler`에서 `setBoolean` 메서드를 제거한다. 

추상 `set` 함수는 `String` 인수를 받아들이나 `BooleanArgumentMarshaler`는 인수를 사용하지 않는다. `set` 함수에 인수를 정의한 이유는 `StringArgumentMarshaler`와
`IntegerArgumentMarshaler`에서 필요하기 때문이다.

다음으로 `get` 메서드를 `BooleanArgumentMarshaler`로 옮긴다. `get` 함수는 반환 타입이 `Object`여야 한다.

```java
public boolean getBoolean(char arg) { 
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean)am.get();
}
```

순전히 위 코드를 컴파일할 목적으로 `ArgumentMarshaler`에 `get` 함수를 추가했다.

```java
private abstract class ArgumentMarshaler {
    // ...
    public Object get() {
        return null;
    }
}
```

`ArgumentMarshaler`의 `get`을 추상 메서드로 만든 후 `BooleanArgumentMarshaler`에 `get`을 구현했다.

```java
private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;

    // ...
    public abstract Object get();
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }

    public Object get() {
        return booleanValue;
    }
}
```

`ArgumentMarshaler`의 `getBoolean` 함수를 제거한 후 `protected` 변수인 `booleanValue`를 `BooleanArgumentMarshaler`로 옮기고 `private`로 선언한다. `String` 타입도 동일한
방식으로 변경했다.

```java
private void setStringArg(char argChar) throws ArgsException { 
    currentArgument++;
    try {
        stringArgs.get(argChar).set(args[currentArgument]); 
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_STRING; 
        throw new ArgsException();
    }
}
// ...
public String getString(char arg) {
    Args.ArgumentMarshaler am = stringArgs.get(arg);
    return am == null ? "" : (String) am.get(); 
}
// ...
private abstract class ArgumentMarshaler {
    private int integerValue;

    public void setInteger(int i) {
        integerValue = i;
    }

    public int getInteger() {
        return integerValue;
    }

    public abstract void set(String s);

    public abstract Object get();
}


private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;

    public void set(String s) {
        booleanValue = true;
    }

    public Object get() {
        return booleanValue;
    }
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";

    public void set(String s) {
        stringValue = s;
    }

    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
    }

    public Object get() {
        return null;
    }
}
```

마지막으로 `Integer` 인수 타입에도 같은 과정을 반복했다. `Integer` 타입은 구문을 분석해야 하므로 조금 더 복잡하지만, `NumberFormatException`이라는 개념 전체가 `IntegerArgumentMarshaler`
에 숨겨지므로 결과는 더 좋았다.

```java
private boolean isIntArg(char argChar) {
    return intArgs.containsKey(argChar);
}

private void setIntArg(char argChar) throws ArgsException { 
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        intArgs.get(argChar).set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar; 
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw e;
    } 
}
// ...
private void setBooleanArg(char argChar) {
    try {
        booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {
    }
}
// ...
public int getInt(char arg) { 
    Args.ArgumentMarshaler am = intArgs.get(arg); 
    return am == null ? 0 : (Integer) am.get();
}
// ...
private abstract class ArgumentMarshaler {
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
}
// ...
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }

    public Object get() {
        return intValue;
    }
}
```

처음에 나오는 (인수 타입마다 따로 만든) 맵 세 개를 없앴다. 그러나 시스템이 깨지기 때문에 맵을 그냥 없애지는 못했다. 대신 `ArgumentMarshaler`로 맵을 만들어 원래 맵을 교체라고 관련 메서드를 변경했다.

```java
public class Args {
    // ...
    private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> intArgs = new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();

    // ...
    private void parseBooleanSchemaElement(char elementId) {
        ArgumentMarshaler m = new BooleanArgumentMarshaler();
        booleanArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }

    private void parseIntegerSchemaElement(char elementId) {
        ArgumentMarshaler m = new IntegerArgumentMarshaler();
        intArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }

    private void parseStringSchemaElement(char elementId) {
        ArgumentMarshaler m = new StringArgumentMarshaler();
        stringArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }
    // ...
}
```

다음으로 `isBooleanArg`를 다음과 같이 변경했다.

```java
private boolean isBooleanArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof BooleanArgumentMarshaler;
}
```

`isIntArg`와 `isStringArg`도 똑같이 변경했다.

```java
private boolean isIntArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof IntegerArgumentMarshaler;
}

private boolean isStringArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof StringArgumentMarshaler;
}
```

그리고 `marshalers.get`을 호출하는 코드를 모두 제거했다. 

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar); 
    if (isBooleanArg(m))
        setBooleanArg(argChar); 
    else if (isStringArg(m)) 
        setStringArg(argChar);
    else if (isIntArg(m)) 
        setIntArg(argChar);
    else 
        return false;
    return true; 
}

private boolean isIntArg(ArgumentMarshaler m) { 
    return m instanceof IntegerArgumentMarshaler;
}

private boolean isStringArg(ArgumentMarshaler m) { 
    return m instanceof StringArgumentMarshaler;
}

private boolean isBooleanArg(ArgumentMarshaler m) { 
    return m instanceof BooleanArgumentMarshaler;
}
```

`isxxxArg` 메서드도 별도로 선언할 이유가 없으므로 인라인 코드로 만들었다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(argChar);
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar); 
    else 
        return false;
    return true;
}
```

다음으로 `set` 함수에서 기존 `HashMap`을 `marshalers` `HashMap`으로 교체하기 시작했다. 먼저 `boolean` 인수부터 시작했다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m);
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar); 
    else 
        return false;
    return true;
}
// ...
private void setBooleanArg(ArgumentMarshaler m) { 
    try {
        m.set("true"); // was: booleanArgs.get(argChar).set("true"); 
    } catch (ArgsException e) { 
    }
}
```

`String`과 `Integer`도 똑같이 변경했다. 이로써 예외 처리 코드를 `setArgument` 함수에 넣을 수 있었다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    try {
        if (m instanceof BooleanArgumentMarshaler) 
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler) 
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler) 
            setIntArg(m);
        else 
            return false;
    } catch (ArgsException e) {
        valid = false; errorArgumentId = argChar; 
        throw e;
    }
    return true; 
}

private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        m.set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();     
    } catch (ArgsException e) {
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER; 
        throw e;
    } 
}

private void setStringArg(ArgumentMarshaler m) throws ArgsException { 
    currentArgument++;
    try {
        m.set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException(); 
    }
}
```

이제 원래 맵 세 개를 제거해도 괜찮은 시점에 가까워졌다. `getBoolean` 함수를 다음과 같이 변경했다.

```java
public boolean getBoolean(char arg) { 
    Args.ArgumentMarshaler am = marshalers.get(arg);
    boolean b = false;
    try {
        b = am != null && (Boolean) am.get(); 
    } catch (ClassCastException e) {
        b = false; 
    }
    return b;
}
```

인수 테스트를 FitNess에서 구현했기 때문에 `ClassCastException`를 사용했다. 그런데 FitNess 테스트로는 `boolean`이 아닌 인수로 `getBoolean`을 호출하면 무조건 `false`를 반환했다. 
유닛 테스트는 그렇지 않았다. 

이제 `boolean` 맵을 사용하는 코드를 제거할 수 있게 되었다.

```java
private void parseBooleanSchemaElement(char elementId) { 
    ArgumentMarshaler m = new BooleanArgumentMarshaler(); 
    // removed: booleanArgs.put(elementId, m); 
    marshalers.put(elementId, m);
}
```

이로써 `boolean` 맵도 제거가 가능하다.

```java
public class Args {
    // ...
    // removed: private Map<Character, ArgumentMarshaler> booleanArgs = new HashMap<Character, ArgumentMarshaler>();
    // removed: private Map<Character, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> intArgs = new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();
    // ...
}
```

다음으로 `String`과 `Integer` 인수도 `boolean`과 같이 변경하고 맵을 제거했다.

```java
private void parseBooleanSchemaElement(char elementId) {
    marshalers.put(elementId, new BooleanArgumentMarshaler());
}

private void parseIntegerSchemaElement(char elementId) { 
    marshalers.put(elementId, new IntegerArgumentMarshaler());
}

private void parseStringSchemaElement(char elementId) { 
    marshalers.put(elementId, new StringArgumentMarshaler());
}
// ...
public String getString(char arg) { 
    Args.ArgumentMarshaler am = marshalers.get(arg); 
    try {
        return am == null ? "" : (String) am.get();
    } catch (ClassCastException e) {
        return"";
    }
}

public int getInt(char arg) { 
    Args.ArgumentMarshaler am = marshalers.get(arg); 
    try {
        return am == null ? 0 : (Integer) am.get();
    } catch (Exception e) { 
        return 0;
    }
}
// ...
public class Args {
    // ...
    // removed: private Map<Character, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
    // removed: private Map<Character, ArgumentMarshaler> intArgs = new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers = new HashMap<Character, ArgumentMarshaler>();
}
// ...
```

다음으로 거의 사용하지 않는 `parse` 메서드 세 개를 인라인 코드로 만들었다.

```java
private void parseSchemaElement(String element) throws ParseException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail))
        marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (isStringSchemaElement(elementTail))
        marshalers.put(elementId, new StringArgumentMarshaler());
    else if (isIntegerSchemaElement(elementTail)) {
        marshalers.put(elementId, new IntegerArgumentMarshaler());
    } else {
        throw new ParseException(String.format(
                "Argument: %c has invalid format: %s.", elementId, elementTail), 0); 
    }
}
```

이제 전체 그림을 한 번 더 확인할 단계이다. 다음은 첫 번째 리팩토링을 끝낸 버전이다.

```java
package com.objectmentor.utilities.getopts;

import java.text.ParseException; 
import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, ArgumentMarshaler> marshalers =
            new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private int currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;

    private enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT
    }

    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }

    private boolean parse() throws ParseException {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }

    private boolean parseSchema() throws ParseException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                String trimmedElement = element.trim(); 
                parseSchemaElement(trimmedElement);
            }
        }
        return true;
    }
    
    private void parseSchemaElement(String element) throws ParseException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (isBooleanSchemaElement(elementTail)) 
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (isStringSchemaElement(elementTail)) 
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (isIntegerSchemaElement(elementTail)) {
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        } else {
            throw new ParseException(String.format(
                    "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
        }
    }

    private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
            throw new ParseException(
                    "Bad character:" + elementId + "in Args format: " + schema, 0);
        }
    }

    private boolean isStringSchemaElement(String elementTail) {
        return elementTail.equals("*");
    }

    private boolean isBooleanSchemaElement(String elementTail) {
        return elementTail.length() == 0;
    }

    private boolean isIntegerSchemaElement(String elementTail) {
        return elementTail.equals("#");
    }

    private boolean parseArguments() throws ArgsException {
        for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
            String arg = args[currentArgument];
            parseArgument(arg);
        }
        return true;
    }

    private void parseArgument(String arg) throws ArgsException {
        if (arg.startsWith("-"))
            parseElements(arg);
    }

    private void parseElements(String arg) throws ArgsException {
        for (int i = 1; i < arg.length(); i++)
            parseElement(arg.charAt(i));
    }

    private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            unexpectedArguments.add(argChar);
            errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
            valid = false;
        }
    }

    private boolean setArgument(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        try {
            if (m instanceof BooleanArgumentMarshaler) 
                setBooleanArg(m);
            else if (m instanceof StringArgumentMarshaler)
                setStringArg(m);
            else if (m instanceof IntegerArgumentMarshaler) 
                setIntArg(m);
            else
                return false;
        } catch (ArgsException e) {
            valid = false;
            errorArgumentId = argChar;
            throw e;
        }
        return true;
    }

    private void setIntArg(ArgumentMarshaler m) throws ArgsException {
        currentArgument++;
        String parameter = null;
        try {
            parameter = args[currentArgument];
            m.set(parameter);
        } catch (ArrayIndexOutOfBoundsException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }

    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
        currentArgument++;
        try {
            m.set(args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }

    private void setBooleanArg(ArgumentMarshaler m) {
        try {
            m.set("true");
        } catch (ArgsException e) {
        }
    }

    public int cardinality() {
        return argsFound.size();
    }

    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }

    public String errorMessage() throws Exception {
        switch (errorCode) {
            case OK:
                throw new Exception("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
                return unexpectedArgumentMessage();
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.",
                        errorArgumentId);
            case INVALID_INTEGER:
                return String.format("Argument -%c expects an integer but was '%s'.",
                        errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format("Could not find integer parameter for -%c.",
                        errorArgumentId);
        }
        return "";
    }

    private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Argument(s) -");
        for (char c : unexpectedArguments) {
            message.append(c);
        }
        message.append(" unexpected.");
        return message.toString();
    }

    public boolean getBoolean(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        boolean b = false;
        try {
            b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
            b = false;
        }
        return b;
    }

    public String getString(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
            return "";
        }
    }

    public int getInt(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Integer) am.get();
        } catch (Exception e) {
            return 0;
        }
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }

    public boolean isValid() {
        return valid;
    }

    private class ArgsException extends Exception {
    }

    private abstract class ArgumentMarshaler {
        public abstract void set(String s) throws ArgsException;

        public abstract Object get();
    }

    private class BooleanArgumentMarshaler extends ArgumentMarshaler {
        private boolean booleanValue = false;

        public void set(String s) {
            booleanValue = true;
        }

        public Object get() {
            return booleanValue;
        }
    }

    private class StringArgumentMarshaler extends ArgumentMarshaler {
        private String stringValue = "";

        public void set(String s) {
            stringValue = s;
        }

        public Object get() {
            return stringValue;
        }
    }

    private class IntegerArgumentMarshaler extends ArgumentMarshaler {
        private int intValue = 0;

        public void set(String s) throws ArgsException {
            try {
                intValue = Integer.parseInt(s);
            } catch (NumberFormatException e) {
                throw new ArgsException();
            }
        }

        public Object get() {
            return intValue;
        }
    }
}
```

열심히 고쳤지만 결과는 다소 실망스럽다. 구조만이 조금 나아졌을 뿐이다. 처음에 나오는 변수는 그대로 남아있으며, `setArgument`에서 타입을 일일이 확인하는 코드도 그대로이다. 게다가 `set` 함수는 흉하다. 
오류 처리 코드도 마찬가지다.

`setArgument` 함수에서 타입을 일일이 확인하는 코드를 제거하고 싶다. 즉, `ArgumentMarshaler.set`만 호출하면 충분하도록 만들고 싶다. `setIntArg`, `setStringArg`, `setBooleanArg`를 
해당 `ArgumentMarshaler` 파생 클래스로 내린다는 의미이다. 하지만 `setIntArg`를 자세히 살펴보면 `args`와 `currentArgument`라는 인스턴스 변수를 사용한다. 즉, `IntArgumentMarshaler`
로 내리려면 `args`와 `currentArgument`를 인수로 넘겨야 한다는 의미이다. 그러면 코드가 지저분해진다. 

이를 해결하기 위해 `args` 배열을 리스트로 변환한 후 `Iterator`를 `set` 함수로 전달하면 된다. 다음은 변경한 코드이다.

```java
public class Args {
    private String schema;
    // removed: private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, ArgumentMarshaler> marshalers =
            new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private Iterator<String> currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    private List<String> argsList;

    private enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT
    }

    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        valid = parse();
    }

    private boolean parse() throws ParseException {
        if (schema.length() == 0 && argsList.size() == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }
    // ...
    private boolean parseArguments() throws ArgsException {
        for (currentArgument = argsList.iterator(); currentArgument.hasNext(); ) {
            String arg = currentArgument.next();
            parseArgument(arg);
        }
        return true; 
    }
    // ...
    private void setIntArg(ArgumentMarshaler m) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            m.set(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }

    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
        try {
            m.set(currentArgument.next());
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }
}
```

이제 `set` 함수를 적절한 파생 클래스로 내려도 괜찮아졌다. 우선 `setArgument` 클래스를 다음과 같이 변경한다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
        // removed: else 
            // removed: return false;
    } catch (ArgsException e) { 
        valid = false;
        errorArgumentId = argChar; 
        throw e;
    }
    return true; 
}
```

위 변경은 중요하다. `if-else`가 연쇄적으로 이어지는 구문을 완전히 제거하기 위해 오류 코드를 꺼냈다. 

이제 `set` 함수를 옮길 차례다. `setBooleanArg` 함수를 먼저 고친다. `setBooleanArg` 함수는 모든 책임을 단순히 `BooleanArgumentMarshaler`로 전가한다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false; 
    try {
        if (m instanceof BooleanArgumentMarshaler) 
            setBooleanArg(m, currentArgument);
        else if (m instanceof StringArgumentMarshaler) 
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
    } catch (ArgsException e) { 
        valid = false; 
        errorArgumentId = argChar; 
        throw e;
    }
    return true; 
}
// ...
private void setBooleanArg(ArgumentMarshaler m, 
        Iterator<String> currentArgument) 
        throws ArgsException {
    // removed: try {
        m.set("true");
    // removed: } catch(ArgsException e) {
    // removed: }
} 
```

`setBooleanArg`는 `iterator`가 필요 없다. 그러나 `setIntArg`, `setStringArg`에서 필요하다. 또한 `setBooleanArg`, `setIntArg`, `setStringArg` 함수 모두 `ArgumentMarshaler`의
추상 메서드로 호출해야 하기 위해 `iterator`를 전달한다.

```java
private abstract class ArgumentMarshaler {
    public abstract void set(Iterator<String> currentArgument) throws ArgsException;
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
}
```

각 파생 클래스에도 `set` 메서드를 추가한다.

```java
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        booleanValue = true;
    }

    public void set(String s) {
        // removed: booleanValue = true;
    }

    public Object get() {
        return booleanValue;
    }
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator<String> currentArgument) throws ArgsException {
    }

    public void set(String s) {
        stringValue = s;
    }

    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
    }

    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }

    public Object get() {
        return intValue;
    }
}
```

이제 `setBooleanArg`를 제거해도 된다.

```java

private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            m.set(currentArgument);
        else if (m instanceof StringArgumentMarshaler) 
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler) 
            setIntArg(m);
    } catch (ArgsException e) {
        valid = false; 
        errorArgumentId = argChar;
        throw e;
    }
    return true; 
}
```

이제 `set` 함수는 `BooleanArgumentMarshaler`에 속한다. `String`과 `Integer` 인수도 똑같이 변경한다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler) 
            m.set(currentArgument);
        else if (m instanceof StringArgumentMarshaler)
            m.set(currentArgument);
        else if (m instanceof IntegerArgumentMarshaler)
            m.set(currentArgument);
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true; 
}
// ...
private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }

    public void set(String s) {
    }

    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            set(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }

    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }

    public Object get() {
        return intValue;
    }
}
```

이제 타입을 일일이 확인하던 코드를 제거한다.

```java
private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false; 
    try {
        m.set(currentArgument);
        return true;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar; 
        throw e;
    } 
}
```

다음으로 `IntegerArgumentMarshaler`에서 몇 가지 허술한 코드를 고쳐 정리한다.

```java
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw new ArgsException();
        }
    }

    public Object get() {
        return intValue;
    }
}
```

또한 `ArgumentMarshaler`를 인터페이스로 변환한다.

```java
private interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;

    Object get();
}
```

이제 구조에 새로운 인수 타입을 추가하기 쉬워졌다. 변경할 코드는 아주 적으며 나머지 시스템에 영향을 미치지 않는다. 우선 시스템이 `double` 인수 타입을 제대로 받아들이는지 확인할
테스트 케이스부터 추가한다.

```java
public void testSimpleDoublePresent() throws Exception { 
    Args args = new Args("x##", new String[] {"-x","42.3"}); 
    assertTrue(args.isValid());
    assertEquals(1, args.cardinality()); 
    assertTrue(args.has('x'));
    assertEquals(42.3, args.getDouble('x'), .001); 
}
```

이제 스키마 구문분석 코드를 정리하고 `##` 감지 코드를 추가한다. 여기서 `##`은 `double` 인수 타입을 의미한다.

```java
private void parseSchemaElement(String element) throws ParseException {
    char elementId=element.charAt(0);
    String elementTail=element.substring(1);
    validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
        marshalers.put(elementId,new BooleanArgumentMarshaler());
    else if (elementTail.equals("*"))
        marshalers.put(elementId,new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
        marshalers.put(elementId,new IntegerArgumentMarshaler());
    else if (elementTail.equals("##"))
        marshalers.put(elementId,new DoubleArgumentMarshaler());
    else 
        throw new ParseException(String.format(
                "Argument: %c has invalid format: %s.",elementId,elementTail),0);
}
```

다음으로 `DoubleArgumentMarshaler` 클래스를 작성한다.

```java
private class DoubleArgumentMarshaler implements ArgumentMarshaler {
    private double doubleValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            doubleValue = Double.parseDouble(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_DOUBLE;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_DOUBLE;
            throw new ArgsException();
        }
    }

    public Object get() {
        return doubleValue;
    }
}
```

새로운 `ErrorCode`가 필요하다.

```java
private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT, 
    MISSING_DOUBLE, INVALID_DOUBLE
}
```

또한 `getDouble` 함수도 필요하다.

```java
public double getDouble(char arg) { 
    Args.ArgumentMarshaler am = marshalers.get(arg); 
    try {
        return am == null ? 0 : (Double) am.get(); 
    } catch (Exception e) {
        return 0.0; 
    }
}
```

이제 오류 처리가 제대로 도는지 확인한다. 테스트 케이스는 구문분석이 불가능한 문자열을 `##` 인수에 전달해 오류 처리 코드 동작을 확인한다.

```java
public void testInvalidDouble() throws Exception {
Args args = new Args("x##", new String[] {"-x","Forty two"}); assertFalse(args.isValid());
assertEquals(0, args.cardinality()); assertFalse(args.has('x'));
assertEquals(0, args.getInt('x'));
        assertEquals("Argument -x expects a double but was 'Forty two'.",
        args.errorMessage());
        }
// ...
public String errorMessage() throws Exception { 
    switch (errorCode) {
        case OK:
            throw new Exception("TILT: Should not get here.");
        case UNEXPECTED_ARGUMENT:
            return unexpectedArgumentMessage();
        case MISSING_STRING:
            return String.format("Could not find string parameter for -%c.", errorArgumentId);
        case INVALID_INTEGER:
            return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
        case MISSING_INTEGER:
            return String.format("Could not find integer parameter for -%c.", errorArgumentId);
        case INVALID_DOUBLE:
            return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
        case MISSING_DOUBLE:
            return String.format("Could not find double parameter for -%c.",errorArgumentId);
    }
    return ""; 
}
```

다음 테스트 케이스는 `double` 인수를 빠뜨린 경우다.

```java
public void testMissingDouble() throws Exception { 
    Args args = new Args("x##", new String[]{"-x"}); 
    assertFalse(args.isValid());
    assertEquals(0, args.cardinality());
    assertFalse(args.has('x'));
    assertEquals(0.0, args.getDouble('x'), 0.01); 
    assertEquals("Could not find double parameter for -x.", args.errorMessage());
}
```

예외 코드는 아주 흉할뿐더러 사실상 `Args` 클래스에 속하지도 않는다. 게다가 `ParseException`을 던지지만 `ParseException`은 `Args` 클래스에 속하지 않는다. 그러므로 모든 
예외를 하나로 모아 `ArgsException` 클래스를 만든 후 독자 모듈로 옮긴다.

```java
public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;

    public ArgsException() {
    }

    public ArgsException(String message) {
        super(message);
    }

    public enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT, MISSING_DOUBLE, INVALID_DOUBLE
    }
}
// ...
public class Args {
    // ...
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ArgsException.ErrorCode errorCode = ArgsException.ErrorCode.OK;
    private List<String> argsList;

    public Args(String schema, String[] args) throws ArgsException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        valid = parse();
    }

    private boolean parse() throws ArgsException {
        if (schema.length() == 0 && argsList.size() == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }
    
    private boolean parseSchema() throws ArgsException { 
        // ...
    }

    private void parseSchemaElement(String element) throws ArgsException { 
        // ...
        else
            throw new ArgsException(
                    String.format("Argument: %c has invalid format: %s.", elementId, elementTail));
    }

    private void validateSchemaElementId(char elementId) throws ArgsException { 
        if (!Character.isLetter(elementId)) {
            throw new ArgsException(
                    "Bad character:" + elementId + "in Args format: " + schema);
        } 
    }
    // ...
    private void parseElement(char argChar) throws ArgsException { 
        if (setArgument(argChar)) 
            argsFound.add(argChar); 
        else {
            unexpectedArguments.add(argChar);
            errorCode = ArgsException.ErrorCode.UNEXPECTED_ARGUMENT; 
            valid = false;
        } 
    }
    // ...
}
private class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            errorCode = ArgsException.ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }

    public Object get() {
        return stringValue;
    }
}

private class IntegerArgumentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ArgsException.ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ArgsException.ErrorCode.INVALID_INTEGER;
            throw new ArgsException();
        }
    }

    public Object get() {
        return intValue;
    }
}

private class DoubleArgumentMarshaler implements ArgumentMarshaler {
    private double doubleValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            doubleValue = Double.parseDouble(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ArgsException.ErrorCode.MISSING_DOUBLE;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ArgsException.ErrorCode.INVALID_DOUBLE;
            throw new ArgsException();
        }
    }

    public Object get() {
        return doubleValue;
    }
}
```

이제 `Args` 클래스가 던지는 예외는 `ArgsException` 뿐이다. `ArgsException` 모듈은 잡다한 오류 지원 코드가 들어갈 합당하고 당연한 장소다. 덕분이 `Args` 모듈이 깨끗해져 차후 확장도 쉽다. 

이제 `Args` 모듈에서 예외/오류 처리 코드를 완벽하게 분리했다. 

```java
package com.objectmentor.utilities.args;

import junit.framework.TestCase;

public class ArgsTest extends TestCase {
    public void testCreateWithNoSchemaOrArguments() throws Exception {
        Args args = new Args("", new String[0]);
        assertEquals(0, args.cardinality());
    }

    public void testWithNoSchemaButWithOneArgument() throws Exception {
        try {
            new Args("", new String[]{"-x" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }

    public void testWithNoSchemaButWithMultipleArguments() throws Exception {
        try {
            new Args("", new String[]{"-x", "-y" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }

    public void testNonLetterSchema() throws Exception {
        try {
            new Args("*", new String[]{});
            fail("Args constructor should have thrown exception");
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME, e.getErrorCode());
            assertEquals('*', e.getErrorArgumentId());
        }
    }

    public void testInvalidArgumentFormat() throws Exception {
        try {
            new Args("f~", new String[]{});
            fail("Args constructor should have throws exception");
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_FORMAT, e.getErrorCode());
            assertEquals('f', e.getErrorArgumentId());
        }
    }

    public void testSimpleBooleanPresent() throws Exception {
        Args args = new Args("x", new String[]{"-x" });
        assertEquals(1, args.cardinality());
        assertEquals(true, args.getBoolean('x'));
    }

    public void testSimpleStringPresent() throws Exception {
        Args args = new Args("x*", new String[]{"-x", "param" });
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals("param", args.getString('x'));
    }

    public void testMissingStringArgument() throws Exception {
        try {
            new Args("x*", new String[]{"-x" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_STRING, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }

    public void testSpacesInFormat() throws Exception {
        Args args = new Args("x, y", new String[]{"-xy" });
        assertEquals(2, args.cardinality());
        assertTrue(args.has('x'));
        assertTrue(args.has('y'));
    }

    public void testSimpleIntPresent() throws Exception {
        Args args = new Args("x#", new String[]{"-x", "42" });
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals(42, args.getInt('x'));
    }

    public void testInvalidInteger() throws Exception {
        try {
            new Args("x#", new String[]{"-x", "Forty two" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
            assertEquals("Forty two", e.getErrorParameter());
        }
    }

    public void testMissingInteger() throws Exception {
        try {
            new Args("x#", new String[]{"-x" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }

    public void testSimpleDoublePresent() throws Exception {
        Args args = new Args("x##", new String[]{"-x", "42.3" });
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals(42.3, args.getDouble('x'), .001);
    }

    public void testInvalidDouble() throws Exception {
        try {
            new Args("x##", new String[]{"-x", "Forty two" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
            assertEquals("Forty two", e.getErrorParameter());
        }
    }

    public void testMissingDouble() throws Exception {
        try {
            new Args("x##", new String[]{"-x" });
            fail();
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
}
```

```java
public class ArgsExceptionTest extends TestCase {
    public void testUnexpectedMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, 'x', null);
        assertEquals("Argument -x unexpected.", e.errorMessage());
    }

    public void testMissingStringMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_STRING,
                'x', null);
        assertEquals("Could not find string parameter for -x.", e.errorMessage());
    }

    public void testInvalidIntegerMessage() throws Exception {
        ArgsException e =
                new ArgsException(ArgsException.ErrorCode.INVALID_INTEGER, 'x', "Forty two");
        assertEquals("Argument -x expects an integer but was 'Forty two'.", e.errorMessage());
    }

    public void testMissingIntegerMessage() throws Exception {
        ArgsException e =
                new ArgsException(ArgsException.ErrorCode.MISSING_INTEGER, 'x', null);
        assertEquals("Could not find integer parameter for -x.", e.errorMessage());
    }

    public void testInvalidDoubleMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.INVALID_DOUBLE,
                'x', "Forty two");
        assertEquals("Argument -x expects a double but was 'Forty two'.",
                e.errorMessage());
    }

    public void testMissingDoubleMessage() throws Exception {
        ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_DOUBLE,
                'x', null);
        assertEquals("Could not find double parameter for -x.", e.errorMessage());
    }
}
```

```java
public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;

    public ArgsException() {
    }

    public ArgsException(String message) {
        super(message);
    }

    public ArgsException(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }

    public ArgsException(ErrorCode errorCode, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
    }

    public ArgsException(ErrorCode errorCode, char errorArgumentId, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
        this.errorArgumentId = errorArgumentId;
    }

    public char getErrorArgumentId() {
        return errorArgumentId;
    }

    public void setErrorArgumentId(char errorArgumentId) {
        this.errorArgumentId = errorArgumentId;
    }

    public String getErrorParameter() {
        return errorParameter;
    }

    public void setErrorParameter(String errorParameter) {
        this.errorParameter = errorParameter;
    }

    public ErrorCode getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }

    public String errorMessage() throws Exception {
        switch (errorCode) {
            case OK:
                throw new Exception("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
                return String.format("Argument -%c unexpected.", errorArgumentId);
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.",
                        errorArgumentId);
            case INVALID_INTEGER:
                return String.format("Argument -%c expects an integer but was '%s'.",
                        errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format("Could not find integer parameter for -%c.",
                        errorArgumentId);
            case INVALID_DOUBLE:
                return String.format("Argument -%c expects a double but was '%s'.",
                        errorArgumentId, errorParameter);
            case MISSING_DOUBLE:
                return String.format("Could not find double parameter for -%c.",
                        errorArgumentId);
        }
        return "";
    }

    public enum ErrorCode {
        OK, INVALID_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME, MISSING_STRING,
        MISSING_INTEGER, INVALID_INTEGER,
        MISSING_DOUBLE, INVALID_DOUBLE
    }
}
```

```java
public class Args {
    private String schema;
    private Map<Character, ArgumentMarshaler> marshalers =
            new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private Iterator<String> currentArgument;
    private List<String> argsList;

    public Args(String schema, String[] args) throws ArgsException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        parse();
    }

    private void parse() throws ArgsException {
        parseSchema();
        parseArguments();
    }

    private boolean parseSchema() throws ArgsException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                parseSchemaElement(element.trim());
            }
        }
        return true;
    }

    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (elementTail.length() == 0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (elementTail.equals("*")) marshalers.put(elementId, new StringArgumentMarshaler());
        else if (elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if (elementTail.equals("##")) marshalers.put(elementId, new DoubleArgumentMarshaler());
        else
            throw new ArgsException(ArgsException.ErrorCode.INVALID_FORMAT,
                    elementId, elementTail);
    }

    private void validateSchemaElementId(char elementId) throws ArgsException {
        if (!Character.isLetter(elementId)) {
            throw new ArgsException(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME, elementId, null);
        }
    }

    private void parseArguments() throws ArgsException {
        for (currentArgument = argsList.iterator(); currentArgument.hasNext(); ) {
            String arg = currentArgument.next();
            parseArgument(arg);
        }
    }

    private void parseArgument(String arg) throws ArgsException {
        if (arg.startsWith("-"))
            parseElements(arg);
    }

    private void parseElements(String arg) throws ArgsException {
        for (int i = 1; i < arg.length(); i++)
            parseElement(arg.charAt(i));
    }

    private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            throw new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, argChar, null);
        }
    }

    private boolean setArgument(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null)
            return false;
        try {
            m.set(currentArgument);
            return true;
        } catch (ArgsException e) {
            e.setErrorArgumentId(argChar);
            throw e;
        }
    }

    public int cardinality() {
        return argsFound.size();
    }

    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }

    public boolean getBoolean(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        boolean b = false;
        try {
            b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
            b = false;
        }
        return b;
    }

    public String getString(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
            return "";
        }
    }

    public int getInt(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Integer) am.get();
        } catch (Exception e) {
            return 0;
        }
    }

    public double getDouble(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Double) am.get();
        } catch (Exception e) {
            return 0.0;
        }
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }
}
```

소프트웨어 설계는 분할만 잘해도 품질이 크게 높아진다. 적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다. 관심사를 분리하면 코드를 이해하고 보수하기 훨씬 더 쉬워진다.

주목할만한 코드는 `ArgsException`의 `errorMessage` 메서드다. 이 메서드는 명백히 SRP 위반이다. `Args` 클래스가 오류 메시지 형식까지 책임졌기 때문이다. `Args` 클래스는
인수를 처리하는 클래스지 오류 메시지 형식을 처리하는 클래스가 아니기 때문이다. 하지만 `ArgsException` 클래스가 오류 메시지 형식을 처리해야 할까? 이는 절충안이다. `ArgsException`에게
맡겨서 안 되는 생각하는 독자라면 새로운 클래스가 필요하다. 하지만 미리 깔끔하게 만들어진 오류 메시지로 얻는 장점은 무시하지 못한다. 

## 결론

코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다.