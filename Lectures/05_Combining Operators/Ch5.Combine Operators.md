# Ch5. Combine Operators
> -  지난 시간까지 Transforming 및 Filtering Operator를 다루어보면서 operator가 어떻게 작동하는지, 어떻게 upstream을 조작하고 publisher chain을 구성하는지 확인할 수 있습니다. 
> - 이 장에서는 더 복잡하지만 유용한 operator 범주 중 하나인 combine operator에 대해 배우게 됩니다.
> - 이 operator들을 통해 다른 publisher가 방출하는 이벤트를 결합(combine)하고 다양한 데이터 조합을 만들 수 있습니다.
> - combine이 왜 필요할까요? 사용자 이름, 비밀번호, 체크박스와 같은 사용자로부터 여러 입력이 있는 양식에 대해 생각해 보세요. 이 모든 데이터들을 결합하여 단일 publisher에게 필요한 정보를 구성할 수 있을겁니다.
> - 또한, 각 operator가 어떻게 작동하는지, 그리고 상황에 따라 필요한 operator를 선택하는 방법에 대해 알게 된다면 코드의 일관성은 더 높아질 것입니다.

## 1. Prepending
- Prenpend 계열 operator들은 명칭에서 추측할 수 있듯이 원래의 publisher의 값보다 먼저 방출되는 값을 준비하여 전달하게 됩니다. 즉, 앞쪽에 값을 추가하여 먼저 방출되고 기존의 publisher의 값이 방출되게 됩니다.
- 이 섹션에서는 `prepend(Output...)`, `prepend(Sequence)` 및 `prepend(Publisher)` 에 대해 다뤄보겠습니다.

### 1. `prepend(Output...)`
- 이 친구는 원래의 publisher와 동일한 output type의 값들을 취하게 되고, 이들은 원래의 publisher에 앞서 차례로 방출됩니다.
![img](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/1.%20prepend.png?raw=true)
- 아래의 코드를 추가해보겠습니다.

	```swift
	// 1
	let publisher = [3, 4].publisher
	  
	// 2
	publisher
	  .prepend(1, 2)
	  .sink(receiveValue: { print($0) })
	  .store(in: &subscriptions)
	```

- 위 코드에서
	1. `3`, `4` 라는 값을  방출하는 publisher를 선언합니다.
	2. `prepend` 를 활용하여 기존의 pulbisher의 값 방출 이전에 `1`, `2` 라는 값이 방출되도록 합니다. 
- playground를 실행시켜보면 debug consol에 다음과 같이 출력됩니다.

	```swift
	1
	2
	3
	4
	```
- 우리는 지난 시간에 operator들은 chain으로 연결될 수 있다는 것을 확인했습니다. 따라서 다음 코드를 위 코드의 `prepend` 뒤에 추가할 수 있겠죠.

	```swift
	.prepend(-1, 0)
	```
- 이렇게 하면 다음과 같이 debug consol과 연결됩니다.

	```swift
	-1
	0
	1
	2
	3
	4
	```
- 여기서 주목할 것은 operator의 실행 순서입니다. 마지막에 연결된 prepend부터 upstream에 영향을 미치기 때문에 `-1`과 `0`이 앞에 있고, 그 다음에 `1` `2`, 마지막으로 원래의 publisher 값이 출력되게 됩니다.

## 2. `prepend(Sequence)` 
- 이번 operator는 방금 다룬  것과 거의 비슷합니다. 다만 sequence를 받는 것만 다른 점이죠. 예를 들면 `Array`나 `Set`을 받을 수 있습니다.
![img](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/2.%20prependSequence.png?raw=true)

- 다음 코드를 작성해봅시다.

	```swift
	// 1
	let publisher = [5, 6, 7].publisher
	  
	// 2
	publisher
	  .prepend([3, 4])
	  .prepend(Set(1...2))
	  .sink(receiveValue: { print($0) })
	  .store(in: &subscriptions)
	```
- 위 코드에서
	1. `5`, `6`, `7`을 방출하는 publisher를 선언합니다.
	2. `prepend(Sequence)` 를 두 번 활용하여 기존의 publisher에 연결합니다. 처음엔 `Array`, 두 번째에는 `Set`이 불려질겁니다.
- 코드를 실행해보면 다음과 같이 출력됩니다.

	```swift
	1
	2
	3
	4
	5
	6
	7
	``` 
> **Note**: `Set` 에 대해 기억할 한가지는 `Array` 와는 반대로 순서를 보장하지 않는다는 것입니다. 따라서 방출되는 아이템의 순서 또한 보장하지 않습니다. 즉, 위 샘플 코드에서 `1`, `2`는 `2`, `1`로 출력될 수도 있습니다.
- 이번엔 위 코드에 한 개의 prepend를 더 추가해보겠습니다. Swift에서는 다양한 형태로 Sequence를 만들 수 있으니까요😉

	```swift
	.prepend(Set(1...2))	//기존의 prepend에 
	.prepend(stride(from: 6, to: 11, by: 2))	//얘를 추가해주세요
	``` 
- `stride`를 활용하여 6부터 11까지 2만큼의 보폭을 가지는(Strideable)한 놈을 만들어냈습니다. Strideable 역시 Sequence이기 때문에 `prepend(Sequence)`에 넣어줄 수 있습니다. 이렇게 하면 다음과 같이 출력되겠죠.

	```swift
	6
	8
	10
	1
	2
	3
	4
	5
	6
	7
	```
- 보시다시피 최종적으로 선언된 아이부터 upstream에 영향을 미치고 다음 prepend가 차례차례 실행된 다음 원래의 publisher가 값을 방출하게 됩니다.

## 3. `prepend(Publisher)`
- 방금까지 다룬 두 prepend operator는 원래의 publisher보다 앞서서 값을 방출했습니다. 하지만 두 개의 서로 다른 publisher가 있고 각자의 값을 함께 붙이고 싶다면 어떨까요? 원래 publisher의 값 앞에 두 번째 publisher가 방출한 값을 추가하려면 `prepend(Publisher)`를 사용할 수 있습니다.
![img](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/3.%20prependPublisher.png?raw=true)
- 다음 코드를 확인해봅시다.

  ```swift
  // 1
  let publisher1 = [3, 4].publisher
  let publisher2 = [1, 2].publisher
  
  // 2
  publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
  ```
- 위 코드에서 
	1. 두 개의 publisher를 만들었습니다. 하나는 `3`, `4`를 방출하고 다른 하나는 `1`, `2`를 방출하겠네요.
	2. `publisher2`를 `publisher1` 뒤에 붙입니다.	이로써 `publisher1`의 값들은  `publisher2`의 값들이 모두 방출된 다음에 방출될 것입니다. 
- 코드를 실행하면 다음과 같이 consol에 찍힐겁니다.

	```swift
	1
	2
	3
	4
	```
- 예상대로, 값 `1`과 `2`는 먼저 `publisher2`에서 방출됩니다된. 그 다음에 `3`과 `4`는 `publisher1`에 의해 방출된다.
- 한 가지 더 알아보기 위해  다음 예제 코드를 작성해보겠습니다.

  ```swift
  // 1
  let publisher1 = [3, 4].publisher
  let publisher2 = PassthroughSubject<Int, Never>()
  
  // 2
  publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)

  // 3
  publisher2.send(1)
  publisher2.send(2)
  ```
  - 이 예제는 이전 예제와 비슷하지만, `publisher2`의 타입을 값을 수동으로 푸시할 수 있는 `PassthroughSubject` 로 설정했습니다.
  - 코드를 살펴보면
	  1. 두 개의 publisher를 만들었습니다. 하나는 `3`, `4`를 가지는 `publisher`,  두 번째는 다이나믹하게 값을 받을 수 있는 `PassthroughSubject` 입니다. 
	  2. `publisher1` 다음에 `publisher2`를 `prepend`로 연결합니다.
	  3. `1`, `2` 각각의 값을 `publisher2`에 보냅니다. 
- 실행해보기 전에 잠시 코드를 살펴보죠. 어떻게 작동될 것 같으신가요? 예상대로 실행되는지 한번 봅시다.

	```swift
	1
	2
	```
- 음? 뭐죠? 왜 `publisher2`에게 보낸 겨우 두 개의 값만 방출된 걸까요? prepend는 기존의 publisher 앞에 값을 붙이는거라더니?
- 잠깐 생각해 봅시다. Combine은 어떻게 `publisher2`가 값 방출을 끝냈다는 것을 알 수 있을까요? 다시보면 값을 방출하긴 했는데 완료 이벤트는 없습니다. 먼저 진행된 publisher는 반드시 완료되어야 합니다. 그래야 다음 publisher가 값 방출을 시작할 수 있겠죠.
- 위 예제의 마지막 코드 뒤에 다음 코드를 추가합시다.

  ```swift
  publisher2.send(completion: .finished)
  ```
- 이렇게 하면 Combine이 `publisher2`가 종료된 것을 인지하고 `publisher1`에서 값이 방출될 수 있도록 할 겁니다. 코드를 실행해볼까요.
	
	```swift
	1
	2
	3
	4
	```

## 4. Appending
- 이번에 다룰 operator들은  publisher가 방출한 이벤트를 다른 값과 연결하는 아이들입니다.. 
- 지금까지 다루었던 `prepend` 연산자와 비슷하지만 `append(Output...)`, `append(Sequence)` 및 `append(Publisher)`를 사용하여 값들을 처리할 것입니다.

### 1. `append(Output...)`
- `append(Output...)`은 `prepend`와 비슷하게 작동합니다. 다만 원래의 publisher가 `.finished`이벤트로 완료한 후에 항목을 추가하게 됩니다.
![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/4.%20append.png?raw=true)
- 다음 코드를 함께 확인해봅시다.
		
	```swift
	// 1
	let publisher = [1].publisher
		
	// 2
	publisher
		.append(2, 3)
		.append(4) 
		.sink(receiveValue: { print($0) })
		.store(in: &subscriptions)’
	```
- 코드를 살펴보면
	1. `1` 하나의 값을 방출하는 publisher를 생성합니다.
	2. `append`를 두 번 사용합니다. 첫 번째 `append`에는 `2`와 `3`을 넣어주고 `4`를 추가하는 두 번째 `append`를 추가합니다.
- 작성한 후 결과가 어떻게 나올지 한번 생각해봅시다. (⏰...) 결과는 다음과 같습니다.
	
	```swift
	1
	2
	3
	4
	```
-  아마 append를 활용한 코드에서 여러분이 예상하셨던 것과 같은 결과가 나왔을 것 같습니다. 각각의 `append` 연산자들은 upstream이 완료된 후에 자신의 값을 추가하게 됩니닫.
-  이  것은 곧  upstream이 **반드시 완료(complete)** 되어야 한다는 것을 의미합니다. 그렇지 않으면 Combine이 직전의 publisher가 종료되었다는 것을 알지 못해서 아무런 작동을 하지 않는 것처럼 보일테니까요.
-  확인을 위해 다음 코드를 작성해봅시다.

	```swift
	// 1
	let publisher = PassthroughSubject<Int, Never>()
	
	publisher
		.append(3, 4)
		.append(5)
		.sink(receiveValue: { print($0) })
		.store(in: &subscriptions)
	  
	// 2
	publisher.send(1)
	publisher.send(2)’
	```
- 위 코드는 앞서 작성한 코드와 유사하지만 두 가지 차이점이 있습니다. 
	1. `publisher`가 이제 `PassthroughSubject`가 되었네요, 즉 수동으로 값을 보낼 수 있게 되었습니다.
	2. `PassthroughSubject`를 통해 `1`과 `2`를 보냈습니다. 
- 코드를 실행시켜보면 `publisher`가 다음과 같이 값을 방출한 것을 확인할 수 있습니다. 

	```swift
	1
	2
	```
- 두 개의 `append` 연산자들은 아무런 영향을 주지 못합니다. 왜냐하면 상위의 `publisher`가 완료되기 전까진 아무런 액션을 취하지 않기 때문이죠. 앞서 작성한 코드 마지막에 아래 코드를 추가하여 `publisher`를  완료시켜줍시다.

	```swift
	publisher.send(completion: .finished)
	```
- 다시 코드를 실행시키면 우리가 예상한대로 잘 표현되는 것을 알 수 있습니다.

	```swift
	1
	2
	3
	4
	5
	```
	
### 2. `append(Sequence)`
- 이번에는 `Sequence`를 받는 `append` 연산자입니다. 
![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/5.%20appendSequence.png?raw=true)
- 다음 코드를  작성해보겠습니다.

```swift
// 1
let publisher  = [1, 2, 3].publisher

publisher
	.append([4, 5])	//  2
	.append(Set([6, 7]))	// 3
	.append(stride(from: 8, to: 11, by: 2))	// 4
	.sink(receiveValue: { print($0) })
	.store(in: & subscriptions)
```
- 위 코드는 우리가 `prepend(Sequence)`를 확인할 때 작성했던 코드와 유사합니다. 하나씩 따라가보면, 
	1. `1`, `2`, `3`을 뱉는 `publisher`를 작성해줍니다.
	2. `4`, `5` 값과 순서를 갖는 `Array`를 append 해줍니다.
	3. `6`, `7` 값과 순서를 갖지 않는 `Set`를 append 해줍니다.
	4. `8`부터 `11`까지 `2`의 간격으로 값을 뱉는 `Striddeable`을 추가해줍니다. 
- 코드를 실행시켜봅시다.

```swift
1
2
3
4
5
7	// 순서가 바뀔 수 있음
6	// 순서가 바뀔 수 있음
8
10
```
- 보시다시피 `append` 연산자들은 상위 `publisher`들이 반드시 완료된 이후에 동작하게 됩니다. 
- 주석에 표현한 것처럼 `6`과 `7`은 순서를 보장하지 않으므로 방출 순서가 바뀔 수 있습니다.

### 3. `append(Publisher)`
- append 계열의 마지막 멤버는 다양한 `Publisher`를 취하는 연산자 입니다.
![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/6.%20appendPublisher.png?raw=true)

	```swift
	// 1
	let publisher1 = [1, 2].publisher
	let publisher2 = [3, 4].publisher
	  
	// 2
	publisher1
		.append(publisher2)
		.sink(receiveValue: { print($0) })
		.store(in: &subscriptions)’
	```
- 위 코드를 하나씩 살펴보면 
	1. 두 개의 `publisher`를 만들었구요, 첫 번째 `publisher`는 `1`, `2`를, 두 번째 `publisher`는 `3`, `4`를 방출합니다.
	2. `publisher2`를 `publisher1`에 `append` 합니다. 따라서 `publisher2`가 방출하는 모든 값은 `publisher1`의 값이 모두 방출된 이후에 방출될 것입니다.  
- 실행시켜보면 다음과 같이 확인할 수 있습니다. 

	```swift
	1
	2
	3
	4
	```

## 5. 고급 Combining
- 지금까지 appending과 prepending에 대한 모든 내용을 확인했습니다. 
- 이제 서로 다른 `publisher`들이 좀 더 복잡하게 결합하는 operator들에 대해 확인해볼 것입니다. 상당히 복잡하게 작동하는 것처럼 보일 것이지만 그만큼 유용하게 쓰이는 연산자들일거예요. 이들이 어떤 방식으로 구동되는지 확인하는 시간을 들일만한 가치가 있을겁니다. 

### 1. `switchToLatest`
- `switchToLatest`는 복잡하지만 아주 유용합니다. 
- 이 연산자는 최신의 publisher가 값을 방출할 때 기존의 publisher 구독을 취소하고 최신의 publisher로 구독을 전환(*switching*) 합니다. 
- 따라서 이 연산자는 오직 *스스로* publisher들을 방출하는 publisher에게만 사용할 수가 있습니다. (무슨 말이냐구요? 아래 그림과 코드를 보시죠 😄 ) 

![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/7.%20switchToLatest.png?raw=true)

	```swift
	// 1
	let publisher1 = PassthroughSubject<Int, Never>()
	let publisher2 = PassthroughSubject<Int, Never>()
	let publisher3 = PassthroughSubject<Int, Never>()
	
	// 2
	let publishers = PassthroughSubject<PassthroughSubject<Int, Never>, Never>()
	
	// 3
	publishers
		.switchToLatest()
		.sink(receiveCompletion: { _ in print("Completed!") },
			receiveValue: { print($0) })
		.store(in: &subscriptions)
	
	// 4
	publishers.send(publisher1)
	publisher1.send(1)
	publisher1.send(2)
	
	// 5
	publishers.send(publisher2)
	publisher1.send(3)
	publisher2.send(4)
	publisher2.send(5)
	
	// 6
	publishers.send(publisher3)
	publisher2.send(6)
	publisher3.send(7)
	publisher3.send(8)
	publisher3.send(9)
	
	// 7
	publisher3.send(completion: .finished)
	publishers.send(completion: .finished)’
	```
- 으, 코드가 꽤 긴데요, 보시는 것보단 간단하니 너무 걱정하지 마세요. 한번 하나씩 살펴봅시다.
	1. 세 개의 `PassthroughSubjects`를 만들었구요, 정수를 받고 에러는 발생시키지 않는 애들입니다.
	2. 그 다음엔 다른 `PassthroughSubject`를 받는 `PassthroughSubject`를 만들었습니다. 예를 들면 앞서 만든 `publisher1`, `publisher2`, `publisher3`를 이 `publishers`로 넣을 수 있겠네요.
	3. `switchToLasted`를 `publishers`에 사용합니다. 이제 매번 서로 다른 publisher들이 `publishers` subject를 통해서 보내질 것이고 이 때마다 새로운 publisher로 전환(switch) 하고 이전의 구독을 취소하게 될거예요. 
	4. `publisher1`을 `publishers`로 보냅니다. 그리고 `publisher1`을 통해 `1`과 `2`를 보냅니다.
	5. 이제 `publisher2`를 `publishers`로 보냅니다. 이를 통해 `publisher1`에 대한 구독은 취소될거예요. `3`을 `publisher1`을 통해 보내줍니다만 이건 무시되겠죠. 그리고 `4`와 `5`를 `publisher2`를 통해 보내줍니다. 이들은 방출될 겁니다. 이제 `publisher2`가 현재 구독되는 publisher일테니까요.
	6. `publisher3`을 보내고 이는 `publisher2`에 대한 구독을 취소시킬겁니다. 이 후에 `6`을 `publisher2`를 통해 보내지만 이 역시 무시될겁니다. 이 후 `publisher3`을 통해 보내지는 `6`, `7`, `8`, `9`는 구독될겁니다.
	7. 마지막으로, completion 이벤트를 현재 구독되는 publisher인 `publisher3`을 통해 보내줍니다. 그리고 또 다른 completion 이벤트를 `publishers`에게도 보내줍니다. 이를 통해 현재 활성화 상태인 모든 구독은 완료될겁니다.
- marble diagram과 함께 확인하셨다면 아마 결과를 이미 예상하실거예요.

	```swift
	1
	2
	4
	5
	7
	8
	9
	completed!
	```
- 만약 이게 실제 app에서 어떻게 유용할 수 있는지 의문스러운 분들은 이런 상황을 생각해보세요.
- 사용자가 네트워크 요청을 할 수 있는 버튼이 있습니다. 만약 버튼을 한번 탭한 즉시 또 같은 버튼을 탭한다면 네트워크 요청이 두 번 되겠죠. 하지만 이렇게 진행 중인 요청이 있을 때 중복해서 요청이 들어올 수 있고, 이럴 때는 가장 마지막(*latest*) 요청에 대해서만 처리하고 싶다면 어떻게 할 수 있을까요? 바로 `switchToLatest`가 여러분을 구원해줄 수 있을거예요.
- 코드로 한번 살펴보겠습니다.

	```swift
	let url = URL(string: "https://source.unsplash.com/random")!
	  
	// 1
	func getImage() -> AnyPublisher<UIImage?, Never> {
		return URLSession.shared
			.dataTaskPublisher(for: url)
			.map { data, _ in UIImage(data: data) }
			.print("image")
			.replaceError(with: nil)
			.eraseToAnyPublisher()
	}
	
	// 2
	let taps = PassthroughSubject<Void, Never>()
	
	taps
		.map { _ in getImage() } // 3
		.switchToLatest() // 4
		.sink(receiveValue: { _ in })
		.store(in: &subscriptions)
	
	// 5
	taps.send()
	
	DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
		taps.send()
	}
	DispatchQueue.main.asyncAfter(deadline: .now() + 3.1) {
		taps.send()
	}
	```
-이번에도 코드가 꽤 길고 복잡해보이는데요, 간단합니다. 살펴볼게요.
	1. `getImage()`라는 함수를 선언합니다. Unsplash의 공공API를 통해 무작위 이미지를 가져올 수 있는 네트워크 요청을 할겁니다. Combine의 기본 표현 중 하나인 `URLSession.dataTaskPublisher`를 이용할거구요, 이와 관련해서는 Section 3, "Combine in Action"에서 다뤄볼겁니다.
	2. 사용자의 버튼 탭 액션을 방출하는 `PassthroughSubject`를 만들어줍니다. 
	3. 버튼이 탭 되면 `taps`는 무작위 이미지를 받아오는 네트워크 요청인 `getImage()`로 매핑될겁니다. 즉 `Publisher<Void, Never>`가 publisher들의 publisher인 `Publisher<<Publisher<UIImage?, Never>, Never>`로 변환할겁니다,.
	4. 이전 예제 코드에서처럼 publisher들의 publisher에 `switchToLatest()`를 사용합니다. 이로써 새로운 publisher가 나올 때마다 직전의 publisher에 대한 구독을 취소하게 될겁니다.
	5. `DispatchQueue`를 활용하여 3개의 버튼 탭 이벤트를 구현합니다. 첫 번째 탭은 즉시 발생할거구요, 두 번째 탭은 3초 뒤에, 세 번째 탭은 두 번째 탭이 발생하고 0.1초 뒤에 발생할겁니다.
- 코드를 실행시켜볼까요.

	```swift
	image: receive subscription: (DataTaskPublisher)
	image: request unlimited
	image: receive value: (Optional(<UIImage:0x600000364120 anonymous {1080, 720}>))
	image: receive finished
	image: receive subscription: (DataTaskPublisher)
	image: request unlimited
	image: receive cancel
	image: receive subscription: (DataTaskPublisher)
	image: request unlimited
	image: receive value: (Optional(<UIImage:0x600000378d80 anonymous {1080, 1620}>))
	image: receive finished
	```
- 여기서 주목해야할 것은 실제 2개의 이미지만 가져왔다는겁니다. 왜냐하면 두 번째 탭 0.1초 뒤에 불린 세 번째 탭으로 인해 두 번째 탭에 대한 구독은 취소되었기 때문입니다. 이 것이 `image: receive cancel`라고 표현된 것이죠.

### 2. `merge(with:)`
- 이 연산자는 서로 같은 타입을 같은 publisher를 **교차**하며 나타내게 됩니다.
![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/11.%20merge.png?raw=true)
- 코드와 함께 확인해보겠습니다.

	```swift
	// 1
	let publisher1 = PassthroughSubject<Int, Never>()
	let publisher2 = PassthroughSubject<Int, Never>()
	
	// 2
	publisher1
		.merge(with: publisher2)
		.sink(receiveCompletion: { _ in print("Completed") },
	      		receiveValue: { print($0) })
		.store(in: &subscriptions)
	
	// 3
	publisher1.send(1)
	publisher1.send(2)
	
	publisher2.send(3)
	
	publisher1.send(4)
	
	publisher2.send(5)
	
	// 4
	publisher1.send(completion: .finished)
	publisher2.send(completion: .finished)
	```
- 위 코드는 marble diagram을 나타낸겁니다. 하나씩 살펴봅시다.
	1. 두 개의 `PassthroughSubject`를 만들어줍니다. 둘다 `Int` 타입을 받고 에러는 방출하지 않습니다. 
	2. `publisher1`과 `publisher2`를 합쳐(Merge) 줍니다. Combine을 통해 이러한 방식으로 최대 8개의 서로 다른 publisher를 merge 할 수 있습니다. 
	3. `1`과 `2`를 `publisher1`에서 보내주고 `3`을 `publisher2`로, 그리고 다시 `4`를 `publisher1`에 넣어주고 마지막으로 `5`를 `publisher2`를 통해 전달합니다. 
	4. 두 publisher에 완료 이벤트를 보냅니다. 
- 코드를 실행시키면 다음과 같이 나타납니다.

```swift
1
2
3
4
5
Completed
```

### 3. `combineLatest`
- `combineLatest`를 이용하면 서로 다른 publisher들을 결합시킬 수 있습니다. 또한 결합되는 publisher들이 꼭 같은 타입일 필요도 없습니다. 아주 유용한 포인트죠. 단, 상호 교차하며 서로 다른 타입을 방출하는 것이 아니라 이런 타입을 **tuple** 형태로 묶어서 방출하게 됩니다.
- 여기서 또 한 가지 짚고 넘어갈 부분은, `combineLatest`로 결합된 모든 publisher들은 최소 한 번은 값을 방출해야 한다는 것입니다. 

![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/12.%20combineLatest.png?raw=true)
- 코드와 함께 살펴봅시다.

	```swift
	// 1
	let publisher1 = PassthroughSubject<Int, Never>()
	let publisher2 = PassthroughSubject<String, Never>()
	
	// 2
	publisher1
		.combineLatest(publisher2)
		.sink(receiveCompletion: { _ in print("Completed") },
			receiveValue: { print("P1: \($0), P2: \($1)") })
		.store(in: &subscriptions)
	
	// 3
	publisher1.send(1)
	publisher1.send(2)
	  
	publisher2.send("a")
	publisher2.send("b")
	  
	publisher1.send(3)
	  
	publisher2.send("c")
	
	// 4
	publisher1.send(completion: .finished)
	publisher2.send(completion: .finished)
	```
- 이 코드 역시 marble diagram을 표현한 것입니다.
	1. 두 개의 `PassthroughSubject`를 만들어줍니다. 하나는 `Int` 타입을 받구요, 다른 하나는 `String` 타입을 받습니다. 둘다 에러는 없네요.
	2. `publisher2`를 `publisher1`과 결합시켜줍니다. Combine을 통해서는 한 번에 최대 4개의 서로 다른 publisher들을 결합할 수 있습니다. 
	3. `publisher1`에 `1`, `2`를 보내고 `publisher2`에 `"a"`와 `"b"`를 보냅니다. 그리고 `publisher1`에 `3`을, `publisher2`에 `"c"`를 보내보죠.
	4. 두 publisher들에게 완료 이벤트를 보냅니다.
- 코드를 실행시키면 다음과 같이 표현됩니다.
	```swift
	P1: 2, P2: a
	P1: 2, P2: b
	P1: 3, P2: b
	P1: 3, P2: c
	Completed
	```
- 보시면 `publisher1`이 방출한 `1`은 `combineLatest`를 통해서는 보여지지 않는 것을 확인할 수 있습니다. 
- 이는 `combineLatest`는 모든 publisher들이 최소 1개의 값을 방출한 다음부터 동작하기 때문입니다. 
- 여기서는 `"a"`가 방출되었을 때 가장 최신의 `publisher1` 값은 `2`였기 때문에 `(2, "a")`가 방출된 것입니다.

### 4. `zip`
- 드디어 이 장의 마지막 연산자입니다. 바로 `zip`인데요, 아마 Swift의 기본 라이브러리에 `Sequence` 타입에 대해 같은 이름의 메소드가 있어서 이 연산자가 어떤 역할을 할지 추측하실 수 있을겁니다. 
- 예상하신 것처럼 기본 Swift의 zip과 비슷하게 동작합니다. 같은 index 상에 있는 값들을 tuple로 조합하여 방출하게 됩니다. zip은 각각의 publisher 모두가 값을 방출하길 기다렸다가 tuple로 조합하게 됩니다. 
- 이 것은 만약 여러분이 두개의 publisher를 고정(zipping) 시켰다면 여러분은 두 개의 publisher가 값을 방출할 때마다 하나의 tuple을 받게 된다는 의미입니다.

![](https://github.com/fimuxd/Combine/blob/master/Lectures/05_Combining%20Operators/13.%20zip.png?raw=true)
- 이 diagram을 코드로 살펴보겠습니다.

	```swift
	// 1
	let publisher1 = PassthroughSubject<Int, Never>()
	let publisher2 = PassthroughSubject<String, Never>()
	
	// 2
	publisher1
		.zip(publisher2)
		.sink(receiveCompletion: { _ in print("Completed") },
			receiveValue: { print("P1: \($0), P2: \($1)") })
		.store(in: &subscriptions)
	
	// 3
	publisher1.send(1)
	publisher1.send(2)
	publisher2.send("a")
	publisher2.send("b")
	publisher1.send(3)
	publisher2.send("c")
	publisher2.send("d")
	
	// 4
	publisher1.send(completion: .finished)
	publisher2.send(completion: .finished) 
	```

	1. 두 개의 `PassthroughSubject`를 만들어줍니다. 첫 번째는 `Int` 타입을, 두 번째는 `String` 타입을 받고 있습니다. 둘 다 에러를 발생시키진 않습니다.
	2. `publisher1`과 `publisher2`를 `zip`으로 연결해줍니다. 
	3. 각각의 값들을 publisher들에게 보내줍니다.
	4. 두 publisher들에 완료 이벤트를 전달합니다. 
- 코드를 실행시키면 다음과 같이 확인됩니다.

	```swift
	P1: 1, P2: a
	P1: 2, P2: b
	P1: 3, P2: c
	Completed
	```
- 주목해야할 것은 zip으로 결합된 publisher는 다른 publisher가 값을 방출할 때까지 기다린다는 점입니다. 

***

##### Artwork/images/designs: from Combine: Asynchronous Programming with Swift, available at www.raywenderlich.com