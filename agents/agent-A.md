# Agent A — FZ001~FZ013 담당 (설정·연동 + 접수·현황)

**모델**: Claude Sonnet (AG에서 선택)
**담당 그룹**: A(FZ001~005) + B(FZ006~013)
**예상 TC 수**: ~35개

---

## 로딩 파일

```
RULES.md
specs/FZ001.md ~ specs/FZ013.md
conditions/FZ001-표1.csv ~ conditions/FZ013-표1.csv
permissions/iz-annuity-permissions.md
domain/iz-state-transitions.csv
domain/ipazon-workflow-rules.md
rules/tc-conventions.md
rules/forbidden-patterns.md
```

---

## 담당 화면 목록

| 화면코드 | 화면명 | 복잡도 | TC 집중 포인트 |
|---------|--------|--------|--------------|
| FZ001 | 고객상세보기 진입경로 | 하 | 진입 조건 분기 |
| FZ002 | 고객상세보기 | 중 | 연차료 메뉴 출력 조건 |
| FZ003 | IPAZON admin | 중 | 연동 ON/OFF 조건 |
| FZ0031 | WIPS연차료담당자 세팅 | 하 | 자동 계정 생성 확인 |
| FZ004 | 관리자>비용 체크 | 하 | 연차료 현황리스트 출력 조건 |
| FZ005 | 연차료서비스 OFF 메일 | 하 | 메일 발송 조건 |
| FZ006 | 연차료 현황리스트 | 중 | 리스트 출력 조건 |
| FZ007 | 접수·현황 | 상 | 버튼 조건 분기 전체 |
| FZ008 | 건 상태 정의 | 중 | 12개 상태값 전이 규칙 |
| FZ009 | 접수·현황 출력항목 | 중 | 필드 출력 포맷 |
| FZ010 | 접수·현황 필터 | 중 | 필터 조건 조합 |
| FZ011 | TASK 불러오기 팝업 | 상 | TASK 출력 조건 (권리종류/출원상태 등) |
| FZ012 | 담당자변경 이슈워크 | 중 | 자동 Done 처리 조건 |
| FZ013 | 워크플로 종료요청 이슈워크 | 중 | 자동 Done 처리 조건 |

---

## TC 생성 핵심 포인트

### FZ007 (접수·현황) — 이 그룹 가장 복잡한 화면

조건 테이블의 모든 행(C01~) 1:1 TC 생성:
- 버튼별 TASK 미선택 시 alert
- 버튼별 상태 불일치 시 alert (접수조건 N 포함)
- 접수취소 confirm 메시지 원문 확인
- 제외 confirm 메시지 원문 확인

### FZ011 (TASK 불러오기) — 권한별 출력 범위 차이 TC 필수

| 역할 | TASK 출력 범위 |
|------|--------------|
| 출원담당자 (출원일반) | 전체 TASK |
| 협력담당자 (협력일반) | 본인이 협력담당자인 TASK만 |
| 유지료담당자 (연차일반) | 유지료담당자인 TASK만 |

### FZ008 (건 상태 정의) — iz-state-transitions.csv 참조

allowed:false 전이는 TC 생성 금지:
- 미접수에서 직접 검토중 전이 금지
- 역방향 전이 금지 전체

---

## 이 그룹 특화 MUST NOT

- WIPS연차료서비스 연동 OFF 상태에서 접수 버튼 동작 TC 금지
- 유지료담당자(wipsfee*) 계정 수동 로그인 TC 금지
- 접수조건 N인 TASK 접수 성공 TC 금지

---

## Agent 공유 규칙

- TC ID 채번: `{화면코드}-TC-{3자리순번}` (예: FZ007-TC-001)
- TC 출력 형식: rules/tc-conventions.md
- 금지 패턴: rules/forbidden-patterns.md
- 품질 기준: rules/quality-standards.md
