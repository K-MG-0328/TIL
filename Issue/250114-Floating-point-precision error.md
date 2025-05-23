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
