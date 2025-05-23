## 데드락
데드락은 두 개 이상의 프로세스 또는 스레드가 서로 상대방이 가진 자원을 기다리며 무한정 대기하는 현상을 의미합니다.  
서로가 원하는 자원을 점유하고 있으며, 상대가 가진 자원을 기다리기 때문에 더 이상 작업이 진행되지 않는 상태입니다.  
**데드락의 네 가지 발생 조건이 동시에 성립되지 않도록 설계하거나 관리하여 예방해야합니다.**

### 데드락의 발생 조건 
#### 상호 배제(Mutual Exclusion)
특정 자원은 한번에 하나의 프로세스만 사용할 수 있어야합니다.

#### 점유와 대기(Hold and Wait)
프로세스가 이미 자원을 점유한 상태에서 다른 자원을 기다리는 상태

#### 비선점(No Preemption)
한 프로세스가 점유한 자원을 다른 프로세스가 강제로 빼앗을 수 없어야 합니다.

#### 순환 대기(Circular Wait)
프로세스들이 원형으로 서로가 점유한 자원을 기다리는 순환 구조가 형성되어야 합니다.

### 데드락의 예시
**상황**
•	프로세스 A는 자원 X를 가지고 자원 Y를 기다림.  
•	프로세스 B는 자원 Y를 가지고 자원 X를 기다림.   
 위 상황을 조건으로 분석하면  
 **상호배제** : X, Y는 한 번에 한 프로세스만 사용 가능 (X, Y가 한 프로세스가 아닌 여러 프로세스가 자원을 점유할 수 있다면 데드락 발생하지 않음)  
 **점유와 대기** : A, B 모두 자원을 가진 상태로 기다림  (타임아웃이나 또는 자원을 점유했지만 추가 자원을 기다리지 않는다면 데드락 발생하지 않음)  
 **비선점** : 서로의 자원을 뺏을 수 없음 (자원을 뺏을 수 있다면 데드락 발생하지 않음)  
 **순환 대기** : A->B->A의 순환 구조 (서로가 자원을 점유한 채 자원을 얻기 위해 대기 상태 해당 구조가 아니라면 데드락 발생하지 않음)  
  
### 자원은 하나의 프로세스나 스레드만 사용할 수 있는 것인가?
모든 자원이 무조건 하나의 프로세스나 스레드만 사용할 수 있는 것은 아니며 자원의 성격에 따라 두 가지로 나눌 수 있습니다.   
  
**상호 배제 자원(Mutual Exclusion Resource)**  
한 번에 단 하나의 프로세스나 스레드만 점유 가능한 자원입니다.  
대표적인 예:  
•	프린터, 하드디스크, 특정 하드웨어 장치  
•	데이터베이스의 특정 레코드에 대한 쓰기(write) 권한  
•	메모리의 특정 영역에 대한 쓰기 권한(배타적 접근이 요구될 때)  
•	Java의 synchronized 메소드나 블록, Lock 객체 등  
이런 자원들은 한 번에 한 프로세스(스레드)가 독점적으로 사용하기 때문에, 데드락의 핵심 조건 중 하나인 “상호배제” 조건을 만족하게 됩니다.  

**공유 가능 자원 (Sharable Resource)**  
여러 프로세스나 스레드가 동시에 참조(접근)할 수 있는 자원입니다.  
대표적인 예:  
	•	읽기 전용 데이터 (Read-only Data)  
	•	데이터베이스에서의 읽기(Read) 권한  
	•	다수의 스레드가 동시에 접근해도 문제가 없는 리소스 (예: 웹 페이지, 캐시 데이터 등)  
이런 자원은 동시에 여러 프로세스나 스레드가 접근할 수 있으며, 상호배제 조건을 만족하지 않기 때문에 데드락을 일으키지 않습니다.

### 데드락 예방을 위한 조건 깨기 예시 
 **상호배제** : 공유 가능한 자원은 가능한 한 동시에 접근 허용 - Read-Write Lock 등을 활용 (읽기 전용 데이터에 동시 접근 허용)   
 **점유와 대기** : 프로세스가 필요한 모든 자원을 미리 한번에 점유하도록 설계(필요한 자원을 한 번에 요청하도록 설계)    
 **비선점** : 필요한 경우 강제로 자원을 빼앗을 수 있도록 설계 (자원을 일정 시간 이후 자동 회수)  
 **순환 대기** : 자원을 요청하는 순서를 항상 동일한 순서로 설정 (프로세스들이 자원을 알파벳 순서로 요청)  
•	자원 획득 순서를 통일하여 순환 대기 조건 제거  
•	점유 시간을 짧게 유지하고, lock을 최소한만 사용  
•	데이터베이스에서는 트랜잭션을 가능한 짧게 유지  
•	락에 타임아웃 설정을 적용해 오래 대기하지 않도록 설정  


 ### 발생할 수 있는 사례
 **데이터베이스 트랜잭션에서의 데드락**   
 트랜잭션 A가 UPDATE users SET balance=balance-100 WHERE id=1; (user 1 잠금)    
 트랜잭션 B가 UPDATE users SET balance=balance+100 WHERE id=2; (user 2 잠금)  
 이후  
 트랜잭션 A가 UPDATE users SET balance=balance+100 WHERE id=2; (대기 상태)  
 트랜잭션 B가 UPDATE users SET balance=balance-100 WHERE id=1; (대기 상태)  
두 개의 트랜잭션이 상대가 잠근 row를 기다리며 순환 대기 상태가 되어 데드락이 발생

**스프링에서 @Transactional 사용 시 데드락**  

	@Transactional
	public void transferMoney(Long fromUserId, Long toUserId, int amount) {
	    User fromUser = userRepository.findByIdForUpdate(fromUserId); // Lock 걸림
	    User toUser = userRepository.findByIdForUpdate(toUserId);     // 다른 트랜잭션과 순서가 반대라면 데드락 가능성!
	    fromUser.decreaseBalance(amount);
	    toUser.increaseBalance(amount);
	}



