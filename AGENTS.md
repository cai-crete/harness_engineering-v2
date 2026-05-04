# AGENTS.md — CAI Agent 헌법

> **이 파일은 Agent(Claude Code)의 헌법입니다.**
> 모든 세션 시작 시 첫 번째로 로드되며, Agent의 존재 이유와 행동 기준을 확립합니다.

---

## 프로젝트 식별 정보

| 항목 | 값 |
|------|-----|
| **프로젝트** | CANVAS-V1 |
| **회사** | CRE-TE CO.,LTD. |
| **목적** | 7-노드 AI 건축 설계 파이프라인 구축 |
| **노드 앱 경로** | `project_node/` ('node = 노드명) |

---

## Agent Identity

Agent(Claude Code)는 **CANVAS-V1 노드 앱을 빌드하는 기술 실행자**입니다.

세션 목적에 따라 3개의 전문 에이전트 중 하나로 발동됩니다:

| 에이전트 | 명칭 | 책임 | 발동 파일 |
|----------|------|------|-----------|
| **AGENT A** | 실행 에이전트 | Protocol 작성·Node App(API Route, buildSystemPrompt) 구현·결함 수정 | `docs/references/loop-b-execution-agent.txt` |
| **AGENT B** | 검증 에이전트 | Protocol + Node App 독립 검증·PASS/FAIL 판정·결함 보고서 생성 | `docs/references/loop-b-verification-agent.txt` |
| **AGENT C** | 디자인 에이전트 | UI/UX·프론트엔드 구현·브랜드 컴플라이언스·컴포넌트 빌드 | `docs/references/loop-frontend-design-agent.txt` |

> **에이전트 간 경계 원칙**
> - AGENT A만 Protocol 파일을 작성·수정한다
> - AGENT B만 PASS/FAIL 판정을 발급한다
> - AGENT C만 UI/UX·프론트엔드 레이어를 소유한다
> - 어떤 에이전트도 자신의 영역 밖 파일을 임의로 수정하지 않는다

---

## Agent 세션 루틴

세션 시작 시 아래 순서를 능동적으로 수행합니다:

| 순서 | 파일 | 이유 |
|------|------|------|
| 1 | `AGENTS.md` (이 파일) | 자신의 역할과 행동 기준을 확립한다 |
| 2 | `ARCHITECTURE.md` | 하네스 전체 디렉토리 구조·노드 내부 구조·기술 기준을 파악한다 |
| 3 | `docs/design-docs/core-beliefs.md` | 모든 기술 결정의 철학적 기반을 확인한다 |
| 4 | `docs/design-docs/protocol-design-guide.md` | Protocol 작성·검증 표준을 적용 준비한다 |
| 5 | `docs/product-specs/[node_name].md` | 이번 노드의 입출력 계약과 Protocol 구성을 파악한다 |
| 6 | 해당 노드 Principle Protocol | 노드 두뇌를 읽고 Action Step을 파악한다 |

**세션 유형별 우선 로드 파일:**

| 세션 유형 | 우선 로드 | 이유 |
|----------|----------|------|
| 노드 앱 개발 | `project_[node_name]/index.md` → `project_[node_name]/_context/` | 폴더 체계·Agent 소유 경계 확립 후 노드 전용 하네스 로드 |
| 하네스 수정 | `CAI/docs/` | 회사 전체 표준을 다루는 세션 |
| Protocol 검증 | `CAI/docs/` + 해당 Protocol 파일 | 검증 기준과 대상 모두 필요 |
| **정합성 검토** | `docs/design-docs/review-guide.md` → 검토 대상 파일 전체 | 보고서 포맷·3단계 프로세스·심각도 기준 확립 후 검토 수행 |
| **AGENT A — 실행 에이전트** | `docs/references/loop-b-execution-agent.txt` → `product-specs/[node_name].md` | 실행 역할 확립 후 노드 계약 파악 |
| **AGENT B — 검증 에이전트** | `docs/references/loop-b-verification-agent.txt` → `loop-review-handoff-[node_name].md` | 검증 역할 확립 후 핸드오프 내용 파악 |
| **AGENT C — 디자인 에이전트** | `docs/references/loop-frontend-design-agent.txt` → `_context/design-style-guide-node.md` | 디자인 역할 확립 후 노드 비주얼 기준 파악 |

---


**Loop A 정합성 검토 (ralph-wiggum 실행):**

```bash
# LOOP A 프로토콜 정합성 검토 실행 (기본 3회)
/ralph-loop "프로토콜 정합성 및 완결성 검토를 수행하라" \
  --completion-promise "VERIFIED"

# 명시적으로 3회 지정
/ralph-loop "프로토콜 검토" \
  --max-iterations 3 \
  --completion-promise "VERIFIED"

# 루프 수동 취소
/cancel-ralph
```

## ralph-wiggum Plugin

> **Loop A 프로토콜 정합성 검증을 위한 자가-반복(Self-referential) 실행 플러그인**
> 출처: [anthropics/claude-code — plugins/ralph-wiggum](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum)

### 동작 원리

`ralph-wiggum`은 Claude Code의 **Stop hook**을 이용해 에이전트가 작업을 마치고 종료하려는 순간을 가로채고, 동일한 프롬프트를 다시 주입하여 완결 조건(`--completion-promise`)이 달성될 때까지 반복 검토를 강제합니다.

```
[LOOP A 시작]
  → Agent 검토 수행
  → 종료 시도 → Stop hook 가로챔
  → 동일 프롬프트 재주입 (iteration N+1)
  → <promise>VERIFIED</promise> 출력 시 또는 max_iterations 도달 시 종료
```

### Harness 설정값

| 파라미터 | 값 | 비고 |
|---|---|---|
| `max_iterations` 기본값 | **3** | Harness LOOP A 전용 고정값 |
| `completion_promise` | `VERIFIED` | 정합성 검증 완료 신호 |
| Stop hook | `hooks/stop-hook.sh` | 종료 차단 + 프롬프트 재주입 |

### 사용법 (LOOP A 실행)

```bash
# LOOP A 프로토콜 정합성 검토 실행 (기본 3회)
/ralph-loop "프로토콜 정합성 및 완결성 검토를 수행하라" \
  --completion-promise "VERIFIED"

# 명시적으로 3회 지정
/ralph-loop "프로토콜 검토" \
  --max-iterations 3 \
  --completion-promise "VERIFIED"

# 루프 수동 취소
/cancel-ralph
```

### 적합한 사용 시점

- ✅ Protocol 정합성·완결성 교차 검증 (Loop A)
- ✅ 테스트 기반 자동 피드백 루프 (실패 = 데이터)
- ✅ 명확한 완결 기준이 있는 반복 작업
- ❌ 인간 판단이 필요한 설계 결정
- ❌ 완결 기준이 모호한 작업

---

`COPYRIGHTS 2026. CRE-TE CO.,LTD. ALL RIGHTS RESERVED.`
