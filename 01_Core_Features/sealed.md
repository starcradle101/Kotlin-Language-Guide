# Sealed

## 정의

`sealed` 제어자는 제한된 서브클래스를 만들기 위해 사용된다.

- `sealed`를 사용하면 자동으로 추상 클래스가 된다.

sealed class / interface의 서브타입들은 다음 조건을 만족해야 한다.

- sealed가 선언된 클래스 / 인터페이스와 같은 패키지 및 모듈에서 정의되어야 한다.
- 지역적이 될 수 없고, 객체 표현식을 사용해 정의할 수 없다.

### 지역 클래스

**지역 클래스(Local Class)** 란 함수나 코드 블럭 내에 정의된 클래스를 의미한다.

```kotlin
fun someFunction() {
  class LocalError : SealedResult // Sealed 타입 상속 불가
}
```

sealed class / interface의 서브타입은 최상위 클래스이거나, sealed 타입 내부에 중첩된 클래스여야 한다.

이는 sealed의 목적을 생각하면 이해하기 쉽다.

- **가능한 모든 서브타입을 제한**하고 컴파일러에게 알리는 것

반면 지역 클래스는 함수 실행 범위 내에서만 존재하기 때문에 컴파일러가 추적하기 어렵고, sealed 타입의 목적과 어긋난다.

### 객체 표현식

**객체 표현식(Object Expression)** 이란 익명 객체를 생성할 때 주로 사용된다.

```kotlin
interface Callback

val hander = object Callback { /* ... */ }

sealed class SomeResult
object Success : SomeResult()
// ...
val error = object: SomeResult() { /* ... */ } // 허용되지 않음
```

sealed class / interface를 익명 객체로 구현하는 것이 금지된 이유는 **이름이 없어 서브타입에 포함해 추적하기 어렵기 때문**이다.

### `enum`과의 차이

enum은 값의 집합을 표현할 때 사용한다. 클래스는 값보다 더 많은 역할을 하며, 인스턴스를 여러 개 만들거나, 데이터를 담을 수 있다.

enum과 비교했을 때 sealed의 가장 큰 장점은 when에서 is 표현식을 통해 계층구조에서 가능한 모든 타입을 다룰 수 있다는 것이다.

## sealed와 when 사용 시 주의할 점

`sealed`를 사용할 때 만약 모든 서브타입을 case로 명시하면, 컴파일러는 이러한 `when` 문이 철저하게 처리되고 있다고 판단해 else 분기를 강제하지 않는다.

문제는 다음과 같은 상황에서 발생한다. 만약 내가 외부 라이브러리를 사용하고 있고, 해당 라이브러리 내에 sealed 처리된 클래스가 있다고 가정해보자.

```kotlin
sealed class Event {
  object Clicked : Event()
  class DataReceived(val data: String) : Event()
}
```

나의 모듈에서는 다음과 같이 해당 Event를 처리하고 있다.

```kotlin
fun handleEvent(event: Event) {
  when(event) {
    is Clicked -> println("버튼 클릭")
    is DataReceived -> println("데이터 수신: ${event.data})
  }
}
```

현재 상황에서는 else가 없어도 컴파일 오류가 발생하지 않는데, **가능한 모든 서브타입을 다 다루기 때문**이다.

그런데 해당 라이브러리의 개발자가 추후 Event에 새로운 서브타입을 추가했다고 가정해보자.

```kotlin
sealed class Event {
  object Clicked : Event()
  class DataReceived(val data: String) : Event()
  object NewEvent: Event() // 새로운 이벤트 추가
}
```

이제 나의 모듈을 새로운 라이브러리 버전으로 빌드하려고 하면, when 문에서 exhaustive 관련 오류가 발생한다.

## 용례

일반적으로 안드로이드 개발에서는 화면의 상태, 이벤트 등에 대해 sealed class를 사용한다.

```kotlin
sealed class UiState {
  object Loading : UiState()
  data class Success(val users: List<User>) : UiState()
  data class Failure(val message: String): UiState()
}

fun render(state: UiState) {
    when (state) {
        UiState.Loading -> showLoadingIndicator()
        is UiState.Success -> showUserList(state.users)
        is UiState.Error -> showErrorDialog(state.message)
        UiState.Idle -> showInitialScreen()
    }
}
```
