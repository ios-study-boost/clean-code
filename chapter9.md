# 9장. 단위 테스트

## 규칙
- [TDD 법칙 세 가지](#tdd-법칙-세-가지)
- [깨끗한 테스트 코드 유지하기](#깨끗한-테스트-코드-유지하기)
  - [테스트는 유연성, 유지보수성, 재사용성을 제공한다](#테스트는-유연성-유지보수성-재사용성을-제공한다)
- [깨끗한 테스트 코드](#깨끗한-테스트-코드)
  - [도메인에 특화된 테스트 언어](#도메인에-특화된-테스트-언어)
  - [이중 표준](#이중-표준)
- [테스트 당 assert 하나](#테스트-당-assert-하나)
  - [테스트 당 개념 하나](#테스트-당-개념-하나)
- [F.I.R.S.T](#first)
- [결론](#결론)

## TDD 법칙 세 가지

- **첫째 법칙**: 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- **둘째 법칙**: 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- **셋째 법칙**: 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

위 세가지 규칙을 따르면서 일하면 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 나온다. 하지만 실제 코드와 맞먹을 정도로 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기

실제 코드가 진화하면 테스트 코드도 변해야 한다. 그런데 테스트 코드가 지저분할수록 변경하기 어려워진다. 실제 코드를 변경해 기존 테스트 케이스가 실패하기 시작하면, 지저분한 코드로 인해, 실패하는 테스트 케이스를 점점 더 통과시키기 어려워진다. 그래서 테스트 코드는 계속해서 늘어나는 부담이 되버린다.

**테스트 코드는 실제 코드 못지 않게 중요하다.** 테스트 코드는 사고와 설계와 주의가 필요하다. 실제 코드 못지 않게 깨끗하게 짜야 한다.

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다

코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목은 바로 **단위 테스트**다. 테스트 케이스가 없다면 모든 변경이 잠정적인 버그다.

테스트 케이스가 있다면 아키텍처가 부실한 코드나 설계가 모호하고 엉망인 코드라도 별다른 우려 없이 변경할 수 있다. 아니, 오히려 안심하고 아키텍처와 설계를 개선할 수 있다.

그러므로 실제 코드를 점검하는 자동화된 단위 테스트 슈트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠다. 테스트는 유연성, 유지보수성, 재사용성을 제공한다. 테스트 케이스가 있으면 **변경**이 쉬워지기 때문이다.

따라서 테스트 코드가 지저분하면 코드를 변경하는 능력이 떨어지며 코드 구조를 개선하는 능력도 떨어진다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만들려면? **가독성**이 필요하다. 테스트 코드에서 가독성을 높이려면? 명료성, 단순성, 풍부한 표현력이 필요하다. 테스트 코드는 최소한의 표현으로 많은 것을 나타내야 한다.

다음 코드를 보자. FitNess에서 가져온 코드다. 아래 테스트 케이스 세 개는 이해하기 어렵기에 개선할 여지가 충분하다.
1. `addPage`와 `assertSubString`을 부르느라 중복되는 코드가 매우 많다. 자질구레한 사항이 너무 많아 테스트 코드의 표현력이 떨어진다.
2. `PathParser` 호출을 살펴보면, `PathParser`는 문자열을 `pagePath` 인스턴스로 변환한다. `pagePath`는 웹 로봇<sup>crawler</sup>이 사용하는 객체다. 이 코드는 테스트와 무관하며 테스트 코드의 의도만 흐린다.
3. `responder` 객체를 생성하는 코드와 `response`를 수집해 변환하는 코드 역시 잡음에 불과하다.
4. `resource`와 인수에서 요청 URL을 만드는 어설픈 코드도 보인다.
5. 읽는 사람을 고려하지 않는다. 

```swift
public func testGetPageHieratchyAsXml() throws -> Void {
    crawler.addPage(root, PathParser.parse("PageOne"))
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"))
    crawler.addPage(root, PathParser.parse("PageTwo"))

    request.setResource("root")
    request.addInput("type", "pages")
    var responder: Responder = SerializedPageResponder()
    var response: SimpleResponse = responder.makeResponse(FitNesseContext(root), request) as? SimpleResponse
    var xml: String = response.getContent()

    assertEquals("text/xml", response.getContentType())
    assertSubString("<name>PageOne</name>", xml)
    assertSubString("<name>PageTwo</name>", xml)
    assertSubString("<name>ChildOne</name>", xml)
}

public func testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws -> Void {
    var pageOne: WikiPage = crawler.addPage(root, PathParser.parse("PageOne"))
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"))
    crawler.addPage(root, PathParser.parse("PageTwo"))

    var data: PageData = pageOne.getData()
    var properties: WikiPageProperties = data.getProperties()
    var symLinks: WikiPageProperty = properties.set(SymbolicPage.PROPERTY_NAME)
    symLinks.set("SymPage", "PageTwo")
    pageOne.commit(data)

    request.setResource("root")
    request.addInput("type", "pages")
    var responder: Responder = SerializedPageResponder()
    var response: SimpleResponse = responder.makeResponse(FitNesseContext(root), request) as? SimpleResponse
    var xml: String = response.getContent()

    assertEquals("text/xml", response.getContentType())
    assertSubString("<name>PageOne</name>", xml)
    assertSubString("<name>PageTwo</name>", xml)
    assertSubString("<name>ChildOne</name>", xml)
    assertNotSubString("SymPage", xml)
}

public func testGetDataAsHtml() throws -> Void {
    crawler.addPage(root, PathParser.parse("PageOne"), "test page")

    request.setResouce("TestPageOne")
    request.addInput("type", "data")
    var responder: Responder = SerializedPageResponder()
    var response: SimpleResponse = responder.makeResponse(FitNesseContext(root), request) as? SimpleResponse
    var xml: String = response.getContent()

    assertEquals("text/xml", response.getContentType())
    assertSubString("test page", xml)
    assertSubString("<Test", xml)
}
```

다음 코드를 살펴보자. 위 코드를 개선한 코드로, 정확히 동일한 테스트를 수행한다. 하지만 좀 더 깨끗하고 좀 더 이해하기 쉽다.

```swift
public func testGetPageHieratchyAsXml() throws -> Void {
    makePages("PageOne", "PageOne.ChildOne", "PageTwo")

    submitRequest("root", "type:pages")

    assertResponseIsXml()
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>")
}

public func testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws -> Void {
    var page: WikiPage = makePage("PageOne")
    makePages("PageOne.ChildOne", "PageTwo")

    addLinkTo(page, "PageTwo", "SymPage")
    
    submitRequest("root", "type:pages")

    assertResponseIsXml()
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>")
    assertResponseDoesNotContain("SymPage")
}

public func testGetDataAsHtml() throws -> Void {
    makePageWithContent("TestPageOne", "test page")

    submitRequest("TestPageOne", "type:data")

    assertResponseIsXml()
    assertResponseContains("test page", "<Test")
}
```

각 테스트는 명확히 세 부분으로 나눠진다.
1. 테스트 자료를 만든다.
2. 테스트 자료를 조작한다.
3. 조작한 결과가 올바른지 확인한다.

잡다하고 세세한 코드를 거의 다 없앴다는 사실에 주목한다. 테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다. 그러므로 코드를 읽는 사람은 온갖 잡다하고 세세한 코드에 헷갈릴 필요 없이 코드가 수행하는 기능을 재빨리 이해한다.

### 도메인에 특화된 테스트 언어

방금 전의 코드는 도메인에 특화된 언어<sup>DSL</sup>로 테스트 코드를 구현하는 기법을 보여준다. 흔히 쓰는 시스템 조작 API를 사용하는 대신 API 위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용하므로 테스트 코드를 짜기도 읽기도 쉬워진다. 이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다. 즉, 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 **언어**다.

숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리팩터링해야 마땅하다.

### 이중 표준

테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 **다르다**. 단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다.

```swift
@Test
public func turnOnLoTempAlarmAtThreashold() throws -> Void {
    hw.setTemp(WAY_TOO_COLD); 
    controller.tic(); 
    assertTrue(hw.heaterState());   
    assertTrue(hw.blowerState()); 
    assertFalse(hw.coolerState()); 
    assertFalse(hw.hiTempAlarm());       
    assertTrue(hw.loTempAlarm());
}
```

위 코드를 읽으면 코드에서 점검하는 상태 이름과 상태 값을 확인하느라 눈길이 이리저리 흩어진다. `heaterState`라는 상태를 보고서는 왼쪽으로 눈길을 돌려 `assertTrue`를 읽는다. 테스트 코드를 읽기가 어렵다.

```swift
@Test
public func turnOnLoTempAlarmAtThreashold() throws -> Void {
    wayTooCold()
    assertEquals("HBchL", hw.getState())
}
```

`tic` 함수는 `wayTooCold`라는 함수를 만들어 숨겼다. `assertEquals`에 들어있는 이상한 문자열에 주목하자. 대문자는 '켜짐<sup>on</sup>'이고 소문자는 '꺼짐<sup>off</sup>'을 뜻한다. 문자는 항상 {heater, blower, cooler, hi-temp-alarm, lo-temp-alarm} 순서다.

비록 위 방식이 [그릇된 정보를 피하라](./chapter2.md#그릇된-정보를-피하라)는 규칙의 위반에 까갑지만 여기서는 적절해보인다. 일단 의미만 안다면 눈길이 문자열을 따라 움직이며 결과를 재빨리 판단한다.

```swift
public func getState() -> String {
    var state = ""
    state += heater ? "H" : "h"
    state += blower ? "B" : "b"
    state += cooler ? "C" : "c"
    state += hiTempAlarm ? "H" : "h"
    state += lotempAlarm ? "L" : "l"
    return state
}
```

코드가 그리 효율적이지 못하다. (자바에서) 효율을 높이려면 `StringBuffer`가 더 적합하다. 이 애플리케이션은 실시간 임베디드 시스템이다. 즉, 컴퓨터 자원과 메모리가 제한적일 가능성이 높다. 하지만 **테스트** 환경은 자원이 제한적일 가능성이 낮다.

이것이 이중 표준의 본질이다. 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제없는 방식이 있다. 대게 메모리나 CPU 효율과 관련 있는 경우다. 코드의 깨끗함과는 **철저히** 무관하다.

## 테스트 당 assert 하나

`JUnit`으로 테스트 코드를 짤 때는 함수마다 `assert` 문을 단 하나만 사용해야 한다고 주장하는 학파가 있다. 가혹한 규칙이라 여길지도 모르지만, `assert` 문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.

[깨끗한 테스트 코드](#깨끗한-테스트-코드)에서 봤던 코드는 어떨까? "출력이 XML이다"라는 `assert` 문과 "특정 문자열을 포함한다"는 `assert` 문을 하나로 병합하는 방식이 불합리해 보인다. 하지만 다음 코드와 같이 테스트를 두 개로 쪼개 각자가 `assert`를 수행하면 된다.

```swift
public func testGetPageHierarchyAsXml() throws -> Void {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo")

    whenRequestIsIssued("root", "type:pages")

    thenResponseShouldBeXML()
}

public func testGetPageHierarchyHasRightTags() throws -> Void {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo")

    whenRequestIsIssued("root", "type:pages")

    thenResponseShouldContain("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>")
}
```

위에서 함수 이름을 바꿔 given-when-then이라는 관례를 사용했다는 사실에 주목한다. 그러면 테스트 코드를 읽기가 쉬워진다. 불행하게도, 위에서 보듯이, 테스트를 분리하면 중복되는 코드가 많아진다.

TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있다. given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스에 두면 된다. 아니면 완전히 독자적인 테스트 클래스를 만들어 `@Before` 함수에 given/when 부분을 넣고 `@Test` 함수에 then 부분을 넣어도 된다. 하지만 모두가 배보다 배꼽이 더 크다. 이것저것 감안해 보면 `assert`문을 여럿 사용하는 편이 좋다고 생각한다.

'단일 assert 문'이라는 규칙이 훌륭한 지침이라 생각한다. 하지만 때로는 주저 없이 함수 하나에 여러 `assert` 문을 넣기도 한다. 단지 `assert` 문 개수는 최대한 줄여야 좋다는 생각이다.

### 테스트 당 개념 하나

어쩌면 "테스트 함수마다 한 개념만 테스트하라"는 규칙이 더 낫겟다. 이것저것 잡다한 개념을 연속으로 테스트하는 김 함수는 피한다.

```swift
// BAD
/**
 * addMonth() 메서드를 테스트하는 장황한 코드
 */
public func testAddMonths() {
    var d1: SerialDate = SerialDate.createInstance(31, 5, 2004);

    var d2: SerialDate = SerialDate.addMonths(1, d1); 
    assertEquals(30, d2.getDayOfMonth()); 
    assertEquals(6, d2.getMonth()); 
    assertEquals(2004, d2.getYYYY());

    var d3: SerialDate = SerialDate.addMonths(2, d1); 
    assertEquals(31, d3.getDayOfMonth()); 
    assertEquals(7, d3.getMonth()); 
    assertEquals(2004, d3.getYYYY());

    var d4: SerialDate = SerialDate.addMonths(1, SerialDate.addMonths(1, d1)); 
    assertEquals(30, d4.getDayOfMonth());
    assertEquals(7, d4.getMonth());
    assertEquals(2004, d4.getYYYY());
}
```

테스트 함수는 각각 다음 기능을 수행한다.
- (5월처럼) 31로 끝나는 달의 마지막 날짜가 주어지는 경우
  1. (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안 된다.
  2. 두 달을 더하면 그리고 두 번째 달이 31로 끝나면 날짜는 31일이 되어야 한다.
- (6월처럼) 30일로 끝나는 달의 마지막 날짜가 주어지는 경우
  1. 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되면 안 된다.

위 코드는 각 절에 `assert` 문이 여럿이라는 사실이 문제가 아니다. 한 테스트 함수에서 여러 개념을 테스트한다는 사실이 문제다.

가장 좋은 규칙은 "개념 당 assert 문 수를 최소로 줄여라"와 "테스트 함수 하나는 개념 하나만 테스트하라"

## F.I.R.S.T

깨끗한 테스트는 다음 다섯 가지 규칙을 따르는데, 각 규칙에서 첫 글자를 따오면 FIRST가 된다.

- **빠르게**<sup>Fast</sup>: 테스트는 빨라야 한다. 테스트는 빨리 돌아야 한다는 말이다. 테스트가 느리면 자주 돌릴 엄두를 못 낸다. 자주 돌리지 않으면 초반에 문제를 찾아내 고치지 못한다. 코드를 마음껏 정리하지도 못한다. 결국 코드 품질이 망가지기 시작한다.
- **독립적으로**<sup>Independent</sup>: 각 테스트는 서로 의존하면 안 된다. 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안 된다. 각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 괜찮아야 한다. 테스트가 서로에게 의존하면 하나가 실패할 때 나머지도 잇달아 실패하므로 원인을 진단하기 어려워지며 후반 테스트가 찾아내야 할 결함이 숨겨진다.
- **반복가능하게**<sup>Repeatable</sup>: 테스트는 어떤 환경에서도 반복 가능해야 한다. 실제 환경, QA 환경, 버스를 타고 집으로 가는 길에 사용하는 (네트워크에 연결되지 않은) 노트북 환경에서도 실행할 수 있어야 한다. 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다. 게다가 환경이 지원되지 않기에 테스트를 수행하지 못하는 상황에 직면한다.
- **자가검증하는**<sup>Self-Validating</sup>: 테스트는 부울<sup>bool</sup> 값으로 결과를 내야 한다. 성공 아니면 실패다. 통과 여부를 알려고 로그 파일을 읽게 만들어서는 안 된다. 통과 여부를 보려고 텍스트 파일 두 개를 수작업으로 비교하게 만들어서도 안 된다. 테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며 지루한 수작업 평가가 필요하게 된다.
- **적시에**<sup>Timely</sup>: 테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다. 실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다. 어떤 실제 코드는 테스트하기 너무 어렵다고 판명날지 모른다. 테스트가 불가능하도록 실제 코드를 설계할지도 모른다.

## 결론

테스트 코드는 실제 코드만큼이나 프로젝트 건강에 중요하다. 어쩌면 실제 코드보다 더 중요할지도 모르겠다. 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다.
- 테스트 코드는 지속적으로 깨끗하게 관리하자. 
- 표현력을 높이고 간결하게 정리하자. 
- 테스트 API를 구현해 도메인 특화 언어<sup>Domain Specific Language, DSL</sup>를 만들자.
- 테스트 코드를 깨끗하게 유지하자.