# TC Pilot — 신규 프로젝트 TC 추출 팀 온보딩 가이드

> **프로젝트**: 25IZ18 연차료연동 화면기획 (FZ001~FZ048, 총 62개 화면)
> **방식**: 각자 전체 범위 독립 진행 → 결과 비교
> **목적**: AG + Claude Code TC 추출 워크플로우 체화 + 결과 품질 비교
> **GitHub**: https://github.com/doi0301/tc-project-v2

---

## 도구 역할 분담 — AG vs Claude Code

```
AG (Antigravity)                    Claude Code CLI (터미널)
────────────────────────────────    ────────────────────────────────
멀티모달 이미지 파싱                파일 생성·수정·저장
병렬 에이전트 (Agent Teams)         품질 게이트 자동 실행
브라우저 직접 실행·확인             Playwright 터미널 실행
시각적 작업 관리 (Manager View)     Git 정리·PR
```

**"샌드위치" 패턴**:
```
[Claude Code]       [AG]                  [Claude Code]
세팅 & 검증   →   파싱 & TC 병렬 생성  →  품질 게이트 & Git
Step 1            Step 2~4               Step 5~6
```

---

## 파일 구성 (GitHub에서 clone)

```
tc-project-v2/
└── tc-pilot/
    ├── RULES.md                      ← 규칙 체계 진입점 (건드리지 말 것)
    ├── SCREEN-INVENTORY.csv          ← 화면 목록
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

# 4. Playwright 설치 (자동화 테스트까지 할 경우)
npm init -y
npm install -D @playwright/test
npx playwright install chromium
claude mcp add playwright npx @playwright/mcp@latest
```

---

## 전체 작업 흐름

```
Step 1. 공통 데이터 처리        [Claude Code]  Excel → L4/L6 파일 변환
Step 2. 화면 인벤토리 확인      [둘 다]        SCREEN-INVENTORY.csv 파악
Step 3. 기획서 파싱 → L1/L2    [AG]           PPT 이미지 멀티모달 파싱
Step 4. TC 생성                 [AG]           Agent Teams 병렬 실행
Step 5. 품질 게이트             [Claude Code]  파일 수정 + 커버리지 체크
Step 6. Playwright              [AG + Claude Code]  AG: 브라우저 실행 / CC: 반복 수정
```

---

## Step 1. 공통 데이터 처리 — Claude Code

> **왜 Claude Code?** 파일을 직접 저장하는 작업. 터미널 네이티브.

```bash
cd tc-project-v2/tc-pilot
claude
```

### 1-A. 권한 매트릭스 생성 (L4)

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

### 1-C. DB 명세서 → L5 데이터 필드 생성 (화면별 On-demand)

> **언제 실행하나**: Step 1에서 한꺼번에 하지 않습니다.
> 각 화면 specs 파일 생성 후, 출력 항목·입력 필드가 있는 화면마다 실행합니다.
> 예: FZ009(접수·현황 출력항목), FZ016(연차정보 입력 필수값)

**준비 (1회)**: DB 명세서 xlsx 파일을 tc-pilot/ 폴더에 복사

**연차료 관련 테이블 목록 먼저 파악:**
```
IPAZON_DB명세서_V3_9_2.xlsx 에서
"연차" 또는 "annuity" 또는 "fee" 가 포함된 테이블명을 전부 추출해줘.
```

**화면별 L5 JSON 생성 (TC 생성 직전, 화면코드만 바꿔서 재사용):**
```
IPAZON_DB명세서_V3_9_2.xlsx 와 specs/FZ009.md 를 읽고
data/FZ009-fields.json 을 생성해줘.

각 필드마다:
- iz_name: 화면 표시 한글 필드명
- db_table / db_column: DB 매핑
- direction: send / receive / both
- format: 날짜·숫자·텍스트 등
- tc_note: TC 주의사항

direction:both → tc_note에 "데이터 정합성 TC 필수"
불확실 → tc_note에 "[확인필요]"
```

**자주 쓰는 화면:**

| 화면코드 | 화면명 | 포인트 |
|---------|--------|--------|
| FZ009 | 접수·현황 출력항목 | 연차료 대상건 목록 출력 필드 |
| FZ016 | 연차정보 입력 필수값 | 필수/선택 구분 + 입력방식 |
| FZ032 | 리마인더 출력항목 | 발송번호·관리번호 연관관계 |
| FZ041 | 납부현황 출력항목 | IZ↔연차료 양방향 데이터 |

---

## Step 2. 화면 인벤토리 확인

`SCREEN-INVENTORY.csv` 열어서 전체 구조 파악:

```
그룹 A. 설정·연동     FZ001~FZ005   (5개)   복잡도 하~중
그룹 B. 접수·현황     FZ006~FZ013   (8개)   복잡도 중~상  ← 핵심
그룹 C. 입력·팝업     FZ014~FZ021   (10개)  복잡도 중~상  ← 핵심
그룹 D. 접수프로세스   FZ022~FZ025   (4개)   복잡도 하~중
그룹 E. 리마인더·납부  FZ026~FZ042   (17개)  복잡도 중~상
그룹 F. 워크스페이스   FZ043~FZ048   (6개)   복잡도 중~상
```

**권장 처리 순서**: B → C → E → A → D → F

---

## Step 3. 기획서 파싱 → L1/L2 생성 — AG

> **왜 AG?** 멀티모달 이미지 처리 + 결과 파일을 브라우저에서 즉시 확인 가능.
> 파싱 결과를 눈으로 보면서 조건 누락 여부를 바로 검토할 수 있음.

### 슬라이드 이미지 추출 (Claude Code 터미널에서 1회)

```bash
python -m markitdown 25IZ18_연차료연동_화면기획_V1_33.pptx > ppt-text.md
pdftoppm -jpeg -r 120 기획서.pdf slides/slide
```

### AG에서 그룹 단위 파싱 프롬프트

> 그룹 단위로 관련 슬라이드를 묶어서 한 번에 전달. 순서 강제 아님.
> 한 프롬프트에 8~10장 이내로 제한 (초과 시 파싱 품질 저하).

**그룹 B (접수·현황) 예시:**

```
첨부한 슬라이드 이미지(slide-011~018.jpg)와 ppt-text.md 텍스트를 함께 분석해서
아래 파일들을 생성해줘.

대상 화면: FZ006, FZ007, FZ008, FZ009, FZ010, FZ011, FZ012, FZ013

각 화면마다 생성:
1. specs/FZ00X.md
   - YAML frontmatter: screen_code, screen_name, path,
     screen_type, related, conditions_table
   - 본문: 진입조건 / 버튼 및 조건 / 접근권한

2. conditions/FZ00X-표1.csv
   - 컬럼: id, trigger, case, action, message, tc_generated
   - 기획서의 모든 조건 분기를 1행=1조건으로
   - tc_generated 초기값: false
   - 불확실한 항목: message에 [확인필요] 표기

추가 참조:
- permissions/iz-annuity-permissions.md (역할별 접근 조건)
- domain/ipazon-workflow-rules.md (워크플로 규칙)
```

### 파싱 품질 체크리스트

파싱 완료 후 AG 브라우저 프리뷰에서 확인:
- [ ] 버튼/링크마다 조건 분기가 CSV 1행으로 분리됐는지
- [ ] alert/confirm/toast 메시지 원문이 message 컬럼에 있는지
- [ ] 접근 권한 조건이 specs에 명시됐는지
- [ ] [확인필요] 항목 기획자에게 확인했는지

---

## Step 4. TC 생성 — AG (Agent Teams)

> **왜 AG?** Agent A/B/C/D를 동시에 실행하면 순차 처리 대비 시간이 4분의 1로 단축됨.
> Claude Code는 병렬 실행 불가.

### agents/ 폴더 파일 안내

`tc-pilot/agents/` 폴더에 각 에이전트용 지시 파일이 미리 작성되어 있습니다.
AG에서 에이전트를 생성할 때 이 파일 내용을 **초기 프롬프트로 붙여넣으면** 됩니다.

| 파일 | 담당 그룹 | 화면 범위 |
|------|---------|---------|
| agents/agent-lead.md | 팀 리드 (Opus) | 계획·배분·통합·품질게이트 |
| agents/agent-A.md | A+B 그룹 (Sonnet) | FZ001~FZ013 |
| agents/agent-B.md | C+D 그룹 (Sonnet) | FZ014~FZ025 |
| agents/agent-C.md | E 그룹 (Sonnet) | FZ026~FZ042 |
| agents/agent-D.md | F 그룹 (Sonnet) | FZ043~FZ048 |

### AG에서 병렬 실행 조작 순서

AG는 프롬프트 한 줄로 자동 분기되지 않습니다. **[New Agent] 버튼을 직접 4번 클릭**해서 각각 생성합니다.

```
1. AG Agent Manager 열기

2. [New Agent] 클릭 → Agent A 생성
   Workspace: tc-project-v2/tc-pilot
   Model: Claude Sonnet / Mode: Planning
   초기 프롬프트: agents/agent-A.md 파일 전체 내용 붙여넣기 후
   → "담당 화면 TC 생성 시작해줘" 추가 입력

3. [New Agent] 클릭 → Agent B 생성 (Agent A와 동시에)
   초기 프롬프트: agents/agent-B.md 전체 붙여넣기

4. [New Agent] 클릭 → Agent C 생성
   초기 프롬프트: agents/agent-C.md 전체 붙여넣기

5. [New Agent] 클릭 → Agent D 생성
   초기 프롬프트: agents/agent-D.md 전체 붙여넣기

→ Manager View에서 4개 진행상황 동시 모니터링
→ 각 에이전트 완료 시 output/TC-IZ-FZ*.md 파일 생성 확인
```

> **파일 충돌 없는 이유**: 각 에이전트 담당 FZ 코드 범위가 겹치지 않아
> output 파일명이 독립적으로 저장됩니다.
> 공통 파일(RULES.md, rules/, permissions/, domain/)은 읽기만 하므로 충돌 없음.

### 단일 순차 실행 (병렬이 어렵거나 처음 시도할 때)

AG 또는 Claude Code에서:

```
/tc:batch B → 확인
/tc:batch C → 확인
/tc:batch E → 확인
/tc:batch A → 확인
/tc:batch D → 확인
/tc:batch F → 확인
```


---

## Step 5. 품질 게이트 — Claude Code

> **왜 Claude Code?** 파일 직접 수정(tc_generated 업데이트 등) + 정밀 추론.
> AG가 놓친 엣지 케이스를 Claude Code가 잡아주는 이중 검증 구조.

```bash
cd tc-project-v2/tc-pilot
claude
```

```
output/ 폴더의 모든 TC 파일을 읽고
rules/quality-standards.md 체크리스트 전항목 실행해줘.

체크 항목:
1. conditions/ CSV tc_generated false 잔여 행
2. 권한 역할별 TC 커버리지 (출원담당자/협력담당자/유지료담당자 등)
3. MUST NOT 위반 TC 없음
4. iz-state-transitions.csv 상태 전이 커버리지
5. E2E 연결 TC 존재 여부

실패 항목은 원인 출력 후 보완 TC 자동 추가.
최종 커버리지 리포트 출력.
```

---

## Step 6. Playwright — AG + Claude Code

> **AG**: 브라우저가 직접 떠서 화면 보면서 셀렉터 오류 즉시 수정 가능.
> **Claude Code**: 반복 수정, 터미널 실행, CI/CD 연결.

**AG에서 코드 생성 + 1차 실행:**
```
output/TC-IZ-FZ007.md 에서 T1 P1 정상 케이스를 읽고
Playwright MCP로 아래 시나리오를 브라우저에서 실행해줘.

시나리오:
1. yeon023 계정(출원일반)으로 로그인
2. 연차료 > 접수·현황 메뉴 진입
3. [TASK 불러오기] 버튼 클릭 → 팝업 출력 확인
4. TASK 1건 선택 → 저장
5. 리스트에 선택한 TASK 출력 확인

완료 후 output/playwright/FZ007-normal.spec.ts 로 저장.
실패 시 스크린샷 자동 저장.
```

**Claude Code에서 반복 수정 + 안정화:**
```bash
BASE_URL=http://dev.ipazon.com npx playwright test FZ007-normal.spec.ts
```

---

## 비교 기준 (각자 완료 후)

| 비교 항목 | 확인 방법 |
|---------|---------|
| 총 TC 수 | output/ 파일별 카운트 |
| 조건 커버리지 | tc_generated true 비율 |
| 누락 조건 | 다른 사람 CSV와 교차 비교 |
| 금지 패턴 위반 | forbidden-patterns 대조 |
| 도메인 규칙 TC | WHEN-THEN 조건 반영 여부 |

**핵심 포인트**: "같은 화면인데 누가 더 많은 조건을 TC로 뽑았는가"
→ 이게 곧 기획서 구조화 품질의 차이

---

## 주의사항 — 이번 프로젝트 특이점

### FZ 코드 체계
- 서브 화면: FZ0031, FZ0151, FZ0161~0163, FZ0171, FZ0181, FZ0191, FZ0210~0213, FZ0341
- 서브 화면은 부모 화면 `related` 필드에 포함, 별도 specs 파일 생성

### 첨부 파일 역할
| 파일 | TC에서의 역할 |
|------|-------------|
| IPAZON_권한및이벤트조건.xlsx | L4 권한 매트릭스 소스 + 테스트 계정 |
| IPAZON_워크항목.xlsx | L6 도메인 규칙 소스 |
| PF0101_P프로젝트_항목.xlsx | L5 데이터 필드 소스 |
| IPAZON_DB명세서_V3_9_2.xlsx | L5 DB 매핑 (화면별 On-demand) |

### 상태 전이 전체 흐름
```
미접수 → 접수중 → 접수완료 → 검토중 → 검토완료 → 미납부 → 입금대기 → 입금완료 → 납부완료
         ↓           ↓(반려3경로)  ↓            ↓(지시여부N)
       접수취소     접수취소      납부포기      납부포기
         ↓
        OUT(제외)
```
참조: `domain/iz-state-transitions.csv`

### MUST NOT 패턴 (TC 생성 금지)
- WIPS연차료서비스 연동 OFF 상태에서 API 호출 TC
- 유지료담당자(wipsfee*) 계정 수동 로그인 TC
- 접수조건 N인 TASK 접수 성공 TC
- 역방향 상태 전이 TC (iz-state-transitions.csv allowed:false 참조)
- 미접수→검토중·납부완료 직접 전이 TC
- 지시여부 Y 없이 검토완료→입금대기 직접 전이 TC

### WHEN-THEN 핵심 규칙
- WHEN 접수 클릭 → THEN 신건 발수값 TASK uid 전송 + 연차건 KEY 생성
- WHEN 공동출원인 Y → THEN 공동고객 확인요청 메일 발송
- WHEN Ack 발송 → THEN 이관연차·이관연차마감일·존속기간만료일 업데이트
- WHEN 리마인더 최신알 경과 → THEN 리마인더 자동 재발송
- WHEN 오청구 발생 → THEN 청구 재발송 (미납부 상태 유지)
- WHEN 지시여부 N → THEN 납부포기 전이
