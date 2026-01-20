# Kotlin in Action 2/E

> 이 글은 [Kotlin IN Action 2/E](https://product.kyobobook.co.kr/detail/S000215768644) 책을 읽고 개인 생각과 학습 테스트를 포함하여 정리한 내용입니다.

> 예상 독자

* 자바 경험이 있는 개발자

* 서버 개발자 or 안드로이드 개발자와 같이 JVM에서 실행될 프로젝트를 구축하는 모든 개발자

## 목차

- [9장. 연산자 오버로딩과 다른 관례](#9장-연산자-오버로딩과-다른-관례)
    - [산술 연산자 오버로딩](#산술-연산자-오버로딩)
    - [비교 연산자 관례](#비교-연산자-관례)
    - [컬렉션과 범위 관련 관례](#컬렉션과-범위-관련-관례)
    - [구조 분해 선언](#구조-분해-선언-destructuring-declaration)
- [10장. 고차 함수](#10장-고차-함수)
    - [고차 함수 개념 및 만드는 방법](#고차-함수-개념-및-만드는-방법)
    - [고차 함수의 성능 개선: Inline 함수](#고차-함수의-성능-개선-inline-함수)
    - [람다 안에서의 흐름 제어 (return)](#람다-안에서의-흐름-제어-return)
- [11장. 제네릭스](#11장-제네릭스)
    - [제네릭 타입 파라미터와 제약](#제네릭-타입-파라미터와-제약)
    - [실행 시점 제네릭스: 소거와 실체화](#실행-시점-제네릭스-소거와-실체화) 
    - [변성 (Variance): 기저 타입과 타입 인자의 관계](#변성-variance-기저-타입과-타입-인자의-관계)
    - [사용 지점 변성 (Use-site Variance)](#사용-지점-변성-use-site-variance)


---

# 9장. 연산자 오버로딩과 다른 관례

`관례`란 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법이다.
코틀린은 이를 통해 자바보다 더 간결하고 직관적인 구문을 제공한다.

## 산술 연산자 오버로딩

연산자를 오버로딩하는 함수 앞에는 반드시 `operator` 키워드가 있어야 한다.
이는 해당 함수가 코틀린의 관례를 따른다는 것을 명시한다.

- 오버로딩과 오버라이딩의 차이점
    - `오버로딩`은 같은 이름의 함수를 매개변수 타입이나 개수를 다르게 하여 여러 개 정의하는 것이고,
    - 반면에 `오버라이딩`은 상위 클래스에 정의된 함수를 하위 클래스에서 재정의하여 사용하는 것이다.

- 참고 사항
    - 연산자를 정의할 때 두 피연산자가 같은 타입일 필요는 없으며,
    - 코틀린은 자동으로 교환 법칙(a + b == b + a)을 지원하지 않으므로 필요 시 직접 정의해야 한다.

- 복합 대입 연산자 오버로딩
    - 코틀린은 + 연산자뿐 아니라 그와 관련 있는 연산자인 +=도 자동으로 지원한다. 이를 `복합 대입 연산자`라 부른다.

- 단항 연산자 오버로딩
    - 틀린은 -a와 같이 하나의 값에 작용하는 단항 연산자도 제공한다.

## 비교 연산자 관례

코틀린에서는 기본 타입 값뿐만 아니라 모든 객체에 대해 비교 연산을 수행할 수 있다.

- 동등성 연산자 (equals)
    - `==`는 내부적으로 `equals`를 호출한다.
    - 자바와 달리 코틀린에서는 비교 연산자를 직접 사용할 수 있어, equals나 compareTo를 사용한 코드보다 더 간결하며 이해하기 쉽다.
    - equals를 구현할 때는 식별자 비교 연산자(===)를 사용해 자신과의 비교를 최적화하는 경우가 많다. `===`를 오버로딩할 수는 없다는 사실을 기억하자.
    - `Any`에서 상속받은 equals가 확장 함수보다 우선순위가 높기 때문에 equals를 확장 함수로 정의할 수 없다는 사실에 유의하자.

- 순서 연산자 (compareTo)
    - `<`, `>`, `<=`, `>=` 연산자는 `compareTo` 호출로 컴파일된다.
    - 정렬, 최댓값, 최솟값 등 값을 비교해야 하는 알고리즘에 사용할 클래스는 Comparable 인터페이스를 구현해야 한다.
    - `compareTo` 메서드는 객체 간의 크기를 비교해 정수로 나타내준다.
    - equals와 마찬가지로 compareTo에도 operator 변경자가 붙어있으므로, 하위 클래스에서 오버라이드할 때 함수 앞에 operator를 붙일 필요가 없다.
    - 코틀린 표준 라이브러리의 `compareValuesBy` 함수를 사용하면 compareTo를 쉽고 간결하게 정의할 수 있다.

## 컬렉션과 범위 관련 관례

- `in` 연산자: 객체가 컬렉션이나 범위에 포함되는지 검사한다. (`contains` 함수와 연결)

- `..<` 연산자 (Open Range): 1.9 버전부터 정식 도입된 열린 범위 연산자다.
    - 상계(마지막 값)를 포함하지 않는 범위를 만들 때 사용한다.
    - `rangeTo` 연산자는 닫힌 범위를, `rangeUntil` 연산자는 열린 범위(상계 미포함)를 만든다.

> 정수 범위 비교: .. vs ..<

리스트의 인덱스나 특정 경계값 직전까지의 범위를 다룰 때 유용하다.

```kotlin
fun main() {
    // 1. rangeTo (..): 닫힌 범위 (10 포함)
    val closedRange = 1..10
    println("Closed: " + closedRange.joinToString())
    // result: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10

    // 2. rangeUntil (..<): 열린 범위 (10 미포함)
    val openRange = 1..<10
    println("Open: " + openRange.joinToString())
    // result: 1, 2, 3, 4, 5, 6, 7, 8, 9
}
```

> 리스트 인덱스 순회

기존에는 리스트의 마지막 인덱스까지 순회하기 위해 0`..list.size - 1`를 사용했지만,
`..<`를 사용하면 더 직관적으로 표현할 수 있다.

```kotlin
val features = listOf("Operator", "Convention", "Destructuring")

// 기존 방식 (size - 1 필요)
for (i in 0..features.size - 1) { /* ... */
}

// 열린 범위 방식 (size 그대로 사용 가능, 더 직관적임)
for (i in 0..<features.size) {
    println("Feature $i: ${features[i]}")
}
```

> 내부 동작: rangeTo와 rangeUntil

연산자는 내부적으로 특정 함수 호출로 변환되므로, 필요에 따라 직접 함수를 호출할 수도 있다.

| 연산자   | 대응하는 함수(관례)  | 특징                 |
|-------|--------------|--------------------|
| `..`  | `rangeTo`    | 마지막 값(상계)을 포함함     |
| `..<` | `rangeUntil` | 마지막 값(상계)을 포함하지 않음 |

```kotlin
val start = 0
val end = 5

// 연산자 사용 방식
val r1 = start..end        // 내부적으로 start.rangeTo(end) 호출
val r2 = start..<end       // 내부적으로 start.rangeUntil(end) 호출

// 직접 함수 호출 방식 (결과는 위와 동일)
val r3 = start.rangeTo(end)
val r4 = start.rangeUntil(end)

println(r1 == r3) // true
println(r2 == r4) // true
```

## 구조 분해 선언 (Destructuring Declaration)

구조 분해를 사용하면 복합적인 값을 분해해서 별도의 여러 지역 변수를 한꺼번에 초기화할 수 있다.
이는 내부적으로 `componentN` 함수를 호출하는 관례를 이용한다.

```kotlin
data class NameComponents(val name: String, val extension: String)

fun splitFilename(fullName: String): NameComponents {
    val result = fullName.split('.', limit = 2)
    return NameComponents(result[0], result[1])
}

fun main() {
    // 구조 분해 선언을 통해 name과 ext를 한 번에 초기화
    val (name, ext) = splitFilename("kotlin.kt")
    println("Name: $name, Extension: $ext")
}
```

- 효과와 활용
    - 구조 분해 선언은 함수에서 **여러 값을 반환**할 때 매우 유용하다.
    - `_` 문자를 사용해 필요하지 않은 구조 분해 값은 무시할 수 있다.

- 한계와 단점
    - 위치 기반: 구조 분해는 프로퍼티 이름이 아니라 **인자의 위치**에 따라 결정된다.
      따라서 데이터 클래스의 프로퍼티 순서를 바꾸면 구조 분해를 사용하는 곳에서 예상치 못한 오류가 발생할 수 있다.
    - 복잡성: 너무 복잡한 엔티티에 대해 구조 분해를 사용하는 것은 가독성을 해칠 수 있으므로 가능한 한 피해야 한다.

- 실무 사례 - TossBank SLASH 24 "[지속 가능한 마이그레이션 전략](https://www.youtube.com/watch?v=LwH9h8dG3PQ)"
    - 영상에서 언급된 분할 정복과 컴포지트 패턴은 복잡한 도메인 로직(할부 금액 계산 등)을 작은 단위로 쪼개어 최적화하는 전략을 보여주고 있다.
    - **할부금액 계산 모듈**처럼 복잡한 도메인 로직을 분할 정복 방식으로 최적화할 때, 계산 결과로 나오는 여러 값(원금, 이자 등)을 구조 분해 선언으로 처리하면 로직의 흐름을 훨씬 명확하게 파악할 수
      있다.

# 10장. 고차 함수

## 고차 함수 개념 및 만드는 방법

`고차 함수`란 다른 함수를 **인자로 받거나** 또는 **함수를 반환하는** 함수이다.
코틀린에서는 함수를 변수처럼 다룰 수 있기 때문에(일급 객체), 람다나 함수 참조를 통해 함수를 값으로 주고받을 수 있다.

### 함수 타입 정의

고차 함수를 정의하려면 먼저 전달받을 "함수의 규격"인 함수 타입을 선언해야 한다.

- 기본 문법: `(파라미터 타입) -> 반환 타입`
    - (Int, String) -> Unit: Int와 String을 받아 아무것도 반환하지 않는 함수.
    - () -> Unit: 인자가 없고 반환값도 없는 함수.

- Nullable 함수 타입: 함수 타입 전체가 null이 될 수 있다면 괄호로 감싸고 물음표를 붙인다.
    - ((Int, Int) -> Int)? = null

### 고차 함수 구현 및 호출

고차 함수는 공통적인 실행 흐름(틀)을 정의하고, 변화하는 부분(로직)은 외부에서 주입받는 전략 패턴과 유사하다.

```kotlin
// predicate라는 이름으로 (Char) -> Boolean 타입의 함수를 받는다.
fun String.filter(predicate: (Char) -> Boolean): String {
    val sb = StringBuilder()
    for (index in 0..<length) {
        val element = get(index)
        // 인자로 받은 'predicate' 함수를 호출하여 조건을 검사한다.
        if (predicate(element)) {
            sb.append(element)
        }
    }
    return sb.toString()
}

// 사용 예시: "대문자만 남겨줘"라는 로직을 람다로 전달
println("Kotlin123".filter { it in 'A'..'Z' }) // 결과: K
```

위와 같이 고차함수는 "틀"만 만들어두고, 구체적인 로직(함수)은 외부에서 주입받는 것이다.

- filter는 "문자열을 돌면서 담는다"는 틀만 갖고 있고, "어떤 문자를 담을지"는 외부에서 predicate라는 함수로 결정해서 넘겨주는 원리다.

### 함수 타입 파라미터의 기본값과 널 처리

자바에서는 전략 패턴 등을 위해 인터페이스를 구현해야 했지만, 코틀린에서는 함수 타입의 기본값을 지정해 코드를 훨씬 단순화할 수 있다.

함수 타입에 기본값 지정하기

- joinToString 예제처럼 transform 로직을 기본적으로 제공하면서, 필요한 경우에만 사용자가 직접 정의하도록 할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        // transform 함수를 호출하여 요소를 문자열로 변환 
        result.append(transform(element))
    }

    result.append(postfix)
    return result.toString()
}

fun main() {
    val letters = listOf("Alpha", "Beta")
    println(letters.joinToString()) // Alpha, Beta
    println(letters.joinToString { it.lowercase() }) // alpha, beta
    println(
        letters.joinToString(
            separator = "! ", postfix = "! ",
            transform = { it.uppercase() })
    ) // ALPHA! BETA! 
}
```

### 함수를 함수에서 반환하기

함수를 반환하는 고차 함수는 "조건에 따라 로직 자체를 선택해서 내보낼 때" 매우 유용하다.

- 실무적인 예시: 배송비 계산, 세금 계산 등 프로그램의 상태에 따라 계산식(로직) 자체가 바뀌어야 하는 경우.

```kotlin
// NOTE 10.7
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

// Delivery 타입에 따라 '어떤 함수'를 사용할지 결정해서 반환한다.
fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount } // 특송 계산 로직 반환
    }
    return { order -> 1.2 * order.itemCount } // 일반 계산 로직 반환
}

fun main() {
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    println("Shipping costs ${calculator(Order(3))}") // Shipping costs 12.3
    // 만약 Delivery.STANDARD 이면 Shipping costs 3.5999999999999996
}
```

### 람다를 활용한 중복 제거와 코드 재사용성

아주 복잡한 구조를 만들어야만 피할 수 있는 코드 중복도 람다를 활용하면 간결하게 제거할 수 있다.
람다를 사용하면 단순히 **값** 뿐만 아니라, 데이터를 다루는 **행동(로직) 자체**를 추출하여 재사용할 수 있기 때문이다.

단계적 개선: 처음에는 특정 OS의 평균 방문 시간만 구하는 함수로 시작하지만, 요구사항이 늘어날수록 함수는 계속 늘어날 위험이 있다.

- 1단계 (특정 값 기반): averageDurationFor(os: OS) — 오직 OS로만 필터링 가능.
- 2단계 (복잡한 조건): 모바일 사용자(iOS, Android)나 특정 경로('/') 방문자의 평균을 구해야 한다면? 함수 내부 로직이 중복되기 시작한다.
- 3단계 (고차 함수 기반): 필터링 조건 자체를 람다로 받아서 처리한다.

아래와 같이 조건을 함수 타입 파라미터로 받으면,
어떤 복잡한 필터링 요구사항이 와도 **단 하나의 함수**로 대응할 수 있다.

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID),
)

// (고차 함수 적용 전)
//fun List<SiteVisit>.averageDurationFor(os: OS) =
//    filter { it.os == os }
//        .map(SiteVisit::duration).average()


// 핵심: '어떤 조건으로 필터링할 것인가'라는 행동(람다)을 인자로 받는다.
// (SiteVisit) -> Boolean 타입의 함수를 인자로 받는 고차 함수
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate) // 주입받은 외부 로직(람다)으로 필터링을 수행함
        .map(SiteVisit::duration)
        .average()

fun main() {
    // 1. 특정 OS(WINDOWS) 필터링 로직을 주입
    println(log.averageDurationFor { it.os == OS.WINDOWS }) // 23.0

    // 2. 모바일 OS(IOS, ANDROID) 필터링 로직을 주입
    val averageMobileDuration = log.averageDurationFor {
        it.os in setOf(OS.IOS, OS.ANDROID)
    }
    println(averageMobileDuration) // 12.15

    // 3. 특정 경로("/") 필터링 로직을 주입 (함수 하나로 모든 대응 가능!)
    println(log.averageDurationFor { it.path == "/" }) // 24.1
}
```

## 고차 함수의 성능 개선: inline 함수

람다를 활용하면 코드가 간결해지지만, 내부적으로는 람다마다 익명 클래스 객체가 생성되는 오버헤드가 발생한다.
코틀린은 이를 해결하기 위해 `inline` 변경자를 제공한다.

> 호출 동작 과정 비교

| 구분    | 일반적인 고차 함수 호출                     | 인라인 함수 호출 (inline)             |
|-------|-----------------------------------|--------------------------------|
| 객체 생성 | 람다 실행을 위해 **익명 클래스 객체를 매번 생성**한다. | 객체를 생성하지 않고 코드를 직접 삽입한다.       |
| 바이트코드 | 함수 호출 명령어(`invoke`)가 포함된다.        | 함수 본문이 호출 지점의 바이트코드로 치환된다.     |
| 실행 방식 | 새로운 스택 프레임을 생성하고 제어권을 넘긴다.        | 원래 코드의 일부인 것처럼 **순차적으로 실행**된다. |

### 인라인이 작동하는 방식

람다 본문과 함께 인라이닝 (Full Inlining)

- 인라인 함수를 호출하면서 `람다 식({ ... })`을 직접 넘기면, 함수 본문과 전달된 람다의 본문이 **모두 호출 지점에 인라이닝**된다.
    - 특징: 람다 본문이 인라인 함수의 일부로 간주되어 익명 클래스로 감싸지 않는다.
    - 결과: 여러 곳에서 다른 람다로 호출하면, 각 호출 지점마다 서로 다른 코드가 따로따로 인라이닝된다.

함수 타입 변수 전달 시 (Partial Inlining)

- 아래 예시 코드는 인라인 함수를 호출할 때 람다 본문 대신 **함수 타입의 변수**를 넘기는 경우이다.

```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) { // body는 변수
        synchronized(lock, body) // synchronized는 inline 함수
    }
}
```

- 한계: 컴파일 시점에 변수(body)에 어떤 로직이 들어올지 알 수 없다.
- 결과: synchronized 함수의 본문은 호출 지점에 복사(인라이닝)되지만,
    - **람다(body)는 일반적인 경우와 마찬가지로 호출된다**
    - 즉, 객체 생성 방지 효과를 100% 누릴 수 없다.

### 인라인 함수의 제약

`inline`은 모든 경우에 사용할 수 있는 마법이 아니다.
함수 본문에서 파라미터로 받은 람다를 어떻게 사용하는지에 따라 인라이닝이 불가능한 경우가 있다.

- 람다를 다른 변수에 저장하거나 나중에 사용해야 하는 경우
    - 인라인 함수의 본문에서 파라미터로 받은 람다를 호출하는 대신, 다른 변수에 저장하거나 다른 객체의 프로퍼티로 넘기는 경우에는 인라이닝을 할 수 없다.
    - 이유: 람다 코드가 호출 지점에 펼쳐져야 하는데, 변수에 저장하려면 람다를 표현하는 **객체**가 어딘가에는 존재해야 하기 때문이다.
- noinline 변경자
    - 둘 이상의 람다를 인자로 받는 함수에서 특정 람다만 인라이닝을 금지하고 싶을 때 사용한다.
    - 인라인 함수가 전달받은 람다를 인라이닝할 수 없는 방식으로 다룰 때 컴파일러는 `Illegal usage of inline-parameter` 오류를 보고하며,
    - 이때 `noinline`을 붙여 해결할 수 있다.

### 인라인을 언제 선언해야 할까

inline 키워드를 사용한다고 해서 항상 성능이 좋아지는 것은 아니다. 그렇기에 신중히 결정해야 한다.

- 일반 함수 호출은 이미 충분히 빠르다.
    - JVM은 이미 강력한 인라이닝을 지원한다. 코드 실행을 분석하여 가장 이익이 되는 방향으로 호출을 인라이닝하며, 이는 JIT 컴파일 과정에서 일어난다.
    - JVM 방식은 각 함수 구현이 바이트코드에 딱 한 번만 있으면 되지만, 코틀린의 inline은 모든 호출 지점에 코드를 복사하므로 **코드 중복**이 발생한다.
- 람다를 인자로 받는 함수를 인라이닝할 때 이득이 큰 이유
    - 함수 호출 비용뿐만 아니라 람다를 표현하는 클래스와 인스턴스 객체를 만들 필요가 없어진다.
    - 현재의 JVM은 함수 호출과 람다를 코틀린의 inline만큼 똑똑하게 인라이닝해주지 못하는 경우가 많다.
- 주의사항
    - inline 변경자를 붙일 때는 **코드의 크기**에 주의해야 한다. 함수 본문이 크면 모든 호출 지점에 코드가 복사되어 전체 바이트코드 크기가 아주 커질 수 있다.
    - 코틀린 표준 라이브러리의 인라인 함수들을 보면 대부분 크기가 아주 작다는 것을 알 수 있다.

한마디로 정리하면, 인라인은 무분별한 최적화 도구가 아니라, **고차 함수의 오버헤드를 줄이기 위한 정교한 도구**다.

### 자원 관리를 위한 인라인 함수 활용 (withLock, use, useLines)

코틀린은 자원(파일, 락, DB 트랜잭션 등)을 획득하고 해제하는 반복적인 try/finally 패턴을 인라인 함수로 캡슐화하여 제공한다.

- `withLock`: Lock 인터페이스의 확장 함수로, 락을 자동으로 획득하고 작업이 끝나면 안전하게 해제해준다.
- `use`: Closeable 자원을 처리할 때 사용하며, 람다가 정상 종료되거나 예외가 발생하더라도 자원이 확실히 닫히도록 보장한다. (자바의 try-with-resources 문과 같은 역할을 수행한다.)
- `useLines`: 파일의 각 줄을 시퀀스로 접근할 수 있게 해주어 대용량 파일도 효율적이고 코틀린답게 처리할 수 있다.

## 람다 안에서의 흐름 제어 (return)

람다 안에서 `return`을 사용하면 호출한 외부 함수까지 종료될 수 있다. 이를 제어하는 방법을 이해해야 한다.

### 비로컬 반환

람다 안에서 return을 사용하면 람다를 호출한 함수가 종료된다. 이는 **인라인 함수**에서만 가능하다.

- 인라인 함수는 컴파일 시 함수 본문이 호출 지점에 복사되므로,
- 람다 안의 return이 컴파일되면 결국 자신을 둘러싼 함수를 종료시키는 return이 되기 때문이다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return // lookForAlice 함수 자체를 종료시킴 (비로컬 반환)
        }
    }
    println("Alice is not found") // Alice를 찾으면 이 문장은 실행되지 않음
}
```

### 로컬 반환

람다 안에서만 실행을 멈추고 싶다면, **레이블**을 사용해야 한다.

- 사용법: 람다식 앞에 `이름@`를 붙이고, `return@이름`으로 반환한다.
- 암시적 레이블: 람다를 인자로 받는 함수의 이름(예: forEach)을 레이블로 사용할 수도 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    // 명시적 레이블 사용
    people.forEach label@{ // 레이블 정의
        if (it.name == "Alice") return@label // 정의한 레이블로 로컬 반환
    }

    // 암시적 레이블 사용
    people.forEach {
        if (it.name == "Alice") return@forEach
    }
}
```

### 익명 함수: 로컬 반환의 대안

레이블은 코드가 복잡해지면 가독성이 떨어질 수 있다.
이때는 익명 함수를 사용하면 일반 함수처럼 `return`이 해당 익명 함수만 종료시킨다.

- 익명 함수는 일반 함수와 비슷해 보이지만 이름이 없으며, `return`은 가장 가까운 `fun` 키워드로 정의된 함수를 종료시킨다는 규칙을 따른다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun(person) { // 익명 함수 사용
        if (person.name == "Alice") return // 가장 가까운 fun인 익명 함수만 종료 (로컬 반환)
        println("${person.name} is not Alice")
    })
}
```

로컬 반환을 만드는 두가지 방법은 아래 표와 같다.

| 방법       | 문법 (Syntax)              | 특징                                                          |
|----------|--------------------------|-------------------------------------------------------------|
| 람다 + 레이블 | `return@forEach`         | 람다의 현재 실행(이번 회차)만 끝내고 다음 요소로 넘어간다. (자바의 `continue`와 유사)     |
| 익명 함수    | `fun(person) { return }` | 람다 대신 함수를 통째로 넘기는 형태다. `return`이 익명 함수만 종료시키므로 루프가 계속 유지된다. |

# 11장. 제네릭스

자바와 마찬가지로 코틀린의 제네릭스는 타입 안정성을 보장하기 위해 도입되었지만,
하위 호환성을 위해 로(raw) 타입을 허용하는 자바와 달리 코틀린은 처음부터 제네릭을 강제한다는 점이 다르다.

## 제네릭 타입 파라미터와 제약

### Raw 타입의 부재와 타입 추론

코틀린은 자바 1.5 이전 버전과의 호환성 문제에서 자유롭기 때문에, 제네릭 타입의 인자를 항상 명시하거나 컴파일러가 추론할 수 있어야 한다.

- 타입 추론: `listOf("A", "B")`와 같이 값을 통해 컴파일러가 `List<String>`임을 추론한다.
- 명시적 선언: 빈 리스트를 만들 때는 `val readers: MutableList<String> = mutableListOf()` 처럼 타입을 명시해야 한다.

### 타입 파라미터 제약

제네릭 함수나 클래스에서 사용할 수 있는 타입의 범위를 제한할 수 있다.

- 상계(Upper Bound) 지정: 자바의 `extends`와 유사하게 `:`을 사용한다.

(`상계`라는 말 대신 "상한선(제한선)" 혹은 "이 부모의 자식들만 들어와" 라고 생각하면 쉽다.)

```kotlin
// Number의 하위 타입만 허용 - "T는 무조건 Number의 자식이어야 해!" (Int, Double, Float 등 통과)
fun <T : Number> oneHalf(value: T): Double {
    return value.toDouble() / 2.0
}
```

- 다중 제약 (where): 타입 파라미터가 둘 이상의 인터페이스나 클래스를 구현해야 할 경우 `where` 절을 사용한다.

```kotlin
// "T는 CharSequence(읽기) 기능도 있어야 하고, 동시에 Appendable(쓰기) 기능도 있어야 해!"
fun <T> ensureTrailingPeriod(seq: T) where T : CharSequence, T : Appendable {
    // CharSequence 덕분에 endsWith 사용 가능
    if (!seq.endsWith('.')) {
        // Appendable 덕분에 append 사용 가능
        seq.append('.')
    }
}
```

- Null 가능성 제어: 기본적으로 제네릭 <T>는 `<T: Any?>`와 같아서 **null을 허용**합니다. null을 허용하지 않으려면 `<T : Any>`로 상계를 지정해야 한다.

## 실행 시점 제네릭스: 소거와 실체화

JVM의 제네릭스는 **타입 소거** 방식을 사용합니다. 즉, 실행 시점에는 List<String>이나 List<Int>나 모두 단순한 `List` 객체로 취급된다.

### 타입 소거의 한계와 스타 프로젝션

실행 시점에 타입 인자가 지워지기 때문에(List<String>인지 List<Int>인지 모름), is 검사를 통해 구체적인 제네릭 타입을 확인할 수 없다.

- 스타 프로젝션(*): "어떤 정해진 타입이 있긴 한데, 정확히 뭔지는 모른다"는 것을 표현한다. 자바의 List<?>와 유사하다.
  - if (value is List<*>): 리스트인지는 확인할 수 있지만, 원소 타입은 모른다.
  - as? List<String>: 캐스팅은 가능하지만, 컴파일러가 Unchecked cast 경고를 띄운다. 실행 시점에는 성공하더라도 나중에 잘못된 타입의 원소를 꺼낼 때 예외가 발생할 수 있다.
  - 쓰기는 불가능하고 읽기만 가능하다.

```kotlin
fun checkList(list: MutableList<*>) {
  // 1. 읽기: 가능 (뭐가 들어있든 최소한 Any?인 건 확실하니까)
  val item = list.get(0)

  // 2. 쓰기: 불가능 (이 리스트가 Int 리스트면 String을 넣으면 안 되니까)
  // list.add("Hello") // 컴파일 에러!
}
```

### 실체화된 타입 파라미터 (Reified)

코틀린은 `inline` 함수의 특성을 이용해 타입 정보를 실행 시점까지 유지할 수 있는 기법을 제공한다.
이를 실체화된(Reified) 타입 파라미터라고 한다.
- 작동 원리: 함수를 `inline`으로 선언하면 컴파일러가 함수 본문을 호출 지점에 그대로 갖다 붙인다. 이때 제네릭 타입 파라미터 대신 구체적인 타입을 바이트코드에 심어버리기 때문에 실행 시점에도 타입 정보를 알 수 있다.
- 활용: 타입 검사(`is`), 캐스팅(`as`), 리플렉션 객체(`::class`) 사용이 가능하다.

```kotlin
// inline과 reified 키워드 필수
inline fun <reified T> isA(value: Any) = value is T

fun main() {
    println(isA<String>("abc")) // true
    println(isA<Int>("abc"))    // false
}
```

아래와 같이 ObjectMapper 쓸 때 Reified 사용하면 타입을 인자로 넘기는 척만 해도 된다.

```kotlin
inline fun <reified T> parseJson(json: String): T {
    // T::class.java에 접근 가능! 왜? 컴파일러가 T를 User로 바꿔치기 해주니까.
    return objectMapper.readValue(json, T::class.java) 
}
val user = parseJson<User>(json) // 깔끔함
```

## 변성 (Variance): 기저 타입과 타입 인자의 관계

변성은 기저 타입이 같고 타입 인자가 다른 여러 타입(예: List<String> vs List<Any>)이 서로 어떤 하위 타입 관계를 갖는지 설명하는 개념이다.

쉽게 말해 "구성 요소의 상속 관계가, 껍데기(제네릭)의 상속 관계로 이어지는가?"를 정의하는 규칙이다.

### 하위 클래스 (Subclass) vs 하위 타입 (Subtype)

- 하위 클래스: 상속 관계 (Int는 Number의 하위 클래스).
- 하위 타입: 어떤 타입 A의 값이 필요한 모든 곳에 타입 B의 값을 넣어도 문제가 없다면, B는 A의 하위 타입이다.
  - 중요: 널이 될 수 없는 타입(Int)은 널이 될 수 있는 타입(Int?)의 하위 타입이다. (`Int` 값을 `Int?` 변수에 넣을 수 있음)

### 공변성 (Covariance): out - 읽기 전용

"생산자(Producer)는 `out`이다." "자식은 부모가 될 수 있다."

- 개념: A가 B의 하위 타입일 때, Producer<A>도 Producer<B>의 하위 타입이 유지된다. -> 자식 타입 상자(List<String>)를 부모 타입 상자(List<Any>)로 취급해도 되나? -> YES
- 키워드: `out T`
- 조건: T는 오직 **반환 타입(아웃 위치)** 에만 쓰여야 한다.
- 예시: `List<out T>` (읽기 전용 리스트)

```kotlin
// String은 Any의 하위 타입이므로, List<String>은 List<Any>의 하위 타입이다.
val strings: List<String> = listOf("a", "b")
val objects: List<Any> = strings // 가능 (공변성)
```

### 반공변성 (Contravariance): in - 쓰기 전용

"소비자(Consumer)는 `in`이다." "부모는 자식을 대신할 수 있다."

- 개념: A가 B의 하위 타입일 때, 관계가 역전되어 Consumer<B>가 Consumer<A>의 하위 타입이 된다. -> 부모 타입 처리기(Comparator<Any>)를 자식 타입 처리기(Comparator<String>)로 써도 되나? -> YES
- 키워드: `in T`
- 조건: T는 오직 파라미터 타입에만 쓰여야 한다.
- 예시: `Comparator<in T>`

```kotlin
// Any를 비교할 수 있는 놈은 String도 비교할 수 있다.
val anyComparator = Comparator<Any> { a, b ->
  a.hashCode() - b.hashCode()
}
val strComparator: Comparator<String> = anyComparator // 가능 (반공변성)

fun main() {
  val names = listOf("Charlie", "Alice", "Bob")
  // sortedWith는 Comparator<String>을 원하지만, Comparator<Any>를 넣어도 작동함
  println(names.sortedWith(anyComparator))
}
```

> 변성 정리표

| **변성** | **키워드** | **역할**           | **타입 관계**                          | **예시**                   |
|--------|---------|------------------|------------------------------------|--------------------------|
| 공변성    | `out`   | 생산자 (Read-only)  | 유지됨 (`List<String>` ⊂ `List<Any>`) | `List`, `Iterator`       |
| 반공변성   | `in`    | 소비자 (Write-only) | 뒤집힘 (`Comp<Any>` ⊂ `Comp<String>`) | `Comparator`, `Consumer` |
| 무공변성   | 없음      | 읽기/쓰기 모두         | 관계없음 (서로 다름)                       | `MutableList`            |


## 사용 지점 변성 (Use-site Variance)

클래스 선언 시점에 변성을 지정하지 못했더라도(예: MutableList), 사용하는 시점에 변성을 제한할 수 있다. 
이를 타입 프로젝션(Type Projection)이라 한다.

```kotlin
// source에서는 읽기만 할 것이므로 out 프로젝션 적용
fun <T> copyData(source: MutableList<out T>, dest: MutableList<T>) {
    // source.add() 같은 쓰기 메서드는 호출 불가
    for (item in source) {
        dest.add(item)
    }
}

val strings = mutableListOf("A", "B")
val objects = mutableListOf<Any>()

// 원래라면 MutableList<String>을 MutableList<Any> 파라미터에 못 넣음.
// 하지만 'out Any'로 선언했기 때문에 "String 리스트도 읽기 전용으로는 받을게"가 성립됨.
copy(strings, objects)
```

이 기능은 자바의 List<? extends T>와 완전히 동일한 역할을 한다.