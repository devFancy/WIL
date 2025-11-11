# 가치있는 테스트를 위한 전략과 구현

> 아래는 토스 기술 블로그에서 업로드된 [해당 글](https://toss.tech/article/test-strategy-server)을 보고 정리한 글입니다.
>
> 발행 날짜: 2024. 10. 24

## Prologue

그동안 테스트 코드를 작성해오면서 나름의 전략을 세워왔지만, 여전히 배워나가야 할 부분들이 많다고 느꼈다.

그러면서 자연스럽게 다른 회사들은 어떻게 테스트 전략을 수립하고 실행했는지 궁금했다.

그러던 중 토스 기술 블로그의 "가치있는 테스트를 위한 전략과 구현"이라는 글을 발견했고, 해당 글이 실무에서 겪는 문제를 해결해 나간 구체적인 경험이 잘 담겨 있다고 느껴졌다.

토스팀은 어떻게 테스트 전략을 수립하고 구현했는지 그 과정을 자세히 살펴보고, 정리하고자 이 글을 읽고 정리하게 되었다.


---

## 1. 가치 있는 테스트 코드에 대하여

테스트에 대한 이상과 현실

- 테스트가 현실에서 올바른 가치를 지니지 못하는 여러가지 이유가 존재한다.
    - 작성해야 하는 테스트의 절대적인 양이 많음
    - 구현에 강결합되어 깨지기 쉬움
    - 테스트 코드 자체가 이해하기 어렵게 작성되어있음
    - 테스트가 느리게 실행되며 간헐적으로 실패함

좋은 테스트와 그렇지 않은 테스트

- 좋은 테스트는 **FIRST** 규칙을 따라야 한다.
    - Fast: 테스트는 빠르게 동작하여 자주 돌릴 수 있어야 함
    - Independent: 각각의 테스트는 독립적이며 서로 의존해서는 안 됨
    - Repeatable: 어느 환경에서도 반복 가능해야함
    - Self-Validating: 테스트는 성공 또는 실패로 bool 값으로 결과를 내어 자체적으로 검증되어야 함
    - Timely: 테스트하려는 실제 코드를 구현하기 직전에 구현해야 함

- 또한, 완전성과 간결성과 같은 특징들도 좋은 테스트가 되는데 일조한다.
    - 완전한 테스트 -> 사람이 결과에 도달하기까지의 필요한 모든 정보를 가지고 있는 테스트
    - 간결한 테스트 -> 코드가 복잡하지 않고 불필요한 정보가 포함하지 않은 테스트

- 결국 좋은 테스트는 **비즈니스 적으로 가치 있는 테스트**여야 한다.
    - 커버리지 향상 등의 목적은 테스트 작성에 많은 시간을 소비하고 의미가 없다.

가치있는 테스트 코드 작성 전략 "세 가지"

- 작성 가치를 고려하여 선택적으로 작성하기 -> 약 20% 가 시스템의 핵심 기능을 담당함
    - **가치 있는 20%의 테스트로 80% 이상의 신뢰성을 얻는다.**
- 최대한 실용적으로 작성하기
    - 하나의 테스트로 모든 계층을 커버하면 부담 최소화할 수 있고, 문서화 역할까지 하면 -> 일석이조 !
    - 적지만 많은 계층을 커버할 수 있는 `통합 테스트`를 작성한다.
- 확실한 목적에 따라 작성하기
    - 크게 세분화하면 -> 도메인 정책 테스트, 유스케이스 테스트, 직렬화/역직렬화 테스트

## 2. 도메인 정책 테스트 코드 작성하기

### 단위 테스트로 작성하기

`도메인 정책 테스트`란 특정한 도메인 정책이 올바른지 빠르게 검증하는 걸 의미한다. 비즈니스 정책을 검증하므로 문서화의 역할까지 할 수 있다.

- 여기서 `도메인 정책`은 도메인 객체 내에 표현되는 비즈니스 정책으로, "프로모션 참여대상은 18세 이하"로 도메인 엔티티 내에 표현될 수 있다.

```kotlin
data class Member(
        val name: String,
        val age: Int,
) {

    fun canParticipatePromotion(): Boolean {
        return age <= MAX_PROMOTION_AGE
    }

    companion object {
        const val MAX_PROMOTION_AGE = 18
    }
}
```

- 참고로 테스트 대상을 SUT(System-Under-Test)라고 부르며, **경계 값**을 중심으로 테스트하면 더욱 좋은 테스트를 작성할 수 있다.
    - 해피(성공) 케이스: 프로모션 참가는 18세 이하까지 가능함
    - 예외(실패) 케이스: 프로모션 참가는 19세 이상부터 불가능함

```kotlin
class MemberTest : FunSpec({

    context("canParticipatePromotion") {
        test("프로모션 참가는 18세 이하까지 가능함") {
            listOf(12, 17, 18).forEach {
                val member = Member(
                        name = "MangKyu",
                        age = it,
                )

                member.canParticipatePromotion() shouldBe true
            }
        }

        test("프로모션 참가는 19세 이상부터 불가능함") {
            listOf(19, 20, 30).forEach {
                val member = Member(
                        name = "MangKyu",
                        age = it,
                )

                member.canParticipatePromotion() shouldBe false
            }
        }
    }
})
```

### 테스트 대역이 아닌 실제 객체 사용하기

SUT와 협력이 필요한 객체가 존재하는데, 이를 `협력자`(Collaborator)라고 부른다.

- 협력자의 경우 실체 객체 또는 테스트 대역을 사용할 수 있다.
- 여기서 테스트 대역은 가짜 객체를 통칭하는 의미로 다섯 가지로 세분화한다. -> 자세한 건 개인 블로그에
  작성한 [포스팅](https://devfancy.github.io/Practical-Testing2/#test-double)을 참고하자!
    - Dummy object, Spy, Stub, Mock, Fake

실제 객체를 사용하고자 하는 집단을 고전파(Classicist), 테스트 대역을 사용하고자 하는 집단을 런던파(London School 또는 Mockist) 라고 한다.

- 토스 포인트 지급 시에 포인트 지급 코드가 미성년자일 경우, 그렇지 않은 경우 달라지는 로직이 있다고 가정한다.
- 고전파는 실제 객체를 생성해서 두 객체의 협력을 통해 최종 결과를 테스트한다.
- 반면, 런던파는 가짜 객체를 만들고 결과를 강제로 지정해서 특정 기능만 고립시켜 테스트한다.
    - 이렇게 되면 -> 런던파는 테스트가 "어떻게 동작하는지" 같은 **내부 구현에 너무 의존적**이게 된다.
    - 또한, 비즈니스 로직에 대해 리팩터링을 하게 되었을 때 -> 기능은 그대로임에도 테스트가 깨지는 **강결합 문제**가 발생한다.
- 이러한 경험을 바탕으로 **실제 객체를 활용**하는 것이 더 유용하다고 판단한다.
    - 모의 객체 사용을 최소화 -> 가능한 실제 객체를 활용!
    - 실제 구글에서도 과거에 모의 객체를 사용하다가 많은 문제가 생겼고, 이러한 교훈을 바탕으로 실제 객체를 활용하는 기조가 형성되었다고 한다.
    - 관련 구글 공식 블로그 (원본 자료) (궁금해서 찾아봤다 ㅎㅎ)
        - ["Testing on the Toilet: Don't Overuse Mocks" (2013년)](https://testing.googleblog.com/2013/05/testing-on-toilet-dont-overuse-mocks.html)
        - [Increase Test Fidelity By Avoiding Mocks" (2024년)](https://testing.googleblog.com/2024/02/increase-test-fidelity-by-avoiding-mocks.html)

도메인 엔티티 생성을 위한 코드가 다른 모듈에도 필요해질 수 있다.

- 이때 test-fixtures 라는 라이브러리를 활용할 수 있다.
- 자세한 내용은 다른 토스 아티클인 [테스트 의존성 관리로 높은 품질의 테스트 코드 유지하기](https://toss.tech/article/how-to-manage-test-dependency-in-gradle)
  을 참고하자.

## 3. 유스케이스 테스트 코드 작성하기

`유스케이스 테스트`는 인수 테스트로서 사용자의 여정이 올바르게 동작하는지 검증하는 걸 의미한다.

### 통합 테스트 및 인수 테스트로 작성하기

우리의 목표는 "고객으로부터 비즈니스 가치를 창출하는 것"이다.

- 특정 사용자의 여정이 의도한 대로 이루어지는지 보장하기 위해 인수 테스트를 작성하고
- 이를 통해 사용자의 여정을 따라가므로 문서화의 역할까지 할 수 있다.
- (관련된 자세한 코드가 궁금하면, 토스 아티클에서 확인하자.)

### 애노테이션 기반의 테스트 컨텍스트 추상화

스프링 부트는 `관례`를 선호하는 방식으로 설계되어 있다.

- 따라서 이러한 철학에 맞게 커스텀 애노테이션을 추가하여 테스트가 동작될 수 있도록 했다.

```kotlin
@Retention(AnnotationRetention.RUNTIME)
@ActiveProfiles("test")
@SpringBootTest(
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
        classes = [MainApplication::class],
        properties = ["spring.profiles.active=test"]
)
@AutoConfigureMetrics
@TestExecutionListeners(
        value = [AcceptanceTestExecutionListener::class],
        mergeMode = TestExecutionListeners.MergeMode.MERGE_WITH_DEFAULTS
)
@Import(LazyInitExcludeConfiguration::class)
annotation class AcceptanceTest(
        @get:AliasFor("setUpScripts")
        val value: Array<String> = [],
        val setUpScripts: Array<String> = []
)

@Profile("test")
@Configuration
private class LazyInitExcludeConfiguration {
    @Bean
    fun lazyInitializationExcludeFilter(): LazyInitializationExcludeFilter {
        return LazyInitializationExcludeFilter.forBeanTypes(
                AESCipherSupport::class.java,
        )
    }
}
```

테스트 속도 최적화를 위해 빈이 필요한 순간에 lazy하게 생성될 수 있도록 `spring.main.lazy-initialization=true` 옵션을 활용했다.

- 이때 일부 빈들이 Eager하게 생성하지 않아 테스트가 실패하는 문제가 발생했다.
- 이를 해결하기 위해 Eager하게 생성되어야 하는 빈들은 명시적으로 지정하기 위해 `LazyInitExcludeConfiguration` 이 추가했다.

커스텀 애노테이션을 통해 또 하나의 포인트는 **테스트 컨텍스트의 재사용**이다. -> `@AcceptanceTest` 애노테이션

- 각각의 테스트마다 스프링 컨테이너를 띄우면 테스트 속도가 느려진다.
- 속도 문제를 해결하고자 컨텍스트를 캐싱하여 동일한 컨텍스트를 활용하는 테스트끼리 공유하도록 했습니다
- 이를 통해 초기 1회 테스트 비용을 지불하면 -> 이후의 테스트들에서는 훨씬 빠른 실행 속도로 처리될 수 있다.
- 컨텍스트 캐싱에 대한 자세한 내용은
  [공식 문서](https://docs.spring.io/spring-framework/reference/testing/testcontext-framework/ctx-management/caching.html)
  를 활용하자.

### 테스트 데이터 셋업과 테스트 격리를 위한 TestExecutionListener 추가

> 자세한 내용은 해당 아티클을 참고하자. (실무에서 비슷한 상황을 마주치게 된다면 그때 다시보자.)

테스트 격리성을 위해 테스트 메서드 실행 직전에 JSON 파일을 읽어 INSERT 하고,
테스트 실행 직후에는 사용자 테이블을 모두 TRUNCATE 하도록 처리했다.

- 참고로, 테스트에서는 H2 DB를 사용하고 있어서, 관련 문법들로 처리했다.

### 통합 테스트에서 테스트 대역의 활용

불가피하게 테스트 대역을 활용하거나 테스트 대역이 적합한 순간들이 있다.

따라서 다음과 같은 선택 기준을 세웠지만 절대적인 기준이 아니다. (팀바팀)

- 토스 외부의 서비스인 경우 -> `Fake` 객체를 구현
- 토스 내부의 서비스인 경우
    - 비즈니스 외적인 부수효과가 존재하는 경우: `Dummy` 객체를 사용
    - 데이터 확보가 어려운 경우: 모의 객체 프레임워크를 통해 `Stub` 사용
    - 그 외 대부분인 경우: 실제 객체를 사용

## 4. 직렬화/역직렬화 테스트 코드 작성하기

직렬화/역직렬화 테스트는 직렬화/역직렬화가 문제가 없는지만 빠르게 검증한다.

### 단위 테스트로 작성하기

단위 테스트로 작성할 때,

- 일부 필드명이 바뀌는 경우 -> 호환성 문제가 발생할 수 있다.
- 이를 해결하고자 캐시 버저닝을 사용하는데, 다음과 같이 두 가지 규칙을 세워서 캐시에 대한 관리 비용을 줄이고자 했다.
    - 캐싱의 활용을 최소화할 것
    - 호환성 검증 테스트를 작성할 것

사내 모니터링을 통해 불가피하게 캐시가 필요한 순간에만 도입하고자 했다.

- 또한, 이렇게 캐시를 도입할 때 -> 반드시 테스트를 작성하고
- 해당 테스트가 실패할 경우 "호환성을 고려하라" 는 테스트 신호를 의식적으로 챙기고 있다.

### 승인 테스트로 작성하기

`승인 테스트`(Approval Test)는 주어진 입력들에 대한 출력을 스냅샷으로 저장하고, 테스트 수행 결과가 동일한지를 검사하는 걸 의미한다.

실제 프로덕션 환경에서 캐시 저장소의 값을 읽어 활용하는 것이

- 일종의 스냅샷이 저장된 상태에서 호환성 유무를 판단하는 것과 유사한 승인 테스트로 작성하게 되었다.
- [ApprovalTests.Java 라이브러리](https://github.com/approvals/ApprovalTests.Java)를 활용하고 있다.

```kotlin
class ChargingChannelApprovalTest {

    @Test
    fun chargingChannelApprovalTest() {
        val chargingChannel = ChargingChannel.create(
                userId = -1L,
                bankCode = 23,
                expirationDate = null,
        )

        val result = JsonUtil.writeValueAsString(chargingChannel)

        Approvals.verify(result)
    }
}
```

- 그러고 나면 txt 파일에 저장된 스냅샷과 테스트 실행 결과를 비교하여 테스트 성공/실패를 결정하게 된다.

## 5. 그 외 기타 테스트 작성하기

전체적인 테스트를 작성할 가치를 판단하고, 그렇지 않다면 부분 기능만을 테스트하도록 대체할 수 있다.

- 내부 로직이 복잡한 경우 -> 특정 기능만을 테스트 코드로 작성한다.
- 단, 모든 코드를 이렇게 변경하진 않고 기준을 정한다.
    - 예시. 비즈니스 생명 주기가 상대적으로 짧고 국소적인 기능일 경우
- 상황에 따라 가치 판단을 하여 적합한 전략을 가져간다.

또한, '학습 테스트'를 통해 특정 도구의 동작이나 기능을 학습할 수도 있다.

## 6. 현재 방식의 한계와 개선 방법

유스케이스 테스트에서 개선 방법

- 테스트 셋업이 어렵기 때문에, 테스트가 주는 가치를 생각해 보자.
- 밀폐성과 충실성 사이에서 시스템에 맞게 테스트의 균형점을 찾자.
- 테스트가 첫 번째 고객이라는 관점으로 생각하면 -> (팀과의 합의를 통해) 테스트라는 고객을 위해 프로덕션 코드를 변경할 수도 있다.

## 마무리

테스트를 통해 충분히 가치를 느낄 수 있는 부분도 많다.

- 코드 리팩터링 시에 실제 잘못된 변경을 사전에 탐지함
- 애플리케이션 설정 등이 잘못되어 서버 구동에 실패함
- 스프링 부트 버전업 시 외부 라이브러리 호환성 문제를 런타임에 발견함

테스트는 소프트웨어와 개발자를 모두 더 나아지게 하므로
성숙한 개발자가 되기 위해 테스트를 작성할 필요가 있다.

모든 프로덕션 코드에 테스트를 작성하는 것이 아닌, 상황에 맞게 실용적인 테스트 전략을 가져갈 필요가 있다.
(모든 문제를 한 방에 해결하는 완벽한 방안은 없다 -> 모든 테스트 전략에는 Trade-off가 존재한다 !)

## Review

토스의 기술 블로그에 있는 해당 글을 정리하면서, 실무 기반의 경험이 담긴 글이면서 구체적으로 어떤 고민을 했고 어떤 실용적인 전략을 선택했고 실행했는지를 엿볼 수 있어서 너무 좋았다.

사실, 인수 테스트 부분과 3번의 여러 테스트 관련 심화 내용들에 대해 제대로된 경험해 본 적이 없었다.

이번 글을 통해 토스팀처럼 비즈니스가 크고 정책이 많고 복잡한 서비스일수록 인수 테스트가 주는 가치가 크다고 생각이 들고, 추후 꼭 적용해봐야겠다는 동기부여도 얻었다.

다만, 서비스 성장 단계를 고려해야 한다고 생각이 들었다.

서비스 초기에는 정책과 요구사항이 자주 바뀔 수 있기 때문에, 처음부터 인수 테스트를 도입하는 것은 오히려 관리 비용을 증가시킬 수 있다고 본다.

우선 단위 테스트부터 작성하고 서비스가 안정화되는 시점 이후부터 통합 테스트, 그리고 인수 테스트를 점진적으로 도입하고 작성하는 게 시간과 비용 측면에서 더 합리적인 전략이라고 판단했다.
