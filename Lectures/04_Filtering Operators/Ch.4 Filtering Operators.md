# Ch.4 Filtering Operators

## 시작하기

- 이 장에서 제공하는 예제 파일인 **Starter.playground** 를 확인하세요

> **Note**: 이 장에서 다룰 대부분의 operator들은 `filter` vs. `tryFilter` 와 같은 형태로 대응되는 `try` operator들을 가집니다. 둘의 차이점은 앞서 얘기했던대로 closure를 *떨궈주느냐* 에만 있습니다. publisher에 error가 떨어지면 publisher는 종료되고 `try` 형태의 operator는 이를 closure로 던져줄 것입니다. 이 장에서는 각 개념에 집중하기 위해 `try` 를 제외한 operator들만을 다룰 것입니다.



## Filtering 기본

### `Filter`

- 가장 먼저 다룰 것은 publisher의 값 중 전달할 값을 조건부로 결정하는 필터링의 기본 사항들입니다.
- 제일 쉬운 방법은 이름부터 명확한 `filter` 입니다. 이는 `Bool` 을 반환하는 closure를 통해 제공된 조건과 일치하는 값만 전달합니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/1.%20filter.png?raw=true" width = 400>

- 다음 예제 코드를 확인해봅시다.

  ```swift
  example(of: "filter") {
    // 1
    let numbers = (1...10).publisher
    
    // 2
    numbers
      .filter { $0.isMultiple(of: 3) }
      .sink(receiveValue: { n in
        print("\(n) is a multiple of 3!")
      })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `10` 까지 숫자를 방출하고 나서 완료되는 publisher를 생성합니다. `Sequence` 타입의  `publisher` property를 이용하여 생성할 수 있겠죠.

  - 2: `filter` operator를 이용해서 3의 배수에 해당하는 숫자만 print 되도록 합니다.

  - 콘솔 결과는 다음과 같습니다.

    ```
    3 is a multiple of 3!
    6 is a multiple of 3!
    9 is a multiple of 3!
    ```



### `removeDuplicates()`

- 앱을 사용하다보면 반복해서 여러 번 발생하는 액션을 무시하고 싶을 때가 있습니다. 예를 들면 사용자가 "a"를 연속으로 5번 입력한 다음 "b"를 입력할 때 과도하게 입력한 "a"를 무시하고 싶을 수 있죠. Combine은 이와 같은 상황에 아주 적절한 operator를 제공합니다. 바로 `removeDuplicates` 입니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/2.%20removeDuplicates.png?raw=true" width = 400>

- 보시다시피 이 operator는 아무런 argument도 갖지 않습니다! `removeDuplicate` 는 `String ` 같이 `Equatable` 을 준수하는 모든 값에 대해 자동적으로 작동합니다.

- 다음 `removeDuplicates()` 예제를 확인해봅시다. 

  ```swift
  example(of: "removeDuplicates") {
    // 1
    let words = "hey hey there! want to listen to mister mister ?"
                    .components(separatedBy: " ")
                    .publisher
    // 2
    words
      .removeDuplicates()
      .sink(receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `Array<String>` 형태의 문장을 단어 단위로 분리하여 publisher로 만들었습니다.

  - 2: `words` publisher에 `removeDuplicates()` 를 적용합니다.

  - 콘솔의 결과 입니다.

    ```
    ——— Example of: removeDuplicates ———
    hey
    there!
    want
    to
    listen
    to
    mister
    ?
    ```

    - 보시다시피 두번 반복된 "hey"와 "mister"가 무시된 것을 볼 수 있습니다.

> **Note**: `Equatable` 을 준수하지 않는 값이라면 어떻게 사용할 수 있을까요? `removeDuplicated` 에는 두 개의 값으로 closure를 취하는 또 다른 operator를 제공합니다. 여기서 `Bool` 을 반환하여 값이 같은지 확인합니다.



## 압축하고 무시하기

### `CompactMap`

- 우리는 종종 `Optional` 값을 방출하는 publisher를 만들거나 `nil` 을 반환하는 동작을 구현해야합니다. 하지만 일일히 모든  `nil` 을 처리하고 싶겠어요. 

<img src= "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/3.%20compactMap.png?raw=true" width = 400>

- 다음 코드를 살펴봅시다.

  ```swift
  example(of: "compactMap") {
    // 1
    let strings = ["a", "1.24", "3",
                   "def", "45", "0.23"].publisher
    
    // 2
    strings
      .compactMap { Float($0) }
      .sink(receiveValue: {
        // 3
        print($0)
      })
      .store(in: &subscriptions)
  }
  ```

  - 1: 유한한 문자 배열을 방출하는 publisher를 생성합니다. 
  - 2: `compactMap` 을 이용하여 각 문자열을 `Float` 타입으로 변경합니다. 만약 문자열을 `Float` 로 변경할 수 없다면 `nil` 을 반환하겠죠. 
  - 3: `Float` 으로 변환이 된 아이들만 print 합니다.

  - 콘솔 결과는 다음과 같습니다.

    ```
    ——— Example of: compactMap ———
    1.24
    3.0
    45.0
    0.23
    ```



### `ignoreOutput`

- publisher가 어떤 값을 내는지는 중요하지 않고 값 방출을 끝냈는지만 알고 싶을 때가 있습니다. 이럴 때는 `ignoreOutput` 을 사용할 수 있어요. 

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/4.%20ignoreOutput.png?raw=true" width = 400>

- 다이어그램에서 확인할 수 있듯이 어떤 값이 얼마나 나오든 신경쓰지 않습니다. 오직 완료 이벤트가 방출되었을 때만 알려주죠. 다음 코드를 살펴봅시다.

  ```swift
  example(of: "ignoreOutput") {
    // 1
    let numbers = (1...10_000).publisher
    
    // 2
    numbers
      .ignoreOutput()
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
  }	
  ```

  - 1: `1` 부터 `10000` 까지 10,000 개의 값을 방출하는 publisher를 생성합니다. 

  - 2: `ignoreOutput` operator를 추가하여 방출하는 모든 값은 무시하고 완료 이벤트만 받도록 합니다.

  - 콘솔 결과는 다음과 같습니다.

    ```
    ——— Example of: ignoreOutput ———
    Completed with: finished   
    ```



## 값 찾기

- Swift 기본 라이브러리에서도 `first(where:)` 와 `last(where:)` 를 제공하듯이, Combine에도 동일한 이름의 operator가 있습니다. 이들은 publisher가 방출하는 값 중 *오직* 첫 번째 또는 마지막 값만을 전달합니다.

### `first(where:)`

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/5.%20first.png?raw=true" width = 400>

- 흥미로운 점은 이 operator는 *lazy* 하다는 겁니다. 즉, 조건에 일치하는 값을 찾는데 필요한만큼의 값만 취합니다. 일치하는 것을 찾으면 subscription을 취소하고 완료시킵니다. 다음 코드를 확인해봅시다.

  ```swift
  example(of: "first(where:)") {
    // 1
    let numbers = (1...9).publisher
    
    // 2
    numbers
      .first(where: { $0 % 2 == 0 })
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `9` 까지 숫자를 방출하는 새로운 publisher를 생성합니다.

  - 2: `first(where:)` 를 이용하여 *짝수* 에 해당하는 첫 번째 방출 값을 찾습니다.

  - 콘솔 결과 값은 다음과 같습니다.

    ```
    ——— Example of: first(where:) ———
    2
    Completed with: finished
    ```

- 예상했던대로 작동하죠? 하지만 잠깐만요. 앞서 말한대로 정말 조건에 맞는 값을 찾은 다음에 취소되는게 맞을까요? 확인하기 위해 `print("numbers")` operator를 위 코드의 `numbers` 바로 아래에 추가한 뒤 확인해봅시다. 콘솔 결과는 다음과 같아요. 

  ```ㅇ
  ——— Example of: first(where:) ———
  numbers: receive subscription: (1...9)
  numbers: request unlimited
  numbers: receive value: (1)
  numbers: receive value: (2)
  numbers: receive cancel
  2
  Completed with: finished
  ```

  - 보시다시피 `first(where:)` 가 일치하는 값을 찾으면 subscription을 통해 취소를 전송하여 더 이상 값을 방출하지 않습니다. 편리하네요!



### `last(where:)`

- `first(where:)` 에 반대 개념인 이 아이는 아주 탐욕스럽습니다. 조건에 해당하는 값을 찾기 위해서 값이 모두 방출될 때까지 기다리거든요. 이 때문에 upstream publisher는 반드시 특정시점에 종료되는 아이여야 합니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/6.%20last.png?raw=true" width = 400>

- 다음 코드를 살펴봅시다.

  ```swift
  example(of: "last(where:)") {
    // 1
    let numbers = (1...9).publisher
    
    // 2
    numbers
      .last(where: { $0 % 2 == 0 })
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `9` 까지 숫자를 방출하는 새로운 publisher를 생성합니다.

  - 2: `last(where:)` 를 이용하여 *마지막* 짝수 값을 확인합니다.

  - 콘솔 결과는 다음과 같습니다.

    ```
    ——— Example of: last(where:) ———
    8
    Completed with: finished
    ```

- 앞서 이 operator가 작동하려면 publisher가 반드시 완료되어야 한다고 했던 것 기억하시나요? 왜 그럴까요? 그러지 않으면 이 operator가 방출되는 값 중 조건에 맞는 것이 마지막인지 알 수 없기 때문에 publisher의 전체 범위를 알아야 합니다. 이러한 작동방식을 확인하기 위해 방금 작성한 코드 전체를 다음 코드로 대체해봅시다.

  ```swift
  example(of: "last(where:)") {
    let numbers = PassthroughSubject<Int, Never>()
    
    numbers
      .last(where: { $0 % 2 == 0 })
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
    
    numbers.send(1)
    numbers.send(2)
    numbers.send(3)
    numbers.send(4)
    numbers.send(5)
  }
  ```

  - 여기서는 `PassthroughSubject` 를 이용하여 이벤트가 전송되도록 합니다. 위 코드를 실행해보면 아무 값도 찍히지 않는 것을 확인할 수 있어요. 

  - 해당 publisher는 절대 완료되지 않기 때문에 조건에 맞는 값을 확인할 방법이 없습니다. 이를 수정하려면 다음 코드를 마지막 라인에 추가해주어야 합니다. subject가 완료되었음을 알 수 있게요. 

    ```swift
    numbers.send(completion: .finished)
    ```

  - 다시 실행해보면 잘 작동하는 것을 알 수 있습니다. 

    ```
    ——— Example of: last(where:) ———
    4
    Completed with: finished
    ```

    

## 값 무시하기

- 값을 내려주는 것은 publisher를 사용할 때 종종 활용할 수 있는 유용한 기능입니다. 예를 들어 한 publisher의 값을 무시하고 두 번째 publisher가 게시를 시작할 때를 기다리거나 스트림이 시작할 때 특정 양의 값을 무시하고 값을 주려는 경우 사용할 수 있습니다.

### `dropFirst`

- 관련하여 세 개의 publisher를 제공하는데 먼저 가장 간단한 놈인 `dropFirst` 부터 확인해보죠.
- `dropFirst` operator는 기본 값은 `1` 이며 1일 경우 생략가능한  `count` parameter를 가지고 있습니다. `count` 에 해당하는 개수의 값들은 무시하고 그 다음 값부터 방출합니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/7.%20dropFirst.png?raw=true" width = 400>

- 코드로 확인해봅시다.

  ```swift
  example(of: "dropFirst") {
    // 1
    let numbers = (1...10).publisher
    
    // 2
    numbers
      .dropFirst(8)
      .sink(receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `10` 까지의 숫자를 방출하는 publisher를 생성합니다.

  - `dropFirst(8)` 을 이용하여 처음 8개의 값은 무시하고 그 다음 값인  `9` 와 `10` 만 방출되게 합니다. 

  - 콘솔 결과는 다음과 같습니다.

    ```
    ——— Example of: dropFirst ———
    9
    10
    ```



### `drop(while:)`

- 이 operator는 closure를 사용하고 해당 조건이 만족될 때까지 publisher가 내보낸 값을 무시하는 매우 유용한 녀석입니다. closure 내 구현한 조건이 충족되는 즉시 값이 통과되기 시작합니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/8.%20dropWhile.png?raw=true" width = 400>

- 코드로 확인해봅시다. 

  ```swift
  example(of: "drop(while:)") {
    // 1
    let numbers = (1...10).publisher
    
    // 2
    numbers
      .drop(while: { $0 % 5 != 0 })
      .sink(receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `10` 까지의 숫자를 방출하는 publisher를 생성합니다.

  - 2: `drop(while:)` 을 사용하여 첫 번째 5의 배수가 나올 때까지 기다리도록 합니다. 조건에 부합하는 값이 나오는대로 값은 흐르기 시작하고 더 이상 값을 무시하지 않습니다.

  - 콘솔 결과입니다.

    ```
    ——— Example of: drop(while:) ———
    5
    6
    7
    8
    9
    10
    ```

  - 보시다시피 `5` 를 받자마자 "5로 나눌 수 있니?" 라는 질문에 `true` 라는 답이 나왔고 따라서 `5` 부터 이후 전체 값을 방출하게 된 것입니다.

- 어떠신가요? 보셨을 때 이거 `filter` 아니야? 라고 생각할 수 있습니다. 둘 다 closure를 받고 closure 내의 조건으로 방출되는 값을 조정하니까요. 하지만 차이점이 있습니다. 

  - 첫 번째는 `filter` 는 closure 가 `true` 를 반환할 때 값을 방출하지만 `drop(while:)` 은 closure 가 `true` 가 되기 전까지 값을 무시한다는 겁니다. 
  - 더 중요한 두 번째 차이점은 `filter` 는 모든 값이 upstream으로부터 나오기 전까지는 멈추지 않는다는거죠. `filter` 의 조건이 `true` 로 이미 나왔더라도 방출되는 값들은 계속 조건에 부합하는지 확인받습니다. 반면에 `drop(while:)` 은 조건에 부합한 뒤에는 *절대*  재확인하지 않습니다. 

- 차이점 확인을 위해 위 예제의  `.drop(while: { $0 % 5 != 0 })` 부분을 다음 코드로 대체해봅시다.

  ```swift
  .drop(while: {
    print("x")
    return $0 % 5 != 0
  })
  ```
  - 확인을 위해 `print("x")` operator를 삽입해봤어요. 콘솔 결과는 다음과 같습니다.

    ```
    x
    x
    x
    x
    x
    5
    6
    7
    8
    9
    10
    ```

  - 보시다시피 `x` 는 정확히 5번만 반복됩니다. 조건에 부합하는 `5` 가 나온 이후에는 closure가 다시 실행되지 않는 것이죠. 



### `drop(untilOutputFrom:)`

- filtering 계열 중 가장 정교한 operator 입니다. 한 번 상황을 상상해볼까요? 사용자가 버튼을 여러 번 탭하고 있어요. 하지만 여러분들은 `isReady` 라는 publisher가 어떤 결과를 방출하기 전까진 이 모든 액션을 무시하고 싶습니다. 이런 상황에 이 녀석은 아주 유용합니다. `drop(untilOutputFrom:)` 은 두 번째 publisher가 값 방출을 시작하기 전까진 방출되는 값을 무시합니다. 

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/9.%20dropUntilOutPutFrom.png?raw=true" width = 400>

- 가장 위에는 `isReady` 스트림이 있구요, 두 번째가 사용자의 탭 이벤트 스트림인데 이벤트들을 `drop(untilOutputFrom:)` 를 통해 `isReady` publishe 를 argument로 뒀습니다.

- 코드로 확인해볼까요?

  ```swift
  example(of: "drop(untilOutputFrom:)") {
    // 1
    let isReady = PassthroughSubject<Void, Never>()
    let taps = PassthroughSubject<Int, Never>()
    
    // 2
    taps
      .drop(untilOutputFrom: isReady)
      .sink(receiveValue: { print($0) })
      .store(in: &subscriptions)
    
    // 3
    (1...5).forEach { n in
      taps.send(n)
      
      if n == 3 {
        isReady.send()
      }
    }
  }
  ```
  - 1: 값을 보낼 2개의 `PassthroughSubject` 를 생성합니다. 하나는 `isReady` 로 또 다른 subject로 사용자의 탭 이벤트가 발생할 동안 기다릴 녀석이죠. 

  - 2: `drop(untilOutputFrom:)` 을 이용하여 `isReady` 가 최소한 한 개 이상의 값을 방출하기 전까진 어떠한 탭 이벤트도 무시하도록 합니다.

  - 3: 위 다이어그램에서 표현한 것처럼 다섯 개의 "탭" 이벤트를 보낼건데요, 3개의 탭을 보낸 후에는 `isReady` 에게도 값을 보내주세요.

  - 콘솔 결과는 다이어그램에서 표현한 것과 동일합니다.

    ```
    ——— Example of: drop(untilOutputFrom:) ———
    4
    5
    ```

  

## 값 제한하기

- 지금까지는 특정 조건이 충족될 때까지 값을 무시하거나 건너 뛰는 방법을 배웠습니다. 여기서는 이와 반대되는 상황을 해결합니다. 조건이 충족될 때까지 값을 수신한 후 publisher가 완료하도록 합니다. 예를 들어 양을 알 수 없는 값을 방출할 때 하나의 값만 원하고 나머지는 신경쓰고 싶지 않은 상황 같은 것이죠. 

- Combine은 `prefix` 계열의 operator를 이용하여 이 상황을 해결하도록 합니다. 이들은 `drop` 계열과는 반대로 조건이 충족될 동안 값을 *받습니다.*

  

### `prefix(_:)` 

- `dropFirst` 와 반대되는 개념인 `prefix(_:)` 는 선언한 갯수*만큼*의 값을 수신한 뒤 완료됩니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/10.%20prefix.png?raw=true" width = 400>

- 다음 코드로 살펴봅시다.

  ```swift
  example(of: "prefix") {
    // 1
    let numbers = (1...10).publisher
    
    // 2
    numbers
      .prefix(2)
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `10` 까지 숫자를 방출하는 publisher를 생성합니다. 

  - 2: `prefix(2)` 를 사용하여 최초 2개의 값만 받도록 합니다. 해당하는 갯수의 값이 방출되는대로 publisher는 완료됩니다.

  - 결과는 다음과 같습니다.

    ```
    ——— Example of: prefix ———
    1
    2
    Completed with: finished
    ```

- `first(where:)` 처럼 이 operator는 *lazy* 합니다. 즉, 필요한만큼의 값만 받고 종료됩니다. 또한 조건에 맞는 값인  `1` 과  `2` 가 방출되어 완료된 후에 나오는 모든 `numbers` 는 무시합니다.

  

### `prefix(while:)`

- `prefix(while:)` 은 upstream publisher가 closure 내부 조건을 `true` 로 통과하는 값을 방출할 동안만 값을 받습니다. `false` 에 해당하는 값이 나오자마자 publisher 는 종료됩니다.

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/11.%20prefixWhile.png?raw=true" width = 400>

- 코드로 봅시다.

  ```swift
  example(of: "prefix(while:)") {
    // 1
    let numbers = (1...10).publisher
    
    // 2
    numbers
      .prefix(while: { $0 < 3 })
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
  }
  ```

  - 1: `1` 부터 `10` 까지 값을 방출하는 publisher 를 생성합니다.

  - 2: `prefix(while:)` 을 통해 `3` 보다 작은 값이 나오는 동안은 값을 받지만 그 이상의 값이 나오는 즉시 publisher는 완료됩니다.

  - 콘솔 결과 입니다.

    ```
    ——— Example of: prefix(while:) ———
    1
    2
    Completed with: finished
    ```



### `prefix(untilOutputFrom:)`

- 두 번째 publisher가 값 방출하기 전까진 값을 무시했던 `drop(untilOutputFrom)` 의 반대 개념으로 `prefix(untilOutputFrom)` 은 두 번째 publisher가 값을 방출할 때까지 값을 *수신*합니다. 

<img src = "https://github.com/fimuxd/Combine/blob/master/Lectures/04_Filtering%20Operators/12.%20prefixUntilOutputFrom.png?raw=true" width = 400>

- 코드를 봅시다.

  ```swift
  example(of: "prefix(untilOutputFrom:)") {
    // 1
    let isReady = PassthroughSubject<Void, Never>()
    let taps = PassthroughSubject<Int, Never>()
    
    // 2
    taps
      .prefix(untilOutputFrom: isReady)
      .sink(receiveCompletion: { print("Completed with: \($0)") },
            receiveValue: { print($0) })
      .store(in: &subscriptions)
    
    // 3
    (1...5).forEach { n in
      taps.send(n)
      
      if n == 2 {
        isReady.send()
      }
    }
  }
  ```

  - 1: 두 개의 `PassthroughSubject` 를 생성합니다. 

  - 2: `prefix(untilOutputFrom: isReady)`  를 통해 `isReady` 가 최소 하나의 값을 방출할 동안 탭 이벤트를 받도록 합니다.

  - 3: 위 다이어그램과 동일하게 다섯 개의 "탭" 이벤트를 보낼건데요, 두 개를 보낸다음엔 `isReady` 에도 값을 보내주세요.

  - 콘솔 결과 입니다.

    ```
    ——— Example of: prefix(untilOutputFrom:) ———
    1
    2
    Completed with: finished
    ```



## Summary

- filtering operator를 사용하면 upstream publisher가 방출한 값을 downstream, 다른 operator 들에게 보낼 수 있습니다.
- 값 자체에 신경 쓰지 않고 완료 이벤트만을 원할 경우엔 `ignoreOutput` 을 쓸 수 있습니다.  
- `first(where:)` `last(where:)` 를 사용하여 조건과 일치하는 첫 번째 또는 마지막 값을 찾을 수 있습니다.
- `first` 계열의 operator는 *lazy* 합니다. 필요한만큼의 값만 취한 다음 완료해버립니다. `last` 스타일의 operator 는 욕심쟁이죠. 조건을 충족시키는 마지막 값을 결정하기 위해 방출하는 값의 전체 범위를 모두 알아야 합니다.
- `drop` 계열의 operator를 사용하여 downstream 에 값을 보내기 전에 upstream publisher가 방출한 값들을 무시할 수 있습니다.
- 마찬가지로  `prefix` 계열 operator를 사용하여 완료 전에 upstream publisher가 방출할 수 있는 값의 개수를 제어할 수 있습니다.



***

##### Artwork/images/designs: from Combine: Asynchronous Programming with Swift, available at www.raywenderlich.com