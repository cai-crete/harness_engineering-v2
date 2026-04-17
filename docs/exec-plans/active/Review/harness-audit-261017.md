# [EXEC-PLAN] 하네스 전체 감사 및 범용화 작업
**유형:** Review  
**작성일:** 2026-04-17  
**담당 에이전트:** AGENT B (검증) + AGENT A (수정)  
**상태:** 계획 수립 완료 — 실행 대기 중 (사용자 승인 필요)

---

## 배경 및 목적

현재 하네스(`harness_engineering-v2`)는 10개 노드 전체에 공통 적용될 인프라임에도
불구하고, 내부 파일 곳곳에 **N01, N10, print, 기획자들** 등 특정 노드 번호·명칭이
하드코딩되어 있다. 또한 `AGENTS.md`가 실제 존재하지 않는 파일명을 참조하는
**Critical 정합성 오류**가 발견되었다.

이 exec-plan은 세 가지 작업을 단계별로 수행한다:

1. **Phase 1** — 노드 특정 참조 제거 (범용화)
2. **Phase 2** — hooks / references 정합성 교정
3. **Phase 3** — AGENTS.md / ARCHITECTURE.md 구조 개선

---

## 발견된 문제 목록

### 🔴 CRITICAL — 파일명 불일치 (에이전트 동작 실패 유발)

| # | 위치 | 현재 참조 | 실제 파일명 |
|---|------|-----------|------------|
| C-1 | `AGENTS.md` 에이전트 표 + 디렉토리 목록 | `loop-b-execution-agent.txt` | `loop-review-execution-agent.txt` |
| C-2 | `AGENTS.md` 에이전트 표 + 디렉토리 목록 | `loop-b-verification-agent.txt` | `loop-review-verification-agent.txt` |
| C-3 | `AGENTS.md` 에이전트 표 + 디렉토리 목록 | `loop-c-design-agent.txt` | `loop-frontend-design-agent.txt` |
| C-4 | `.claude/hooks/session-init.sh` line 18 | `loop-b-execution-agent.txt` | `loop-review-execution-agent.txt` |
| C-5 | `loop-review-verification-agent.txt` line 42 | `loop-a-verification-agent.txt` | `loop-protocol-verification-agent.txt` |
| C-6 | `loop-orchestrator.txt` line 95 | `loop-prontend-design-agent.txt` | `loop-frontend-design-agent.txt` |

### 🟠 HIGH — AGENTS.md 구조 누락

| # | 문제 |
|---|------|
| H-1 | `loop-orchestrator.txt` 가 AGENTS.md에 전혀 언급되지 않음 — 에이전트가 오케스트레이터 존재 자체를 모름 |
| H-2 | `loop-protocol-verification-agent.txt` 가 AGENTS.md에 전혀 언급되지 않음 |
| H-3 | AGENTS.md 에이전트 표에 Orchestrator 역할이 없음 |
| H-4 | AGENTS.md의 "Loop A" = ralph-wiggum 자가반복 루프로 정의하지만, references에서 "Loop A" = Protocol 정합성 4-체크로 다르게 사용됨 → 루프 명칭 충돌 |

### 🟡 MID — 노드 특정 참조 (범용화 필요)

| # | 위치 | 내용 |
|---|------|------|
| M-1 | `AGENTS.md` 디렉토리 목록 line 175 | `N10-print.md` → `[node].md` |
| M-2 | `ARCHITECTURE.md` lines 54, 64-73, 121-152 | N10 전용 예외 섹션 → 비표준 노드 일반화 |
| M-3 | `loop-review-execution-agent.txt` lines 61-72 | `N01 기준` + `N02~N10` 구체 구현 경로 하드코딩 |
| M-4 | `loop-review-verification-agent.txt` lines 88-111 | `N01` / `N02~N10` 분기 체크리스트 |
| M-5 | `loop-frontend-design-agent.txt` lines 35-36, 51-66 | `N01`, `project.XX`, `PRD_기획자들.md` 참조 |
| M-6 | `loop-review-verification-agent.txt` line 267 | `docs/progress/N[node-name]-progress.md` — N 접두사 |

### 🔵 LOW — 개선 사항

| # | 위치 | 내용 |
|---|------|------|
| L-1 | `post-protocol.js` line 26 | 패턴 `/protocol-.*\.txt/` — `.md` 확장자 Protocol 파일 미감지 |
| L-2 | `AGENTS.md` ralph-wiggum 섹션 | Loop A 명칭 충돌을 해소하기 위해 "Loop Protocol"로 재명명 |
| L-3 | `ARCHITECTURE.md` | "Project-10" 언급 → 프로젝트 의존 없이 일반화 가능 |

---

## Phase 1 — 노드 특정 참조 제거 (범용화)

**목표:** 어떤 노드 작업을 시작해도 하네스 파일이 특정 노드를 가정하지 않도록 한다.

- [x] `AGENTS.md` — 디렉토리 목록의 `N10-print.md` → `[node_name].md` 수정 (M-1)
- [x] `ARCHITECTURE.md` — N10/비표준 노드 섹션 전체 삭제; Claude API → Gemini 2.5 Pro 전면 교체 (M-2, 사용자 방향 변경)
- [x] `loop-review-execution-agent.txt` — `N01 기준` 구체 경로 제거; 범용 Gemini 경로로 대체 (M-3)
- [x] `loop-review-execution-agent.txt` — CONSTRAINTS REMINDER `N01`/`N02~N10` 분기 → 단일 일반 체크 통합 (M-3)
- [x] `loop-review-verification-agent.txt` — V3 IMPLEMENTATION CHECK `N01 기준`/`N02~N10` 분기 → product-spec 기반 일반 체크리스트 (M-4)
- [x] `loop-frontend-design-agent.txt` — `project.XX` → `project_[node_name]`; N01-specific 빌드 시퀀스 → 범용 Next.js 빌드 시퀀스 (M-5)
- [x] `loop-review-verification-agent.txt` PA-1 — `N[node_name]-progress.md` → `[node_name]-progress.md` (M-6)
- [x] **추가** `api-patterns-llms.txt` — Claude/Anthropic SDK 패턴 전체 → Gemini SDK 패턴 재작성
- [x] **추가** `FRONTEND.md` — Anthropic SDK 예제 → Gemini SDK; 프로젝트 경로 수정
- [x] **추가** `SECURITY.md` — ANTHROPIC_API_KEY → GEMINI_API_KEY; system → systemInstruction
- [x] **추가** `core-beliefs.md` — N02/N03/N05/N08 노드번호 제거
- [x] **추가** `protocol-design-guide.md` — `[node-name]` → `[node_name]`; 경로 수정
- [x] **추가** `SKILL.md` (code-reviewer) — `[node-name]` → `[node_name]`
- [x] **추가** `product-specs/index.md` — N01~N10 표기 제거; 범용 표기 교체
- [x] **추가** `product-specs/node-spec-template.md` — N[XX] 표기 제거; `[node_name]` 통일
- [x] **추가** `project_node/_context/business-context.md` — N10 print 특정 표기 제거
- [x] **추가** 모든 references 파일 `[node-name]` → `[node_name]` (전체 치환)

---

## Phase 2 — Hooks / References 정합성 교정

**목표:** 에이전트가 파일 로드를 시도할 때 실제 존재하는 파일을 가리키도록 한다.

> **파일명 결정 (확정):**
> `loop-frontend-design-agent.txt` 파일명 유지. "Prontend"는 오표기이며 "Frontend"로 통일한다.
> → "Loop Prontend" → **"Loop Frontend"**, `loop-prontend-*` → `loop-frontend-*`

---

### 2-A. Critical 파일명 불일치 수정

**2-A-1. AGENTS.md 에이전트 표 파일명 수정**
- [x] AGENT A 발동 파일: `loop-b-execution-agent.txt` → `loop-review-execution-agent.txt` (C-1)
  - 위치: `## Agent Identity` 테이블 AGENT A 행
- [x] AGENT B 발동 파일: `loop-b-verification-agent.txt` → `loop-review-verification-agent.txt` (C-2)
  - 위치: `## Agent Identity` 테이블 AGENT B 행
- [x] AGENT C 발동 파일: `loop-c-design-agent.txt` → `loop-frontend-design-agent.txt` (C-3)
  - 위치: `## Agent Identity` 테이블 AGENT C 행

**2-A-2. AGENTS.md 세션 유형별 로드 파일명 수정**
- [x] AGENT A 행: `loop-b-execution-agent.txt` → `loop-review-execution-agent.txt`
- [x] AGENT B 행: `loop-b-verification-agent.txt` → `loop-review-verification-agent.txt`
- [x] AGENT C 행: `loop-c-design-agent.txt` → `loop-frontend-design-agent.txt`

**2-A-3. AGENTS.md 디렉토리 목록 파일명 수정**
- [x] `loop-b-execution-agent.txt` → `loop-review-execution-agent.txt`
- [x] `loop-b-verification-agent.txt` → `loop-review-verification-agent.txt`
- [x] `loop-c-design-agent.txt` → `loop-frontend-design-agent.txt`
- [x] 목록에 `loop-orchestrator.txt`, `loop-protocol-verification-agent.txt` 항목 추가

**2-A-4. `.claude/hooks/session-init.sh` 수정**
- [x] line 18: `loop-b-execution-agent.txt` → `loop-review-execution-agent.txt` (C-4)
  - 대상 문자열: `"docs/references/loop-b-execution-agent.txt 를 로드하세요."`

**2-A-5. `loop-orchestrator.txt` — "Prontend" → "Frontend" 전면 수정 (C-6 재처리)**
- [x] `Loop Prontend` → `Loop Frontend` (전체 치환)
- [x] `loop-prontend-handoff` → `loop-frontend-handoff` (전체 치환)
- [x] `loop-prontend-report` → `loop-frontend-report` (전체 치환)
- [x] `loop-prontend-design-agent.txt` → `loop-frontend-design-agent.txt` (Phase 1에서 이미 수정됨 — 확인)
- [x] `active/Prontend/` → `active/Frontend/` (전체 치환)

**2-A-6. 완료 확인**
- [x] `loop-review-verification-agent.txt` — `loop-a-verification-agent.txt` → `loop-protocol-verification-agent.txt` (Phase 1 완료)

---

### 2-B. Loop 명칭 충돌 해소

**현재 충돌 상황:**

| 위치 | "Loop A" 사용 의미 |
|------|--------------------|
| `AGENTS.md` ralph-wiggum 섹션 | Stop hook 기반 자가반복 루프 (ralph-wiggum) |
| `loop-protocol-verification-agent.txt` 제목 | Protocol 정합성 4-체크 검증 에이전트 |
| `loop-review-verification-agent.txt` V1 헤더 | V1 단계에서 호출하는 4-체크 실행 |

**확정 명칭 체계:**

| 루프 명칭 | 구현 | 역할 |
|-----------|------|------|
| **Loop Protocol** | ralph-wiggum (Stop hook) | Protocol 파일 자가반복 교정 루프 |
| **Loop Review** | Orchestrator → Agent A/B | Protocol + Node App 구현·검증 루프 |
| **Loop Frontend** | Orchestrator → Agent C/B | 프론트엔드 구현·검증 루프 |

- [x] `loop-review-verification-agent.txt` — V1 헤더 수정:
  - 현재: `--- PHASE V1: LOOP A (Protocol Consistency) ---`
  - 변경: `--- PHASE V1: PROTOCOL CONSISTENCY (loop-protocol-verification-agent.txt 실행) ---`
- [x] `loop-review-verification-agent.txt` — "Loop Prontend" → "Loop Frontend" 전체 치환
- [x] `loop-orchestrator.txt` 상단 역할 설명 — "Loop Review and Loop Prontend" → "Loop Review and Loop Frontend" 수정
- [x] `loop-frontend-design-agent.txt` — "Loop Prontend" → "Loop Frontend" 전체 치환
- [x] `AGENTS.md` ralph-wiggum 섹션 — "Loop A" → "Loop Protocol" 변경 (Phase 3에서 처리)

---

### 2-C. Low 개선 (선택 적용)

- [ ] `post-protocol.js` line 26 — 패턴 확장: `/protocol-.*\.(txt|md)/` (L-1)
  - 현재: `if (/protocol-.*\.txt/.test(filePath))`
  - 변경: `if (/protocol-.*\.(txt|md)/.test(filePath))`
  - *사용자 확인 후 적용*

---

## Phase 3 — AGENTS.md 전면 재구조화

**목표:** AGENTS-yh.md 참조본을 기준으로 AGENTS.md를 재작성한다. 단, 아래 두 가지 사항을 반영한다:
1. 파일명은 `loop-frontend-design-agent.txt` 유지, "Prontend" 표기 없음
2. 프로젝트 식별 정보 섹션은 현재 상태 유지. 신규 섹션이 필요하면 AGENTS.md 참조 경로 방식 사용.

---

### 비교 분석 — 현재 vs AGENTS-yh.md

| 섹션 | 현재 AGENTS.md | AGENTS-yh.md (참조) | 조치 |
|------|----------------|---------------------|------|
| 프로젝트 식별 정보 | 현재 상태 | 유사하나 일부 다름 | **유지** (사용자 지시) |
| `## Agent Identity` 표 | AGENT A/B/C, 파일명 오류 | ORCHESTRATOR 추가, 파일명 교정 | 전면 교체 |
| 에이전트 경계 원칙 | 4개 항목 (A/B/C + 일반) | 5개 항목 (+Orchestrator, +Loop Protocol) | 항목 추가 |
| 세션 시작 순서 | 6행 테이블 | 7단계 순차 목록 (progress 파일 읽기 포함) | 형식 변경 |
| 세션 유형별 우선 로드 | 7행 (stale 경로) | 4행 (역할별 간결) | 간소화 |
| 핵심 금지 행동 | 없음 (CLAUDE.md에 일부 있음) | 신규 섹션 | **별도 파일 작성 + AGENTS.md 참조** |
| 참조 파일 인덱스 | 디렉토리 구조로 대체 | 신규 섹션 (표) | **AGENTS.md 직접 추가** |
| 실행 환경 | 없음 | 신규 섹션 | **별도 파일 작성 + AGENTS.md 참조** |
| Loop Protocol 섹션 | ralph-wiggum 상세 + 반복 사용법 | 삭제 (plugin 문서로) | **3-루프 요약 표 + 명령어만 유지** |
| 디렉토리 구조 섹션 | 전체 트리 코드블록 | 삭제 (코드베이스에서 파악) | **섹션 삭제** |

---

### 3-A. 에이전트 표 전면 교체 + 섹션명 변경

**섹션명 변경:** `## Agent Identity` → `## 에이전트 역할 및 경계`

**교체 대상 표 (확정):**

| 에이전트 | 명칭 | 책임 | 발동 파일 |
|----------|------|------|-----------|
| **ORCHESTRATOR** | 오케스트레이터 | Loop Review·Loop Frontend 이관·Agent A 역할 직접 수행·Agent B/C 서브에이전트 호출 | `docs/references/loop-orchestrator.txt` |
| **AGENT A** | 실행 에이전트 | Protocol 작성·Node App(API Route, buildSystemPrompt) 구현·결함 수정 | `docs/references/loop-review-execution-agent.txt` |
| **AGENT B** | 검증 에이전트 | Protocol + Node App 독립 검증·PASS/FAIL 판정·Progress 갱신·exec-plan 완료 처리 | `docs/references/loop-review-verification-agent.txt` |
| **AGENT C** | 디자인 에이전트 | UI/UX·프론트엔드 구현·브랜드 컴플라이언스·컴포넌트 빌드 | `docs/references/loop-frontend-design-agent.txt` |

- [x] 현재 AGENT A/B/C 표 → 위 ORCHESTRATOR + AGENT A/B/C 4행 표로 교체
- [x] 섹션명 `Agent Identity` → `에이전트 역할 및 경계` 변경

---

### 3-B. 에이전트 경계 원칙 수정

**교체 내용 (5항목):**

> **에이전트 간 경계 원칙**
> - AGENT A만 Protocol 파일을 작성·수정한다
> - AGENT B만 PASS/FAIL 판정을 발급한다
> - AGENT C만 UI/UX·프론트엔드 레이어를 소유한다
> - ORCHESTRATOR는 Loop Review와 Loop Frontend에서만 동작한다
> - Loop Protocol은 ralph-wiggum이 담당한다

- [x] 현재 4항목 → 위 5항목으로 교체

---

### 3-C. 세션 시작 순서 재작성

**목표:** 테이블 형식 → 7단계 순차 목록으로 변경, progress 파일 읽기 추가

**변경 내용 (확정):**

```
세션 시작 시 아래 순서를 능동적으로 수행합니다:

1. `AGENTS.md` (이 파일) — 역할·행동 기준 확립
2. `ARCHITECTURE.md` — 기술 구조 파악
3. `docs/progress/[node_name]-progress.md` — 현재 노드 진행 상황 파악
4. `docs/design-docs/core-beliefs.md` — 철학적 기반 확인
5. `docs/design-docs/protocol-design-guide.md` — Protocol 표준 확인
6. `docs/product-specs/[node_name].md` — 노드 입출력 계약 파악
7. 해당 노드 Principle Protocol — Action Step 파악
```

- [x] 현재 6행 테이블 → 위 7단계 순차 목록으로 교체

---

### 3-D. 세션 유형별 우선 로드 간소화

**목표:** 7행 → 4행 (역할별 정리), 파일명 교정 포함

**변경 내용 (확정):**

| 세션 유형 | 우선 로드 |
|----------|----------|
| ORCHESTRATOR | `docs/references/loop-orchestrator.txt` → 해당 exec-plan |
| AGENT A — 실행 | `docs/references/loop-review-execution-agent.txt` → `product-specs/[node_name].md` |
| AGENT B — 검증 | `docs/references/loop-review-verification-agent.txt` → `loop-review-handoff-[node_name].md` |
| AGENT C — 디자인 | `docs/references/loop-frontend-design-agent.txt` → `_context/design-style-guide-node.md` |

- [x] 현재 7행 세션 유형별 테이블 → 위 4행으로 교체

---

### 3-E. 핵심 금지 행동 — 별도 파일 작성 + 참조 추가

**이유:** AGENTS-yh.md의 금지 행동 목록은 CLAUDE.md의 "절대 하지 말아야 할 것들"과 중복될 수 있다.
CLAUDE.md가 훅을 통해 자동 주입되므로, 금지 행동 목록은 `docs/OPERATIONS.md`(또는 `.claude/hooks/OPERATIONS.md`)에 정의하고 AGENTS.md에서 참조한다.

- [x] `.claude/hooks/OPERATIONS.md` 파일 내용 확인 (이미 존재) — 현재 내용 검토
- [x] 기존 OPERATIONS.md에 누락된 금지 행동 항목 추가:
  - Protocol 결함을 코드 레이어에서 보완
  - exec-plan 없이 작업 시작 (`_template.md` 수정 포함)
  - 불명확한 정보로 임의 추측하여 진행
  - 모르는 부분은 반드시 사용자에게 질문 후 진행
- [x] `AGENTS.md` 세션 루틴 섹션 끝에 참조 줄 추가 (CLAUDE.md 참조로 변경):
  - `> 상세 금지 행동 목록: `.claude/CLAUDE.md` 참조`

---

### 3-F. 참조 파일 인덱스 섹션 신규 추가

**위치:** 세션 루틴 섹션 뒤 (디렉토리 구조 섹션 교체)
**방침:** 디렉토리 구조 코드블록 대신, 파일별 목적을 설명하는 표로 교체

**추가 내용 (확정):**

```markdown
## 참조 파일 인덱스

| 파일 | 목적 |
|------|------|
| `ARCHITECTURE.md` | 기술 구조 지도·코드맵·하네스 디렉토리 |
| `docs/design-docs/core-beliefs.md` | CAI 핵심 철학 |
| `docs/design-docs/protocol-design-guide.md` | Protocol 작성·검증·컴플라이언스 표준 |
| `docs/references/loop-orchestrator.txt` | ORCHESTRATOR 지침 (Loop Review·Frontend 이관) |
| `docs/references/loop-review-execution-agent.txt` | AGENT A 실행 지침 (Loop Review) |
| `docs/references/loop-review-verification-agent.txt` | AGENT B 검증 지침 (Loop Review·Frontend) |
| `docs/references/loop-frontend-design-agent.txt` | AGENT C 디자인 지침 (Loop Frontend) |
| `docs/references/loop-protocol-verification-agent.txt` | Protocol 정합성 4-체크 검증 (Agent B V1에서 호출) |
| `.claude/plugins/ralph-wiggum/` | Loop Protocol 자가반복 검증 플러그인 |
| `.claude/hooks/OPERATIONS.md` | 에이전트 금지 행동 목록 |
| `docs/progress/[node_name]-progress.md` | 노드별 세션 간 컨텍스트 유지 |
```

- [x] 위 `## 참조 파일 인덱스` 섹션 추가 (디렉토리 구조 섹션 유지 + 인덱스 섹션 신규 추가)
  - 사용자 지시: 디렉토리 구조 섹션 유지 요청

---

### 3-G. Loop Protocol 섹션 간소화

**목표:** ralph-wiggum 중복 상세 설명 제거 → 3-루프 요약 표 + 명령어만 유지

**삭제 대상:**
- `ralph-wiggum Plugin` 섹션 전체 (동작 원리, Harness 설정값, 사용법 중복, 적합한 사용 시점)
- "Loop A 정합성 검토 (ralph-wiggum 실행):" 제목 줄 + 앞 코드블록 (line 67~81 — 아래 새 섹션과 중복)

**유지·교체 내용 (확정):**

```markdown
## 루프 체계

| 루프 | 트리거 | 담당 | 완결 조건 |
|------|--------|------|---------|
| **Loop Protocol** | Protocol 파일 저장 시 자동 | ralph-wiggum (Stop hook) | `VERIFIED` 출력 |
| **Loop Review** | active/Review/ exec-plan 존재 | ORCHESTRATOR + Agent A/B | PASS 판정 |
| **Loop Frontend** | active/Frontend/ exec-plan 존재 | ORCHESTRATOR + Agent C/B | PASS 판정 |

**Loop Protocol 실행 (ralph-wiggum):**

```bash
/ralph-loop "프로토콜 정합성 및 완결성 검토를 수행하라" --completion-promise "VERIFIED"
/cancel-ralph  # 루프 수동 취소
```

상세 동작 원리: `.claude/plugins/ralph-wiggum/` 참조
```

- [x] line 67~131 (중복 코드블록 + `ralph-wiggum Plugin` 섹션 전체) 삭제
- [x] 위 "루프 체계" 섹션으로 교체

---

## 실행 순서 (갱신)

```
Phase 2-A (파일명 수정 + "Prontend"→"Frontend" 전면 수정)
→ Phase 2-B (Loop 명칭 충돌 해소)
→ Phase 3-A/B (에이전트 표·경계 원칙)
→ Phase 3-C/D (세션 루틴 재작성)
→ Phase 3-E (OPERATIONS.md 보강 + 참조 추가)
→ Phase 3-F (참조 파일 인덱스 추가, 디렉토리 구조 섹션 제거)
→ Phase 3-G (Loop 섹션 간소화)
→ Phase 2-C (선택: post-protocol.js 패턴 확장)
```

Phase 2-A의 "Prontend"→"Frontend" 전면 수정은 다른 모든 작업의 전제조건이다.

---

## 완료 기준

- [x] `docs/references/` 내 모든 파일 간 상호 참조가 실제 파일명과 일치
- [x] `AGENTS.md`의 에이전트 표·세션 루틴·참조 인덱스가 실제 파일 구조와 일치
- [x] 하네스 파일 전체에서 노드 특정 문자열이 제거됨 (단, product-spec과 Protocol 파일은 예외)
- [x] `Prontend` / `Loop Prontend` / `prontend` 표기가 하네스 파일 어디에도 없음
- [x] 세 루프 명칭(Loop Protocol / Loop Review / Loop Frontend)이 모든 파일에서 일관되게 사용됨
- [x] 이 exec-plan의 모든 체크리스트 항목이 완료됨

---

*작성: AGENT B (검증·감사) — 2026-04-17*
*갱신: Phase 2/3 상세 계획 추가 (AGENTS-yh.md 참조 반영, Prontend→Frontend 수정 포함) — 2026-04-17*
