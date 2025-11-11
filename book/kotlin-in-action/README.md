# Kotlin in Action 2/E

> 이 글은 [Kotlin IN Action 2/E](https://product.kyobobook.co.kr/detail/S000215768644) 책을 읽고 정리한 내용입니다.

대상 독자

* 자바 경험이 있는 개발자

* 서버 개발자 or 안드로이드 개발자와 같이 JVM에서 실행될 프로젝트를 구축하는 모든 개발자

---

# 1장. 코틀린이란 무엇이며, 왜 필요한가?

## 1.1 코틀린의 주요 특성

* `정적 타입` 지정 언어로, 컴파일 시점에 많은 오류를 잡아낼 수 있다.

* 코틀린의 주목적은 현재 자바가 사용되고 있는 모든 용도에 적합하면서도 **더 간결하고 생산적이며 안전한 대체 언어를 제공**하는 것이다.

* 정적 타입 지정으로 인해 코틀린 성능, 신뢰성 유지보수성이 모두 좋아진다.

    * 성능: 어떤 메서드를 호출해야 할지를 살펴보지 않아도 된다.

    * 신뢰성: 컴파일러가 타입을 사용해 프로그램을 검증하므로, 실행 시점에 프로그램이 실패할 가능성이 줄어든다.

* 객체지향과 함수형 프로그래밍 스타일을 모두 지원한다.

* `코루틴`을 통해 동시성과 비동기 프로그래밍의 문제를 해결할 수 있다.

    * 비동기 코드를 순차적 코드와 비슷한 로직을 작성할 수 있으며,

    * 자식-부모 관계로 동시성 코드를 구조화할 때 도움이 된다.

## 1.2 코틀린의 철학

* 코틀린은 실용적인 언어다. (실제 문제를 해결하기 위해 만들어진)

    * 대규모 시스템을 개발해본 다년간의 IT 업계 경험을 바탕으로 이뤄졌으며, 수많은 소프트웨어 개발자가 선택한 언어이다.

    * 인텔리제이 IDEA 플러그인의 개발과 컴파일러의 개발이 맞물려 있다. 이를 통해 개발 환경에 대한 생산성을 향상시킨다.

* 코틀린은 간결하다.

    * 코드가 더 간단하고 간결할수록 내용을 파악하기 쉽다. -> '간결하다'라는 의미는 해당 언어로 작성된 코드를 읽을 때 그 의도를 쉽게 파악할 수 있다는 것이다.

    * 코틀린의 `타입 추론`으로 인해 직접 타입을 지정할 필요가 없다. -> 컴파일러가 해당 문맥으로부터 타입을 추론할 수 있기 때문이다.

* 코틀린은 안전하다.

    * JVM에서 실행됨으로써 메모리 안전성을 보장받고 버퍼 오버플로우를 방지할 수 있다.

    * 읽기 전용 변수(val 키워드)를 정의할 수 있고, 불변 변수들을 묶어 불변(데이터) 클래스로 만들 수 있다.

* 코틀린은 상호운용성이 좋다.

    * 기존 자바 라이브러리를 최대한 활용한다. -> 대부분 자바 표준 라이브러리 클래스에 의존하며 코틀린에서 컬렉션을 더 쉽게 활용할 수 있게 해주는 함수를 몇 가지 더 확장할 뿐이다.

---

# 2장. 코틀린 기초

## 2.1. 함수와 변수

### 함수

함수 선언은 fun 키워드로 시작한다.

```kotlin
fun main() {
    println("Hello, world!") // 세미콜론 필요 없음.
}
```

파라미터와 반환값이 있는 경우 함수 선언을 아래와 같이 한다.

```kotlin
// 파라미터에서 변수 이름과 타입은 콜론(:) 으로 구분한다.
// 파라미터와 반환 타입 역시 콜론(:) 으로 구분한다.
// 블록 본문 함수(block body function) 라고 부른다.
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

아래와 같이 식 본문을 사용해서 함수를 더 간결하게 정의할 수 있다.

```kotlin
// 식 본문 함수(expression body function)
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

식 본문 함수의 경우에만 반환 타입이 생략 가능하고, 일반적으로 반환 타입을 사용한다.

### 변수

변수 선언은 키워드(val 또는 var)로 시작하고 그 뒤에 변수 이름이 온다.

변수 선언에서 타입을 지정해도 되고 생략해도 결과는 동일하게 나온다.

```kotlin
fun main() {
    val nickname: String = "fancy"
    val nickname2 = "fancy"
    println(nickname) // 결과: fancy
    println(nickname2) // 결과: fancy
}
```

* val(=value): 읽기 전용 참조로 선언하며, 초기화하고 나면 다른 값으로 대입할 수 없다.

* var(variable): 재대입 가능한 참조로 선언하며, 초기화가 이뤄진 다음에도 다른 값으로 대입할 수 있다.

    * 변수의 값은 변경될 수 있지만, 변수의 타입은 고정된다.

코틀린에서 기본적으로 **모든 변수를 `val` 키워드를 사용하고, 반드시 필요한 경우에 한해서만 해당 변수를 `var` 키워드로 변경**한다.

### 문자열 템플릿

표준 입력을 통해 이름을 지정하면 프로그램이 그 이름을 사용해 출력할 수도 있다.

```kotlin
fun main() {
    // 문자열 템플릿
    val input = readln()
    val name = if (input.isNotBlank()) input else "Kotlin"
    println("Hello, $name!")
    println("Hello, ${name}!")
}
// "fancy" 라고 입력하면 -> 결과가 "Hello, fancy!" 로 나온다.
```

## 2.2. 클래스와 프로퍼티

`클래스`란 데이터를 캡슐화하고 캡슐화한 데이터를 다루는 코드를 한 주체 안에 가두는 것을 의미한다.

`프로퍼티`란 필드와 접근자 메서드를 묶어 부른다.

코틀린은 기본 가시성으로 `public` 이므로 아래와 같이 지정자가 생략해도 된다.

```kotlin
class Person(
        // val은 읽기 전용으로 private 필드와 getter 를 제공한다.
        val name: String,
        // var은 변경 가능하므로 private 필드와 getter, setter 를 제공한다.
        var age: Integer,
)

val person = Person("fancy", 28)
println(person.name) // 프로퍼티로 직접 접근하면 getter가 호출된다.
println(person.age)
```

## 2.3 enum과 when

코틀린에서는 enum class 를 아래와 같이 정의한다.

```kotlin
enum class Color {
    RED,
    ORANGE,
    YELLOW
    ;
}
```

여기서 when 식을 직접 사용하면 아래와 같다.

```kotlin
// 함수의 반환값으로 when 식을 직접 사용한다.
fun getColor(color: Color) =
        when (color) {
            Color.RED -> "Red"
            Color.ORANGE -> "Orange"
            Color.YELLOW -> "Yellow"
        }

fun getColor2(color: Color): String {
    return when (color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "${color.name}"
    }
}

// 모든 분기 식에서 만족하는 조건을 찾을 수 없다면 else 분기를 계산한다.
fun getColor3(color1: Color, color2: Color, color3: Color) =
        when {
            (color1 == Color.RED || color2 == Color.ORANGE || color3 == Color.YELLOW) -> "RED ORANGE YELLOW"
            else -> throw RuntimeException()
        }

fun main() {
    println(getColor(Color.YELLOW)) // 결과: Yellow
    println(getColor2(Color.YELLOW)) // 결과: YELLOW
    println(getColor3(Color.RED, Color.ORANGE, Color.YELLOW)) // 결과: RED ORANGE YELLOW
}
```

### 스마트 캐스트

`스마트 캐스트`란 컴파일러가 타입을 대신 변환해주는 것을 의미한다.

* 스마트 캐스트를 사용한다면, 프로퍼티는 반드시 `val` 키워드여야 하며, 커스텀 접근자를 사용하면 안된다.

* var 이거나 커스텀 접근자를 사용하면 해당 프로퍼티에 대한 접근이 항상 같은 값을 내놓는다고 확신할 수 없기 때문이다.

아래는 스마트 캐스팅을 적용한 예시 코드이다.

```kotlin
interface Expr

// Num, Sum 에 있는 프로퍼티가 val 로 지정되어 있으므로 스마트 캐스팅을 지원한다.
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
        when (e) {
            is Num -> e.value
            is Sum -> eval(e.left) + eval(e.right)
            else -> throw IllegalArgumentException()
        }

fun main() {
    val sum = Sum(Num(3), Num(2))
    // is 로 타입을 한 번 검사하면, 컴파일러가 '해당 변수에 대한 타입이 보장된다'고 판단하고 자동으로 타입을 변환해준다.
    if (sum.left is Num) {
        println(sum.left.value) // 결과: 3
    }
    if (sum.right is Num) {
        println(sum.right.value) // 결과: 2
    }

    println(eval(Sum(Num(3), Num(2)))) // 결과: 5
}
/**
 * 최종 결과:
 * 3
 * 2
 * 5
 */
```

## 2.4 이터레이션

> for 루프의 다양한 사용법을 정리했다.

```kotlin
// 범위를 쓸 때는 `..` 연산자를 사용한다.
fun main() {
    // 1부터 10까지 출력
    for (i in 1..10) {
        println(i) // 결과: 1, 2, 3, ..., 10
    }

    // 1부터 10까지 2씩 증가
    for (i in 1..10 step 2) {
        println(i) // 결과: 1, 3, 5, 7, 9
    }

    // 10부터 1까지 2씩 감소
    for (i in 10 downTo 1 step 2) {
        print("$i, ") // 결과: 10, 8, 6, 4, 2
    }

    // map의 key, value를 for문으로 풀어낼 수 있다.
    for ((key, value) in mutableMapOf(Pair("A", 1))) {
        println("$key: $value") // 결과: A: 1
    }

    // withIndex를 활용하면 리스트의 index, 값을 가져올 수 있다.
    for ((index, value) in mutableListOf(1, 2, 3).withIndex()) {
        println("$index: $value")
    }
}
```

> in 연산자로 범위 검사를 할 수 있다.

```kotlin
fun main() {
    println(isLetter('q')) // 결과: true
    println("K" in "A".."Z") // 결과: true
}

fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z' // a <= c && c <= z로 변환된다.
```

### 예외 처리

코틀린에서 자바와 가장 큰 차이는 `throws` 절이 없다는 점이다.

코틀린은 IOException 과 같은 체크 예외가 아닌 모두 언체크 예외로 이루어져 있다.

(참고로 `체크 예외`란 명시적으로 처리해야만 하는 유형의 예외를 말한다.)

예외를 잡아내고 싶은 경우 try-catch 문을 사용한다.


---

# 3장. 함수 정의와 호출

- 자바와 달리 코틀린 컬렉션 인터페이스가 디폴트 값으로 `읽기 전용`이라는 사실을 기억하자.
- 코틀린에서는 함수의 디폴트 파라미터 값은 `함수 선언` 쪽에 인코딩된다는 사실을 기억하자.
    - 자바에서는 디폴트 파라미터 값이라는 개념이 존재하지 않는다.

**최상위 프로퍼티**

- 자바에서는 클래스 내에서만 변수가 사용 가능하지만, 코틀린에서는 top level에서 프로퍼티를 선언할 수 있다.

```kotlin
const var opCount = 0 //top level property
fun performOperation() {
    opCount++
    // ...
}
```

- 위의 코드와 같이 const 키워드를 사용하면 → 자바에서의 public static final 필드로 노출한 것과 같다.

**확장 함수**

- 코틀린 언어를 자바 프로젝트에 통합하는 경우, 코틀린으로 직접 변환할 수 없거나 변환하지 못한 기존 자바 코드를 처리할 수 있어야 한다. 이때 자바 API 를 재작성하지 않는 것이 효율적인데, 이런 경우 확장
  함수를 사용한다.
- `확장 함수`란 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.

```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
// String -> 수신 객체 타입(receiver type), this -> 수신 객체(receiver object)

fun String.lastChar2(): Char = get(length - 1)
// 수신 객체 멤버를 this 없이 접근할 수 있다.

fun main() {
    // NOTE: 3.3절
    println("Kotlin".lastChar()) // n
    println("Kotlin".lastChar2()) // n
}
```

- 위의 코드처럼 확장 함수를 사용하면 확장하고 싶은 클래스의 메서드나 프로퍼티에 직접 접근할 수 있다.
    - 확장 함수가 캡슐화를 깨지는 않는다는 사실을 기억하자.

- 확장 함수는 정적 메서드와 같은 특성을 가진다. → 확장 함수를 하위 클래스에 오버라이드 할 수 없다.
    - **클래스의 멤버가 아니기 때문에** 오버라이드가 불가능하다.
    - 확장 함수는 `정적(static)` 메서드로 컴파일한다는 사실을 기억하자.

```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button : View() {
    override fun click() = println("Button clicked") // 오버라이드 가능 -> 런타임에 실제 객체의 타입에 맞는 함수가 호출된다.
}

fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a Button!")

fun main() {
    // NOTE: 3.3.4절 - 확장 함수는 오버라이드할 수 없다.
    val view: View = Button() // 변수의 '타입'은 View, '실제 객체'는 Button -> 컴파일에 변수의 타입이 호출된다.
    view.showOff() // I'm a view!
}
```

- 위의 코드처럼 View 타입에 해당하는 showOff 확장 함수가 호출된다.
    - showOff()는 View 나 Button 클래스 내부에 존재하는 함수가 아니다.
    - 실제 클래스 바깥에 선언된 헬퍼 함수에 가깝다.
    - 코틀린이 컴파일할 때 이코드는 자바의 정적 메서드로 인식한다.
    - 컴파일 시점에 변수의 선언 타입에 맞는 함수가 고정이기 때문에, 런타임에 실제 객체를 따라가는 오버라이드가 일어나지 않는다.

### 문자열과 정규식 다루기

자바에서 split() 메서드를 사용할 때, split(".")가 마침표(.)를 기준으로 문자열을 나눌 것이라 오해하기 쉽다.

- 하지만 자바의 split() 메서드는 파라미터로 받은 문자열을 항상 정규식(Regex)으로 해석한다.
- 정규식에서 마침표(.)는 '모든 문자'를 의미하는 특수 기호이므로, 의도한 대로 동작하지 않는다.
- 그렇기 때문에 자바에서 문자 그대로 분리하기 위해서는 정규식 이스케이프 구문을 사용해야 한다. -> split("\\.")

반면, 코틀린에서는 여러 split 확장 함수를 제공하여 이 점을 개선했다.

- 코틀린은 정규식으로 분리하는 함수와 일반 텍스트로 분리하는 함수를 `파라미터 타입`으로 구분한다.
  - split(String): 파라미터가 일반 `String` 이면, 문자 그대로 분리한다.
  - split(Regex): 파라미터가 `Regex` 타입이면, 정규식으로 분리한다.

- 따라서 개발자는 전달하는 값의 타입을 통해, 정규식 분리인지 일반 텍스트 분리인지 의도를 명확하게 코드에 드러낼 수 있다.

```kotlin
fun main() {
    // NOTE: 3.5
    val ip = "192.168.0.1"

// 단순 문자열로 분리
    val partsString = ip.split(".")
    println("단순 문자열 분리: $partsString") // [192, 168, 0, 1]

// 모든 문자로 쪼갤 경우 Regex 로 만든다.
    val partsRegex = ip.split(Regex("\\."))
    println("정규식으로 분리: $partsString") // [192, 168, 0, 1]
}
```

이처럼 코틀린은 파라미터 타입을 명확히 구분함으로써, 개발자의 실수를 줄이고 코드의 가독성을 높여준다.

### 코드 깔끔하게 다루기: 로컬 함수와 확장

코드 깔끔하게 다듬기 위해서는 `로컬 함수`를 활용한다.

- 아래 코드와 같이 검증 코드를 로컬 함수로 분리하면 중복을 없애는 동시에 코드 구조를 깔끔하게 유지할 수 있다.

최초 작성된 버전(v1)

```kotlin
class User(val id: Int, val name: String, val address: String) {

    // 한 필드를 검증하는 로컬 함수를 정의한다.
    fun saveUser(user: User) {

        fun validate(user: User,
                     value: String,
                     filedName: String) {
            if (value.isEmpty()) {
                throw IllegalArgumentException(
                        "Can't save user ${user.id}: empty $filedName"
                )
            }
        }

        // 로컬 함수를 호출해서 각 필드를 검증한다.
        validate(user, user.name, "Name")
        validate(user, user.address, "Address")
    }
}
```

- 위와 같이 로컬 함수를 통해 다른 필드에 대한 검증도 쉽게 추가할 수 있다.
- 하지만 User 객체를 로컬 함수에게 전달해야 하는 점이 눈에 거슬린다.
- 다행히 로컬 함수는 **자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있어**서 아래와 같이 개선할 수 있다.

개선된 버전(v2)

```kotlin

class User(val id: Int, val name: String, val address: String) {

    fun saveUser(user: User) {

        // saveUser 함수의 user 파라미터를 중복 사용하지 않는다.
        fun validate(value: String,
                     filedName: String) {
            if (value.isEmpty()) {
                // 바깥 함수의 파라미터에 직접 접근할 수 있다.
                throw IllegalArgumentException(
                        "Can't save user ${user.id}: " + "empty $filedName"
                )
            }
        }

        // 로컬 함수를 호출해서 각 필드를 검증한다.
        validate(user.name, "Name")
        validate(user.address, "Address")
    }
}
```

- 위의 코드에 대해 User 클래스를 확장한 함수로 검증 로직을 만들면 아래와 같다.

개선된 버전 (v3)

```kotlin
class User(val id: Int, val name: String, val address: String) {
    fun User.validateBeforeSave() {
        fun validate(value: String,
                     filedName: String) {
            if (value.isEmpty()) {
                // User의 프로퍼티를 직접 사용할 수 있다.
                throw IllegalArgumentException(
                        "Can't save user $id: empty $filedName"
                )
            }
        }

        validate(name, "Name")
        validate(address, "Address")
    }
}
```

- `User.validateBeforeSave()` 와 같이 확장 함수를 로컬 함수로도 정의할 수 있다.
- 하지만 내포된 함수의 깊이가 깊어지면 코드를 읽기가 어려워지기 때문에, 일반적으로 한 단계만 함수를 내포하는 것을 권장한다.