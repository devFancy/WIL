# Kotlin in Action 2/E, 5-8장

> 이 글은 [Kotlin IN Action 2/E](https://product.kyobobook.co.kr/detail/S000215768644) 책을 읽고 개인 생각과 학습 테스트를 포함하여 정리한 내용입니다.

> 예상 독자

* 자바 경험이 있는 개발자

* 서버 개발자 or 안드로이드 개발자와 같이 JVM에서 실행될 프로젝트를 구축하는 모든 개발자

## 목차

- [5장. 람다를 사용한 프로그래밍](#5장-람다를-사용한-프로그래밍)
  - [람다식과 멤버 참조](#람다식과-멤버-참조)
  - [함수형 인터페이스 - 단일 추상 메서드](#함수형-인터페이스---단일-추상-메서드)
  - [수신 객체 지정 람다 - with, apply, also](#수신-객체-지정-람다---with-apply-also)
- [6장. 컬렉션과 시퀀스](#6장-컬렉션과-시퀀스)
  - [컬렉션에 대한 함수형 API](#컬렉션에-대한-함수형-api)
  - [시퀀스 - 게으르게 계산한다](#시퀀스---게으르게-계산한다)
  - [배운 점](#배운-점)
- [7장. 널이 될 수 있는 값](#7장-널이-될-수-있는-값)
  - [null이 될 수 있는 타입](#null이-될-수-있는-타입)
  - [null을 다루는 방법](#null을-다루는-방법)
  - [예외 없이 안전하게 타입 캐스트하기: as?](#예외-없이-안전하게-타입-캐스트하기-as)
  - [널 아님 단언 선언하기: !!](#널-아님-단언-선언하기-)
  - [let 함수](#let-함수)
  - [나중에 초기화하기: lateinit](#나중에-초기화하기-lateinit)
  - [타입 파라미터의 널 가능성](#타입-파라미터의-널-가능성)
  - [널 가능성과 자바](#널-가능성과-자바)



---

# 5장. 람다를 사용한 프로그래밍

람다식 또는 람다는 기본적으로 **다른 함수에 넘길 수 있는 작은 코드 조각**을 의미한다.

- 람다식을 사용하면 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있어서, 코틀린 표준 라이브러리는 람다를 많이 사용한다.

5장에서는 람다가 무엇이고 람다 함수의 전형적인 사용 패턴을 알아보고,

코틀린에서는 어떻게 사용하는지, 멤버 참조와 람다의 관계를 살펴보고자 한다.

마지막으로 수신 객체 지정 람다를 살펴본다.

## 람다식과 멤버 참조

어떠한 동작을 간단히 기술할 수 있는 방법으로 람다를 활용할 수 있다.

아래와 같이 코틀린의 표준 라이브러리 함수를 사용하면 간단히 표현할 수 있다.

> 가장 나이가 많은 사람을 반환할 경우 -> maxByOrNull 사용 !

```kotlin
fun main() {
    val people = listOf(Person("fancy", 29), Person("junyong", 30))
    // NOTE: 가장 큰 값을 반환
    println(people.maxByOrNull { it.age }) // result: Person(name=junyong, age=30)
}

data class Person(val name: String, val age: Int)
```

- 여기서 `{ it.age }`는 선택자 로직을 구현한다. -> 선택자는 어떤 원소를 인자로 받아 비교에 사용할 값을 반환한다.
- 람다가 인자를 하나(컬렉션 원소)만 받고 그 인자에 구체적 이름을 붙이고 싶지 않기 때문에 **it** 이라는 암시적 이름을 사용한다.
- 위와 같은 예제에서는 원소가 Person 의 객체이므로 반환의 기준값이 Person 객체의 age 프로퍼티에 저장된 나이가 된다.

### 람다식의 문법

코틀린의 람다식은 아래와 같이 항상 줄괄호(`{}`)로 둘러싸여 있고, 인자 주변에는 괄호가 없다.

(중요!) 화살표(->)를 기준으로 파라미터와 본문을 구분한다.

- `{ x: Int, y: Int -> x + y }`
    - `x: Int, y: Int` : 파라미터
    - `x + y`: 본문 (처리 내용)

람다식을 아래와 같이 3가지 형태로 구성할 수 있다.

- people.maxByOrNull({ p: Person -> p.age })
- people.maxByOrNull() { p: Person -> p.age } -> 람다가 유일한 인자일 경우
- (추천!) people.maxByOrNull { p: Person -> p.age } -> 람다가 어떤 함수의 유일한 인자이고, 괄호 뒤에 람다를 사용한 경우 빈 괄호를 없앤다.

**람다가 함수의 유일한 인자라면, 괄호 없이 람다를 바로 사용하는 방법**을 적극 추천한다.

코틀린에서 인자가 여러 개일 때의 규칙

- 규칙1: 마지막 인자만 람다인 경우 -> 람다를 밖으로 빼내는 스타일을 선호한다. -> 이를 `후행 람다`라고 한다.
- 규칙2: 둘 이상의 람다를 인자로 받는 함수의 경우 -> 둘 이상의 람다를 괄호 밖으로 빼낼 수 없기 때문에, 이름 붙인 인자를 사용해 괄호 안에 넣는 것을 권장한다.

```kotlin
fun main() {
    val people = listOf(Person("fancy", 29), Person("junyong", 30))
    // NOTE: 규칙 1 스타일 - 마지막 인자가 람다이므로 밖으로 뺌
    println(people.joinToString(" ") { it.name })

    // NOTE: 규칙2 스타일 - 인자가 많아 헷갈린다면 괄호 안에 이름을 붙여 명시
    val names = people.joinToString(
        separator = " ",
        transform = { p: Person -> p.name }
    )
    println(names) // result: fancy junyong
}

data class Person(val name: String, val age: Int)
```

컴파일러는 람다의 파라미터 타입을 추론할 수 있다. -> 파라미터 타입을 명시하지 않아도 된다.

- `people.maxByOrNull{ p: Person -> p.age }` : 파라미터 타입을 명시한 경우
- `people.maxByOrNull{ p -> p.age }` : 파라미터 타입을 명시하지 않은 경우 컴파일러가 추론

그리고 람다의 파라미터 이름을 따로 지정하지 않으면 자동으로 `it`이라는 이름이 만들어진다.

- 만약 람다 안에 람다가 내포되는 경우 **각 람다의 파라미터를 명시해주는 것**이 낫다. -> 권장!

지금까지는 한 문장으로 이뤄진 작은 람다만 다뤘지만, 본문이 여러 줄로 이뤄진 경우도 있다.

이런 경우에는 본문의 맨 마지막에 있는 식이 람다의 결과값이 된다.

### 변수 캡처

함수 파라미터를 람다 안에서 사용하면 아래와 같다.

> 함수 파라미터를 람다 안에서 사용하는 경우

```kotlin
fun main() {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessagesWithPrefix(errors, "Error: ")
    // result
    // Error:  403 Forbidden
    // Error:  404 Not Found
}

fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}
```

- forEach 람다는 자신이 둘러싼 영역에 정의된 `prefix` 변수와 다른 변수에 접근할 수 있다.
- 참고: `forEach`는 컬렉션의 모든 원소에 대해 람다를 호출해준다. -> 일반적인 for 루프보다 훨씬 간결하지만 다른 장점은 많지 않으니 성급하게 바꾸지 않아도 된다.

여기서 코틀린과 자바 람다의 차이점은 코틀린 람다 안에서는 **final 변수가 아닌 변수에 접근할 수 있다는 점**이다.

- 즉, 람다 안에서 바깥의 변수를 변경해도 된다.
- 이렇게 **람다 안에서 접근할 수 있는 외부 변수**를 '람다가 캡쳐한 변수' -> `변수 캡처`라 부른다.

비동기 코드(이벤트 핸들러 등)에서 람다 사용 시 주의해야 할 점이 있다.

- 람다 안에서 로컬 변수를 변경할 수 있지만, **람다가 언제 실행되는지** 시점을 주의해야 한다.
- 예를 들어, 이벤트 핸들러는 함수가 이미 종료되어 **값을 반환한 이후에 실행**될 수 있다.
- 따라서 나중에 실행되어 값이 바뀌는 것을 추적하려면, 해당 변수를 함수 내부(로컬)가 아니라 **함수 외부(클래스 프로퍼티 등)에 선언**해야 한다.

```kotlin
class ClickCounter {
    var clicks = 0 // 변수를 함수 밖(클래스 레벨)으로 뺀다.

    fun setupButton(button: Button) {
        button.setOnClickListener {
            clicks++ // 이제 언제 눌려도 이 변수는 살아있음
        }
    }
}
```

### 멤버 참조

`val getAge = Person::age` -> `클래스:멤버` (멤버 -> 프로퍼티 or 메서드)

- `::` 을 사용하는 식을 `멤버 참조`(member reference)라고 부른다.

멤버 참조는 정확히 한 메서드를 호출하거나 한 프로퍼티에 접근하는 함수 값을 만들어준다.

people.maxByOrNull { p: Person -> p.age }
-> (멤버 참조 사용 시) `people.maxByOrNull(Person::age)`

또한, 멤버 참조는 클래스 이름이 아닌, **특정 객체(인스턴스)** 에 대한 멤버 참조를 만들 수 있다.

> 특정 객체에 대한 멤버 참조 - 예시 코드

```kotlin
fun main() {
    // 1. 기존 방식 (클래스::멤버)
    val p = Person("fancy", 29)
    val ageFunction = Person::age
    println(ageFunction(p))
    // result: 29

    // 2. 바운드 멤버 참조 (인스턴스::멤버) -> 대상이 p로 고정됨
    val myageFunction = p::age
    println(myageFunction()) // 실행할 때 인자가 필요 없음! (이미 p인 걸 아니까)
    // result: 29
}
data class Person(val name: String, val age: Int)
```

- 위와 같이 바운드 멤버 참조를 사용하면 참조를 생성할 때 **해당 객체를 함께 캡처(저장)** 한다.
- 따라서 함수를 실행할 때 **수신 객체(대상)를 인자로 넘길 필요가 없다.**

## 함수형 인터페이스 - 단일 추상 메서드

추상 메서드가 **단 하나만 있는 인터페이스**를 말한다. (`SAM`: Single Abstract Method)

코틀린에서는 `fun interface` 키워드를 사용해 정의한다.

- 추상 메서드는 딱 1개여야 하지만, 구현된 메서드(비추상 메서드)는 여러 개 가질 수 있다.

```kotlin
fun interface Calculator {
    fun calculate(x: Int, y: Int): Int // 추상 메서드가 딱 1개!
}
```

### SAM 변환

코틀린은 람다를 함수형 인터페이스가 필요한 자리에 인자로 넘길 때, 자동으로 인터페이스 구현체로 바꿔준다.

이를 SAM 변환이라 한다.

```kotlin
fun main() {
    // 정의된 함수는 'Calculator' 인터페이스를 원한다.
    // 람다만 던졌는데 자동으로 Calculator로 변환된다 (SAM 변환)
    doMath { x, y -> x + y }
}

fun doMath(calc: Calculator) {
    println(calc.calculate(1, 2))
}
```

### SAM 생성자

컴파일러가 자동으로 변환해주지 못하는 상황(주로 변수에 담거나, 반환할 때)에서, 람다를 **명시적으로 인터페이스로 포장**해주는 함수다.

- 인터페이스 이름과 같은 이름의 함수처럼 사용한다.
- `인터페이스명 { 람다 }`

주로 자동 변환이 안되는 두 가지 상황에서 사용된다.

- CASE 1: 람다를 인터페이스 타입 변수에 저장할 때

```kotlin
// 우변이 그냥 람다({..})면 뭔지 모르니 Calculator 라고 람다 앞에 작성해준다.
val calculator = Calculator { x, y -> x + y }
```

- CASE 2: 함수가 인터페이스 타입을 반환해야 할 때

```kotlin
fun createAllDoneRunnable(): Runnable {
    // 람다만 리턴하면 타입을 못 맞추므로 SAM 생성자를 사용한다.
    return Runnable { println("All done!") }
}
```

## 수신 객체 지정 람다 - with, apply, also

람다가 끝나고 무엇을 반환하는가?

- with: 객체로 여러 작업을 수행한 뒤 **람다의 결과(마지막 줄)를 반환**하고 싶을 때 사용한다.
    - 목적: 객체를 사용해 무언가 계산하거나 결과를 만들어낼 때.

- apply: 객체를 설정(초기화)한 뒤, **그 객체 자체(this)를 반환**받고 싶을 때 사용한다.
    - 목적: 객체 생성 후 초기화할 때.

- also: 체 자체를 반환받으면서, **객체를 인자(it)로 받아 추가적인 작업을 수행**하고 싶을 때 사용한다.
    - 목적: 체의 유효성 확인, 상태 변경, 로깅 등 객체를 사용하는 중간 단계의 작업을 수행할 때.

### with

`with` 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 **수신 객체**로 만든다.

- 람다 안에서 명시적인 this 참조를 사용해서 수신 객체에 접근할 수 있다.
- 또는 메서드나 프로퍼티 이름만 사용해 접근할 수 있다.

언제 사용하는가? -> 반환하려는 결과가 수신 객체 자체가 아니라, 람다 안에서 계산된 별도의 결과값일 때 사용한다.

아래 예시 코드와 같이 StringBuilder 객체를 받아서, 최종적으로는 String을 반환하는 경우에 사용된다.

```kotlin
fun main() {
    println("common: " + alphabet())
    println("with(CASE 1): " + alphabet2())
    println("with(CASE 2): " + alphabet3())
    println("with(CASE 3): " + alphabet4())
}

// NOTE: 일반적인 방법 - 알파벳을 만드는 경우
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z')
        result.append(letter)
    result.append("\nNow I know the alphabet!")
    return result.toString()
}

// NOTE: CASE 1) with를 사용해 알파벳을 만드는 경우
fun alphabet2(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            this.append(letter)
        }
        this.append("\nNow I know the alphabet!")
        this.toString()
    }
}

// NOTE: CASE 2) with를 사용해 알파벳을 만드는 경우 - this 를 생략할 수 있다.
fun alphabet3(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("\nNow I know the alphabet!")
        toString()
    }
}

// NOTE: CASE 3) with를 사용해 알파벳 만든 경우 - this 를 생략 + 불필요한 stringBuilder 변수를 없앤 경우
fun alphabet4() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```

### apply

람다의 결과 대신 **수신 객체가 필요한 경우**에는 `apply` 라이브러리 함수를 사용한다.

언제 사용하는가? -> 객체 생성과 동시에 초기화 작업을 수행하고, **변수에 그 객체를 바로 할당**하고 싶을 때 유용하다.

아래 예제(`alphabet5()`)에서 apply 블록 마지막 줄의 `toString()`은 무시된다.

- apply는 무조건 `StringBuilder` 객체 자신을 반환하기 때문이다.

```kotlin
fun main() {
    println("apply: " + alphabet5())
}

fun alphabet5() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString() // 이 줄은 실행은 되지만 반환값에는 영향을 주지 않는다.
}
```

만약 apply를 쓰면서 최종적으로 String을 반환하고 싶다면, apply가 끝난 뒤에 체이닝을 해야 한다.

```kotlin
// apply로 초기화를 끝낸 뒤(.apply{...}), 마지막에 .toString()을 호출
fun alphabetApplyToString() = StringBuilder().apply {
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("\nNow I know the alphabet!")
    }.toString()
```

with 과 유일한 차이점은 apply는 **항상 자신에 전달된 객체(=수신 객체)를 반환한다**는 점이다.

- 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 apply가 유용하다.
- 자바의 Builder 패턴이나 별도의 초기화 코드를 대체하여 본문을 간결한 식(=)으로 표현할 수 있다.

### also

(apply와 마찬가지로) 수신 객체를 받아 어떤 동작을 수행한 후 **다시 수신 객체 그대로를 반환**한다.

차이점은 also의 람다 안에서는 **수신 객체를 인자(`it`)** 로 받는다.

- 그래서 파라미터 이름을 부여하거나 기본값인 `it`을 사용해야 한다.

언제 사용하는가? -> 객체의 속성을 전혀 건드리지 않고 객체를 사용하는 추가적인 작업을 수행할 때 사용한다.

- e.g. 객체 상태 변경 후 DB 저장, 데이터 유효성 검사(validate), 디버깅 로그 출력 등

```kotlin
fun main() {
    val names = listOf("mjy", "fancy", "junyong")
    val uppercaseNames = mutableListOf<String>()
    val reversedLongNames = names
        .map { it.uppercase() }
        .also { uppercaseNames.addAll(it) } // 부수 작업 1: 다른 리스트에 데이터 백업
        .filter { it.length > 5 }
        .also { println(it) } // 부수 작업 2: 로그 출력
        .reversed()

    println("uppercaseNames: $uppercaseNames") // result: uppercaseNames: [MJY, FANCY, JUNYONG]
    println("reversedLongNames: $reversedLongNames") // reversedLongNames: [JUNYONG]
}
```

### 표로 정리하면..

| 함수    | 참조 방식  | 반환값   | 주 용도                         |
|-------|--------|-------|------------------------------|
| with  | `this` | 람다 결과 | 객체로 여러 작업 묶어서 실행             |
| apply | `this` | 수신 객체 | 객체 초기화 (설정)                  |
| also  | `it`   | 수신 객체 | 객체를 사용하는 추가 작업 (상태 변경, 로깅 등) |

---

# 6장. 컬렉션과 시퀀스

이 장에서는 코틀린 표준 라이브러리에서 컬렉션을 다룰 때 필수적으로 사용되는 함수형 API들을 다룬다.

그리고 컬렉션과 시퀀스의 차이점을 알아보고자 한다.

## 컬렉션에 대한 함수형 API

주의할 점은 이 함수들은 연쇄 호출 시 단계마다 **새로운 컬렉션(List, Set 등)을 생성**한다는 것이다.

### 원소 제거와 반환

원소 제거와 반환을 위해 `filter`와 `map`을 활용한다.

- `filter`: 조건(술어)을 만족하는 원소만 남긴다. -> 여기서 조건(술어)는 중괄호 `{...}` 안의 내용을 의미한다.
- `map`: 원소를 변환하여 새로운 값으로 구성된 컬렉션을 만든다.

```kotlin
val numbers = listOf(1, 2, 3, 4)
// 짝수만 남기고(filter), 제곱으로 변환(map)
// filter에서 [2, 4] 리스트 생성 -> map에서 [4, 16] 리스트 생성
val result = numbers.filter { it % 2 == 0 }.map { it * it }
println(result) // [4, 16]
```

### 데이터 누적과 집계

또한, 데이터 누적과 집계를 위해 `reduce`와 `fold` 함수를 사용한다. -> 컬렉션의 모든 원소를 바탕으로 하나의 결과값을 만들어낼 때 사용한다.

- `reduce`: 첫 번째 원소를 초기값으로 사용하며, 그 이후 원소들을 차례로 연산에 적용합니다. (빈 컬렉션일 경우 예외 발생 가능)
- `fold`: reduce와 달리, 임의의 초기값을 지정할 수 있고, 해당 초기값부터 시작하여 연산을 누적한다. (빈 컬렉션이어도 안전함, 반환 타입 변경 가능)

```kotlin
val list = listOf(1, 2, 3)

// reduce: 1 + 2 -> 3 + 3 -> 6
val sumReduce = list.reduce { acc, i -> acc + i }

// fold: 초기값 10 + 1 -> 11 + 2 -> 13 + 3 -> 16
val sumFold = list.fold(10) { acc, i -> acc + i }
```

### 조건 검사와 검색

컬렉션에 대해 자주 수행하는 연산으로, 컬렉션의 데이터 상태를 확인하거나 특정 데이터를 찾을 때 사용한다.

- `all`: 모든 원소가 조건을 만족하면 true를 반환한다.
- `any`: 하나라도 조건을 만족하면 true를 반환한다.
- `none`: 조건을 만족하는 원소가 하나도 없으면 true를 반환한다.
- `count`: 조건을 만족하는 원소의 개수를 반환한다.
- `find`: 조건을 만족하는 첫 번째 원소를 반환한다. (없으면 null 반환)

```kotlin
val ages = listOf(25, 14, 30, 20)
val canEnterBar = { age: Int -> age >= 20 }

println(ages.all(canEnterBar))  // false (14세 존재)
println(ages.any(canEnterBar))  // true
println(ages.count(canEnterBar)) // 3
println(ages.find(canEnterBar))  // 25 (가장 먼저 발견된 값)
```

### 리스트 분리

조건에 따라 컬렉션을 두 개의 그룹(만족 O, 만족 X)으로 나눌 때, `partition` 함수를 사용한다.

- 이때, 반환 타입이 `Pair<List, List>`이므로 구조 분해 선언을 통해 변수에 할당할 수 있다.

```kotlin
val nums = listOf(1, 2, 3, 4)

// 짝수 리스트(evens)와 홀수 리스트(odds)로 분리
val (evens, odds) = nums.partition { it % 2 == 0 }

println(evens) // [2, 4]
println(odds)  // [1, 3]
```

### 그룹화

특정한 기준(Key)에 따라 원소들을 분류하여 `Map<Key, List<Value>>` 형태로 반환할 때 `groupBy`를 사용한다.

- String의 확장 함수인 `first` 등을 멤버 참조(Member Reference) 문법(`::`)으로 전달하여 작성할 수 있다.

```kotlin
val words = listOf("apple", "banana", "apricot", "blueberry", "cherry")

// 첫 글자(it.first())를 기준으로 그룹화
// 멤버 참조 사용: words.groupBy(String::first) 와 동일
val byFirstLetter = words.groupBy { it.first() }

println(byFirstLetter)
// 출력: {a=[apple, apricot], b=[banana, blueberry], c=[cherry]}
```

### 컬렉션을 맵으로 변환

원소를 그룹핑하는 것이 아니라, 각 원소에서 키와 값을 추출해 Map<Key, Value> 형태로 만들고 싶을 때 `associate` 함수를 사용한다.

- 여기서 주의해야할 점은 키(Key)가 중복될 경우, `groupBy`와 달리 **마지막에 처리된 원소가 이전 값을 덮어쓰게 된다.**

```kotlin
// 람다 안에서 Key to Value 쌍을 반환해야하는 경우

val people = listOf("Alice", "Bob", "Charlie")

// 이름 -> 이름의 길이 형태의 맵을 생성
// { it to it.length } 부분이 키와 값을 정의합니다.
val nameToLength = people.associate { it to it.length }

println(nameToLength)
// 출력: {Alice=5, Bob=3, Charlie=7}
```

### 가변 컬렉션 원소 변경

`replaceAll`, `fill` 함수는 `MutableList`(수정 가능한 리스트)에서만 사용할 수 있으며,
새로운 리스트를 만드는 것이 아니라 기존 리스트의 내용을 직접 수정한다.

- `replaceAll`: 각 원소를 람다의 결과로 대체한다. -> map과 비슷하지만, 새 리스트를 만들지 않고 원본을 바꾼다.
- `fill`: 리스트의 모든 원소를 특정 값 하나로 덮어쓴다.

```kotlin
val items = mutableListOf("Apple", "Banana", "Cherry")

// 1. replaceAll: 규칙에 따라 내용을 바꿈
// 모든 원소를 대문자로 변경
items.replaceAll { it.uppercase() }
println(items) // [APPLE, BANANA, CHERRY]

// 2. fill: 모든 원소를 동일한 값으로 채움
// 리스트 내용을 전부 "Empty"로 초기화
items.fill("Empty")
println(items) // [Empty, Empty, Empty]
```

### 빈 값 처리 (ifEmpty vs ifBlank)

이 두 함수는 값이 없을 때 사용할 기본값을 지정하는 함수이다.

주로 문자열(String) 처리에 쓰이지만, `ifEmpty`는 컬렉션에도 쓸 수 있다.

가장 큰 차이점은 "**공백을 어떻게 취급하느냐**" 이다.

- `ifEmpty`: 길이가 0이어야만 동작한다.
- `ifBlank`: 길이가 0이거나, 공백 문자(스페이스, 탭, 줄바꿈 등)만 있을 때도 동작한다.

만약 "   "(공백이 3칸)의 경우

- `ifEmpty`는 "글자가 3개 있다"고 판단하여 동작을 안하고,
- `ifBlank`는 "의미 있는 내용이 없다"고 판단하여 동작한다.

```kotlin
val emptyStr = ""       // 진짜 빈 문자열
val blankStr = "   "    // 공백만 있는 문자열

// 1. ifEmpty 예시
println(emptyStr.ifEmpty { "기본값" }) // "기본값" (길이가 0이라서)
println(blankStr.ifEmpty { "기본값" }) // "   "   (길이가 3이라서 그대로 나옴)

// 2. ifBlank 예시 - 유효성 검사 기준에서 더 엄격하다.
println(emptyStr.ifBlank { "기본값" }) // "기본값"
println(blankStr.ifBlank { "기본값" }) // "기본값" (공백만 있어서 비어있다고 간주)
```

- 사용자 입력(ID, 닉네임 등)을 검증할 때 `ifBlank`를 사용하는 것을 권장한다.
- 단순히 리스트나 맵이 비어있는지 확인할 때는 `ifEmpty`를 사용한다.

## 시퀀스 - 게으르게 계산한다.

핵심: 시퀀스는 `데이터가 많거나 연산 단계가 많을 때` 성능을 최적화하기 위해 사용한다.

시퀀스는 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄하는 방법을 제공한다.

- 일반 컬렉션 함수(map, filter 등)는 단계마다 중간 결과물(임시 컬렉션)을 생성한다. -> 이는 데이터가 수백만 개라면 메모리 낭비가 심해진다.
- 반면, 시퀀스는 중간 결과를 저장하지 않고, 연산을 정의해 두었다가 최종 결과가 필요할 때(최종 연산 시) 한꺼번에 계산한다.

### 시퀀스 연산 - 중간, 최종

중간 연산

- map, filter 처럼 또 다른 시퀀스를 반환한다.
- 이 단계에서는 아무런 계산도 수행하지않고, 어떻게 계산할지만 초점을 둔다.

최종 연산

- toList, count, first 처럼 결과를 반환한다.
- 이 함수가 호출되는 순간, 미뤄뒀던 모든 계산이 수행된다.

컬렉션과 시퀀스는 데이터를 처리하는 방향이 다르다.

- 컬렉션은 모든 원소에 대해 1단계(map)를 끝내고, 그 결과물 전체에 대해 2단계(filter)를 수행한다.
- 시퀀스는 첫 번째 원소가 1단계 -> 2단계를 거쳐 결과가 되고, 그 다음 두 번째 원소가 1단계 -> 2단계를 거친다.

=> 무슨 차이일까?

- map을 하고 find로 첫 번째 값만 찾을 때,
- 컬렉션은 모든 원소를 map 하지만, 시퀀스는 조건을 만족하는 원소를 찾으면 뒤에 있는 원소들은 쳐다보지도 않고 연산을 끝낸다.

```kotlin
val list = listOf(1, 2, 3, 4)

// 1. 일반 컬렉션 (Eager)
// [1, 2, 3, 4] -> map 전체 수행 -> [1, 4, 9, 16] (임시 리스트 생성)
// -> find 수행 -> 4보다 큰 첫 값 9 반환
val resultList = list.map { it * it }.find { it > 3 }

// 2. 시퀀스 (Lazy)
// 1 -> map(1*1=1) -> find(1>3? X)
// 2 -> map(2*2=4) -> find(4>3? O) -> 4 반환! (끝)
// * 3, 4는 계산조차 안 함 (효율적)
val resultSeq = list.asSequence()
    .map { it * it }
    .find { it > 3 }
```

컬렉션과 달리, 시퀀스는 모든 연산은 각 원소에 대해 순차적으로 적용된다는 점이다.

- 즉, 첫 번째 원소가 처리되고, 다시 두 번째 원소가 처리되듯이 -> 모든 원소에 대해 적용된다.

### 시퀀스 만들기

컬렉션에서 변환하기(asSequence): 이미 있는 리스트나 세트를 시퀀스로 바꿀 때 사용한다.

```kotlin
val list = listOf(1, 2, 3)
val sequence = list.asSequence() // 시퀀스로 변환
```

직접 생성하기 (generateSequence): 이전 원소를 바탕으로 다음 원소를 계산하여 시퀀스를 만든다. `무한 시퀀스`를 만들 때 자주 사용된다.

```kotlin
// 0부터 시작해서 1씩 증가하는 시퀀스
val naturalNumbers = generateSequence(0) { it + 1 }

// 100까지만 가져와서 합계 구하기 (takeWhile이 없으면 무한 루프)
val sum = naturalNumbers.takeWhile { it <= 100 }.sum()
println(sum) // 0부터 100까지의 합
```

## 배운 점

- 컬렉션 연산의 비용과 시퀀스의 효율성에 대해 알게됐다.
    - 중간 결과를 저장하지 않고 원소별로 순차 처리(세로 방향)하므로 대용량 데이터 처리나 연쇄 연산 시에는 시퀀스를 사용하자!
- 집계 -> reduce는 빈 컬렉션에서 예외가 발생할 수 있으므로, 초기값을 지정할 수 있고 안전한 fold 함수를 사용하는 것이 좋다.
- 유효성 검사 -> 안전을 위해 ifBlank 함수를 사용하자.
- 변환 -> 키 중복 시 덮어쓰기가 발생하는 associate와 리스트로 묶어주는 groupBy의 차이를 명확히 구분해서 사용해야 한다.

---

# 7장. 널이 될 수 있는 값

## null이 될 수 있는 타입

타입 이름 뒤에 물음표(`?`)를 붙이면 변수나 프로퍼티에 null 참조를 저장할 수 있다.

- `Type?` = Type 또는 null

즉, 물음표가 없는 타입은 null 참조를 저장할 수 없다는 뜻이다.

- 기본적으로 코틀린의 모든 타입은 null이 아닌 타입이므로, null을 허용하려면 명시적으로 뒤에 `?`를 붙여야 한다.

코틀린은 이를 통해 컴파일 시점에 널 관련 오류를 미리 감지하고, 실행 시점에 발생할 수 있는 예외를 방지한다.

## null을 다루는 방법

### 안전한 호출자(safe call): `?.`

- null 검사와 메서드 호출을 한 번의 연산으로 수행한다.
- 수신 객체가 null이 아니라면 일반 메서드처럼 호출하고, null이라면 호출을 무시하고 결과값으로 null을 반환한다.

```kotlin
fun main() {
    // NOTE: 7.2
    val ceo = Employee("junyong", null)
    val developer = Employee("fancy", ceo)

    println(managerName(developer)) // junyong
    println(managerName(ceo)) // null
}

class Employee(val name: String, val manager: Employee?)

fun managerName(employee: Employee): String? = employee.manager?.name
```

`?.` 연산자를 사용하면 여러 개 연쇄해 사용할 수 있다.

### 엘비스 연산자 (Elvis Operator): ?:

- null 대신 사용할 **기본값** 을 지정할 때 사용한다.
- `?:`의 왼쪽 값이 null이 아니면 그 값을 쓰고, null이면 오른쪽 값을 쓴다.

```kotlin
fun main() {
    // NOTE: 7.4
    println(strLenSafe("abc")) // 3
    println(strLenSafe(null)) // 0 (length가 null이므로 0 반환)
}
fun strLenSafe(s: String?): Int = s?.length ?: 0
```

이 패턴은 함수의 **전제조건을 검사**할 때 특히 유용하다.

return이나 throw 같은 연산도 식이므로 엘비스 연산자 오른쪽에 올 수 있다.

```kotlin
fun main() {
    // NOTE: 7.5
    val address = Address("송도국제대로", 21986, "인천광역시", "대한민국")
    val kakaobank = Company("Kakaobank", address)
    val person = Person("junyong", kakaobank)

    printlnShippingLabel(person)
    // 송도국제대로
    // 21986 인천광역시 대한민국

    printlnShippingLabel(Person("fancy", null)) // 예외 발생!! (IllegalArgumentException)
}


// NOTE: 7.5
class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    val address = person.company?.address
        ?: throw IllegalArgumentException("No address")
    // 위에서 null 체크를 통과했으므로, 여기부터 address는 null이 아님을 보장받음
    // with를 썼을 때 (this가 address가 됨 -> 변수명 생략 가능)
    with(address) {// address는 null이 아니다.
        println(streetAddress)
        println("$zipCode $city $country")
    }
    // with를 안 썼을 때 (address 변수명을 계속 써야 함 -> 반복 발생)
    println(address.streetAddress)
    println("${address.zipCode} ${address.city} ${address.country}")
}
```

printShippingLabel 함수는 모든 정보가 제대로 있으면 주소를 출력하고,
주소가 없으면 NPE 대신 "No address"라는 메시지를 가진 명시적인 예외(IllegalArgumentException)를 발생시킨다.

참고: `with` 함수

- `with(obj) { ... }`를 사용하면 블록 안에서 obj가 수신 객체(this)가 된다.
- 따라서 `obj.streetAddress`라고 반복해서 쓸 필요 없이, 바로 `streetAddress`로 접근할 수 있어 코드가 깔끔해진다.

## 예외 없이 안전하게 타입 캐스트하기: as?

자바에서는 타입을 변환하기 전에 `instanceof`로 확인하지 않으면 `ClassCastException`이 발생할 위험이 있다.

코틀린의 `as?` 연산자는 변환하려는 타입과 호환되지 않을 경우, 예외를 던지는 대신 `null`을 반환한다.

- `as`: 변환 실패 시 예외 발생 (Unsafe Cast)
- `as?`: 변환 실패 시 null 반환 (Safe Cast)

```kotlin
fun main() {
    val obj: Any = 123 // Int 타입

    // 1. 성공 케이스: Int로 변환 시도
    val safeInt: Int? = obj as? Int
    println(safeInt) // 123 (성공)

    // 2. 실패 케이스: String으로 변환 시도
    val safeString: String? = obj as? String
    println(safeString) // null (변환 실패, 하지만 에러는 안 남)
}
```

as?는 단독으로 쓰기보다, 변환에 실패했을 때의 기본값이나 복귀(return) 처리를 위해 엘비스 연산자와 함께 자주 쓰인다.

가장 대표적인 예시는 equals() 메서드 구현이다.

```kotlin
class Person(val name: String, val age: Int) {
    override fun equals(other: Any?): Boolean {
        // 1. other를 Person으로 캐스팅 시도 (as?)
        // 2. 실패하면 null이 반환되므로, 엘비스 연산자(?:) 뒤의 return false 실행
        val otherPerson = other as? Person ?: return false

        // 3. 여기까지 왔다면 otherPerson은 null이 아닌 Person 타입임이 보장됨 (스마트 캐스트)
        return otherPerson.name == name && otherPerson.age == age
    }
}
```

## 널 아님 단언 선언하기: !!

`널 아님 단언`은 널이 될 수 있는 타입(Nullable)을 널이 될 수 없는 타입(Non-null)으로 강제로 바꿀 때 사용하는 가장 단순한 도구중 하나이다.

- 느낌표를 두 번(`!!`) 사용하면 "이 값은 절대로 널이 아니다"라고 선언하는 것과 같다.

```kotlin
fun main() {
    ignoreNulls(null)
}

// NOTE: 7.7
fun ignoreNulls(str: String?) {
    // "str은 절대 null이 아니야"라고 컴파일러에게 알린다.
    val strNotNull: String = str!!

    // 하지만 실제 값이 null이므로, 위 줄에서 NullPointerException(NPE)이 발생한다.
    println(strNotNull.length)
}
```

> 핵심 포인트 및 주의사항

- 컴파일러를 향한 강한 확신
    - `!!`는 컴파일러에게 "나는 이 값이 null이 아님을 100% 확신해. 만약 내 생각이 틀렸다면 예외(NPE)가 터져도 감수할게" 하는 말과 같다.
- 디버깅의 어려움 -> 한 줄에 여러 개 사용 금지
    - `!!`를 사용하여 예외가 발생했을 때, 스택 트레이스(Stack Trace)는 **몇 번째 줄**에서 에러가 났는지는 알려주지만, 그 줄의 **어떤 식**에서 났는지는 알려주지 않는다.
- 따라서 `person!!.company!!.address!!` 처럼 한 줄에 여러 단언문을 연달아 쓰는 것은 피해야 한다.
- 정확히 어떤 값이 null이라서 터졌는지 파악하기가 어렵기 때문이다.

## let 함수

`let`은 자신의 수신 객체(여기서는 email)를 람다 내부로 전달하는 스코프 함수이다.

(중요) 특히 **안전한 호출 연산자(?.)** 와 함께 사용하면, null이 아닌 경우에만 특정 코드 블록을 실행하는 패턴으로 자주 사용된다.

- 가장 흔한 사용 사례는 Nullable 타입의 값을 Non-null 타입만 받는 함수의 인자로 넘길 때이다.

```kotlin
fun main() {
    var email: String? = "junyong@example.com"
    // email이 null이 아니므로 블록 실행 -> sendEmailTo 호출됨
    email?.let { sendEmailTo(it) }

    email = null
    // email이 null이므로 let 블록 자체가 실행되지 않음 -> 아무 일도 안 일어남
    email?.let { sendEmailTo(it) } // null이므로 호출 안함
}

// NOTE: 7.9
fun sendEmailTo(email: String) { // 널이 아닌 String만 받을 수 있음
    println("Sending email to $email")
}
```

> 핵심 포인트

- `if (email != null)`과 같이 기존의 if문으로 널 체크를 하는 것보다, `?.let { ... }` 패턴을 사용하면 코드가 간결해진다.
- `it` 으로 접근
    - let 블록 내부에서는 값이 `null`이 아님이 보장되므로, 별도의 변수 선언 없이 `it`이라는 이름으로 안전하게 접근할 수 있다.
    - 물론 `email?.let { value -> sendEmailTo(value) }` 처럼 이름(value)을 직접 지정할 수도 있다.

## 나중에 초기화하기: lateinit

코틀린에서는 클래스의 **널이 아닌 프로퍼티**를 생성자 안에서 초기화하지 않으면 컴파일 에러가 발생한다.
하지만, 의존성 주입(DI)이나 유닛 테스트의 `@Setup` 메소드처럼 **생성자 이후에 초기화가 일어나는 경우**가 있다.

이때 초기화 값을 억지로 null로 맞추면, 실제 사용할 때마다 `!!`나 `?.`를 써야 하는 불편함이 생긴다.
이를 해결해주는 것이 바로 `lateinit`이다.

```kotlin
class MyService {
    fun performAction(): String = "Action Done"
}

class MyTest {
    // 1. 초기화 없이 선언 (null로 초기화할 필요 없음)
    lateinit var myService: MyService

    fun setUp() {
        // 2. 나중에(생성자 호출 이후) 실제 값 초기화
        myService = MyService()
    }

    fun testAction() {
        // 3. 널 검사 없이 바로 사용 가능 (컴파일러가 초기화되었다고 믿음)
        println(myService.performAction())
    }
}
```

> 핵심 포인트 및 제약사항

- 무조건 `var`여야 한다.
    - val은 생성 시점에 값이 고정되어야 하므로, 나중에 값을 채워 넣는 lateinit과는 함께 쓸 수 없다.
- 원시 타입은 불가능
    - Int, Long, Boolean, Double 등 자바의 원시 타입에 해당하는 프로퍼티에는 lateinit을 붙일 수 없다.
    - lateinit은 내부적으로 초기화되지 않은 상태를 null로 구분하는데, 원시 타입은 null을 가질 수 없기 때문이다.
- 초기화 확인
    - 만약 초기화되기 전에 접근하면 `UninitializedPropertyAccessException` 예외가 발생한다.

## 타입 파라미터의 널 가능성

코틀린에서 제네릭 클래스나 함수의 타입 파라미터 `T`는 기본적으로 널이 될 수 있는 타입(`Any?`)으로 취급된다.
따라서 `T`라고만 적어도 실제로는 `T?`처럼 동작한다.

만약 타입 파라미터가 절대 널이 아님을 보장하고 싶다면, Upper Bound(타입 상한)를 `Any`로 명시해야 한다.

```kotlin
// 1. 기본값: T는 null이 될 수 있음 (T : Any?)
fun <T> printHashCodeOrNull(t: T) {
    println(t?.hashCode()) // t가 null일 수 있으므로 안전한 호출(?.) 필요
}

// 2. 널 불가 선언: T는 null이 될 수 없음 (T : Any)
fun <T : Any> printHashCode(t: T) {
    println(t.hashCode()) // t는 항상 null이 아니므로 바로 접근 가능
}
```

## 널 가능성과 자바

자바와 상호작용할 때 코틀린은 자바의 널 가능성 어노테이션(Annotation)을 적극적으로 활용한다.

### 어노테이션이 있는 경우

자바 코드에 널 관련 어노테이션이 붙어 있다면 코틀린은 그에 맞는 타입으로 변환한다.

| 자바 어노테이션           | 코틀린 타입    | 의미        |
|--------------------|-----------|-----------|
| `@Nullable String` | `String?` | 널이 될 수 있음 |
| `@NotNull String`  | `String`  | 널이 될 수 없음 |

### 어노테이션이 없는 경우: 플랫폼 타입

가장 주의해야 할 점은 **자바 코드에 어노테이션이 없는 경우**이다.

이때 코틀린은 해당 값이 널인지 아닌지 알 수 없으므로 **플랫폼 타입**으로 취급한다.

```java
/* Java Class */
public class Person {
    public String getName() { /**/ } // 어노테이션 없음
}
```

```kotlin
/* Kotlin */
val person = Person()
// 둘 다 컴파일 가능하지만, 런타임에 name이 null이면 위쪽 코드는 에러 발생
val name1: String = person.name  // (위험) 널이 아님을 확신할 때
val name2: String? = person.name // (안전) 널일 수도 있다고 가정할 때
```