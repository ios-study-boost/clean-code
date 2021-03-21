# 5장. 형식 맞추기

프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야 한다.

## 규칙
- [형식을 맞추는 목적](#형식을-맞추는-목적)
- [적절한 행 길이를 유지하라](#적절한-행-길이를-유지하라)
- [가로 형식 맞추기](#가로-형식-맞추기)
- [팀 규칙](#팀-규칙)

## 형식을 맞추는 목적

오늘 구현한 기능이 다음 버전에서 바뀔 확률은 아주 높다. 그런데 오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다. 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다. 원래 코드는 사라질지라도 개발자의 스타일과 규율은 사라지지 않는다.

## 적절한 행 길이를 유지하라
200줄 정도인 파일로도 커다란 시스템을 구축할 수 있다. 일반적으로 큰 파일보다 작은 파일이 이해하기 쉽다.

**신문 기사처럼 작성하라**

- 이름은 간단하면서도 설명이 가능하게 짓는다.
- 소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명한다.
- 아래로 내려갈수록 의도를 세세하게 묘사한다.
- 마지막에는 가장 저차원 함수와 세부 내역이 나온다.

**개념은 빈 행으로 분리하라**

- 거의 모든 코드는 왼쪽에서 오른쪽으로 그리고 위에서 아래로 읽힌다.
- 각 행은 수식이나 절을 나타낸다.
- 일련의 행 묶음은 완결된 생각 하나를 표현한다.
- 생각 사이는 빈 행을 넣어 분리해야 마땅하다.

**세로 밀집도**

줄바꿈이 개념을 분리한다면 세로 밀집도는 연관성을 의미한다. 서로 밀접한 코드 행은 세로로 가까이 놓여야 한다.

🚫 Bad
```swift
public class ReporterConfig {
    /*
    * 리포터 리스너의 클래스 이름
    */
    private var className = String()

    /*
    * 리포터 리스너의 속성
    */
    private var properties: [Property] = [Property]()
    public func addProperty(property: Property) {
        properties.append(property)
    }
}
```

👍 Correct
```swift
public class ReporterConfig {
    private var className = String()
    private var properties: [Property] = [Property]()

    public func addProperty(property: Property) {
        properties.append(property)
    }
}
```

**수직 거리**

- 타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다.
- 같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성을 표현한다.
  - 여기서 연관성이란 한 개념을 이해하는 데 다른 개념이 중요한 정도다.
- **변수 선언**: 변수는 사용하는 위치에 최대한 가까이 선언한다.
- **인스턴스 변수**: 인스턴스 변수는 클래스 맨 처음에 선언한다.
- **종속 함수**: 한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다.
- **개념적 유사성**: 개념적인 친화도가 높을수록 코드를 가까이 배치한다.

**세로 순서**

- 호출되는 함수를 호출하는 함수보다 나중에 배치한다. 그러면 소스 코드 모듈이 고차원에서 저차원으로 자연스럽게 내려간다.
- 가장 중요한 개념을 가장 먼저 표현한다.
- 세세한 사항은 가장 마지막에 표현한다.

## 가로 형식 맞추기

(저자는) 120자 정도의 행 길이를 제한한다.

**가로 공백과 밀집도**

- 할당문은 왼쪽 요소와 오른쪽 요소가 분명히 나뉜다. 공백을 넣으면 두 가지 주요 요소가 확실히 나뉜다는 사실이 더욱 분명해진다. 
  - `var lineSize = line.count`

- 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않는다. 함수와 임수는 서로 밀접하기 때문이다. 
  - `recordWidestLine(lineSize)`

- 함수를 호출하는 코드에서 괄호 안 인수는 공백으로 분리한다. 쉼표를 강조해 인수가 별개라는 사실을 보여주기 위해서다. 
  - `lineWidthHistogram.addLine(lineSize, lineCount)`

- 연산자 우선순위를 강조하기 위해서도 공백을 사용한다.
  - `b*b - 4*a*c`

**가로 정렬**

```swift
public class FitNesseExpediter: ResponseSender {
    private var socket:          Socket
    private var input:           InputStream
    private var output:          OutputStream
    private var requestProgress: Double

    public func FitNesseExpediter(s:       Socket,
                                  context: FitNesseContext) throws {
        socket =          s
        input =           s.getInputStream()
        output =          s.getOutputStream()
        requestProgress = 10000
    }
}
```

위와 같은 정렬은 별로 유용하지 못하다.
- 코드가 엉뚱한 부분을 강조해 진짜 의도가 가려진다.
- 변수 유형은 무시하고 변수 이름부터 읽게 된다.
- 할당 연산자는 보이지 않고 오른쪽 피연산자에 눈이 간다.

선언문과 할당문은 별도로 정렬하지 않는다.
- 정렬하지 않으면 오히려 중대한 결함을 찾기 쉽다.
- 정렬이 필요할 정도로 목록이 길다면 문제는 목록 **길이**지 정렬 부족이 아니다.
- 선언부가 길다면 클래스를 쪼개야 한다는 의미다.

**들여쓰기**

- 범위<sup>scope</sup>로 이뤄진 계층을 표현하기 위해 우리는 코드를 들여쓴다. 들여쓰는 정도는 계층에서 코드가 자리잡은 수준에 비례한다.
- 들여쓰기가 없다면 코드를 읽기란 거의 불가능하다.
- **들여쓰기 무시하기**: (저자는) 다음과 같이 한 행에 범위를 뭉뚱그린 코드를 피한다.  
    🚫 Bad
    ```swift
    public class CommentWidget: TextWidget {
        public func render() -> String { return "" }
    }
    ```
    👍 Correct
    ```swift
    public class CommentWidget: TextWidget {
        public func render() -> String {
            return ""
        }
    }
    ```

**가짜 범위**

- 빈 `while`문이나 `for`문 형태의 구조는 가능한 피한다.
- 피하지 못할 때는 빈 블록을 올바로 들여쓰고 괄호로 감싼다.

## 팀 규칙

프로그래머라면 각자 선호하는 규칙이 있다. 하지만 팀에 속한다면 자신이 선호해야 할 규칙은 바로 팀 규칙이다.
- 팀은 한 가지 규칙에 합의해야 한다.
- 모든 팀원은 그 규칙을 따라야 한다.