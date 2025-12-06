# Spring Boot, Kotlin 함수형으로 안전하게 서킷 브레이커 구현하기

## Prologue

외부 API 연동 시 주의해야할 점 중 하나는 외부 시스템에 장애가 발생하는 경우입니다.

외부 시스템의 응답 지연이나 에러가 우리 시스템의 스레드 고갈로 이어지는 것을 방지하고,
장애 상황에 유연하게 대처하기 위해 서킷 브레이커(Circuit Breaker) 패턴을 사용합니다.

이번 글에서는 서킷 브레이커의 핵심 개념과 상태 변화, 그리고 Resilience4j를 활용한 구체적인 설정 방법을 정리해 봅니다.

그리고 이후에 SpringBoot, Kotlin 기반으로 서킷 브레이커를 적용한 경험을 공유하고자 합니다.

## CircuitBreaker란

![](./img/circuit-breaker-concept-and-apply-springboot-kotlin-1.png)

`CircuitBreaker`(서킷 브레이커)는 문제가 발생한 지점을 감지하여 실패하는 요청을 차단하고(Open),
이를 통해 시스템의 장애 확산을 막고 장애 복구를 도와주는 기능을 제공합니다.

서킷 브레이커를 지원하는 대표적인 라이브러리는 다음과 같습니다.

- Netflix Hystrix (deprecated)
    - MSA 환경에서 서킷 브레이커 패턴을 구현하여 장애 전파를 막고 장애를 감지하고 복구할 수 있는 내결함성을 갖는 오픈소스 라이브러리입니다.
    - 현재는 개발이 중단되어 유지보수 모드이며, Resilience4j 사용을 권장하고 있습니다.
- Resilience4j
    - Java 8의 함수형 프로그래밍을 기반으로 설계된 경량화 라이브러리입니다.
    - CircuitBreaker 외에도 RateLimiter, Retry, TimeLimiter 등 다양한 코어 모듈을 제공합니다.

### 슬라이딩 윈도우(Sliding Window) 알고리즘

서킷 브레이커는 호출 결과를 저장하고 집계하기 위해 슬라이딩 윈도우 알고리즘을 사용합니다.

Resilience4j는 두 가지 타입을 제공합니다.

- Count-based sliding window (횟수 기반): 마지막 N번의 호출 결과를 집계합니다. (예시. 최근 100번의 요청 중 실패율 계산)
- Time-based sliding window (시간 기반): 마지막 N초 동안의 호출 결과를 집계합니다. (예시. 최근 10초 동안 들어온 요청들의 실패율 계산)

## CircuitBreaker의 3가지 상태

서킷 브레이커는 일반적으로 3가지 상태를 가지며, 이외에 특수 상태로 2가지를 가집니다.

![](./img/circuit-breaker-concept-and-apply-springboot-kotlin-2.png)

- `Closed` (정상 상태)
    - 평소 정상적으로 요청을 처리하는 상태입니다.
    - 실패율이나 느린 응답률이 임계치를 넘으면 `Open` 상태로 전환되며, 서킷을 끊어버립니다.
- `Open` (장애/차단 상태)
    - 장애가 감지되어 외부 요청을 즉시 차단하는 상태입니다.
    - 요청 시 CallNotPermittedException 예외를 발생시키거나 Fallback 함수를 실행합니다.
    - 장애 판단 기준
        - Failure Rate: 실패 혹은 오류 응답의 비율.
        - Slow Call Rate: 설정한 시간보다 오래 걸린 요청의 비율.
- `Half_Open` (검증 상태)
    - `Open` 상태에서 일정 시간이 지난 후, 시스템이 복구되었는지 확인하기 위해 진입하는 상태입니다.
    - 제한된 횟수의 요청만 허용하여 성공 여부를 확인한 뒤, 다시 `Closed`나 `Open`으로 결정합니다.

특수 상태

- `Disabled`: 서킷 브레이커를 비활성화하여 항상 요청을 허용합니다.
- `Forced_Open`: 강제로 서킷을 열어 항상 요청을 거부합니다.

## Resilience4j 주요 설정값

Resilience4j의 `CircuitBreakerConfig`에서 사용하는 주요 설정값을 상태 전환 흐름에 따라 정리했습니다.

### Closed -> Open

실패율(Failure Rate) 또는 느린 호출율(Slow Call Rate)이 임계치를 초과할 때 전환됩니다.

- 단, **최소한의 호출 횟수**가 쌓여야 계산이 시작됩니다.

> 관련 설정값에 대해 설명과 기본값을 정리했습니다.

- `failureRateThreshold`
    - 실패율이 이 설정값(%) 이상이 되면 서킷이 Open 상태로 전환됩니다.
    - 기본값: 50

- `slowCallRateThreshold`
    - 느린 호출의 비율이 이 설정값(%) 이상이 되면 서킷이 Open 상태로 전환됩니다.
    - 기본값: 100

- `slowCallDurationThreshold`
    - 이 시간(ms)보다 오래 걸리는 호출을 느린 호출(Slow Call)로 간주합니다.
    - 기본값: 60000 [ms] (60초)

- `minimumNumberOfCalls`
    - 실패율이나 느린 호출율을 계산하기 위해 필요한 최소한의 호출 횟수입니다.
    - 이 횟수만큼 데이터가 쌓이기 전에는 서킷이 절대 열리지 않습니다.
    - 기본값: 100

- `slidingWindowType`
    - 호출 결과를 집계하는 방식입니다. `COUNT_BASED`(최근 N회) 또는 `TIME_BASED`(최근 N초) 중 선택합니다.
    - 기본값: `COUNT_BASED`

- `slidingWindowSize`
    - 슬라이딩 윈도우의 크기입니다. `COUNT_BASED`라면 호출 횟수(N회), `TIME_BASED`라면 시간(N초)을 의미합니다.
    - 기본값: 100

### Open -> Half_Open

Open 상태에서 일정 시간이 지나면 Half_Open 상태로 전환을 시도합니다.

- `waitDurationInOpenState`
    - 설명: 서킷이 Open 상태로 진입한 후, Half_Open 상태로 전환되기 전까지 대기하는 시간(ms)입니다.
    - 기본값: 60000 [ms] (60초)

- `automaticTransitionFromOpenToHalfOpenEnabled`
    - `true`로 설정 시, 대기 시간이 지나면 별도의 모니터링 스레드가 자동으로 상태를 Half_Open으로 변경합니다.
    - `false`일 경우, 대기 시간이 지난 후 **첫 번째 요청이 들어와야** 상태가 변경됩니다.
    - 기본값: false

### Half_Open

서킷이 열려있던 상태에서, 서비스가 복구되었는지 확인하기 위해 **제한된 횟수의 요청**만 허용합니다.

- `permittedNumberOfCallsInHalfOpenState`
    - Half_Open 상태일 때 허용되는 요청의 횟수입니다.
    - 이 횟수만큼의 호출 결과를 바탕으로 서킷을 다시 닫을지(Closed) 열지(Open) 결정합니다.
    - 기본값: 10

- `maxWaitDurationInHalfOpenState`
    - Half_Open 상태에서 머무를 수 있는 최대 시간입니다.
    - 0일 경우: 허용된 횟수(permittedNumberOfCalls)가 다 찰 때까지 무한정 기다립니다.
    - 0보다 큰 경우: 해당 시간이 지나면 강제로 Open 상태로 전환됩니다.
    - 기본값: 0

### Half_Open -> Closed

Half_Open 상태에서의 집계 결과가 정상 범위(임계치 미만)일 경우 전환됩니다.

- 별도의 전용 프로퍼티 없이, 앞서 설정한 `failureRateThreshold`와 `slowCallRateThreshold`를 기준으로 판단합니다.
- `permittedNumberOfCallsInHalfOpenState` 만큼의 요청을 수행한 후, **실패율이 임계치보다 낮으면** Closed 상태로 돌아갑니다.

### Half_Open -> Open

Half_Open 상태에서의 집계 결과가 여전히 비정상(임계치 이상)일 경우 전환됩니다.

- 마찬가지로 `failureRateThreshold`와 `slowCallRateThreshold`를 기준으로 판단합니다.
- **실패율이 임계치보다 같거나 높으면** 다시 Open 상태로 돌아가며, 다시 `waitDurationInOpenState` 만큼 대기해야 합니다.

### 예외 처리 설정 (Record & Ignore)

모든 예외가 서킷을 열어야 하는 장애는 아닙니다.

비즈니스 로직상의 예외는 무시하고, 네트워크 에러만 실패로 기록하고 싶을 때 사용합니다.

- `recordExceptions`
    - 실패로 기록할 예외 클래스들을 지정합니다.
    - 여기에 지정된 예외(및 그 하위 클래스)가 발생하면 **실패 카운트**가 증가합니다.
    - 참고: 목록을 명시하면, 여기에 포함되지 않은 다른 예외들은 성공으로 간주됩니다.
    - 기본값: empty (기본적으로 모든 예외를 실패로 간주하지만, 명시하면 해당 예외만 실패로 봅니다.)
- `ignoreExceptions`
    - 실패나 성공 어느 쪽으로도 기록하지 않고 무시할 예외 클래스들을 지정합니다.
    - 비즈니스 로직 에러(예: UserNotFoundException) 처리에 유용합니다.
    - 기본값: empty
- 우선순위 주의사항
    - ignoreExceptions가 recordExceptions보다 우선순위가 높습니다.
    - 즉, 어떤 예외가 두 설정 목록에 모두 포함되어 있다면, 해당 예외는 **무시**됩니다.
- `recordFailurePredicate`
    - 예외 객체를 인자로 받아 **실패 여부**(true/false)를 반환하는 커스텀 함수를 설정합니다.
    - 복잡한 예외 판별 로직이 필요할 때 사용합니다.
    - 기본값: throwable -> true (모든 예외를 실패로 간주)
- `ignoreExceptionPredicate`
    - 예외 객체를 인자로 받아 **무시 여부**(true/false)를 반환하는 커스텀 함수를 설정합니다.
    - 기본값: throwable -> false (무시하지 않음)

실무에서는 recordExceptions에 IOException, TimeoutException과 같은 명확한 인프라/네트워크 장애만 등록하고, 나머지 예외는 성공으로 처리되도록 하는 방식을 선호합니다.

이렇게 하면 코드 버그로 인해 서킷이 열려 정상적인 서버가 차단되는 상황을 방지할 수 있습니다.


---

## Spring Boot, Kotlin 기반 CircuitBreaker 실습해보기

> 아래 코드와 관련한 코드는 [Github](https://github.com/devFancy/kotlin-java-playground/tree/main/springboot-kotlin-circuit-breaker)을 참고해 주시면 감사하겠습니다.

앞서 CircuitBreaker의 개념과 주요 설정값을 알아봤습니다.

이번에는 Spring Boot, Kotlin 기반으로 이를 실제로 적용한 과정을 공유하고자 합니다.
(아래부터는 Spring Boot, Kotlin 의 기본적인 문법에 대한 설명은 생략했습니다.)

먼저 `build.gradle.kts` 에 서킷 브레이커와 관련하여 의존성을 추가합니다.

> build.gradle.kts

```kotlin
dependencies {
    implementation("org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j")
    implementation("org.springframework.boot:spring-boot-starter-aop")
}
```

spring-boot-starter-aop는 `@CircuitBreaker` 어노테이션 방식을 사용할 때 프록시(Proxy) 기반으로 동작하기 위해 필수적입니다.

그런 다음, `application.yml` 파일에 아래와 같이 Circuit Breaker 설정값을 정의합니다.

> application.yml

```yaml
# @CircuitBreaker name에 지정된 인스턴스가 없으면 default 설정을 기반으로 생성합니다.
resilience4j:
  circuitbreaker:
    configs:
      default:
        minimum-number-of-calls: 5          # 집계에 필요한 최소 호출 수
        sliding-window-size: 5              # CLOSED 상태에서 최근 N회 호출을 기준으로 실패율(failureRateThreshold) 계산
        failure-rate-threshold: 10          # 실패 10% 이상 시 OPEN 상태로 전환
        slow-call-duration-threshold: 500   # 500ms 이상 소요 시 실패로 간주
        slow-call-rate-threshold: 10        # slowCallDurationThreshold 초과 비율이 10% 이상 시 OPEN 상태로 전환
        wait-duration-in-open-state: 10000   # OPEN -> HALF_OPEN 전환 전 대기 시간 (ms)
        permitted-number-of-calls-in-half-open-state: 5   # HALF_OPEN -> CLOSE or OPEN 으로 판단하기 위해 허용할 호출 횟수
```

## 기존 @CircuitBreaker 방식의 한계

가장 보편적으로 알려진 방법은 `@CircuitBreaker` 어노테이션을 사용하는 것입니다.

> CircuitAnnotationController.kt

```kotlin
@RestController
class CircuitAnnotationController() {

    // fallbackMethod는 본 함수와 파라미터, 반환타입이 일치해야 합니다.
    @CircuitBreaker(name = "user", fallbackMethod = "failSample")
    @GetMapping("/annotation/user/{id}")
    fun getSampleUser(@PathVariable id: String): User {
        val list = listOf(
            IllegalStateException("illegalState"),
            IllegalArgumentException("illegalArgument"),
        )
        throw list.random()
    }

    // IllegalArgumentException 발생 시 호출
    private fun failSample(throwable: IllegalArgumentException): User {
        return User("IllegalArgumentException name", "IllegalArgumentExceptionfail introduce")
    }

    // IllegalStateException 발생 시 호출
    private fun failSample(e: IllegalStateException): User {
        return User("IllegalStateException name", "IllegalStateExceptionfail introduce")
    }
}
```

`@CircuitBreaker`의 name 속성에 해당하는 설정이 없으면, `application.yml`의 default 옵션에 있는 값으로 name 속성의 CircuitBreaker가 생성됩니다.

또한, `fallbackMethod` 속성에 메서드명을 지정하면 예외가 발생했을 때 지정된 메서드가 대신 실행되어 기본 응답을 내려줄 수 있습니다.

만약 서킷이 Open 상태로 바뀌어서 차단된 경우, 더 이상 요청은 전달되지 않으면서 Resilience4j는 CallNotPermittedException을 발생시킵니다.

이 예외 또한 CallNotPermittedException 예외를 받아서 처리하는 failSample 함수를 구현해서 처리할 수도 있습니다.

### 어노테이션 방식은 무엇이 문제일까?

하지만 실무에서 이 방식을 사용하다 보면 몇 가지 문제가 있습니다.

- 런타임 에러 위험
    - fallbackMethod 이름을 문자열("failSample")로 지정하기 때문에, 오타가 나거나 메서드 파라미터, 반환타입이 달라지더라도 컴파일 시점에 알 수 없습니다.
    - 애플리케이션이 실행되고 나서야 에러를 발견하게 됩니다.
- 낮은 응집도
    - 비즈니스 로직과 장애 처리 로직(fallback)이 하나의 클래스 안에 혼재될 수 있습니다.
    - 이 경우에서 실패하게 되면 어떤 fallback이 실행되는지 찾기 위해 코드를 찾아야 합니다.
- 구현체 의존성
    - 서킷이 열렸을 때 발생하는 `CallNotPermittedException`은 Resilience4j 라이브러리의 고유 예외입니다.
    - 이를 처리하려면 비즈니스 로직이 특정 라이브러리(Resilience4j)를 알고 있어야 합니다.
- 같은 클래스의 내부 메서드 호출 불가능
    - Spring AOP는 프록시 패턴으로 동작하므로, 같은 클래스 내부의 메서드를 호출(this.method)할 때는 서킷 브레이커가 동작하지 않습니다.

## 해결 방안: 함수형 스타일로 개선하기

위의 문제들을 해결하기 위해, Kotlin의 함수형 프로그래밍 스타일을 도입하여 구조를 개선해보겠습니다.

### 핵심 인터페이스 및 구현체

> CircuitBreaker.kt

```kotlin
interface CircuitBreaker {
    /**
     * @param name 서킷 브레이커 설정 이름 (기본값: "default")
     * @param block 서킷 브레이커로 보호하며 실행할 로직
     */
    fun <T> run(name: String = "default", block: () -> T): Result<T>
}

@Component
class DefaultCircuitBreaker(
    private val factory: CircuitBreakerFactory<*, *>
) : CircuitBreaker {
    private val log = LoggerFactory.getLogger(javaClass)

    override fun <T> run(
        name: String,
        block: () -> T
    ): Result<T> {
        val circuitBreaker = factory.create(name)

        // runCatching을 사용하여 예외를 던지지 않고 Result 객체로 반환
        return kotlin.runCatching {
            circuitBreaker.run(block) { e ->
                log.warn("[CircuitBreaker: $name] 실행 중 예외 발생: ${e.message}", e)
                // 라이브러리 전용 예외를 커스텀 예외로 변환하여 전파
                throw e.convertToCustomException()
            }
        }
    }
}
```

위 코드가 수행하는 역할은 다음과 같습니다.

- `DefaultCircuitBreaker` 역할
    - Spring Cloud에서 제공하는 `CircuitBreakerFactory`는 create 메서드로 인스턴스를 생성하고,
    - `run` 메서드의 첫 번째 인자로 실행할 로직을, 두 번째 인자로 예외 발생 시 처리할 fallback 함수를 전달합니다.
    - 즉, DefaultCircuitBreaker 클래스는 이러한 복잡한 CircuitBreakerFactory의 사용법을 직접 노출하지 않고, 내부로 캡슐화하여 비즈니스 로직에서 쉽게 사용할 수 있도록 감추는
      역할을 합니다.

- Result 타입
    - Kotlin의 `Result<T>` 타입을 사용하여 성공과 실패를 하나의 객체로 캡슐화합니다.
    - **체이닝이 가능한 이유**
        - 일반적인 `try-catch`는 예외가 발생하면 코드의 실행 흐름이 끊기고 catch 블록으로 넘어갑니다.
        - 반면, Result 타입은 예외가 발생해도 이를 `Result.Failure` 라는 객체에 담아 반환합니다. 즉, 실패조차도 하나의 값으로 취급하게 됩니다.
        - 따라서 뒤에 나오는 CircuitUtilController 부분에서 `run(...).fallback(...).getOrThrow()` 처럼 앞 단계의 결과 여부에 따라 다음 단계로 자연스럽게
          이어질 수 있게 만들 수 있습니다.

- convertToCustomException
    - Resilience4j의 `CallNotPermittedException`을 직접 던지지 않고, 제가 정의한 커스텀 예외로 변환합니다.
    - 이를 통해 비즈니스 로직은 특정 라이브러리에 의존하지 않게 됩니다.

> runCatching 은 왜 사용했나요? - 관련 [코틀린 공식문서](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/run-catching.html)

runCatching은 Kotlin 1.3부터 도입된 표준 라이브러리 함수로, 예외가 발생할 수 있는 코드를 안전하게 실행하고 그 결과를 `Result` 객체로 감싸줍니다.

내부 코드를 살펴보면 아래와 같습니다.

```kotlin
@InlineOnly
@SinceKotlin("1.3")
public inline fun <R> runCatching(block: () -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}
```

위와 같이 블록 내부에서 발생한 모든 Throwable을 확인해서 `Result.failure(e)`로 변환해 줍니다.

또한 `inline` 함수로 정의되어 있는데, 이는 컴파일 시점에 함수의 내용이 호출 위치에 그대로 복사되는 방식으로 동작합니다.

따라서 람다 함수 실행을 위해 별도의 객체를 생성하는 메모리 오버헤드가 없으며, 성능 저하 없이 깔끔한 구문을 사용할 수 있습니다.

추가적으로 Result은 단순한 객체가 아니라 `@JvmInline` value class로 정의되어 있습니다.

```kotlin
@JvmInline
public value class Result<out T> @PublishedApi internal constructor(
    @PublishedApi
    internal val value: Any?
) : Serializable
```

이는 `Optional` 같은 일반적인 래퍼 클래스와 달리, **런타임에 추가적인 힙 메모리 할당을 최소화한**다는 장점이 있습니다.

- 컴파일러가 가능한 경우 래퍼 객체를 생성하지 않고 내부의 값을 직접 사용하도록 최적화합니다.
- 덕분에 서킷 브레이커처럼 빈번하게 호출되는 로직에서도 GC 부담을 줄이고 높은 성능을 유지할 수 있습니다.

### 유틸리티 확장 함수로 정의하기

이제 Result 타입을 더욱 유용하게 다루기 위한 확장 함수를 정의합니다.

> CircuitBreakerUtils.kt

```kotlin
// Result<T> 확장 함수: 실패 시 무조건 fallback 실행
fun <T> Result<T>.fallback(function: (e: Throwable?) -> T): Result<T> = when (this.isSuccess) {
        true -> this
        false -> kotlin.runCatching { function(this.exceptionOrNull()) }
    }

// NOTE: Result<T> 확장 함수: 서킷이 열린 경우(CircuitOpenException)에만 fallback 실행
fun <T> Result<T>.fallbackIfOpen(function: (e: Throwable?) -> T): Result<T> = when (this.exceptionOrNull()) {
    is CircuitOpenException -> runCatching { function(this.exceptionOrNull()) }
    else -> this
}

// NOTE: Resilience4j 예외를 커스텀 예외로 변환
fun Throwable.convertToCustomException(): Throwable = when (this) {
    is CallNotPermittedException -> CircuitOpenException()
    else -> this
}

class CircuitOpenException(message: String = "Circuit breaker is open") : RuntimeException(message)
```

- fallback: 어떤 예외든 실패했다면 전달받은 지정된 로직을 실행하여 Result로 감싸서 응답합니다.

- fallbackIfOpen: 일반적인 런타임 에러는 그대로 두고, **서킷이 열려서 차단된 경우**에만 전달받은 지정된 로직을 실행하여 Result로 감싸서 응답합니다.

- convertToCustomException: 앞서 `DefaultCircuitBreaker` 클래스 정의 부분에서 설명한 `CallNotPermittedException` 예외를 커스텀
  예외(`CustomException`)로 변경하는 함수입니다.

- CircuitOpenException: 라이브러리 종속적인 예외(CallNotPermittedException)를 대신할 커스텀 예외입니다.

### 개선된 Controller

최종적으로 아래와 같이 Controller 클래스를 개선할 수 있습니다.

> CircuitUtilController.kt

```kotlin
@RestController
class CircuitUtilController(
    private val circuitBreaker: CircuitBreaker
) {

    @GetMapping("/util/users/fallback")
    fun getFallbackSampleUser(): User {
        return circuitBreaker.run("fallback-user") {
            throw RuntimeException("runtime")
        }.fallback {
            User("Fallback name", "Fallback introduce")
        }.getOrThrow()
    }

    @GetMapping("/util/users/open")
    fun getFallbackOpenSampleUser(): User {
        return circuitBreaker.run("fallback-open-user") {
            throw RuntimeException("runtime")
        }.fallbackIfOpen {
            User("Fallback Open Default name", "Fallback Open Default introduce")
        }.getOrThrow()
    }
}
```

코드가 실행(run) -> 실패 시 대처(fallback) -> 값 추출(getOrThrow) 의 형태로 읽히므로 전보다 가독성이 높아지고 의도가 명확해졌습니다.

참고로, 위 코드에서 `circuitBreaker.run("name") { ... }` 를 보면, 실행할 로직이 담긴 중괄호가 소괄호 밖에 나와있는 것을 볼 수 있습니다.

- 이는 Kotlin의 후행 람다 문법을 이용한 것입니다.
    - 함수의 마지막 매개변수가 람다인 경우, 해당 람다식을 소괄호 바깥으로 뺄 수 있습니다.
    - 즉, circuitBreaker.run("name", { ... }) 라고 써야 할 것을, 후행 람다 덕분에 더 직관적으로 표현한 것입니다.

## 테스트로 검증해보기

마지막으로, 제가 만든 구조가 의도대로 동작하는지 검증해보겠습니다.

Mockk를 사용하여 간단하게 단위 테스트를 작성할 수 있습니다.

> CircuitUtilControllerTest.kt

```kotlin
class CircuitUtilControllerTest {

    private val circuitBreaker: CircuitBreaker = mockk()
    private val controller = CircuitUtilController(circuitBreaker)

    @Test
    fun `Controller는 CircuitBreaker가 실패해도 fallback 값을 반환한다`() {
        // given
        // execute가 호출되면 무조건 실패(Result.failure)를 반환하도록 설정
        every {
            circuitBreaker.run<Any>(any(), any())
        } returns Result.failure(RuntimeException("Error"))

        // when
        val user = controller.getFallbackSampleUser()

        // then
        assertThat(user.name).isEqualTo("Fallback name")
    }
}
```

![](./img/circuit-breaker-concept-and-apply-springboot-kotlin-3.png)

> CircuitBreakerUtilsTest.kt

```kotlin
class CircuitBreakerUtilsTest {

    @Test
    fun `fallback은 실패 시 대체 값을 반환한다`() {
        // given
        val failureResult = Result.failure<String>(RuntimeException("Error"))

        // when
        val result = failureResult.fallback { "Recovered" }

        // then
        assertThat(result.getOrNull()).isEqualTo("Recovered")
    }

    @Test
    fun `fallbackIfOpen은 CircuitOpenException일 때만 대체 값을 반환한다`() {
        // given: 서킷 오픈 예외 발생
        val circuitOpenResult = Result.failure<String>(CircuitOpenException())

        // when
        val result = circuitOpenResult.fallbackIfOpen { "Recovered from Open" }

        // then
        assertThat(result.getOrNull()).isEqualTo("Recovered from Open")
    }

    @Test
    fun `fallbackIfOpen은 일반 예외일 때는 무시하고 에러를 유지한다`() {
        // given: 일반 런타임 예외
        val runtimeError = RuntimeException("General Error")
        val failureResult = Result.failure<String>(runtimeError)
        var isFallbackExecuted = false

        // when
        val result = failureResult.fallbackIfOpen {
            isFallbackExecuted = true
            "Should not happen"
        }

        // then
        // 결과가 여전히 실패인지 확인
        assertThat(result.isFailure).isTrue()
        assertThat(result.exceptionOrNull()).isEqualTo(runtimeError)
        // fallback 블록 내부가 실행되지 않았는지 확인
        assertThat(isFallbackExecuted).isFalse()
    }

    @Test
    fun `Resilience4j 예외가 커스텀 예외로 잘 변환된다`() {
        // given
        val r4jException = mockk<CallNotPermittedException>()
        every { r4jException.fillInStackTrace() } returns r4jException

        // when
        val converted = r4jException.convertToCustomException()

        // then
        assertThat(converted).isInstanceOf(CircuitOpenException::class.java)
    }
}
```

![](./img/circuit-breaker-concept-and-apply-springboot-kotlin-4.png)

## 마무리하기

지금까지 외부 API 연동 시 발생할 수 있는 장애에 유연하게 대처하기 위해 서킷 브레이커(CircuitBreaker) 패턴을 도입하는 과정을 정리했습니다.

우선 Resilience4j의 핵심 개념과 상태, 그리고 다양한 설정값들이 실제로 어떻게 동작하는지 살펴봤습니다.

단순히 라이브러리를 적용하는 것에 그치지 않고, 기존 @CircuitBreaker 어노테이션 방식이 가진 한계점을 분석하고 이를 해결하는 과정을 공유했습니다.

그 결과, Spring Boot와 Kotlin 환경에서 runCatching과 Result 패턴, 그리고 확장 함수를 활용해서 함수형 스타일로 서킷 브레이커 구조를 개선했습니다.

이 과정을 통해 얻은 이점들을 정리한 결과 아래와 같습니다.

- 문자열 기반의 설정에서 코드로 제어해서 컴파일 타임의 안정성을 확보했습니다.
- 비즈니스 로직과 장애 복구 로직을 파이프라인 형태로 연결해서 가독성과 응집도를 높였습니다.
- inline 함수를 통한 성능 최적화와 Result 타입의 value class의 메모리 효율성에 대해서도 다시 한번 이해하게 되었습니다.

물론, 코드를 개선했다고 끝은 아닙니다. 실무에서는 서킷 브레이커의 설정값에 대한 튜닝이 중요합니다.

본문에서 다룬 임계치나 슬라이딩 윈도우 크기는 예시일 뿐이며,
실제 운영 환경에서는 서비스의 트래픽 패턴, 외부 시스템의 응답 속도, 장애 허용 범위와 같은 내부 정책에 맞춰
지속적인 모니터링과 튜닝이 동반되어야 한다고 생각합니다.

이번 글이 안정적인 서비스를 고민하는 분들께, 그리고 더 나은 코드를 작성하기 위해 고민하는 분들께 작은 도움이 되길 바라며 글을 마치겠습니다.

## Reference

- [[공식문서] CircuitBreaker](https://resilience4j.readme.io/docs/circuitbreaker)
- [[공식문서] Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker#overview)
- [[공식문서] Kotlin - runCatching](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/run-catching.html)
- [[우아한형제들] 개발자 의식의 흐름대로 적용해보는 서킷브레이커](https://techblog.woowahan.com/15694/) - 2024.01.11
- [[당근테크] 서킷브레이커 사용 방식 개선하기 | 당근 SERVER 밋업 2회](https://www.youtube.com/watch?v=ThLfHtoEe1I) - 2023.11.23
- [[올리브영] Circuitbreaker를 사용한 장애 전파 방지](https://oliveyoung.tech/2023-08-31/circuitbreaker-inventory-squad/) - 2023.08.31
- [[개인 블로그] Spring - CircuitBreaker](https://backtony.tistory.com/83)
