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

