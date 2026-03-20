# CLAUDE.md — Sport-Id-Passport

This file provides context for AI assistants working on the Sport-Id-Passport codebase.

---

## Project Overview

**Sport-Id-Passport** (`sportid-passport`) is a Next.js application for managing athletic credentials with QR code support. It is currently in the initial setup/configuration phase — no application source code has been written yet.

**Domain purpose:** Digital identity/passport system for athletes, likely including QR-code-based credential display, data visualization (activity/stats charts), and animated UI components.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16.1.6 (App Router) |
| Language | TypeScript 5 (strict mode) |
| UI | React 19 |
| Styling | Tailwind CSS 4 (via PostCSS) |
| API | tRPC v11 + React Query v5 |
| Validation | Zod v4 |
| QR Codes | qrcode |
| Charts | Recharts 3 (transpiled by Next.js) |
| Animation | Framer Motion 12 |
| Icons | Lucide React |
| Notifications | React Hot Toast |

---

## Repository Structure

```
Sport-Id-Passport/
├── CLAUDE.md                  # This file
├── README.md                  # Basic Next.js getting-started readme
├── next.config.ts             # Next.js config with security headers
├── tsconfig.json              # TypeScript config (@/* → ./src/*)
├── eslint.config.mjs          # ESLint flat config (ESLint 9)
├── postcss.config.mjs         # PostCSS with Tailwind CSS 4
├── package.json               # Dependencies and scripts
├── package-lock.json          # Locked dependency tree
├── next-env.d.ts              # Auto-generated Next.js types (do not edit)
└── src/                       # Application source root (not yet created)
    └── app/                   # Next.js App Router root (not yet created)
```

> **Note:** The `src/` and `src/app/` directories do not yet exist. All new source code should go under `src/` following the path alias `@/*` → `./src/*`.

---

## Development Commands

```bash
npm run dev      # Start local dev server at http://localhost:3000
npm run build    # Compile production build
npm start        # Serve production build
npm run lint     # Run ESLint across the project
```

There is no test runner configured yet. When adding tests, prefer **Vitest** (compatible with Next.js and Vite-style tooling) or **Jest** with `ts-jest`.

---

## TypeScript Configuration

- **Strict mode enabled** — all strict TypeScript checks apply.
- **Path alias:** `@/*` resolves to `./src/*`. Use this for all internal imports.
- **Target:** ES2017 (broad browser compatibility).
- **JSX:** `react-jsx` (no need to import React in every file).
- **Module resolution:** `bundler` (Next.js bundler-aware resolution).

Example import:
```ts
import { SportCard } from '@/components/SportCard';
```

---

## ESLint Configuration

Uses ESLint 9 flat config (`eslint.config.mjs`):
- Extends `eslint-config-next/core-web-vitals` and `eslint-config-next/typescript`.
- Ignores: `.next/`, `out/`, `build/`, `next-env.d.ts`.

Run linting before committing:
```bash
npm run lint
```

---

## Security Headers

`next.config.ts` applies comprehensive HTTP security headers to all routes (`/(.*)`):

| Header | Value / Purpose |
|---|---|
| `Strict-Transport-Security` | 2-year HTTPS enforcement with subdomains + preload |
| `X-Frame-Options` | `SAMEORIGIN` — prevents clickjacking |
| `X-Content-Type-Options` | `nosniff` — no MIME-type sniffing |
| `X-XSS-Protection` | `1; mode=block` — legacy XSS filter |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Camera off, microphone off, geolocation self-only, payment/USB/FLoC disabled |
| `Content-Security-Policy` | `self`-only; `data:` and `blob:` allowed for QR code canvas output |
| `X-DNS-Prefetch-Control` | `on` |

**Important CSP notes:**
- `unsafe-eval` and `unsafe-inline` are allowed for `script-src` (required by Next.js HMR and RSC).
- `img-src` allows `data:` and `blob:` specifically to support QR code canvas-to-dataURL output.
- `connect-src` is restricted to `self` — no external API calls allowed without updating the CSP.
- `frame-ancestors 'none'` — the app cannot be embedded in any iframe.

When adding external resources (fonts, CDN images, analytics, external APIs), update the relevant CSP directives in `next.config.ts`.

**Other config notes:**
- `poweredByHeader: false` — hides `X-Powered-By: Next.js` response header.
- `recharts` is listed in `transpilePackages` because it ships as ES modules.
- Image optimization: no remote patterns; SVG is allowed; data URIs accepted.

---

## API Architecture (tRPC)

The project uses **tRPC v11** for end-to-end type-safe API calls:

- `@trpc/server` — define routers and procedures on the server.
- `@trpc/client` — vanilla client for server-to-server or standalone calls.
- `@trpc/react-query` — React hooks backed by React Query v5.

**Convention:** Define tRPC routers under `src/server/trpc/` and expose them via a Next.js API route at `src/app/api/trpc/[trpc]/route.ts`.

Input validation must use **Zod** schemas on all tRPC procedures.

---

## Conventions & Coding Standards

### File & Folder Naming
- React components: `PascalCase.tsx` (e.g., `SportCard.tsx`).
- Utilities/helpers: `camelCase.ts` (e.g., `formatDate.ts`).
- tRPC routers: `camelCase.router.ts` (e.g., `athlete.router.ts`).
- Directories: `kebab-case/` (e.g., `src/components/athlete-card/`).

### Component Style
- Prefer **React Server Components** (RSC) by default in the App Router.
- Add `'use client'` only when the component needs browser APIs, state, or event handlers.
- Use Tailwind CSS utility classes directly in JSX — no CSS modules unless necessary.
- Keep components focused; extract sub-components into the same directory.

### Imports
- Use the `@/` path alias for all internal imports.
- Group imports: external packages first, then internal `@/` imports.
- Do not import React explicitly (JSX transform handles it).

### Data Validation
- Validate all external inputs (API bodies, URL params, environment variables) with **Zod**.
- Export inferred Zod types alongside schemas: `export type Athlete = z.infer<typeof athleteSchema>`.

### QR Code
- QR codes are display-only (no camera scanning in this POC).
- Generate via the `qrcode` package. Output as `data:` URI to stay within CSP.

### Charts
- Use **Recharts** for all data visualizations.
- It is already configured in `transpilePackages` — no additional setup needed.

### Animations
- Use **Framer Motion** for transitions and micro-interactions.
- Keep animations subtle; respect `prefers-reduced-motion`.

---

## Environment Variables

No environment variables are configured yet. When adding them:

- **Client-side variables** must be prefixed with `NEXT_PUBLIC_`.
- **Server-only variables** (secrets, DB URLs) must not use `NEXT_PUBLIC_` prefix.
- Store values in `.env.local` (gitignored). Document required variables here.

Example:
```
# .env.local (gitignored — never commit)
DATABASE_URL=postgresql://...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

---

## Git Workflow

- Default development branch: `master`.
- Feature branches follow the pattern: `claude/<description>-<session-id>`.
- Commit messages should be descriptive and written in the imperative mood.
- The repository remote is: `http://local_proxy@127.0.0.1:34387/git/Saadfd466/Sport-Id-Passport`.

---

## What Does Not Exist Yet

The following need to be created as development begins:

- `src/app/` — App Router root (`layout.tsx`, `page.tsx`, `globals.css`)
- `src/components/` — Reusable UI components
- `src/server/` — tRPC routers and server-side logic
- `src/lib/` — Shared utilities and type definitions
- `.env.local` — Local environment configuration (gitignored)
- Test suite — No testing framework configured yet
- CI/CD pipeline — No GitHub Actions or other CI configured
