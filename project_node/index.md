# project_[node_name]/ — 노드 앱 구조 인덱스

- 이 파일은 `project_[node_name]/` 디렉토리의 폴더 체계, 구성 요소의 역할, Agent 소유 영역, 상호작용 흐름, 확장 규칙을 정의합니다.
- 새 노드 생성 시 이 파일을 복사하고 `[node_name]`을 실제 노드명으로 교체하세요.
- 실제 구조의 기준: `ARCHITECTURE.md` > 이 파일 > 코드

---

## 0. 문서 적용 원칙

| 원칙 | 내용 |
|---|---|
| 구조 기준 | 이 문서는 `project_[node_name]/` 내부 구조의 기준 문서다 |
| 상위 정렬 | 전체 시스템 구조는 `ARCHITECTURE.md`를 우선한다 |
| Agent 경계 | Agent별 소유 영역은 `AGENTS.md`의 원칙을 따른다 |
| 단일 경로 | 하나의 기능은 하나의 기준 파일만 가진다 |
| 최소 중복 | 폴더 트리, 역할표, 흐름도에서 동일 설명을 반복하지 않는다 |
| 코드 종속 금지 | 코드 구현이 이 문서와 충돌하면 구조 정의를 먼저 확인한다 |

---

## 1. 폴더 체계

```text
project_[node_name]/
├── index.md                               ← 이 파일
├── _context/                              ← 노드 전용 하네스 / AI 제어 레이어
│   ├── protocol/                          ← Protocol 버전 아카이브
│   │   ├── protocol-[node_name]-v1.txt
│   │   └── protocol-[node_name]-v[N].txt  ← 현재 사용 버전
│   ├── [knowledge-doc].txt                ← 선택 Knowledge Doc
│   ├── brand-guidelines.md                ← 브랜드 정체성 및 시각 원칙
│   ├── business-context.md                ← 비즈니스 배경 및 서비스 가치
│   └── design-style-guide-node.md         ← 노드별 디자인 정책 Part B
├── app/                                   ← Next.js App Router 레이어
│   ├── page.tsx                           ← 사용자 입력 UI
│   ├── layout.tsx                         ← 페이지 공통 레이아웃
│   ├── generate/                          ← AI API 호출 라우트
│   │   └── route.ts
│   └── components/                        ← 노드 전용 UI 컴포넌트
└── lib/                                   ← 공유 로직 레이어
    ├── prompt.ts                          ← 시스템 프롬프트 조합 함수 (필수)
    ├── types.ts                           ← 노드 전용 타입 정의 (선택)
    ├── [feature]Utils.ts                  ← 기능별 유틸리티 (선택)
    ├── index.ts                           ← lib/ 공개 API 배럴 (선택)
    └── styles/                            ← lib 전용 스타일 토큰 (선택)
````

---

## 2. 구성 요소 역할

| 경로                                    | 담당 Agent    | 역할                                        | 핵심 규칙                                        |
| ------------------------------------- | ----------- | ----------------------------------------- | -------------------------------------------- |
| `_context/protocol/`                  | AGENT A     | Principle Protocol 보관                     | 기존 버전 수정·삭제 금지                               |
| `_context/[knowledge-doc].txt`        | AGENT A     | 선택 Knowledge Doc                          | Protocol 보조 문서로만 사용                          |
| `_context/brand-guidelines.md`        | AGENT C     | 브랜드 정체성, 컬러, 타이포그래피, 보이스 앤 톤 정의           | 브랜드 방향 변경 시 수정                               |
| `_context/business-context.md`        | AGENT A     | 시장 문제, 서비스 비전, 타겟 사용자, 비즈니스 가치 정의         | Protocol 작성 기준으로 사용                          |
| `_context/design-style-guide-node.md` | AGENT C     | 노드별 디자인 예외사항 및 추가 규칙 정의                   | CAI 공통 디자인 시스템과 함께 적용                        |
| `app/page.tsx`                        | AGENT C     | 사용자 입력 UI, 결과 렌더링                         | AI API 직접 호출 금지                              |
| `app/layout.tsx`                      | AGENT C     | 폰트, 메타데이터, 공통 레이아웃                        | AI 호출·비즈니스 로직 포함 금지                          |
| `app/generate/route.ts`               | AGENT A     | 입력 검증, Prompt 조합 호출, Gemini API 요청, 응답 반환 | `buildSystemPrompt()` 없이 Gemini API 직접 호출 금지 |
| `app/components/`                     | AGENT C     | 노드 전용 React 컴포넌트                          | Protocol 직접 접근 금지                            |
| `lib/prompt.ts`                       | AGENT A     | 시스템 프롬프트 조합 함수                            | Prompt 조합의 단일 기준 파일                          |
| `lib/types.ts`                        | AGENT A     | 노드 전용 타입 정의                               | 필요 시 선택 사용                                   |
| `lib/[feature]Utils.ts`               | AGENT A / C | 기능별 유틸리티                                  | 단일 책임 원칙 준수                                  |
| `lib/index.ts`                        | AGENT A     | lib 공개 API 배럴                             | 선택 사용                                        |
| `lib/styles/`                         | AGENT C     | lib 전용 스타일 토큰                             | UI 레이어와 역할 혼합 금지                             |

---

## 3. Protocol 관리 규칙

| 항목    | 규칙                                                     |
| ----- | ------------------------------------------------------ |
| 파일명   | `protocol-[node_name]-v[N].txt` 형식                     |
| 위치    | `_context/protocol/`                                   |
| 담당    | AGENT A                                                |
| 수정 방식 | 기존 파일 수정 금지, 새 버전 파일 생성                                |
| 삭제 규칙 | 이전 버전 삭제 금지                                            |
| 사용 경로 | `lib/prompt.ts` → `app/generate/route.ts` → Gemini API |

---

## 4. Prompt 조합 규칙

`lib/prompt.ts`는 시스템 프롬프트 조합의 단일 기준 파일입니다.

```ts
function buildSystemPrompt(
  principleProtocol: string,
  knowledgeDocs?: string[]
): string
```

| 규칙         | 내용                                                              |
| ---------- | --------------------------------------------------------------- |
| 단일 함수      | 시스템 프롬프트는 `buildSystemPrompt()`를 통해서만 구성한다                      |
| 코드 하드코딩 금지 | Protocol 내용을 `route.ts`, `page.tsx`, `components/`에 직접 작성하지 않는다 |
| 파일명 고정     | `lib/buildSystemPrompt.ts`는 사용하지 않는다                            |
| 계층 우선순위    | `Principle Protocol > Knowledge Docs > User Input`              |

---

## 5. 구성 요소 상호작용 흐름

```text
사용자 (브라우저)
    │
    │  이미지 / 텍스트 / 파라미터 전송
    ▼
app/page.tsx
    │
    │  fetch('/generate')
    ▼
app/generate/route.ts
    │
    │  입력 검증 + 형식 변환
    │
    ├─ lib/prompt.ts
    │       └─ buildSystemPrompt()
    │
    ├─ _context/protocol/protocol-[node_name]-v[N].txt  (필수)
    ├─ _context/[knowledge-doc].txt                     (선택)
    ├─ _context/brand-guidelines.md                     (선택)
    ├─ _context/business-context.md                     (선택)
    └─ _context/design-style-guide-node.md              (선택)
    │
    ▼
Gemini API
    │
    │  model: ARCHITECTURE.md 또는 환경 설정 기준
    │  systemInstruction: buildSystemPrompt(...)
    │  contents: [{ role: "user", parts: [입력 데이터] }]
    ▼
app/generate/route.ts
    │
    │  응답 후처리
    ▼
app/page.tsx
    │
    │  결과 렌더링
    ▼
사용자 (브라우저)
```

---

## 6. Agent별 소유 영역

| 파일 / 폴더                               | 담당 Agent    | 권한          |
| ------------------------------------- | ----------- | ----------- |
| `_context/protocol/`                  | AGENT A     | 작성·버전업·보존   |
| `_context/[knowledge-doc].txt`        | AGENT A     | 작성·수정       |
| `_context/brand-guidelines.md`        | AGENT C     | 작성·수정       |
| `_context/business-context.md`        | AGENT A     | 작성·수정       |
| `_context/design-style-guide-node.md` | AGENT C     | 작성·수정       |
| `app/page.tsx`                        | AGENT C     | 구현·수정       |
| `app/layout.tsx`                      | AGENT C     | 구현·수정       |
| `app/generate/route.ts`               | AGENT A     | 구현·수정       |
| `app/components/`                     | AGENT C     | 구현·수정       |
| `lib/prompt.ts`                       | AGENT A     | 구현·수정       |
| `lib/types.ts`                        | AGENT A     | 작성·수정       |
| `lib/[feature]Utils.ts`               | AGENT A / C | 역할 기준 작성·수정 |
| `lib/index.ts`                        | AGENT A     | 작성·수정       |
| `lib/styles/`                         | AGENT C     | 작성·수정       |

---

## 7. 선택적 폴더

기본 구조 외에 노드 특성에 따라 폴더를 추가할 수 있습니다.

단, 추가 시 이 문서에 역할, 추가 조건, 담당 Agent를 기록해야 합니다.

| 폴더                     | 사용 조건                                       | 담당 Agent    |
| ---------------------- | ------------------------------------------- | ----------- |
| `public/`              | 정적 에셋, 폰트, 아이콘, UI 표시용 샘플 이미지가 필요할 때        | AGENT C     |
| `types/`               | 입출력 타입이 복잡하거나 여러 파일에서 공유될 때                 | AGENT A     |
| `_context/samples/`    | Stage B 동적 테스트 케이스 또는 Protocol 검증 샘플이 필요할 때 | AGENT B / A |
| `lib/imageProcessing/` | 입력 이미지 전처리 또는 출력 이미지 후처리가 필요할 때             | AGENT A     |
| `lib/parsers/`         | 복합 입력 또는 AI 응답 파싱 로직이 필요할 때                 | AGENT A     |
| `lib/renderers/`       | 다이어그램, SVG, Canvas, 이미지 오버레이 렌더링이 필요할 때     | AGENT C     |
| `templates/`           | A3 보고서, A0 패널, 증명서 등 고정 출력 템플릿이 필요할 때       | AGENT C / A |

---

## 8. 폴더 추가 규칙

| 규칙          | 내용                                                                                 |
| ----------- | ---------------------------------------------------------------------------------- |
| 단일 책임       | 폴더 하나는 명확히 구분되는 하나의 관심사만 담는다                                                       |
| 명칭 명확성      | 폴더명은 내부 파일의 성격을 즉시 드러내야 한다                                                         |
| Agent 소유 명시 | 폴더 추가 시 담당 Agent를 함께 기록한다                                                          |
| 레이어 경계 준수   | UI는 `app/`, API는 `app/generate/`, Prompt 조합은 `lib/prompt.ts`, 정적 에셋은 `public/`에 둔다 |
| 문서 선행       | 구조 변경 후 코드 수정 전에 이 파일을 먼저 갱신한다                                                     |

---

## 9. 구조 변경 시 주의사항

| 변경 항목        | 규칙                                                                                                         |
| ------------ | ---------------------------------------------------------------------------------------------------------- |
| 폴더·파일 추가     | 이 파일에 역할·조건·담당 Agent를 기록한 뒤 추가한다                                                                           |
| `lib/` 파일 추가 | 단일 책임·명확한 명칭 원칙을 지킨다                                                                                       |
| Protocol 수정  | 반드시 버전 번호를 올려 새 파일을 생성한다                                                                                   |
| Agent 경계     | 자신의 소유 영역 밖 파일을 임의로 수정하지 않는다                                                                               |
| 신규 노드 생성     | 저장소의 `project_node/` 템플릿을 복사한 뒤 실제 노드 경로인 `project_[node_name]/` 형식으로 rename하고, `[node_name]` 플레이스홀더를 교체한다 |
| 모델 변경        | `ARCHITECTURE.md` 또는 환경 설정 기준을 따른다                                                                         |
| API Key 관리   | `GEMINI_API_KEY`는 환경변수로만 접근한다                                                                              |

---

## 10. 금지 패턴

```text
- `app/page.tsx`에서 Gemini API 직접 호출
- `app/components/`에서 Gemini API 직접 호출
- `app/generate/route.ts`에서 `buildSystemPrompt()` 없이 Gemini API 호출
- `route.ts` 또는 UI 코드에 Protocol 내용 하드코딩
- `lib/buildSystemPrompt.ts` 생성
- 기존 Protocol 파일 덮어쓰기
- 기존 Protocol 파일 삭제
- Agent 소유 영역 밖 파일 임의 수정
- 사용자 업로드 이미지를 서버에 영구 저장
- API Key를 코드에 하드코딩
```

---

## 11. 최소 필수 구조

```text
project_[node_name]/
├── index.md
├── _context/
│   └── protocol/
│       └── protocol-[node_name]-v1.txt
├── app/
│   ├── page.tsx
│   ├── layout.tsx
│   └── generate/
│       └── route.ts
└── lib/
    └── prompt.ts
```

---

## 12. 최종 기준
구조 판단 우선순위는 다음과 같습니다.

```text
1. ARCHITECTURE.md
2. project_[node_name]/index.md
3. AGENTS.md의 Agent 경계 원칙
4. 실제 코드
```

코드는 이 문서를 따라야 하며, 코드 구현이 이 문서와 충돌할 경우 구조 정의를 먼저 검토·수정합니다.

---


