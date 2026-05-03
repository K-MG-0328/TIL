## 직렬화란
객체나 데이터 구조를 저장하거나 전송할 수 있는 형식으로 변환하는 과정    
직렬화 전 : 메모리에서 사용되는 객체(java 객체 등)  
직렬화 후 : 저장하거나 전송할 수 있는 이진 데이터 나 텍스트 형식(JSON, XML, 문자열)으로 변환  

### 직렬화가 필요한 이유
메모리 객체를 바로 전송하거나 저장할 수 없기 때문  
Java 객체 -> JSON -> 네트워크 전송  
Java 객체 -> JSON -> DB 저장, Redis 저장   

### Redis에서 객체를 직렬화 했을 때 필드에 LocalDateTime 필드가 존재할 경우 예외 발생
LocalDateTime 타입 필드는 기본적으로 직렬화를 지원하지 않기 때문에, Redis에 저장하거나 읽어올 때 직렬화를 명시적으로 처리해야 합니다. Java의 기본 직렬화 방식이나 JSON 직렬화 라이브러리는 LocalDateTime을 바로 직렬화하지 못합니다.  

#### 해결 방법
엔티티 필드에 JSON 직렬화 설정을 추가로 해줘야합니다.  

    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    private LocalDateTime createdAt;

    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    private LocalDateTime updatedAt;

### 더 깔끔한 해결책: JavaTimeModule 등록
필드마다 어노테이션을 다는 대신, ObjectMapper에 모듈을 한 번 등록하면 모든 시간 타입(LocalDateTime, ZonedDateTime, Duration 등)을 처리할 수 있습니다.

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        ObjectMapper mapper = new ObjectMapper()
            .registerModule(new JavaTimeModule())                                  // LocalDateTime 등 시간 타입
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)               // ISO-8601 문자열로 출력
            .activateDefaultTyping(BasicPolymorphicTypeValidator.builder()
                    .allowIfBaseType(Object.class).build(),
                ObjectMapper.DefaultTyping.NON_FINAL);

        var template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(factory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer(mapper));
        return template;
    }
}
```

빌드: `com.fasterxml.jackson.datatype:jackson-datatype-jsr310` 의존성 필요.

### Java 기본 Serializable vs JSON 직렬화
| 항목 | `java.io.Serializable` | JSON (Jackson 등) |
|---|---|---|
| 형식 | 바이너리 | 텍스트 |
| 가독성 | 낮음 | 높음 |
| 언어 호환성 | Java 전용 | 언어 무관 |
| 보안 | 역직렬화 공격 취약 (악명 높음) | 일반적으로 안전 |
| 버전 호환 | `serialVersionUID`로 관리 | 필드 추가/삭제에 유연 |
| 성능 | 빠름 | Jackson 기준 빠름, Gson은 느림 |

**Java 기본 Serializable은 역직렬화 시 임의 클래스 인스턴스화로 RCE(Remote Code Execution) 취약점이 발생한 사례가 많아 외부 입력 역직렬화에는 거의 사용하지 않습니다.** 내부 캐시에서만 제한적으로 쓰거나, 차라리 JSON으로 통일하는 것이 안전합니다.

### serialVersionUID는 왜 명시해야 하나요?
`Serializable`을 구현한 클래스에서 명시하지 않으면 컴파일러가 클래스 구조 기반으로 자동 계산합니다. 클래스에 필드 하나만 추가/삭제해도 UID가 바뀌어 **이미 직렬화된 데이터를 역직렬화할 때 `InvalidClassException`** 이 발생합니다.

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L;   // 명시적으로 고정
    private String name;
}
```

호환성을 위해 명시하되, 의도적으로 호환을 깨는 변경(필드 타입 변경 등)을 할 때는 UID도 같이 올립니다.

### 순환 참조와 자기 참조 처리
양방향 연관관계가 있는 JPA 엔티티를 그대로 JSON으로 직렬화하면 무한 루프(StackOverflowError)가 발생합니다.

```java
public class Order {
    @JsonManagedReference   // 정방향 (포함됨)
    private List<OrderItem> items;
}

public class OrderItem {
    @JsonBackReference      // 역방향 (직렬화 제외)
    private Order order;
}
```

또는 `@JsonIdentityInfo`로 ID 기반 참조 사용, 또는 DTO로 분리해서 직렬화하는 것이 가장 깔끔합니다.

### 다양한 시간 타입 직렬화 결과
JavaTimeModule + `WRITE_DATES_AS_TIMESTAMPS=false` 기준 출력:

| 타입 | 직렬화 결과 |
|---|---|
| `LocalDateTime` | `"2025-01-28T15:30:00"` |
| `LocalDate` | `"2025-01-28"` |
| `ZonedDateTime` | `"2025-01-28T15:30:00+09:00"` |
| `Instant` | `"2025-01-28T06:30:00Z"` |
| `Duration` | `"PT15M30S"` (ISO-8601 duration) |
| `Period` | `"P1Y2M3D"` |

### 큰 객체 그래프 직렬화 시 성능 팁
- **Streaming API** 사용: Jackson의 `JsonGenerator` / `JsonParser`로 메모리에 전체 트리를 올리지 않고 처리
- **불필요한 필드 제외**: `@JsonIgnore` 또는 DTO 분리
- **Lazy Loading 주의**: JPA 엔티티를 그대로 직렬화하면 모든 lazy 컬렉션이 로드되어 N+1 가능성. DTO 변환 후 직렬화 권장
- **압축**: Redis에 큰 객체를 저장한다면 gzip 등의 압축 적용 검토

### 직렬화 보안 체크리스트
- 외부 입력으로 받은 데이터를 `ObjectInputStream.readObject()`로 역직렬화하지 말 것
- Jackson 사용 시 `enableDefaultTyping`을 무차별 허용하지 말고 `PolymorphicTypeValidator`로 화이트리스트
- JWT 같은 토큰 페이로드를 신뢰하기 전에 시그니처 검증
- DTO에 사용자가 통제하는 필드명을 그대로 매핑하지 말고 명시적으로 허용 필드만 정의
