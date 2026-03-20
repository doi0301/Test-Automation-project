# TC Pilot — 신규 프로젝트 TC 추출 팀 온보딩 가이드

> **프로젝트**: 25IZ18 연차료연동 화면기획 (FZ001~FZ048, 총 62개 화면)
> **방식**: 각자 전체 범위 독립 진행 → 결과 비교
> **목적**: AG + Claude Code TC 추출 워크플로우 체화 + 결과 품질 비교
> **GitHub**: https://github.com/doi0301/tc-project-v2

---

## 파일 구성 (GitHub에서 clone)

```
tc-project-v2/
└── tc-pilot/
    ├── RULES.md                      ← AG 규칙 체계 진입점 (건드리지 말 것)
    ├── SCREEN-INVENTORY.csv          ← 화면 목록 (이 파일 참조)
    ├── rules/
    │   ├── tc-conventions.md         ← TC 형식 기준 (공통 — 건드리지 말 것)
    │   └── quality-standards.md      ← 품질 게이트 기준
    ├── specs/                        ← 각자 작성 (L1)
    ├── conditions/                   ← 각자 작성 (L2)
    ├── domain/                       ← 각자 작성 (L3, L6)
    ├── permissions/                  ← 각자 작성 (L4)
    ├── data/                         ← 각자 작성 (L5)
    └── output/                       ← TC 결과물 저장
```

> **공통 고정 파일**: `RULES.md`, `rules/tc-conventions.md`, `rules/quality-standards.md`
> 이 3개는 TC 형식 통일을 위해 수정하지 않는다.

---

## 사전 설치 (1회)

```bash
# 1. Claude Code 설치 (Node.js 18+ 필요)
npm install -g @anthropic-ai/claude-code

# 2. 설치 확인
claude --version

# 3. GitHub clone
git clone https://github.com/doi0301/tc-project-v2.git
cd tc-project-v2/tc-pilot

# 4. Playwright 설치 (선택 — 자동화 테스트까지 할 경우)
npm init -y
npm install -D @playwright/test
npx playwright install chromium
claude mcp add playwright npx @playwright/mcp@latest
```

---

## 전체 작업 흐름

```
Step 1. 공통 데이터 처리 (1회)
  → Excel 2개를 L4, L6 파일로 변환
  → 모든 화면 TC에 적용되는 기반 데이터

Step 2. 화면 인벤토리 확인
  → SCREEN-INVENTORY.csv 에서 전체 62개 화면 파악
  → 슬라이드 번호 기준으로 작업 순서 결정

Step 3. 기획서 파싱 → L1/L2 생성
  → PPT 슬라이드 이미지 + 텍스트 파싱
  → 화면 그룹 단위로 묶어서 처리

Step 4. TC 생성 (/tc:go)
  → Claude Code에서 실행
  → 그룹 단위 배치 처리

Step 5. 품질 게이트
  → 커버리지 확인 + 금지 패턴 검증
```

---

## Step 1. 공통 데이터 처리 — Excel 변환

### 1-A. 권한 매트릭스 생성 (L4)

Claude Code에서 실행:

```
IPAZON_권한및이벤트조건.xlsx 파일을 읽고
permissions/ipazon-permissions.md 를 생성해줘.

포함할 내용:
1. 역할 정의표 (담당자종류 시트 기반)
   - 역할명, 권한코드, TASK 열람 범위, 처리 가능 WORK
2. WORK별 처리자·열람자 매트릭스 (WORK별 처리자및열람자 시트 기반)
   - TC에서 자주 쓰는 WORK 그룹별로 정리
3. 테스트 계정 목록 (환경세팅 시트 기반)
   - yeon000~yeon028 계정별 역할, 소속, 권한코드
   형식: | USER | ID | 역할 | 권한 | TC 활용 시나리오 |
```

### 1-B. 워크플로 도메인 규칙 생성 (L6)

```
IPAZON_워크항목.xlsx 파일을 읽고
domain/ipazon-workflow-rules.md 를 생성해줘.

포함할 내용:
1. MUST — 워크 처리 규칙 (필수값, 처리자 조건)
2. MUST NOT — 금지 패턴
   (권한 없는 처리자 실행 성공 TC 금지 등)
3. WHEN-THEN — 조건부 규칙
   (특정 권리종류일 때만 출현하는 항목 등)
4. 연차료연동(FZ) 관련 항목 우선 추출
   (WORKFLOW = 권리유지 관련 항목 집중)
```

---

## Step 2. 화면 인벤토리 확인

`SCREEN-INVENTORY.csv` 열어서 전체 구조 파악:

```
그룹 A. 설정·연동    FZ001~FZ005   (5개)   복잡도 하~중
그룹 B. 접수·현황    FZ006~FZ013   (8개)   복잡도 중~상  ← 핵심 그룹
그룹 C. 입력·팝업    FZ014~FZ021   (10개)  복잡도 중~상  ← 핵심 그룹
그룹 D. 접수프로세스  FZ022~FZ025   (4개)   복잡도 하~중
그룹 E. 리마인더·납부 FZ026~FZ042   (17개)  복잡도 중~상
그룹 F. 워크스페이스  FZ043~FZ048   (6개)   복잡도 중~상
```

**권장 처리 순서**: B → C → E → A → D → F
(복잡도 높은 것부터 → 패턴 파악 후 단순한 것 처리)

---

## Step 3. 기획서 파싱 → L1/L2 생성

### 슬라이드 이미지 추출

```bash
# PPT → PDF → 이미지 변환 (tc-pilot/ 폴더에서 실행)
python -m markitdown 25IZ18_연차료연동_화면기획_V1_33.pptx > ppt-text.md

# 이미지 추출 (LibreOffice + pdftoppm 필요)
# 또는 PPT를 수동으로 PDF로 저장 후:
pdftoppm -jpeg -r 120 기획서.pdf slides/slide
```

### 그룹 단위 파싱 프롬프트 (Claude.ai 웹 또는 AG)

**그룹 B (접수·현황) 예시:**

```
첨부한 슬라이드 이미지(slide-011~018.jpg)와 텍스트를 분석해서
아래 파일들을 생성해줘.

대상 화면: FZ006, FZ007, FZ008, FZ009, FZ010, FZ011, FZ012, FZ013

각 화면마다 생성:
1. specs/FZ00X.md
   - YAML frontmatter: screen_code, screen_name, path, screen_type, related, conditions_table
   - 본문: 진입조건 / 버튼 및 조건 / 접근권한

2. conditions/FZ00X-표1.csv
   - 컬럼: id, trigger, case, action, message, tc_generated
   - 기획서의 모든 조건 분기를 1행=1조건으로
   - tc_generated 초기값: false
   - 불확실한 항목: message에 [확인필요] 표기

추가 참조:
- permissions/ipazon-permissions.md (역할별 접근 조건)
- domain/ipazon-workflow-rules.md (워크플로 규칙)
```

### 파싱 품질 체크리스트

각 화면 파싱 완료 후 확인:
- [ ] 버튼/링크마다 조건 분기가 CSV 1행으로 분리됐는지
- [ ] alert/confirm/toast 메시지 원문이 message 컬럼에 있는지
- [ ] 접근 권한 조건이 specs에 명시됐는지
- [ ] [확인필요] 항목을 기획자에게 확인했는지

---

## Step 4. TC 생성

### Claude Code 실행

```bash
cd tc-project-v2/tc-pilot
claude
```

### 그룹 단위 TC 생성 프롬프트

```
RULES.md와 tc-pilot/ 폴더 전체를 읽고
그룹 B (FZ006~FZ013) TC를 생성해줘.

실행: /tc:batch B

각 화면 완료 시:
- TC 목록 출력 후 확인 요청
- conditions/*.csv tc_generated 업데이트
- output/TC-IZ-FZ00X.md 로 저장

금지 패턴: rules/forbidden-patterns.md 준수
```

### 전체 실행 순서

```
/tc:batch B   → FZ006~FZ013 (접수·현황)    확인
/tc:batch C   → FZ014~FZ021 (입력·팝업)    확인
/tc:batch E   → FZ026~FZ042 (리마인더·납부) 확인
/tc:batch A   → FZ001~FZ005 (설정·연동)    확인
/tc:batch D   → FZ022~FZ025 (접수프로세스)  확인
/tc:batch F   → FZ043~FZ048 (워크스페이스)  확인
```

---

## Step 5. 품질 게이트

```
output/ 폴더의 모든 TC 파일을 읽고
rules/quality-standards.md 체크리스트 실행해줘.

체크 항목:
1. conditions/ CSV tc_generated false 잔여 행
2. 권한 역할별 TC 커버리지 (출원담당자/협력담당자/유지료담당자 등)
3. MUST NOT 위반 TC 없음
4. 상태 전이 커버리지
5. E2E 연결 TC 존재 여부

최종 커버리지 리포트 출력.
```

---

## 비교 기준 (각자 완료 후)

각자 TC 추출 완료 후 아래 항목으로 비교:

| 비교 항목 | 확인 방법 |
|---------|---------|
| 총 TC 수 | output/ 파일별 TC 카운트 |
| 조건 커버리지 | tc_generated true 비율 |
| 누락 조건 | 다른 사람 CSV와 교차 비교 |
| 금지 패턴 위반 | forbidden-patterns 대조 |
| 도메인 규칙 TC | WHEN-THEN 조건 반영 여부 |

**비교에서 가장 중요한 포인트**:
"같은 화면인데 누가 더 많은 조건을 TC로 뽑았는가" →
이게 곧 기획서 구조화 품질의 차이

---

## 주의사항 — 이번 프로젝트 특이점

### FZ 코드 체계
- 서브 화면: FZ0031, FZ0151, FZ0161~0163, FZ0171, FZ0181, FZ0191, FZ0210~0213, FZ0341
- 서브 화면은 부모 화면의 `related` 필드에 포함시키고 별도 specs 파일 생성

### Excel 3개의 역할
| 파일 | TC에서의 역할 |
|------|-------------|
| IPAZON_권한및이벤트조건.xlsx | L4 권한 매트릭스 소스 + 테스트 계정 |
| IPAZON_워크항목.xlsx | L6 도메인 규칙 소스 (워크 조건, 필수값) |
| PF0101_P프로젝트_항목.xlsx | L5 데이터 필드 소스 (항목별 입력방식, 필수여부) |

### 연차료연동 특화 도메인 규칙
플로우차트 + 기획서 기반. TC 생성 전 반드시 숙지.
**참조 파일**: `domain/iz-state-transitions.csv` (플로우차트 기반 전체 상태 전이 정의)

**상태 전이 전체 흐름**:
```
미접수 → 접수중 → 접수완료 → 검토중 → 검토완료 → 미납부 → 입금대기 → 입금완료 → 납부완료
         ↓           ↓(반려3경로)  ↓            ↓(지시여부N)
       접수취소     접수취소      납부포기      납부포기
         ↓
        OUT(제외)
```

**Actor 구분**:
- IZ (출원담당자, 협력담당자): TASK 불러오기, 연차정보 입력, 접수, 접수취소, 제외, 권리유지 검토
- WIPSFee (연차료팀): Ack 발송, 반려(긴급/중복), 리마인더 발송, 청구, 입금확인, 납부완료

**MUST NOT 패턴** (이 TC는 생성하면 안 됨):
- WIPS연차료서비스 연동 OFF 상태에서 API 호출 TC 금지
- 유지료담당자(wipsfee*) 계정으로 수동 로그인 TC 금지
- 접수조건 N인 TASK 접수 성공 TC 금지
- 역방향 상태 전이 TC 금지 (iz-state-transitions.csv allowed:false 참조)
- 미접수에서 검토중·납부완료로 직접 전이 TC 금지
- 접수중에서 미접수로 역방향 전이 TC 금지
- 지시여부 Y 없이 검토완료→입금대기 직접 전이 TC 금지

**WHEN-THEN 핵심 규칙** (플로우차트 확인):
- WHEN 접수 클릭 → THEN 신건 발수값 TASK uid 전송 + 연차건 KEY 생성
- WHEN 공동출원인 Y → THEN 공동고객 확인요청 메일 발송 (WIPSFee)
- WHEN 공동고객이 늦게 접수 → THEN 반려(긴급) 또는 반려(중복) 분기
- WHEN Ack 발송 → THEN 이관연차·이관연차마감일·존속기간만료일 업데이트
- WHEN 리마인더 최신알 경과 → THEN 리마인더 자동 재발송
- WHEN 오청구 발생 → THEN 청구 재발송 (미납부 상태 유지)
- WHEN 지시여부 N → THEN 납부포기 전이 (전송 불가)
