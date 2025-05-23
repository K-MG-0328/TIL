### NIO가 생긴 이유는 무엇인가요?
스레드의 자원의 효율성 문제 때문입니다.
기존의 Blocking I/O의 경우에는 I/O 작업이 끝날 때까지 스레드를 대기 상태로 만들어 스레드 낭비가 발생하였습니다.
하지만 Non-blocking I/O의 경우에는 I/O 작업 시 채널의 데이터를 확인한 후 데이터가 없다면 스레드를 대기 상태로 만들지않고 다음 코드를 진행합니다.
이 후에 Selector에 등록된 채널들을 모니터링하면서 이벤트가 발생하면 해당 채널에서 데이터의 입출력을 처리합니다. 
이렇게 함으로써 한정된 스레드를 낭비하지않게되고 스레드가 대기함으로써 생기는 메모리 낭비라든가 잔여 스레드끼리의 컨텍스트 스위칭에 대한 비용 발생으로 인한 문제들이 해결됩니다.

###  NIO에서 Channel의 용도는 무엇인가요?
I/O 작업에 대한 데이터 데이터 통로를 생성합니다.

###  NIO에서 Buffer의 용도는 무엇인가요?
버퍼는 데이터를 일시적으로 저장하기 위한 메모리 블록입니다.
채널에서 읽은 데이터를 버퍼에 저장되고 애플리케이션은 버퍼에서 데이터를 읽습니다.

###  NIO에서 Buffer의 상태를 확인하기 위한 메소드들에는 어떤 것들이 있나요?
position() - 데이터를 읽거나 쓰는 현재 위치
limit() - 읽기/쓰기가 가능한 데이터의 끝
capacity() - 버퍼의 최대크기

###  NIO는 어떻게 스레드가 대기하지 않고 작업을 계속할 수 있게 설계가 되었나요?
Non-blocking I/O 방식을 도입하여 스레드가 대기하지 않고 작업을 계속 할 수 있도록 설계되었습니다.
Non-blocking I/O는 Non-blocking 모드에서는 데이터가 준비되지 않았을 때, 작업을 중단하지않고 바로 다음 코드로 진행합니다.
채널을 Selector에 등록하면 스레드는 채널에서 이벤트가 발생했을 때만 작업을 수행합니다. 즉, 스레드는 데이터가 준비되지 않은 상태에서 기다리지 않고 다른 작업을 계속 수행할 수 있습니다.

### Blocking과 Non-blocking I/O의 차이는 그리고 Non-blocking이 가능한 이유는?
Blocking과 Non-blocking I/O의 가장 큰 차이는 스레드가 대기 상태로 전환되느냐 안 되느냐의 차이입니다. 이 차이는 I/O 작업 호출 시 데이터를 반환하느냐 그렇지 않느냐에 의해 결정됩니다.
예를 들어 Blocking에서는 대기가 발생하는 이유가 I/O 작업을 요청하게되면 데이터를 읽어올 때까지 반환 값을 반환하지 않기 때문입니다.
그러나 Non-blocking에서는 I/O 작업을 요청하게되면 디스크에서 데이터 읽음 여부에 따라 채널에서 반환값을 반환하기 때문입니다.
그렇게되면 스레드는 다음 작업을 하게되고 대기 상태에 빠지지 않게됩니다.
Non-blocking 모드에서 Selector는 등록된 채널들을 모니터링하여 특정 채널에서 데이터가 준비되었을 때 발생하는 이벤트를 감지합니다. 
그리고 이벤트가 발생한 채널을 반환하며 반환된 채널에서 데이터를 읽거나 쓰는 작업은 애플리케이션이 수행합니다.

### 대기 상태의 스레드가 문제가 되는 이유는 무엇인가요?
스레드는 제한된 리소스입니다. OS, VM에서는 스레드에 제한이 있습니다.
대기 상태의 스레드는 CPU를 사용하지 않지만 메모리 등 다른 시스템 리소스를 점유하게됩니다. 
그리고 스레드들이 대기 상태에 들어가게되면 남은 스레드로 컨텍스트 스위칭이 증가하게되는데 이것은 시스템 성능 저하의 원인이 됩니다. 

### Selector는 어떻게 커널부분을 모니터링 할 수 있는가?
Selector는 운영 체제의 멀티플렉싱 메커니즘(epoll, kqueue 등)을 호출하여 커널에서 발생하는 이벤트를 감지합니다.
커널은 파일 디스크립터의 상태를 감시하고, 이벤트가 발생했을 때 이를 Selector에 알립니다.

### 운영 체제의 I/O 멀티플렉싱 기능이란?
I/O 멀티플렉싱(I/O Multiplexing)은 운영 체제가 여러 I/O 자원(파일 디스크립터, 소켓 등)을 비동기적으로 감시하고, 준비된 자원만을 선택적으로 처리할 수 있도록 제공하는 기능

### Blocking IO 동작 방식
CPU가 장시간 소요되는 I/O 작업을 처리할 때 어떻게 처리되는가?
I/O 작업을 요청한 프로세스는 대기 상태로 전환 후 CPU는 운영체제의 스케줄러에 의해 대기 중인 다른 프로세스나 스레드를 실행하고 실행 가능한 다른 작업이 없다면, CPU는 유휴 상태로 전환한다. 이 후 I/O 작업이 완료되면, 해당 장치(예: 디스크 컨트롤러)가 인터럽트(Interrupt)를 발생시켜 CPU에 작업 완료를 알린다. CPU는 인터럽트를 처리하기 위해 현재 실행 중인 작업을 일시 중단하고, 인터럽트 핸들러를 실행한다. I/O 작업이 완료되면, 운영체제는 대기 상태에 있던 프로세스를 준비 상태(ready state)로 전환하고 cpu 스케줄러에 의해 해당 프로세스는 다시 cpu를 할당받아 실행된다.   
