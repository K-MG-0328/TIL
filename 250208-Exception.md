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

### Throwable 계층 구조
```
Throwable
├── Error           (복구 불가, 예: OutOfMemoryError, StackOverflowError)
└── Exception
    ├── (Checked)   (예: IOException, SQLException)
    └── RuntimeException  (Unchecked, 예: NullPointerException, IllegalArgumentException)
```

### Custom Exception을 어떻게 만드나요?
도메인별로 의미 있는 예외를 정의하면 호출자가 어떤 케이스를 처리해야 하는지 명확해집니다.

```java
public class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(Long orderId) {
        super("주문을 찾을 수 없습니다. orderId=" + orderId);
    }
}

// 사용
public Order findOrder(Long id) {
    return orderRepository.findById(id)
        .orElseThrow(() -> new OrderNotFoundException(id));
}
```

원칙:
- 도메인 예외는 보통 **RuntimeException 상속** (호출자가 강제로 catch하지 않도록)
- 메시지에 식별 가능한 정보 포함
- 필요시 추가 필드(예: errorCode)를 두어 글로벌 예외 처리에서 활용

### try-with-resources는 언제 쓰나요?
`AutoCloseable`을 구현한 리소스(InputStream, Connection 등)는 try-with-resources로 자동 close 됩니다.

```java
// 권장
try (BufferedReader reader = Files.newBufferedReader(path)) {
    return reader.readLine();
}  // close()가 자동으로 호출됨

// 비권장 (close 누락 위험)
BufferedReader reader = Files.newBufferedReader(path);
try {
    return reader.readLine();
} finally {
    reader.close();
}
```

### 예외 처리 모범 사례
- **빈 catch 블록 금지**: 최소한 로그라도 남겨야 함. `catch (Exception ignored) {}`은 디버깅 지옥의 시작.
- **로그에 stack trace 포함**: `log.error("실패: {}", message, e)` 형태로 e를 마지막 인자로 넘겨야 trace가 출력됨.
- **예외 wrapping 시 cause 유지**: `throw new MyException("...", e)` ← 원인을 잃지 않음.
- **너무 넓은 catch 지양**: `catch (Exception e)` 대신 구체적 예외부터 잡기.
- **흐름 제어에 예외 사용 금지**: 예외 생성·throw는 비싸고 의도가 흐려짐. 반환값/Optional로 표현.

### Spring에서의 글로벌 예외 처리
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handle(OrderNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("ORDER_NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handle(MethodArgumentNotValidException e) {
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_FAILED", e.getMessage()));
    }
}
```

각 컨트롤러에 try-catch를 흩뿌리지 않고 한 곳에서 응답 형식을 통일할 수 있습니다.

### 멀티스레드 환경에서의 예외
- 새 스레드에서 발생한 예외는 호출 스레드로 전파되지 않습니다.
- `ExecutorService.submit()`은 `Future.get()`을 호출할 때 비로소 `ExecutionException`으로 감싸 던집니다.
- 처리되지 않은 예외는 `Thread.UncaughtExceptionHandler`로 잡을 수 있습니다.

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<String> future = executor.submit(() -> { throw new RuntimeException("oops"); });
try {
    future.get();   // 여기서 ExecutionException으로 던져짐
} catch (ExecutionException e) {
    Throwable cause = e.getCause();   // 원본 예외
}
```
