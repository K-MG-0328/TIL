### 커밋메시지 작성요령
    •	<type>: 커밋의 유형을 나타냅니다.
      	feat: 새로운 기능 추가
      	fix: 버그 수정
      	docs: 문서 변경
      	style: 코드 포맷팅 등 기능 변화가 없는 변경
      	refactor: 코드 리팩토링 (기능 변화 없음)
      	test: 테스트 코드 추가/수정
      	chore: 빌드, 설정 파일 변경
      	perf: 성능 개선
      	ci: CI/CD 관련 변경
  	•	<scope> (선택 사항): 영향을 받는 모듈, 기능, 파일 등을 나타냅니다.
  	•	<subject>: 간단한 변경 요약 (50자 이내, 소문자로 시작).
  	•	<body> (선택 사항): 변경한 이유나 상세 설명 (한 줄 띄우고 작성).
  	•	<footer> (선택 사항): 브레이킹 체인지(BREAKING CHANGE)나 이슈 링크.

feat(board): 조회수 증가 기능에 Redis 캐싱 추가
- Redis를 사용해 조회수 데이터를 캐싱하도록 구현
- 키에 TTL 7일 설정 추가
- 매 분마다 데이터베이스와 동기화

fix(view): 조회수 캐싱 시 Redis 키 포맷 오류 수정
- Redis 키 생성 시 잘못된 형식 문제 해결
- boardId 변환 오류로 인해 발생한 NumberFormatException 수정

docs(redis): Redis 설정 관련 문서 추가
- README에 Redis TTL 설정 방법 추가
- 캐싱 정책 및 데이터 구조 설명 포함

refactor(service): Redis 조회수 동기화 로직 분리
- Redis 동기화 로직을 RedisViewCountService로 이동
- 서비스 계층의 책임을 분리하여 가독성 및 유지보수성 개선

perf(cache): Redis 조회수 캐싱 성능 최적화
- Redis 키 스캔 범위를 줄이기 위해 패턴을 개선
- 데이터베이스 동기화 과정에서 불필요한 쿼리 제거

style(board): Redis 관련 코드 포맷 정리
- 메서드 간 공백 정리
- 변수명 통일: boardId -> board_id

test(redis): Redis 조회수 캐싱 테스트 코드 추가
- TTL 설정 테스트 추가
- 데이터베이스 동기화 로직 검증 테스트 작성
