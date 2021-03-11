# 3. 함수



## 작게, 더 작게!

얼마나 짧아야 좋을까?
일반적으로 10줄 이내이고 목록 3-3 처럼 2~4줄 정도로 작성했을 때 좋다

if/else, while 문 등에 들어가는 블록은 한줄이어야 한다
함수이름을 적절히 짓는다면 코드를 이해하기 쉬워진다.

중첩 구조는 지양하자
들여쓰기 수준은 1~2단 중첩 구조가 깊어질수록 함수는 이해하기 어려워진다

```swift
// 목록 3-3
// 설정(setup) 페이지,해제(teardown) 페이지를 테스트 페이지에 넣은후 HTML로 렌터딩하는 함수
func renderPageWithSetupsAndTeardowns(pageData: PageData, isSuite: Bool) -> String {
	if isTestPage(pageData) {
		includeSetupAndTeardownPages(pageData, isSuite)
	}
	return pageData.getHtml()
}
```



------



## 한 가지만 해라!

> 함수는 한 가지를 해야한다. 그 한 가지를 잘 해야한다. 그 한 가지만을 해야한다.

한 가지를 어떻게 구별할 것인가? 목록 3-3 은 한가지만 하는가? 세 가지를 하는가?

1. 페이지가 테스트 페이지인기 판단한다.
2. 그렇다면 설정 페이지와 해제 페이지를 넣는다.
3. 페이지를 HTML로 렌더링한다.

위의 3단계는 함수 이름(설정,해제 페이지를 랜더링) 아래 추상화 수준이 하나다.
고로, 한가지일을 하는 함수다

추상화 수준이란 무엇인가?

먼저 추상화란 사소한 요소를 제거하고 **핵심만 뽑아내는 것**

추상화 수준은 핵심을 뽑아내는 정도로 단계가 낮아질수록 구체적이며 높아질수록 일반적이다.
아래의 주소예시를 보자

1. 실제 주소 - *476 N Bond St., Fresno, CA 94420*.
2. 빌딩 -  I live in the blue apartment complex on Michigan Ave.
3. 동 - I live in Jackson Heights, Queens
4. 도시 -  *I’m live in Chicago, IL*.
5. 국가 -  *I’m from the United States*.
6. 행성 -  *I’m from the planet Earth. Take me to your leader.*

```swift
// 설정 페이지,해제 페이지를 테스트 페이지에 넣은후 HTML로 렌터딩하는 함수
func renderPageWithSetupsAndTeardowns(pageData: PageData, isSuite: Bool) -> String {
	if isTestPage(pageData) {
		// 추상화 수준이 낮아짐
		if isSuite { 
			let suitePage = SuitePage()
			pageData.append(suitePage)
		}
		let setupPage = SetupPage()
		pageData.append(setupPage)
	}
	return pageData.getHtml()
}
```

renderPageWithSetupsAndTeardowns는 설정, 해제 페이지를 테스트 페이지에 넣어 HTML로 렌더링 하는 함수인데 4번라인에서 설정 페이지에대해서 추상화 수준이 낮아진다
한 함수에서 추상화 수준이 섞이면 즉, 전체적인 기능과 세부기능이 섞이면 읽는 사람이 햇갈린다.

단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다 - 로버트 C. 마틴



------



## 서술적인 이름을 사용해라

👍🏻 Good

```swift
x.insert(y, at: z)          “x, insert y at z”
x.subViews(havingColor: y)  “x's subviews having color y”
x.capitalizingNouns()       “x, capitalizing nouns”
```

👎🏻 Bad

```swift
x.insert(y, position: z)
x.subViews(color: y)
x.nounCapitalize()
```



------



## 함수 인수를 줄여라

> 함수에서 이상적인 인수 개수는 0개(무항)이다.

테스트 관점에서 보면 인수가 많아 질수록 유효한 값으로 모든 조합을 구성해 테스트하기 어렵다.



------



## 부수효과(side effect)를 일으키지 마라

함수에서 한가지일을 하겠다고 하고 몰래 다른 일도 하는 것은 SRP(단일 책임 원칙)에 위배되는 행위이다

```swift
func checkPassword(userName: String, passWord: String) -> Bool {
	let user = UserDB.findByName(userName)
	if user != nil {
		if user.passWord == passWord { 
			Session.initialize() // SIDE EFFECT
			return true
		}
	}
	return false
}
```

checkPassword함수가 일으키는 side effect는 Session.initialize() 호출이다.
checkPassword의 기능은 암호를 확인하는일이며 세션을 초기화 한다는 사실이 들어나지 않는다.

