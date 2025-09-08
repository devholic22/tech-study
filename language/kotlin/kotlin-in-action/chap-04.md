# 4장: 클래스, 객체, 인터페이스
## 4.1 클래스 계층 정의

### 코틀린 인터페이스

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!") // 디폴트 구현
}

// 구현체
class Button : Clickable {
    override fun click() = println("I was clicked")
}
```

- 인터페이스를 정의할 때는 class 대신 Interface를 사용
- 인터페이스는 여러 개 확장할 수 있으나 클래스는 하나만 확장 가능
- override 키워드 필수

만약 서로 다른 인터페이스가 이름이 겹치는 디폴트 구현 함수를 가지고 있고, 하나의 클래스가 그 인터페이스들을 구현한다면 컴파일러 오류가 발생하기에 직접 지정해야 함. (디폴트 구현 함수가 아닌 단순 함수라면 문제 X)

```kotlin
interface Clickable {
    fun showOff() = ...
}

interface Focusable {
    fun showOff() = ...
}

class Button : Clickable, Focusable {
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

### 클래스 제약 사항

기본적으로 코틀린 클래스에는 아래 제약이 있다.

- 하위 클래스를 만들 수 없다.
- 기반 클래스의 메서드를 하위 클래스가 재정의할 수 없다.

이는 클래스를 잘못 재정의했을 경우, 그 클래스의 취지와 다르게 사용될 여지가 있기 때문이다.

따라서 코틀린의 클래스와 메서드는 기본적으로 final이다. 그리고 어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 하고, 재정의를 허용하고 싶은 메서드나 프로퍼티도 open 변경자를 붙여야 한다.

```kotlin
open class RichButton : Clickable {
    fun disable() { ... } // final, 재정의 불가능
    open fun animate() { ... } // 재정의 가능
    override fun click() { ... } // 열러 있는 메서드 재정의 (만약 final을 붙였다면 불가능)
}
```

### 추상 클래스

```kotlin
abstract class Animated {
    abstract val animationSpeed: Double
    val keyframes: Int = 20
    open val frames: Int = 60
    
    abstract fun animate() // 추상 함수
    open fun stopAnimating() { ... }
    fun animateTwice() { ... }
}
```

- 추상 클래스는 인스턴스화 할 수 없음
- 추상 프로퍼티는 하위 클래스가 반드시 값이나 접근자를 제공해야 함
- 추상 함수의 하위 클래스는 추상 함수를 반드시 재정의해야 함
- 추상 클래스의 추상이 아닌 함수/프로퍼티는 기본적으로 열려 있지 않으나, open 지정 가능

### 가시성 변경자

## 4.2 뻔하지 않은 생성자나 프로퍼티를 갖는 클래스 선언

### 클래스 초기화: 주 생성자와 초기화 블록

코틀린은 주 생성자와 부 생성자를 구분하며, 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

```kotlin
class User(val nickname: String)
```

위의 코드처럼 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 주 생성자라 한다.

위의 행위를 그대로 구현하는 다른 작성법 (부 생성자)은 아래와 같다.

```kotlin
class User constructor(_nickname: String) {
    val nickname: String
    
    init {
        nickname = _nickname
    }
}
```

- constructor는 주/부 생성자 정의를 시작할 때 사용한다.
- init 키워드는 클래스 객체가 만들어질 때 실행될 초기화 코드가 들어간다.
- 주 생성자 선언 시 별다른 어노테이션이나 가시성 변경자가 없다면 constructor를 작성하지 않아도 된다.

아래 코드는 부모 클래스의 생성자를 호출하는 자식 생성자의 예이다.

```kotlin
open class User(val nickname: String) { ... }
class SocialUser(nickname: String): User(nickname) { ... }
```

생성자를 정의해두지 않았다면 기본 생성자가 만들어진다.

```kotlin
open class Button
class RadioButton: Button()
```

생성자를 private화 하여 인스턴스화를 막을 수도 있다.

```kotlin
class Secretive private constructor(private val agentName: String) {}
```

### 부 생성자: 상위 클래스를 다른 방식으로 초기화

```kotlin
open class Downloader {
    constructor(url: String?) { ... }
    constructor(uri: URI?) { ... }
}

// 상속
class MyDownloader : Downloader {
    constructor(url: String?): super(url) { ... }
    constructor(uri: URI?): super(uri) { ... }
}
```

이렇게 1:1 생성자를 매핑할 수도 있지만

```kotlin
class MyDownloader : Downloader {
    constructor(url: String?): this(URI(url))
    constructor(uri: URI?): super(uri)
}
```

이렇게 this()를 활용하여 절약할 수도 있다.

### 인터페이스에 선언된 프로퍼티 구현

코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있고, 이것은 이 인터페이스를 구현하는 클래스가 해당 프로퍼티의 값을 얻을 수 있는 방법을 제공해야 한다는 뜻이다.

```kotlin
interface User {
    val nickname: String
}

// 1
class PrivateUser(override val nickname: String): User

// 2
class SubscribingUser(val email: String): User {
    override val nickname: String
        get() = email.substringBefore('@'))
}

// 3
class SocialUser(val accountId: Int): User {
    override val nickname = getFacebookName(accountId)
}

fun getFacebookName(accountId: Int) = "..."
```

인터페이스의 프로퍼티에도 게터/세터를 둘 수 있지만, 인터페이스는 상태를 가지지 않기 때문에 이 때의 게터/세터에서는 뒷받침하는 필드를 참조할 수 없다. (단 결과를 반환하는 것은 가능하다.)

(뒷받침하는 필드란 프로퍼티에 해당되는 실제 필드를 의미한다. 근데 인터페이스에는 상태를 담을 수 없으므로 필드 저장이 불가능하다.)

```kotlin
interface EmailUser {
    val email: String // 무조건 재정의 필수
    val nickname: String // 상속 가능
        get() = email.substringBefore('@') // nickname을 사용할 수 없으나 email 계산식 이용 가능
}
```

### 게터와 세터에서 뒷받침하는 필드에 접근

게터와 세터에서 뒷받침하는 필드에 접근하고자 할 경우에는 field 를 쓰면 된다.

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println(
                """
                Address was changed for $name:
                "$field" -> "$value". // 뒷받침하는 필드 값 읽기
                """.trimIndent())
            field = value // 뒷받침하는 필드 값 변경
        }
}
```

field를 사용하지 않는 커스텀 접근자 구현을 정의한다면, 컴파일러는 프로퍼티가 아무 정보도 저장하지 않는 것으로 이해하고 뒷받침하는 필드를 생성하지 않는다.

```kotlin
class Person(var birthYear: Int) {
    var ageIn2050
        get() = 2050 - birthYear // 게터 안에 필드 참조 X
        set(value) {
            birthYear = 2050 - value // 세터 안에 필드 참조 X, 뒷받침하는 필드 생성 안됨
        }
}
```

### 접근자의 가시성 변경

접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같으나, set 앞에 가시성 변경자를 추가해서 이를 제어할 수 있음.

```kotlin
class LengthCounter {
    var counter : Int = 0
        private set // setter 막기
    
    fun addWord(word: String) { ... }
}
```

## 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임

### 데이터 클래스

코틀린의 데이터 클래스는 toString(), equals(), hashCode()를 모두 기본적으로 구현한다.

equals, hashCode는 주 생성자에 나열된 모든 프로퍼티를 고려하여 만들어진다.

```kotlin
data class Customer(val name: String, val postalCode: Int)
```

뿐만 아니라, 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있도록 copy 메서드를 제공한다.

```kotlin
fun main() {
    val lee = Customer("lee", 1000)
    lee.copy(postalCode = 2000) // Customer("lee", 2000)
}
```

### 클래스 위임

코틀린에서는 하위 클래스가 상위 클래스의 메서드를 재정의하면서 상위 클래스가 의도했던 방향과 다르게 재정의될 가능성을 사전에 방지하기 위해 전부 final 로 만들고 반드시 필요한 경우에는 open으로 열게 하였으나, 종종 상속을 허용하지 않는 클래스에게 새로운 동작을 추가해야 하는 경우가 있다.

이럴 때 데코레이터를 이용할 수 있는데, 상속할 수 없는 기존 클래스를 필드로 가진 데코레이터를 만들고 정의해야 하는 새로운 메서드는 데코레이터의 메서드로, 기존 기능이 그대로 필요한 메서드는 기존 클래스에게 요청을 전달한다.

```kotlin
class DelegatingCollection<T>: Collection<T> {
    private val innerList = arrayListOf<T>() // 기존 클래스
    
    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun contains(element: T): Boolean = innerList.contains(element)
    ...
}
```

다만 이 방식은 준비해야 할 코드가 많은데, 코틀린에서는 쉽게 by 키워드를 이용하면 된다.

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = mutableListOf<T>()
): Collection<T> by innerList
```

## 4.4 object 키워드: 클래스 선언과 인스턴스 생성을 한꺼번에 하기

## 4.5 부가 비용 없이 타입 안정성 추가: 인라인 클래스

## 요약

- 코틀린의 인터페이스는 자바 인터페이스와 비슷하지만 디폴트 구현과 프로퍼티도 포함할 수 있다.
- 모든 코틀린 선언은 기본적으로 final이며 public이다.
- 선언이 final이 되지 않게 만들려면 (상속과 오버라이딩이 가능하게 하려면) 앞에 open을 붙여야 한다.
- internal 선언은 같은 모듈 안에서만 볼 수 있다.
- 내포 클래스는 기본적으로 내부 클래스가 아니다. 바깥쪽 클래스에 대한 참조를 내포 클래스 안에 포함시키려면 inner 키워드를 클래스 선언 앞에 붙여야 한다.
- sealed 클래스를 직접 상속하는 클래스나 sealed 인터페이스에 대한 구현들은 모두 컴파일 시점에 컴파일러에 보여야 한다.
- 초기화 블록과 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화 할 수 있다.
- field 식별자를 통해 프로퍼티 접근자 (게터와 세터) 안에서 프로퍼티의 데이터를 저장하는 데 뒷받침하는 필드를 참조할 수 있다.
- 데이터 클래스를 사용하면 컴파일러가 equals, hashCode, toString, copy 등의 메서드를 자동으로 생성해준다.
- 클래스 위임을 사용하면 위임 패턴을 구현할 때 비슷한 위임 코드를 반복하는 것을 피할 수 있다.
- 객체 선언은 싱글턴 클래스를 정의하는 코틀린식 방법이다.
- (패키지 수준 함수의 프로퍼티와 더불어) 동반 객체는 자바의 정적 메서드와 필드 정의를 대신한다.
- 동반 객체도 다른 (싱글턴) 객체와 마찬가지로 인터페이스를 구현할 수 있으며, 동반 객체에 대해서도 확장 함수와 프로퍼티를 정의할 수 있다.
- 코틀린의 객체 식은 자바의 익명 내부 클래스를 대신하며, 여러 인터페이스를 구현하거나 객체가 포함된 영역에 있는 변수의 값을 변경할 수 있는 등 자바 익명 내부 클래스보다 더 많은 기능을 제공한다.
- 인라인 클래스를 사용하면 수명이 짧은 객체를 많이 할당함으로써 발생할 수 있는 성능 저하를 피하면서도 프로그램에 새로운 타입 안정성 계층을 추가할 수 있다.
