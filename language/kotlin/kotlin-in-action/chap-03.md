# 3장: 함수 정의와 호출

# 3.1 코틀린에서 컬렉션 만들기

```kotlin
// Set
val sets = setOf(1, 7, 53)

// List
val lists = listOf(1, 7, 53)

// Map
val maps = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

// 기능
val strings = listOf("first", "second", "fourteenth")
strings.last() // 마지막 원소
strings.shuffled() // 섞기
sets.sum() // 합계
```

- 코틀린 컬렉션 인터페이스는 기본적으로 읽기 전용
- 코틀린은 표준 자바 컬렉션 클래스를 사용 → 코틀린, 자바 간 호출 시 서로 변환할 필요 없음
- 코틀린 컬렉션은 자바 컬렉션보다 더 많은 기능을 제공함

# 3.2 함수를 호출하기 쉽게 만들기

### 함수의 이름 인자 전달과 기본값 지정

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String = "" // 기본값 지정 가능
): String { ... }

// 이름 붙여 호출
joinToString(collection, separator = " ", prefix = " ", postfix = ".")
```

- 코틀린에서는 인자에 이름을 붙여 전달할 수 있다.
- 코틀린에서는 인자의 기본값을 지정할 수도 있다.

### 정적 유틸리티 클래스 없애기

자바에서는 유틸리티 메서드를 만들 때 무조건 클래스가 필요했지만, 코틀린에서는 최상단에 함수만 작성해도 동작한다.

```kotlin
fun joinToString(...): String { ... }
```

이를 자바로 컴파일하면, 파일 이름에 대응되는 클래스가 만들어진다. (ex: join.kt일 경우 JoinKt)

```java
public class JoinKt {
    public static String joinToString(...) { ... }
}
```

함수 뿐 아니라 프로퍼티도 최상단으로 올릴 수 있다.

```kotlin
// Kotlin
const val UNIX_LINE_SEPARATOR = "\n"

// Java
public static final String UNIX_LINE_SEPARATOR = "\n";
```

# 3.3 메서드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

코틀린에서는 특정 함수를 특정 클래스의 멤버 함수로써 호출할 수 있는 (선언은 클래스의 밖에 되어있는) 기능이 있다. 이는 기존 코드와 코틀린 코드를 자연스럽게 통합할 수 있도록 도와준다. 이를 확장 함수라 한다.

```kotlin
fun String.lastChar() : Char = this.get(this.length - 1)
// 수신 객체 타입.함수 = 수신 객체 (this)

// 실제 호출
import strings.lastChar // 임포트 필요

fun main() {
    println("Kotlin".lastChar())
}
```

단, 확장 함수에서는 클래스 내부에서만 사용할 수 있는 private / protected 멤버를 사용할 수 없다.

### 확장 함수는 재정의할 수 없다.

위의 설명에 기술하였듯이, 확장 함수는 선언이 클래스의 밖에 되어 있다. 따라서 아래 그림처럼, View와 Button 클래스의 밖에 따로 따로 정의되기 때문에 재정의가 먹히지 않는다.

<img width="659" height="285" alt="스크린샷 2025-09-02 오후 10 32 57" src="https://github.com/user-attachments/assets/674e7bcc-5957-4982-a0e2-3b174b02aeda" />

```kotlin
// Kotlin
open class View
class Button : View()

fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")

fun main() {
    val view: View = Button()
    view.showOff() // 출력: "I'm a view!"
}

// Java: 파일 이름명.함수(수신 객체)
class Demo {
    public static void main(String[] args) {
        View view = new Button();
        ExtensionsKt.showOff(view); // "I'm a view!"
    }
}

// ExtensionsKt 내부 코드: 컴파일 시의 타입으로 결정됨.
public final class ExtensionsKt {
    public static final void showOff(@NotNull View $this) {
        System.out.println("I'm a view!");
    }

    public static final void showOff(@NotNull Button $this) {
        System.out.println("I'm a button!");
    }
}
```

### 확장 프로퍼티

확장 프로퍼티도 프로퍼티에 수신 객체 클래스를 추가하면 된다.

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
```

자바에서는 StringUtilKt.getLastChar(”Java”), StringUtilKt.setLastChar(sb, ‘!’) 처럼 게터/세터를 명시적으로 호출해야 한다.
# 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

# 3.5 문자열과 정규식 다루기

# 3.6 코드 깔끔하게 다듬기: 로컬 함수와 확장

# 요약
- 코틀린은 자체 컬렉션 클래스를 정의하지 않지만 자바 클래스를 확장해서 더 풍부한 API를 제공한다.
- 함수 파라미터의 기본값을 정의하면 오버로딩한 함수를 정의할 필요성이 줄어든다. 이름붙인 인자를 사용하면 함수의 인자가 많을 때 함수 호출의 가독성을 더 향상시킬 수 있다.
- 코틀린 파일에서 클래스 멤버가 아닌 최상위 함수와 프로퍼티를 직접 선언할 수 있다. 이를 통해 코드 구조를 더 유연하게 만들 수 있다.
- 확장 함수와 프로퍼티를 사용하면 외부 라이브러리에 정의된 클래스를 포함해 모든 클래스의 API를 그 클래스의 소스코드를 바꿀 필요 없이 확장할 수 있다.
- 중위 호출을 통해 인자가 하나밖에 없는 메서드나 확장 함수를 더 깔끔한 구문으로 호출할 수 있다.
- 코틀린은 정규식과 일반 문자열을 처리할 때 유용한 다양한 문자열 처리 함수를 제공한다.
- 자바 문자열로 표현하려면 수많은 이스케이프가 필요한 문자열의 경우 3중 따옴표 문자열을 사용하면 더 깔끔하게 표현할 수 있다.
- 로컬 함수를 써서 코드를 더 깔끔하게 유지하면서 중복을 제거할 수 있다.
