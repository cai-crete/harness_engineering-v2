# _context/ Index — Agent Document Reading Guide

> **이 파일은 `_context/` 폴더의 최상위 색인입니다.**
> 이 노드에서 작동하는 모든 에이전트는 작업 시작 전에 반드시 이 문서를 먼저 읽고, 아래에 명시된 우선순위와 적용 범위에 따라 나머지 문서를 로드하십시오.

---

## 폴더 구조

```
_context/
├── index.md                          ← 지금 이 파일 (에이전트 진입점)
├── business-context.md               ← 비즈니스 배경 및 서비스 목적
├── brand-guidelines.md               ← 브랜드 정체성 및 시각 원칙
├── design-style-guide-node.md        ← 노드 전용 UI/문서 디자인 시스템
├── protocol-node-vN                  ← 노드 통합 프로토콜 (마스터 문서)
├── guideline-Gemini_Architectural_Image_Analysis_Generation_EN_v1.1.md
│                                     ← Gemini 이미지 분석·생성 에이전트 전용 헌법
└── protocol/                         ← (예정) 노드별 세부 프로토콜 폴더
    └── [node-id]-protocol-vN.md      ← 각 노드의 세부 프로토콜 파일
```

---

## 에이전트 유형별 필수 독서 목록

### A. 모든 에이전트 공통 (MANDATORY — ALL AGENTS)

아래 문서는 에이전트 유형에 관계없이 **반드시** 읽어야 합니다. 순서를 지키십시오.

| 순서 | 파일 | 목적 |
|------|------|------|
| 1 | `index.md` (이 파일) | 전체 문서 구조 및 독서 우선순위 파악 |
| 2 | `business-context.md` | 서비스의 존재 이유, 사용자 페르소나, 핵심 가치 |
| 3 | `brand-guidelines.md` | 브랜드 톤, 시각 원칙, 금지 표현 |
| 4 | `protocol-node-vN` | 이 노드의 통합 프로토콜 (작동 규칙 전체) |

### B. UI/문서 생성 에이전트 추가 필독

| 순서 | 파일 | 목적 |
|------|------|------|
| 5 | `design-style-guide-node.md` | 노드 전용 레이아웃, 타이포그래피, 컴포넌트 규칙 |

### C. Gemini 이미지 분석·생성 에이전트 추가 필독

| 순서 | 파일 | 목적 |
|------|------|------|
| 5 | `guideline-Gemini_Architectural_Image_Analysis_Generation_EN_v1.1.md` | 건축 이미지 분석 및 생성의 절대 규칙 (Constitution) |
| 6 | `protocol/[해당 노드]-protocol-vN.md` | 해당 노드에 적용되는 세부 분석·생성 프로토콜 |

---

## 문서별 권한 및 성격 요약

### `business-context.md`
- **성격:** 불변 참조 문서
- **권한:** 모든 에이전트의 작업 목적 기준점
- **변경 주체:** 프로젝트 오너(PO)만 수정 가능

### `brand-guidelines.md`
- **성격:** 불변 참조 문서
- **권한:** 시각·언어 표현의 최종 기준
- **변경 주체:** 브랜드 담당자만 수정 가능

### `design-style-guide-node.md`
- **성격:** 버전 관리 문서
- **권한:** UI/문서 생성 에이전트의 디자인 결정 기준
- **변경 주체:** 디자인 리드 승인 후 업데이트

### `protocol-node-vN`
- **성격:** 마스터 프로토콜 (통합본)
- **권한:** 이 노드의 모든 세부 프로토콜을 포괄하는 상위 규칙
- **변경 주체:** `vN` 버전 번호를 올려 교체 (구버전 보존)
- **참고:** `protocol/` 하위 세부 프로토콜과 충돌 시 이 마스터 문서가 우선
- **헌법 적용 의무:** 이 문서를 **작성하거나 수정할 때 반드시 `guideline-Gemini_*` 헌법을 먼저 참조**하십시오. 마스터 프로토콜의 모든 조항은 헌법의 원칙 위에 수립되어야 하며, 헌법과 충돌하는 조항은 무효입니다.

### `guideline-Gemini_Architectural_Image_Analysis_Generation_EN_v1.1.md`
- **성격:** Constitutional 문서 (헌법적 효력)
- **권한:** Gemini 이미지 에이전트의 분석·생성 행동을 지배하는 최상위 규칙
- **적용 범위:** 건축 이미지 분석, 이미지 생성 프롬프트 작성, 렌더링 재구성 등 이미지 관련 모든 작업
- **변경 주체:** 버전 번호 갱신 방식으로만 교체 (현재 v1.1)

### `protocol/` (예정)
- **성격:** 노드별 세부 프로토콜 모음
- **권한:** 마스터 프로토콜(`protocol-node-vN`)의 위임 범위 내에서 세부 행동 규칙 정의
- **적용:** Gemini 이미지 에이전트가 특정 노드 작업 수행 시 해당 노드 프로토콜을 추가 로드

---

## 문서 간 권한 충돌 해소 원칙

우선순위가 높은 문서가 낮은 문서를 덮어씁니다.

```
Constitution (guideline-Gemini_*)
        ↓
마스터 프로토콜 (protocol-node-vN)
        ↓
브랜드·비즈니스 컨텍스트
        ↓
세부 프로토콜 (protocol/[node]-vN)
        ↓
디자인 시스템 (design-style-guide-node)
        ↓
에이전트 자체 추론
```

충돌 발생 시 에이전트는 상위 문서를 따르고, 하위 문서의 해당 규칙을 무시합니다. 충돌이 심각한 경우 사용자에게 보고하십시오.

---

## 새 프로토콜 파일 추가 시 절차

`protocol/` 폴더에 새 노드 세부 프로토콜을 추가할 때:

1. 이 `index.md`의 폴더 구조 다이어그램에 항목 추가
2. 해당 에이전트 유형의 필수 독서 목록(섹션 B 또는 C)에 행 추가
3. `protocol-node-vN` 마스터 프로토콜에 위임 조항 명시

---

## 기존 프로젝트에 이 index.md를 처음 도입할 때

이 `index.md`를 기존 프로젝트에 처음 도입하는 경우, 아래 절차를 따르십시오.

1. `_context/` 폴더 안에 **마스터 프로토콜 파일이 이미 존재하는지 확인**하십시오.
2. 파일이 존재한다면, 이 `index.md` 문서 전체에서 `protocol-node-vN`이라고 표기된 부분을 **해당 파일의 실제 이름으로 교체**하십시오.
   - 폴더 구조 다이어그램, 필수 독서 목록 표, 문서별 권한 요약, 충돌 해소 원칙 다이어그램 모두 해당됩니다.
3. 파일이 존재하지 않는다면, `protocol-node-vN` 표기를 유지하고 추후 실제 파일 작성 시 교체하십시오.

---

*Last updated: 2026-05-08 | Maintained by: CAI Project*
