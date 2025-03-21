### 1.Array는 어떤 자료구조인가요?
 Array는 연관된 data를 메모리상에 연속적이며 순차적으로 미리 할당된 크기만큼 저장하는 자료구조입니다. 
배열의 장점으로는 조회와 마지막인덱스에 값을 append하는 명령이 빠르다는 것입니다.
조회를 자주해야하는 작업에 적합한 것 같습니다.
배열의 단점으로는 고정된 배열의 크기를 가져가야하는 만큼 낭비되는 메모리가 많을 수도 있다는 것입니다.
또는 추가적인 오버헤드가 발생할 수 있습니다.

#### 1-2 그럼 array는 왜 조회와 마지막 요소에 값을 추가하는게 빠른가?
배열의 경우 첫번째 인덱스의 주소의 값을 알게되면 데이터가 순차적으로 저장되어있을 것이기 때문에 n 번째 인덱스에 접근하다고 했을 때 첫번째 인덱스 주소 + (n번째 인덱스 * 데이터크기)의 값을 구하게되면
빅오 1의 시간으로 오퍼레이션을 수행할 수 있게됩니다. 중간의 요소를 조회하는 것도 마찬가지이지만 삽입 삭제의 경우에는 데이터를 삽입 삭제시 해당 데이터 인덱스 기점으로 왼쪽이나 오른쪽으로 쉬프트를 해줘야하므로
빅오 n의 시간을 가지게됩니다.

#### 1-3 추가적인 오버헤드는 왜 발생하는가
배열의 경우에는 size를 넘어서게되면 저장을 할 수 없지만 동적배열의 경우에는 size를 넘어선 데이터를 저장하게되면은 배열을 resize하게됩니다.
기존배열의 1.5배나 2배정도되는 가량의 배열을 새로 생성하고 기존 배열의 데이터를 새로 생성된 배열에 다시 할당하게됩니다. 그리고 기존 배열은 삭제하게됩니다.
이 과정에서 추가적인 리소스를 사용하게합니다. 
	
### 2.Linked List는 어떤 자료구조인가요
LinkedList는 각 노드마다 데이터를 저장하고 다음 노드의 주소를 참조하는 형태를 가진 물리적으로는 비연속적이나 논리적으로는 연속적 자료구조입니다.
또한 데이터가 추가되는 시점에 메모리를 할당하기 때문에 메모리를 좀 더 효율적으로 사용할 수 있다는 장점이 있습니다.

### 3.Array와 LinkedList의 차이점은 무엇인가요?
배열은 메모리상에 미리 할당된 크기만큼 데이터를 순차적으로 저장하는 자료구조이지만 LinkedList는 각 노드에 데이터가 저장되고 각 노드가 다음 노드의 주소를 참조하는 형태입니다. 
그런만큼 메모리도 동적으로 할당됩니다.
배열은 중간에 요소를 삭제할 필요성이 적거나 빠른 데이터 접근이 필요한 경우 많이 사용될 수 있을 것 같고, 연결리스트의 경우에는 데이터의 수정이 빈번한 경우 사용을 고려할 수 있을 것 같습니다.

### 4.미리 예상한 것보다 더 많은 size의 데이터를 저장하느라 Array의 사이즈를 넘어서게되었습니다. 이 때 어떻게 해결할 수 있을까요?
 고정된 배열을 사용하는 것보다 동적으로 저장하는 arrayList를 사용하면됩니다. 이런 것을 동적 배열이라고 하는데 
 동적배열은 기존의 배열의 사이즈를 넘어서게되면 기존의 배열보다 더 큰 사이즈의 배열을 선언하게되고 기존의 배열에 있던 데이터를 옮겨서할당합니다.
 또 다른 방식으로는 size를 예측하기 어렵다면 linkedlist를 사용하는 것도 하나의 방법 같습니다.
 
### 5.Dynamic Array는 어떤 자료구조인가요?
Array의 경우에는 고정된 크기의 값을 가지므로서 크기 이상의 데이터가 왔을 때 데이터를 저장할 수 없습니다.
하지만 Dynamic Array의 경우에는 Array의 size보다 큰 데이터를 저장해야할 때 기존 배열보다 큰 배열을 선언합니다.
이 후에 기존의 배열에 있던 데이터를 새로 생성한 배열에 옮겨서 할당합니다. 그리고 기존의 배열은 삭제하는 과정이 발생합니다.
이런 식으로 Dynamic Array의 경우에는 저장공간이 차게되면 resize를 하여 유동적으로 size를 조절하여 데이터를 저장하는 자료구조입니다.
 
### 6. Dynamic Array에서 사이즈가 꽉차서 resize하는 과정은 얼만큼의 시간이 걸리게 될까요?
빅오 n의 시간만큼 걸릴 것으로 생각이 듭니다. 예를들어 기존 배열의 사이즈가 10만큼이고 새로운배열에 할당하면 기존 배열의 크기만큼 새로운 배열에 할당해야하므로 10만큼의 시간이 걸릴 것 입니다.
  
### 7. Dynamic Array와 LinkedList의 차이를 알려주세요
 둘 다 메모리를 동적으로 사용한다는 것은 비슷하지만 동적배열 같은 경우에는 낭비되는 메모리가 있을 수 있지만 연결리스트 같은 경우에는 낭비되는 메모리가 없을 것 입니다.
 데이터를 변경하는 부분에서도 많은 차이가 있을 것입니다. 예를 들어 동적 배열의 경우에는 인덱스의 마지막요소나 배열의 요소의 조회하는 경우에는 빅오의 일의 시간복잡도를 가지게될 것이지만,
 링크드리스트는 각 노드가 다음 노드의 주소를 참조하는 형태를 가지고 있기 때문에 순차적으로 조회해야하는 특성이 있습니다. 그래서 빅오 n의 시간복잡도를 가지는 차이가 있습니다.
 만약 링크드리스트가 다음노드의 주소만 참조하는게 아니라 이전노드의 주소도 참조하고 있다면 인덱스 마지막요소에 추가하는 부분은 동적 배열과 같은 시간복잡도를 가지게 될 것입니다.
 
 그렇다면 동적배열의 단점으로는 중간 데이터를 삽입 삭제 할 경우에는 시간복잡도 n의 시간을 가집니다. 그 이유로는 배열에 경우에는 순차적으로 데이터가 저장되어있는데 중간에 데이터를 삭제하거나 삽입하게되면
  그 이후의 모든 데이터를 왼쪽이나 오른쪽으로 쉬프트를 해야하기 때문입니다.

그에 반해 링크드리스트의 경우에는 데이터를 삽입 삭제 할때 노드의 이전노드와 다음노드의 주소만 알게되면 빅오 1의 시간복잡도를 가지게됩니다.
다만, 삽입 삭제할 노드를 조회를 해야하므로 빅오 n의 시간복잡도를 가지게됩니다. 총 결과적으로는 링크드리스트의 데이터를 삽입, 삭제시 빅오 n의 시간 복잡도를 가지게 될 것입니다.
	
### 8. array와 linkedlist의 memory allocation (메모리 할당)은 언제 발생하며 메모리의 어느 영역을 할당 받나요?
 array는 compile 단계에서 stack memory에 할당을 하게되고 정적 메모리할당이라고 합니다.
 linkedlist의 경우 runtime 단계에서 새로운 노드를 추가할 때마다 heap memory에 할당되게 됩니다. 이를 동적메모리 할당이라고 합니다.
 
 
### 9. queue는 어떠한 자료구조인가요?
 queue는 선입선출(First in First out)의 자료구조입니다. enqueue, dequeue 빅오 1의 시간복잡도를 가지게됩니다.
 
### 10. queue의 구현방식의 종류와 각 구현방식을 설명해주세요
queue의 구현방식으로는 array-based queue와 list-based queue가 있습니다. 
array-based queue의 경우에는 circular queue로 구현하는게 일반적입니다. 이는 메모리를 효율적으로 사용하기 위함입니다.
list-based queue의 경우에는 singly-linked list(단일 연결 리스트)로 구현을 합니다.

### 11. circular queue는 어떠한 자료구조인가요? 
circular queue의 경우에는 array-based queue를 효율적으로 확장한 자료구조입니다.
배열의 앞과 끝이 순환되는 구조를 가지고 있습니다. 일반적인 배열기반 큐에서는 dequeue를 했을 경우 배열의 앞쪽에는 공간이 비어 메모리를 낭비가 됩니다.
원형 큐에서는 이를 해결하기위해 2개의 포인터를 두고 이를 first와 rear라고 하겠습니다. first 부분에는 값을 dequeue할 위치를 가리키고 rear 부분에는 값을 enqueue할 위치를 가리킵니다.
데이터가 삽입되거나 삭제될 때 배열의 끝으로 가게되면 다시 배열의 처음으로 돌아오게되는 자료구조입니다.

#### 11-1. circular queue를 동적배열로 구현하고 resize시 동작 구현은 어떻게 되나요?
queue의 저장공간이 모두 차게되면 새로운 더 큰 사이즈의 queue를 할당하고 front 포인터를 시작으로 새로운 queue에 순차적으로 저장하게됩니다. 

### 12. stack은 어떠한 자료구조인가요?
stack은 후입선출(Last in First out)의 자료구조입니다. 
	
### 13. queue와 prioriy queue에 대해서 설명해주세요
queue는 선입선출 구조로 먼저 들어온 enqueue 된 값이 dequeue가 되는 자료구조입니다. queue의 시간복잡도는 O(1)의 시간복잡도를 가지게 됩니다.
prioriy queue는 우선순위 조건에 해당 되는 값이 먼저 dequeue가 되는 자료구조입니다. prioriy queue의 삽입 삭제는 O(logn)의 시간복잡도를 가집니다.

prioriy queue는 heap 자료구조로 구현됩니다. prioriy queue는 물리적으로는 배열의 형태를 가지고 있지만 논리적으로는 이진완전트리의 구조를 가지고 있습니다.
그러면 배열로 이진완전트리의 구조를 어떻게 구현하냐면 부모노드의 왼쪽 자식노드는 2n의 인덱스를 가지게되고 오른쪽 자식노드는 2n+1의 인덱스를 가지게 됩니다.
반대로 자식노드는 부모노드를 n/2의 인덱스로 접근할 수 있습니다. 

prioriy queue의 값을 삽입하게 되면 queue의 마지막 인덱스에 삽입하게 되고 부모노드(n/2)와 비교를 하게됩니다. maxheap이나 minheap 의 조건에 따라 요소의 값을 스왑하게됩니다.
결론적으로 logn의 시간복잡도를 가지게됩니다.

### 14. **중요 BST(Binary Search Tree) 이진탐색트리는 어떤 자료구조인가요?
이진탐색트리는 정렬된 트리입니다. 부모 노드를 기준으로 왼쪽 노드는 부모 노드보다 작은 값의 노드들로 구성되어있고 오른쪽 노드는 부모 노드 기준으로 큰 값들로 구성되어있습니다. 
이진탐색트리는 저장시에 정렬이 되는 자료구조입니다. 조회 삽입 삭제 모두 logn의 시간복잡도를 가지고있습니다. 

BST에서 조회, 삽입, 삭제를 하게된다면 루트 노드 기준으로 탐색하고자 하는 값이 작다면 왼쪽으로 크다면 오른쪽으로 각 노드와 비교하여 탐색할 것입니다.
BST에서 삽입를 하게된다면 루트 노드부터 서브트리까지 순차적으로 비교 후 마지막 레벨에 저장되게 될 것입니다.
BST에서 삭제를 하게된다면 삭제 요소를 탐색 후 삭제 요소의 노드의 오른쪽 가장 작은값을 삭제요소의 노드 자리에 위치시킵니다.

#### 14-1 BST의 worstcase는 log(n)입니다. 어떠한 경우에 worstcase가 발생하나요?
이진탐색트리의 루트노드를 기준으로 한쪽으로 치우쳐져 있을 때 발생합니다. 
 
#### 14-2 worstcase의 해결방법으로는 무엇이있을까요?
AVL트리와 Red-Black tree를 사용하면 된다는 것으로 알고있습니다.
	 
### 15. ** 중요 Hash Table은 어떤 자료구조인가요?
Hash Table은 key와 value로 구성된 자료구조입니다. key의 값을 hash 함수로 연산하여 얻은 해시의 값을 얻고 해시의 값으로 hash table의 인덱스로 사용합니다.
조회, 삽입, 삭제시에 빅오 1의 시간복잡도를 가지는 자료구조입니다. 
	
#### 15-1 좋은 해쉬함수는 무엇인가요?
연산속도가 빨라야하며 해쉬의 키값이 최대한 중복으로 나오지 않는 것이 좋은 해쉬함수입니다.
	   
### 16. **** 중요 HashTable에서 Collision이 발생하면 어떻게 되나요? 해결방법에는 무엇이 있을까요?
	
해쉬테이블에서 충돌이 발생하게되면 open addressing과 Chaining 방법을 사용하면됩니다.
open addressing이란 충돌이 발생하게되면 hashTable의 빈 슬롯을 찾아 저장하는 방법입니다. 
새로운 빈 슬롯을 찾는 방법으로는 선형탐사(Linear Probing)와 제곱탐사(Quadratic Probing)와 이중해싱(Double Hashing) 방법이 있습니다.
선형 탐사의 경우 충돌이 났을 경우 인덱스에서 +1 +2 +3 순으로 하나씩 탐색하는 방법을 말합니다.
제곱 탐사의 경우 충돌이 났을 경우 인덱스에서 1^2 2^2 3^2 순으로 탐색하는 방법을 말합니다.
이중 해싱의 경우에는 충돌이 났을 경우 두번 째 해시함수를 이용해 나온 키 값으로 순회하는 방법을 말합니다.
오픈 어드레싱 방법을 사용하면 클러스터링 현상이 발생할 수 있습니다.

**자료구조에서 클러스터링이란 충돌처리과정에서 데이터가 일정 구역에 밀집되는 현상을 말합니다.

체이닝 방법은 충돌이 발생했을 경우 해시테이블의 인덱스마다 LinkedList를 사용하여 동일한 인덱스의 LinkedList에 데이터를 추가하는 방법을 말합니다.
worstCase의 경우 O(n)의 시간복잡도를 가집니다. 
	
#### 16-1 체이닝 방법의 worstcase에는 O(n)의 시간복잡도를 가진다고했는데 어떠한 상황인가요?
정말 희박하지만 n의 모든 요소들이 해시값이 동일하다고 했을 때 하나의 인덱스의 연결리스트에 모두 저장되는 상황을 말합니다. 
그럴 경우 연결리스트의 요소를 탐색해야하므로 O(n)의 시간복잡도를 가진다고 할 수 있습니다.
  
#### 16-2 이중해싱이 무엇인지 설명해주세요
이중 해싱의 경우에는 충돌이 났을 경우 두번 째 해시함수를 이용해 나온 값으로 탐사의 폭을 설정하는 것을 말합니다.
