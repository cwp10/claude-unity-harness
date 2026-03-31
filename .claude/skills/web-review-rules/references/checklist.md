# Next.js / TypeScript 코드 리뷰 상세 체크리스트

## 아키텍처 (Architecture)
| 항목 | 심각도 |
|------|--------|
| `'use client'` 파일에서 DB 직접 접근·서버 전용 패키지 import | 🔴 |
| Server Component에서 `useState`·`useEffect` 사용 | 🔴 |
| `useEffect` + `fetch` 조합 → Server Action 대체 필요 | 🟡 |
| API Route 남용 (Server Action으로 대체 가능) | 🟡 |
| 관심사 분리 위반 (한 컴포넌트가 너무 많은 역할) | 🟡 |
| 컴포넌트 30줄 초과 → 분리 제안 | 🟢 |

## 타입 안전성 (Type Safety)
| 항목 | 심각도 |
|------|--------|
| `any` 타입 사용 | 🟡 |
| `as` 타입 단언 남용 | 🟡 |
| 함수 반환 타입 누락 | 🟡 |
| `unknown` 대신 `any` 사용 | 🟡 |
| non-null assertion (`!`) 남용 | 🟡 |
| 누락된 props interface 정의 | 🟢 |

## 성능 (Performance)
| 항목 | 심각도 |
|------|--------|
| `<img>` 태그 사용 → `next/image` 미사용 | 🟡 |
| `next/font` 미사용 (외부 폰트 직접 import) | 🟡 |
| 의존성 배열 없는 `useMemo`·`useCallback` | 🟡 |
| 불필요한 `'use client'` 선언 (서버 렌더 가능한 컴포넌트) | 🟡 |
| 대용량 라이브러리 전체 import (트리쉐이킹 미적용) | 🟡 |

## 보안 (Security)
| 항목 | 심각도 |
|------|--------|
| 인증 없는 API Route | 🔴 |
| `NEXT_PUBLIC_` 아닌 환경변수 클라이언트에서 참조 | 🔴 |
| `dangerouslySetInnerHTML` 사용 | 🔴 |
| SQL 인젝션 위험 (raw query 사용) | 🔴 |
| 민감 정보 클라이언트 번들 포함 | 🔴 |
| CSRF 방어 누락 (Server Action 미사용 form) | 🟡 |

## Next.js 규칙
| 항목 | 심각도 |
|------|--------|
| `error.tsx` 누락 (에러 바운더리 없음) | 🟡 |
| `loading.tsx` 누락 | 🟡 |
| `layout.tsx`에 `metadata` 누락 | 🟡 |
| `next.config`에 `images` 설정 누락 | 🟡 |
| `getServerSideProps` 사용 → App Router 패턴으로 대체 필요 | 🟡 |
