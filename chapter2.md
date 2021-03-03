# 2장 의미 있는 이름
소프트웨어에서 이름은 어디나 쓰인다. 이 장에서는 이름을 잘 짓는 간단한 규칙을 알아본다.

## 규칙
- [의도를 분명히 밝혀라](#의도를-분명히-밝혀라)
- [그릇된 정보를 피하라](#그릇된-정보를-피하라)
- [의미 있게 구분하라](#의미-있게-구분하라)
- [발음하기 쉬운 이름을 사용하라](#발음하기-쉬운-이름을-사용하라)
- [검색하기 쉬운 이름을 사용하라](#검색하기-쉬운-이름을-사용하라)
- [인코딩을 피하라](#인코딩을-피하라)
- [자신의 기억력을 자랑하지 마라](#자신의-기억력을-자랑하지-마라)
- [클래스 이름](#클래스-이름)
- [메서드 이름](#클래스-이름)
- [기발한 이름은 피하라](#기발한-이름은-피하라)
- [한 개념에 한 단어를 사용하라](#한-개념에-한-단어를-사용하라)
- [말장난을 하지 마라](#말장난을-하지-마라)
- [해법 영역에서 가져온 이름을 사용하라](#해법-영역에서-가져온-이름을-사용하라)
- [문제 영역에서 가져온 이름을 사용하라](#문제-영역에서-가져온-이름을-사용하라)
- [의미 있는 맥락을 추가하라](#의미-있는-맥락을-추가하라)
- [불필요한 맥락을 없애라](#불필요한-맥락을-없애라)
- [마치면서](#마치면서)

## 의도를 분명히 밝혀라
변수나 함수 그리고 클래스 이름은 다음과 같은 굵직한 질문에 모두 답해야 한다. _변수(혹은 함수나 클래스)의 존재 이유는? 수행 기능은? 사용 방법은?_ 따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말이다.

의도가 드러나는 이름을 사용하면 코드 이해와 변경이 쉬워진다.

```swift
func getThem() -> Array<Array<Int>> {
    var list1: Array<Array<Int>> = Array<Array<Int>>()
    for x in theList {
        if x[0] == 4 {
            list1.append(x)
        }
    }
    return list1
}
```

위 코드는 암암리에 독자가 다음과 같은 정보를 안다고 가정한다.
1. `theList`에 무엇이 들었는가?
2. `theList`에서 0번째 값이 어째서 중요한가?
3. 값 4는 무슨 의미인가?
4. 함수가 반환하는 리스트 `list1`을 어떻게 사용하는가?

개념에 이름만 붙여도 코드가 상당히 나아진다.

```swift
func getFlaggedCells() -> Array<Array<Int>> {
    var flaggedCells: Array<Array<Int>> = Array<Array<Int>>()
    for cell in gameBoard {
        if cell[StatusValue] == Flagged {
            flaggedCells.append(cell)
        }
    }
    return flaggedCells
}
```
간단한 클래스 생성, 좀 더 명시적인 함수를 사용해 개선하면 다음과 같다.
```swift
func getFlaggedCells() -> Array<Cell> {
    var flaggedCells: Array<Cell> = Array<Cell>()
    for cell in gameBoard {
        if cell.isFlagged() {
            flaggedCells.append(cell)
        }
    }
    return flaggedCells
}
```

## 그릇된 정보를 피하라
프로그래머는 코드에 그릇된 단서를 남겨서는 안 된다. 그릇된 단서는 코드 의미를 흐린다. 나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해도 안 된다.

- 여러 계정을 그룹으로 묶을 때, 실제 `List`가 아니라면, `accountList`라 명명하지 않는다.<sup id="f1">[1](#footnote1)</sup>
  - `accountGroup`, `bunchOfAccounts`, `Accounts` 등으로 명명한다.

- 서로 흡사한 이름을 사용하지 않도록 주의한다.
  - `XYZControllerForEfficientHandlingOfStrings`
  - `XYZControllerForEfficientStorageOfStrings`

- 유사한 개념은 유사한 표기법을 사용한다. 이것도 정보다. 
  - 일관성이 떨어지는 표기법은 그릇된 정보다.

## 의미 있게 구분하라
컴파일러나 인터프리터만 통과하려는 생각으로 코드를 구현하는 프로그래머는 스스로 문제를 일으킨다.

- 컴파일러를 통과할지라도 연속된 숫자를 덧붙이거나 불용어<sup>noise word</sup>를 추가하는 방식은 적절하지 못하다.
  - 이름이 달라야 한다면 의미도 달라져야 한다.

- 연속적인 숫자를 덧붙인 이름(a1, a2, …, aN)은 아무런 정보를 제공하지 못하는 이름이다.

- 함수 인수 이름으로 `souce`와 `destination`을 사용한다면 코드 읽기가 훨씬 더 쉬워진다.
    ```swift
    public static func copyChars(a1: Array<Character>, a2:inout Array<Character>) {
        for i in 0..<a1.count {
            a2[i] = a1[i]
        }
    }
    ```
- `Info`나 `Data`는 의미가 불분명한 불용어다.
  - `ProductInfo`
  - `ProductData`

- a나 the와 같은 접두어는 의미가 분명히 다르다면 사용해도 무방하다.
  - `zork`라는 변수가 있다는 이유만으로 `theZork`라 이름 지어서는 안 된다.

- 읽는 사람이 차이를 알도록 이름을 지어라.
  - `moneyAmount` vs `money`
  - `customerInfo` vs `customer`
  - `accountData` vs `account`
  - `theMessage` vs `message`

## 발음하기 쉬운 이름을 사용하라
발음하기 어려운 이름은 토론하기도 어렵다. 발음하기 쉬운 이름은 중요하다.

🚫 Bad
```swift
class DtaRcrd102 {
    private var genymdhms: Date
    private var modymdhms: Date
    private let pszqint: String = "102"
    /* ... */
}
```
👍 Good
```swift
class Customor {
    private var generationTimestamp: Date
    private var modificationTimestamp: Date
    private let recordId: String = "102"
    /* ... */
}
```

## 검색하기 쉬운 이름을 사용하라
이름 길이는 범위 크기에 비례해야 한다. 변수나 상수를 코드 여러 곳에서 사용한다면 검색하기 쉬운 이름이 바람직하다. 

예를 들어, 숫자 5를 그냥 사용하기보다는 `WORK_DAYS_PER_WEEK`와 같이 상수로 선언하고 사용하면 검색이 쉬워진다.<sup id="f2">[2](#footnote2)</sup>

## 인코딩을 피하라
유형이나 범위 정보까지 인코딩에 넣으면 그만큼 이름을 해독하기 어려워진다. 

**헝가리식 표기법**  
과거에는 컴파일러가 타입을 점검하지 않아 프로그래머에게 타입을 기억할 단서가 필요했다. 그러나 최근에는 컴파일러가 타입을 기억하고 강제한다. 게다가 클래스와 함수는 점차 작아지는 추세여서 변수를 선언한 위치와 사용하는 위치가 멀지 않다.

```swift
var phoneString: PhoneNumber
// 타입이 바뀌어도 이름은 바뀌지 않는다!
```

**멤버 변수 접두어**  
이제는 멤버 변수에 `m_`이라는 접두어를 붙일 필요도 없다. 클래스와 함수는 접두어가 필요없을 정도로 작아야 마땅하다. 또한 멤버 변수를 다른 색상으로 표시하거나 눈에 띄게 보여주는 IDE를 사용해야 마땅하다.

**인터페이스 클래스와 구현 클래스**  
```
💡 여기서 말하는 인터페이스 클래스는 스위프트의 프로토콜과 비슷한 개념
```
인터페이스 클래스와 구현 클래스 중 하나를 인코딩해야 한다면 구현 클래스에 하라.

## 자신의 기억력을 자랑하지 마라
독자가 코드를 읽으면서 변수 이름을 자신이 아는 이름으로 변환해야 한다면 그 변수 이름은 바람직하지 못하다. 이는 일반적으로 문제 영역이나 해법 영역에서 사용하지 않는 이름을 선택했기 때문에 생기는 문제다.

- 문자 하나만 사용하는 변수 이름은 문제가 있다.
  - 루프에서 반복 횟수를 세는 변수 `i`, `j`, `k`는 괜찮다. (`l`은 절대 안된다!)

- **명료함이 최고**
  - 전문가 프로그래머는 남들이 이해하는 코드를 작성

## 클래스 이름
클래스 이름과 객체 이름은 명사나 명사구가 적합하다. 
- `Customer`, `WikiPage`, `Account`, `AddressParser`

동사는 사용하지 않는다.
- `Manager`, `Processor`, `Data`, `Info`

## 메서드 이름
메서드 이름은 동사나 동사구가 적합하다.
- `postPayment`, `deletePage`, `save`

생성자<sup>Constructor</sup>를 중복정의<sup>overload</sup>할 때는 정적 팩토리 메서드를 사용한다. 메서드는 인수를 설명하는 이름을 사용한다. 생성자 사용을 제한하려면 해당 생성자를 `private`로 선언한다.

👍 Good<sup id="f3">[3](#footnote3)</sup>
```swift
var fulcrumPoint: Complex = Complex.makeFromRealNumber(23.0)
```
🚫 Bad
```swift
var fulcrumPoint: Complex = Complex(23.0)
```

## 기발한 이름은 피하라
재미난 이름보다 명료한 이름을 선택해라. 의도를 분명하고 솔직하게 표현하라.

## 한 개념에 한 단어를 사용하라
추상적인 개념 하나에 단어 하나를 선택해 이를 고수한다. 
- `fetch`, `retrieve`, `get`
  - 똑같은 메서드를 클래스마다 다르게 부르면 혼란스럽다.

- `controller`, `manager`, `driver`
  - `DeviceManager`와 `ProtocolController`는 근본적으로 어떻게 다른가?
  - 이름이 다르면 독자는 당연히 클래스도 다르고 타입도 다르리라 생각한다.

## 말장난을 하지 마라
한 단어를 두 가지 목적으로 사용하지 마라. 다른 개념에 같은 단어를 사용한다면 그것은 말장난에 불과하다.

- 모든 `add` 메서드의 매개변수와 반환값이 의미적으로 똑같다면 문제가 없다.
  - 하지만 때로는 프로그래머가 같은 맥락이 아닌데도 '일관성'을 고려해 `add`라는 단어를 선택한다.

## 해법 영역에서 가져온 이름을 사용하라
코드를 읽을 사람도 프로그래머이므로 전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어 등을 사용해도 괜찮다. 모든 이름을 문제 영역<sup>domain</sup>에서 가져오는 정책은 현명하지 못하다.

## 문제 영역에서 가져온 이름을 사용하라
적절한 '프로그래머 용어'가 없다면 문제 영역<sup>domain</sup>에서 이름을 가져온다. 도메인 개념과 관련이 깊은 코드라면 도메인에서 이름을 가져와야 한다.

## 의미 있는 맥락을 추가하라
클래스, 함수, 이름 공간에 의미를 넣어 맥락을 부여한다. 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다.

- `firstName`, `lastName`, `street`, `houseNumber`, `city`, `state`, `zipcode`
  - 변수를 훑어보면 주소라는 사실을 금방 알아챈다.
    - 하지만 어느 메서드가 `state`라는 변수 하나만 사용한다면 `state`가 주소의 일부라는 사실을 금방 알아챌까?

  - `addrFirstName`, `addrLastName`, `addrState`
    - `addr`라는 접두어를 추가해 쓰면 맥락이 좀 더 분명해진다.
    - 변수가 좀 더 큰 구조에 속한다는 사실이 적어도 독자에게는 분명해진다.
  
  - `Adress` 클래스
    - 변수가 좀 더 큰 개념에 속한다는 사실이 컴파일러에게도 분명해진다.

## 불필요한 맥락을 없애라
- '고급 휘발유 충전소<sup>Gas Station Deluxe</sup>'
  - 모든 클래스 이름을 `GSD`로 시작하겠다는 생각은 전혀 바람직하지 못하다.

- 일반적으로 짧은 이름이 긴 이름보다 좋다. 단, 의미가 분명한 경우에 한해서다. 이름에 불필요한 맥락을 추가하지 않도록 주의한다.

- `accountAddress`, `customerAddress`
  - 클래스 인스턴스로는 좋은 이름이나 클래스 이름으로는 적합하지 못하다.
  - `Address`는 클래스 이름으로 적합하다.

## 마치면서
여느 코드 개선 노력과 마찬가지로 이름 역시 나름대로 바꿨다가는 누군가 질책할지도 모른다. 그렇다고 코드를 개선하려는 노력을 중단해서는 안 된다.

---

<sup id="footnote1">1</sup> 실제 컨테이너가 `List`인 경우라도 컨테이너 유형을 이름에 넣지 않는 편이 바람직하다. [↩](#f1)

<sup id="footnote2">2</sup> 스위프트에서 올바른 표기법은 `workDaysPerWeek`이다.<sup>[참고](https://gist.github.com/godrm/d07ae33973bf71c5324058406dfe42dd#일반-규칙)</sup> [↩](#f2)

<sup id="footnote3">3</sup> 팩토리 메서드 이름은 'make'로 시작해야 한다.<sup>[참고](https://gist.github.com/godrm/d07ae33973bf71c5324058406dfe42dd#말하는-것처럼-술술-써지도록-작성하라)</sup> [↩](#f3)
