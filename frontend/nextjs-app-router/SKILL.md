# Next.js App Router

You are an expert Next.js App Router developer. Follow these conventions for all Next.js projects.

## Core Rules

### 1. File Convention
- Layouts: `layout.tsx` — shared UI wrapper, preserves state across navigations
- Pages: `page.tsx` — route content, default Server Component
- Loading: `loading.tsx` — Suspense fallback for route segment
- Error: `error.tsx` — error boundary with reset button
- Not Found: `not-found.tsx` — custom 404
- Route handlers: `route.ts` — API endpoints (never in same folder as `page.tsx`)

### 2. Server vs Client Components
- **Default: Server Component.** Only add `'use client'` when necessary.
- Client component triggers: `useState`, `useEffect`, `onClick`, browser APIs, context consumers.
- Push client logic as deep as possible — keep parent as Server Component.
- Pass data as props from Server to Client, not via fetch in Client.

### 3. Data Fetching
- Server Components: `async` component → `await fetch()` directly. No `useEffect` + `useState`.
- Deduplication: Next.js automatically deduplicates `fetch` with same URL. Don't cache manually.
- Revalidation: `fetch(url, { next: { revalidate: 3600 } })` for ISR.
- NEVER fetch in Client Components unless strictly necessary (e.g., polling, real-time).
- Route handlers for mutations: `route.ts` with `export async function POST()`.

### 4. Caching Strategy
- Static (default): `force-static`. Generated at build time. Use for blog, docs, marketing.
- ISR: `revalidate` in fetch or `export const revalidate = 3600` in page.
- Dynamic: `force-dynamic`. Generated per request. Use for dashboards, user data.
- `export const dynamic = 'force-static' | 'force-dynamic' | 'auto'`.
- `export const revalidate = false | 0 | number`.

### 5. Metadata & SEO
```tsx
import type { Metadata } from 'next';
export const metadata: Metadata = {
  title: 'Page Title',
  description: 'Page description for SEO',
  openGraph: { images: ['/og.png'] },
};
// Dynamic: export async function generateMetadata({ params }) { ... }
```

### 6. Navigation
- Use `<Link>` for client-side navigation, not `<a>`.
- `useRouter` only for imperative navigation (form submit, callback).
- `redirect()` for server-side redirects (auth, not-found).
- Parallel routes: `@modal` for modals, `@sidebar` for persistent sidebar.

### 7. Middleware
```ts
// middleware.ts at root
export function middleware(request: NextRequest) {
  // Auth check, redirect, header modification
}
export const config = { matcher: ['/dashboard/:path*'] };
```

### 8. Folder Structure
```
src/app/
├── (public)/          ← route group (no URL segment)
│   ├── layout.tsx     ← public layout
│   └── page.tsx       ← /
├── (dashboard)/       ← protected route group
│   ├── layout.tsx     ← dashboard layout (sidebar + auth check)
│   └── dashboard/
└── api/               ← route handlers
```

## Anti-Patterns
- ❌ `'use client'` on entire page — keep Server by default
- ❌ `useEffect` for data fetching in Client Component
- ❌ `useState` + `useEffect` to mirror props — use props directly
- ❌ Importing server-only code (db, fs) in Client Component
- ❌ Using `<a>` for internal links — use `<Link>`
