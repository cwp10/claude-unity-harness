# 웹 패턴 코드 레퍼런스

## Server Action 패턴 (폼 처리 핵심)

```typescript
// actions/user-actions.ts
'use server'

export type ActionResult<T = void> =
  | { success: true; data?: T; message?: string }
  | { success: false; error: string }

// useActionState 와 함께 사용
export async function createUserAction(
  prevState: ActionResult | null,
  formData: FormData
): Promise<ActionResult<User>> {
  const validated = createUserSchema.safeParse(Object.fromEntries(formData))
  if (!validated.success) {
    return { success: false, error: validated.error.errors[0].message }
  }
  try {
    const user = await db.user.create({ data: validated.data })
    revalidatePath('/users')
    return { success: true, data: user, message: '사용자가 생성됐습니다' }
  } catch {
    return { success: false, error: '서버 오류가 발생했습니다' }
  }
}

// 컴포넌트에서 사용
'use client'
import { useActionState } from 'react'

export function CreateUserForm() {
  const [state, action, isPending] = useActionState(createUserAction, null)
  return (
    <form action={action} className="space-y-4">
      <Input name="name" placeholder="이름" required />
      <Input name="email" type="email" placeholder="이메일" required />
      {state?.success === false && (
        <p className="text-sm text-destructive">{state.error}</p>
      )}
      {state?.success === true && (
        <p className="text-sm text-green-600">{state.message}</p>
      )}
      <Button type="submit" disabled={isPending}>
        {isPending ? '처리 중...' : '생성'}
      </Button>
    </form>
  )
}
```

---

## Zod 스키마 패턴

```typescript
// lib/validations/user.ts
import { z } from 'zod'

export const createUserSchema = z.object({
  name: z.string().min(2, '2자 이상 입력해주세요').max(50, '50자 이하로 입력해주세요'),
  email: z.string().email('올바른 이메일 형식이 아닙니다'),
  role: z.enum(['admin', 'user', 'viewer']).default('user'),
})

export const updateUserSchema = createUserSchema.partial().extend({
  id: z.string().uuid(),
})

export type CreateUserDto = z.infer<typeof createUserSchema>
export type UpdateUserDto = z.infer<typeof updateUserSchema>
```

---

## 커스텀 훅 패턴

```typescript
// hooks/use-optimistic-list.ts
// React 19 useOptimistic 활용
'use client'
import { useOptimistic, useTransition } from 'react'

export function useOptimisticList<T extends { id: string }>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition()
  const [items, addOptimistic] = useOptimistic(
    initialItems,
    (state, newItem: T) => [...state, newItem]
  )

  const addItem = (newItem: T, action: () => Promise<void>) => {
    startTransition(async () => {
      addOptimistic(newItem)
      await action()
    })
  }

  return { items, addItem, isPending }
}
```

---

## 에러 처리 패턴 (Next.js App Router)

```typescript
// app/error.tsx - 런타임 에러
'use client'
export default function Error({ error, reset }: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen gap-4">
      <h2 className="text-xl font-semibold">오류가 발생했습니다</h2>
      <p className="text-muted-foreground">{error.message}</p>
      <Button onClick={reset}>다시 시도</Button>
    </div>
  )
}

// app/not-found.tsx - 404
export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen gap-4">
      <h2 className="text-4xl font-bold">404</h2>
      <p className="text-muted-foreground">페이지를 찾을 수 없습니다</p>
      <Button asChild>
        <Link href="/">홈으로</Link>
      </Button>
    </div>
  )
}
```

---

## 성능 체크리스트
- [ ] Image: `next/image` 사용
- [ ] Font: `next/font` 사용
- [ ] 대용량 컴포넌트: `dynamic()` 지연 로딩
- [ ] 리스트 key: 안정적인 id (index 금지)
- [ ] 서버 상태: Server Component 우선
- [ ] 클라이언트 상태: `useOptimistic` 활용 (React 19)
- [ ] 폼 처리: Server Action 우선, RHF는 복잡한 경우만


---

## UI 패턴 참조
shadcn/ui 조합·TailwindCSS·데이터 테이블 패턴은
`web-patterns/references/ui-patterns.md` 참조
