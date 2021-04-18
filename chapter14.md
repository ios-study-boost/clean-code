# 14장. 점진적인 개선

이 장은 점진적인 개선을 보여주는 사례 연구다.

## 규칙
- [Args 구현](#args-구현)
  - [어떻게 짰느냐고?](#어떻게-짰느냐고)
- [Args 1차 초안](#args-1차-초안)
  - [그래서 멈췄다](#그래서-멈췄다)
  - [점진적으로 개선하다](#점진적으로-개선하다)
- [String 인수](#string-인수)
- [결론](#결론)


##### 목록 14-1 간단한 Args 사용법
```java
public static void main(String[] args) {
    try {
        Args = arg = new Args("l,p#,d*", args);
        boolean logging = arg.getBoolean('l');
        int port = arg.getInt('p');
        String directory = arg.getString('d');
        executeApplication(logging, port, directory);
    } catch (ArgsException e) {
        System.out.printf("Argument error: %s\n", e.errorMessage());
    }
}
```

## Args 구현

아주 주의 깊게 읽어보기 바란다. 스타일과 구조에 신경을 썼으므로 흉내 낼 가치가 있다고 믿는다.

##### 목록 14-2 Args.java
```java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
import java.util.*;

public class Args {
    private Map < Character, ArgumentMarshaler > marshalers;
    private Set < Character > argsFound;
    private ListIterator < String > currentArgument;

    public Args(String schema, String[] args) throws ArgsException {
        marshalers = new HashMap < Character, ArgumentMarshaler > ();
        argsFound = new HashSet < Character > ();

        parseSchema(schema);
        parseArgumentStrings(Arrays.asList(args));
    }

    private void parseSchema(String schema) throws ArgsException {
        for (String element: schema.split(","))
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

    private void parseArgumentStrings(List < String > argsList) throws ArgsException {
        for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
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

여기저기 뒤적일 필요 없이 위에서 아래로 코드가 읽힌다는 사실에 주목한다. 코드를 주의 깊게 읽었다면 `ArgumentMarshaler` 인터페이스가 무엇이며 파생 클래스가 무슨 기능을 하는지 이해하리라.

##### 목록 14-3 ArgumentMarshaler.java
```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator < String > currentArgument) throws ArgsException {
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

##### 목록 14-6 IntegerArgumentMarshaler.java
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

나머지 `DoubleArgumentMarshaler`와 `StringArrayArgumentMarshaler`는 다른 파생 클래스와 똑같은 패턴이므로 코드를 생략한다.

##### 목록 14-7 ArgsException.java
```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;

    public ArgsException() {}

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

    public String errorMessage() {
        switch (errorCode) {
        case OK:
            return "TILT: Should not get here.";
        case UNEXPECTED_ARGUMENT:
            return String.format("Argument -%c unexpected.", errorArgumentId);
        case MISSING_STRING:
            return String.format("Could not find string parameter for -%c.", errorArgumentId);
        case INVALID_INTEGER:
            return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
        case MISSING_INTEGER:
            return String.format("Could not find integer parameter for -%c.", errorArgumentId);
        case INVALID_DOUBLE:
            return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
        case MISSING_DOUBLE:
            return String.format("Could not find double parameter for -%c.", errorArgumentId);
        case INVALID_ARGUMENT_NAME:
            return String.format("'%c' is not a valid argument name.", errorArgumentId);
        case INVALID_ARGUMENT_FORMAT:
            return String.format("'%s' is not a valid argument format.", errorParameter);
        }
        return "";
    }

    public enum ErrorCode {
        OK,
        INVALID_ARGUMENT_FORMAT,
        UNEXPECTED_ARGUMENT,
        INVALID_ARGUMENT_NAME,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        MISSING_DOUBLE,
        INVALID_DOUBLE
    }
}
```

이처럼 단순한 개념을 구현하는데 코드가 너무 많이 필요해 놀랄지도 모르겠다. 한 가지 이유는 위 코드가 정적 타입 언어인 자바로 작성되어 있기 때문이다. 루비, 파이썬 등과 같은 언어를 사용했다면 프로그램이 훨씬 작아졌으리라. [Args 루비 버전](https://github.com/unclebob/rubyargs/tree/master)

이름을 붙인 방법, 함수 크기, 코드 형식에 주목하라. 여기저기 자잘한 구조나 스타일이 거슬릴지 모르겠지만 전반적으로 깔끔한 구조에 잘 짜인 프로그램으로 여겨주면 좋겠다. 

예를 들어, 날짜 인수나 복소수 인수 등 새로운 인수 유형을 추가하는 방법이 명백하다. 고칠 코드도 별로 없다. 

### 어떻게 짰느냐고?

일단 진정하기 바란다. 나는 위 프로그램을 처음부터 저렇게 구현하지 않았다. 더욱 중요하게는 여러분이 깨끗하고 우아한 프로그램을 한 방에 뚝딱 내놓으리라 기대하지 않는다.

프로그래밍은 과학보다 공예<sup>craft</sup>에 가깝다. 깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 **뒤에 정리해야 한다**는 의미다.

초안을 만들고, 그 초안을 고쳐 2차 초안을 만들고, 계속 고쳐 최종안을 만들어야 한다. 깔끔한 프로그램을 내놓으려면 단계적으로 개선해야 한다.

대다수 신참 프로그래머는 이 충고를 충실히 따르지 않는다. 그들은 무조건 돌아가는 프로그램을 목표로 잡는다. 일단 프로그램이 '돌아가면' 다음 업무로 넘어간다. '돌아가는' 프로그램은 그 상태가 어떻든 그대로 버려둔다. 이런 행동은 전문가로서 자살 행위와 같다.

## Args 1차 초안

내가 맨 처음 짰던 `Args` 클래스다. 코드는 '돌아가지만' 엉망이다.

##### 목록 14-8 Args.java(1차 초안)
```java
import java.text.ParseException;
import java.util.*;

public class Args {
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set < Character > unexpectedArguments = new TreeSet < Character > ();
    private Map < Character, Boolean > booleanArgs = new HashMap < Character, Boolean > ();
    private Map < Character, String > stringArgs = new HashMap < Character, String > ();
    private Map < Character, Integer > intArgs = new HashMap < Character, Integer > ();
    private Set < Character > argsFound = new HashSet < Character > ();
    private int currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;

    private enum ErrorCode {
        OK,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        UNEXPECTED_ARGUMENT
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
        } catch (ArgsException e) {}
        return valid;
    }

    private boolean parseSchema() throws ParseException {
        for (String element: schema.split(",")) {
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
        else if (isIntegerSchemaElement(elementTail))
            parseIntegerSchemaElement(elementId);
        else
            throw new ParseException(String.format("Argument: %c has invalid format: %s.",
                elementId, elementTail), 0);
    }
}

private void validateSchemaElementId(char elementId) throws ParseException {
    if (!Character.isLetter(elementId)) {
        throw new ParseException("Bad character:" + elementId + "in Args format: " + schema, 0);
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
    else
        unexpectedArguments.add(argChar);
    errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
    valid = false;
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
        return String.format("Could not find string parameter for -%c.", errorArgumentId);
    case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
    case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.", errorArgumentId);
    }
    return "";
}

private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s) -");
    for (char c: unexpectedArguments) {
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

private class ArgsException extends Exception {}
}
```

이처럼 지저분한 코드를 보고 처음 든 생각이 "저자가 그냥 내버려두지 않아서 진짜 다행이야!"이기 바란다. 만약 그렇다면 자신이 대충 짜서 남겨둔 코드를 남들이 어떻게 느낄지 생각하기 바란다.

사실 '1차 초안'은 낯 뜨거운 표현이다. 목록 14-8은 명백히 미완성이다. 인스턴스 변수 개수만도 압도적이다. "TILT"와 같은 희한한 문자열, `HashSets`와 `TreeSets`, `try-catch-catch` 블록 등 모두가 지저분한 코드에 기여하는 요인이다.

인수 유형을 단계적으로 추가했고(`String`, `Integer`), 코드는 통제를 벗어나기 시작했다. 코드는 완전히 엉망이 되어버렸다.

### 그래서 멈췄다

추가할 인수 유형이 적어도 두 개는 더 있었는데 그러면 코드가 훨씬 더 나빠지리라는 사실이 자명했다. 계속 밀어붙이면 프로그램은 어떻게든 완성하겠지만 그랬다가는 너무 커서 손대기 어려운 골칫거리가 생겨날 참이었다. 코드 구조를 유지보수하기 좋은 상태로 만들려면 지금이 적기라 판단했다.

그래서 나는 기능을 더 이상 추가하지 않기로 결정하고 리팩터링을 시작했다.

나는 새 인수 유형을 추가하려면 주요 지점 세 곳에다 코드를 추가해야 한다는 사실을 깨달았다.
1. 인수 유형에 해당하는 `HashMap`을 선택하기 위해 스키마 요소의 구문을 분석한다.
2. 명령행 인수에서 인수 유형을 분석해 진짜 유형으로 변환한다.
3. `getXXX` 메서드를 구현해 호출자에게 진짜 유형을 반환한다.

인수 유형은 다양하지만 모두가 유사한 메서드를 제공하므로 클래스 하나가 적합하다 판단했다. 그래서 `ArgumentMarshaler`라는 개념이 탄생했다.

### 점진적으로 개선하다

프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다. 어떤 프로그램은 그저 그런 '개선'에서 결코 회복하지 못한다. '개선' 전과 똑같이 프로그램을 돌리기가 아주 어렵기 때문이다.

그래서 나는 테스트 주도 개발<sup>Test-Driven Development, TDD</sup>이라는 기법을 사용했다. TDD는 언제 어느때라도 시스템이 돌아가야 한다는 원칙을 따른다. 다시 말해, TDD는 시스템을 망가뜨리는 변경을 허용하지 않는다. 변경을 가한 후에도 시스템이 변경 전과 독같이 돌아가야 한다는 말이다.

변경 전후에 시스템이 똑같이 돌아간다는 사실을 확인하려면 언제든 실행이 가능한 자동화된 테스트 슈트가 필요하다. 앞서 `Args` 클래스를 구현하는 도앙ㄴ에 나는 이미 단위 테스트 슈트와 인수 테스트를 만들어 놓았었다.

그래서 나는 시스템에 자잘한 변경을 가하기 시작했다. 코드를 변경할 때마다 시스템 구조는 조금씩 `ArgumentMarshaler` 개념에 가까워졌다. 또한 변경 후에도 시스템은 변경 전과 다름없이 돌아갔다.

## String 인수

`String` 인수를 추가하는 과정은 `boolean` 인수와 매우 유사했다. 별로 놀랄 부분은 없으리라 생각한다. 단, 내가 모든 논리를 (파생 클래스를 만드는 대신) `ArgumentMarshaler` 클래스에 곧바로 넣었다는 사실을 의아하게 여길지도 모르겠다.

```java
private Map<Chracter, ArgumentMarshaler> stringArgs = new HashMap<Character, ArgumentMarshaler>();
```

이번에도 앞서 구현과 마찬가지로 한 번에 하나씩 고치면서 테스트를 계속 돌렸다. 테스트 케이스가 하나라도 실패하면 다음 변경으로 넘어가기 전에 오류를 수정했다. 

지금쯤이면 내 의도를 눈치챘으리라. 일단 각 인수 유형을 처리하는 코드를 모두 `ArgumentMarshaler` 클래스에 넣고 나서 `ArgumentMarshaler` 파생 클래스를 만들어 코드를 분리할 작정이었다. 그러면 프로그램 구조를 조금씩 변경하는 동안에도 시스템의 정상 동작을 유지하기 쉬워지기 때문이다.

리팩터링을 하다보면 코드를 넣었다 뺐다 하는 사례가 아주 흔하다. 단계적으로 조금씩 변경하며 매번 테스트를 돌려야 하므로 코드를 여기저기 옮길 일이 많아진다. 리팩터링은 루빅 큐브 맞추기와 비슷하다. 큰 목표 하나를 이루기 위해 자잘한 단계를 수없이 거친다. 각 단계를 거쳐야 다음 단계가 가능하다.

소프트웨어 설계는 분할만 잘해도 품질이 크게 높아진다. 적절한 장소를 만들어 코드만 분리해도 설계가 좋아진다. 관심사를 분리하면 코드를 이해하고 보수하기 훨씬 더 쉬워진다.

## 결론

그저 돌아가는 코드만으로는 부족하다. 돌아가는 코드가 심하게 망가지는 사례는 흔하다. 단순히 돌아가는 코드에 만족하는 프로그래머는 전문가 정신이 부족하다. 

나쁜 코드보다 더 오랫동안 더 심각하게 개발 프로젝트에 악영향을 미치는 요인도 없다. 나쁜 코드는 썩어 문드러진다. 점점 무게가 늘어나 팀의 발목을 잡는다. 속도가 점점 느려지다 못해 기어가는 팀도 많이 봤다. 너무 서두르다가 이후로 영원히 자신들의 운명을 지배할 악성 코드라는 굴레를 짊어진다.

물론 나쁜 코드도 깨끗한 코드로 개선할 수 있다. 하지만 비용이 엄청나게 많이 든다. 모듈은 서로서로 얽히고설켜 뒤엉키고 숨겨진 의존성이 수도 없이 생긴다. 반면 처음부터 코드를 깨끗하게 유지하기란 상대적으로 쉽다. 

코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다.