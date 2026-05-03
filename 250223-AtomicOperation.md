## 원자적 연산이란?
컴퓨터 과학에서 사용하는 원자적 연산(AtomicOperation)의 의미는 해당 연산이 더 이상 나눌 수 없는 단위로 수행된다는 것을 의미합니다. 즉, 원자적 연산은 중단되지 않고, 다른 연산과 간섭 없이 완전히 실행되거나 전혀 실행되지 않는 성질을 가지고 있습니다.  
멀티스레드 상황에서 다른 스레드의 간섭 없이 안전하게 처리되는 연산입니다.  

예를 들어 
int i = 1 다음 연산은 둘로 쪼갤 수 없는 원자적 연산이다. 
int i = 1; 연산 중에, 다른 스레드가 i를 읽으면 “0에서 1로 바뀌어가는 중간 단계”를 보지 않고, 오직 0(바뀌기 전 값) 또는 1(바뀐 후 값) 만 보게 됩니다.
자바 메모리 모델에서 int는 단일 32비트로 읽고 쓰는 것이 보장됩니다. 즉, CPU 차원에서 한 번의 쓰기 명령으로 처리되는 것이 일반적이므로, 중간 상태(절반만 쓰임)를 볼 일이 없습니다.

## long / double 처럼 64bit(8byte) 자료형에 대입은 원자적 연산이 아닌 것인가?
만약 cpu나 JVM이 32bit 환경이라면 64bit 자료형인 long / double은 32bit 단위로 쪼개져 두 번의 연산으로 이뤄질 수 있으므로 중간 값(일부만 쓰인 값)이 관측될 수 있고 cpu나 JVM이 64bit 환경이라면 long / double은 한 번의 연산으로 처리가 될 것이므로 원자적 연산으로 처리될 것입니다.  
하지만 자바에서는 64비트 연산은 무조건 원자적이다라고 보장하지 않으므로 volatile, AtomicLong, synchronized 같은 동기화 수단을 이용해야합니다.   

## volatile은 무엇인가?
volatile는 자바에서 변수 선언부에 붙일 수 있는 키워드로, 멀티스레드 환경에서 가시성(visibility), 재배치 방지(Ordering)를 보장합니다. 

    private volatile int days;

### 가시성(visibility)을 보장한다는 것을 무엇을 말하는걸까?
가시성이라는 것은 "한 스레드가 변수의 값을 수정했을 때 다른 스레드가 그 변경된 값을 언제 볼 수 있는가?" 에 대한 문제입니다.   
멀티스레드 환경에서 원자적이지 않은 연산의 경우에는 한 스레드에서 쓰기 작업을 하고 있을 때 동시에 다른 스레드가 읽기 작업을 하게되면 중간 상태 값이 보이게되고 엉뚱한 값이 표출되게 될 수 있습니다.
예를 들어 32bit 환경에서 64비트 자료형을 대입한다고하면 cpu는 64비트 자료형에 대해서 한번에 처리할 수 없으므로 2번의 연산을 처리할 것입니다. 그 과정에서 하위 32비트 상위 32비트 씩 값을 변경한다고 했을 때 하위 32비트 값만 변경되었을 때 다른 스레드에게 그 값이 표출될 수 있습니다.
가시성을 보장한다는 말은 한 스레드가 volatile 키워드를 사용한 변수에 쓰기 작업을 한다고 했을 때 중간 상태 값을 CAS 기법을 이용해서 읽기 작업을 하는 스레드가 중간 상태 값이 아닌 이전 값 또는 완전히 변경된 값을 보여주게됩니다. 
즉, 스레드가 완전한 쓰기 작업을 끝내기 전까지의 가장 최신의 값(이전 값) 그리고 쓰기 작업을 끝낸 후에 가장 최신 값(이후 값)을 보도록합니다.  

### 재배치 방지는 무엇을 말하는걸까?
컴파일러 또는 CPU 차원에서 성능 최적화를 위해 프로그램의 일부 명령들을 원래 소스 코드 순서와 다르게 실행하거나 메모리에 기록하는 순서를 바꾸는 것을 말합니다.   
단일 스레드 관점에서는 이러한 재배치가 프로그램의 최종 결과가 같다면 문제되지 않습니다. 그러나 멀티스레드 환경에서는 다른 스레드가 해당 변수(메모리)를 읽는 시점이 달라져 "내가 예상한 순서대로 값이 관측되지 않는다"는 문제가 생길 수 있습니다.
단순하게 예를 들어 

    x = 1;   // (1)
    y = 2;   // (2)

라는 코드가 존재할 때 컴파일러나 CPU가 (2)번 문장을 먼저 실행 → 메모리에 기록한 뒤 그 다음 (1)번 문장을 실행할 수 있습니다.  
단일 스레드라면 결과는 같을 수 있지만, 다른 스레드가 x와 y를 동시에 읽고 있다면 “y가 이미 2가 되었는데 x는 아직 0” 같은 상황을 보게 되어, 프로그래머 예상(“x를 1로 먼저 바꾼 다음 y를 2로 바꿨으니, y가 2라면 x도 1이겠지”)을 깨버립니다.  
volatile로 선언된 변수는 volatile 키워드로 선언한 변수 기준으로 읽기/쓰기에 대해서 volatile 변수의 이전의 모든 메모리 연산은 volatile 변수보다 앞서 완료되어야하고 이후의 모든 메모리 연산은 volatile 변수 이후의 수행되어야합니다.   
재배치 방지(Ordering guarantee) 는 멀티스레드 환경에서 정해진 순서대로 명령어가 실행되었다고 ‘관측’되도록 보장하는 것을 의미합니다.  


## CAS(Compare-And-Set)은 무엇인가?
Compare-And-Set(CAS)은 멀티스레드 환경에서 원자적으로(atomic) 값을 갱신하기 위한 기법입니다. 주어진 ‘기대 값(expected value)’과 메모리에 저장된 ‘현재 값(current value)’을 비교하여, 두 값이 일치하면 새 값(new value)으로 바꾸고 성공을 반환하며, 불일치하면 아무 변경 없이 실패를 반환합니다. 이 과정을 단 한 번의 불가분한(atomic) 연산으로 보장하기 때문에, 락(lock) 없이도 안전한 동시성 제어(특히 카운터 증가, 연결 리스트 수정 등)에 널리 사용됩니다. 

## AtomicInteger는 어떻게 동작하나요?
`java.util.concurrent.atomic` 패키지의 클래스들은 내부적으로 CAS를 루프로 감싸 실패하면 재시도하는 방식(spin)으로 동작합니다.

```java
AtomicInteger counter = new AtomicInteger(0);

// incrementAndGet의 의사 구현
int expected, next;
do {
    expected = counter.get();
    next = expected + 1;
} while (!counter.compareAndSet(expected, next));
```

* `synchronized`와 달리 OS 스레드 블록·해제가 없으므로 경합이 적을 때 훨씬 빠릅니다.
* 경합이 매우 심하면 spin 비용이 커질 수 있어 Java 8+에서는 `LongAdder`/`DoubleAdder`처럼 셀을 분산해 경합을 줄이는 클래스도 제공됩니다.

```java
LongAdder adder = new LongAdder();
adder.increment();         // 카운터 증가만 필요할 때 AtomicLong보다 빠름
long total = adder.sum();  // 합산 시점에 모든 셀을 합쳐 반환
```

## CAS의 ABA 문제란?
스레드 A가 값을 읽고(0) 비교 갱신을 준비하는 사이에, 다른 스레드가 0 → 1 → 0으로 바꿔 놓으면, A는 여전히 "0"이라 보고 갱신을 성공시킵니다. 단순 값만 비교하기 때문에 **중간에 값이 바뀌었다 원래대로 돌아온 상황**을 감지하지 못하는 게 ABA 문제입니다.

연결 리스트의 노드 포인터처럼 값이 같아도 객체의 정체가 다르면 큰 문제가 됩니다.

해결책으로 **버전(스탬프)** 을 함께 비교합니다.

```java
AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);
int[] stampHolder = new int[1];
String value = ref.get(stampHolder);
boolean updated = ref.compareAndSet(value, "B", stampHolder[0], stampHolder[0] + 1);
```

값이 같더라도 stamp가 바뀌었다면 CAS는 실패합니다. `AtomicMarkableReference`도 비슷한 용도(boolean mark)로 사용됩니다.

## volatile, synchronized, Lock, Atomic의 비교
| 메커니즘 | 가시성 | 원자성 | 재배치 방지 | 비용 |
|---|---|---|---|---|
| `volatile` | O | X (단일 read/write만) | O | 가장 저렴 |
| `synchronized` | O | O | O | 경합 시 OS 블록 |
| `ReentrantLock` | O | O | O | synchronized와 유사 + 추가 기능(tryLock, fairness) |
| `Atomic*` (CAS) | O | O | O | 경합 적을 때 매우 빠름, 심하면 spin 부담 |

* 단순 가시성만 필요하면 `volatile` (예: 종료 플래그)
* 복합 연산(읽고-수정하고-쓰기)이 필요하면 `Atomic*` 또는 락
* 여러 변수를 함께 보호해야 한다면 `synchronized`/`Lock` (CAS는 단일 변수만 가능)

## JMM과 happens-before 관계
Java Memory Model(JMM)은 "어떤 쓰기 작업이 어떤 읽기 작업에 보이는지"를 happens-before 관계로 정의합니다. 대표적인 happens-before:
- 같은 스레드 내의 프로그램 순서
- `volatile` 쓰기 → 이후의 같은 변수 읽기
- `synchronized` 블록 종료 → 같은 락 획득
- `Thread.start()` → 그 스레드 내부 첫 동작
- 그 스레드의 마지막 동작 → 다른 스레드의 `join()` 반환

이 규칙을 만족하지 않는 두 작업은 메모리에서 어느 순서로 보이는지 보장되지 않습니다. 동기화 도구를 쓰는 진짜 이유는 락 자체가 아니라 happens-before 관계를 설정해 가시성과 순서를 보장하는 것입니다.