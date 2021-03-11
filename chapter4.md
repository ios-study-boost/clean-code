# 4. 주석



## 코드로 의도를 표현해라

프로그래밍 언어를 사용해 의도를 표현할 능력이 있다면, 주석은 필요하지 않다

개발자들이 주석을 유지하고 보수하기란 현실적으로 불가능하다 

그러므로 주석은 오래될수록 코드에서 멀어지고, 완전히 그릇될 가능성이 커진다

```swift
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if (employee.flags & HOURLY_FLAG) && employee.age > 65 {
} 
```

```swift
if employee.isEligibleForFullBenefits() {
}
```

조금만 더 생각하면 코드로 대다수 의도를 표현할 수 있다

## 주석은 나쁜 코드를 보완하지 못한다

우리가 일반적으로 주석을 추가하는 이유는 코드 품질이 나쁘기 때문이다.



------



## 좋은 주석

결과를 경고한는 주석 - 장시간이 걸리는 테스트 케이스

TODO 주석

의도를 명료하게 밝히는 주석

```swift
assert(a.compare(a) == 0) // a == a
assert(a.compare(b) != 0) // a != b
```



## 나쁜 주석

주석으로 처리한 코드

주석처리된 코드는 다른 사람들이 지우기를 주저한다. 이유가 있어서 남겨놓은 코드라고 생각한다

그러다보면 쓸모없는 코드가 쌓여간다

```swift
let foreground = Color(red: 32, green: 64, blue: 128)
// let newPart = factory.makeWidget(gears: 42, spindles: 14)
//let ref = Link(target: destination)
```

