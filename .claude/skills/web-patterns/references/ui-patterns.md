# 웹 UI 패턴 레퍼런스

## shadcn/ui 컴포넌트 조합 패턴

```typescript
// components/features/user/user-form.tsx
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import {
  Card, CardContent, CardDescription,
  CardFooter, CardHeader, CardTitle
} from '@/components/ui/card'
import { cn } from '@/lib/utils'

interface UserFormProps {
  className?: string
  onSuccess?: () => void
}

export function UserForm({ className, onSuccess }: UserFormProps) {
  const [state, action, isPending] = useActionState(createUserAction, null)

  return (
    <Card className={cn('w-full max-w-md', className)}>
      <CardHeader>
        <CardTitle>사용자 추가</CardTitle>
        <CardDescription>새 사용자 정보를 입력해주세요</CardDescription>
      </CardHeader>
      <form action={action}>
        <CardContent className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="name">이름</Label>
            <Input id="name" name="name" placeholder="홍길동" required />
          </div>
          <div className="space-y-2">
            <Label htmlFor="email">이메일</Label>
            <Input id="email" name="email" type="email" required />
          </div>
          {state?.success === false && (
            <p className="text-sm text-destructive">{state.error}</p>
          )}
        </CardContent>
        <CardFooter>
          <Button type="submit" disabled={isPending} className="w-full">
            {isPending ? '처리 중...' : '추가하기'}
          </Button>
        </CardFooter>
      </form>
    </Card>
  )
}
```

---

## TailwindCSS v4 패턴

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  /* 커스텀 색상 */
  --color-brand: oklch(55% 0.2 250);
  --color-brand-foreground: oklch(98% 0 0);

  /* 커스텀 폰트 */
  --font-sans: 'Pretendard', ui-sans-serif;
}
```

```typescript
// cn() 유틸 활용
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// 사용 예시
<div className={cn(
  'rounded-lg border bg-card p-6',
  isActive && 'border-brand',
  className
)} />
```

---

## 데이터 테이블 (TanStack Table)

```typescript
// components/features/users/user-table.tsx
'use client'
import {
  useReactTable, getCoreRowModel, getSortedRowModel,
  getFilteredRowModel, flexRender,
} from '@tanstack/react-table'

const columns: ColumnDef<User>[] = [
  { accessorKey: 'name', header: '이름' },
  { accessorKey: 'email', header: '이메일' },
  {
    id: 'actions',
    cell: ({ row }) => (
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="ghost" size="icon">
            <MoreHorizontal className="h-4 w-4" />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent>
          <DropdownMenuItem>수정</DropdownMenuItem>
          <DropdownMenuItem className="text-destructive">삭제</DropdownMenuItem>
        </DropdownMenuContent>
      </DropdownMenu>
    ),
  },
]
```

---

