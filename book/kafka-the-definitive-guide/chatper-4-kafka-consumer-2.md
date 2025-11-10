# [카프카 핵심 가이드] CHAPTER 4. 카프카 컨슈머: 컨슈머 설정 및 오프셋과 커밋


> 이 글은 [카프카 핵심 가이드](https://product.kyobobook.co.kr/detail/S000201464167?utm_source=google&utm_medium=cpc&utm_campaign=googleSearch&gad_source=1) 책을 읽고 정리한 글입니다.
>
> 이 글에서 다루는 모든 코드는 [깃허브](https://github.com/devFancy/springboot-coupon-system)에서 확인하실 수 있습니다.
>
> `참고`: 본 글에서 소개되는 코드 예시는 현재 시점의 구현을 바탕으로 작성되었으며, 프로젝트가 발전함에 따라 내용이 변경되거나 개선될 수 있음을 미리 알려드립니다.


## Prologue

"카프카 핵심 가이드" 4장 내용을 바탕으로 [이전 글](https://devfancy.github.io/Kafka-Consumer/) 에 이어 컨슈머 설정 및 오프셋과 커밋에 대해 내용을 정리했다.

이론적인 개념 설명과 더불어, 실제 Spring Kafka 환경에서 어떻게 컨슈머를 구현하고 사용하는지 개인 프로젝트 코드를 통해 함께 살펴본다.


---

## 컨슈머 설정하기

컨슈머의 성능과 가용성에 영향을 주는 몇몇 중요한 설정들이 있다. 아래는 여러 설정들 중에 중요한 속성 위주로 정리했다.


## 데이터 가져오기

컨슈머가 브로커로부터 데이터를 얼마나, 어떻게 가져올지를 결정하는 설정이다. 처리량(Throughput)과 지연 시간(Latency) 사이의 균형을 맞추는 것이 중요하다.

### fetch.min.bytes

컨슈머가 브로커로부터 데이터를 가져올 때, 최소한 이 크기(byte) 이상의 데이터가 모였을 때만 응답하도록 요청하는 설정이다. (기본값: 1)

토픽에 메시지가 드문드문 들어오는 상황에서, 이 값을 적절히 높이면 브로커는 데이터가 충분히 쌓일 때까지 기다렸다가 한 번에 보내주므로 양쪽의 부하를 줄일 수 있다.

단, 이 값을 높이면 응답을 기다리는 시간 때문에 지연 시간은 늘어날 수 있다.

### fetch.max.wait.ms

`fetch.min.bytes`에 설정된 데이터가 모일 때까지 브로커가 **최대 얼마나 기다릴지**를 결정한다. (기본값: 500ms)

카프카는 `fetch.min.bytes`에 도달하거나, `fetch.max.wait.ms` 시간이 초과되면 둘 중 먼저 만족하는 조건에 따라 컨슈머에게 데이터를 응답한다.

이 설정은 컨슈머의 최대 지연 시간을 제어하는 데 사용된다.

### max.poll.records

`poll()` 메서드를 한 번 호출할 때 가져오는 **최대 메시지 개수**를 지정한다. (기본값: 500)

가져온 메시지를 하나씩 처리하는 로직이 매우 오래 걸린다면, 이 값을 작게 조절하여 `poll()` 루프의 실행 시간을 제어할 수 있다.

이는 아래에서 설명할 `max.poll.interval.ms`와 밀접한 관련이 있다.


## 그룹 안정성과 상태 관리

컨슈머가 그룹 내에서 '살아있음'을 알리고, 비정상적인 상황에서 어떻게 동작할지를 결정하는 매우 중요한 설정이다.

### session.timeout.ms

> [공식 문서](https://kafka.apache.org/documentation/#consumerapi) 기준으로 Kafka Consumer 에서는 `session.timeout.ms` 기본값이 45초이다.
>
> 2021.04.21 기준, `session.timeout.ms` 값은 기본값이 10초에서 45초로 변경된 것을 확인할 수 있다. (관련 내용: [KIP-735: Increase default consumer session timeout](https://cwiki.apache.org/confluence/display/KAFKA/KIP-735%3A+Increase+default+consumer+session+timeout))


컨슈머가 브로커(그룹 코디네이터)에게 하트비트(생존 신호)를 보내지 않고 버틸 수 있는 최대 시간이다. (기본값: 45,000ms = 45초)

만약 이 시간 동안 하트비트가 없으면, 코디네이터는 해당 컨슈머에 장애가 발생했다고 판단하고 그룹에서 제외시킨 뒤 **리밸런싱**을 시작한다.

### heartbeat.interval.ms

컨슈머가 얼마나 자주 하트비트를 보낼지 결정하는 주기이다. (기본값: 3,000ms = 3초)

이 값은 반드시 `session.timeout.ms`보다 낮아야 하며, 보통 **1/3 수준으로 설정**하는 것이 일반적인 규칙이다.

### max.poll.interval.ms

`poll()` 호출 사이의 최대 시간 간격이다. (기본값: 300,000ms = 5분)

하트비트는 백그라운드 스레드에서 보내므로, 메시지를 처리하는 메인 스레드가 멈춰도 하트비트는 계속 전송될 수 있다.

`poll()`을 호출한 후 다음 `poll()`이 이 시간 안에 호출되지 않으면, 컨슈머는 그룹에서 이탈하고 리밸런싱이 발생한다.

메시지 처리 로직이 **5분 이상** 걸릴 가능성이 있다면 이 값을 반드시 늘려야 한다.

> `session.timeout.ms` 과 `max.poll.interval.ms` 의 차이점은?

* `session.timeout.ms`은 컨슈머 **프로세스 자체**가 살아있는지(Liveness) 감시하는 역할이고,

* `max.poll.interval.ms`은 컨슈머의 **메시지 처리 로직**이 일을 하고 있는지(Progress) 감시하는 역할이다.

#### 리밸런싱 방지하기 위한 방법

`max.poll.interval.ms` 설정을 잘못 다루면 컨슈머 그룹에서 리밸런싱이 발생할 수 있다.

`max.poll.records`를 비정상적으로 높게 설정하고, `max.poll.interval.ms`를 총 처리 시간보다 낮게 설정하면 리밸런싱이 발생할 수 있다.

이를 위해 애플리케이션의 총 처리 시간을 고려해서 `max.poll.records` 를 낮게 설정하거나 `max.poll.interval.ms` 를 늘리는 방안을 고려한다.

## 파티션 할당 전략

리밸런싱이 발생했을 때, 그룹 내 컨슈머들에게 파티션을 어떻게 분배할지 결정하는 알고리즘이다. `partition.assignment.strategy`로 설정한다.

* `Range` (기본값): 토픽별로 파티션을 정렬한 뒤 컨슈머에게 연속된 범위를 할당한다. 파티션 개수가 컨슈머 수로 나누어 떨어지지 않으면 특정 컨슈머에 할당이 몰릴 수 있다.

* `RoundRobin`: 모든 파티션을 모아 컨슈머들에게 하나씩 순서대로(Round-Robin) 할당한다. 비교적 균등한 분배를 보장한다.

* `Sticky`: 최대한 균등하게 분배하면서, 리밸런싱 시 **파티션 이동을 최소화**하여 안정성을 높인다.

* `CooperativeSticky`: `Sticky` 전략의 개선판으로, **협력적 리밸런싱**을 지원한다. 리밸런싱 중에도 재할당 대상이 아닌 컨슈머들은 **작업을 멈추지 않아** 서비스 중단을 최소화하는 가장 진보된 방식이다.


## 오프셋과 커밋

`poll()`을 호출하면 컨슈머 그룹이 아직 읽지 않은 레코드를 리턴한다. 즉, 컨슈머는 파티션 내에서 어디까지 읽었는지 위치를 추적해야 한다.

카프카에서는 **파티션의 현재 위치를 업데이트하는 작업**을 `오프셋 커밋`(Offset Commit)이라고 부른다.

안전한 처리 흐름은 "메시지 처리 완료 -> 오프셋 커밋" 순서를 따르는 것이다.

이는 책의 페이지를 다 읽고 나서 그 뒤에 책갈피(오프셋)를 꽂는 것과 같다. 이렇게 해야 컨슈머에 장애가 발생해도 어디까지 읽었는지 정확히 알 수 있다.

컨슈머는 자신이 성공적으로 처리한 마지막 메시지의 오프셋을 커밋하고, 이 정보는 `__consumer_offsets`라는 토픽에 저장된다.

리밸런스가 발생하면, 컨슈머는 이 커밋된 오프셋을 읽어와서 작업이 중단된 지점부터 처리를 재개한다.

이 오프셋 관리는 매우 중요하다. 만약 오프셋 커밋이 실제 처리 시점과 맞지 않으면 데이터 중복이나 유실이 발생할 수 있다.

* 메시지 중복: 실제 처리한 메시지보다 **이전 오프셋**이 커밋된 상태에서 장애가 발생하면, 리밸런스 후 다음 컨슈머가 이전 오프셋부터 다시 읽기 시작하여 메시지가 중복 처리된다.

* 메시지 유실: 실제 처리한 메시지보다 **이후 오프셋**이 커밋된 상태에서 장애가 발생하면, 리밸런스 후 다음 컨슈머가 커밋된 오프셋부터 읽기 시작하여 중간 메시지들이 유실된다.

### auto.offset.reset

컨슈머 그룹이 특정 파티션에 대해 저장된 오프셋 없이 처음으로 읽기를 시작하거나, 기존 오프셋이 더 이상 유효하지 않을 때 어떻게 동작할지를 결정한다.

(만약 유효한 오프셋이 있다면, 컨슈머는 이 설정을 무시하고 마지막으로 커밋된 지점부터 작업을 이어간다.)

* `latest` (기본값): 가장 최신 메시지, 즉 컨슈머가 실행된 **이후**에 들어오는 메시지부터 읽기 시작한다.

* `earliest`: 파티션에 저장된 **가장 오래된** 메시지부터 읽기 시작한다. (모든 데이터를 처음부터 처리)

* `none`: 유효한 오프셋이 없으면 예외(Exception)를 발생시킨다.

### enable.auto.commit

`poll()` 메서드가 주기적으로 오프셋을 자동으로 커밋할지 여부를 결정한다. (기본값: true)

* `true` (자동 커밋): 가장 쉬운 방법이지만, **메시지 유실 가능성**이 있다.

    * `auto.commit.interval.ms`(기본 5초)마다 `poll()`로 가져온 최신 오프셋을 자동으로 커밋하는데,

    * 이때 컨슈머는 메시지의 실제 처리 성공 여부는 확인하지 않으므로 처리 중 장애가 발생하면 해당 메시지는 유실될 수 있다.

* `false` (수동 커밋): 대부분의 개발자는 직접 커밋 시점을 제어하여 신뢰성을 높이고자 한다.

    * **애플리케이션이 명시적으로 커밋하려 할 때만 오프셋이 커밋**되게 할 수 있다.

    * `commitSync()`나 `commitAsync()` 같은 API를 사용하여, 메시지 처리가 성공적으로 완료되었음을 보장한 후에만 오프셋을 커밋한다.

    * 이는 메시지 유실을 방지하지만, 처리 로직과 커밋 사이에 장애가 발생하면 중복이 발생할 수 있다.

## 실제 프로젝트에 적용하기: 쿠폰 시스템 Consumer 설정

지금까지 카프카 컨슈머의 여러 이론적인 설정과 개념에 대해 알아보았다.

이제 실제 스프링 부트(Spring Boot) 환경의 쿠폰 시스템 프로젝트에서 이러한 개념들이 어떻게 코드로 구현되었는지 `KafkaConsumerConfig` 클래스를 통해 종합적으로 살펴보자.

> KafkaConsumerConfig.java

```java

@Configuration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.consumer.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public ConsumerFactory<String, CouponIssueMessage> consumerFactory() {
        JsonDeserializer<CouponIssueMessage> deserializer = new JsonDeserializer<>(CouponIssueMessage.class);
        deserializer.setRemoveTypeHeaders(false);
        deserializer.addTrustedPackages("*");
        deserializer.setUseTypeMapperForKey(true);
  
        // 공통 필수 설정
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, deserializer);
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "group_1");
  
        // 1. 신뢰성 관련 설정
        // 1-1) 오프셋 수동 커밋 설정 -> 메시지 처리 완료 후 애플리케이션이 직접 커밋 시점을 제어하여, 메시지 유실을 방지.
        config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        // 1-2) 가장 최신 메시지, 즉 컨슈머가 실행된 이후에 들어오는 메시지부터 읽기 시작한다.
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
  
        // 2. 안정성 관련 설정
        // 2-1) 컨슈머가 브로커(그룹 코디네이터)에게 하트비트를 보내지 않고 버틸 수 있는 최대 시간 (기본값: 45,000 ms = 45초)
        config.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 30000); // 30초로 설정
        // 2-2) 컨슈머가 얼마자 자주 하트비트를 보낼지 결정하는 주기. (기본값: 3,000ms = 3초)
        config.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 10000); // 10초로 설정
        // 2-3) poll()로 가져온 메시지를 처리하는 데 허용된 최대 시간. (기본값: 300,000ms = 5분)
        config.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 600000); // 10분
        // 2-4) 리밸런싱 시 중단을 최소화하는 '협력적 리밸런스' 전략 사용.
        config.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG, "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
  
        // 3. 성능 관련 설정
        // 3-1) poll() 요청 시 브로커가 응답해야 할 최소 데이터 크기. (기본값: 1 byte)
        config.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, 2);
        // 3-2) poll() 요청 시 브로커에서 대기할 최대 시간. (기본값: 500ms)
        config.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, 1000);
        // 3-3) 컨슈머가 브로커로부터 한 번의 poll() 호출로 가져올 최대 메시지 개수. (기본값: 500)
        config.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 1000);
  
        return new DefaultKafkaConsumerFactory<>(config, new StringDeserializer(), deserializer);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, CouponIssueMessage> kafkaListenerContainerFactory(
            ConsumerFactory<String, CouponIssueMessage> consumerFactory,
            RecordInterceptor<String, CouponIssueMessage> mdcRecordInterceptor
    ) {
        ConcurrentKafkaListenerContainerFactory<String, CouponIssueMessage> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);

        // 신뢰성: AckMode를 MANUAL로 지정하여 수동 커밋 활성화
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);

        // 성능: 토픽의 파티션 수와 일치시켜 병렬 처리 성능을 최적화
        factory.setConcurrency(3);

        // 기타: 로깅 및 모니터링 설정
        factory.getContainerProperties().setObservationEnabled(true);
        factory.setRecordInterceptor(mdcRecordInterceptor);
        return factory;
    }
}
```

위 코드는 신뢰성, 안정성, 성능을 모두 고려하여 다음과 같이 설정했다.

> 신뢰성

메시지를 유실 없이, 최소 한 번 이상(At-least-once) 처리하는 것을 보장하기 위한 설정이다.

* `ENABLE_AUTO_COMMIT_CONFIG`를 `false`로, `AckMode`를 `MANUAL`로 설정했다.

    * 이 조합을 통해 카프카가 자동으로 오프셋을 커밋하는 것을 막고, `@KafkaListener`에서 비즈니스 로직이 완전히 성공했을 때만 `ack.acknowledge()`를 호출하여 직접 오프셋을 커밋할
      수 있다.

    * 이는 메시지 유실을 방지하는 확실한 방법이다.

* `AUTO_OFFSET_RESET_CONFIG`는 `latest`로 설정했다.

    * 이를 통해 새로운 컨슈머 그룹이 서비스를 시작하거나 오프셋 정보가 유실되었을 때,

    * 컨슈머가 실행된 이후에 들어오는 메시지부터 읽기 시작한다.

> 안정성

컨슈머가 예기치 않은 상황에서도 그룹에서 이탈하지 않고 꾸준히 작업을 수행하도록 보장하기 위한 설정이다.

* 컨슈머 생존 및 동작 감지를 위해 3가지 타임아웃을 명시적으로 설정했다.

    * `SESSION_TIMEOUT_MS_CONFIG`(30초)와 `HEARTBEAT_INTERVAL_MS_CONFIG`(10초)는 컨슈머 프로세스의 생존(Liveness)을,

    * `MAX_POLL_INTERVAL_MS_CONFIG`(10분)은 메시지 처리 로직의 동작(Progress)을 감시한다.

    * 이 값들을 넉넉하게 설정하여, 일시적인 부하나 긴 처리 시간으로 인해 컨슈머가 처리가 멈춘 '실패' 상태로 간주되어 컨슈머 그룹에서 쫓겨나는 상황을 방지한다.

* 또한, `PARTITION_ASSIGNMENT_STRATEGY_CONFIG`를 `CooperativeStickyAssignor`로 지정했다.

    * 이는 컨슈머 인스턴스가 추가되거나 제거될 때 발생하는 리밸런싱 과정에서 전체 컨슈머 그룹의 작업 중단('Stop-the-world')을 막고,

    * **변경이 필요한 파티션만 재할당**하여 서비스의 가용성을 극대화하는 좋은 선택이다.

> 성능

병렬 처리와 효율적인 데이터 가져오기라는 두 가지 전략을 통해 성능을 최적화하는 설정이다.

* 컨슈머의 처리량을 높이는 가장 확실한 방법은 병렬 처리를 활용하는 것이다. 코드에서는 `setConcurrency(3)` 설정을 통해 3개의 컨슈머 스레드가 각각 다른 파티션의 메시지를 동시에 처리하도록 했다.

    * 이는 마치 은행에 예금 상품 가입 신청서가 하나의 창구에만 쌓여 업무가 지연되자, 은행 지점장이 가입 전용 창구(컨슈머 스레드)를 3개로 늘리는 것과 같다.

    * 토픽의 파티션이 3개일 때: 카프카는 3개의 창구에 미리 분류해 둔 신청서 묶음(파티션)을 하나씩 1:1로 배정한다.

        * 1번 창구(텔러) → 1번 신청서 묶음 전담

        * 2번 창구(텔러) → 2번 신청서 묶음 전담

        * 3번 창구(텔러) → 3번 신청서 묶음 전담

    * 그 결과, 3개의 창구에서 동시에 가입 신청(메시지)이 처리되므로, 이론적으로 전체 처리 속도가 3배로 향상된다. 이처럼 최대 성능을 내기 위해서는 보통 `Concurrency` 값을 파티션 개수와
      동일하게 맞춘다.

* 효율적인 데이터 가져오기를 위해 `poll` 관련 설정을 튜닝했다.

    * `MAX_POLL_RECORDS_CONFIG`를 1000으로 높여 한 번에 대량의 메시지를 가져오도록 하고,

    * `FETCH_MAX_WAIT_MS_CONFIG`를 1000ms(1초)로, `FETCH_MIN_BYTES_CONFIG`를 2바이트로 설정하여 트래픽이 적을 때도 **최대 1초간 기다리며 최소 2바이트 이상의
      데이터를 최대한 모아** 한 번에 가져오도록 했다.

    * 이러한 설정은 약간의 지연 시간(Latency)을 감수하는 대신, 불필요한 네트워크 통신을 줄여 시스템의 전체적인 **처리량(Throughput)을 극대화**한다.

## References

* [카프카 핵심 가이드](https://product.kyobobook.co.kr/detail/S000201464167?utm_source=google&utm_medium=cpc&utm_campaign=googleSearch&gad_source=1)
