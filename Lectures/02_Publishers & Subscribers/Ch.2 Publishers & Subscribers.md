# Ch.2: Hello, Publishers & Subscribers

## A. 시작하기
- *ePub을 구매하시면 예제파일을 제공합니다. 제 발번역은 구매 후 참고용, 서브로 보시는 것을 추천드립니다. ([구매하기](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift))*

## B. Hello Publisher
- Combine의 핵심은 `Publisher` protocol 입니다. 이 protocol은 시간이 지남에 따라 하나 이상의 Subscriber에게 일련의 값을 전송할 수 있는 유형으로 어떤 구성요소를 가지는지 나타내고 있습니다. 즉, Publisher는 특정 값을 가지는 이벤트를 생성합니다.
- 이전에 Apple platform에서 개발한 적이 있다면 Publisher를 `NotificationCenter`와 같은 놈이라고 생각할 수 있습니다. 실제로 `NotificationCenter`에는 실제로 broadcast 알림을 게시할 수 있는 `Publisher` 타입의 `publisher(for:object:)` 라는 method가 있습니다.
- 다음 코드를 살펴봅시다.
	
	```swift
	example(of: "Publisher") {
  	// 1
  	let myNotification = Notification.Name("MyNotification")

  	// 2
  	let publisher = NotificationCenter.default
    	.publisher(for: myNotification, object: nil)
	}
	```

	- 1: notification의 이름을 생성했습니다.
	
	- 2: `NotificationCenter`의 default center에 접근하고 `publisher(for:object:)` method 를 호출합니다. 이 method의 return 값을 내부상수로 선언하였습니다. 

	- `publisher(for:object:)`를 **Option-click** 해보면 이 method의 return 값이 default notification center가 notification을 broadcast 할 때마다 이벤트를 방출하는 `Publisher` 인 것을 알 수 있습니다. 이미 notification center에서 publisher 없이 notification을 broadcast 할 수 있는 것 같지 않나요? 
	- 바로 `NotificationCenter`와 같은 기존 API를 통해 이전 방법에서 Combine과 같은 새로운 방법으로 연결될 수 있습니다. 
- Publisher는 두 종류의 이벤트를 방출합니다.
	- 1: 값, 요소(element) 라고도 불리는 아이
	- 2:  완료 이벤트
- publisher는 0개 이상의 값을 방출할 수 있지만 일반적인 완료 또는 에러로 나타날 수 있는 완료 이벤트는 단 한개만 방출합니다. 한 번 publisher가 완료 이벤트를 방출하고 나면 종료되어서 더 이상의 이벤트 방출을 발생하지 않습니다.
- 위 코드를 좀 더 진행시켜 보겠습니다.

	```swift
	// 3
	let center = NotificationCenter.default

	// 4
	let observer = center.addObserver(
  		forName: myNotification,
  		object: nil,
  		queue: nil) { notification in
    		print("Notification received!")
		}

	// 5
	center.post(name: myNotification, object: nil)

	// 6
	center.removeObserver(observer)
	```

	주석 번호대로 따라가보면,

	- 3: default notification center를 핸들링 할 수 있도록 상수를 하나 만들어줬습니다.
	- 4: 위 1번에서 이름을 만들어준 notification을 볼 수 있도록 observer를 만들어줍니다.
	- 5: notificationd을 만들어준 이름과 같이 post 합니다.
	- 6: observer를 notification center에서 제거합니다.

	- 위 코드를 playground에서 실행시킨다면 콘솔에 `Notification received!` 가 입력되는 것을 확인할 수 있습니다.
	- 다만 위 코드 예제에서는 outputd이 실제로 publisher를 통해 나온 것이 아니기 때문에 약간 오해의 소지가 있습니다. 이를 완전히 이해하려면 subscriber를 알아야 합니다.

## C. Hello Subscriber
- `Subscriber` protocol은 publisher로부터 받을 수 있는 값을 정의합니다. 기본 흐름에 중점을 두고 다음 예제 코드를 보겠습니다.
	```swift
	example(of: "Subscriber") {
  	let myNotification = Notification.Name("MyNotification")

  	let publisher = NotificationCenter.default
    	.publisher(for: myNotification, object: nil)

  	let center = NotificationCenter.default
	}
	```

	- 만약 notification을 지금 post 한다면, publisher는 해당 값을 방출하지 않을 것입니다. 중요한 부분인데요, publisher는 최소 한 개 이상의 subscriber만 가질 때 값을 방출합니다. 

### 1. Subscribing with `sink(_:_:)`
- 위 코드 예제에 이어서 subscription을 publisher에 생성하는 방법을 보겠습니다.
	```swift
	let subscription = publisher
  		.sink { _ in
    		print("Notification received from a publisher!")
  		}

	```

	- 위 코드에서 `sink` 라는 subscription을 publisher에 생성해주었습니다. `sink` 를 **Option-click** 해보면 subscriber에게 closure를 연결하여 publisher의 output을 처리하는 방법을 아주 쉽게 제공함을 알 수 있습니다. 위 코드에서는 closure를 무시하고 대신 notification을 받았을 때 print 만 찍도록 했습니다.
	- 위 코드를 돌려보면 `Notification received from a publisher!` 라는 문구가 콘솔에 표시됩니다.
- `sink` operator는 publisher가 방출하는 값들을 계속해서 받을 것입니다. 이 것을 무제한 수요*unlimited demand* 라고 하며 이후 별도로 다루게 될 것입니다. 
- 방금의 예제 코드에서는 무시했지만 `sinK` 연산자느 실제로 두 개의 closure를 제공합니다. 하나는 완료 이벤트 수신을 처리하고 다른 하나는 값 수신을 처리하게 됩니다. 다음 코드를 함께 봅시다.
	```swift
	example(of: "Just") {
  	// 1
  	let just = Just("Hello world!")
	  
  	// 2
  	_ = just
    	.sink(
      		receiveCompletion: {
        		print("Received completion", $0)
      		},
      		receiveValue: {
        		print("Received value", $0)
    		}
    	)
	}
	```
 
	- 1: 기본 값 형식으로 publisher를 만들 수 있는 `Just`를 이용하여 publisher를 생성합니다. 
	- 2: publisher에 대한 subscription을 작성하고 수신된 각 이벤트에 대해 print가 찍히도록 했습니다.

	- playground을 실행하면 다음과 같이 표시됩니다.
	
	```
	——— Example of: Just ———
	Received value Hello world!
	Received completion finished
	```

	- `Just`를 **Option-click** 해보면 각 subscriber에게 output을 한 번 방출한 후 완료되는 publisher라고 설명하고 있습니다. 그럼 다음 코드를 추가하면 어떻게 출력될까요?

	```swift
	_ = just
  		.sink(
    		receiveCompletion: {
      			print("Received completion (another)", $0)
    		},
    		receiveValue: {
      			print("Received value (another)", $0)
  			}
  		)
	```

	- `Just`는 각각의 새로운 subscriber에게 정확히 한 번 output을 내보낸 후 종료됩니다. 따라서 두 번의 subscription이 있었으므로 print도 추가적으로 찍히게 됩니다.
		```
		Received value Hello world!
		Received completion finished
		Received value (another) Hello world!
		Received completion (another) finished
		```

### 2. Subscribing with `assign(to:on:)`
- `sink`와 더불어 `assign(to:on:)` 내장 operator를 사용하면 받은 값을 object의 KVO 호환 property에 할당할 수 있습니다. 다음 예제 코드를 살펴봅시다.

	```swift
	example(of: "assign(to:on:)") {
  	// 1
  	class SomeObject {
    	var value: String = "" {
      		didSet {
        		print(value)
      		}
    	}
  	}
  	
  	// 2
  	let object = SomeObject()
  	
  	// 3
  	let publisher = ["Hello", "world!"].publisher
  	
  	// 4
  	_ = publisher
    	.assign(to: \.value, on: object)
	}
	```

	- 1: 새 값을 print 하는 `didSet` property Observer와 class를 작성합니다.
	- 2: 해당 class instance를 선언합니다.
	- 3: String 배열을 통해 publisher를 생성합니다.
	- 4: object를 통해 발생하는 각각의 값을 `value` 속성에 할당해서 publisher를 구독합니다.

	- 콘솔에는 다음과 같이 표시될 것입니다.
		```
		——— Example of: assign(to:on:) ———
		Hello
		world!
		```

- 지금은 `sink` operator를 사용하는데 중점을 둘 것이며, 이후 Ch.8 In Practice: Project "Collage" 에서의 실습을 통해 `assign`을 더 살펴볼 것입니다.

## D. Hello Cancellable
- subscriber가 완료되고 더 이상 publisher로부터 값을 받을 수 없는 경우 subscription을 취소하여 리소스를 확보하고 network call이 계속 발생하지 않도록 해야합니다.
- subscription은 `AnyCancellable` 의 instance를 "취소 토큰"으로 반환하기 때문에 subscription이 완료되면 subscription을 취소 할 수 있습니다. 
- `AnyCancellable`은 `Cancellable` protocol을 따르며, 이를 위해서는 `cancel()` method가 필요합니다. 
- 앞서 작성했던 **Subscriber** 예제 코드에 다음 코드를 추가하여 마무리 해봅시다.

	```swift
	// 1
	center.post(name: myNotification, object: nil)
	
	// 2
	subscription.cancel()
	```

	- 1: (원래와 동일하게) notification을 post 합니다.
	- 2: subscription을 취소합니다. subscription에 `cancel()` 호출을 할 수 있는데 이 것은 `Subscription` protocol이 `Cancellable` 을 상속했기 때문입니다.

	- 콘솔에는 다음과 같이 표시될 것입니다.
		```
		——— Example of: Subscriber ———
		Notification received from a publisher!’
		```

- subscription에서 `cancel()`을 명시적으로 호출하지 않으면 publisher가 완료될 때까지 또는 일반적인 메모리 관리로 인해 저장된 subscription이 초기화되지 않을 때까지 계속됩니다. 

## E. 흐름 이해하기
- 백문이 불여일견이라고, publisher와 subscriber간의 상호 작용을 설명하기 위해 다음 그림을 참고해보겠습니다. 
	
	<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/02_Publishers%20&%20Subscribers/1.%20flow.png?raw=true" width = 400>

	- 1: subscriber가 publisher로 subscribe 됩니다.
	- 2: publisher가 subscription을 생성하고 subscriber에게 전달합니다.
	- 3: subscriber는 값을 요청합니다.
	- 4: publisher가 값을 전달합니다.
	- 5: publisher가 완료를 전송합니다.

- `Publisher` 와 이 protocol의 매우 중요한 extension 을 살펴보겠습니다. 

	```swift
	public protocol Publisher {
  		// 1
  		associatedtype Output

  		// 2
  		associatedtype Failure : Error

  		// 4
  		func receive<S>(subscriber: S)
    		where S: Subscriber,
    		Self.Failure == S.Failure,
    		Self.Output == S.Input
	}
	
	extension Publisher {
  		// 3
  		public func subscribe<S>(_ subscriber: S)
    		where S : Subscriber,
    		Self.Failure == S.Failure,
    		Self.Output == S.Input
	}
	```

	- 1: publisher가 생성할 수 있는 값 타입
	- 2: publisher가 발생시킬 수도 있는 error 타입. 만약 error를 발생시키지 않는다고 보장할 수 있다면 `Never`.
	- 3: subscriber는 publisher의 `subscribe(_:)` 을 호출할 수 있습니다.
	- 4: `subscribe(_:)` 구현체는 `receive(subscriber:)`를 호출하여 subscriber를 publisher에 연결합니다. 즉, subscription을 만듭니다.

	- associated type은 subscription을 만드려면 subscriber가 반드시 일치시켜야하는 publisher의 인터페이스 입니다.

- 이제 `Subscriber` protocol을 살펴보겠습니다.

	```swift
	public protocol Subscriber: CustomCombineIdentifierConvertible {
  		// 1
  		associatedtype Input

  		// 2
  		associatedtype Failure: Error

  		// 3
  		func receive(subscription: Subscription)

  		// 4
  		func receive(_ input: Self.Input) -> Subscribers.Demand

  		// 5
  		func receive(completion: Subscribers.Completion<Self.Failure>)
	}
	```

	- 1: subscriber가 받을 수 있는 값 타입
	- 2: subscriber가 받을 수 있는 error 타입 또는 subscriber가 error를 받지 않는다면 `Never`
	- 3: publisher가 subscription을 전달하기 위해 subscriber의 `receive(subscription:)`을 호출합니다.
	- 4: publisher가 방출하는 새로운 값들을 전달하기 위해 subscriber의 `receive(_:)`를 호출합니다.
	- 5: publisher가 값 생성이 종료되었거나 error가 발생하였을 때 종료를 알리기 위해 subscriber의 `receive(completion:)`을 호출합니다.

- publisher와 subscriber는 subscription을 통해 연결됩니다. 아래의 `Subscription` protocol을 확인해봅시다.

	```swift
	public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
  		func request(_ demand: Subscribers.Demand)
	}
	```

	- subscriber는 `request(_:)`을 호출하여 최대 또는 무제한의 값을 받을 수 있음을 나타낼 수 있습니다.

> **Note**: subscriber가 받을 수 있는 값의 수를 나타내는 subscriber 개념은 **backpressure 관리** 라고 합니다. 이런 개념이 없으면 subscriber는 처리할 수 있는 것보다 많은 값을 publisher로부터 받아서 문제가 될 수 있습니다. Backpressure에 대해서는 Ch.18 Custom Publisher & Handling Backpressure 에 대해서 더 자세히 다룰 것입니다.

- `Subscriber` 에서는 `receive(_:)`가 `Demand`를 반환합니다. `receive(_:)`에서 `subscription.request(_:)`을 처음 호출할 때 subscriber가 받으려는 최대 값 수를 지정하더라도 새 값을 받을 때마다 최대 값을 조절할 수 있다는 의미입니다.

> **Note**: `Subscriber.receive(_:` 에서 `max` 를 조정하여 최대 값을 조절할 수 있습니다. 즉, 새 `max` 값이 현재 `max` 값에 추가됩니다. `max`값은 반드시 양수여야하며 음수를 전달하면 `fatalError`가 발생합니다. 즉, 새 값을 받을 때마다 기존의 `max` 값을 늘릴 수 있지만 줄일 수는 없습니다. 


## F. 사용자 subscriber 만들기
- 지금까지 배운 것을 연습해보겠습니다. 다음과 같이 새로운 예제 코드를 확인해봅시다.

	```swift
	example(of: "Custom Subscriber") {
  		// 1
  		let publisher = (1...6).publisher
  		
  		// 2
  		final class IntSubscriber: Subscriber {
    		// 3
    		typealias Input = Int
    		typealias Failure = Never

    		// 4
    		func receive(subscription: Subscription) {
      			subscription.request(.max(3))
    		}
    	
    		// 5
    		func receive(_ input: Int) -> Subscribers.Demand {
      			print("Received value", input)
      			return .none
    		}
    	
    		// 6
    		func receive(completion: Subscribers.Completion<Never>) {
      			print("Received completion", completion)
    		}
  		}
	}
	```

	- 1: 범위의 `Int` 값을 `publisher` property를 통해 publisher 생성
	- 2: `IntSubscriber` 라는 사용자 subscriber 정의
	- 3: type aliases를 통해 subscriber가 `Int` input 및 에러를 생성하지 않음을 명시
	- 4: publisher를 통해 호출된 `receive(subscription:)`를 통해 subscriber가 subscription시 최대 3개의 값을 수신할 것임을 지정하고 subscription의 `.request(_:)` 를 호출합니다.
	- 5: 수신한 각 값을 print 하고 `.none` 을 반환하여 subscriber가 수요를 조정하지 않음을 나타냅니다. 즉, `.none`은 `.max(0)` 과 같은 의미입니다.
	- 6: 완료 이벤트를 print 합니다.

- publisher가 뭐든 방출하려면 subscriber가 필요합니다. 따라서 다음 코드를 추가합니다.

	```swift
	let subscriber = IntSubscriber()

	publisher.subscribe(subscriber)
	```

	- 이 코드를 통해 publisher의 `Output`과 `Failure` 타입과 일치하는 subscriber를 생성하였습니다. 이렇게 생성한 subscriber를 publisher에 붙여주었네요.
	- 콘솔에는 다음과 같이 표현될 것입니다.
		```
		——— Example of: Custom Subscriber ———
		Received value 1
		Received value 2
		Received value 3
		```

	- 완료 이벤트를 받지 않은 것을 확인할 수 있습니다. 왜냐하면 publisher에 한정된 갯수의 값이 있고 `.max(3)`을 통해 수요를 지정했기 때문입니다. 

- 우리가 만든 사용자 subscriber의 `receive(_:)` 에서 `.none()` 을 `.unlimited`로 변경하여 `receive(_:)` method를 다음과 같이 수정합니다.
	```swift
	func receive(_ input: Int) -> Subscribers.Demand {
  		print("Received value", input)
  		return .unlimited
	}
	```

	- 이제 콘솔에는 다음과 같이 표현됩니다.
		```
		——— Example of: Custom Subscriber ———
		Received value 1
		Received value 2
		Received value 3
		Received value 4
		Received value 5
		Received value 6
		Received completion finished
		```

- 이제 다시 `.unlimited`를 `.max(1)`으로 변경하고 playground를 다시 실행한다면 이벤트를 받을 때마다 `max` 값을 1씩 늘리도록 지정한 것이기 때문에 `.unlimited`를 반환했을 때와 동일한 결과가 표시됩니다.
- `.max(1)`을 다시 `.none`으로 변경하고 이번에는 publisher 를 `String` 배열로 변경하겠습니다. 즉 기존의
	
	```swift
	let publisher = (1...6).publisher
	``` 
	
	이 배열을 다음과 같이 변경하는 것입니다.
	
	```swift
	let publisher = ["A", "B", "C", "D", "E", "F"].publisher
	```
	- playground를 실행하면 `subscribe` method에 `String` 및 `IntSubscriber.Input`(`Int`) 이 동일해야 한다는 error가 발생할 것입니다. publisher 의 `Output` 및 `Failure` 유형이 subscriber의 `Input` 및 `Failure` 유형과 일치해야 둘 사이의 subscription을 만들 수 있기 때문입니다. 오류를 해결하려면 `publisher` 정의를 다시 `Int` 범위로 변경해야겠죠.

## G. Hello Future
- subscriber에게 단일 값을 내보내고 완료되는 publisher를 만드는데 `Just`를 사용할 수 있는 것과 같이 `Future`는 단일 결과를 비동기적으로 생성한 다음 완료됩니다. 다음 예제 코드를 작성해보겠습니다.
	```swift
	example(of: "Future") {
  		func futureIncrement(
    		integer: Int,
    		afterDelay delay: TimeInterval) -> Future<Int, Never> {

  		}
	}
	```

	- 이렇게 `Int`와 `Never` 타입을 가지는 `Future`를 반환하는 놈을 하나 만들었습니다.
- 이제 subscription 들을 저장할 `subscriptions` 묶음을 추가할겁니다. 장시간 실행되는 비동기 작업의 경우 subscription을 저장하지 않으면 현재의 코드 범위가 종료되는 즉시 subscription이 취소되기 때문입니다. 다음 코드를 위 `Future` 의 body에 채워넣습니다.

	```swift
	Future<Int, Never> { promise in
		DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
			promise(.success(integer + 1))
		}
	}
	```

	- 이 코드는 `Future`를 정의하며, 함수 호출시 지정한 값을 이용하여 `delay`후 `Int`를 증가시킬 것임을 알 수 있습니다.
- `Future`는 결국 하나의 값을 만들어 완료되거나 실패될 publisher 입니다. 이는 값이나 error가 있을 때 closure를 호출하여 이를 수행하며, 해당 closure를 `promise` 라고 합니다.
- `Future`를 **Command-click**하여 **Jump to Definition**으로 가봅시다. 다음과 같은 내용을 볼 수 있을거예요.
	
	```swift
	final public class Future<Output, Failure> : Publisher
		where Failure: Error {
		public typealias Promise = (Result<Output, Failure>) -> Void
  		...
	}
	```

	- `promise`는 `Future`가 방출한 단일 값 또는 error를 포함하는 `Result`를 받는 closure에 대한 `typealias` 입니다.
- 다시 코드로 돌아가서 다음 내용을 추가합시다.
	```swift
	// 1
	let future = futureIncrement(integer: 1, afterDelay: 3)

	// 2
	future
  		.sink(receiveCompletion: { print($0) },
  			receiveValue: { print($0) })
  		.store(in: &subscriptions)
	```

	- 1: 3초 지연 후 전달받은 정수를 증가시키도록 이전 코드에 만든 기능을 사용하여 `Future`를 생성
	- 2: 수신된 값과 완료 이벤트를 subscription 및 print하고 결과로 받은 `subscriptions`을 `subscriptions` set에 저장(store)합니다. 이 장의 뒷 부분에서 collection에 `subscription`을 저장하는 방법에 대해 자세히 알아볼 것입니다. 일단 여기서는 넘어가죠.

	- 콘솔에는 다음과 같이 표시됩니다.

		```
		——— Example of: Future ———
		2
		finished
		```

- 다음과 같이 두 번째 subscription을 `Future`에 추가합니다.

	```swift
	future
		.sink(receiveCompletion: { print("Second", $0) },
			receiveValue: { print("Second", $0) })
	  	.store(in: &subscriptions)
	```

	- 위 코드를 실행하기 전에 `print("Original")` 이라는 코드를 기존 예제 코드 중 `futureIncrement` 함수 내 `DispatchQueue` block 바로 위에 추가합니다. 그리고 한번 실행시켜보면 입력한 지연 시간 이후 두 번째 subscription이 같은 값을 받은 것을 확인할 수 있습니다. `Future`는 자신의 `promise`를 재실행하지 않고 대신 이 출력 값을 share하거나 replay 합니다.
	- 콘솔에는 다음과 같이 나올 것입니다.
		```
		——— Example of: Future ———
		Original
		2
		finished
		Second 2
		Second finished
		```

	- 우리가 print로 입력했던 `Orginal` 또한 subscription이 발생하기 전에 print 되는 것을 알 수 있습니다. 이는 `Future`가 만들어지자마자 실행되었기 때문입니다. 일반적인 publisher와 같이 subscriber가 필요하지 않습니다. *(헐 이상해)*
- 이 장에서 작성한 예제 중 마지막에 작성한 몇 개의 예제에서는 방출할 값이 유한한 publisher가 순차적이고 동기적으로 값을 방출하는 것을 확인하였습니다. 하지만 처음에 작성했던 notification center 예제의 경우 다음과 같은 경우에는 값을 무기한, 무제한, 비동기적으로 방출하는 publisher를 확인할 수 있습니다.
	1. 기본 notification 발신자가 notification을 방출합니다.
	2. 지정된 notification에 subscriber가 있습니다.
- 동일한 작업을 계속 수행해야할 때 `Future`를 사용할 수 있을 것입니다.
- 다음 내용을 보기 전에 작성한 `Future` 예제를 playground에서 주석처리하세요. 그렇지 않으면 마지막 예제 후 지연된 출력 값이 print 될 것입니다. 


## H. Hello Subject
### 1. PassthroughSubject
- **subject**를 살펴봅시다. subject는 Combine 으로 작성되지 않은 코드가 Combine의 subscriber에게 값을 보낼 수 있도록 중간자 역할을 하는 아이입니다.
- 다음 코드를 작성해봅시다.
	```swift
	example(of: "PassthroughSubject") {
		// 1
		enum MyError: Error {
	    	case test
	    }
	  	
	  	// 2
	  	final class StringSubscriber: Subscriber {
	    	typealias Input = String
	    	typealias Failure = MyError
	    	
	    	func receive(subscription: Subscription) {
	      		subscription.request(.max(2))
	    	}
	    	
	    	func receive(_ input: String) -> Subscribers.Demand {
	      		print("Received value", input)
	      		// 3
	      		return input == "World" ? .max(1) : .none
	    	}
	    	
	    	func receive(completion: Subscribers.Completion<MyError>) {
	      		print("Received completion", completion)
	    	}
	  	}
	  	
	  	// 4
	  	let subscriber = StringSubscriber()
	}
	```

	- 1: 사용자 error 타입을 정의합니다.
	- 2: `String` 값과 `MyError` error를 받을 사용자 subscriber를 정의합니다.
	- 3: 수신할 값에 따라 `Demand` 를 조정합니다.
	- 4: 사용자 subscriber 객체를 생성합니다.

	- 입력이 `"World"` 일 때 `receive(_:)` 에서 `.max(1)` 을 반환하면 새로운 최대값이 3(기존 최대값 + 1)으로 설정됩니다. 
- 사용자 error 유형을 정의하고 수요를 조정하기 위해 수신한 값을 조정한 것외에 새로운 내용은 없어보입니다. 다음 코드를 추가해봅시다.
	```swift
	// 5
	let subject = PassthroughSubject<String, MyError>()

	// 6
	subject.subscribe(subscriber)

	// 7
	let subscription = subject
		.sink(
	    	receiveCompletion: { completion in
	      		print("Received completion (sink)", completion)
	    	},
	    	receiveValue: { value in
	      		print("Received value (sink)", value)
	    	}
	  	)
	```

	- 5: `String`과 `MyError`를 갖는 `PassthroughSubject` 객체를 생성합니다.
	- 6: subject가 subscriber를 구독하도록 합니다.
	- 7: `sink` 를 이용하여 또 다른 subscription을 생성합니다.

- Passthrough Subject를 사용하면 필요에 따라 새로운 값을 게시할 수 있습니다. 다른 publisher와 마찬가지로 미리 발생할 수 있는 값과 error 유형을 선언해야 합니다. subscriber가 해당 Passthrough Subject를 subscribe 하려면 해당 유형을 자신의 Input 및 Failure 유형과 일치시켜야 합니다. 
- 이제 값과 subcription을 통해서 받을 수 있는 Passthrough Subject를 만들었으니 값을 보내야겠죠. 다음 코드를 추가해봅시다.
	```swift
	subject.send("Hello")
	subject.send("World")
	```

	- subject의 `send` method를 통해 위 두 개의 값은 동시해 전송됩니다.
	- 콘솔을 보면 다음과 같이 표현됩니다.
		```
		——— Example of: PassthroughSubject ———
		Received value Hello
		Received value (sink) Hello
		Received value World
		Received value (sink) World
		```

	- 각각의 subscriber들이 값을 받았음을 확인할 수 있습니다.
- 다음 코드를 추가합시다.
	```swift
	// 8
	subscription.cancel()

	// 9
	subject.send("Still there?")
	```

	- 1: 두 번째 subscription을 취소합니다.
	- 2: 또 다른 값을 전송합니다.

- playground를 돌려보면 예상한 것과 가이 첫 번째 subscriber만 값을 받은 것을 알 수 있습니다. 왜냐하면 두 번째 subscriber의 subscription을 값 전송 전에 취소했기 때문입니다. 콘솔에는 다음과 같이 찍히겠죠.
	```
	——— Example of: PassthroughSubject ———
	Received value Hello
	Received value (sink) Hello
	Received value World
	Received value (sink) World
	Received value Still there?
	```

- 다음 코드를 추가해 봅시다.
	```swift
	subject.send(completion: .finished)
	subject.send("How about another one?")
	```

- playground를 돌리면 두 번째 subscriber가 `"How about another one?"` 값을 받지 않을 것을 알 수 있습니다. 왜냐하면 그 전에 완료 이벤트를 받았기 때문입니다. 첫 번째 subscriber는 완료 이벤트나 값을 받지 않습니다. 왜냐하면 이전에 취소 되었기 때문이죠. 콘솔엔 다음과 같이 나타날겁니다.
	```
	——— Example of: PassthroughSubject ———
	Received value Hello
	Received value (sink) Hello
	Received value World
	Received value (sink) World
	Received value Still there?
	Received completion finished
	```
- 다음 코드를 완료 이벤트를 보내기 직전에 추가해봅시다.
	```swift
	subject.send(completion: .failure(MyError.test))
	```

- playground를 다시 돌려보면 다음 내용을 콘솔에서 확인할 수 있습니다.
	```
	——— Example of: PassthroughSubject ———
	Received value Hello
	Received value (sink) Hello
	Received value World
	Received value (sink) World
	Received value Still there?
	Received completion failure(...MyError.test)
	```

	- 첫 번째 subscriber가 error를 수신했지만 error *후에* 전송된 완료 이벤트는 수신되지 않았습니다. 이는 publisher가 *단일* 완료 이벤트(보통 완료 또는 error 여부)를 보내면 종료된다는 것을 나타냅니다. 
- `PassthroughSubject`는 명령형 코드를 선언적인 Combine 세계에 연결하는 방법입니다. 하지만 때때로 명령형 코드에서 publisher의 현재 값을 보고 싶을 수 있습니다. 이를 위해 `CurrentValueSubject`라는 subject도 있습니다.
- 각 subscription을 값으로 저장하는 대신 `AnyCancellable` collection에 여러 subscription을 저장할 수 있습니다. 그러면 collection이 초기화 될 때 collection에 추가된 각각의 subscription이 자동으로 취소됩니다.

### 2. CurrentValueSubject
- 새로운 예제를 작성해봅시다.
	```swift
	example(of: "CurrentValueSubject") {
		// 1
		let subject = CurrentValueSubject<Int, Never>(0)
	  	
	  	// 2
	  	subject
	    	.sink(receiveValue: { print($0) })
	    	.store(in: &subscriptions) // 3
	}
	```

	- 1: `Int`와 `Never` 타입을 갖는 `CurrentValueSubject`를 생성합니다. 여기서 초기값을 0으로 두었습니다.
	- 2: subject에 subcription을 생성하고 받은 값을 print할 수 있도록 합니다.
	- 3: subcription을 `subscriptions` 묶음에 저장합니다. 이 set는 사본이 아니라 동일한 set가 업데이트 되도록 `inout` parameter를 통해 전달됩니다.

- `CurrentValueSubject`는 반드시 초기값으로 생성해야 합니다. 새 subscriber는 즉시 초기값 또는 해당 subject에 의해 방출된 최신 값을 받습니다. playground를 실행해봅시다.
	```
	——— Example of: CurrentValueSubject ———
	0
	```

- 자, 이제 다음 코드를 통해 두 개의 새 값들을 전송합니다.
	```swift
	subject.send(1)
	subject.send(2)
	```
	- 다시 playground를 돌려보면 `1`, `2`가 찍히는 것을 확인할 수 있습니다.
- Passthrough Subject와는 달리 Current Value Subject는 값을 언제든 확인할 수 있습니다. 다음 코드를 추가하여 subject의 현재 값을 확인해봅시다. *(헐.. 신기)*
	```swift
	print(subject.value)
	```
- subject 이름이 Current Value(현재 값)인 것으로 추측할 수 있듯이 `value` 속성에 접근하여 현재 값을 얻을 수 있습니다. 
- Current Value Subjectr에 `send(_:)`를 호출하는 것은 새 값을 보낼 수 있는 방법입니다. 또 다른 방법으로는 `value` 속성에 새 값을 직접 할당하는 것입니다. *(wow!)*
	```swift
	subject.value = 3
	print(subject.value)
	```

	- playground를 돌려보면 `2`, `3`이 두번씩 print 되는 것을 볼 수 있습니다. 
- 이제 다음 코드를 통해 새로운 subscription을 current value subject에 생성해줍시다.
	```swift
	subject
  		.sink(receiveValue: { print("Second subscription:", $0) })
  		.store(in: &subscriptions)
	```

- 앞서 subscription 묶음이 subscription을 자동으로 취소한다는 점을 배웠는데 어떻게 확인할 수 있을까요? `print()` operator를 이용하면 모든 방출 이벤트를 콘솔에 기록할 수 있습니다. subject와 sick 사이 subscription에 `print()` operator를 삽입하여 각 subscription의 시작을 다음과 같이 수정합니다.
	```swift
	subject
	  	.print()
	  	.sink...
	```
- 이제 playground를 실행하면 다음과 같이 확인됩니다.
	```
	——— Example of: CurrentValueSubject ———
	receive subscription: (CurrentValueSubject)
	request unlimited
	receive value: (0)
	0
	receive value: (1)
	1
	receive value: (2)
	2
	2
	receive value: (3)
	3
	3
	receive subscription: (CurrentValueSubject)
	request unlimited
	receive value: (3)
	Second subscription: 3
	receive cancel
	receive cancel
	```

- 여기서 한 가지 질문이 떠오를 수 있습니다. subject의 `value` 값으로 완료 이벤트를 주입할 수 있을까? 다음 코드를 주입해봅시다.
	```swift
	subject.value = .finished
	```
	- 안되죠? 이 코드는 error로 표시됩니다. `CurrentValueSubject`의 `value` 속성은 정말 말 그대로 값을 의미합니다. 완료 이벤트는 `send(_:)`를 통해 존재합니다. 에러가 발생한 코드를 다음과 같이 수정해줍시다.
		```swift
		subject.send(completion: .finished
		```
- playground를 다시 돌려보면 다음과 같은 결과가 출력되는 것을 확인할 수 있습니다.
	```
	receive finished
	receive finished
	```

## I. 동적 수요 조정
- 앞서서 우린 `Subscriber.receive(_:)`를 통해 수요를 조정할 수 있다는 것을 배웠습니다. 새 예제 코드로 다음 내용을 봅시다.

	```swift
	example(of: "Dynamically adjusting Demand") {
		final class IntSubscriber: Subscriber {
	    	typealias Input = Int
	    	typealias Failure = Never
	    	
	    	func receive(subscription: Subscription) {
	      		subscription.request(.max(2))
	    	}
	    
	    	func receive(_ input: Int) -> Subscribers.Demand {
	      		print("Received value", input)
	      		
		      	switch input {
		      	case 1:
		        	return .max(2) // 1
		      	case 3:
		        	return .max(1) // 2
		      	default:
		        	return .none // 3
		        }
		    }
	    
	    	func receive(completion: Subscribers.Completion<Never>) {
				print("Received completion", completion)
	    	}
	  	}
	  	
	  	let subscriber = IntSubscriber()
	  	
	  	let subject = PassthroughSubject<Int, Never>()
	  	
	  	subject.subscribe(subscriber)
	  	
		  subject.send(1)
		  subject.send(2)
		  subject.send(3)
		  subject.send(4)
		  subject.send(5)
		  subject.send(6)
	}
	```

	- 앞서 다루었던 예제와 거의 유사하기 때문에, 여기서는 `receive(_:)` method 에 중점을 두고 볼 것입니다. 보면 사용자 설정 subscriber 내에서 지속적으로 수요를 조정합니다.

	- 1: 새 `max` 값은 4개다. (기존 `max` 2 + 새로운 `max` 2)
	- 2: 새 `max` 값은 5개다. (이전 `max` 4 + 새로운 `max` 1)
	- 3: `max` 값은 5로 유지된다. (이전 `max` 4 + 새로운 `max` 0)

- playground로 돌려보면 다음 내용을 콘솔에서 확인할 수 있습니다.
	```
	——— Example of: Dynamically adjusting Demand ———
	Received value 1
	Received value 2
	Received value 3
	Received value 4
	Received value 5
	```

	- 예상한대로 5개의 값이 나오지만 6개 값은 출력되지 않습니다. 
- 여기서 알아야 하는 중요한 사실이 하나 있습니다. 바로 publisher의 세부정보는 subscriber에게는 숨겨질 수 있다는 것입니다.

## J. Type 삭제
- subscrbier가 publisher에 대한 세부정보에 엑세스하지 않고도 publisher로부터 이벤트를 subscribe 할 수 있게 하려는 경우가 있습니다. 새 예제 코드를 살펴봅시다.
	```swift
	example(of: "Type erasure") {
		// 1
	  	let subject = PassthroughSubject<Int, Never>()
	  	
	  	// 2
	  	let publisher = subject.eraseToAnyPublisher()
	  	
	  	// 3
	  	publisher
	    	.sink(receiveValue: { print($0) })
	    	.store(in: &subscriptions)
	  	
	  	// 4
	  	subject.send(0)
	}
	```

	- 1: `PassthroughSubject`를 생성합니다.
	- 2: subject에 type 삭제 publisher를 생성합니다.
	- 3: type 삭제 publisher를 구독합니다.
	- 4: `PassthroughSubject`를 통해 새 값을 전송합니다.

	- `Publisher`에 **Option-click** 해보면 `AnyPublisher<Int, Never>` 를 확인할 수 있습니다.
- `AnyPublisher`는 type 삭제된 struct로 `Publisher` protocol을 따릅니다. type 삭제는 subscriber 또는 downstream의 publisher에게 노출하고 싶지 않은 publisher의 세부사항을 가릴 수 있게 해줍니다. 
- 여기까지 봤을 때 뭔가 데자뷰 같은 것 느껴지지 않나요? 그렇다면 그건 바로 또 다른 type 삭제 타입을 이미 봤었기 때문입니다. 바로 `AnyCancellable`인데요 이 녀석도 type 삭제된 class로 `Cancellable`을 따릅니다. subscriber가 더 많은 값을 요청하는 등의 작업을 수행하기 위해 기본 subscription에 엑세스하지 않고도 subscription을 취소할 수 있습니다.
- *publisher*에 적용할 수 있는 또 다른 type 삭제에 대한 예시가 있습니다. 바로 한 쌍의 private - public property를 사용하여 해당 propery 소유자가 private publisher에게 값을 보내고 외부 호출자는 구독은 가능하지만 값을 보낼 수는 없는 public publisher에만 엑세스할 수 있도록 할 때 입니다.
- `AnyPublisher`는 `send(_:)` operator가 없습니다. 따라서 publisher에 값을 추가할 수는 없습니다. 
- `eraseToAnyPublisher()` operator는 `AnyPublisher` 객체를 통해 제공된 publisher를 감싸서 publisher가 사실은 `PassthroughSubject`라는 사실을 숨깁니다. 이런 작업은 `Publisher<UIImage, Never>`처럼 `Publisher` protocol을 확정할 수 없을 때도 필요합니다.
- `Publisher`가 type 삭제 되었고 새 값을 보낼 수 없다는 것을 확인하고 싶다면 방금 작성했던 예제 코드에 다음 코드를 추가해 보세요. 
	```swift
	publisher.send(1)
	```
	- `type 'AnyPublisher<Int, Never>' has no member 'send'` 라는 에러가 생성되는 것을 확인할 수 있습니다.

## Summary
- Publisher는 시간이 지남에 따라 일련의 값을 하나 이상의 subscriber에게 동기적 또는 비동기적으로 전송합니다.
- Subscriber는 값을 받기 위해 publisher를 subscription 할 수 있습니다. 그러나 subscriber의 input 과 failure 유형은 publisher의 output과 failure 유형과 반드시 일치해야 합니다.
- publisher를 구독하는데 사용할 수 있는 내장 연산자는 `sink(_:)` 와 `assign(to:go:)`가 있습니다.
- subscriber는 값을 받을 때마다 값에 대한 수요를 증가시킬 수 있지만 감소시킬 수는 없습니다.
- 리소스를 확보하고 원하지 않는 부작용을 방지하려면 각 subscription이 완료될 때 취소해야합니다.
- subscription을 `AnyCancellable` 객체 또는 collection에 저장하여 할당 해제 시점에 자동 취소되도록 할 수 있습니다.
- Future는 나중에 단일 값을 비동기적으로 받고자 할 때 사용할 수 있습니다.
- Subject는 외부 호출자가 초기값의 유무와 관계없이 subscriber에게 여러 개의 값을 비동기적으로 보낼 수 있는 publisher입니다. 
- type 삭제는 호출자가 기존 type의 추가 세부 정보에 엑세스할 수 없도록 합니다. 
- `print()` operator를 사용하여 방출되는 모든 이벤트 로그를 콘솔을 통해 확인할 수 있습니다.

***
##### Artwork/images/designs: from Combine: Asynchronous Programming with Swift, available at www.raywenderlich.com
