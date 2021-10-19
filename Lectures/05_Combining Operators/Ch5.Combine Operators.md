# Ch5. Combine Operators
> 지난 시간까지 Transforming 및 Filtering Operator를 다루어보면서 operator가 어떻게 작동하는지, 어떻게 upstream을 조작하고 publisher chain을 구성하는지 확인할 수 있습니다. 
이 장에서는 더 복잡하지만 유용한 operator 범주 중 하나인 combine operator에 대해 배우게 됩니다. 이 operator들을 통해 다른 publisher가 방출하는 이벤트를 결합(combine)하고 다양한 데이터 조합을 만들 수 있습니다.
combine이 왜 필요할까요? 사용자 이름, 비밀번호, 체크박스와 같은 사용자로부터 여러 입력이 있는 양식에 대해 생각해 보세요. 이 모든 데이터들을 결합하여 단일 publisher에게 필요한 정보를 구성할 수 있을겁니다.
또한, 각 operator가 어떻게 작동하는지, 그리고 상황에 따라 필요한 operator를 선택하는 방법에 대해 알게 된다면 코드의 일관성은 더 높아질 것입니다.

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
