### 컬렉션 클래스들의 제네릭 타입이 런타임에서 어떻게 처리되는가?
컬렉션 클래스의 제네릭 타입들은 런타임 시점에 타입 이레이저에 의해 타입 정보가 제거되며 Object 또는 특정 상위 타입으로 변환됩니다.  
ex) 제네릭 타입이 Object로 변환

    class Box<T> {
        private T value;
    
        public void set(T value) { this.value = value; }
        public T get() { return value; }
    }


    class Box {
        private Object value; // T → Object로 변환됨
    
        public void set(Object value) { this.value = value; }
        public Object get() { return value; }
    }

ex) 제네릭 타입이 특정 상위 타입으로 변환

    class Box<T extends Number> { // Number의 하위 타입만 가능
        private T value;
    
        public void set(T value) { this.value = value; }
        public T get() { return value; }
        }
    
    class Box {
        private Number value; // T → Number로 변환됨
    
        public void set(Number value) { this.value = value; }
        public Number get() { return value; }
    }

### 타입 이레이저(Type Erasure)란?
타입 이레이저란 제네릭이 컴파일될 때 제네릭 타입 정보가 제거되는 과정을 의미합니다. 즉, 제네릭은 컴파일 타임에만 존재하며 런타임에는 사라지는 특성을 가집니다.  

### 타입 이레이저로 제네릭 타입을 제거하는 이유는?
1. 기존 Java(JDK 1.4 이하)와 호환되도록 하기 위해(jdk 1.4 이하에서는 제네릭이 없었으므로 강제 형 변환이 필요했었습니다.)  
2. 런타임 성능 최적화(런타임에 타입 검사를 수행하면 성능이 저하 및 불피룡한 메모리 사용 증가)
즉, 타입 이레이저는 컴파일 시 타입 안정성 보장과 런타임 성능 최적화를 동시에 달성하기 위한 것입니다.

### 이를 우회하는 방법(Super Type Token)은?
기본적으로 제네릭은 런타임 시에 타입 이레이저에 의해 제네릭 정보가 제거됩니다. 하지만 슈퍼 타입 토큰을 이용해 제네릭을 런타임에도 유지할 수 있습니다.

핵심은 **익명 클래스를 통해 부모 타입의 제네릭 정보를 클래스 메타데이터에 박제**하는 것입니다. 익명 클래스의 부모 타입 정보(`getGenericSuperclass()`)는 컴파일된 클래스 파일에 남아 리플렉션으로 읽을 수 있습니다.

```java
public abstract class TypeReference<T> {
    private final Type type;

    protected TypeReference() {
        Type superClass = getClass().getGenericSuperclass();
        this.type = ((ParameterizedType) superClass).getActualTypeArguments()[0];
    }

    public Type getType() { return type; }
}

// 사용: 익명 클래스로 인스턴스화 → 제네릭 정보가 보존됨
TypeReference<List<String>> ref = new TypeReference<List<String>>() {};
System.out.println(ref.getType()); // java.util.List<java.lang.String>
```

Spring의 `ParameterizedTypeReference`나 Jackson의 `TypeReference`가 모두 같은 원리로 동작합니다.

### 와일드카드(`? extends`, `? super`)와 PECS 원칙
제네릭은 기본적으로 **불공변(invariant)** 입니다. `List<Number>`와 `List<Integer>`는 상속 관계가 없어서 서로 대입할 수 없습니다. 이를 유연하게 다루기 위해 와일드카드를 사용합니다.

* **`? extends T` (상한 경계)**: T 또는 T의 자식 타입. 데이터를 **읽을 때(Producer)** 사용.
* **`? super T` (하한 경계)**: T 또는 T의 부모 타입. 데이터를 **쓸 때(Consumer)** 사용.

이 규칙을 외우는 것이 **PECS — Producer Extends, Consumer Super**입니다.

```java
// 읽기 전용: List<Integer>, List<Double>도 받을 수 있음
public static double sumAll(List<? extends Number> numbers) {
    double total = 0;
    for (Number n : numbers) total += n.doubleValue(); // 읽기 OK
    // numbers.add(1); // 컴파일 에러: 어떤 하위 타입인지 모르므로 쓰기 금지
    return total;
}

// 쓰기 전용: List<Integer>, List<Number>, List<Object>에 모두 채울 수 있음
public static void fillWithOne(List<? super Integer> list) {
    list.add(1);            // 쓰기 OK
    // Integer x = list.get(0); // 컴파일 에러: Object로만 꺼낼 수 있음
}
```

### 제네릭 메서드 vs 제네릭 클래스는 어떻게 다른가요?
* **제네릭 클래스**: 클래스 자체가 타입 파라미터를 가지며, 인스턴스화 시점에 타입이 결정됩니다.
* **제네릭 메서드**: 메서드 단위로 타입 파라미터를 선언하며, 호출 시점에 타입이 결정됩니다. `static` 메서드에서도 사용 가능합니다.

```java
// 제네릭 클래스
public class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

// 제네릭 메서드 (반환 타입 앞에 <T>)
public class Utils {
    public static <T> List<T> repeat(T item, int n) {
        List<T> result = new ArrayList<>();
        for (int i = 0; i < n; i++) result.add(item);
        return result;
    }
}

List<String> hellos = Utils.repeat("hi", 3); // 호출 시 T = String 결정
```

### 제네릭 사용 시 흔한 실수는?
1. **`List<Object>`와 `List<String>`은 무관**합니다. `List<String>`을 `List<Object>` 파라미터에 전달할 수 없습니다. 다형성을 원한다면 와일드카드를 사용해야 합니다.
2. **제네릭 배열은 만들 수 없습니다.** `new T[10]`이나 `new List<String>[10]` 모두 컴파일 에러입니다. 타입 이레이저로 인해 런타임에 타입 안전성을 보장할 수 없기 때문입니다.
3. **`instanceof T`나 `instanceof List<String>`은 사용할 수 없습니다.** 런타임에는 타입 정보가 없기 때문입니다. `instanceof List<?>`처럼 와일드카드만 가능합니다.
4. **정적 컨텍스트에서 클래스 타입 파라미터 사용 금지**. `static` 필드나 메서드는 인스턴스가 아닌 클래스 단위라 `T`에 접근할 수 없습니다.

