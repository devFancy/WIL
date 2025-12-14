# Sentry 알아보기

## 왜 Sentry를 사용하는가?

애플리케이션 모니터링 도구는 많지만, 그중 Sentry를 선택한 이유는 단순한 에러 수집을 넘어, **코드 레벨의 관측 가능성**을 제공하기 때문이다.

[공식문서](https://docs.sentry.io/product/)에 따르면, Sentry는 개발자가 성능 문제와 에러를 식별하고 디버깅하는 데 도움을 주는 소프트웨어 모니터링 도구로, 주요 특징은 다음과 같다.

> 특징 1. 에러 모니터링과 그룹화 (Error Monitoring)

Sentry는 처리되지 않는 예외를 자동으로 포착한다.
단순히 로그를 쌓는 것이 아니라, 유사한 에러를 하나의 이슈(Issue)로 그룹화하여 관리 효율성을 높인다.
또한, 소스 코드와 연동되어 문제가 발생한 코드 라인을 정확히 알려주고, `Suspect Commits` 기능을 통해 해당 에러를 유발했을 가능성이 높은 커밋을 찾아주기도 한다.

> 특징 2. 추적 (Tracing)

마이크로서비스(MSA) 환경이나 프론트엔드와 백엔드가 분리된 환경에서는 단일 시스템의 로그만으로 문제의 원인을 찾기 어렵다.
Sentry의 `Distributed Tracing` 기능을 활용하면 프론트엔드에서 백엔드, 그리고 데이터베이스까지 이어지는 요청의 전체 흐름을 시각화하여 병목 구간이나 에러 발생 지점을 신속하게 파악할 수 있다.

(Distributed Tracing에 대한 자세한 내용은 공식문서에 나와있는 [Distributed Tracing](https://docs.sentry.io/concepts/key-terms/tracing/distributed-tracing/) 글을 참고해 주시기 바란다.)

> 특징 3. 성능 모니터링 및 프로파일링 (Performance & Profiling)

에러뿐만 아니라 애플리케이션의 성능 지표도 관리할 수 있다.
처리량(Throughput), 지연 시간(Latency) 등의 메트릭을 추적하며, 프로파일링(Profiling)을 통해 실제 운영 환경에서 어떤 함수가 실행 속도를 저하시키는지 코드 레벨에서 분석할 수 있다.

> 특징 4. 개발 워크플로우 통합 (Seamless Developer Workflow)

Slack, Jira, GitHub 등 다양한 협업 도구와 연동할 수 있다.
에러 발생 시 즉시 알림을 받고, 이슈 트래커에 티켓을 생성하거나 담당자를 지정하는 등의 워크플로우를 자동화하여 대응 속도를 높일 수 있다.


## Reference

* [[공식문서] Sentry](https://docs.sentry.io/product/)

* [[공식문서] Sentry - Spring Boot](https://docs.sentry.io/platforms/java/guides/spring-boot/)

* [[공식문서] Sentry - Filtering](https://docs.sentry.io/platforms/java/guides/spring-boot/configuration/filtering/)