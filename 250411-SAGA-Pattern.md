# SAGA 패턴이란
SAGA 패턴이란 마이크로서비스 환경에서 분산 트랜잭션을 처리할 때 사용하는 소프트웨어 설계 패턴입니다.  
마이크로서비스아키텍처(MSA)에서는 서비스 간 데이터가 독룁된 데이터베이스로 나눠져 있습니다.  
이때 여러 서비스에서 동시에 변경이 일어날 때, 데이터의 일관성 이슈가 발생합니다.  

## 오케스트레이션(Orchestration) 방식
오케스트레이션 방식은 SAGA 패턴을 구현할 때, 트랜잭션의 모든 단계를 중앙에서 제어하는 별도의 오케스트레이터(Orchestrator)를 두는 방식입니다.  
즉, 오케스트레이터는 트랜잭션의 상태를 관리하고, 각 서비스에게 순차적으로 요청을 보내 작업을 수행하도록 하며, 오류 발생 시 보상(compensation)을 관리합니다.  
<img width="244" alt="image" src="https://github.com/user-attachments/assets/00ac381a-cdb9-4f04-a17c-3a38816a17b0" />  
  
<img width="396" alt="image" src="https://github.com/user-attachments/assets/53285638-9e79-4de6-9485-a51e0d746e6f" />  

#### 보상(Compensation) 작업이란?
보상 작업(Compensation)은 트랜잭션의 일부 단계가 실패했을 때, 그 이전 단계에서 이미 수행된 작업을 다시 되돌리는 작업을 의미합니다.  
실패가 발생한 시점까지 진행되었던 작업을 취소(Undo) 하는 작업입니다.  
일반적인 트랜잭션에서 이야기하는 롤백과 비슷한 개념이지만 완전히 같지는 않습니다.

#### 롤백(Rollback)과 보상(Compensation)의 차이
롤백(Rollback)  
	•	일반적인 데이터베이스 트랜잭션(ACID 기반)에서 사용하는 개념입니다.  
	•	트랜잭션 내의 모든 작업이 완벽히 원상태로 되돌려집니다.  
	•	트랜잭션 중간 단계의 데이터가 다른 트랜잭션에게 노출되지 않으며, 완전히 이전 상태로 복원됩니다.  
 
<img width="229" alt="image" src="https://github.com/user-attachments/assets/20a9b723-22c2-49cd-8fd2-c2228bcd3de6" />  

보상(Compensation)  
	•	SAGA 패턴 등 분산 트랜잭션에서 사용하는 개념입니다.  
	•	여러 단계의 작업이 독립된 트랜잭션으로 처리되고 완료된 상태이기 때문에, 롤백과 달리 이전 상태를 자동으로 복원하지 않습니다.  
	•	보상 작업은 “이미 완료된 작업의 영향을 반대로 수행하는 새로운 작업“을 의미합니다.  
  
<img width="302" alt="image" src="https://github.com/user-attachments/assets/43cfc591-e9d6-41f0-b46b-36428b40c1a3" />    
  
즉, 보상 작업은 “논리적으로” 이전 작업을 되돌리는 추가 작업을 수행하는 방식입니다.

![image](https://github.com/user-attachments/assets/56113ff3-1505-4ae6-ab59-18a470bc90e5)


## 코레오그래피(Choreography) 방식
코레오그래피 방식은 **중앙 제어자 없이 서비스 간에 메시지 또는 이벤트를 통해 자율적으로 처리하는 방식** 입니다.       
서비스가 이벤트를 발행한 후 각 서비스들에게 이벤트를 발행 서비스들 중 실패한 이벤트가 있다면 해당 서비스에서 실패 이벤트를 발행한 후 다른 서비스에게 이벤트를 발행 이벤트를 받은 서비스는 보상 작업을 수행하는 방식  
① 서비스가 자신의 작업 완료 후 이벤트 발행 → ② 이벤트를 받은 서비스들이 독립적으로 처리 → ③ 실패가 발생하면, 실패 이벤트를 발행 → ④ 실패 이벤트를 받은 서비스들이 보상 작업 수행
![image](https://github.com/user-attachments/assets/927bebe9-3f9c-450d-b424-50a9012df1c0)

**코레오그래피 방식에서의 이벤트**  
	•	이벤트는 시스템 내에서 일어난 의미 있는 상태 변경을 알리는 메시지입니다.  
	•	이벤트는 일반적으로 단방향으로 전달됩니다. (발행자 → 수신자)  
	•	한 번 발행된 이벤트는 수정되지 않고, 새로운 이벤트로 처리됩니다.  



## 2PC(Tow-Phase Commit)란?
2PC(Two-Phase Commit)란, 분산 환경에서 데이터 일관성을 강력하게 보장하기 위한 분산 트랜잭션 프로토콜입니다.  
여러 서비스(노드) 간에 트랜잭션이 일어날 때 모두 성공하거나 모두 실패하도록 보장하는 방식입니다.  

2PC 프로토콜은 **코디네이터(Coordinator)**와 **참여자(Participant)**로 구성되어 있습니다.

이 프로토콜은 크게 두 단계로 이루어져 있습니다.  

✅ Phase 1: Prepare (준비 단계)  
Coordinator 가 모든 Participant에게 커밋 가능한지(Commit Request)를 물어봅니다.  
Participant 는 트랜잭션 처리가 가능한지 확인 후, 가능한 경우 "준비 완료(Prepared)" 메시지를 보내고, 불가능하면 "실패(Abort)" 메시지를 보냅니다.  
모든 참여자가 "준비 완료" 응답을 보낼 때만 2단계로 넘어갈 수 있습니다.  
  
✅ Phase 2: Commit (커밋 단계)  
모든 Participant가 "준비 완료"를 응답했다면, Coordinator는 트랜잭션을 "Commit(커밋)"하라고 지시합니다.  
만약 하나라도 "실패" 응답을 보낸 경우, Coordinator는 전체 트랜잭션을 "Abort(중단)"하라고 Participant들에게 지시합니다.  
즉, 모두 성공할 수 있을 때만 Commit이 이루어집니다.  
  
<img width="335" alt="image" src="https://github.com/user-attachments/assets/36d6e3a1-852c-431d-bbc9-a22086f94c8a" />  

    
2PC 단점
성능 저하: 참여자의 응답을 기다리는 동안 블로킹(Block) 상태가 발생하여 성능이 떨어질 수 있습니다.  
단일 장애점(Single point of failure): Coordinator에 문제가 발생하면 시스템이 중단될 수 있습니다.  
장애 복구가 복잡: Coordinator 또는 Participant의 장애 시 복구가 어려워질 수 있습니다.  
