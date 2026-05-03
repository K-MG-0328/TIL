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

---

### 전체 커밋 메시지 형식 (Conventional Commits)
```
<type>(<scope>): <subject>

<body>

<footer>
```

### 좋은 커밋 메시지의 7가지 규칙 (Tim Pope의 가이드)
1. 제목과 본문을 빈 줄로 분리
2. 제목은 50자 이내
3. 제목 첫 글자는 대문자(또는 소문자로 통일)
4. 제목 끝에 마침표 X
5. 제목은 명령형으로 작성 ("Added" X, "Add" O)
6. 본문은 72자마다 줄바꿈
7. 본문에는 **What과 Why**를 설명 (How는 코드가 보여줌)

### Type 선택 가이드
| Type | 사용 시점 | 예시 |
|---|---|---|
| `feat` | 새로운 기능 추가 (사용자 관점에서) | 로그인 API 추가 |
| `fix` | 버그 수정 | NPE 발생 케이스 수정 |
| `docs` | 문서만 변경 | README 업데이트 |
| `style` | 코드 동작에 영향 없는 포맷 | 들여쓰기, 세미콜론, 따옴표 통일 |
| `refactor` | 기능·동작 변화 없는 구조 개선 | 메서드 추출, 변수 이름 변경 |
| `perf` | 성능 개선 (동작은 동일) | N+1 해결, 쿼리 최적화 |
| `test` | 테스트 코드만 추가/수정 | 통합 테스트 추가 |
| `build` | 빌드 시스템·외부 의존성 | gradle, package.json 변경 |
| `ci` | CI/CD 설정 | GitHub Actions, Jenkinsfile |
| `chore` | 그 외 자잘한 변경 | .gitignore, 설정 파일 |
| `revert` | 이전 커밋 되돌리기 | 커밋 메시지에 원래 해시 명시 |

### Subject 작성 패턴
**Bad → Good 사례**
| Bad (수동태/과거형/모호) | Good (명령형/구체적) |
|---|---|
| `Fixed bug` | `fix(auth): JWT 만료 시 401 대신 500 반환되던 문제 수정` |
| `Updated code` | `refactor(order): 주문 검증 로직을 OrderValidator로 분리` |
| `Added feature` | `feat(payment): 카카오페이 결제 수단 추가` |
| `WIP` | (커밋 자체를 squash 또는 rebase로 정리) |

### Body가 필요한 경우
- 변경 동기 설명 (왜 이렇게 했는가)
- 이전 동작과의 차이
- 검토 시 참고할 트레이드오프
- 외부 이슈/PR 링크

```
fix(payment): 결제 실패 시 주문 상태가 PAID로 남던 문제 수정

PaymentService에서 결제 실패 응답을 받은 뒤에도 OrderStatus를 PAID로
변경하던 버그가 있어 환불 요청이 정상 흐르지 않았다. 트랜잭션 경계를
PaymentService → OrderService 순서로 재정렬하여 결제 실패 시 주문이
ROLLBACK 되도록 수정.

Closes #142
```

### Breaking Change 표기
호환을 깨는 변경은 footer에 `BREAKING CHANGE:`를 명시하거나 type 뒤에 `!`를 붙입니다.

```
feat(api)!: 응답 포맷을 ApiResponse<T> 래퍼로 통일

BREAKING CHANGE: 기존에 직접 객체를 반환하던 모든 API가
{"success": true, "data": ...} 형태로 변경되어 클라이언트 수정 필요.
```

### 한국어로 작성하는 패턴
한국어로 쓸 때는 명령형 어미가 어색하므로 보통 "~함" 또는 "~ 추가/수정/제거" 같은 명사형 종결을 씁니다.

```
feat(board): 게시글 검색 기능 추가
fix(member): 비밀번호 변경 시 세션 만료 처리 누락 수정
```

### 한 커밋에 한 가지 일만
- 기능 추가 + 리팩터링 = 두 커밋으로 분리
- 같은 PR이라도 의미 단위로 쪼개기 → 코드 리뷰가 쉬워지고 revert가 안전해짐
- 흩어진 작업이라면 `git add -p`로 hunk 단위 staging 후 분리 커밋

### 커밋 전 체크리스트
- [ ] 변경된 파일이 의도한 것만 포함되어 있는가? (`git diff --staged`)
- [ ] 디버깅 코드(`console.log`, `System.out.println`, `print`) 남아있지 않은가?
- [ ] 주석 처리된 죽은 코드 제거됐는가?
- [ ] 테스트가 통과하는가?
- [ ] Subject 50자 이내, body 72자 줄바꿈?

### 자동화 도구
- **commitlint**: 커밋 메시지 형식 자동 검증 (Husky pre-commit hook)
- **commitizen** (`cz`): 대화형으로 커밋 메시지 작성 가이드
- **standard-version / semantic-release**: 커밋 타입 기반으로 자동 버전 태깅 + CHANGELOG 생성

[Git 명령어 정리](250129-Git.md)와 함께 보면 좋습니다.
