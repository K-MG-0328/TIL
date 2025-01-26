# Optional 클래스란
Optional 클래스는 null 참조로 인한 NullPointerException 문제를 방지하기 위해 설계되었습니다.
Optional은 값이 있을 수도 있고 없을 수도 있는 객체를 표현하는 컨테이너로 명시적으로 값이 비어 있는 상태를 처리할 수 있도록합니다.

## Optional 생성 방법
of() - 값이 반드시 null이 아니라고 보장되는 경우 사용합니다. null값이 전달 되면 NullPointerException이 발생합니다.   
```Optional<String> optional = Optional.of("Hello"); ```

ofNullable()
값이 null일 수도 있는 경우 사용합니다. null 값이 전달되면 빈 Optional 객체를 생성합니다.  
```Optional<String> optional = Optional.ofNullable(null);```

empty()
빈 Optional 객체 생성  
```Optional<String> optional = Optional.empty();```

## Optional 주요 메서드

isPresent() / isEmpty()
Optional 객체에 값이 있는지 확인합니다.
```
Optional<String> optional = Optional.of("Hello");

if(optional.isPresent()){
	System.out.println(optional.get()); // "Hello"
}

Optional<String> emptyOptional = Optional.empty();
System.out.println(emptyOptional.isEmpty()); // true
```

get()
Optional에 저장된 값을 반환합니다. 값이 없으면 NoSuchElementException이 발생합니다.
```
Optional<String> optional = Optional.of("Hello");
System.out.println(optional.get()); // "Hello"
```

orElse()
Optional에 값이 있으면 반환하고, 없으면 기본값 반환.
```
Optional<String> optional = Optional.empty();
String result = optional.orElse("Default");
System.out.println(result); // "Default"
```

orElseGet()
값이 없을 때 기본값을 제공하는 Supplier를 실행.
```
Optional<String> optional = Optional.empty();
String result = optional.orElseGet(() -> "Generated Default");
System.out.println(result); // "Generated Default"
```

orElse()	orElseGet() 차이  
대체 값 실행 시점 : **orElse**(항상 실행 (값 유무와 관계없이)) **orElseGet**(값이 없을 때만 실행)  
대체 값 제공 방식	: **orElse**(직접 제공)	**orElseGet**(Supplier 함수형 인터페이스를 통해 제공)  
사용 예	: **orElse**(대체 값 생성이 간단하거나 가벼운 경우)	**orElseGet**(대체 값 생성 비용이 크거나 복잡한 경우)  

orElseThrow()
Optional에 값이 없으면 예외를 발생.  
```
Optional<String> optional = Optional.empty();
String result = optional.orElseThrow(() -> new IllegalArgumentException("No value present"));
```

ifPresent()
값이 있는 경우 특정 동작 수행.  
```
Optional<String> optional = Optional.of("Hello");
optional.ifPresent(value -> System.out.println(value)); // "Hello"
```
map()
Optional의 값을 변환.  
```
Optional<String> optional = Optional.of("Hello");
Optional<Integer> length = optional.map(String::length);
System.out.println(length.get()); // 5
```

flatMap()
Optional의 값을 중첩되지 않도록 처리.  
```
Optional<String> optional = Optional.of("Hello");
Optional<String> upper = optional.flatMap(value -> Optional.of(value.toUpperCase()));
System.out.println(upper.get()); // "HELLO"
```

filter()
조건에 따라 값을 필터링.  
```
Optional<String> optional = Optional.of("Hello");
Optional<String> result = optional.filter(value -> value.startsWith("H"));
System.out.println(result.isPresent()); // true
```

## Optional 사용 사례
### null 체크 대신 Optional 사용

```
public Optional<String> getUserName(User user) {
    return Optional.ofNullable(user.getName());
}

// 값이 있는 경우 처리
getUserName(user).ifPresent(name -> System.out.println("User name: " + name));
```
### 값이 없을 때 기본값 제공
```
Optional<String> name = Optional.empty();
String result = name.orElse("Anonymous");
System.out.println(result); // "Anonymous"
```
### 중첩된 null 처리
```
public Optional<Address> getAddress(User user) {
    return Optional.ofNullable(user.getAddress());
}

public Optional<String> getCity(User user) {
    return getAddress(user).map(Address::getCity);
}
```

## Optional 사용 시 주의사항
필드에 Optional 사용 지양 : Optional은 주로 메서드 반환값에 사용됩니다.   필드나 메서드 매개변수에는 사용하지 않는 것이 권장됩니다.

null 대신 빈 Optional 반환 : Optional 자체가 null이 되지 않도록 항상 **Optional.empty()**를 반환해야 합니다.

get() 사용 자제 : get() 메서드는 값이 없을 경우 예외를 발생시키므로, 안전한 대안을 사용하는 것이 좋습니다.
