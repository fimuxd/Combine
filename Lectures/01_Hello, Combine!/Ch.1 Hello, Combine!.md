# Ch.1: Hello, Combine!

Apple은 Combine에 대해 다음과 같이 설명합니다. 

> Combine은 앱이 이벤트를 처리하는 방법에 대해 선언적 접근 방식을 제공한다. 여러 델리게이트 콜백 또는 completion handler closure를 구현하는 대신 주어진 이벤트 소스에 대해 single processing chain을 작성할 수 있다. chain의 각 부분은 이전 단계에서 받은 요소에 대해 어떠한 조치를 취하는 결합연산자들을 의미한다. 

명확한 설명이지만 처음에는 추상적으로 들릴 수 있습니다. 하나씩 살펴보면서 최종적으로 Combine으로 작성된 앱을 완성해봅시다.

## A. 비동기 프로그래밍
 단일 쓰레드 언어에서는 한 줄씩 순차적으로 코드가 실행됩니다. 예를 들면 다음과 같은 코드입니다.
 
 ```swift
begin
	var name = "Bo-Young"
	print(name)
	name += " Park"
	print(name)
end
 ```

동기식 코드는 이해하기 쉽습니다. 특히 data의 상태를 설명하기에 쉽죠. 단일 쓰레드로 실행하면 항상 현재의 데이터 상태를 확인할 수 있습니다. 위 코드는 항상 "Bo-Young" 을 print 하고 " Park" 을 print 하겠죠. 

그렇다면 멀티 쓰레드 언어를 작성한다고 생각해봅시다. UIKit과 Swift로 작성된 iOS 앱과 같이 비동기 이벤트 중심 UI framework를 실행하는 것처럼요. 다음과 같은 코드를 살펴보죠.

```swift
--- Thread 1 ---
begin
  var name = "Tom"
  print(name)

--- Thread 2 ---
name = "Billy Bob"

--- Thread 1 ---
  name += " Harding"
  print(name)
end
```

이렇게 `name` 에 `"Tom"` 이라는 값을 주고 그 다음에 `"Harding"` 을 더해줬습니다. 이전과 똑같아요. 하지만 동시에 동작하는 다른 쓰레드가 있기 때문에 `name` 값이 `"Billy Bob"` 으로 설정될 수도 있습니다. 
코드가 다른 코어에서 동시에 실행될 때는 코드의 어떤 부분이 먼저 수정되거나 실행될지 알 수 없습니다. 위 예제의 Thread 2 는 다음과 같이 실행됩니다.

- 원래 코드와 다른 CPU 코어에서 정확히 동시에 실행
- `name += "Harding"` 직전에 실행하므로 원래 값 `"Tom"` 대신 `"Billy Bob"` 이 표시

이 코드를 실행할 때 발생하는 것은 시스템 로드 상태에 따라 다르며 프로그램을 실행할 때마다 다른 결과가 나타날 수 있습니다. 비동기적으로 동시에 실행되는 코드를 실행하는 순간 변경 가능한 상태 관리는 필수적입니다. 

## B. Foundation, UIKit/AppKit
Apple은 매 해를 거듭하며 비동기 프로그래밍 방식을 개선했습니다. 우리 대부분은 이미 그 코드들을 사용하고 있습니다. 주로 쓰는 건 다음과 같을거예요.

- **`NotificationCenter`**: 사용자가 기기 방향을 변경하거나, 소프트웨어 키보드가 화면에 표시/숨김 처리될 때와 같이 특정 이벤트가 발생할 때마다 코드를 실행합니다. 
- **델리게이트 패턴**: 다른 object를 대신하여 또는 함께 작동하는 object를 정의할 수 있습니다. 예를 들어 AppDelegate 에 새로운 remote notification이 도착하면 어떻게 작동할 것인지 정의할 수 있지만 이 코드가 언제 실행될지, 몇 번 실행될지는 전혀 알 수 없습니다. 
- **Grand Central Dispatch, Operations**: 수행할 작업을 추상화하는데 도움이 됩니다. 이를 사용하여 코드가 순차적으로 실행되도록 예약하거나 우선 순위가 다른 여러 대기열에서 동시에 여러 작업을 실행하도록 할 수 있습니다. 
- **Closures**: 전달가능한 코드 뭉치를 만들어서 다른 object가 해당 코드를 실행할지 여부, 횟수 및 컨텍스트를 결정할 수 있습니다.

일반적으로 대부분의 코드는 작업을 비동기식으로 수행하고 모든 UI 이벤트는 본질적으로 비동기적이므로 전체 앱 코드가 실행될 순서를 가정하는 것은 사실상 불가능합니다. 또한 괜찮은 비동기식 프로그램 작성은 쉽지 않습니다. 비동기 코드와 리소스 공유는 재현이나 추적이 어렵고 궁극적으로 수정하기도 어려운 문제를 일으킬 수 있습니다. 이런 문제의 원인은 실제 앱이 각각의 고유한 인터페이스를 가지는 비동기 API들을 사용하는데 있습니다. 

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/1.%20asynchronous.png" width = 400>

Combine은 이렇게 혼란스러운 비동기 프로그래밍 세계를 질서정연하게 정리하는데 도움이 되는 새로운 언어를 Swift 생태계에 도입하는 녀석입니다. Apple은 Combine의 API를 Foundation framework 깊숙히 통합시켰기 때문에 `Timer`, `NotificationCenter`, core framework와 같은 **Core Data** 들은 이미 Combine을 사용하고 있습니다. 따라서 Combine을 기존의 코드에 도입하는 것은 아주 쉬울거예요. 또한 Apple은 놀랍고 새로운 UI framework인  **SwiftUI** 를 설계하여 Combine과 쉽게 통합되도록 했습니다. 여기 Apple이 Combine을 사용하여 반응형 프로그래밍을 사용하는 방법을 알 수 있도록 시스템 계층 구조에서 Combine이 어디에 있는지 보여주는 다이어그램이 있습니다. 

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/2.%20combine.png" width = 400>

Foundation에서 SwiftUI에 이르기까지 다양한 시스템 framework는 Combine에 의존하며 "전통적인" API의 대안으로 Combine 으로의 통합을 제공합니다. Combine은 Apple의 framework이므로 `Timer` 또는 `NotificationCenter` 와 같이 이미 테스트로 검증되고 잘 만들어져있는 API를 제거하고 대체하는 것을 목표로 하지 않습니다. 대신 Combine 으로 통합되어 서로 비동기식으로 커뮤니케이션하려는 앱의 모든 유형이 Combine을 사용할 수 있도록 합니다. 

## C. Combine의 근간
선언형, 반응형 프로그래밍은 새로운 개념이 아닙니다. 꽤 오래전부터 있던 개념이지만 지난 10년 동안 급부상한 개념입니다. 2009년 Microsoft 팀이 .NET(Rx.NET) 용 Reactive Extensions 라는 라이브러리를 시작하였고, 이를 2012년에 오픈 소스로 만들었습니다. 그 이후로 많은 언어가 그 개념을 사용하기 시작했습니다. 현재 RxJS, RxKotlin, RxScala, RxPHP 등과 같은 많은 Rx 표준이 있습니다. Apple 플랫폼에는 Rx표준을 구현하는 RxSwift와 같은 제3자 반응형 framework가 있습니다.
Combine은 Rx와 다르지만 Reactive Stream과 유사한 표준을 구현합니다. Reactive Stream은 Rx와 몇 가지 주요한 차이점이 있지만 둘 다 대부분의 핵심 개념이 유사합니다. 지금까지 위에서 언급한 Rx framework를 사용하지 않았더라도 걱정할 필요 없습니다. 지금까지 반응형 프로그래밍은 Apple 플랫폼, 특히 Swift에서 주요 개념이 아니었거든요. 
그러나 Apple은 iOS 13/macOS Catalina 부터 내장 시스템 framework인 Combine을 통해 반응형 프로그래밍 지원을 Apple 생태계에 하게 되었습니다. iOS 13/macOS Catalina 이상을 지원하는 앱에서만 Combine을 사용할 수 있는 제약은 있지만, 이는 Apple의 다른 기술들처럼 빠르게 지원이 확산되고 최종적으로는 Combine 기술에 대한 수요가 급증하게 될 것 입니다. 

## D. Combine 기초
Combine의 세 가지 핵심 요소는 Publisher, Subscriber, Operator 입니다. Ch.2 Publisher & Subscriber 에서 Publisher와 Subscriber에 대해 자세히 배우고 Section 2에서는 Operator들을 다룰 것입니다. 여기서는 각각의 타입에 대한 목적과 역할 등 일반적인 내용을 훑어보도록 하겠습니다. 

### 1. Publishers
- Publishers는 Subscribers와 같이 하나 이상의 대상에게 시간이 지남에 따라 값을 방출할 수 있는 유형입니다. 
- 수학 계산, 네트워킹, 사용자 이벤트 처리 등 모든 Publishers는 여러 이벤트들을 다음 세 가지 유형으로 생성할 수 있습니다.
	1. Publishers의 generic 유형인 `Output`
	2. 성공적인 완료
	3. error와 함께 완료. Publishers의 `Failure`
- Publishers는 0개 이상의 output 값을 가질 수 있으며, 성공 또는 실패로 인해 완료되면 더 이상 이벤트를 생성하지 않습니다. 

다음은 `Int` 값을 방출하는 Publishers를 타임 라인에 시각화한 것입니다. 

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/3.%20publishers.png" width = 400>

- 위 그림에서 파란색 상자는 타임 라인에서 특정 시간에 방출되는 값을 나타내고 숫자는 방출된 값을 나타냅니다. 그림 오른쪽에 보이는 것과 같은 수직선(|)은 성공적으로 스트림이 완료되었다는 것을 의미합니다. 
- 위 Publisher가 다룰 수 있는 이벤트 유형은 매우 보편적이기 때문에 모든 종류의 동적 데이터를 나타낼 수 있습니다. 즉, Combine의 Publishers를 사용하여 앱의 모든 작업을 처리할 수 있는 것입니다. 델리게이트를 추가하거나 completion callback을 주입하는 대신 Publishers를 사용할 수 있습니다. 
- Publishers의 가장 좋은 기능 중 하나는 error 처리 기능이 내장되어 있다는 것입니다. error 처리를 원하는 경우 마지막에 선택적으로 하지 않게 설계되어 있습니다. 
- 위 그림을 통해 알 수 있듯이 Publishers protocol은 두 가지 유형을 가집니다.
	- `Publisher.Output`은 publisher의 방출 값입니다. 만약 publisher가 `Int`에 특화되었다면 `String` 이나 `Date` 타입을 방출할 수는 없습니다. 
	- `Publisher.Failure`는 publisher가 뱉을 수 있는 error 타입입니다. 만약 publisher가 절대 실패하지 않는다면, 이를 `Never` 라는 실패 타입으로 표현할 수 있습니다.
- 띠라서 주어진 publisher를 구독할 때 어떤 값이 떨어질지, 실패한다면 어떤 에러가 떨어질지 예상 가능합니다.

### 2. Operators
- Operators는 `Publisher` protocol로 선언된 method이며, 선언된 Publishers와 동일하거나 새로운 Publishers로 반환합니다.
- 여러 Operator를 차례로 호출해서 체인을 연결할 수 있기 때문에 매우 유용합니다. 이들은 매우 독립적으로 구성가능하기 때문에 하나의 구독 사이에서 아주 복잡한 로직을 구현하도록 서로 결합될 수 있습니다.
- Operator가 퍼즐 조각처럼 서로 잘 맞도록 하는 방법은 간단합니다. Output이 다음 Input 유형과 일치하지 않으면 결합할 수 없습니다.

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/4.%20operator.png" width = 400>

- Operator는 항상 Input 및 Output을 가지고 있으며 일반적으로는 이를 upstream, downstream이라고 합니다. 이를 통해 공유 상태 (앞서 논의한 비동기적 프로그래밍의 핵심 문제 중 하나)를 피할 수 있습니다.
- Operator는 이전 operator로부터 받은 데이터 작업에 중점을 두고 그 결과를 체인의 다음 operator에게 제공합니다. 즉, 비동기적으로 실행되는 다른 코드는 작업 중인 데이터를 "건너뛰고" 변경할 수 없습니다. 

### 3. Subscribers
- 모든 구독은 subscriber와 함께 종료됩니다. Subscribers는 일반적으로 방출된 값 또는 완료 이벤트를 통해 "무언가"를 하는 역할입니다.

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/5.%20subscriber.png" width = 400>

- 현재, Combine은 데이터 스트림 작업을 간단하게 해줄 수 있는 두 개의 빌트인 subscriber를 제공하고 있습니다.
	- **sink** subscriber는 output 값과 완료를 수신할 수 있는 closure를 제공합니다. 
	- **assign** subscriber는 개발자가 별도 작성한 코드가 없어도 결과 output을 데이터 모델 또는 UI control의 특정 속성에 바인딩하여 키 경로를 통해 화면에 직접 데이터를 표시할 수 있게 합니다. 

- 데이터에 별도의 요구 사항이 있는 경우 subscriber를 커스터마이징 하는 것이 publisher를 커스터마이징 하는 것보다 훨씬 쉽습니다. Combine은 매우 간단한 protocol 세트를 제공하기 때문에 기본적으로 제공하는 도구 중 적합한 것을 찾을 수 없을 때마다 커스터마이징한 도구를 사용할 수 있게 해줍니다. *(무슨 말이지? extension Reactive 같은거 말하는건가)*

### 4. Subscriptions
> **Note**: 여기서는 **subscription** 이라는 용어를 Combine의 `Subscription` protocol 뿐만 아니라 해당 object, publisher, operator, subscriber 전체 체인을 의미하는 용도로도 사용합니다.

- subscription(구독)이 끝날 때 subscriber를 추가하면 체인이 시작될 때까지 publisher가 "활성화" 됩니다. Output을 수신할 subscriber가 없다면 publisher는 값을 방출하지 않습니다.
- subscription은 사용자 정의 코드 및 에러처리를 사용하여 비동기 이벤트 체인을 한 번만 선언할 수 있다는 점에서 아주 좋은 개념입니다.
- 즉, 모든 코드를 Combine으로 구현한다고 상상해보면, subscription을 통해 전체 앱의 로직을 구현하고, 한번 구현이 완료되면 시스템이 데이터를 푸시하거나 가져오거나 object 또는 다른 object를 호출할 필요 없이 모든 것이 실행되도록 할 수 있습니다. (😯)

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/6.%20subscription.png" width = 400>

- subscription 코드가 성공적으로 컴파일되고 작성한 코드에 논리적인 문제가 없다면 - 끝! 설계를 한대로 subscription은 사용자의 제스처, 타이머가 꺼지거나 다른 항목이 publisher 중 하나를 활성화 시킬 때 마다 비동기식으로 "실행" 될 것입니다.
- 또한 `Cancellable`이라는 Combine에서 제공하는 protocol 덕분에 subscription을 메모리로 관리할 필요가 없습니다.
- 시스템에서 제공하는 두 subscriber 모두 `Cancellable` protocol 을 준수합니다. 즉, subscription 코드 (예: 전체 publisher, operator, subscriber 호출 체인) 가 `Cancellable` object를 반환합니다. 해당 object를 메모리에서 해제할 때마다 전체 subscription이 취소되고 해당 리소스가 메모리에서 해제됩니다.
- 즉, subscription을 UIViewController 속성에 저장하여 subscription 수명을 쉽게 "바인드" 할 수 있습니다. 이렇게 하면 사용자가 view 스택에서 UIViewController를 닫을 때마다 해당 속성이 초기화되지 않고 subscription도 취소됩니다. (*몬말이징..어떻게 한다는거지*) 또는 이 프로세스를 자동화하기 위해 `Set<[AnyCancellable]>` 프로퍼티를 설정하고 원하는 만큼 subscription을 많이 넣을 수도 있습니다. 이 프로퍼티가 메모리에서 해제되면 그 안의 subscription들도 모두 자동으로 취소 및 해제됩니다.

## E. "일반(standard)" 코드 대비 Combine 코드의 장점은?
*너무 당연한 말이지만* Combine 없이도 최고의 앱을 만들 수 있습니다. 다만 Combine을 사용하는 것은 Core Data, `URLSession`, UIKit과 같은 추상화를 직접 작성하는 것보다 편리하고 안전하며 효율적입니다.
- Combine은 비동기 코드에 다른 추상화를 추가하는 것을 목표로 합니다. 시스템 수준의 또 다른 추상화 수준은 테스트를 긴밀하게 연결될 수 있고 오래 지속 가능하며 안전한 기술임을 의미합니다. 
- 물론 Combine이 각자의 프로젝트에 적합한지 여부를 판단하고 결정하는 것은 개발자 각자의 몫입니다. 
- Combine을 채택하면 다음과 같은 혜택이 있습니다.
	- Combine은 시스템 레벨에서 통합되었습니다. 즉, Combine 자체는 공개적으로 사용할 수 없는 언어 기능을 사용하기 때문에 사용자가 직접 조작할 수 없는 API 형태로 제공됩니다.
	- 델리게이트, `IBAction`, closure를 통한 "이전" 스타일의 비동기 코드는 처리해야 할 버튼 등에 대해 사용자가 직접 코드를 작성하도록 하고 이는 테스트할 양이 많아진다는 의미가 됩니다. Combine은 모든 비동기 연산을 추상화하여 이미 테스트로 검증된 "operator" 를 제공합니다.
	- 모든 비동기 작업이 동일한 인터페이스인 `Publisher`를 사용하여 이루어진다면 구성과 재사용성이 아주 강력해집니다.
	- Combine의 operator는 쉽게 구성될 수 있습니다. 새로운 operator를 만든다면 나머지 Combine 코드와 즉시 결합되고 사용될 수 있을 것입니다.
	- 비동기 코드 테스트는 일반적으로 동기 코드 테스트 보다 복잡합니다. 그러나 Combine을 사용하면 비동기 연산자가 이미 테스트 되어 있기 때문에 비즈니스 로직만 테스트 하면 됩니다. 즉, subscription이 예상 결과를 출력하는지 여부를 입력하고 테스트하면 됩니다.
- 이와 같이 대부분의 장점은 안전성과 편의성에 있습니다. framework가 Apple에서 제공된다는 사실을 생각한다면 Combine 코드 작성에 시간 투자할만하쥬?

## F. 앱 구조
- Combine은 앱 구조에 영향을 주는 framework가 아닙니다. MVC, MVVM, VIPER 등 어디서든 사용할 수 있습니다.
- 코드에서 개선하려는 부분에서만 Combine 코드를 선택적으로 추가할 수 있으며 도 아니면 모 식의 선택을 하지 않아도 괜찮습니다. 데이터 모델을 변환하거나 네트워킹 계층을 조정하거나 기존 기능을 그대로 유지하면서 앱에 추가한 새 코드에서만 Combine을 사용하여 시작할 수 있습니다. 
- Combine과 SwiftUI를 동시에 채택하면 약간 다른 이야기가 될 수 있습니다. 이 경우 MVC 아키텍처에서 C를 삭제하는 것이 좋습니다만 이는 Combine과 SwiftUI를 함께 사용하기 때문에 요구되는 것입니다. 이 둘은 같은 공간에 있을 때 더욱 간단하게 작동합니다.
- View Controller는 Combine/SwiftUI 팀에 필요없습니다. 데이터 모델에서 View에 이르기까지 반응형 프로그래밍을 사용하는 경우 View를 제어하기 위해 특별한 Controller가 필요하지 않습니다. (✨)

<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/7.%20architecture.png" width = 400>

- 더 자세한 내용은 Ch.15 In Practice: SwiftUI & Combine에서 다루겠습니다.

## G. Summary
- Combine은 시간이 지남에 따라 비동기 이벤트를 처리하기 위한 선언적이고 반응적인 framework입니다.
- 비동기 프로그래밍 도구를 통합하고 변경가능한 상태를 처리하고 에러 처리와 같은 기존 문제를 해결하는 것을 목표로 합니다.
- Combine은 시간이 지남에 따라 이벤트를 생성하는 Publisher, upstream 이벤트를 비동기적으로 처리하고 조작하는 operator, subscriber가 결과를 활용하고 작업을 수행하는 세 가지 주요 유형을 중심으로 활용됩니다.

## H. Tips
- 우리는 이렇게 개념부터 시작해서 여러 연산자들을 배우고 시험해볼 것입니다. 다른 system framework와 달리 Xcode playground 를 이용하면 각 chapter를 진행하면서 결과도 즉시 확인할 수 있습니다. (*playground 로 reactive 미쳤다리*)

***
##### Artwork/images/designs: from Combine: Asynchronous Programming with Swift, available at www.raywenderlich.com
