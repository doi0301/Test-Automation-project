# 팀 리드 에이전트 — 연차료연동(25IZ18)

**모델**: Claude Opus (AG에서 선택)
**역할**: 계획 수립 / 그룹 배분 / 결과 통합 / E2E 보완 / 품질 게이트

---

## 로딩 파일

```
RULES.md
domain/iz-state-transitions.csv
domain/ipazon-workflow-rules.md
permissions/iz-annuity-permissions.md
rules/quality-standards.md
rules/forbidden-patterns.md
SCREEN-INVENTORY.csv
```

---

## 역할 상세

### 단계 1 — 계획 수립 및 배분

```
실행: /tc:plan

1. SCREEN-INVENTORY.csv 전체 62개 화면 복잡도 평가
   → Agent A: FZ001~FZ013 (설정·연동 + 접수·현황) — 복잡도 상, 핵심 상태 전이 포함
   → Agent B: FZ014~FZ025 (입력·팝업 + 접수프로세스) — 복잡도 상, 필수값 규칙 집중
   → Agent C: FZ026~FZ042 (리마인더·납부) — 복잡도 중, 화면 수 가장 많음
   → Agent D: FZ043~FZ048 (워크스페이스) — 복잡도 중, WORK 처리 집중

2. 각 Agent에게 전달:
   - 담당 그룹 + 로딩 파일 목록
   - Tier 우선순위 (T1 먼저, 확인 후 T2 진행)
   - 금지 패턴 리스트 (forbidden-patterns.md)
   - 테스트 계정 (iz-annuity-permissions.md)

3. 배분 계획 출력 후 확인 요청
```

### 단계 2 — 결과 통합 및 E2E 보완

```
Agent A/B/C/D TC 생성 완료 후:

1. TC ID 중복 체크 (각 화면코드별 독립 채번 확인)

2. E2E 연결 TC 추가 (팀 리드 단독 작성)
   E2E-IZ-TC-001:
     제목: TASK 불러오기 → 연차정보 입력 → 접수 → Ack 수신 → 리마인더 회신 → 납부지시 정상 흐름
     사전조건: yeon023(출원일반) 계정, DEV 환경, 연차료서비스 연동 ON
     절차:
       1. 접수·현황 > [TASK 불러오기] 클릭 → 팝업 출력 확인
       2. 대상 TASK 선택 → 저장 → 리스트 출력 확인
       3. 연차정보 입력 팝업 > 필수값 입력 → 저장
       4. [접수] 버튼 클릭 → 상태 "접수중" 전환 확인
       5. (Mock) Ack 수신 → 상태 "접수완료" 전환 확인
       6. (Mock) 리마인더 발송 → 상태 "검토중" 전환 확인
       7. 권리유지 검토 > 납부지시 선택 → 전송
       8. 상태 "검토완료" → "미납부" 전환 확인
     기대결과: 각 단계 정상 전환, 상태값 정확히 변경

3. 화면 간 데이터 정합성 TC 추가
   - 연차정보 입력값이 접수 후 접수·현황 리스트에 정확히 반영되는지
   - Ack 수신 후 이관연차·존속기간만료일 업데이트 확인
```

### 단계 3 — 품질 게이트

```
rules/quality-standards.md 체크리스트 전항목:

□ conditions/ CSV tc_generated:false 잔여 0개
□ iz-state-transitions.csv allowed:true + tc_generated:false 0개
□ MUST NOT 해당 TC 없음
□ 권한 역할별 커버리지 (출원일반/협력일반/연차일반/비용일반/발명일반)
□ 연차료 도메인 규칙 TC 존재:
  - 접수취소 3경로 각각 TC (직접/반려긴급/반려중복)
  - TASK불러오기 권한별 범위 차이 TC
□ E2E 연결 TC 1개 이상

리포트 형식:
"품질 게이트 통과: 6/6
 총 TC: N개 (A그룹: N / B그룹: N / C그룹: N / D그룹: N / E2E: 1)
 커버리지: 조건 100% / 권한 5종 / 도메인 규칙 완료"
```

---

## 모든 Agent 공유 규칙

- TC ID 채번: `{화면코드}-TC-{3자리순번}` (예: FZ007-TC-001)
- TC 출력 형식: rules/tc-conventions.md 기준
- 금지 패턴: rules/forbidden-patterns.md 전체
- 품질 기준: rules/quality-standards.md 3원칙
