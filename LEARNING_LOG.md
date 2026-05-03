# 학습 로그 — 2026-05-02 보충 작업 정리

이 문서는 TIL 프로젝트 전체를 4개 라운드로 나누어 보충한 작업 기록입니다. 각 파일별로 **무엇이 추가되었는지**와 **왜 그것이 학습할 가치가 있는지**를 정리했습니다. 핵심 개념·코드 예시·다이어그램이 빠져있던 부분을 채우는 데 초점을 맞췄으며, 사용자가 직접 작성한 본문은 그대로 보존했습니다.

학습 순서 추천: 각 파일에서 "추가된 핵심 개념"을 먼저 보고, 그 개념이 잘 이해되지 않으면 해당 파일의 새 섹션을 정독하는 식으로 활용하면 좋습니다.

---

## 라운드 1 — 가장 시급했던 7개 (인덱스/미완성/매우 짧음)

### `README.md`
- **이전**: "Today I Learned" 한 줄만 있음
- **추가**: 카테고리별 파일 인덱스 (Java / JVM·동시성 / Spring·웹 / 데이터·메시징 / 분산 시스템 / 인프라 / 트러블슈팅), 작성 컨벤션
- **학습 포인트**: 이 README 자체가 카테고리 분류 학습 자료. 어떤 주제들이 백엔드 학습에서 묶이는지 큰 그림을 잡을 때 활용

### `250514-webflux.md`
- **이전**: 37줄에서 미완성, "연산자란?", "핸들러와 라우터" 섹션이 비어있음
- **추가**:
  - MVC vs WebFlux 처리 모델 mermaid 다이어그램
  - 핵심 연산자(map, flatMap, filter, zip, onErrorResume 등) 표 + 코드
  - `map`과 `flatMap`의 차이를 직관적으로 보여주는 예시
  - RouterFunction + HandlerFunction 코드, Annotation 방식과 비교
  - WebFlux를 쓰면 안 되는 경우 (blocking 라이브러리)
- **학습 포인트**: `map`은 동기 변환, `flatMap`은 반환값이 또 다른 Mono/Flux일 때 평탄화. 이걸 헷갈리면 `Mono<Mono<T>>`가 만들어진다. **WebFlux는 만능이 아니다** — JDBC 같은 blocking을 쓰면 이벤트 루프가 막혀 오히려 망가진다는 점이 가장 중요

### `250208-FailoverStrategy.md`
- **이전**: 1KB, 3가지 전략을 단순 나열
- **추가**: mermaid 시퀀스 다이어그램, 3가지 전략 비교표, 멱등성·DLQ·Outbox 주의사항, SAGA 글 상호참조
- **학습 포인트**: 비동기 처리 시 **멱등성(Idempotency)** 보장이 핵심. 같은 메시지가 두 번 들어와도 한 번만 처리되도록 idempotency key를 검증해야 한다. 재시도는 항상 exponential backoff + DLQ 조합

### `250114-WEB.md`
- **이전**: 1.4KB 평문, 단계 구분 없음
- **추가**: 전체 흐름 시퀀스 다이어그램, 단계별 헤딩, HTTP/HTTPS 차이 + TLS Handshake 4단계, 상태 코드 분류 표
- **학습 포인트**: 4xx는 클라이언트 책임, 5xx는 서버 책임. 디버깅 시 어디부터 봐야 할지 결정하는 첫 분기점. TLS Handshake는 비대칭키로 대칭키를 안전하게 교환하는 과정

### `250420-JPA.md`
- **이전**: 1.8KB, 빈 섹션 ("각각의 경우에 발생할 문제점은?") 존재
- **추가**:
  - 1:N/1:1/N:M 각각의 설계 문제점과 권장 패턴 (N:M은 별도 엔티티로!)
  - `mappedBy` = "FK는 반대편이 관리한다", 양방향 편의 메서드 패턴
  - Lazy vs Eager 비교표 + **모든 연관관계 LAZY 권장**
  - **N+1 문제** 발생 원인과 3가지 해결책 (fetch join / @EntityGraph / @BatchSize) — fetch join은 페이징과 함께 쓸 수 없음
  - cascade와 orphanRemoval의 차이
- **학습 포인트**: 실무 JPA의 가장 흔한 함정이 N+1 문제. 로그를 켜고 SQL 실행 횟수를 직접 세어보지 않으면 ORM의 lazy loading이 자동으로 N개의 쿼리를 발생시켜도 알아채기 어려움. fetch join은 페이징 불가, 그래서 컬렉션엔 BatchSize도 자주 씀

### `250207-Interface-Abstract.md`
- **이전**: 1.8KB, 코드 예시 없음
- **추가**: 비교표(다중 상속, 필드, 생성자, default 메서드), 인터페이스 코드(Payable + default), 추상클래스 코드(HttpClient + 공통 필드), default 메서드 등장 배경(Collection.stream() 사례), 다이아몬드 문제 해결, `@FunctionalInterface`
- **학습 포인트**: default 메서드는 **기존 인터페이스에 메서드를 추가해도 구현체가 깨지지 않게 하기 위해** 등장했다(Collection.stream()이 대표 사례). 함수형 인터페이스는 추상 메서드 1개 + default 여러 개

### `250217-Generic.md`
- **이전**: 2.2KB, Type Erasure 정의는 있으나 PECS·실제 코드 없음
- **추가**:
  - Super Type Token 실제 구현 코드 (`getGenericSuperclass()` + `ParameterizedType`)
  - **PECS 원칙** (Producer Extends, Consumer Super) + 코드
  - 제네릭 메서드 vs 제네릭 클래스 작성법
  - 흔한 실수 4가지 (List<Object> vs List<String>, 제네릭 배열, instanceof, static)
- **학습 포인트**: **PECS = Producer Extends, Consumer Super**. 데이터를 꺼내 쓸 때는 `? extends T`(읽기), 데이터를 넣을 때는 `? super T`(쓰기). Spring의 `ParameterizedTypeReference`나 Jackson의 `TypeReference`가 모두 Super Type Token 패턴

---

## 라운드 2 — 중간 크기 핵심 글 8개

### `250126-Optional.md`
- **추가**: 안티패턴 표(`isPresent + get`, 필드 사용 등), `orElse` vs `orElseGet` 실행 시점 차이를 보여주는 코드, Stream + `Optional::stream` 패턴
- **학습 포인트**: `orElse(expensiveCall())`은 값이 있어도 항상 실행됨. 비용 큰 기본값은 무조건 `orElseGet(() -> ...)`. Stream에 `Optional`을 흘릴 때는 `flatMap(Optional::stream)`으로 빈 값을 자연스럽게 거른다

### `250126-StringConstantPool.md`
- **추가**: 메모리 다이어그램(Heap 안의 String Pool과 new String 객체 관계), `intern()` 메서드, Pool 위치의 버전별 변화(PermGen → Heap)
- **학습 포인트**: `new String("x")`는 항상 새 Heap 객체를 만들고, 그 내부 value 배열은 Pool의 리터럴을 공유(Java 7+). `intern()`은 Pool과 정렬해주지만 비용이 있어 일반 코드에서는 거의 사용하지 않는다

### `250208-Enum.md`
- **추가**: Enum의 컴파일 후 구조(Enum 상속 + public static final 인스턴스), 필드+추상 메서드 패턴(다형성), 인터페이스 구현, EnumMap/EnumSet
- **학습 포인트**: Enum이 자연스럽게 싱글턴인 이유는 컴파일 후 구조 때문. **if/else 분기를 Enum의 추상 메서드로 옮기면** 새 정책 추가 시 컴파일러가 누락된 구현을 알려줌. EnumSet은 비트 벡터, EnumMap은 배열로 구현되어 일반 Set/Map보다 훨씬 빠름

### `250111-NIO-BlockingIO.md`
- **추가**: Channel + Buffer 코드(`flip()` 의미 강조), Selector 사용 예시(non-blocking 서버), I/O 모델 비교표(Blocking / Non-blocking / Multiplexing / Async), Direct Buffer 주의사항
- **학습 포인트**: NIO는 보통 "Non-blocking"이라 부르지만 실제 의미는 **I/O Multiplexing**. 진짜 비동기는 NIO.2(`AsynchronousChannel`). `flip()`은 "여기까지 썼으니 이제 읽자"는 의미로 가장 자주 실수하는 부분

### `250127-Redis.md`
- **추가**: 자료구조별 시간복잡도 표(String/Hash/List/Set/ZSet/Bitmap/HyperLogLog/Stream), 캐싱 전략 4가지 비교표, Eviction Policy 6가지, Replication/Sentinel/Cluster 차이, Cache Stampede / Hot Key / Big Key 함정
- **학습 포인트**: Redis = 단순 KV가 아니라 자료구조 서버. 순위표는 Sorted Set, DAU 추적은 Bitmap, 유니크 카운트는 HyperLogLog. **Cache Stampede**는 TTL 만료 시점에 동시 요청이 모두 DB로 몰리는 현상 — 만료 직전 비동기 갱신이나 jitter로 완화

### `250223-AtomicOperation.md`
- **추가**: AtomicInteger의 CAS 루프 의사 구현, LongAdder(경합 분산), **ABA 문제**와 AtomicStampedReference 해결, volatile/synchronized/Lock/Atomic 비교표, JMM happens-before
- **학습 포인트**: CAS는 락 없이 빠르지만 **ABA 문제**가 있다. 단순 값 비교만 하므로 0→1→0 같은 변화를 못 잡음. 해결은 stamp(버전)을 함께 비교. 락의 진짜 가치는 happens-before 관계 설정 — 가시성과 순서 보장

### `250411-Deadlock.md`
- **추가**: 송금 코드의 데드락 해결 패턴(자원 획득 순서 통일), `jstack`/`SHOW ENGINE INNODB STATUS` 디버깅 명령, Wait-for Graph 다이어그램, 재현 코드, 락 타임아웃
- **학습 포인트**: 데드락 4 조건 중 가장 깨기 쉬운 게 **순환 대기**. 자원에 전역 순서를 부여하면 자동으로 사라진다(예: ID 작은 것부터 락). MySQL/Postgres는 자동 감지 후 한쪽 롤백 — 애플리케이션은 재시도 로직만 갖추면 됨

### `250222-MemoryLeak.md`
- **추가**: 5가지 누수 패턴 코드(static 컬렉션, ThreadLocal, 리스너, 익명 클래스의 외부 참조, 리소스 미반환), Heap Dump 명령(`jmap`, `-XX:+HeapDumpOnOutOfMemoryError`), GC 알고리즘 비교, OOM 메시지별 원인 매핑
- **학습 포인트**: ThreadLocal 누수는 톰캣처럼 스레드를 풀로 재사용하는 환경에서 가장 흔함 — `try-finally`로 항상 `remove()` 호출. 익명 비정적 내부 클래스는 외부 인스턴스 참조를 잡고 있어, 외부 객체가 무거우면 같이 못 죽음. 람다는 외부 변수를 캡처하지 않으면 안전

---

## 라운드 3 — 큰 글의 심화 보충 7개

### `250121-Process.md`
- **추가**: 프로세스/스레드 메모리 구조 mermaid, Context Switch 비용의 4가지 원천(레지스터·TLB·캐시·모드 전환), IPC 6가지 방식 비교표
- **학습 포인트**: 스레드 간 Context Switch가 프로세스 간보다 빠른 이유는 **TLB와 캐시를 비울 필요가 없기 때문**. IPC 중 Shared Memory가 가장 빠르지만 동기화 부담을 직접 짊어져야 함. Socket은 같은 호스트에서도 일관된 모델을 주는 대신 커널을 거치는 비용

### `250215-JVM.md`
- **추가**: GC 알고리즘 비교표(Serial/Parallel/CMS/G1/ZGC/Shenandoah)와 선택 가이드, 자주 쓰는 GC 옵션, JIT 최적화 기법(Inlining, Escape Analysis, Tiered Compilation), JVM Warm-up 대응, PermGen → Metaspace 변천
- **학습 포인트**: 일반 웹 서버 → G1 GC, 응답시간 민감 → ZGC. **튜닝 전 GC 로그부터 본다** — 일시정지 시간, Full GC 빈도, 회수 후 점유율을 확인하지 않고 옵션을 바꾸는 건 추측. JVM Warm-up이 끝나기 전엔 같은 코드도 수~수십 배 느릴 수 있어, 부하 분산기로 트래픽 받기 전에 워밍업 필요

### `250131-SpringSecurity.md`
- **추가**: 인증 흐름 mermaid, **CustomAuthenticationFilter 전체 코드**(Spring Security 6, `requireExplicitSave` 대응), CustomAuthProvider 코드, SecurityConfig 등록 코드, OAuth2 Grant Type 6종 비교표, 보안 모범 사례 체크리스트
- **학습 포인트**: Spring Security 6에서 `requireExplicitSave(true)`가 기본값이 되어 SecurityContext를 **명시적으로 저장**해야 인증이 유지됨. SPA는 Authorization Code + PKCE, 서버-서버는 Client Credentials. JWT를 쓸 땐 `none` 알고리즘 거부와 짧은 AccessToken + RefreshToken 회전이 필수

### `250113-DataStructure.md`
- **추가**: 자료구조별 시간복잡도 종합 표, AVL vs Red-Black 트리, **Java 표준 라이브러리 매핑 표**(ArrayList/LinkedList/HashMap/TreeMap 등), 메모리 캐시 지역성, 데이터 규모별 선택 가이드
- **학습 포인트**: 시간 복잡도가 같아도 **ArrayList가 LinkedList보다 빠른 경우가 많다** — 캐시 지역성 때문. TreeMap은 Red-Black Tree로 정렬 + 범위 쿼리(`subMap`), HashMap은 충돌 시 트리화. 동시성이 필요하면 `ConcurrentHashMap`, `CopyOnWriteArrayList`

### `250114-Java.md`
- **추가**: equals/hashCode 계약과 JPA 엔티티에서의 함정, Checked vs Unchecked 사용 가이드, Lambda·Stream 예시와 성능 주의, var(Java 10+) / record(Java 14+)
- **학습 포인트**: equals 재정의 시 hashCode도 함께. JPA 엔티티에서 모든 필드 기반 equals를 만들면 영속화 전후로 hashCode가 바뀌어 Set 컨테이너에서 사라지는 버그가 발생. record는 DTO/VO에 보일러플레이트 제거 — 단, JPA 엔티티는 가변성이 필요해 record로 못 만듦

### `250415-kafka.md`
- **추가**: 미완성이던 **Circuit Breaker / Timeout / Retry** 섹션 완성(Resilience4j 코드, OPEN/CLOSED/HALF_OPEN 상태), Consumer Lag 모니터링, Rebalancing의 STW 문제와 Cooperative/Static Membership, At-Least-Once vs Exactly-Once + 멱등 처리 코드
- **학습 포인트**: Kafka는 기본이 At-Least-Once → 메시지 중복 가능 → **컨슈머가 멱등하게 처리**해야 한다. Exactly-Once는 비용이 커서 대부분 시스템은 At-Least-Once + 멱등 처리로 해결. Rebalancing은 STW를 유발하므로 Cooperative + Static Membership 조합 권장

### `250317-단일서버성능개선.md`
- **추가**: 모니터링 도구·명령어 정리표(top/iostat/jstat/jstack), DB 인덱스 튜닝 핵심 + EXPLAIN 사용법, Caffeine 로컬 캐시 코드(@Cacheable/@CacheEvict), Scale-Up vs Scale-Out 비교, 성능 개선 함정 5가지
- **학습 포인트**: 성능 개선 우선순위는 **코드/쿼리/인덱스 → 캐시 → Scale-Up → Scale-Out**. 스케일링은 마지막 카드. 단일 서버에서는 Redis보다 로컬 캐시(Caffeine)가 더 빠르고 단순. iowait 높음 → 디스크, user 높음 → CPU, context switch 폭증 → 락 경합

---

## 라운드 4 — 소규모/이슈 노트 9개

### `Issue/250114-Floating-point-precision error.md`
- **추가**: IEEE 754 표현 원리(64bit 부호+지수+가수), BigDecimal 주의사항(double 생성자 금지, 나눗셈 scale, equals/compareTo), 성능 비교표, 정수 변환 패턴, epsilon 비교
- **학습 포인트**: **돈 계산은 BigDecimal 또는 정수**. `new BigDecimal(0.1)`은 이미 오차 포함이므로 반드시 문자열 생성자. `new BigDecimal("1.0").equals(new BigDecimal("1.00"))`은 false — 값 비교는 `compareTo() == 0`. 한국 원은 long 정수로 충분

### ⚠️ 파일명에 백스페이스 컨트롤 문자가 들어간 두 파일 — 사용자 정리 필요
원본 저장소에 다음 두 파일이 **백스페이스 컨트롤 문자(0x08)를 포함한 깨진 파일명**으로 저장되어 있었습니다. ls 출력에서는 정상처럼 보이지만 실제 파일명은 깨진 상태이고, 일반 도구로는 접근이 안 됩니다.

| git에 추적된 깨진 이름 | 실제 의도된 이름 |
|---|---|
| `250128-\bCommitMessage.md` | `250128-CommitMessage.md` |
| `Issue/250128-\bSerialization-Issue.md` | `Issue/250128-Serialization-Issue.md` |

**처리 방식**: 정상 파일명으로 새 파일을 만들고 (보충된 내용 포함), 깨진 원본은 그대로 두었습니다. `git status`에서 새 파일은 untracked로, 깨진 파일은 modified가 아닌 추적된 채로 보입니다.

**사용자가 해야 할 일**:
```bash
# zsh의 $'...' 표기로 백스페이스 표현
git rm $'250128-\bCommitMessage.md'
git rm $'Issue/250128-\bSerialization-Issue.md'

# 새로 만든 정상 파일 추가
git add 250128-CommitMessage.md Issue/250128-Serialization-Issue.md
```

또는 Finder에서 직접 깨진 쪽을 식별해 삭제하는 것도 가능합니다 (보통 작은 파일 크기로 구분). 만약 이 작업이 까다롭다면, 두 파일이 공존하는 상태로 두어도 동작에는 문제가 없습니다 — 정상 파일명 쪽이 보충된 최신 내용입니다.

### `Issue/250128-Serialization-Issue.md` (신규 작성)
- **추가**: 보존된 원본 + JavaTimeModule 등록 패턴(필드 어노테이션 대신 ObjectMapper 한 번 설정), Java Serializable vs JSON 비교, **Serializable 보안 위험**, serialVersionUID, 순환 참조 처리(@JsonManagedReference/@JsonBackReference), 시간 타입 직렬화 결과
- **학습 포인트**: Java 기본 Serializable은 **역직렬화 RCE 취약점**으로 외부 입력에는 거의 사용 금지. JPA 양방향 엔티티를 그대로 직렬화하면 무한 루프 — DTO로 분리하는 게 가장 안전한 해법

### `Issue/250208-Installation-Issue.md`
- **추가**: SDKMAN!/java_home + alias로 다중 JDK 관리, "손상됨" 에러 진단 5단계, xattr 사용 시 보안 주의, nGrinder Java 17 호환(`--add-opens`), 진단 체크리스트 명령어
- **학습 포인트**: macOS Gatekeeper 격리 태그(`com.apple.quarantine`)는 **신뢰 가능한 소스에서만** 제거. 다중 JDK는 SDKMAN!이 가장 깔끔. 포트 충돌·Apple Silicon 호환은 의외로 자주 부딪힘

### `250208-Docker-Hypervisor.md`
- **추가**: 베어메탈/VM/컨테이너 구조 mermaid, 성능·비용 비교표(부팅 시간, 메모리, CPU 오버헤드), 사용 시점 가이드, 이미지 vs 컨테이너 구분, **컨테이너 보안의 한계**(커널 공유) + gVisor/Kata
- **학습 포인트**: 컨테이너는 호스트 커널을 공유하므로 **커널 취약점이 곧 격리 우회**. 강한 격리가 필요하면 rootless container, gVisor, Kata Containers. 실무에서는 보통 VM 위에 컨테이너 (EC2 + Docker)

### `250426-EC2-Setting.md`
- **추가**: 보안 강화 체크리스트(SSH 비밀번호 차단, 자동 보안 업데이트, fail2ban), 설정 검증 명령어, 버전 선택 이유, 다중 인스턴스 확장 시 고려사항(무상태화, 시크릿 관리), 트러블슈팅 표
- **학습 포인트**: SSH 설정 변경 시 **새 세션 열어 검증 전까지 기존 세션 닫지 말 것** — 잠기면 EC2 인스턴스를 종료하고 재생성해야 할 수도 있음. 시크릿은 절대 코드에 박지 말고 AWS Secrets Manager / Parameter Store

### `250130-Performance-Testing.md`
- **추가**: 부하 테스트 시나리오 5종(Load/Stress/Spike/Soak/Capacity), 결과 해석 핵심 지표(p95/p99의 중요성), 도구 비교표(nGrinder/JMeter/k6/Locust/Gatling/APM 4종), 스테이징 환경 우선순위, 도구별 의사결정
- **학습 포인트**: **평균 응답시간만 보면 안 된다** — long-tail은 p95/p99에서만 보임. 부하 도구(생성)와 APM(모니터링)은 역할이 달라 **함께** 사용. 데이터 규모는 인덱스 효율성에 매우 민감 — 작은 데이터로 측정한 결과를 단순 곱셈으로 추정 금지

### `250208-Exception.md`
- **추가**: Throwable 계층 구조, Custom Exception 작성 패턴(RuntimeException 상속), try-with-resources, 예외 처리 모범 사례, `@RestControllerAdvice` 글로벌 처리, 멀티스레드에서의 예외 전파(`Future.get()`)
- **학습 포인트**: 도메인 예외는 보통 **RuntimeException 상속** — Checked로 만들면 호출 체인이 throws 선언으로 오염됨. 빈 catch 블록 금지, 흐름 제어에 예외 사용 금지. 새 스레드의 예외는 호출 스레드로 전파되지 않음 — Future.get()이나 UncaughtExceptionHandler 필요

### `250129-Git.md`
- **추가**: Merge vs Rebase 비교, conflict 해결 흐름, stash/cherry-pick/reflog, **reset vs revert vs restore 비교표**, tracking branch와 HEAD/origin/upstream 차이, 흔한 실수 복구
- **학습 포인트**: **이미 푸시된 브랜치는 rebase 금지** — 다른 사람의 히스토리가 어긋남. 푸시된 커밋 되돌리기는 reset이 아니라 **revert**. `--force` 대신 `--force-with-lease`가 안전. reflog는 잃어버린 커밋의 비상구

### `250214-String-StringBuilder-StringBuffer.md`
- **추가**: 한눈 비교표, 반복문에서의 차이를 보여주는 코드, 단일 라인은 컴파일러가 자동 변환한다는 사실, 내부 구현(AbstractStringBuilder), String 불변성의 4가지 이점, 사용 시점 가이드, FAQ 2개
- **학습 포인트**: 단일 라인 `"a" + "b" + i`는 컴파일러가 알아서 StringBuilder로 변환. **반복문 안에서 누적할 때만** 명시적 StringBuilder가 필요. StringBuffer는 사실상 거의 사용 안 함 — 메서드 단위 동기화는 효과가 제한적이고 묶음 원자성도 못 줌

### `250128-CommitMessage.md` (정상 파일명 신규 작성)
- **상태**: 위의 ⚠️ 표시 참조. 깨진 파일명의 원본을 보존하고 정상 파일명으로 새 파일 작성. README에도 인덱스 추가
- **추가**: 원본 type 가이드 + Conventional Commits 형식, Tim Pope의 7가지 규칙, Type 선택 가이드 표, Subject Bad → Good 사례, Body 작성 패턴, Breaking Change 표기, 한국어 작성 패턴, 한 커밋 한 가지 일 원칙, 커밋 전 체크리스트, 자동화 도구(commitlint, commitizen 등)
- **학습 포인트**: 좋은 커밋 메시지의 핵심은 **"무엇이 아니라 왜"**. 코드 diff가 What을 보여주므로 메시지는 Why에 집중. 한 커밋엔 한 가지 일만 — 리뷰가 쉬워지고 revert가 안전. Breaking Change는 `!` 또는 footer로 명시

---

## 마무리: 전체에서 반복적으로 등장한 핵심 패턴

이 보충 작업을 하면서 반복적으로 등장한 패턴이 몇 가지 있습니다. 학습 시 이 줄기를 따라가면 흩어진 주제들이 연결됩니다.

1. **"평균은 거짓말한다"** — 성능 테스트는 p95/p99, GC는 평균이 아닌 일시정지 시간, 캐시는 평균 hit rate가 아닌 stampede 시점

2. **"멱등성(Idempotency)이 분산 시스템의 안전벨트"** — Kafka At-Least-Once, 메시지 큐 재시도, HTTP 재시도 모두 멱등 처리가 필수

3. **"기본값을 항상 의심"** — JPA의 EAGER loading, Spring Security 6의 `requireExplicitSave`, JVM의 GC 알고리즘은 버전마다 기본값이 바뀜

4. **"happens-before가 동기화의 진짜 정체"** — synchronized/volatile/Atomic이 락 자체가 아니라 메모리 가시성·순서를 보장하는 도구

5. **"측정 없는 최적화는 추측"** — JIT warm-up, GC 로그, EXPLAIN, APM. 가설 → 측정 → 개선의 사이클을 깨면 안 됨

6. **"인터페이스 vs 구현체"의 분리는 모든 곳에 등장** — Spring Security의 Filter→Manager→Provider, JPA의 영속성 컨텍스트, Stream의 연산자 체인, Reactor의 Publisher

---

## 작업 영향 요약

- **수정된 파일**: 31개
- **신규 작성**: 1개 (LEARNING_LOG.md), 1개 (Serialization-Issue.md 정상 파일명 버전)
- **수정 안 된 파일**: 0개 (모든 md 파일이 보충됨)
- **남은 정리 필요**: 깨진 파일명의 `Issue/250128-\bSerialization-Issue.md`를 사용자가 수동 삭제

> 학습할 때는 한 라운드씩 읽기보다, 관심 있는 주제를 골라 README의 카테고리에서 진입한 뒤 이 LEARNING_LOG의 해당 섹션을 함께 보는 것을 권장합니다.
