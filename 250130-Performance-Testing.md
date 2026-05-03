### 성능테스트 도구 
nGrinder, (변경)Scouter -> ElasticAPM  

### 요소
Throughput - 처리량
TPS(Transaction Per Seconds) - 1초당 작업량  
Latency - 지연 시간(요청에 대한 응답 시간) 

### 성능 테스트로 성능 개선 방법
1. Throughput 과 Latency 를 활용해 설정 ex) 목표 Throughput, 평균 Latency 설정  
2. 현재 시스템이 어느 정도 트래픽까지 견딜 수 있는지 부하 테스트 진행  
3. 병목 지점 파악 후 성능 개선  * 병목 지점의 Througput이 곧 전체 Throughput이다.  
4. 개선한 시스템이 어느 정도 트래픽까지 견딜 수 있는 지 부하 테스트 진행   
5. 목표 Throughput, Latency가 될 때까지 병목 지점  파악 후 부하테스트 반복 진행  

### 적절한 부하 테스트 시간
1분 간격으로 기록되는 모니터링 도구를 가지고 정확하게 성능을 측정하려면 최소 5분간의 부하테스트는 진행해야 한다. 그래야 일관된 결과값을 얻을 수 있다.

### 프로덕션 환경과 비슷한 데이터 셋팅
데이터베이스는 데이터가 어떻게 저장되어 있는 지와 얼마나 많은 양이 저장되어 있는 지에 따라서 성능 차이가 많이 나므로 실제 프로덕션 환경과 비슷하게 데이터양을 셋팅을 해두고 부하 테스트를 해야한다.
그리고 각각의 아키텍처(EC2, RDS)  CPU와 메모리 사용률이 100퍼인 곳을 확인. 해당 지점이 병목 지점

### nGrinder
#### nGrinder 구성 요소
컨트롤러 - 성능 테스터가 테스트 스크립트를 생성하고 테스트 실행을 구성할 수 있도록 하는 웹 애플리케이션  
에이전트 - 부하를 생성하는 가상 사용자 생성기  

#### nGrinder 실행 
nGrinder 실행 java -jar ngrinder-controller-3.5.9-p1.war 실행
run_agent 실행 후 http://localhost:8080 으로 설정되어있음

### Elastic APM
#### Elastic APM 구성 요소
Elasticsearch – APM 데이터를 저장하는 데이터베이스  
Kibana – APM 데이터를 시각화하는 도구  
APM Server – APM 데이터를 수집하여 Elasticsearch에 전송  
APM Agent – 애플리케이션에서 성능 데이터를 수집하여 APM Server로 전송   

#### Elastic APM 시스템 변수 설정
-javaagent:elastic-apm-agent-1.40.0.jar  
-Delastic.apm.service_name=my-spring-app  
-Delastic.apm.server_urls=http://localhost:8200  
-Delastic.apm.environment=development  
-Delastic.apm.secret_token=  
-Delastic.apm.application_packages=com.example  

### Scouter
#### Scouter 시스템 변수 설정
-javaagent → Scouter 에이전트 적용 :: -javaagent:/Users/gimmingyu/MyProjects/util/scouter/agent.java/scouter.agent.jar
-Dscouter.config → 설정 파일 지정 :: -Dscouter.config=/Users/gimmingyu/MyProjects/util/scouter/server/conf/scouter.conf 
-Dobj_name → Scouter UI에서 표시될 이름 설정 :: -Dscouter.config=/Users/gimmingyu/MyProjects/util/scouter/server/conf/scouter.conf 
--add-opens → Java 9 이상에서 내부 클래스 접근 허용 :: --add-opens java.base/java.lang=ALL-UNNAMED

#### Scouter 실행 
/Users/gimmingyu/MyProjects/util/scouter/server/startup.sh 실행 
/Users/gimmingyu/MyProjects/util/scouter/agent.host/host.sh 실행

### 부하 테스트 시나리오 유형
| 유형 | 목적 | 부하 패턴 |
|---|---|---|
| Load Test | 목표 트래픽에서 정상 동작 확인 | 일정 부하를 일정 시간 |
| Stress Test | 한계점·장애 시작 지점 찾기 | 부하를 점진적으로 증가시키며 break point 탐색 |
| Spike Test | 갑작스러운 트래픽 급증 대응 | 짧은 시간 동안 대량 부하 후 정상화 |
| Soak Test | 장시간 운영 시 메모리 누수·자원 고갈 확인 | 평균 부하를 수 시간~수 일 |
| Capacity Test | 인프라 용량 산정 | 부하를 단계적으로 늘리며 TPS 한계 측정 |

각 단계는 보통 다음 순서로 수행합니다: Load → Stress → Capacity → Spike → Soak.

### 결과 해석 핵심 지표
- **응답 시간 분포**: 평균만 보면 안 되고 **p95, p99**(상위 5%, 1% 응답시간)를 봐야 함. 평균은 빠르지만 일부가 매우 느린 long-tail을 놓치기 쉬움.
- **에러율**: 1% 이상 에러는 사용자 체감으로 명확히 보이는 수준. 부하 테스트 통과 기준에 반드시 포함.
- **TPS와 응답시간의 관계**: 부하가 늘어날 때 TPS는 일정 지점까지 선형 증가하다가 평탄해지고, 응답시간이 급격히 증가하기 시작하는 지점이 시스템 한계.
- **자원 사용률**: CPU 70~80% 정도가 적정. 100% 도달 = CPU 병목, 50% 미만인데 처리량이 안 늘어남 = I/O 또는 락 병목.

### 도구 비교
| 도구 | 부하 생성 | 모니터링 | 사용 시점 |
|---|---|---|---|
| nGrinder | O (분산 에이전트) | 자체 대시보드 | 한국 환경에서 친숙, 무료 |
| JMeter | O | 자체 GUI/CLI | 가장 범용적, 플러그인 풍부 |
| k6 (Grafana) | O | Grafana 연동 | 스크립트(JS)로 작성, CI 연동 쉬움 |
| Locust | O | 자체 웹 UI | Python 스크립트, 분산 부하 |
| Gatling | O | HTML 리포트 | Scala/Java DSL, 정밀한 리포트 |
| Elastic APM | X | O (트랜잭션 추적) | 부하 도구와 함께 사용해 분석 |
| Scouter | X | O (실시간 모니터링) | 한국 환경 친숙, 경량 |
| Pinpoint | X | O (분산 추적) | 마이크로서비스 디버깅 |

부하 도구(부하 생성)와 APM(모니터링)은 역할이 다르므로 **함께** 씁니다. 예: nGrinder로 부하를 주면서 Elastic APM이나 Scouter로 트랜잭션 추적.

### 스테이징 환경 테스트 전략
프로덕션과 동일한 사양·데이터양·외부 의존성을 갖춘 스테이징에서 테스트하는 것이 이상적입니다. 비용 때문에 실제와 동일하게 만들기 어렵다면 다음 우선순위로 맞춥니다:
1. **데이터 규모**: 인덱스 효율성과 쿼리 시간이 데이터양에 매우 민감. 프로덕션 데이터를 익명화해 복제.
2. **인스턴스 사양**: CPU/메모리 동일. 작은 사양에서 측정한 결과를 단순 곱셈으로 추정하면 안 됨.
3. **외부 시스템**: 결제 PG, 외부 API는 mock으로 대체하되 응답시간은 비슷하게.
4. **네트워크 위치**: WAS와 DB가 같은 AZ에 있는지 등 latency 영향 큼.

### 도구별 선택 의사결정
- 한국 환경, 학습/사내 PoC → **nGrinder + Scouter**
- 글로벌 표준, 플러그인 확장 → **JMeter**
- CI/CD에 통합하고 싶다 → **k6**
- 정밀한 리포트와 시나리오 → **Gatling**
- 분산 마이크로서비스 추적 → **Pinpoint** (모니터링)

