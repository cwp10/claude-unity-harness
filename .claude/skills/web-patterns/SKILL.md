---
name: web-patterns
description: >
  Provides Next.js 15, React 19, TypeScript 5 pattern code references including
  Server Actions, shadcn/ui, TailwindCSS v4, Zod, and middleware patterns.
  Loads automatically when writing, designing, or reviewing web project code.
  Do NOT use for Unity, C#, or non-web projects.
allowed-tools: Read, Glob, Grep
user-invocable: false
---

# 웹 패턴 코드 레퍼런스
스택: Next.js 15.5.3 / React 19 / TypeScript 5 / TailwindCSS v4 / shadcn/ui


## 패턴 선택 가이드
| 상황 | 패턴 |
|------|------|
| 폼 처리·데이터 변경 | Server Action + Zod |
| 반복 비즈니스 로직 | 커스텀 훅 |
| 에러 표시 | error.tsx + useActionState |
| UI 컴포넌트 | shadcn/ui + TailwindCSS v4 |
| 데이터 페칭 | Server Component (fetch) |
| 클라이언트 상태 | useState·useReducer |

## 패턴 코드 참조
Server Action, Zod, 커스텀 훅, 에러 처리, 성능 체크리스트:
→ `web-patterns/references/patterns.md`

UI 컴포넌트 패턴 (shadcn/ui, TailwindCSS, 레이아웃):
→ `web-patterns/references/ui-patterns.md`
