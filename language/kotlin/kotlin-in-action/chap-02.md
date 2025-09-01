# 2.1 기본 요소: 함수와 변수

## 인자 없는 함수의 예

```kotlin
fun main() {
    println("Hello, world!")
}
```

- `fun`: 함수 선언
- `main()`: 진입점 인자 생략 가능
- 클래스 안에 함수를 넣을 필요 없음
- 세미콜론 필요 없음

## 인자 있는 함수의 예

```kotlin
fun main(a: Int, b: Int): Int {
    return if (a > b) else b
}
```

- 파라미터 이름 작성 후 타입 작성
  - 타입 생략 가능 (라이브러리 개발 시에는 명시적으로 드러내는 걸 권장)
- if는 식으로 취급, 따라서 값을 만들어 냄

    ```kotlin
    fun max(a: Int, b: Int): Int = if (a > b) a else b // 등호, 식으로만 구성할수도 있음 (식 본문 함수)
    ```
  - 식 (expression): 값을 만들어내며, 다른 식의 하위 요소로 계산 참여
  - 문 (statement): 값을 만들어내지 않음 (값을 변수에 대입해도 그 대입 연산 자체는 아무 값도 돌려주지 않음)

## val과 var의 특징

- `val`은 변경 불가능 (읽기 전용), `var`는 변경 가능
  - 객체가 `val` 로 선언되어 있다면 내부 속성 변경은 가능
- 둘 다 처음에 타입만 정의해두고 이후 따로 선언하는 것 (즉시 초기화 하지 않는 것) 가능

## 문자열 템플릿

```kotlin
fun main() {
    val name = readln()
    println("Hello, ${if (name.isBlank()) "someone" else name}!")
    // Blank input: Hello, someone!
    // "Seb" input: Hello, Seb!
}
```

- 변수 이름 앞에 $ 를 붙이면 변수를 문자열 안에 넣을 수 있음
- 문자열 템플릿 안의 중괄호로 둘러싼 식 안에서 조건, 큰따옴표를 넣을 수 있음

# 2.2 행동과 데이터 캡슐화: 클래스와 프로퍼티

## 클래스 & 프로퍼티

```kotlin
class Person(
    val name: String,
    var isStudent: Boolean
)

fun main() {
    val person = Person("Bob", true) // new 키워드 사용 x
    println(person.name) // getter
    person.isStudent = false // setter
}

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}
```

- 기본적으로 공개 게터/세터 제공
- 커스텀 접근자 생성 가능

## 파일 디렉터리 관련

- 같은 패키지에 속해 있다면 다른 파일에서 정의한 선언이라도 직접 사용 가능
- 다른 패키지에 정의한 선언이라면 해당 선언을 import 해야 함
- 여러 클래스를 같은 파일에 넣거나 파일의 이름도 마음대로 설정할 수 있으나, 대부분의 경우 자바와 같이 패키지별로 디렉터리를 구성하는 것을 권장 (특히 자바와 코틀린을 함께 이용하는 프로젝트라면)

# 2.3 선택 표현과 처리: 이넘과 when

## enum

```kotlin
enum class Color(
    val r: Int, // 이넘 상수의 프로퍼티 정의
    val g: Int,
    val b: Int
) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    BLUE(75, 0, 130);
    
    fun rgb() = (r * 256 + g) * 256 + b // 이넘 클래스 안에서 프로퍼티 정의
    fun printColor() = println("$this is $rgb") // 이넘 클래스 안에서 메서드 정의
}

fun main() {
    println(Color.BLUE.rgb) // 255
    Color.BLUE.printColor() // BLUE is 255
}
```

## when (자바의 switch)

```kotlin
fun getMnemoic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.BLUE -> "Battle"
        // 여러 값도 대응 가능
        Color.INDIGO, Color.VIOLET -> "warm"
    }

fun main() {
    println(getMnemoic(Color.BLUE) // Battle
}

fun getWarmthFromSensor() =
    when (val color = measureColor()) {
        // color가 when 안에서만 쓸 수 있도록 제한됨
        ...
    }
```

- 자바와 달리 각 분기의 끝에 `break` 을 붙일 필요 없음
- when의 인자에 변수를 선언 및 할당하면, when 안에서만 쓸 수 있는 변수로 제한할 수 있음.
- when이 식으로써 기능한다면, 값을 무조건 할당해야 하므로 `else` 값 할당이 필수.

## 스마트 캐스트

코틀린에서는 is 로 타입 검사를 하고, 이후 캐스팅을 명시적으로 지정하지 않아도 바로 타입 이용이 가능함.

```kotlin
fun eval(e: Expr): Int =
    // 식으로써 기능하기에 else 필수
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

# 2.4 대상 이터레이션: while과 for 루프

## while

```kotlin
// while
while (조건) {
    /* ... */
    if (shouldExit) break
}

// do-while
do {
    if (shouldSkip) continue
    /* ... */
} while (조건)

// 내포된 루프에서 레이블 지정
outer@ while (outerCondition) {
    while (innerCondition) {
        if (shouldExitInner) break
        if (shouldSkipInner) continue
        if (shouldExit) break@outer
        if (shouldSkip) continue@outer
        // ...
    }
    // ...
}
```

## for

코틀린에서는 범위를 이용하여 for문을 작성.

```kotlin
fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "피즈버즈"
    i % 3 == 0 -> "피즈 "
    i % 5 == 0 -> "버즈 "
    else -> "$i"
}

fun main() {
    for (i in 1..100) { // 1 ~ 100, 순차적
        print(fizzBuzz(i))
    }
    // 1 2 피즈 4 버즈 피즈 7 ...
}
// 오름차순: .. (ex: 1..100)
// 내림차순: downTo (ex: i in 100 downTo 1)
// 간격: step (ex: 1 in 100 downTo 1 step 2)
```

## 컬렉션, 맵

컬렉션과 맵에서는 in 을 이용.

```kotlin
// 컬렉션 이터레이션
fun main() {
    val collection = listOf("red", "green", "blue")
    for (color in collection) {
        print("$color ")
    }
    // red green blue
}

// 맵 이터레이션
fun main() {
    val binaryReps = mutableMapOf<Char, String>()
    for (char in 'A'..'F') {
        val binary = char.code.toString(radix = 2)
        binaryReps[char] = binary
    }
    
    for ((letter, binary) in binaryReps) {
        println("$letter = $binary")
    }
}
```

## 기타 사용 예

이 외에도 when 안에서 in 을 사용하거나, 특정 범위 안에 포함되는지도 검증할 수 있음.

```kotlin
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know..."
}

fun main() {
    println("Kotlin" in "Java".."Scala") // 알파벳 순서 기준 Java <= Kotlin <= Scala
}
```

# 2.5 코틀린에서 예외 던지고 잡아내기

```kotlin
val percentage =
    if (number in 0..100)
        number
    else
        throw IllegalArgumentException(
            "A percentage value must be between 0 and 100: $number")

fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
    
    println(number)
}
```

- 코틀린에서는 자바와 다르게 체크 예외와 언체크 예외를 구분하지 않음.
- try-catch-finally 의 기본 원리는 자바와 동일.
- 예외를 throw 할 때, new 연산자를 작성할 필요가 없음.
- 자바에서는 try가 문으로만 쓰였지만, 코틀린에서는 식으로써도 기능하여 값 할당이 가능함.

# 요약

- 함수를 정의할 때 `fun` 키워드를 사용한다. `val`과 `var`는 각각 읽기 전용 변수와 변경 가능한 변수를 선언할 때 쓰인다.
- val 참조는 읽기 전용이지만 val 참조가 가리키는 객체의 내부 상태는 여전히 변경 가능할 수 있다.
- 문자열 템플릿을 사용하면 지저분하게 문자열을 연결하지 않아도 된다. 변수 이름 앞에 $를 붙이거나 식을 `${식}`처럼 ${}로 둘러싸면 변수나 식의 값을 문자열 안에 넣을 수 있다.
- 코틀린에서는 클래스를 아주 간결하게 표현할 수 있다.
- 다른 언어에도 있는 `if`는 코틀린에서는 식이며, 값을 만들어낸다.
- 코틀린 `when`은 자바의 switch와 비슷하지만 더 강력하다.
- 어떤 변수의 타입을 검사하고 나면 굳이 그 변수를 캐스팅하지 않아도 검사한 타입의 변수처럼 사용할 수 있다. 컴파일러가 스마트 캐스트를 활용해 자동으로 타입을 바꿔준다.
- for, while, do-while 루프는 자바가 제공하는 동일한 키워드의 기능과 비슷하다. 하지만 코틀린의 for는 자바 for보다 더 편리하다. 특히 맵을 이터레이션하거나 인덱스를 사용해 컬렉션을 이터레이션할 때 코틀린 for가 더 편리하다.
- `1..5` (그리고 1..<5)와 같은 식은 범위를 만들어낸다. 범위와 순열은 코틀린에서 같은 문법과 추상화를 for 루프에 사용할 수 있게 해 주고, 어떤 값이 범위 안에 들어있거나 들어있지 않은지 검사하기 위해 in이나 !in과 함께 사용할 수 있다.
- 코틀린 예외 처리는 자바와 아주 비슷하다. 다만 코틀린에서는 함수가 던질 수 있는 예외를 선언하지 않아도 된다.
