# Horse & Yard Management MVP - Tech Stack & Architecture

## Tech Stack

### Frontend
- **Next.js 14+** (App Router)
  - Full-stack framework (API routes built-in)
  - Server-side rendering for performance
  - Built-in optimization for images, fonts, scripts
  
- **React 18+**
  - Component-based UI
  - Hooks for state management
  
- **TailwindCSS**
  - Utility-first, responsive design
  - Mobile-first approach
  
- **TypeScript**
  - Type safety across codebase
  - Better developer experience
  
- **Zustand**
  - Lightweight state management
  - Offline state sync
  
- **React Query (TanStack Query)**
  - Data fetching & caching
  - Automatic stale data management
  
- **Service Workers + IndexedDB**
  - Offline task list caching
  - Offline note creation & sync queuing

---

## Backend

### API Server
- **Next.js API Routes** (fully sufficient for MVP)
  - Serverless functions
  - Built-in middleware support
  - Deploy anywhere Node.js runs
  
### Alternative: Separate Express Server
- If API becomes large, migrate to Express + Node.js
- Same database layer (Prisma)

---

## Database

### Primary: PostgreSQL
- Relational data (horses, tasks, users, notes)
- Strong consistency
- Battle-tested for production

### ORM: Prisma
- Type-safe database queries
- Auto-generated migrations
- Built-in relationships
- Excellent DX

---

## Authentication & Authorization

### Auth Layer: **NextAuth.js v5**
- Session management
- OAuth support (Google, GitHub, etc.)
- Built for Next.js
- JWT-based tokens for offline mode

### RBAC: Custom Middleware
- Role guards per API route
- Component-level access control
- Easy to audit

---

## Real-Time Features

### Option A (Recommended for MVP)
- **Polling** (5-10 sec intervals)
- Simple, works offline
- Sufficient for notes feed

### Option B (Future Enhancement)
- **Socket.io** on separate server
- Real-time collaboration
- Post-MVP feature

---

## Hosting & Deployment

### Frontend & API Routes
- **Vercel** (recommended)
  - Native Next.js support
  - Automatic deployments
  - $0 startup cost

### Database
- **Supabase** (PostgreSQL + Auth)
  - Managed PostgreSQL
  - Built-in auth (optional)
  - ~$25-50/month for MVP scale
  
  **OR**
  
- **Railway** ($5/month starter)
  - Simple PostgreSQL hosting
  - Good for small projects

---

## Development Tools

### Version Control
- Git + GitHub

### Package Manager
- pnpm (faster than npm/yarn)

### Linting & Formatting
- ESLint
- Prettier

### Testing (Phase 2)
- Vitest (unit tests)
- Playwright (e2e tests)

---

## File Storage

### For Photos & Attachments
- **Supabase Storage** (if using Supabase)
  - ~$5/GB
  - Simple integration
  
  **OR**
  
- **AWS S3** (standard choice)
  - Pay per usage (~$1-5/month for MVP)
  - Industry standard

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT LAYER                         │
│  Next.js (React + TypeScript + TailwindCSS)             │
│  - Responsive (Mobile, Tablet, Desktop)                 │
│  - Offline Mode (Service Worker + IndexedDB)            │
│  - State: Zustand + React Query                         │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTP/REST
┌──────────────────────▼──────────────────────────────────┐
│                  API LAYER                              │
│  Next.js API Routes (/api/...)                          │
│  NextAuth.js (Auth + Session)                           │
│  Middleware (RBAC enforcement)                          │
└──────────────────────┬──────────────────────────────────┘
                       │ Prisma Client
┌──────────────────────▼──────────────────────────────────┐
│              DATABASE LAYER                             │
│  PostgreSQL                                             │
│  Prisma ORM                                             │
└──────────────────────────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│           STORAGE (Files & Images)                      │
│  Supabase Storage OR AWS S3                             │
└──────────────────────────────────────────────────────────┘
```

---

## MVP Deployment Checklist

- [ ] Next.js app on Vercel
- [ ] PostgreSQL on Supabase or Railway
- [ ] NextAuth configured
- [ ] Environment variables set
- [ ] Database migrations run
- [ ] File storage configured
- [ ] SSL/HTTPS enabled
- [ ] Error logging (Sentry optional)
- [ ] Analytics (Vercel Analytics included)

---

## Cost Estimate (Monthly, MVP Scale)

| Service | Cost | Notes |
|---------|------|-------|
| Vercel | $0-20 | Free tier enough for MVP |
| Supabase / Railway | $25-50 | Database hosting |
| S3 / Supabase Storage | $5-10 | File uploads |
| Domain | $10 | Annual |
| **Total** | **$40-80** | Runs entire MVP |

---

## Why This Stack?

✅ **Purpose-built for MVP**
- Next.js eliminates the need for separate frontend + backend repos
- Single deployment, single codebase
- Fast to iterate

✅ **Offline-first thinking**
- Service Workers + IndexedDB built in
- No dependency on external packages
- React Query handles sync automatically

✅ **Type-safe throughout**
- TypeScript + Prisma = end-to-end type safety
- Fewer runtime bugs

✅ **Scales from hobby to production**
- Works great at 10 users
- Works great at 10,000 users
- Easy to migrate pieces if needed

✅ **Cost-effective**
- Most services have generous free tiers
- Pay as you grow
- No vendor lock-in

