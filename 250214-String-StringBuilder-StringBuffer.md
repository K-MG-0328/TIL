### String이란
String은 불변 객체이면서, 한 번 생성된 String 객체는 내용을 바꿀 수 없습니다. 문자열 연산이 자주 일어나면, 매번 새로운 객체를 생성하므로 메모리 사용량이 증가하고 성능에 영향을 줄 수 있습니다.  
Thread-safe 여부와는 상관없이, String 자체가 불변이므로 여러 스레드에서 동시에 읽기만 할 때는 안전하게 사용할 수 있습니다.   

### String 객체는 변경이 가능한데 왜 불변인가?
String 클래스는 내부 배열 value 필드 등을 final로 두고 있습니다. 그 배열을 직접 수정할 수 있는 메서드를 제공하지 않습니다.   
만약 str= str + " hello" 와 같은 연산을 수행하면, 기존 str 객체가 바뀌는게 아니라 새로운 String 객체를 만들어 그 참조를 str에 할당하는 방식으로 동작합니다.   

### 그렇다면 String 객체는 왜 Thread-safe 한 이유는?
String 객체가 Thread-safe하다는 뜻은 멀티스레드 환경에서 각 스레드가  String 객체를 변경을 했을 때 해당 스레드에서 변경된 String는 원본 객체를 수정한게 아니라 새로운 객체를 할당해서 사용한 것이기 때문에 멀티스레드 환경에서 안전합니다.  

### StringBuilder란
StringBuilder는 가변 객체 내부의 버퍼를 사용해 문자열을 조합하고 수정할 수 있습니다. 동기화 미지원 멀티스레드 환경에서 수정하려고 하면 안전하지 않습니다.  그러기 때문에 단일 스레드에서 문자열을 자주 수정하는 상황이라면 StringBuilder를 선택해야합니다.  

### StringBuilder는 왜 멀티스레드 환경에서 사용할 수 없는가?
StringBuilder는 내부의 버퍼를 사용해 문자열을 조합하고 수정할 수 있습니다. 예를 들어 각 스레드에서 StringBuilder의 객체를 동시에 수정했을 때 서로 다른 스레드가 하나의 StringBuilder의 버퍼를 동시에 수정하게 되므로  때문에 멀티스레드 환경에서는 안전하지않습니다. 객체는 힙메모리 영역에 저장되고 각 스레드는 힙메모리의 객체를 공유하므로 멀티스레드 환경에서 안전하지 않습니다. * 멀티 스레드 환경에서 안전하지 않다는 말은 서로 다른 스레드가 같은 인스턴스를 동시에 수정할 가능성이 있는지에 대한 것입니다.

### StringBuffer는 StringBuilder와 같은 가변 객체입니다. 
다만 동기화 지원으로 Thread-safe 합니다. 멀티스레드 환경에서 여러 스레드가 동시에 문자열을 수정해야 할 경우 StringBuffer를 사용합니다. 동기화를 하므로 StringBuilder보다 상대적으로 성능이 떨어질 수 있습니다.   

### 한눈에 비교
| 구분 | String | StringBuilder | StringBuffer |
|---|---|---|---|
| 가변 여부 | 불변 | 가변 | 가변 |
| 동기화 | 불변이라 자연스럽게 안전 | 미지원 | `synchronized` 메서드 |
| 멀티스레드 안전 | O | X | O |
| 속도 | 단발성 OK, 누적 시 느림 | 가장 빠름 | StringBuilder보다 약간 느림 |
| 등장 | Java 1.0 | Java 5 | Java 1.0 |
| 권장 사용 | 짧은 리터럴, 변경 없는 문자열 | 단일 스레드 누적 변경 | 거의 사용 X (lock 분리 가능한 다른 방식 선호) |

### 코드 예시로 보는 차이
```java
// String: 매 연산마다 새 객체 → 반복문에서 비효율
String s = "";
for (int i = 0; i < 10000; i++) {
    s += i;  // 매번 새 String 생성
}

// StringBuilder: 내부 버퍼 재사용
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

> 참고: 단일 라인 `"a" + "b" + i`는 컴파일러가 내부적으로 `StringBuilder`로 변환합니다(또는 Java 9+의 `invokedynamic`/`StringConcatFactory`). **반복문 안에서 누적할 때만** 명시적 StringBuilder가 필요합니다.

### 내부 구현 살펴보기
- `String`: `private final char[] value` (Java 8 이전) / `private final byte[] value` + coder (Java 9+ Compact Strings)
- `StringBuilder` / `StringBuffer`: `AbstractStringBuilder`를 공유. `char[] value` 버퍼와 `count` 필드. `append` 시 버퍼가 부족하면 보통 2배+2로 확장.
- `StringBuffer`의 모든 변경 메서드에는 `synchronized`가 붙어 있어 락 획득·해제 비용이 듭니다.

### String 불변성이 주는 이점
- **String Pool 공유**: 같은 리터럴은 한 인스턴스만 → 메모리 절약
- **해시 캐싱**: 한번 계산한 hashCode를 재사용 (HashMap 키로 안성맞춤)
- **Thread-safe**: 동시에 읽기만 하면 동기화 불필요
- **보안**: 인자로 받은 문자열이 도중에 바뀔 수 없음 (방어적 복사 불필요)

### 어떤 걸 언제 써야 하나요?
| 상황 | 권장 |
|---|---|
| 변경 없는 짧은 문자열, 키, 메시지 | `String` |
| 단일 라인의 단순 연결 (변수 1~2개) | `String` 또는 `String.format` |
| 단일 스레드에서 반복적으로 누적 | `StringBuilder` |
| 멀티스레드 공유가 필요한 누적 | 사실상 거의 없음. 보통 thread-local 버퍼로 분리하거나, 외부에서 lock |
| 대량의 join | `String.join`, `Collectors.joining` (내부적으로 StringBuilder 사용) |
| 포맷팅 | `String.format`, `MessageFormat`, 텍스트 블록(`"""..."""`) |

### 자주 묻는 질문
**Q. 컴파일러가 String 연결을 StringBuilder로 바꿔준다면 차이가 없는 거 아닌가요?**
컴파일러는 **하나의 표현식 안에서**의 연결만 최적화합니다. 반복문이 돌 때마다 `sb = new StringBuilder()`가 새로 생성되므로 누적 효과를 보려면 반복문 밖에 StringBuilder를 두어야 합니다.

**Q. StringBuffer는 정말 안 써도 되나요?**
거의 그렇습니다. 메서드 단위 동기화는 효과가 제한적이고, 멀티스레드 환경에서 여러 append를 묶어 원자적으로 처리하려면 결국 외부 락이 필요합니다. 차라리 thread-local로 StringBuilder를 분리하거나, 합쳐야 할 결과를 큐로 모아 마지막에 한 번에 처리하는 편이 빠릅니다.

[String Constant Pool 정리](250126-StringConstantPool.md)도 함께 보면 String의 메모리 동작을 이해하는 데 도움이 됩니다.
