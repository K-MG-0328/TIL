## 메모리 누수(Memory Leak)란
CS의미로 살펴 볼 때 컴퓨터 프로그램이 필요하지 않은 메모리를 계속 점유하고 있는 현상이며 자바에서는 더 이상 사용되지 않는 객체들이 가비지 컬렉터에 의해 회수되지 않고 계속 누적이 되는 현상을 말한다.
Old 영역에 계속 누적된 객체로 인해 Major GC가 빈번하게 발생하게 되면 프로그램 응답속도가 늦어지면서 성능 저하를 불러온다. 이는 결국 OutOfMemory Error로 프로그램이 종료되게 된다.

### 루트 참조 객체(Root Reference Object)란
Java의 가비지 컬렉션에서 객체가 접근 가능한 시작점 역할을 하는 객체를 의미합니다. 


### GC가 되지 않는 루트 참조 객체는 크게 3가지
#### Static 변수에 의한 객체 참조 
#### 모든 현재 자바 스레드 스택내의 지역 변수, 매개 변수에 의한 객체 참조
#### JNI 프로그램에 의해 동적으로 만들어지고 제거되는 JNI global 객체 참조 



#### 출처
https://junghyungil.tistory.com/133
https://junghyungil.tistory.com/8?category=892275
https://junghyungil.tistory.com/9?category=892275
