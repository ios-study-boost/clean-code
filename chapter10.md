# 10장. 클래스

코드의 표현력과 그 코드로 이루어진 함수에 아무리 신경 쓸지라도 좀 더 차원 높은 단계까지 신경 쓰지 않으면 깨끗한 코드를 얻기는 어렵다. 이 장에서는 깨끗한 클래스를 다룬다.

## 규칙
- [클래스 체계](#클래스-체계)
  - [캡슐화](#캡슐화)
- [클래스는 작아야 한다!](#클래스는-작아야-한다)
  - [단일 책임 원칙<sup>Single Responsibility Principle, SRP</sup>](#단일-책임-원칙supsingle-responsibility-principle-srpsup)
  - [응집도<sup>Cohesion</sup>](#응집도supcohesionsup)
    - [응집도를 유지하면 작은 클래스 여럿이 나온다](#응집도를-유지하면-작은-클래스-여럿이-나온다)
- [변경하기 쉬운 클래스](#변경하기-쉬운-클래스)
  - [변경으로부터 격리](#변경으로부터-격리)

## 클래스 체계

클래스를 정의하는 표준 자바 관례에 따르면, 다음과 같은 순서로 나열한다.

1. 변수 목록
   1. 정적<sup>static</sup> 공개<sup>public</sup> 상수
   2. 정적 비공개<sup>private</sup> 변수
   3. 비공개 인스턴스 변수
   4. 공개 변수가 필요한 경우는 거의 없다.
2. 공개 함수
3. 비공개 함수
   - 자신을 호출하는 공개 함수 직후에 넣는다.

추상화 단계가 순차적으로 내려가기 때문에 프로그램은 신문 기사처럼 읽힌다.

### 캡슐화

변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다. 같은 패키지 안에서 테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면 그 함수나 변수를 `protected`로 선언하거나 패키지 전체로 공개한다. 하지만 그 전에 비공개 상태를 유지할 온갖 방법을 강구한다. 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.

## 클래스는 작아야 한다!

클래스를 만들 때 첫 번째 규칙은 크기다. 클래스는 작아야 한다. "얼마나 작아야 하는가?"

함수는 물리적인 행 수로 크기를 측정했다. 클래스는 다른 척도를 사용한다. 클래스가 맡은 **책임**을 센다.

다음 코드는 `SuperDashboard`라는 클래스로, 공개 메서드 수가 대략 70개 정도다. 대다수 개발자는 클래스가 엄청나게 크다는 사실에 동의하리라.

```swift
public class SuperDashboard: JFrame, MetaDataUser {
    public func getCustomizerLanguagePath() -> String
    public func setSystemConfigPath(systemConfigPath: String)
    public func getSystemConfigDocument() -> String
    public func setSystemConfigDocument(systemConfigDocument: String)
    public func getGuruState() -> Bool
    public func getNoviceState() -> Bool
    public func getMajorVersionNumber() -> Int
    public func getMinorVersionNumber() -> Int
    public func getBuildNumber() -> Int
    // 이하 생략
}
```

하지만 만약 `SuperDashboard`가 메서드 몇 개만 포함한다면?

```swift
public class SuperDashboard: JFrame, MetaDataUser {
    public func getLastFocusedComponent() -> Component
    public func setLastFocused(lastFocused: Component)
    public func getMajorVersionNumber() -> Int
    public func getMinorVersionNumber() -> Int
    public func getBuildNumber() -> Int
}
```

메서드 다섯 개 정도면 괜찮다. 안 그런가? 여기서는 아니다. `SuperDashboard`는 메서드 수가 작음에도 불구하고 **책임**이 너무 많다.

클래스 이름은 해당 클래스 책임을 기술해야 한다. 실제로 작명은 클래스 크기를 줄이는 첫 번째 관문이다. 간결한 이름이 떠오르지 않는다면 필경 클래스 크기가 너무 커서 그렇다. 클래스 이름이 모호하다면 필경 클래스 책임이 너무 많아서다. 

- 클래스 이름
  - 해당 클래스 책임을 기술해야 한다. 작명은 클래스 크기를 줄이는 첫 번째 관문이다.
  - 간결한 이름이 떠오르지 않는다면 → 클래스 크기가 너무 커서 그렇다.
  - 클래스 이름이 모호하다면 → 클래스 책임이 너무 많아서다.
  - 예를 들어, `Processor`, `Manager`, `Super` 등과 같이 모호한 단어가 있다면 클래스에다 여러 책임을 떠안겼다는 증거다.
- 클래스 설명
  - 만일<sup>if</sup>, 그리고<sup>and</sup>, -(하)며<sup>or</sup>, 하지만<sup>but</sup>을 사용하지 않고서 25단어 내외
  - `SuperDashboard`
    - 마지막으로 포커스를 얻었던 컴포넌트에 접근하는 방법을 제공하며, 버전과 빌드 번호를 추적하는 메커니즘을 제공한다.
    - "~하며,"는 `SuperDashboard`에 책임이 너무 많다는 증거

### 단일 책임 원칙<sup>Single Responsibility Principle, SRP</sup>

단일 책임 원칙은 클래스나 모듈을 **변경할 이유**가 하나, 단 하나뿐이어야 한다는 원칙이다. SRP는 '책임'이라는 개념을 정의하며 적절한 클래스 크기를 제시한다. 클래스는 책임, 즉 변경할 이유가 하나여야 한다는 의미다.

`SuperDashboard`는 변경할 이유가 두 가지다.
1. `SuperDashboard`는 소프트웨어 버전 정보를 추적한다. 그런데 버전 정보는 소프트웨어를 출시할 때마다 달라진다.
2. `SuperDashboard`는 자바 스윙 컴포넌트를 관리한다. (`SuperDashboard`는 최상위 GUI 윈도의 스윙 표현인 `JFrame`에서 파생한 클래스다.) 즉, 스윙 코드를 변경할 때마다 버전 번호가 달라진다. (역은 참이 아니다. 때로는 다른 코드를 바꾸고 나서도 버전 번호를 바꾼다.)

책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다. 더 좋은 추상화가 더 쉽게 떠오른다. 

버전 정보를 다루는 메서드 세 개를 따로 빼내 `Version`이라는 독자적인 클래스를 만든다. `Version` 클래스는 다른 애플리케이션에서 재사용하기 아주 쉬운 구조다!

```swift
public class Version {
    public func getMajorVersionNumber() -> Int
    public func getMinorVersionNumber() -> Int
    public func getBuildNumber() -> Int
}
```

우리는 수많은 책임을 떠안은 클래스를 꾸준하게 접한다. 왜일까?

우리들 대다수는 '깨끗하고 체계적인 소프트웨어'보다 '돌아가는 소프트웨어'에 초점을 맞춘다. 관심사를 분리하는 작업은 프로그램만이 아니라 프로그래밍 **활동**에서도 마찬가지로 중요하다.

문제는 우리들 대다수가 프로그램이 돌아가면 일이 끝났다고 여기는 데 있다. '깨끗하고 체계적인 소프트웨어'라는 **다음** 관심사로 전환하지 않는다. 프로그램으로 되돌아가 만능 클래스를 단일 책임 클래스 여럿으로 분리하는 대신 다음 문제로 넘어가버린다.

큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다. 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

### 응집도<sup>Cohesion</sup>

클래스는 인스턴스 변수 수가 작아야 한다. 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다. 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 더 높다. 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높다.

일반적으로 이처럼 응집도가 가장 높은 클래스는 가능하지도 바람직하지도 않다. 응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미기 때문이다.

다음은 `Stack`을 구현한 코드다. 아래 클래스는 응집도가 아주 높다. `size()`를 제 외한 다른 두 메서드는 두 변수를 모두 사용한다.

```swift
public class Stack {
    private var topOfStack = 0
    var elements = [Int]()

    public func size() -> Int {
        return topOfStack
    }

    public func push(_ element: Int) {
        topOfStack += 1
        elements.append(element)
    }

    public func pop() throws -> Int {
        if topOfStack == 0 {
            throw PoppedWhenEmpty()
        }
        topOfStack -= 1
        var element = elements[topOfStack]
        elements.removeLast()
        return element
    }
}
```

'함수를 작게, 매개변수 목록을 짧게'라는 전략을 따르다 보면 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다. 이는 십중팔구 새로운 클래스로 쪼개야 한다는 신호다. 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 쪼개준다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다

큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.

예를 들어, 변수가 아주 많은 큰 함수 하나가 있다. 큰 함수 일부를 작은 함수 하나로 빼내고 싶은데, 빼내려는 코드가 큰 함수에 정의된 변수 넷을 사용한다. 그렇다면 변수 네 개를 새 함수에 인수로 넘겨야 옳을까?

전혀 아니다! 만약 네 변수를 클래스 인스턴스 변수로 승격한다면 새 함수는 인수가 **필요없다**.

불행히도 이렇게 하면 클래스가 응집력을 잃는다. 몇몇 함수만 사용하는 인스턴스 변수가 점점 더 늘어나기 때문이다. 그런데 잠깐만! 몇몇 함수가 몇몇 변수만 사용한다면 독자적인 클래스로 분리해도 되지 않는가? 당연하다. 클래스가 응집력을 잃는다면 쪼개라!

그래서 큰 함수를 작은 함수 여럿으로 쪼개다 보면 종종 작은 클래스 여럿으로 쪼갤 기회가 생긴다. 그러면서 프로그램에 점점 더 체계가 잡히고 구조가 투명해진다.

> 179 페이지 ~ 185 페이지의 실습은 생략합니다.

## 변경하기 쉬운 클래스

대다수 시스템은 지속적인 변경이 가해진다. 그리고 뭔가 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다. 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다.

다음은 주어진 메타 자료로 적절한 SQL 문자열을 만드는 `Sql` 클래스다. 아직 미완성이라 `update` 문과 같은 일부 SQL 기능을 지원하지 않는다. 언젠가 `update` 문을 지원할 시점이 오면 클래스에 '손대어' 고쳐야 한다. 문제는 코드에 '손대면' 위험이 생긴다는 사실이다. 어떤 변경이든 클래스에 손대면 다른 코드를 망가뜨릴 잠정적인 위험이 존재한다. 그래서 테스트도 완전히 다시 해야 한다.

```swift
public class Sql {
    init(table: String, columns: [Column])
    public func create() -> String
    public func insert(fields: [Object]) -> String
    public func selectAll() -> String
    public func findByKey(keyColumn: String, keyValue: String) -> String
    public func select(column: Column, pattern: String) -> String
    public func select(criteria: Criteria) -> String
    public func preparedInsert() -> String
    private func columnList(columns: [Column]) -> String
    private func valuesList(fields: [Object], columns: [Column])
    private func selectWithCriteria(criteria: String) -> String
    private func placeholderList(columns: [Column]) -> String
}
```

`Sql` 클래스는 SRP를 위반한다.
- 새로운 SQL 문을 지원하려면 반드시 `Sql` 클래스에 손대야 한다.
- 기존 SQL 문 하나를 수정할 때도 반드시 `Sql` 클래스에 손대야 한다.

단순히 구조적인 관점에서도 `Sql`은 SRP를 위반한다. `selectWithCriteria`라는 비공개 메서드가 있는데, 이 메서드는 `select`문을 처리할 때만 사용한다.

경험에 의하면 클래스 일부에서만 사용되는 비공개 메서드는 코드를 개선할 잠재적인 여지를 시사한다.

다음과 같은 방법은 어떨까? 위에 있던 공개 인터페이스를 각각 `Sql` 클래스에서 파생하는 클래스로 만들었다. `valueList`와 같은 비공개 메서드는 해당하는 파생 클래스로 옮겼다. 모든 파생 클래스가 공통으로 사용하는 비공개 메서드는 `Where`와 `ColumnList`라는 두 유틸리티 클래스에 넣었다.

```swift
public protocol Sql {
    public init(table: String, columns: [Column])
    public static func generate() -> String
}

public class CreateSql: Sql {
    public init(table: String, columns: [Column])
    override public func generate() -> String
}

public class SelectSql: Sql {
    public init(table: String, columns: [Column])
    override public func generate() -> String
}

public class InsertSql: Sql {
    public init(table: String, columns: [Column], fields: [Object])
    override public func generate() -> String
    private func valuesList(fields: [Object], columns: [Column]) -> String
}

public class SelectWithCriteriaSql: Sql {
    public init(table: String, columns: [Column], criteria: Criteria)
    override public func generate() -> String
}

public class SelectWithMatchSql: Sql {
    public init(table: String, columns: [Column], column: Column, pattern: String)
    override public func generate() -> String
}

public class FindByKeySql: Sql {
    public init(table: String, columns: [Column], keyColumn: String, keyValue: String)
    override public func generate() -> String
}

public class PreparedInsertSql: Sql {
    public init(table: String, columns: [Column])
    override public func generate() -> String
    private func placeholderList(columns: [Column]) -> String
}

public class Where {
    public init(criteria: String)
    public func generate() -> String
}

public class ColumnList {
    public init(columns: [Column])
    public func generate() -> String
}
```

각 클래스는 극도로 단순하다. 코드는 순식간에 이해된다. 함수 하나를 수정했다고 다른 함수가 망가질 위험도 사실상 사라졌다.

재구성한 `Sql` 클래스는 세상의 모든 장점만 취한다!
- SRP를 지원한다.
- OCP<sup>Oepn-Closed Principle</sup>도 지원한다.
    - OCP란 클래스는 확장에 개방적이고 수정에 폐쇄적이어야 한다는 원칙이다.

새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직하다. 이상적인 시스템이라면 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하지는 않는다.

### 변경으로부터 격리

요구사항은 변하기 마련이다. 따라서 코드도 변하기 마련이다. 상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다. 그래서 우리는 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

상세한 구현에 의존하는 코드는 테스트가 어렵다. 예를 들어, `Portfolio` 클래스를 만든다고 가정하자. 그런데 `Portfolio` 클래스는 외부 TokyoStockExchange API를 사용해 포트폴리오 값을 계산한다. 따라서 우리 테스트 코드는 시세 변화에 영향을 받는다.

`Portfolio` 클래스에서 TokyoStockExchange API를 직접 호출하는 대신 StockExchange라는 인터페이스를 생성한 후 메서드 하나를 선언한다.

```swift
public protocol StockExchange {
    func currentPrice(symbol: String) -> Money
}
```

다음으로 `StockExchange` 인터페이스를 구현하는 `TokyoStockExchange` 클래스를 구현한다. 또한 `Portfolio` 생성자를 수정해 `StockExchange` 참조자를 인수로 받는다.

```swift
public class Portfolio {
    private var exchange: StockExchange
    
    public init(exchange: StockExchange) {
        self.exchange = exchange
    }

    // ...
}
```

이제 `TokyoStockExchange` 클래스를 흉내내는 테스트용 클래스를 만들 수 있다. 테스트용 클래스는 `StockExchange` 인터페이스를 구현하며 고정된 주가를 반환한다.

```swift
import XCTest

public class PortfolioTest: XCTestCase {
    private var exchange: FixedStockExchangeStub
    private var portfolio: Portfolio

    override func setUp() {
        super.setUp()

        exchange = FixedStockExchangeStub()
        exchange.fix("MSFT", 100)
        portfolio = Portfolio(exchange)
    }

    func testGicenFiveMSFTTotalShouldBe500() {
        portfolio.add(5, "MSFT")
        XCTAssertEquals(500, portfolio.value())
    }

}
```

위와 같은 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다. 결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미다. 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.

이렇게 결합도를 최소로 줄이면 자연스럽게 또 다른 클래스 설계 원칙인 DIP<sup>Dependency Inversion Principle</sup>를 따르는 클래스가 나온다. 본질적으로 DIP는 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙이다.