# 5. 오류 처리



> 잘못될 가능성은 늘 존재한다. 뭔가 잘못되면 바로 잡을 책은 우리에게 있다
>
> 하지만, 오류 처리 코드로 인해 프로그램 로직을 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다
>



### 오류 코드보다 예외를 사용하라

예외를 사용하지 않은 코드

```swift
func sendShutdown() {
    var handle: DevciceHandle = getHandle(dev1)
    // 디바이스 상태를 점검한다
    if handle != DeviceHandle.Invalid {
        //레코드 필드에 디바이스 상태를 저장한다
        retrieveDeviceRecord(handle)
        // 디바이스가 일시정지 상태가 아니라면 종료한다
        if record.getStaus() != DEVICE_SUSPENDED {
            pauseDevice(handle)
            clearDeviceWorkQueue(handle)
            closeDevice(handle)
        } else {
            os_log("Device suspended. Unable to shut down")
        }
    } else {
        os_log("Invalid handle for: \(dev1)")
    }
}
```

위와 같이 오류코드와 실행 로직을 뒤섞으면 호출자 코드가 복잡해지고 오류 확인을 잊어버리기 쉽다.

따라서 오류 발생 시 예외를 던지는 것이 낫다. 그러면 호출자 코드는 더 깔끔해진다



예외 처리 코드 

```swift
func sendShutdown() {
    do {
        try tryToShutDown()
    } catch {
        os_log("ShutDown Fail")
    }
}

func tryToShutDown() throws {
    var handle: DevciceHandle = getHandle(dev1)
    var record: DeviceRecord = retrieveDeviceRecord(handle)
    
    pauseDevice(handle)
    clearDeviceWorkQueue(handle)
    closeDevice(handle)
}
```



두개의 코드를 비교해보면 앞서서 오류처리와 실행 로직이 뒤섞여 알아보기 어려웠지만 예외 처리한 코드는 실행로직과 예외처리를 분리해 각 개념을 **독립적**으로 살펴보고 이해할 수 있다.





### 미확인 Unchecked 예외를 사용하라

초창기에는 메소드를 선언할때 메소드가 반환할 수있는 모든 예외를 열거했다

```swift
enum AppError: Error { 
  case invalidSelection 
  case insufficientFunds(coinsNeeded: Int) 
  case outOfStock 
}
```

하지만, 확인된 예외는 OCP(open close principle)을 위반한다.



unchecked 예외 

배열에서 존재하지 않는 인덱스에 접근하거나 어떤 수를 0으로 나눌 때 발생하는 오류가 미확인 예외다

즉, 컴파일 단계에서 확인이 불가능하며 실행 단계에서 확인되 명시적인 처리를 강제하지 않는 예외이다.



OCP 위반 예제 

1. 단순 출력을 하는 메소드

   ```swift
   func printA(flag: Bool) {
     	if flag {
         printf("called")
       }
   }
   
   func callFunc(flag: Bool) {
     printA(flag)
   }
   ```

   

2. 프린트를 안할 때 에러 던지기로 수정

   ```swift
   func printA(flag: Bool) throws {
     	if flag {
         printf("called")
       } else {
       	throw PrintErr.notCalled
       }
   }
   
   func callFunc(flag: Bool) {
     do { 
       try printA(flag)
     } catch PrintErr.notCalled{
       print("notCalled")
     }
   }
   ```



예외를 추가함으로써 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다. 이것은 명백히 Open close principle을 위반한다.



개방-폐쇄 원칙은 시스템의 구조를 올바르게 재조직(리팩토링)하여 나중에 이와 같은 유형의 변경이 더 이상의 수정을 유발하지 않도록 하는 것이다. 개방-폐쇄 원칙이 잘 적용되면, 기능을 추가하거나 변경해야 할 때 이미 제대로 동작하고 있던 원래 코드를 변경하지 않아도, 기존의 코드에 새로운 코드를 추가함으로써 기능의 추가나 변경이 가능하다.



### 예외에 의미를 제공하라

오류가 발생한 원인과 위치를 찾기 쉽도록 오류메시지에 충분한 정보를 담아 예외와 함께 던진다





### 호출자를 고려해 예외 클래스를 정의하라

오류를 올바르게 분류해야한다.

아래의 코드는 오류를 형편없이 분류한 사례다. 외부라이브러리가 던질 예외를 모두 호출자에서 처리한다.

```swift
var port = ACMEPort(portNumber: 12)

do {
  try port.open()
} catch DeviceResponseExcepion {
  os_log("Device respone exception")
} catch ATM1212UnlockedExceoption {
  os_log("Unlocked exception")
} catch GMXError {
  os_log("GMXError")
}
```

위 코드는 예외에 대응하는 방식이 예외 유형과 무관하게 일정하다.

호출하는 API를 감싸서 예외 유향 하나를 반환하면 된다

```swift
var port = LocalPort(portNumber: 12)

do { 
  try port.open()
} catch PortDeviceFailure {
  os_log(PortDeviceFailure)
}


class LocalPort {
  private innerPort: ACMEPort
  
  init(portNumber: Int) {
    self.innerPort = portNumber
  }
  
  func open() throws{
    do {
 			 try innerPort.open()
		} catch DeviceResponseExcepion {
 			 throw PortDeviceFailure.DeviceResponseExcepion
		} catch ATM1212UnlockedExceoption {
  		throw PortDeviceFailure.ATM1212UnlockedExceoption
		} catch GMXError {
  		throw PortDeviceFailure.GMXError
		}
  }
}
```

Localport 클래스처럼 ACMEPort를 감싸는 클래스는 매우 유용하다

외부 API를 감싸는 장점

* 에러처리 간결
* 외부 라이브러리와의 의존성 감소
* 테스트 편리
* 외부 라이브러리 설계 방식에대한 의존성 감소





### nil을 반환하지 마라

```swift
func registerItem(item: Item) {
  if item != nil {
    var registry = peristenStore.getItemRegistry()
    if registry != nil {
      var existing = registry.getItem(item.getId())
      if existing.getBillingPeriod().hasRetailOwner() {
        existing.register(item)
      }
    }
  }
}
```

nil을 반환하는 코드는 일거리를 늘릴뿐만 아니라 호출자에게 문제를 떠넘긴다.

하나라도 nil 확인을 빼먹는다면 프로그램은 종료될 지도 모른다

nil을 반환하고싶은 유혹이든다면 그대신 예외를 던지거나 특수 사례 객체를 반환하는 것 이좋다





### 결론

오류 처리를 프로그램 로직과 분리해 작성하면 가독성을 높이고 깨끗한 코드를 작성할 수 있다





















