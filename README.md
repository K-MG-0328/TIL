# Today I Learned

매일 학습한 기술 개념을 정리한 노트 모음입니다. 주로 Java, Spring, 데이터베이스, 인프라, 운영체제·동시성 등 백엔드 개발 전반의 주제를 다룹니다.

## 작성 규칙

- 파일명: `YYMMDD-Topic.md` (예: `250514-webflux.md`)
- 형식: 스스로에게 묻고 답하는 Q&A 스타일을 기본으로 함
- 트러블슈팅 기록은 `Issue/` 디렉터리에 분리

## 목차

### Java 기본·언어 기능
- [Java 객체지향과 실행 과정](250114-Java.md) — Object, 바이트코드, OOP 원칙
- [Interface vs Abstract Class](250207-Interface-Abstract.md) — 다중 상속, default 메서드
- [Generic과 Type Erasure](250217-Generic.md) — 컴파일 시점 타입, PECS, Super Type Token
- [Optional 사용법](250126-Optional.md) — null 안전 처리
- [Enum과 DB 코드 비교](250208-Enum.md) — 상수 관리 전략
- [Exception 분류](250208-Exception.md) — Checked/Unchecked
- [String / StringBuilder / StringBuffer](250214-String-StringBuilder-StringBuffer.md)
- [String Constant Pool](250126-StringConstantPool.md)

### JVM·메모리·동시성
- [JVM 구조와 동작](250215-JVM.md) — Class Loader, Execution Engine, GC
- [Memory Leak과 GC Root](250222-MemoryLeak.md)
- [Atomic Operation과 CAS](250223-AtomicOperation.md) — volatile, ABA 문제
- [Deadlock](250411-Deadlock.md) — 4가지 발생 조건과 예방
- [Process와 Thread](250121-Process.md) — IPC, Context Switch
- [NIO와 Blocking I/O](250111-NIO-BlockingIO.md) — Channel, Buffer, Selector

### Spring·웹
- [WEB 요청 흐름](250114-WEB.md) — DNS, TCP Handshake, HTTP/HTTPS
- [Spring Security](250131-SpringSecurity.md) — 인증·인가, OAuth2.0, OIDC
- [WebFlux](250514-webflux.md) — Reactive Programming, Mono/Flux
- [JPA 연관관계](250420-JPA.md) — N+1, Lazy Loading, mappedBy

### 데이터·메시징
- [자료구조 정리](250113-DataStructure.md) — Array, LinkedList, Hash Table, BST
- [Redis](250127-Redis.md) — 캐싱 전략, RedisTemplate
- [Kafka](250415-kafka.md) — Topic, Partition, Consumer Group

### 분산 시스템·신뢰성
- [SAGA Pattern](250411-SAGA-Pattern.md) — Orchestration vs Choreography
- [Failover Strategy](250208-FailoverStrategy.md) — 결제 시스템 장애 대응
- [단일 서버 성능 개선](250317-단일서버성능개선.md)
- [Performance Testing](250130-Performance-Testing.md) — Throughput, TPS, Latency

### 인프라·도구
- [Docker와 Hypervisor](250208-Docker-Hypervisor.md) — 베어메탈/VM/컨테이너
- [EC2 초기 세팅](250426-EC2-Setting.md)
- [Git 명령어](250129-Git.md) — PR, revert, reset, rebase
- [Commit Message 컨벤션](250128-CommitMessage.md) — Conventional Commits

### 트러블슈팅 기록
- [부동소수점 정밀도 오류](Issue/250114-Floating-point-precision%20error.md)
- [LocalDateTime 직렬화 이슈](Issue/250128-Serialization-Issue.md)
- [nGrinder/Scouter 설치 이슈 (macOS)](Issue/250208-Installation-Issue.md)
