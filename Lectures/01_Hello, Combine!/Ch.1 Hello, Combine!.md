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
- name += "Harding" 직전에 실행하므로 원래 값 "Tom" 대신 "Billy Bob"이 표시

이 코드를 실행할 때 발생하는 것은 시스템 로드 상태에 따라 다르며 프로그램을 실행할 때마다 다른 결과가 나타날 수 있습니다. 비동기적으로 동시에 실행되는 코드를 실행하는 순간 변경 가능한 상태 관리는 필수적입니다. 

## B. Foundation, UIKit/AppKit
Apple은 매 해를 거듭하며 비동기 프로그래밍 방식을 개선했습니다. 우리 대부분은 이미 그 코드들을 사용하고 있습니다. 주로 쓰는 건 다음과 같을거예요.

- `NotificationCenter`: 사용자가 기기 방향을 변경하거나, 소프트웨어 키보드가 화면에 표시/숨김 처리될 때와 같이 특정 이벤트가 발생할 때마다 코드를 실행합니다. 
- 델리게이트 패턴: 다른 object를 대신하여 또는 함께 작동하는 object를 정의할 수 있습니다. 예를 들어 app delegate 에 새로운 remote notification이 도착하면 어떻게 작동할 것인지 정의할 수 있지만 이 코드가 언제 실행될지, 몇 번 실행될지는 전혀 알 수 없습니다. 
- Grand Central Dispatch, Operations: 수행할 작업을 추상화하는데 도움이 됩니다. 이를 사용하여 코드가 순차적으로 실행되도록 예약하거나 우선 순위가 다른 여러 대기열에서 동시에 여러 작업을 실행하도록 할 수 있습니다. 
- Closures: 전달가능한 코드 뭉치를 만들어서 다른 object가 해당 코드를 실행할지 여부, 횟수 및 컨텍스트를 결정할 수 있습니다.

일반적으로 대부분의 코드는 작업을 비동기식으로 수행하고 모든 UI 이벤트는 본질적으로 비동기적이므로 전체 앱 코드가 실행될 순서를 가정하는 것은 사실상 불가능합니다. 또한 괜찮은 비동기식 프로그램 작성은 쉽지 않습니다. 비동기 코드와 리소스 공유는 재현이나 추적이 어렵고 궁극적으로 수정하기도 어려운 문제를 일으킬 수 있습니다. 이런 문제의 원인은 실제 앱이 각각의 고유한 인터페이스를 가지는 비동기 API들을 사용하는데 있습니다. 

	<img src = "https://github.com/fimuxd/combine/blob/master/Lectures/01_Hello%2C%20Combine!/1.%20asynchronous.png" width = 150>




***
##### Artwork/images/designs: from Combine: Asynchronous Programming with Swift, available at www.raywenderlich.com
