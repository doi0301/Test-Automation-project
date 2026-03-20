# TC Pilot — HIPAS 합금특허 검색 서비스

## 프로젝트 개요
현대제철 HIPAS 합금특허 검색 서비스 기획서(25SO01 V1.08)를 6 Layer로 구조화하여
SR01(검색결과), SR10(성분·특성 테이블), AF07(성분분석) 3개 화면의 TC를 자동 생성한다.

---

## 6 Layer 구조

| 레이어 | 경로 | 내용 |
|--------|------|------|
| L1 화면 스펙 | `specs/` | 화면 단위 Markdown + YAML frontmatter |
| L2 조건 테이블 | `conditions/` | 1행=1조건 CSV, tc_generated 추적 |
| L3 상태 전이 | `domain/state-transitions.csv` | allowed 기반 TC 생성 여부 결정 |
| L4 권한 매트릭스 | `permissions/hipas-permissions.md` | 역할별 TC 계정 + 기능 접근 |
| L5 데이터 필드 | `data/patent-list-fields.json` | 양방향 필드 = 정합성 TC 자동 |
| L6 도메인 규칙 | `domain/alloy-domain-rules.md` | MUST NOT = TC 금지, WHEN-THEN = TC 소스 |

---

## /tc:go {화면코드} — 레이어 로딩 순서

```
/tc:go SR01  (또는 SR10, AF07)
```

1. `specs/{화면코드}.md` 로딩 → 대상 화면 파악
2. YAML frontmatter `related` → 관련 화면 specs 추가 로딩
3. YAML frontmatter `conditions_table` → `conditions/` CSV 로딩
4. `domain/state-transitions.csv` → allowed: true 전이만 TC 대상
5. `permissions/hipas-permissions.md` → 권한별 TC 계정 + 버튼 노출 여부
6. `data/` JSON → direction: both 필드 = 데이터 정합성 TC 자동 생성
7. `domain/alloy-domain-rules.md` → MUST NOT 금지 패턴 적용, WHEN-THEN TC 소스
8. `rules/` 전체 → TC 생성 규칙 적용 (tc-conventions, quality-standards, forbidden-patterns)
9. TC 생성 완료 후 → `conditions/` CSV tc_generated: false → true 업데이트

---

## 품질 게이트 (TC 생성 완료 후 자동 실행)

```
체크 1: conditions/ CSV의 tc_generated: false 잔여 행 → 경고 출력
체크 2: state-transitions.csv의 allowed: true + tc_generated: false → 경고
체크 3: domain-rules.md의 MUST NOT 해당 TC 존재 → 삭제
체크 4: 권한 3종 커버리지 (관리자/특허담당자/발명자) 각 1개 이상
체크 5: 합금 도메인 규칙 TC 존재 확인
         - 최대값 없는 문헌 차트 처리 (AF07-C08)
         - 최소값 없는 문헌 차트 처리 (AF07-C09)
         - 미추출 문헌 Y축 제외 (AF07-C13)
체크 6: SR10 소수점 출력 포맷 검증 TC 존재
체크 7: E2E 연결 TC 1개 이상 (SR01 → SR10 → AF07 흐름)
```

---

## 커맨드

| 커맨드 | 사용 시점 | 설명 |
|--------|---------|------|
| `/tc:go {화면코드}` | 첫 화면, 복잡한 화면 | Tier별 점진 생성, 매 단계 확인 후 진행 |
| `/tc:batch {화면그룹}` | 패턴 잡힌 후 | Tier 완료 단위 배치 생성 |
| `/tc:fast` | 단순 반복 | 전체 자동 생성 (확인 없음) |

커맨드 상세 정의 → `commands/tc-go.md`

---

## Agent Teams 구조

```
팀 리드 (Claude Opus)     agents/agent-lead.md
├── Agent A (Sonnet)       agents/agent-A.md  — SR01 담당
├── Agent B (Sonnet)       agents/agent-B.md  — SR10 담당
└── Agent C (Sonnet)       agents/agent-C.md  — AF07 담당
```

이번 세미나: 단일 AG /tc:go 순차 실행 (파일 세팅 완료 상태)
확장 시: Claude Code Agent Teams로 병렬 실행 가능
