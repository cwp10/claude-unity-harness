# 웹 도메인 패턴

## 기술 스택 기준
- Framework: Next.js 15.5.3 (App Router + Turbopack)
- Runtime: React 19.1.0 + TypeScript 5
- Styling: TailwindCSS v4 + shadcn/ui (new-york style)
- Forms: React Hook Form + Zod + Server Actions
- UI: Radix UI + Lucide Icons
- Dev: ESLint + Prettier + Husky + lint-staged

## 프로젝트 폴더 구조

```
src/
├── app/                      ← Next.js App Router
│   ├── (auth)/               ← 인증 라우트 그룹
│   │   ├── login/page.tsx
│   │   └── layout.tsx
│   ├── (dashboard)/          ← 인증 필요 라우트 그룹
│   │   ├── dashboard/page.tsx
│   │   └── layout.tsx
│   ├── api/                  ← API Route Handlers (최소화)
│   ├── globals.css
│   └── layout.tsx
├── actions/                  ← Server Actions
│   ├── user-actions.ts
│   └── auth-actions.ts
├── components/
│   ├── ui/                   ← shadcn/ui 컴포넌트 (수정 금지)
│   ├── features/             ← 기능별 컴포넌트
│   │   ├── auth/
│   │   └── dashboard/
│   └── layouts/              ← 레이아웃 컴포넌트
├── hooks/                    ← 커스텀 훅
├── lib/
│   ├── db.ts                 ← DB 클라이언트
│   ├── auth.ts               ← 인증 설정
│   └── utils.ts              ← cn() 등 유틸
├── services/                 ← 외부 API 호출 레이어
├── types/                    ← 타입 정의
├── constants/                ← 상수
└── middleware.ts             ← 라우트 보호
```

## Server Component vs Client Component

```
데이터 페칭·DB 접근 → Server Component (기본)
onClick·useState·브라우저 API → Client Component ('use client')
둘 다 필요 → Server에서 데이터 페칭 후 Client에 props 전달
```

```typescript
// Server Component (기본, async 가능)
export default async function UserListPage() {
  const users = await db.user.findMany()
  return <UserList users={users} />
}

// Client Component (인터랙션만)
'use client'
export function UserList({ users }: { users: User[] }) {
  const [search, setSearch] = useState('')
  ...
}
```

## Server Actions (폼 처리 핵심)

패턴 코드는 web-patterns 스킬 참조.

핵심 원칙:
- Server Action 우선 / API Route Handler 최소화
- `useActionState` (React 19) 로 상태 관리
- `revalidatePath` 로 캐시 갱신
- `safeParse` 로 유효성 검사 후 에러 반환

## Zod 스키마 + React Hook Form (클라이언트 폼)

패턴 코드는 web-patterns 스킬 참조.

- 단순 폼: Server Action 직접 사용 권장
- 복잡한 클라이언트 인터랙션: React Hook Form + zodResolver

## shadcn/ui 사용 규칙

```typescript
// ✅ shadcn 컴포넌트 import
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card, CardContent, CardHeader } from '@/components/ui/card'

// ✅ cn() 유틸로 클래스 병합
import { cn } from '@/lib/utils'
<div className={cn('base-class', condition && 'conditional-class', className)} />

// ❌ components/ui/ 파일 직접 수정 금지
// → 커스텀 필요 시 components/features/ 에 래퍼 컴포넌트 생성
```

## TailwindCSS v4 규칙

```typescript
// ✅ v4 방식: CSS 변수 직접 사용 가능
// globals.css에서 @theme 정의
// tailwind.config.js 불필요 (v4는 CSS-first 설정)

// ✅ 반응형: mobile-first
<div className="flex flex-col md:flex-row lg:grid lg:grid-cols-3" />

// ✅ 다크모드: class 전략
<div className="bg-white dark:bg-gray-900" />

// ❌ 인라인 style 금지 (Tailwind로 대체)
```

## middleware.ts (라우트 보호)

```typescript
export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')
  const isDashboard = request.nextUrl.pathname.startsWith('/dashboard')
  const isAuth = request.nextUrl.pathname.startsWith('/login')

  if (isDashboard && !token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  if (isAuth && token) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }
}

export const config = {
  matcher: ['/dashboard/:path*', '/login', '/register'],
}
```

## Husky + lint-staged 설정

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{css,md,json}": ["prettier --write"]
  }
}
```

## 성능 기준
- LCP: 2.5초 이하 (Turbopack으로 빌드 속도 향상)
- Image: `next/image` 필수
- Font: `next/font` 필수 (레이아웃 시프트 방지)
- 번들: 초기 JS 200KB 이하 (gzip)
- 무거운 컴포넌트: `dynamic()` 지연 로딩
- 리스트 key: 안정적인 id 사용 (index 금지)

## 금지 사항
- `useEffect`로 데이터 페칭 → Server Component 또는 Server Action 사용
- `components/ui/` 직접 수정
- API Route Handler 남용 → Server Action 우선
- `any` 타입
- `pages/` 디렉토리 (App Router만 사용)
