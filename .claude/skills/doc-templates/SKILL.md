---
name: doc-templates
description: >
  Provides document templates for XML comments, script README, project README,
  handover docs, delivery docs, and architecture docs.
  Loads automatically when the doc-writer agent generates documentation.
  Do NOT use directly — loaded by the doc-writer agent as a reference.
allowed-tools: Read, Write
user-invocable: false
---

# 문서 유형별 템플릿

## A. 코드 주석

작성 규칙:
- public API·함수·컴포넌트에만 필수
- private는 복잡한 로직에만 인라인
- 한국어 + 영어 병기
- 당연한 주석 금지

**Unity C# — XML 주석:**
```csharp
/// <summary>
/// 플레이어에게 데미지를 입힙니다.
/// Applies damage after shield and armor calculation.
/// </summary>
/// <param name="amount">원본 데미지 수치 (방어력 계산 전)</param>
/// <param name="source">데미지 발생원 (null 허용)</param>
/// <returns>실제 적용된 데미지. 무효화되면 0 반환.</returns>
public float TakeDamage(float amount, GameObject source = null)
```

**TypeScript/JS — JSDoc:**
```typescript
/**
 * 사용자에게 데미지를 적용합니다.
 * Applies damage to a user after validation.
 * @param userId - 대상 사용자 ID
 * @param amount - 데미지 수치 (양수)
 * @returns 처리 결과 (성공/실패 + 메시지)
 */
export async function applyDamage(
  userId: string,
  amount: number
): Promise<ActionResult>
```

실행 순서: Read → public API 추출 → 프로젝트 타입 확인 → 주석 작성 → Write 저장

---


---

## 문서 템플릿
README·인수인계·납품·아키텍처 문서 템플릿은
`doc-templates/references/templates.md` 참조
