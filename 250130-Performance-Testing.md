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


	
	
