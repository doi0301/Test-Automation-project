# TC Pilot — 연차료연동(25IZ18) 화면기획

## 프로젝트 개요
IPAZON 연차료연동 서비스(25IZ18) 화면기획서(V1.33, FZ001~FZ048, 62개 화면)를
6 Layer로 구조화하여 TC를 자동 생성한다.

---

## 6 Layer 구조

| 레이어 | 경로 | 내용 |
|--------|------|------|
| L1 화면 스펙 | `specs/` | FZ 화면 단위 Markdown + YAML frontmatter |
| L2 조건 테이블 | `conditions/` | 1행=1조건 CSV, tc_generated 추적 |
| L3 상태 전이 | `domain/iz-state-transitions.csv` | allowed 기반 TC 생성 여부 결정 |
| L4 권한 매트릭스 | `permissions/iz-annuity-permissions.md` | 역할별 TC + 테스트 계정 |
| L5 데이터 필드 | `data/` | 항목별 입력방식·필수여부·DB 필드 |
| L6 도메인 규칙 | `domain/ipazon-workflow-rules.md` | MUST/MUST NOT/WHEN-THEN |

---

## /tc:go {화면코드} — 레이어 로딩 순서

1. `specs/{화면코드}.md` 로딩 → 대상 화면 파악
2. YAML frontmatter `related` → 관련 FZ 화면 specs 추가 로딩
3. YAML frontmatter `conditions_table` → `conditions/` CSV 로딩
4. `domain/iz-state-transitions.csv` → allowed:true 전이만 TC 대상
5. `permissions/iz-annuity-permissions.md` → 역할별 TC 계정 + 처리 조건
6. `data/` JSON → direction:both 필드 = 데이터 정합성 TC 자동 생성
7. `domain/ipazon-workflow-rules.md` → MUST NOT 금지 패턴, WHEN-THEN TC 소스
8. `rules/` 전체 → TC 생성 규칙 적용
9. TC 생성 완료 후 → `conditions/` CSV tc_generated 업데이트

---

## 품질 게이트

```
체크 1: conditions/ CSV tc_generated:false 잔여 행 → 경고
체크 2: iz-state-transitions.csv allowed:true + tc_generated:false → 경고
체크 3: MUST NOT 해당 TC 존재 → 삭제
체크 4: 권한 역할별 TC 커버리지
        (출원일반/협력일반/연차일반/비용일반/발명일반 각 1개 이상)
체크 5: 연차료 도메인 규칙 TC 존재
        - 접수취소 3경로 각각 TC (직접/반려긴급/반려중복)
        - TASK불러오기 권한별 범위 차이 TC
        - 역방향 상태 전이 금지 확인
체크 6: E2E 연결 TC 존재 (미접수→접수중→접수완료→검토중→검토완료→미납부 흐름)
```

---

## 커맨드

| 커맨드 | 사용 시점 |
|--------|---------|
| `/tc:go {화면코드}` | 복잡한 화면 (FZ007, FZ011, FZ017, FZ021) |
| `/tc:batch {그룹}` | 그룹 단위 배치 생성 |
| `/tc:fast` | 단순 반복 화면 |

### 그룹 배치 정의
```
/tc:batch A  → FZ001~FZ005 (설정·연동)
/tc:batch B  → FZ006~FZ013 (접수·현황)
/tc:batch C  → FZ014~FZ021 (입력·팝업)
/tc:batch D  → FZ022~FZ025 (접수프로세스)
/tc:batch E  → FZ026~FZ042 (리마인더·납부)
/tc:batch F  → FZ043~FZ048 (워크스페이스)
```

---

## Agent Teams 구조

```
팀 리드 (Opus)      agents/agent-lead.md
├── Agent A (Sonnet) FZ001~FZ013 (설정·연동 + 접수·현황)
├── Agent B (Sonnet) FZ014~FZ025 (입력·팝업 + 접수프로세스)
├── Agent C (Sonnet) FZ026~FZ042 (리마인더·납부)
└── Agent D (Sonnet) FZ043~FZ048 (워크스페이스)
```

---

## 주요 파일 참조

| 참조 목적 | 파일 |
|---------|------|
| 화면 전체 목록 | `SCREEN-INVENTORY.csv` |
| 권한 + 테스트 계정 | `permissions/iz-annuity-permissions.md` |
| 상태 전이 | `domain/iz-state-transitions.csv` |
| 도메인 규칙 | `domain/ipazon-workflow-rules.md` |
| TC 금지 패턴 | `rules/forbidden-patterns.md` |
| TC 형식 기준 | `rules/tc-conventions.md` |
| 온보딩 가이드 | `ONBOARDING-NEW-PROJECT.md` |
