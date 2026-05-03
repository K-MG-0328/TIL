### 인터페이스와 추상클래스의 공통점과 차이점은 무엇인가요?
인터페이스와 추상클래스 모두 구현 클래스에 추상 메서드를 구현해줘야합니다. 또한 부모 타입으로 자식 객체를 참조가 가능합니다.  
인터페이스는 다중 상속이 가능하다. 추상 클래스는 단일 상속만 가능하다.   
인터페이스는 상태(필드)에 상수만 가능하지만 추상클래스는 일반 멤버 변수(인스턴스 변수) 선언이 가능합니다.  

| 구분 | Interface | Abstract Class |
|---|---|---|
| 다중 상속 | 가능 (여러 인터페이스 구현 가능) | 불가능 (단일 상속만 가능) |
| 필드 | `public static final` 상수만 가능 | 일반 인스턴스 변수 선언 가능 |
| 생성자 | 가질 수 없음 | 가질 수 있음 |
| 메서드 | 추상 메서드 + `default`/`static` 메서드 (Java 8+) + `private` 메서드 (Java 9+) | 추상 메서드 + 구현 메서드 모두 가능 |
| 키워드 | `implements` | `extends` |
| 의도 | "무엇을 할 수 있는가"(역할/기능 명세) | "무엇이며 공통 구현은 무엇인가"(타입 계층 + 공통 코드) |

### 인터페이스와 추상클래스는 언제 사용해야하는걸까?
인터페이스를 고려할 때는 다중 상속의 여부와 특정 기능을 클래스에 구현 및 메서드의 구조만 정의해야할 때 사용을 고려해볼 수 있습니다.  
추상클래스를 고려할 때는 공통 가능을 하위 클래스에서 재사용 할 때 일반 상태값을 (인스턴스 변수)가 필요할 때 객체 생성 시 공통적인 로직 (초기화 등)이 필요할 때 사용을 고려해볼 수 있습니다.   
정리해보자면 인터페이스는 해당 클래스에서 어떠한 기능을 반드시 구현해야할 때 사용하는 것을 고려해볼 수 있을 것 같고 추상클래스는 공통된 기능을 상속된 클래스에서 사용하도록 할 때 고려해볼 수 있는 것 같습니다.  

#### 인터페이스 코드 예시
```java
public interface Payable {
    void pay(long amount);                  // 추상 메서드

    default void payWithLog(long amount) {  // 공통 흐름은 default로 제공
        System.out.println("결제 시작");
        pay(amount);
        System.out.println("결제 완료");
    }
}

public class CardPayment implements Payable {
    @Override
    public void pay(long amount) {
        // 카드사 API 호출
    }
}
```

#### 추상클래스 코드 예시
```java
public abstract class HttpClient {
    protected final String baseUrl;          // 공통 필드

    protected HttpClient(String baseUrl) {   // 생성자로 초기화
        this.baseUrl = baseUrl;
    }

    public final String get(String path) {   // 공통 흐름 고정
        String url = baseUrl + path;
        return doGet(url);
    }

    protected abstract String doGet(String url); // 자식이 구현
}

public class OkHttpClientImpl extends HttpClient {
    public OkHttpClientImpl(String baseUrl) { super(baseUrl); }

    @Override
    protected String doGet(String url) {
        // OkHttp 사용
        return "...";
    }
}
```

### default 메서드는 왜 등장했나요?
Java 8부터 인터페이스에 `default` 키워드로 메서드 본문을 정의할 수 있습니다. 등장 배경은 **기존 인터페이스에 메서드를 추가해도 구현 클래스가 깨지지 않도록 하기 위함**입니다. 대표적인 예가 `Collection`에 `stream()`이 추가될 때, 모든 구현체를 수정하지 않고 `default` 메서드로 호환성을 유지한 사례입니다.

#### 다이아몬드 문제는 어떻게 해결하나요?
두 인터페이스가 같은 시그니처의 default 메서드를 가지고 있고, 한 클래스가 둘을 모두 구현하면 컴파일 에러가 발생합니다. 이때 구현 클래스에서 명시적으로 어느 쪽을 쓸지 또는 자체 구현을 선택해야 합니다.

```java
interface A { default String hello() { return "A"; } }
interface B { default String hello() { return "B"; } }

class C implements A, B {
    @Override
    public String hello() {
        return A.super.hello(); // A의 default를 명시적으로 호출
    }
}
```

### 함수형 인터페이스(`@FunctionalInterface`)는 무엇인가요?
**추상 메서드가 정확히 1개**인 인터페이스로, 람다 식의 타깃 타입이 됩니다. `@FunctionalInterface` 애너테이션은 컴파일러에 의도를 명시해 추상 메서드가 2개 이상이 되면 컴파일 에러를 내도록 강제합니다.

```java
@FunctionalInterface
public interface Discount {
    long apply(long price);    // 추상 메서드 1개

    default long applyAll(long price) { return apply(price) - 100; } // default는 허용
}

Discount halfOff = price -> price / 2;
long result = halfOff.apply(10_000); // 5_000
```

표준 라이브러리에서는 `Function`, `Predicate`, `Consumer`, `Supplier` 등이 대표적입니다.

### 인터페이스와 추상클래스들을 구현한 클래스들의 관계는 무엇인가?
* Dog implements Animal → "Dog is an Animal" (인터페이스)  
* Dog extends Animal → "Dog is an Animal" (추상 클래스)  

인터페이스와 추상 클래스 모두 is-a 관계를 형성할 수 있습니다. has a 관계는 클래스 내부에서 어떠한 클래스를 주입받는 형태가 has a 관계라고 할 수 있습니다.  
