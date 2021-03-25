# 6장. 객체와 자료 구조

## 규칙
- [자료 추상화](#자료-추상화)
- [자료/객체 비대칭](#자료객체-비대칭)
- [디미터 법칙](#디미터-법칙)
  - [기차 충돌](#기차-충돌)
  - [잡종 구조](#잡종-구조)
  - [구조체 감추기](#구조체-감추기)
- [자료 전달 객체](#자료-전달-객체)
  - [활성 레코드](#활성-레코드)
- [결론](#결론)

## 자료 추상화
```swift
public class Point {
    public var x: Double
    public var y: Double
}
```

```swift
public protocol Point {
    func getX() -> Double
    func getY() -> Double
    func setCartesian(x: Double, y: Double)
    func getR() -> Double
    func getTheta() -> Double
    func setPolar(r: Double, theta: Double)
}
```

`Point` 프로토콜은 점이 직교좌표계를 사용하는지 극좌표계를 사용하는지 알 길이 없다. 둘 다 아닐지도 모른다!

변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지는 않는다. 구현을 감추려면 추상화가 필요하다! 그저 (형식 논리에 치우쳐) 조회 함수와 설정 함수로 변수를 다룬다고 클래스가 되지는 않는다. 그보다는 **추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스**다.

```swift
public protocol Vehicle {
    func getFuelTankCapacityInGallons() -> Double
    func getGallonsOfGasoline() -> Double
}
```

```swift
public protocol Vehicle {
    func getPercentFuelRemaining() -> Double
}
```
첫번째 `Vehicle` 프로토콜은 자동차 연료 상태를 구체적인 숫자 값으로 알려준다. 두번째 `Vehicle` 프로토콜은 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다.

```markdown
Point 클래스 < Point 프로토콜 < 첫번째 Vehicle 프로토콜 < 두번째 Vehicle 프로토콜
```

개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야 한다. 아무 생각 없이 조회/설정 함수를 추가하는 방법이 가장 나쁘다.

## 자료/객체 비대칭

- 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
- 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.

```swift
public class Square {
    public var topLeft: Point
    public var side: Double
}

public class Rectangle {
    public var topLeft: Point
    public var height: Double
    public var wdith: Double
}

public class Circle {
    public var center: Point
    public var radius: Double
}

public class Geometry {
    public static let Pi = 3.1415292653589793

    public func area(shape: Object) throws -> Double {
        if let s = shape as? Square {
            return s.side * s.side
        } else if r = shape as? Rectangle {
            return r.height * r.width
        } else if c = shape as? Circle {
            return Pi * c.radius * c.radius
        }
    }
}
```

객체 지향 프로그래머가 위 코드를 본다면 코웃음을 칠지도 모르겠다. 
- 만약 `Geometry` 클래스에 둘레 길이를 구하는 `perimeter()` 함수를 추가하고 싶다면?
  - 도형 클래스는 아무 영향도 받지 않는다. 도형 클래스에 의존하는 다른 클래스도 마찬가지다.
- 반대로 새 도형을 추가하고 싶다면?
  - `Geometry` 클래스에 속한 함수를 모두 고쳐야 한다.

```swift
public class Square: Shape {
    private var topLeft: Point
    private var side: Double

    public func area() -> Double {
        return side * side
    }
}

public class Rectangle: Shape {
    private var topLeft: Point
    private var height: Double
    private var width: Double

    public func area() -> Double {
        return height * width
    }
}

public class Circle: Shape {
    private var center: Point
    private var radius: Double
    public static let Pi = 3.1415292653589793

    public func area() -> Double {
        return Pi * radius * radius
    }
}
```

객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.

때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

## 디미터 법칙

디미터 법칙은 잘 알려진 휴리스턱<sup>heuristic</sup>으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다.

디미터 법칙은 "클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.
- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안 된다.

다음 코드는 (다른 법칙은 제쳐두고서라도) 디미터 법칙을 어기는 듯이 보인다.
```swift
let outputDir: String = ctxt.getOptions().getScratchDir().getAbsolutePath()
```

### 기차 충돌

흔히 위와 같은 코드를 **기차 충돌**<sup>train wreck</sup>이라 부른다. 여러 객차가 한 줄로 이어진 기차처럼 보이기 때문이다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다.

위 코드는 다음과 같이 나누는 편이 좋다.
```swift
var opts: Options = ctxt.getOptions()
var scratchDir: File = opts.getScratchDir()
let outputDir: String = scratchDir.getAbsolutePath()
```

위 예제가 디미터 법칙을 위반하는지 여부는 `ctxt`, `opts`, `scratchDir`이 객체인지 아니면 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다. 반면, 자료구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

코드를 다음과 같이 구현했다면 디미터 법칙을 거론할 필요가 없어진다.
```swift
let outputDir: String = ctxt.options.scratchDir.absolutePath
```

단순한 자료 구조에도 조회 함수와 설정 함수를 정의하라 요구하는 프레임워크와 표준(예, '빈<sup>bean</sup>')이 존재한다.

### 잡종 구조

잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다. 공개 조회/설정 함수는 비공개 변수를 그대로 노출한다.

잡종 구조는 되도록 피하는 편이 좋다. 프로그래머가 함수나 타입을 보호할지 공개할지 확신하지 못해 (더 나쁘게는 무지해) **어중간하게 내놓은 설계에 불과**하다.

### 구조체 감추기

만약 `ctxt`, `opts`, `scratchDir`이 진짜 객체라면? 객체라면 내부 구조를 감춰야 하니까 줄줄이 사탕처럼 엮어서는 안 된다.

```swift
ctxt.getAbsolutePathOfScratchDirectoryOption() // 첫번째

ctxt.getScratchDirectoryOption().getAbsolutePath() // 두번째
```

- 첫 번째 방법은 `ctxt` 객체에 공개해야 하는 메서드가 너무 많아진다.
- 두 번째 방법은 `getScratchDirectoryOption()`이 객체가 아니라 자료 구조를 반환한다고 가정한다.

같은 모듈에서 (한참 아래로 내려가서) 코드를 살펴보면, 임시 디렉터리의 절대 경로를 얻으려는 이유가 임시 파일을 생성하기 위한 목적이라는 사실이 드러난다.

그렇다면 `ctxt` 객체에 임시 파일을 생성하라고 시키자.
```swift
var bos: BufferedOutputStream = ctxt.createScratchFileStream(classFileName)
```

## 자료 전달 객체

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. 이런 자료 구조체를 때로는 자료 전달 객체<sup>Data Transfer Object, DTO</sup>라 한다. 흔히 DTO는 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체다.

좀 더 일반적인 형태는 '빈<sup>bean</sup>' 구조다. 빈은 비공개 변수를 조회/설정 함수로 조작한다.

```swift
public class Address {
    private var street: String
    private var streetExtra: String
    private var city: String
    private var state: String
    private var zip: String

    public init(street: String, streetExtra: String,
                city: String, state: String, zip: String) {
        self.street = street
        self.streetExtra = streetExtra
        self.city = city
        self.state = state
        self.zip = zip
    }

    public func getStreet() -> String {
        return street
    }

    public func getStreetExtra() -> String {
        return streetExtra
    }

    public func getCity() -> String {
        return city
    }

    public func getState() -> String {
        return state
    }

    public func getZip() -> String {
        return zip
    }
}
```

### 활성 레코드

활성 레코드는 DTO의 특수한 형태다. 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조지만, 대게 `save`나 `find`와 같은 탐색 함수도 제공한다. 활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.

활성 레코드는 자료 구조로 취급한다. 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다. (여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.)

## 결론

- 객체
  - 동작을 공개하고 자료를 숨긴다. 
  - 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉽다.
  - 그러나 기존 객체에 새 동작을 추가하기는 어렵다.
  - 새로운 자료 타입을 추가하는 유연성이 필요하면 적합하다.
- 자료 구조
  - 별다른 동작없이 자료를 노출한다. 
  - 기존 자료 구조에 새 동작을 추가하기는 쉽다.
  - 그러나 기존 함수에 새 자료 구조를 추가하기는 어렵다.
  - 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 적합하다.

우수한 소프트웨어 개발자는 편견없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.
