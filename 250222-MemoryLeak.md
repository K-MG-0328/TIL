## 메모리 누수(Memory Leak)란
CS의미로 살펴 볼 때 컴퓨터 프로그램이 필요하지 않은 메모리를 계속 점유하고 있는 현상이며 자바에서는 더 이상 사용되지 않는 객체들이 가비지 컬렉터에 의해 회수되지 않고 계속 누적이 되는 현상을 말한다.
Old 영역에 계속 누적된 객체로 인해 Major GC가 빈번하게 발생하게 되면 프로그램 응답속도가 늦어지면서 성능 저하를 불러온다. 이는 결국 OutOfMemory Error로 프로그램이 종료되게 된다.

### 루트 참조 객체(Root Reference Object)란
루트 참조 객체(Root Reference Object)는 GC가 객체 도달 여부를 판별할 때 시작점이 되는 객체(또는 참조)입니다. 자바에서 이들은 보통 스택의 지역 변수, static 변수, 실행 중인 스레드 등이 해당하며, GC는 이 루트 참조 객체들에서 시작해 연결된 객체를 모두 Mark하고, 그외 도달하지 않는 객체들은 제거(Garbage Collection)합니다.

#### Young Generation과 Old Generation의 객체는 언제 GC의 대상이될까?
각 영역에서 GC 발생 시점에 GC ROOT에 저장되어있는 객체와 GC ROOT의 객체와 연결되어있는 객체를 제외한 모든 객체를 Garbage 처리를 하게됩니다.

#### Young Generation에서 Old Generation으로 넘어가는 동작 원리
Young Generation에서 Minor GC 발생 시점에  eden, survival 0과 1 영역에서 GC ROOT에 저장되어있는 객체와 GC ROOT의 객체가 참조하고 있는 객체를 제외한 모든 객체를 Garbage 처리를 하게되고 살아남은 객체는 survival 영역에 저장되게 됩니다. 각각의 survivol 영역에서 존재하다가 일정 회수 이상 살아남게되면 Old Generation으로 넘어가게됩니다.

### GC가 되지 않는 루트 참조 객체는 크게 3가지
<img width="669" alt="image" src="https://github.com/user-attachments/assets/ea61a3b0-79e6-4a3c-ae1e-bcc4ba477233" />

Method Area의 Static 변수  
모든 현재 자바 스레드 Stack Area 내의 지역 변수 및 매개 변수에 의한 객체 참조  
Native Method Stack 영역에서 JNI(Java Native Interface) 프로그램에 의해 동적으로 만들어지고 제거되는 JNI global 객체 참조  

### Out Of Memory가 발생한다면 어떻게 대처할 수 있을끼?
#### OOM 예상 발생 원인 분석

📌 1. 메모리 누수 (Memory Leak)
	•	불필요한 객체가 GC에 의해 제거되지 않고 계속 남아 있음
	•	static 변수, 캐시 미제거, 이벤트 리스너 미해제 등으로 인해 객체가 힙에 계속 남음
	•	GC Root에서 참조를 유지하고 있어 제거 대상이 아님

📌 2. 힙 메모리 부족 (Heap Size 부족)
	•	애플리케이션이 예상보다 많은 객체를 생성하여 Heap이 가득 참
	•	GC가 객체를 제거해도 계속해서 새 객체가 빠르게 할당되면 OOM 발생

📌 3. GC가 제대로 동작하지 않음 (GC 튜닝 필요)
	•	Old Generation이 가득 차 GC가 객체를 회수하지 못하는 상황
	•	GC 정책이 적절하지 않아 Full GC가 너무 자주 발생하거나, 반대로 충분히 실행되지 않음

📌 4. 네이티브 메모리 부족 (Metaspace, Direct Memory)
	•	Metaspace (클래스 로딩) 또는 Direct Memory (NIO 등) 영역이 부족하면 OOM 발생
	•	GC가 관리하는 힙 영역 외의 공간에서 OOM이 발생할 수도 있음

📌 5. 스레드 개수 과다 생성 (Too Many Threads)
	•	스레드가 너무 많이 생성되면, 스레드 스택을 위한 네이티브 메모리가 부족해 OOM 발생 가능 (OutOfMemoryError: unable to create new native thread)

#### 해결 방법
 #### 1. 메모리 누수(Memory Leak) 확인 및 제거  
  
📌 [원인]  
	•	static 변수나 전역 객체에서 불필요한 객체를 계속 참조하여 GC가 회수하지 못함  
	•	캐시, 이벤트 리스너, ThreadLocal, 내부 클래스 등이 객체를 계속 잡고 있음  
  
📌 [해결 방법]  
	•	Heap Dump 분석 (jmap, VisualVM, Eclipse MAT)을 통해 어떤 객체가 메모리를 차지하고 있는지 확인  
     
 #### 2. 힙 메모리 크기 조정 (-Xms, -Xmx)  
  
📌 [원인]  
	•	Heap 메모리가 너무 작아서 OOM 발생  
	•	애플리케이션의 객체 생성 속도가 빠르고, GC가 충분히 회수하지 못하는 경우  
  
📌 [해결 방법]  
	•	애플리케이션에 적절한 힙 크기 설정 (-Xmx, -Xms)  
 
       java -Xms512m -Xmx2g -jar myapp.jar
         
#### 3. 스레드 개수 제한  
  
📌 [원인]  
	•	OutOfMemoryError: unable to create new native thread  
	•	너무 많은 스레드 생성으로 인해 네이티브 메모리 부족  
  
📌 [해결 방법]  
	•	최대 스레드 개수 제한 (ulimit -u, ThreadPool 사용)  
  
#### 출처
https://junghyungil.tistory.com/133  
https://junghyungil.tistory.com/8?category=892275  
https://junghyungil.tistory.com/9?category=892275  
