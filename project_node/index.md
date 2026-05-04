# project_[node_name]/ — 노드 앱 구조 인덱스

> 이 파일은 `project_[node_name]/` 디렉토리의 폴더 체계와 각 구성 요소의 역할, 상호작용 방식을 정의합니다.
> 새 노드 생성 시 이 파일을 복사하고 `[node_name]`을 실제 노드명으로 교체하세요.
> 실제 구조의 기준: `ARCHITECTURE.md` > 이 파일 > 코드

---

## 폴더 체계

```
project_[node_name]/
├── index.md                               ← 이 파일
├── _context/                              ← 노드 전용 하네스 (AI 제어 레이어)
│   ├── protocol/                          ← Protocol 버전 아카이브
│   │   ├── protocol-[node_name]-v1.txt
│   │   └── protocol-[node_name]-v[N].txt  ← 현재 사용 버전
│   ├── brand-guidelines.md               ← 브랜드 정체성 및 시각 원칙
│   ├── business-context.md               ← 비즈니스 배경 및 서비스 가치
│   └── design-style-guide-node.md        ← 노드별 디자인 정책 Part B
├── app/                                   ← Next.js App Router 레이어
│   ├── page.tsx                           ← 사용자 입력 UI
│   ├── layout.tsx                         ← 페이지 공통 레이아웃
│   ├── generate/                          ← AI API 호출 라우트
│   │   └── route.ts
│   └── components/                        ← 노드 전용 UI 컴포넌트
└── lib/                                   ← 공유 로직 레이어
    ├── prompt.ts                          ← 시스템 프롬프트 조합 함수 (필수)
    ├── types.ts                           ← 노드 전용 타입 정의 (선택)
    ├── [feature]Utils.ts                  ← 기능별 유틸리티 (선택, 복수 가능)
    ├── index.ts                           ← lib/ 공개 API 배럴 (선택)
    └── styles/                            ← lib 전용 스타일 토큰 (선택)
```

---

## 각 구성 요소의 역할

### `_context/` — 노드 전용 하네스

이 디렉토리는 AI의 동작을 제어하는 모든 문서를 보관합니다.
코드가 아닌 **텍스트 문서**로 AI 행동을 정의하는 것이 이 시스템의 핵심 원칙입니다.

#### `_context/protocol/` — Protocol 버전 아카이브

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT A |
| 역할 | Gemini AI의 `systemInstruction`에 주입되는 Principle Protocol 파일 보관 |
| 버전 규칙 | `protocol-[node_name]-v[N].txt` 형식. **이전 버전 삭제 금지** |
| 사용 경로 | `lib/buildSystemPrompt.ts` → `app/generate/route.ts` → Gemini API |

> **불변 규칙**: Protocol 파일은 삭제하지 않는다. 롤백 기반 및 비교 기준으로 영구 보존.

#### `_context/brand-guidelines.md` — 브랜드 정체성

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT C 참조 |
| 역할 | 이 노드의 브랜드 아이덴티티, 핵심 컬러, 타이포그래피, 보이스 앤 톤 정의 |
| 갱신 시점 | 브랜드 방향 변경 시 (노드 초기 설정 후 거의 불변) |

#### `_context/business-context.md` — 비즈니스 컨텍스트

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT A 참조 |
| 역할 | 이 노드가 해결하는 시장 문제, 서비스 비전, 타겟 사용자, 비즈니스 가치 정의 |
| 용도 | Protocol 작성 시 방향성 기준, 신규 세션의 컨텍스트 확립 |

#### `_context/design-style-guide-node.md` — 노드별 디자인 정책

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT C 작성 |
| 역할 | CAI 공통 디자인 시스템(`design-style-guide-CAI.md`) 대비 이 노드의 예외사항 및 추가 규칙 (Part B) |
| 관계 | `/shape` 실행 후 채워짐. Part A(CAI 공통)보다 이 문서가 우선 적용됨 |

---

### `app/` — Next.js App Router 레이어

UI와 API Route를 담당합니다. 사용자와의 접점 전체가 여기에 있습니다.

#### `app/page.tsx` — 사용자 입력 UI

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT C |
| 역할 | 사용자로부터 이미지·텍스트·파라미터를 수집하고, AI 결과를 렌더링 |
| 규칙 | AI API를 직접 호출하지 않는다. 반드시 `app/generate/route.ts`를 경유한다 |

#### `app/layout.tsx` — 페이지 공통 레이아웃

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT C |
| 역할 | 폰트 로드, 전역 메타데이터, 공통 래퍼 컴포넌트 적용 |

#### `app/generate/route.ts` — AI API 호출 라우트

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT A |
| 역할 | 사용자 입력 검증, `buildSystemPrompt()` 호출, Gemini API 요청, 응답 반환 |
| 규칙 | `buildSystemPrompt()` 없이 Gemini API를 직접 호출하지 않는다 |
| 보안 | `GEMINI_API_KEY`는 환경변수로만 접근. 사용자 이미지는 서버에 저장하지 않는다 |

#### `app/components/` — 노드 전용 UI 컴포넌트

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT C |
| 역할 | 이 노드에서만 사용되는 React 컴포넌트 (입력 패널, 결과 뷰어, 로딩 상태 등) |
| 디자인 기준 | `design-style-guide-CAI.md` + `_context/design-style-guide-node.md` |

---

### `lib/` — 공유 로직 레이어

`lib/`에는 `prompt.ts` 하나만 존재하는 것이 최소 구성이지만, 노드의 목적에 따라 파일을 자유롭게 추가할 수 있습니다. `app/` 레이어(UI·Route)와 분리되어야 하는 로직, 여러 컴포넌트에서 공유되는 유틸리티, 노드 전용 타입 정의가 이 디렉토리에 위치합니다.

#### `lib/prompt.ts` — 시스템 프롬프트 조합 함수 (필수)

| 속성 | 내용 |
|------|------|
| 담당 Agent | AGENT A |
| 역할 | Protocol 파일과 Knowledge Docs를 로드하여 Gemini `systemInstruction` 문자열 생성 |
| 시그니처 | `buildSystemPrompt(principleProtocol: string, knowledgeDocs?: string[]): string` |
| 불변 규칙 | 시스템 프롬프트는 이 함수를 통해서만 구성된다. Protocol 내용을 코드에 하드코딩하지 않는다 |

#### `lib/` 추가 파일 — 노드 목적에 따라 확장 가능

| 파일 예시 | 역할 | 담당 Agent |
|-----------|------|-----------|
| `types.ts` | 노드 전용 TypeScript 타입·인터페이스 정의 | AGENT A |
| `[feature]Utils.ts` | 특정 기능에 특화된 유틸리티 함수 모음 (예: `imageUtils.ts`, `htmlUtils.ts`) | AGENT A / C |
| `agentErrors.ts` | 노드 전용 에러 클래스·에러 코드 정의 | AGENT A |
| `export.ts` / `index.ts` | lib/ 내 공개 API를 한 곳에서 re-export하는 배럴 파일 | AGENT A |
| `styles/` | lib에서 사용하는 CSS 토큰·프린트 스타일 등 정적 스타일 파일 | AGENT C |

> **원칙**: `lib/`에 파일을 추가할 때는 파일명이 역할을 즉시 드러내야 하며, 단일 책임을 갖는다. `app/` 레이어의 렌더링·라우팅 로직은 `lib/`에 두지 않는다.

---

## 구성 요소 상호작용 흐름

```
사용자 (브라우저)
    │  이미지 / 텍스트 / 파라미터 전송
    ▼
app/page.tsx
    │  fetch('/generate')
    ▼
app/generate/route.ts
    │  입력 검증 + 형식 변환
    │
    ├─ lib/buildSystemPrompt.ts
    │       │
    │       ├─ _context/protocol/protocol-[node_name]-v[N].txt  (필수)
    │       ├─ _context/[knowledge-doc-1].txt                   (선택)
    │       └─ _context/[knowledge-doc-2].txt                   (선택)
    │       → "[Protocol]\n\n---\n\n[Knowledge Doc]..." 반환
    │
    ▼
Gemini API (gemini-2.5-pro)
    │  systemInstruction: buildSystemPrompt(...)
    │  contents: [{ role: "user", parts: [입력 데이터] }]
    ▼
app/generate/route.ts
    │  응답 후처리 (텍스트 / 이미지 / 구조화 데이터)
    ▼
app/page.tsx
    │  결과 렌더링
    ▼
사용자 (브라우저)
```

**계층 우선순위**: `Principle Protocol > Knowledge Docs > User Input`

---

## Agent별 소유 영역

| 파일 / 폴더 | 담당 Agent | 권한 |
|-------------|-----------|------|
| `_context/protocol/` | AGENT A | 작성·버전업·보존 |
| `_context/[knowledge-doc].txt` | AGENT A | 작성·관리 |
| `_context/brand-guidelines.md` | AGENT C | 참조 (초기 설정 후 변경 시 재작성) |
| `_context/business-context.md` | AGENT A | 참조 |
| `_context/design-style-guide-node.md` | AGENT C | 작성 (`/shape` 실행 후) |
| `app/page.tsx` | AGENT C | 구현·수정 |
| `app/layout.tsx` | AGENT C | 구현·수정 |
| `app/generate/route.ts` | AGENT A | 구현·수정 |
| `app/components/` | AGENT C | 구현·수정 |
| `lib/buildSystemPrompt.ts` | AGENT A | 구현·수정 |

---

## 선택적 폴더 — 노드 유형별 확장

기본 구조 외에 노드의 특성에 따라 폴더를 추가할 수 있습니다. 아래는 대표적인 예시이며 목록이 전부가 아닙니다. 목록에 없는 폴더가 필요한 경우에도 추가 가능하나, 반드시 아래 규칙을 따릅니다.

**폴더 추가 규칙:**
- **단일 책임**: 폴더 하나는 명확히 구분되는 하나의 관심사만 담는다
- **명칭 명확성**: 폴더명은 내부 파일의 성격을 즉시 드러내야 한다 (예: `parsers/`, `renderers/`, `validators/`)
- **Agent 소유 명시**: 폴더를 추가할 때 이 `index.md`의 해당 섹션에 역할·추가 조건·담당 Agent를 함께 기록한다
- **레이어 경계 준수**: UI 로직은 `app/`에, 순수 비즈니스·유틸 로직은 `lib/`에, 정적 에셋은 `public/`에 위치한다

### 공통 선택 폴더 (모든 노드 적용 가능)

#### `public/` — 정적 에셋

```
public/
├── fonts/          ← 자체 호스팅 폰트 파일 (.woff2)
├── icons/          ← 서비스 전용 아이콘·로고 SVG
└── samples/        ← UI에 표시되는 예시 이미지
```

| 추가 조건 | 폰트를 CDN 없이 자체 호스팅하거나, 노드 고유 아이콘이 필요할 때 |
|----------|--------------------------------------------------------------|
| 담당 Agent | AGENT C |

#### `types/` — TypeScript 타입 정의

```
types/
├── input.ts        ← NodeInput 인터페이스 (입력 계약)
├── output.ts       ← NodeOutput 인터페이스 (출력 계약)
└── api.ts          ← API 요청·응답 타입
```

| 추가 조건 | 입출력 타입이 복잡하거나 여러 파일에서 공유될 때 |
|----------|----------------------------------------------|
| 담당 Agent | AGENT A |

#### `_context/samples/` — 테스트용 샘플 데이터

```
_context/samples/
├── input-sample-01.jpg    ← 대표 입력 이미지
├── input-sample-01.json   ← 대표 입력 파라미터
└── expected-output-01.md  ← 기대 출력 (AGENT B 검증 기준)
```

| 추가 조건 | Stage B 동적 테스트 케이스 정의 시, 또는 Protocol 검증 반복 테스트가 필요할 때 |
|----------|---------------------------------------------------------------------------|
| 담당 Agent | AGENT B (검증 케이스 정의), AGENT A (수집·보관) |

---

### 노드 특성별 선택 폴더

#### `lib/imageProcessing/` — 이미지 전처리·후처리

```
lib/imageProcessing/
├── preprocess.ts   ← 입력 이미지 리사이즈·포맷 변환·압축
└── postprocess.ts  ← 출력 이미지 워터마크·크롭·합성
```

| 해당 노드 | N02 Plan, N03 Image, N04 Elevation, N05 Viewpoint |
|----------|--------------------------------------------------|
| 추가 조건 | 입력 이미지를 API 전송 전 가공하거나 출력 이미지를 후처리해야 할 때 |
| 담당 Agent | AGENT A |

#### `lib/parsers/` — 복합 입력 파싱

```
lib/parsers/
├── parameterParser.ts   ← 사용자 파라미터 구조화
└── reportParser.ts      ← AI 응답 텍스트를 구조화 데이터로 파싱
```

| 해당 노드 | N01 Planners (사업 타당성·리스크·파라미터 복합 입력) |
|----------|--------------------------------------------------|
| 추가 조건 | 입력 또는 출력의 파싱 로직이 route.ts에 인라인으로 담기 어려울 때 |
| 담당 Agent | AGENT A |

#### `templates/` — 출력 문서 템플릿

```
templates/
├── report-A3.html   ← A3 보고서 레이아웃 HTML
├── panel-A0.html    ← A0 패널 레이아웃 HTML
└── styles/
    └── print.css    ← 인쇄 전용 CSS (물리 출력 최적화)
```

| 해당 노드 | N07 Print (최종 증명서 자동 포맷팅) |
|----------|----------------------------------|
| 추가 조건 | AI 출력을 기반으로 고정 레이아웃 문서를 렌더링해야 할 때 |
| 담당 Agent | AGENT C (레이아웃), AGENT A (데이터 바인딩) |

#### `lib/renderers/` — 커스텀 시각화 렌더러

```
lib/renderers/
├── diagramRenderer.ts   ← SVG/Canvas 다이어그램 생성
└── overlayRenderer.ts   ← 이미지 오버레이 합성
```

| 해당 노드 | N06 Diagram (동선·조닝·채광 다이어그램), N07 Print |
|----------|--------------------------------------------------|
| 추가 조건 | AI 텍스트 출력을 기반으로 클라이언트 측 시각화를 직접 생성해야 할 때 |
| 담당 Agent | AGENT C |

---

## 구조 변경 시 주의사항

| 규칙 | 내용 |
|------|------|
| 폴더·파일 추가 | 이 파일의 해당 섹션에 역할·조건·담당 Agent를 기록한 뒤 추가한다. 기록 없는 추가는 인정되지 않는다 |
| `lib/` 파일 추가 | 단일 책임·명확한 명칭 원칙을 지키며 자유롭게 추가 가능. 단, `app/` 레이어 로직을 `lib/`에 두지 않는다 |
| Protocol 수정 | 반드시 버전 번호 올려 새 파일 생성. 기존 파일 수정·삭제 금지 |
| Agent 경계 | 자신의 소유 영역 밖 파일을 임의로 수정하지 않는다 |
| 신규 노드 생성 | `project_node/` 전체를 복사 후 이 파일과 `_context/` 내 `[node_name]` 플레이스홀더를 교체 |

---

`COPYRIGHTS 2026. CRE-TE CO.,LTD. ALL RIGHTS RESERVED.`
