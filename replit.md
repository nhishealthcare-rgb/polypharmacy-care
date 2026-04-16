# Overview

This is a **kiosk-style web application** for Korea's National Health Insurance Service (NHIS) "다제약물 관리사업" (Multi-Drug Management Program). It provides an informational touchscreen interface where patients and pharmacists can learn about the program, view details, and submit applications (phone number-based). The UI is designed for large-touch kiosk interaction with big buttons, Korean language throughout, and accessibility features like adjustable font sizes.

# User Preferences

Preferred communication style: Simple, everyday language.

# System Architecture

## Frontend
- **Framework**: React 18 with TypeScript
- **Routing**: Wouter (lightweight client-side router)
- **State Management**: TanStack React Query for server state
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives
- **Styling**: Tailwind CSS with CSS variables for theming, NanumSquare Korean font
- **Build Tool**: Vite with React plugin
- **Path Aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`

### Key Pages
- `/` — Home menu with navigation cards, visitor counter, sharing buttons
- `/business` — Program overview with external links (대한약사회, YouTube)
- `/details` — Detailed program information with infographic and video
- `/pharmacist` — Pharmacist application info, documents list, external links
- `/guide` — Consultation method guide with fee info link

### Content Configuration
- **`client/src/config/content.ts`** — Centralized content file containing all editable text, links, and lists. Non-technical users can modify this single file to update any displayed content (titles, descriptions, external URLs, document lists, etc.) without touching component code. After editing, redeploy to apply changes.

### Design Decisions
- Kiosk-first UI: Large touch targets, no hover-only interactions, font size controls (5 levels stored in localStorage)
- Korean-only interface targeting elderly users with chronic conditions
- Glass-morphism card design with teal/accent color scheme
- Share functionality via SMS deep links and clipboard copy (Web Share API when available)

## Backend
- **Runtime**: Node.js with Express 5
- **Language**: TypeScript, executed via tsx
- **API Pattern**: RESTful JSON API under `/api/` prefix
- **Build**: esbuild for server bundle, Vite for client bundle (via `script/build.ts`)
- **Dev Mode**: Vite dev server middleware with HMR served through Express
- **Production**: Static files served from `dist/public/`, SPA fallback to `index.html`

### API Endpoints
- `POST /api/applications` — Submit an application (phone number). Validated with Zod via drizzle-zod schemas.
- Pages CRUD endpoints defined in `shared/routes.ts` but only partially wired in `server/routes.ts`
- `POST /api/share` — Share endpoint (referenced in client but not fully implemented in routes)

## Shared Layer (`shared/`)
- **Schema**: Drizzle ORM table definitions + Zod validation schemas
- **Routes**: API contract definitions with Zod response schemas (typed API pattern)
- Two tables: `applications` (id, phoneNumber, createdAt) and `pages` (id, slug, title, body)

## Database
- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Driver**: node-postgres (`pg` Pool)
- **Schema Push**: `npm run db:push` via drizzle-kit
- **Connection**: `DATABASE_URL` environment variable (required)
- **Storage Pattern**: `IStorage` interface in `server/storage.ts` with `DatabaseStorage` implementation

## Key Scripts
- `npm run dev` — Development server with Vite HMR
- `npm run build` — Production build (Vite client + esbuild server)
- `npm run start` — Run production build
- `npm run db:push` — Push schema changes to database

# External Dependencies

## Required Services
- **PostgreSQL Database** — Required via `DATABASE_URL` environment variable. Used for storing applications and CMS-style pages.

## Key NPM Packages
- `express` v5 — HTTP server
- `drizzle-orm` + `drizzle-kit` — Database ORM and migrations
- `drizzle-zod` — Auto-generate Zod schemas from Drizzle tables
- `@tanstack/react-query` — Client-side data fetching/caching
- `wouter` — Lightweight React router
- `zod` — Runtime validation on both client and server
- shadcn/ui ecosystem (`@radix-ui/*`, `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`)
- `connect-pg-simple` — PostgreSQL session store (available but sessions not currently active)
- `react-hook-form` + `@hookform/resolvers` — Form handling

## External Links (in-app)
- NHIS logo and images from `nhis.or.kr`
- Korean Pharmaceutical Association portal (`pharmcare.kpanet.or.kr`, `kpanet.or.kr`)
- YouTube video links for program explanation
- NanumSquare font from CDN (`cdn.jsdelivr.net`)
- Google Fonts (Geist Mono, DM Sans, etc.)

## Replit-Specific
- `@replit/vite-plugin-runtime-error-modal` — Error overlay in development
- `@replit/vite-plugin-cartographer` and `@replit/vite-plugin-dev-banner` — Dev tools (conditionally loaded)