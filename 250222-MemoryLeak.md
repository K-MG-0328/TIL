## 메모리 누수(Memory Leak)란
CS의미로 살펴 볼 때 컴퓨터 프로그램이 필요하지 않은 메모리를 계속 점유하고 있는 현상이며 자바에서는 더 이상 사용되지 않는 객체들이 가비지 컬렉터에 의해 회수되지 않고 계속 누적이 되는 현상을 말한다.
Old 영역에 계속 누적된 객체로 인해 Major GC가 빈번하게 발생하게 되면 프로그램 응답속도가 늦어지면서 성능 저하를 불러온다. 이는 결국 OutOfMemory Error로 프로그램이 종료되게 된다.

### 루트 참조 객체(Root Reference Object)란
루트 참조 객체(Root Reference Object)는 GC가 객체 도달 여부를 판별할 때 시작점이 되는 객체(또는 참조)입니다. 자바에서 이들은 보통 스택의 지역 변수, static 변수, 실행 중인 스레드 등이 해당하며, GC는 이 루트 참조 객체들에서 시작해 연결된 객체를 모두 Mark하고, 그외 도달하지 않는 객체들은 제거(Garbage Collection)합니다.

#### Young Generation과 Old Generation의 객체는 언제 GC의 대상이될까?
각 영역에서 GC 발생 시점에 GC ROOT에 저장되어있는 객체와 GC ROOT의 객체와 연결되어있는 객체를 제외한 모든 객체를 Garbage 처리를 하게됩니다.

#### Young Generation에서 Old Generation으로 넘어가는 동작 원리
Young Generation에서 Minor GC 발생 시점에  eden, survivol 0과 1 영역에서 GC ROOT에 저장되어있는 객체와 GC ROOT의 객체가 참조하고 있는 객체를 제외한 모든 객체를 Garbage 처리를 하게되고 살아남은 객체는 survivol 영역에 저장되게 됩니다. 각각의 survivol 영역에서 존재하다가 일정 회수 이상 살아남게되면 Old Generation으로 넘어가게됩니다.

### GC가 되지 않는 루트 참조 객체는 크게 3가지
<img width="669" alt="image" src="https://github.com/user-attachments/assets/ea61a3b0-79e6-4a3c-ae1e-bcc4ba477233" />

Method Area의 Static 변수  
모든 현재 자바 스레드 Stack Area 내의 지역 변수 및 매개 변수에 의한 객체 참조  
Native Method Stack 영역에서 JNI(Java Native Interface) 프로그램에 의해 동적으로 만들어지고 제거되는 JNI global 객체 참조  

#### Out Of Memory가 발생한다면 어떻게 대처 대응 할 수 있을끼?



#### 출처
https://junghyungil.tistory.com/133
https://junghyungil.tistory.com/8?category=892275
https://junghyungil.tistory.com/9?category=892275
