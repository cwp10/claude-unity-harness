# TypeScript 코드 스타일

## 버전 기준
- TypeScript 5.x (strict 모드 필수)
- Next.js 15.5.3 (App Router + Turbopack)
- React 19.1.0

## 기본 규칙
- `any` 타입 사용 금지 → `unknown` 또는 명시적 타입
- `as` 타입 단언 최소화 → 타입 가드 사용
- `!` non-null 단언 금지 → 옵셔널 체이닝 사용
- `var` 금지 → `const` / `let`
- `==` 금지 → `===`
- 빈 catch 블록 금지

## 네이밍 컨벤션
| 대상 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | PascalCase | `UserCard`, `DashboardLayout` |
| 함수·변수 | camelCase | `fetchUserData`, `isLoading` |
| 타입·인터페이스 | PascalCase | `UserProfile`, `ApiResponse` |
| 상수 | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |
| 파일 (컴포넌트) | kebab-case | `user-card.tsx` |
| 파일 (유틸·훅) | kebab-case | `use-auth.ts`, `api-client.ts` |
| Server Action | camelCase + Action suffix | `createUserAction` |

## 타입 정의
```typescript
// interface: 객체 형태, 확장 가능성 있을 때
interface UserProfile {
  id: string
  name: string
  email: string
}

// type: 유니온·교차·유틸리티 타입
type ApiStatus = 'idle' | 'loading' | 'success' | 'error'
type PartialUser = Partial<UserProfile>

// 함수 반환 타입 명시 필수
async function fetchUser(id: string): Promise<UserProfile> { ... }
```

## import 순서
```typescript
// 1. React
import { useState, useTransition } from 'react'

// 2. Next.js
import { useRouter } from 'next/navigation'
import Image from 'next/image'

// 3. 외부 라이브러리
import { useForm } from 'react-hook-form'
import { z } from 'zod'

// 4. 내부 절대 경로 (@/)
import { Button } from '@/components/ui/button'
import { useAuth } from '@/hooks/use-auth'

// 5. 상대 경로
import type { Props } from './types'
```

## Server Action 타입 패턴
```typescript
// actions/user-actions.ts
'use server'

export type ActionResult<T = void> =
  | { success: true; data: T }
  | { success: false; error: string }

export async function createUserAction(
  formData: FormData
): Promise<ActionResult<UserProfile>> {
  try {
    const validated = createUserSchema.parse({
      name: formData.get('name'),
      email: formData.get('email'),
    })
    const user = await db.user.create({ data: validated })
    revalidatePath('/users')
    return { success: true, data: user }
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: error.errors[0].message }
    }
    return { success: false, error: '서버 오류가 발생했습니다' }
  }
}
```

## 에러 처리
```typescript
// Result 패턴
type Result<T> = { success: true; data: T } | { success: false; error: string }

// try/catch (처리 못한 에러는 상위로)
try {
  const data = await fetchUser(id)
} catch (error) {
  if (error instanceof ApiError) { ... }
  throw error
}
```
