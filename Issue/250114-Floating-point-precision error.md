### 부동소수점 오류가 무엇인가?
부동소수점 오류란 10진수 0.1, 0.2를 2진수 변경시 무한소수가 됩니다.. 컴퓨터는 유한한 메모리로 이 값을 표현해야하므로 근사값으로 저장합니다.
여기서 생기는 차이로 인해 진짜 값과 차이가 발생하게되는데 이것을 부동소수점 오류라고합니다.
0.1과 0.2를 더 했을 경우 논리적으로는 0.3으로 결과값이 나와야하지만 컴퓨터에서는 연산시 이진수로 변환 후 연산을 하므로 0.1의 근사값과 0.2의 근사값을 더하게되면서 0.30000000000000004라는 값이 나오게됩니다.
double a = 0.1 + 0.2;
System.out.println(a == 0.3); // false (a == 0.30000000000000004)

#### 부동소수점 오류로 인해 생길 수 있는 문제들은 무엇인가요?
비교 연산의 부정확성이 발생할 수 있습니다. 근사값으로 저장되므로 a == 0.3 같은 비교연산이 실패하게됩니다.
또는 반복문을 통해 누적된 실수 근사값을 저장하게되면 오차로 인해 결과가 왜곡이 될 수 있습니다.

#### 부동소수점 오류를 해결하기 위한 방법은?
BigDecimal을 사용합니다.
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
BigDecimal sum = a.add(b);
여기서 주의 사항은 BigDecimal은 문자열로 초기화해야합니다. 부동소수점 값을 그대로 전달하면 오차가 포함된 상태로 저장되게됩니다.
BigDecimal a = new BigDecimal(0.1); // 0.1에 이미 오차 포함
System.out.println(a); // 출력: 0.1000000000000000055511151231257827021181583404541015625

#### IEEE 754 부동소수점 표현 원리
double은 64비트로 표현됩니다.
- 부호(1bit) + 지수(11bit) + 가수(52bit)
- `0.1`은 이진수에서 `0.0001100110011...`로 무한 반복되는 소수라 가수 52비트 안에 정확히 들어갈 수 없습니다.
- 결과: 가장 가까운 표현 가능한 값으로 반올림 → 미세한 오차 발생

```java
System.out.println(new BigDecimal(0.1).toPlainString());
// 0.1000000000000000055511151231257827021181583404541015625
```

#### BigDecimal 사용 시 주의할 점
- **문자열 생성자**: `new BigDecimal("0.1")` ← 정확
- **double 생성자 금지**: `new BigDecimal(0.1)` ← double의 오차가 그대로 들어감
- **나눗셈은 scale 지정 필수**: `1/3`처럼 떨어지지 않는 나눗셈은 ArithmeticException 발생
  ```java
  BigDecimal result = a.divide(b, 10, RoundingMode.HALF_UP);  // 소수점 10자리, 반올림 모드 명시
  ```
- **equals와 compareTo 차이**: `new BigDecimal("1.0").equals(new BigDecimal("1.00"))`은 **false** (scale이 다름). 값만 비교하려면 `compareTo() == 0`을 사용.

#### 성능 비교
| 타입 | 연산 속도 | 정밀도 | 사용처 |
|---|---|---|---|
| `double` | 매우 빠름 (CPU FPU) | 약 15~17자리, 오차 있음 | 그래픽, 통계, 머신러닝 |
| `BigDecimal` | double의 수십~수백 배 느림 | 임의 정밀도, 정확 | **돈, 회계, 세금 계산** |
| `long` (정수 변환) | double급 빠름 | 정확 (단, 단위 변환 필요) | 화폐의 최소 단위 (원, cent) |

#### 실무에서 자주 쓰는 패턴: 정수 변환
화폐 계산은 `BigDecimal` 또는 **최소 단위 정수**로 처리합니다. 한국 원은 보통 `long` 원 단위로 충분합니다.
```java
long won = 12_300L;            // 12,300원
long discounted = won * 90 / 100;  // 10% 할인 → 11,070원
```

달러처럼 소수점이 있는 화폐는 `cent` 단위 정수로 저장하거나 `BigDecimal`을 사용합니다.

#### 부동소수점 비교는 어떻게?
`==`이나 `equals` 대신 **허용 오차(epsilon)** 기반 비교를 사용합니다.
```java
double a = 0.1 + 0.2;
double b = 0.3;
boolean equal = Math.abs(a - b) < 1e-9;  // true
```

JUnit의 `assertEquals(expected, actual, delta)`도 같은 원리입니다.
