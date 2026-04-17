# FRONTEND.md — 프론트엔드 개발 가이드

---

## 기술 스택

| 레이어 | 표준 |
|--------|------|
| Framework | Next.js (App Router) |
| Styling | Tailwind CSS |
| Language | TypeScript |
| Package Manager | npm |

## 프로젝트 구조 (노드 앱 기준)

```
project_[node_name]/
├── _context/              ← 노드 하네스 컨텍스트
│   ├── brand-guidelines.md
│   ├── business-context.md
│   └── design-style-guide-node.md
├── _context/              ← Principle Protocol + Knowledge Docs
│   ├── protocol-[node_name]-v[N].txt
│   └── [knowledge-doc].txt
├── app/
│   ├── api/
│   │   └── generate/
│   │       └── route.ts   ← Protocol 주입 + AI API 호출
│   ├── page.tsx           ← 메인 UI
│   └── layout.tsx
├── components/
├── lib/
│   └── buildSystemPrompt.ts  ← Protocol 조합 유틸리티
└── public/
```

## Protocol 주입 유틸리티

```typescript
// lib/buildSystemPrompt.ts
export function buildSystemPrompt(
  principleProtocol: string,
  knowledgeDocs: string[] = []
): string {
  return [principleProtocol, ...knowledgeDocs].join("\n\n---\n\n");
}
```

## API Route 표준 패턴

```typescript
// app/api/generate/route.ts
import { GoogleGenAI } from "@google/genai";
import { buildSystemPrompt } from "@/lib/buildSystemPrompt";
import { principleProtocol } from "@/lib/protocol";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

export async function POST(request: Request) {
  const { userInput } = await request.json();

  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    config: { systemInstruction: buildSystemPrompt(principleProtocol) },
    contents: [{ role: "user", parts: [{ text: userInput }] }],
  });

  return Response.json({ result: response.text });
}
```

## 코딩 원칙

- Protocol 내용을 코드 레이어에서 하드코딩하지 않음 — Protocol 파일에서만 관리
- `systemInstruction` 파라미터 null 방어 코드 필수
- 에러 응답은 사용자에게 명확한 메시지로 표시
- 워터마크는 클라이언트 사이드에서 자동 적용

---

`COPYRIGHTS 2026. CRE-TE CO.,LTD. ALL RIGHTS RESERVED.`
