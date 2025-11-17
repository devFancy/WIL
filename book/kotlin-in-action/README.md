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

## 4장. 클래스, 객체, 인터페이스

### 클래스 계층 정의

**코틀린 인터페이스**

- `class {클래스명} : {인터페이스명} {…}`
- 상속이나 인터페이스에서 모두 클래스 이름 뒤에 `콜론(:)`을 붙인다.
- 상위 클래스/인터페이스에 있는 프로퍼티나 메서드를 오버라이드할 때 override 변경자를 꼭 사용해야 한다. (자바의 @Override 어노테이션은 선택)
- 상속한 인터페이스의 동일한 메서드를 구현하고 호출하기 위해서는 아래와 같이 작성한다.

```kotlin
fun main() {
    val button = Button()
    button.showOff()
    // I'm clickable!
    // I'm a focusable!
    button.click()
    // I was clicked
}

class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")

    override fun showOff() {
        // 상위 타입의 구현을 호출할 때는 상위 타입의 이름을 <> 사이에 넣은 super를 사용한다.
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
    // 만약 상속한 구현 중 단 하나만 호출한다면 아래와 같이 작성한다.
    // override fun showOff() = super<Clickable>.showOff()
}

interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

interface Focusable {
    fun showOff() = println("I'm a focusable!")
}
```

**기본적으로 `final` 이 적용된다.**

- 자바와 달리 코틀린에서 모든 클래스와 메서드는 기본적으로 `final` 이다.
- 특별하게 하위 클래스에서 오버라이드하도록 의도된 클래스와 메서드가 아니라면 모두 `final` 로 만드는 것이 예기치 않게 동작하는 문제를 방지할 수 있다.
- 따라서 어떤 클래스의 상속을 허용하기 위해서는 클래스 앞에 `open` 변경자를 붙여야 한다. 그리고 오버라이드를 허용하고 싶은 메서드 또는 프로퍼티의 앞에도 `open` 변경자를 붙여야 한다.

```kotlin
fun main() {
    val themedButton = ThemedButton()
    themedButton.showOff() // I'm a themedButton!
}

open class RichButton : Clickable {
    fun disable() {} // 기본적으로 final 이기 때문에 하위 클래스가 이 메서드를 오버라이드할 수 없다.
    open fun animate() {} // open 변경자를 사용했기 때문에, 하위 클래스가 이 메서드를 오버라이드할 수 있다.
    override fun click() {} // 위와 동일.
}

class ThemedButton : RichButton() {
    override fun animate() {}
    override fun click() {}
    override fun showOff() = println("I'm a themedButton!")
}
```

- 추상 클래스는 아래와 같이 프로퍼티나 메서드 앞에 `abstract` 이 붙어 있으면 하위 클래스가 반드시 오버라이드해야 한다.

    ```kotlin
    abstract class Animated {
        abstract val animationSpeed: Double // 추상 프로퍼티로 하위 클래스가 값을 제공할 필요가 있다.
        // 아래 2개의 프로터티는 추상이 아니기 때문에, 기본적으로 열려있지 않다. open 변경자를 통해 열게 할 수 있다.
        val keyframes: Int = 20
        open val frames: Int = 60
        
        abstract fun animate() // 추상 함수로, 구현이 없기에 하위 클래스에서 이 함수를 반드시 오버라이드해야 한다.
        // 아래 2개의 메서드는 추상이 아니기 때문에, 기본적으로 열려있지 않다. open 변경자를 통해 열게 할 수 있다.
        open fun shopAnimating() {}
        fun animateTwice() {}
    }
    ```

**가시성 변경자**: 기본적으로 `public` 선언!

- `가시성 변경자`란 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어하는 것을 의미한다.
- 코틀린은 public, protected, private 변경자를 제공한다.
- 기본적으로 public 변경자를 제공한다.
- 추가적으로 `internal` 변경자를 제공하는데, 이는 같은 모듈 안에서만 볼 수 있다.
- 눈여겨볼 점은 코틀린에서 protected 멤버는 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보인다.
    - 자바에서는 같은 패키지 안에서 protected 멤버에 접근할 수 있지만, 코틀린은 그렇지 않다.

내포된 클래스와 내부 클래스 차이

- 두 클래스의 가장 큰 차이점은 **바깥 클래스의 멤버에 접근할 수 있는지 여부**이다.
    - 내포된 클래스 (Nested Class): 클래스 안에 다른 클래스를 선언하는 걸 말한다.
        - 바깥 클래스의 프로퍼티(`val`, `var`)나 메서드에 접근할 수 없다.
        - 바깥 클래스의 인스턴스 없이도 생성할 수 있다.
    - 내부 클래스 (Inner Class): 바깥 클래스의 멤버에 접근해야 하는 클래스를 만들 때 `inner` 키워드를 붙여 내부 클래스로 만드는 것을 말한다.
        - 바깥 클래스의 private 멤버를 포함한 모든 멤버에 접근할 수 있다.
        - 반드시 바깥 클래스의 인스턴스를 통해서만 생성할 수 있다.
    - 코틀린에서 바깥쪽 클래스의 인스턴스를 가리키는 참조를 표현하는 방법은 자바와 달리, 내부 클래스인 inner 안에서 바깥쪽 클래스인 Outer의 참조를 접근하려면 `this@Outer` 라고 써야 한다.

        ```kotlin
        class Outer {
            inner class Inner {
                fun getOuterReference(): Outer = this@Outer
            }
        }
        ```

**`sealed` 클래스** → 확장이 제한된 클래스 계층을 정의할 때 사용한다.

- 코틀린 컴파일러는 `when`을 사용해서 Expr 타입의 값을 검사할 때는 꼭 디폴트 분기인 else 분기를 덧붙이게 강제한다.
- 이는 안정성있는 코드이지만 항상 디폴트 분기를 추가해주는 것은 편리하지 않다.
- 이런 문제에 대한 해법을 위해 `sealed` 클래스를 활용한다.

```kotlin
sealed class Expr { // 기반 클래스를 sealed으로 봉인한다.
    class Num(val value: Int) : Expr() // 기반 클래스의 모든 하위 크래스를 중첩 클래스로 나열한다.
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
        when (e) {
            is Expr.Num -> e.value // `when` 식이 모든 하위 클래스를 검사하므로 별도의 `else` 분기가 필요 없다.
            is Expr.Sum -> eval(e.left) + eval(e.right)
        }
```

- when 식에서 `sealed` 클래스의 모든 하위 클래스를 처리한다면 디폴트 분기가 필요 없다. sealed 변경자는 **추상 클래스**임을 기억하자.

### 클래스 선언

- 코틀린은 주 생성자와 부 생성자를 구분하고 초기화 블록을 통해 초기화 로직을 추가할 수 있다.
- 먼저 주 생성자와 초기화 블록을 선언하는 방법을 살펴본 다음, 생성자를 여러개 선언하는 방법을 설명한다. 이후 프로퍼티에 대해 자세히 살펴보고자 한다.

**클래스 초기화**: 주 생성자와 초기화 블록

- 주 생성자는 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 말한다.
    - constructor 키워드는 주 생성자 혹은 부 생성자 정의를 시작할 때 사용한다.
    - 주 생성자는 제한적이고 별도의 코드를 포함할 수 없기 때문에 초기화 블록이 필요하다.
- 초기화 블록은 `init` 키워드로 시작한다.
    - 클래스의 객체가 만들어질 때 실행될 초기화 코드가 들어가고 주 생성자와 함께 사용된다.
    - 필요하면 한 클래스 안에 여러 초기화 블록을 선언할 수 있다.

```kotlin
class User constructor(_nickname: String) { // 주 생성자
    val nickname: String

    init { // 초기화 불록
        nickname = _nickname
    }
}
```

- 만약 주 생성자 앞에 별다른 어노테이션이나 가시성 변경자가 없다면 constructor를 생략해도 된다.
- 또한, 프로퍼티를 초기화하는 코드를 프로퍼티 선언에 포함시킬 수 있기 때문에, 초기화 블록에 작성하는 대신 아래와 같이 작성할 수 있다.

```kotlin
class User(_nickname: String) { // 파라미터가 하나뿐인 주 생성자
    val nickname = _nickname // 파라미터를 주 생성자의 파라미터로 초기화한다.
}
```

- 주 생성자의 파라미터를 갖고 프로퍼티를 초기화한다면 아래와 같이 주 생성자 파라미터 앞에 `val` 키워드를 추가하는 방식으로 사용할 수 있다.

```kotlin
class User(val nickname: String) // 주 생성자의 파라미터를 갖고 프로퍼티를 초기화한다는 의미이다.
class User(val nickname: String, val isSubscribed: Bollean = true) // isSubscribed 파라미터에 디폴트 값제공
```

- 클래스의 인스턴스를 만들 때 아래와 같이 new 키워드 없이 생성자를 직접 호출하면 된다.

```kotlin
fun main() {
    val fancy = User("fancy")
    println(fancy.isSubscribed) // true

    val junYong = User("junYong", false)
    println(junYong.isSubscribed) // false
}

class User(val nickname: String, val isSubscribed: Boolean = true)
```

- 기반 클래스를 초기화하려면 기반 클래스 이름 뒤에 괄호를 치고 생성자 인자를 넘긴다.

```kotlin
open class User(val nickname: String)
class TwitterUser(nickname: String) : User(nickname) 
```

**부 생성자**: 상위 클래스를 다른 방식으로 초기화

- 부 생성자는 constructor 키워드로 시작하고 필요에 따라 얼마든지 부 생성자를 선언할 수 있다.
- 아래와 같이 `super()` 키워드를 통해 자신에 대응하는 상위 생성자를 호출할 수 있다. 이를 통해 각 생성자가 위임한 상위 클래스 생성자를 보여준다.

```kotlin
open class Downloader {
    constructor(url: String?)

    constructor(uri: URI?)
}

class MyDownloader : Downloader {
    constructor(url: String?) : super(url) { // 상위 클래스의 생성자를 호출한다.
    }

    constructor(uri: URI?) : super(uri) {
    }
}
```

- 또한 this()를 통해 클래스 자신의 다른 생성자를 호출할 수 있다.

```kotlin
open class Downloader {
    constructor(url: String?)

    constructor(uri: URI?)
}

class MytDownloader : Downloader {
    constructor(url: String?) : this(URI(url)) {} // 같은 클래스의 다른 생성자에게 위임한다.

    constructor(uri: URI?) : super(uri) {
    }
}
```

**접근자의 가시성 변경**

- set 앞에 가시성으로 private 지정하면 클래스 내부에서만 변경 가능하하다
    - 클래스 외부에서는 변경이 불가능하고, 만약 외부에 변경한다면 컴파일 시점에 오류가 발생할 것이다.

### **컴파일러가 생성한 메서드: data 클래스와 클래스 위임**

- (자바와 마찬가지로) 코틀린의 모든 클래스는 `Any`로부터 `toString`, `equals`, `hashCode` 메서드를 상속받는다.
    - 코틀린에서는 `==` 연산자가 두 객체를 비교하는 기본적인 방법으로 내부적으로 equals를 호출한다.
        - equals는 객체의 동등성(내용)을 검사할 때 사용한다.
        - 이와 달리, 참조 비교(동일한 메모리 주소)를 하기 위해서는 `===` 연산자를 사용한다.
    - 또한, equals 를 오버라이드할 때 반드시 hashCode도 함께 오버라이드해야 한다.
        - hashCode 메서드는 모든 프로퍼티 해시 값을 바탕으로 계산한 해시 값을 반환한다.

**data 클래스**

- data 변경자를 클래스 앞에 붙이면 필요한 메서드(toString, equals, hashCode)를 컴파일러가 자동으로 만들어준다. 이렇게 data 변경자가 붙은 클래스를 데이터 클래스라고 부른다.
- 데이터 클래스의 프로퍼티가 꼭 val일 필요는 없지만, 읽기 전용으로 만들어 불변 클래스로 만드는 것을 권장한다.
- 코틀린 컴파일러는 불변 객체로 쉽게 활용할 수 있도록 copy 메서드를 제공해준다.
    - 객체 메모리에서 직접 바꾸는 대신 복사본을 만든다.

**코틀린 data 클래스와 자바 record 비교**

- Java 14 에 `record`가 처음 도입되었고, 개념적으로 record는 여러 불변 값으로 이뤄진 그룹을 다룬다는 점에서 Kotlin의 data 클래스와 비슷하다. 다만 record에는 copy와 같은 다른
  편의 메서드는 없다.
- Java의 record 에는 더 많은 구조적 제약이 있다.
    - 모든 프로퍼티가 private이며, final 이어야 한다.
    - record는 상위 클래스를 확장할 수 없다.
    - 클래스 본문 안에는 다른 프로퍼티를 정의할 수 없다.
- 상호운용성을 위해 코틀린 data 클래스에 `@JvmRecord` 어노테이션을 추가해 record를 선언할 수 있다. 다만, 이런 경우 data 클래스도 record와 같은 똑같은 제약 사항을 지켜야 한다.

**클래스 위임할 때는 `by` 키워드를 사용하자.**

- 코틀린의 by 키워드는 위임 과정에서 발생하는 모든 코드를 컴파일러가 자동으로 대신 만들어준다. → Delegator 패턴!
- 아래와 같이 by 키워드를 사용하면 Manager 클래스는 Cook 인터페이스의 모든 메서드 구현을 chef 객체에게 위임한다.

```kotlin
fun main() {
    val chef = ExpertChef()
    val manager = Manager(chef)

    // Manager는 Cook의 기능을 직접 구현하지 않았지만,
    // by 키워드를 통해 위임받아 호출할 수 있다.
    println(manager.makePizza()) // 페퍼로니 피자
    println(manager.makePasta()) // 크림 파스타

    // Manager 고유의 기능도 호출 가능하다.
    manager.manageStaff()  // 직원들을 관리합니다.
}

interface Cook {
    fun makePizza(): String
    fun makePasta(): String
}

class ExpertChef : Cook {
    override fun makePizza() = "페퍼로니 피자"
    override fun makePasta() = "크림 파스타"
}

// 위임 클래스 - Delegator 패턴!
// 'Cook'의 역할은 수행해야 하지만, 실제 구현은 'chef'에게 맡긴다(by chef).
class Manager(val chef: ExpertChef) : Cook by chef {

    // makePizza(), makePasta()는 chef에게 자동 위임된다.
    // 코드를 작성할 필요가 없습니다.

    fun manageStaff() {
        // 매니저의 고유 기능
        println("직원들을 관리합니다.")
    }
}
```

- 여기서 중요한 점은 **구현 방식에 대한 의존관계가 생기지 않는다는 것**이다.
    - 상속을 하게 된다면, 부모 클래스의 구현이 바뀌면 자식 클래스까지 영향을 받는다.
    - 하지만 by 키워드를 사용함으로써 클래스에 위임하는 것은 코드를 유연하고 느슨하게 연결하게 만들 수 있다.

### object 키워드: 클래스 선언과 인스턴스 생성을 한번에 하기

- `object` 키워드를 사용하는 여러 상황은 다음과 같다.
    - 객체 선언(Declaration)
    - 동반 객체(Companion)
    - 객체 식(Expression)

**1. 객체 선언을 통해 싱글턴을 쉽게 만들자.**

- 코틀린에서는 객체 선언 기능을 통해 싱글턴을 언어에서 기본적으로 지원한다. 객체 선언은 `object` 키워드로 시작한다.
    - (비교) 자바에서 보통 클래스의 생성자를 private로 제한하고 정적인 필드에 그 클래스의 유일한 객체를 저장하는 싱글턴 패턴을 통해 구현한다.
    - 객체 선언 특성상 생성자를 쓸 수 없다.
- 참고: 싱글턴과 의존관계 주입
    - 소규모 소프트웨어에서는 싱글턴이나 객체 선언이 유용하지만 대규모 소프트웨어에서는 적합하지 않다. 그 이유는 객체 생성을 제어할 방법이 없고 생상자 파라미터를 지정할 수 없기 때문이다.

**2. 동반 객체 → companion object**

- 어떤 클래스와 관련 있는 메서드와 팩토리 메서드를 담을 때 사용한다.
- 자바의 static 처럼 클래스의 인스턴스가 아니라, 클래스 자체에 소속된 멤버(변수, 메서드)를 만들고 싶을 때 사용한다.
- 자바와 다른 차이점은 클래스 자체로 접근해야 하고, 클래스의 인스턴스로는 동반 객체의 멤버(변수, 메서드)에 접근할 수 없다는 점이다.
    - 즉, **클래스 이름**을 사용해 그 클래스에 속한 동반 객체의 메서드를 호출할 수 있다.
- 동반 객체는 클래스당 **단 하나만 선언**할 수 있다.

    ```kotlin
    fun main() {
        MyClass.callMe() //Companion object called!
    
        val myObject = MyClass()
    //    myObject.callMe() // Error: Unresolved reference: callMe
    }
    
    class MyClass {
        companion object {
            fun callMe() {
                println("Companion object called!")
            }
        }
    }
    ```

**3. 객체 식 → 익명 내부 클래스를 다른 방식으로 작성**

- 이름이 없는 일회용 객체가 필요할 때 사용한다.
- 보통 인터페이스를 구현하거나 클래스를 상속할 때 사용한다.
- 객체 선언과 차이점은 아래와 같다.
    - 객체 선언은 `object MySingleton` → 이름이 있고 유일한 1개의 인스턴스이고,
    - 객체 식은 `object : MyInterface` → 이름이 없고, 호출될 때마다 새로운 1개의 인스턴스가 생성된다.

### 인라인 클래스

- 인라인 클래스를 사용하면 성능을 희생하지 않고 타입 안정성을 얻을 수 있다.
- 인라인 클래스로 만들기 위해서는 클래스 앞에 `value` 키워드를 사용하고 `@JvmInline` 어노테이션을 붙여야 한다.
    - 이렇게 하면 실행 시점에 해당 클래스의 인스턴스는 감싸진 프로퍼티로 대체된다. 즉, 클래스의 데이터가 사용되는 시점에 인라인된다. → 정확히 말하면 항상 인라인 되는 것은 아니고, 때로는 래퍼 객체가
      생성되어 박싱된다.

```kotlin
@JvmInline
value class Password(val value: String)
```

- 인라인 클래스는 다음과 같이 제약 조건을 가진다.
    - 반드시 하나의 `val` 프로퍼티만 가져야 하며, 해당 프로퍼티는 주 생성자에서 초기화돼야 한다.
    - 인라인 클래스는 다른 클래스를 상속할 수 없고, 반대 역시 상속할 수 없다. (단, 인터페이스는 가능하다)
- 보통 기본 타입 값의 의미를 명확하게 하고 싶을 때 인라인 클래스를 사용한다.
    - 숫자 타입의 값으로 측정한 값의 단위를 표현할 경우
    - 다른 여러 문자열의 서로 다른 의미를 구분하고 싶은 경우

> 4장과 관련된 도움이 된 글 (실무)

* [Kotlin 객체 생성의 안전성과 유효성 강화하기](https://cheese10yun.github.io/kotlin-pattern-2/) - companion object, value class

* [Kotlin 자주 사용하는 패턴 정리](https://cheese10yun.github.io/kotlin-pattern/) - copy(), by()


---