### 예외와 에러의 차이는 무엇일까?
에러(Error): 복구할 수 없는 치명적인 오류로, 프로그램이 더 이상 정상적으로 실행될 수 없는 상태를 의미합니다.  
	•	예: OutOfMemoryError, StackOverflowError  
	•	JVM 레벨에서 발생하는 문제이며, 개발자가 예외 처리(try-catch)를 하더라도 복구할 수 없습니다.  
	  
예외(Exception): 복구 가능한 오류로, 개발자가 try-catch를 통해 적절히 처리하여 프로그램의 흐름을 유지할 수 있는 오류입니다.  
	•	예: NullPointerException, IOException  
	•	개발자가 직접 예외 처리를 통해 해결할 수 있습니다.  

### Checked Exception, UnChecked Exception 의 차이는 무엇인가?
Checked Exception 예외 처리(try-catch, throws)를 해야하고 UnChecked Exception은 예외 처리가 강제되지 않습니다.  
Checked Exception은 Exception 클래스를 상속받은 예외들이고 UnChecked Exception은 RuntimeException을 상속받은 예외들입니다.  
  
#### 대표적인 Checked Exception
IOException	파일, 네트워크 등의 입출력 문제  
SQLException	데이터베이스 처리 중 오류  
ClassNotFoundException	클래스 로드 실패  
InterruptedException	스레드가 대기 중 인터럽트됨  
  
#### 대표적인 Unchecked Exception
NullPointerException	null 객체에 접근할 때  
ArrayIndexOutOfBoundsException	배열의 인덱스를 초과할 때  
ArithmeticException	0으로 나누는 연산 수행  
IllegalArgumentException	메서드에 잘못된 인수를 전달할 때  
