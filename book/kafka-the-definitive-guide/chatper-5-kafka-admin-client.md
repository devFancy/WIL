# [카프카 핵심 가이드] CHAPTER 5. 프로그램 내에서 코드로 카프카 관리하기: AdminClient를 활용한 토픽 및 컨슈머 그룹 관리


> 이 글은 [카프카 핵심 가이드](https://product.kyobobook.co.kr/detail/S000201464167?utm_source=google&utm_medium=cpc&utm_campaign=googleSearch&gad_source=1) 책을 읽고 정리한 글입니다.
>
> 이 글에서 다루는 모든 코드는 [깃허브](https://github.com/devFancy/springboot-coupon-system)에서 확인하실 수 있습니다.
>
> `참고`: 본 글에서 소개되는 코드 예시는 현재 시점의 구현을 바탕으로 작성되었으며, 프로젝트가 발전함에 따라 내용이 변경되거나 개선될 수 있음을 미리 알려드립니다.


아래는 "카프카 핵심 가이드" 책의 5장 내용을 바탕으로 정리한 글입니다.

## Prologue

이전에는 애플리케이션이 특정한 토픽에 이벤트를 써야 한다고 가정해보자.

그러면 이벤트를 쓰기 시작하기 전에 토픽에 대한 존재여부를 producer.send() 메서드에서 예외가 발생하는 걸 잡아서

사용자에게 토픽을 만들어야 한가도 알려주거나, 카프카 클러스터에 자동 토픽 생성 기능이 켜져 있기를 바라거나,

호환성 같은 것을 포기하고 내부 API를 사용하는 식이었다.

현재 카프카는 AdminClient를 제공한다. 이는 토픽 존재 여부를 확인해서 없으면 즉석에서 생성하는데 AdminClient 를 사용하면 되는 것이다.

이 장에서는 AdminClient 에 대해서 간략하게 살펴보고, 애플리케이션에서 주로 **토픽 관리, 컨슈머 그룹**과 같이 가장 많이 쓰이는 기능에 초점을 맞추자.
(나머지 기능은 실무에서 필요 시 해당 책을 참고하자)


---

## AdminClient 란

카프카의 AdminClient는 **비동기**로 작동한다. 각 메서드는 요청을 클러스터 컨트롤러로 전송한 뒤 1개 이상의 Future 개체를 리턴한다.

여기서 Future 객체는 비동기 작업의 결과를 가리키며, 카프카의 AdminClient는 Future 객체를 Result 객체 안에 감싸는데,

Result 객체는 작업이 끝날 때까지 대기하거나 작업 결과에 대해 일반적으로 뒤이어 쓰는 작업을 수행하는 헬퍼 메서드를 가지고 있다.

예를 들어, 카프카의 AdminClient.createTopics 메서드는 CreateTopicsResult 객체를 리턴하는데,

이 객체는 (1) 모든 토픽이 생성될 때까지 기다리거나, (2) 각각의 토픽 상태를 하나씩 확인하거나, 아니면 (3) 특정한 토픽이 생성된 뒤 해당 토픽의 설정을 가져올 수 있도록 도와준다.

AdminClient ApI가 리턴하는 Future 객체들은 카프카 컨트롤러의 상태가 완전히 업데이트된 시점에서 완료된 것으로 간주된다.

이러한 속성을 `최종적 일관성` (eventual consistency) 라고 하는데,

최종적으로 모든 브로커는 모든 토픽에 대해 알게되지만, 정확히 그게 언제가 될지에 대해서는 아무런 보장을 할 수 없다.

## AdminClient 사용법

AdminClient 를 사용하기 위해 다음과 같이 AdminClient 객체를 생성한다.

``` java
Properties props = new Properties();
props. put(AdminCLientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092"); 
AdminClient admin = AdminClient.create(props);
// TODO: Adminctient를 사용해서 필요한 작업을수행한다. 
admin.close(Duration.ofSeconds(30));
```

여기서 반드시 있어야하는 설정은 클러스터에 대한 URI(연결할 브로커 목록을 쉼표로 구분한 목록) 하나뿐이다.

프로덕션 환경에서는 브로커 중 하나가 장애가 발생할 경우를 대비해서 **최소 3개 이상**의 브로커를 지정하는 것이 보통이다.

(인증된 연결을 설정하는 방법에 대해서는 해당 책의 11장을 참고하자)

AdminClient 사용이 끝나면 마지막으로 close()를 호출하는데,

close()를 호출 할 때 타임아웃을 지정하면, 이는 아직 진행 중인 요청이 완료될 때까지 기다리는 최대 시간을 의미한다.

만약 타임아웃 없이 close()를 호출하면 모든 작업이 완료될 때까지 무기한 대기하게 됩니다.

아래는 AdminClient에 대한 매개변수인데, 프로듀서와 컨슈머에 비해 그리 많지 않지만 중요한 것 위주로 살펴보자.

### client.dns.lookup

이 설정은 부트스트랩 서버 설정된 호스트명을 기준으로 **연결을 검증하고 해석하고 생성**하는 것을 의미한다.

대부분의 경우 기본 설정으로 동작하나, 특별한 네트워크 환경에서는 아래 2가지 설정을 염두하면 좋다.

* 첫 번째인 DNS 별칭을 사용하는 경우에서는

    * 네이밍 컨벤션(e.g. broker1.hostname.com, broker2.hostname.com)을 따르는 브로커가 있으면,

    * 이 모든 브로커 전체를 가리킬 **하나의 DNS 별칭**(e.g. all-brokers.hostname.com)을 만들 수 있다.

    * 관리가 편하다는 장점이 있지만, SASL을 사용해서 인증을 하려고 할 때는 문제가 생긴다.

    * 클라이언트는 별칭인 all-brokers.hostname.com 으로 인증을 시도하지만, 실제 서버의 보안 주체는 broker1.hostname.com 과 같은 개별 호스트명이라 이름이 일치하지 않아 인증이 실패하게 된다.

    * 이때 `client.dns.lookup` 값을 **`resolve_canonical_bootstrap_servers_only` 로 설정**하면,

    * 클라이언트는 DNS 별칭을 실제 브로커들의 **정식 호스트명 목록으로 변환**하여 연결하므로 이 문제를 해결할 수 있다.

* 두 번째인 2개 이상의 IP 주소로 연결되는 하나의 DNS 항목을 사용하는 경우에서는

    * 쿠버네티스(Kubernetes) 환경처럼 로드 밸런서나 프록시 뒤에 브로커를 두는 것이 일반적이다.

    * 이때 고가용성을 위해 `broker1.hostname.com`와 같이 하나의 DNS 이름에 여러 개의 IP 주소로 매핑할 수 있다.

    * 기본적으로 카프카 클라이언트는 DNS 조회 시 반환된 **첫 번째 IP 주소로만 연결을 시도**한다.

    * 만약 해당 IP가 일시적으로 사용할 수 없는 상태라면, 다른 IP들이 정상임에도 불구하고 클라이언트는 브로커에 연결할 수 없게 된다.

    * 이러한 상황에서 `client.dns.lookup` 값을 **`use_all_dns_ips` 로 설정**하면,

    * 클라이언트는 **조회된 모든 IP 주소로 연결을 시도**하여 로드 밸런싱 계층의 고가용성을 최대한 활용할 수 있다.

### request.timeout.ms

이 설정은 AdminClient의 요청 하나가 **응답을 받기까지 기다리는 최대 시간**을 의미하며, 재시도에 걸리는 시간까지 포함한다.

기본값은 `120`초이지만, 이는 컨슈머 그룹 관리처럼 응답에 시간이 오래 걸릴 수 있는 작업을 고려한 것이다.

* 만약 서비스 시작 시 토픽 존재 여부를 확인하는 것처럼 애플리케이션의 핵심 경로에 AdminClient 작업이 포함된다면, 이 타임아웃 값을 적절히 낮추는 것을 고려해야 한다.

* 또한, 각 AdminClient 메서드를 호출할 때마다 `Options` 객체를 넘겨 **해당 메서드에만 적용되는 타임아웃을 별도로 지정**할 수도 있다.

* 예를 들어, 서비스 시작 시 30초 이상 응답이 없으면 일단 다음 작업을 계속 진행하고 나중에 토픽을 확인하도록 로직을 구성할 수 있다.

## 필수적인 토픽 관리 기능

위에서는 AdminClient를 생성하고 설정하는 방법을 살펴보았으니, 이제부터는 AdminClient를 가지고 무엇을 할 수 있는지 알아보고자 한다.

가장 흔한 활용 사례는 `토픽 관리`다.

여기에는 (1) 토픽 목록 조회, (2) 상세 내역 조회, (3) 생성 및 삭제가 있다.

### 토픽 목록 조회하기

클러스터에 있는 모든 토픽의 목록을 간단하게 조회할 수 있다.

``` java
// listTopics()로 모든 토픽 정보를 요청하고, names()로 이름만 가져온다.
ListTopicsResult topics = admin.listTopics();
topics.names().get().forEach(System.out::println);
```

위와 같이 Future 객체에서 get() 메서드를 호출하면, 실행 스레드는 서버가 토픽 이름 집합을 리턴할 때까지 기다리거나 아니면 타임아웃 예외를 발생시킨다.

응답을 받으면 각 토픽의 이름을 화면에 출력한다.

### 토픽 확인 후 없으면 생성하기

애플리케이션을 실행할 때, 특정 토픽이 (1) 존재하는지 확인하고,

(2) 존재 여부 이상의 정보가 필요할 때(해당 토픽이 필요한 만큼의 파티션과 레플리카를 가지고 있는지 확인해야 하는 것과 경우)가 있다.

``` java
try {
    // 1. 먼저 토픽 상세 정보를 요청해본다.
    DescribeTopicsResult result = admin.describeTopics(Collections.singletonList("my-topic"));
    TopicDescription description = result.all().get().get("my-topic");

    // 2. 토픽이 존재하면, 파티션 개수가 3개가 맞는지 확인한다.
    if (description.partitions().size() != 3) {
        System.out.println("토픽의 파티션 개수가 올바르지 않습니다!");
    }

} catch (ExecutionException e) {
    // 3. ExecutionException이 발생했을 때, 그 원인이 '토픽 없음' 때문인지 확인한다.
    if (e.getCause() instanceof UnknownTopicOrPartitionException) {
        System.out.println("토픽이 존재하지 않습니다. 새로 생성합니다.");
        
        // 4. 원하는 설정으로 새 토픽을 생성한다.
        NewTopic newTopic = new NewTopic("my-topic", 3, (short) 1);
        admin.createTopics(Collections.singletonList(newTopic));
    } else {
        // '토픽 없음' 외의 다른 예외일 경우, 에러를 출력한다.
        e.printStackTrace();
    }
}
```

### 토픽 삭제하기

토픽 삭제는 deleteTopics() 메서드로 간단하게 수행할 수 있다.

``` java
admin.deleteTopics(Collections.singletonList("my-topic")).all().get();
```

위와 같이 get() 메서드를 호출하여 삭제 작업이 완료될 때까지 기다린다.

> 주의사항

카프카에서 토픽 삭제는 **되돌릴 수 없는 작업**이다.

휴지통과 같은 기능이 없기 때문에, 실수로 중요한 토픽을 삭제하면 해당 데이터는 영구적으로 유실된다.

따라서 이 기능을 사용할 때는 꼭 삭제해야 하는지 여부를 반드시 확인해야할 필요가 있다.

### 정리

"필수적인 토픽 관리 기능" 챕터에서 살펴본 모든 예제는 `get()` 메서드를 사용했다.

이 방식은 카프카로부터 응답이 올 때까지 스레드를 잠시 멈추고 기다리는 **블로킹(Blocking)** 방식이다.

대부분의 관리 작업은 애플리케이션 시작 시 한 번 실행되거나 가끔 필요할 때만 사용되므로, 이처럼 간단하고 직관적인 방법으로 충분하다.

하지만, 책에서 언급하듯이 **많은 관리 요청을 동시에 처리해야 하는 서버**를 개발하는 경우에는 서버에서 매 요청마다 스레드를 블로킹하면 성능이 크게 저하될 수 있다.

이러한 경우에는 요청을 보내놓고 응답이 오면 처리하는 **비동기(Asynchronous) 콜백 방식**을 사용한다.

이를 통해 스레드를 낭비 없이 효율적으로 사용할 수 있다.

(이 글에서는 가장 필수적인 블로킹 방식에 초점을 맞추었으니, 비동기 처리에 대한 자세한 구현 방법이 궁금하시다면 책의 뒷부분을 참고하자.)

## 컨슈머 그룹 관리 기능

4장에서 컨슈머 API를 사용해서 처리 지점을 되돌려서 오래된 메시지를 다시 토픽으로 읽어오는 방법에 대해 알아봤다.

애플리케이션에 컨슈머 관련 기능을 사용하지 않고, 매세지를 재처리해야 하는 시나리오에는 여러 가지가 있다.

* (1) 사고가 발생한 와중에 오작동하는 애플리케이션을 트러블슈팅하는 경우

* (2) 재해 복구 상황에서 애플리케이션을 새로운 클러스터에서 작동시키려 하는 경우 (재해 복구에 대해서는 해당 책의 10장을 참고하자.)

여기서 AdminClient를 사용해서 프로그램적으로 컨슈머 그룹과 이 그룹들이 커밋한 오프셋을 조회하고 수정하는 방법에 대해 알아가고자 한다.

### 컨슈머 그룹 살펴보기

컨슈머 그룹을 살펴보고 변경하고 싶으면, 가장 먼저 해야할 일은 **컨슈머 그룹의 목록을 조회**하는 것이다.

``` java
// 클러스터가 에러 없이 리턴한 컨슈머 그룹의 목록만 가져온다.
admin.listConsumerGroups().valid().get().forEach(System.out::println);
```

주의할 점은 valid(), get() 메서드를 호출하면 문제가 없는 컨슈머 그룹 목록만 가져온다.

만약 권한 문제나 코디네이터 장애 등으로 조회가 실패한 그룹이 있더라도 예외가 발생하지 않는다.

이런 에러들은 errors() 메서드를 사용해서 따로 확인할 수 있다.

컨슈머 그룹 목록을 얻었다면, 이제 특정 그룹의 상세 정보를 확인할 차례인데,

여기서 특히 컨슈머 그룹이 읽고 있는 각 파티션에 대해 **마지막으로 커밋된 오프셋 값** 이 무엇인지,

그리고 **최신 메시지에서 얼마나 뒤떨어졌는지**(lag)를 아는 것이 중요하다.

AdminClient를 사용해서 커밋 정보를 얻어오는 방법은 아래와 같다.

``` java
// 1. 특정 컨슈머 그룹이 커밋한 오프셋 정보를 가져온다.
Map<TopicPartition, OffsetAndMetadata> offsets =
    admin.listConsumerGroupOffsets(CONSUMER_GROUP)
        .partitionsToOffsetAndMetadata().get();

// 2. 위에서 얻은 파티션 목록을 사용해, 각 파티션의 가장 마지막 오프셋을 요청한다.
Map<TopicPartition, OffsetSpec> requestLatestOffsets = new HashMap<>();
offsets.keySet().forEach(tp -> requestLatestOffsets.put(tp, OffsetSpec.latest()));

Map<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo> latestOffsets =
    admin.listOffsets(requestLatestOffsets).all().get();

// 3. (마지막 오프셋 - 커밋된 오프셋)으로 Lag을 계산하고 출력한다.
for (Map.Entry<TopicPartition, OffsetAndMetadata> e : offsets.entrySet()) {
    long committedOffset = e.getValue().offset();
    long latestOffset = latestOffsets.get(e.getKey()).offset();
    long lag = latestOffset - committedOffset;

    System.out.println("Group: " + CONSUMER_GROUP +
        ", Topic: " + e.getKey().topic() +
        ", Partition: " + e.getKey().partition() +
        ", Committed Offset: " + committedOffset +
        ", Latest Offset: " + latestOffset +
        ", Lag: " + lag);
}
```

* (1) 각각의 토픽 파티션에 대해 마지막으로 커밋된 오프셋을 밸류로 하는 맵을 가져온다. listConsumerGroupOffsets 는 컨슈머 그룹의 모음이 아닌 하나의 컨슈머 그룹을 받는 점을 유념하자.

* (2) 결과에 들어있는 각각의 토픽 파티션에 대해 마지막 메시지의 오프셋을 얻고자 한다.

* (3) 모든 파티션을 반복해서 각각의 파티션에 대해 마지막으로 커밋된 오프셋, 파티션의 마지막 오프셋, 둘 사이의 랙을 출력한다.

### 컨슈머 그룹 수정하기

AdminClient는 컨슈머 그룹을 수정하기 위한 메서드들 역시 가지고 있다. (그룹 삭제, 멤버 제외, 커밋된 오프셋 삭제 혹은 변경)

이러한 것들은 SRE 가 운영 중 비상 상황에서 임기 응변으로 복구를 위한 툴을 제작할 때 자주 사용된다.

이들 중에서도 **커밋된 오프셋을 변경**하는 기능이 가장 유용하다.

* 오프셋을 삭제하는 방법도 있지만, 이는 컨슈머의 auto.offset.reset 설정에 따라 동작이 달라진다. (earliest면 처음부터, latest면 마지막부터 읽기 시작하므로 예측이 어렵다.)

* 따라서 명시적으로 커밋된 오프셋을 **맨 앞으로 변경**하면 컨슈머는 토픽의 맨 앞에서부터 처리를 시작하게 된다.

* 즉, 컨슈머가 '리셋'되는 것이다.

> 오프셋 수정 시 알아야 할 점들

오프셋 토픽의 오프셋 값을 변경해도 컨슈머 그룹에 변경 여부가 전달되지 않는 것도 명심해야 한다.

컨슈머 그룹은 컨슈머가 **(1) 새로운 파티션을 할당받거나 (2) 새로 시작할 때만** 오프셋 토픽에 저장된 값을 읽어올 뿐이다.

카프카에서는 현재 작업이 돌아가고 있는 컨슈머 그룹에 대한 오프셋을 수정하는 것을 허용하지 않는다.

또한, 컨슈머 애플리케이션에서 오프셋을 리셋하고 해당 컨슈머 그룹이 토픽의 맨 처음부터 처리를 시작하도록 할 경우 저장된 상태가 깨질 수 있다는 점을 명심해야 한다.

이러한 이유로 인해 상태 저장소를 적절히 변경해 줄 필요가 있는 것이다.

## References

* [카프카 핵심 가이드](https://product.kyobobook.co.kr/detail/S000201464167?utm_source=google&utm_medium=cpc&utm_campaign=googleSearch&gad_source=1)
