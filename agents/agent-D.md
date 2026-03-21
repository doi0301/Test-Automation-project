# Agent D — FZ043~FZ048 담당 (워크스페이스)

**모델**: Claude Sonnet (AG에서 선택)
**담당 그룹**: F(FZ043~048)
**예상 TC 수**: ~20개

---

## 로딩 파일

```
RULES.md
specs/FZ043.md ~ specs/FZ048.md
conditions/FZ043-표1.csv ~ conditions/FZ048-표1.csv
permissions/iz-annuity-permissions.md
domain/ipazon-workflow-rules.md
rules/tc-conventions.md
rules/forbidden-patterns.md
```

---

## 담당 화면 목록

| 화면코드 | 화면명 | 복잡도 | TC 집중 포인트 |
|---------|--------|--------|--------------|
| FZ043 | 연차료 워크스페이스 | 상 | WORK 처리 전체 조건 |
| FZ044 | 비용 현황리스트 | 중 | 리스트 출력 조건 |
| FZ045 | 비용 워크스페이스 | 상 | 비용 WORK 처리 조건 |
| FZ046 | 비용 현황리스트 및 워크스페이스 출력항목 | 중 | 필드 출력 포맷 |
| FZ047 | WORK(1) | 상 | WORK 처리 조건 분기 |
| FZ048 | WORK(2) | 상 | WORK 처리 조건 분기 |

---

## TC 생성 핵심 포인트

### FZ043/FZ045 (워크스페이스) — WORK 처리 전체

ipazon-workflow-rules.md 기반:
- 각 WORK 처리자 조건 확인 (권한 매트릭스 기준)
- WORK별 필수값 미입력 시 처리 불가 TC

### 뷰 전용 역할 TC 집중

워크스페이스는 뷰 전용 역할(출원리더, 비용일반 등)의 접근 확인 TC 필수:
- 처리 버튼 미출력 확인
- 열람만 가능 확인

### FZ047/FZ048 (WORK) — IPAZON_워크항목.xlsx 참조

각 WORK 항목의:
- 필수값 여부 (O/X)
- 입력방식 (셀렉트/텍스트/자동생성)
- 출현 조건 (권리종류·상태 조건부 출현)

---

## Agent 공유 규칙

- TC ID 채번: `{화면코드}-TC-{3자리순번}` (예: FZ043-TC-001)
- TC 출력 형식: rules/tc-conventions.md
- 금지 패턴: rules/forbidden-patterns.md
- 품질 기준: rules/quality-standards.md
