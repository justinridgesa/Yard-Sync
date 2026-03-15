# Database Setup Instructions

## ⭐ EASIEST OPTION: Supabase Cloud (Recommended for MVP)

Free tier, no installation, production-grade PostgreSQL.

### Step 1: Create Supabase Project (5 minutes)
1. Go to https://supabase.com
2. Click "Sign Up" 
3. Create account with email
4. Click "New Project"
5. Enter:
   - Project name: `yard-sync`
   - Password: (saved for you)
   - Region: Choose closest to you
6. Wait 2-3 minutes for creation

### Step 2: Get Connection String
1. In Supabase dashboard, go to "Settings" → "Database"
2. Find "Connection string" section
3. Copy the PostgreSQL connection string
   - Should look like: `postgresql://postgres.[ID]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres`

### Step 3: Update .env.local
```bash
# Replace the DATABASE_URL line in app/.env.local with your Supabase URL
DATABASE_URL="postgresql://postgres.[ID]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres"
```

### Step 4: Create Database (This folder)
```powershell
cd c:\VibeWS\Yard Sync\app
pnpm run db:generate
pnpm run db:migrate
pnpm run db:seed
pnpm run dev
```

That's it! ✅

---

## Alternative Options

### Option 1: Install PostgreSQL Locally
1. Download from https://www.postgresql.org/download/windows/
2. Install (keep default settings)
3. Note your password
4. Update .env.local: `DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/yard_sync"`

### Option 2: Use Railway.app
1. Go to https://railway.app
2. Sign up with GitHub
3. Create new project → Provision PostgreSQL
4. Copy connection string
5. Update .env.local

### Option 3: Docker Route (requires Docker Desktop)
1. Install Docker Desktop from https://www.docker.com/products/docker-desktop
2. Run: `docker run -d --name yard-sync-db -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15`
3. Update .env.local: `DATABASE_URL="postgresql://postgres:postgres@localhost:5432/yard_sync"`

---

## Which option are you choosing?

Reply with:
- **"supabase"** - Use Supabase Cloud (I'll wait for your connection string)
- **"postgres"** - Local PostgreSQL (install first, give me password)
- **"railway"** - Use Railway.app
- **"docker"** - Use Docker (will need to install Docker Desktop first)
