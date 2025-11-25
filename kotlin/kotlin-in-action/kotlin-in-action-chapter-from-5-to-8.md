# Kotlin in Action 2/E, 5-8장

> 이 글은 [Kotlin IN Action 2/E](https://product.kyobobook.co.kr/detail/S000215768644) 책을 읽고 정리한 내용입니다.

대상 독자

* 자바 경험이 있는 개발자

* 서버 개발자 or 안드로이드 개발자와 같이 JVM에서 실행될 프로젝트를 구축하는 모든 개발자

---

# 5장. 람다를 사용한 프로그래밍

람다식 또는 람다는 기본적으로 **다른 함수에 넘길 수 있는 작은 코드 조각**을 의미한다.

- 람다식을 사용하면 쉽게 공통 고드 구조를 라이브러리 함수로 뽑아낼 수 있어서, 코틀린 표준 라이브러리는 람다를 많이 사용한다.

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
- 규칙2: 둘 이상의 람다를 인자로 받는 함수의 경우 -> 둘 이상의 람다를 괄호 밖으로 빼낼 수 없기 때문에, 이름이 붙인 인자를 사용해 괄호 안에 넣는 것을 권장한다.

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

지금까지는 한 문장으로 이뤄진 작은 란다만 다뤘지만, 본문이 여러 줄로 이뤄진 경우도 있다.

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

추상 메서드가 **단 하나만 있는 인터페이**스를 말한다. (`SAM`: Single Abstract Method)

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

- with: 객체로 여러 작업을 수행한 뒤 람다의 결과(마지막 줄)를 반환하고 싶을 때 사용한다.
    - 목적: 객체를 사용해 무언가 계산하거나 결과를 만들어낼 때.

- apply: 객체를 설정(초기화)한 뒤, 그 객체 자체(this)를 반환받고 싶을 때 사용한다.
    - 목적: 객체 생성 후 초기화할 때.

- also: 객체에 대해 부수적인 작업(로깅, 유효성 검사)을 수행한 뒤 그 객체 자체를 반환받고 싶을 때 사용한다.
  - 목적: 데이터 변경 없이 추가 작업을 수행할 때.

### with

`with` 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 **수신 객체**로 만든다.

- 람다 안에서 명시적인 this 참조를 사용해서 수신 객체 접근할 수 있다.
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

차이점은 also의 람다 안에서는 **수신 객체를 인자(`it`)**로 받는다.
- 그래서 파라미터 이름을 부여하거나 기본값인 `it`을 사용해야 한다.

언제 사용하는가? -> 객체의 속성을 전혀 건드리지 않고 객체를 사용하는 부수적인 작업을 할 때 사용한다.
- e.g. 디버깅 로그 출력, 유효성 검사, 데이터 복사

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

| 함수 | 참조 방식 | 반환값 | 주 용도 |
| --- | --- | --- | --- |
| with | `this` | 람다 결과 | 객체로 여러 작업 묶어서 실행 |
| apply | `this` | 수신 객체 | 객체 초기화 (설정) |
| also | `it` | 수신 객체 | 부수 효과 (로깅, 유효성 검사) |


---

