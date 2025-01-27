## Redis란
Redis는 데이터 처리 속도가 엄청 빠른 NoSQL 데이터베이스이다

### NoSQL 데이터베이스란
Not Only SQL의 약자로 Redis에서는 데이터를 키와 값 쌍으로 저장하며 단순 조회 및 저장에 최적화된 Key-Value 데이터베이스를 사용합니다.

### Redis의 장점
Redis는 in-memory에 모든 데이터를 저장합니다. 그래서 데이터의 처리 성능이 빠릅니다.

### Redis 명령어

#### set 명령어
127.0.0.1:6379> set mingyu:name "mingyu kim"
OK
127.0.0.1:6379> set mingyu:hobby soccer
OK

#### get 명령어
127.0.0.1:6379> get mingyu:name
"mingyu kim"
127.0.0.1:6379> get mingyu:hobby
"soccer"

#### keys * 명령어
127.0.0.1:6379> keys *
1) "mingyu:hobby"
2) "mingyu:name"

#### del 명령어
127.0.0.1:6379> del mingyu:hobby
(integer) 1
127.0.0.1:6379> get mingyu:hobby
(nil)

#### ttl 명령어
127.0.0.1:6379> set mingyu:pet dog ex 30 // 30초 타임아웃 설정
OK
127.0.0.1:6379> ttl mingyu:pet
(integer) 18
127.0.0.1:6379> ttl mingyu:pet
(integer) 12
127.0.0.1:6379> ttl mingyu:pet
(integer) -2
127.0.0.1:6379> ttl mingyu:name
(integer) -1
127.0.0.1:6379> flushall 
OK
127.0.0.1:6379> keys *
(empty array)

### Redis Key Naming Convention
users:100:profile  - 사용자들(users) 중에서 PK가 100인 사용자(user)의 프로필(profile)
products:123:details - 상품들(products) 중에서 PK가 123인 상품(product)의 세부사항(details)


### 데이터를 캐싱할 때 사용하는 전략 

#### Cache Aside(=Look Aside, Lazy Loading) 전략
1. 사용자가 데이터베이스에 저장  
2. 사용자가 데이터를 조회 시도  
3. Redis에 데이터 요청  
4. Cache Miss  
5. DB에 데이터 요청  
6. DB에 데이터 응답  
7. Redis에 데이터 저장  
8. 사용자가 같은 데이터 조회 시도  
9. Redis에 데이터 요청  
10. Cache Hit  
11. 데이터 응답  
애플리케이션이 Redis(Cache)에 먼저 데이터 요청 후 DB에 요청하는 전략  

#### Write Around 전략
기존 Cache Aside 전략과 동일한 흐름을 가집니다.  
다만 차이점으로는 데이터 갱신 시 Cache Aside 전략은 Cache 데이터를 삭제함으로써 DB 데이터와 동기화를 하지만   
Write Around 전략은 데이터 갱신 시 Cache 데이터를 놔두고 DB 데이터만 변경합니다.  

#### 그렇다면 Write Around 에서 데이터 갱신시 DB에 데이터만 저장하고 캐시를 그대로 놔둔다면 DB 데이터와 캐시 데이터의 불일치가 발생하지 않을까?
Write Around는 쓰기 작업에서 캐시를 무시하기 때문에, 캐시와 데이터베이스 간 동기화를 보장하지않습니다.

#### 그렇다면 어떻게 데이터 불일치 문제를 해결하거나 완화할 수 있을까?
캐시 데이터 유효 기간을 설정해 일정 시간이 지나면 자동으로 무효화되도록 설정하는 방법이 있습니다. 하지만 TTL 설정에 따라 데이터가 갱신되기 전까지 불일치가 발생할 수 있습니다.  
  
읽기 요청시 캐시와 데이터베이스의 데이터 일치 여부를 검증하는 방법이 있을 수 있습니다. 하지만 성능이 저하될 수 있습니다.    
  
**다른 캐시 전략과 혼합해서 사용하는 방법 Cache Aside 전략과 함께 사용하여 필요한 경우 캐시를 갱신하거나 무효화하는 방법이 있습니다.**  

