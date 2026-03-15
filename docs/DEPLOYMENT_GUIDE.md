# Development & Deployment Guide

## 🛠️ Local Development Setup

### Prerequisites
- Node.js 18+
- PostgreSQL 14+ (local or Docker)
- pnpm or npm

### Step 1: Clone & Install

```bash
cd app
pnpm install
```

### Step 2: Environment Configuration

Copy `.env.example` to `.env.local`:

```bash
cp .env.example .env.local
```

Edit `.env.local` with your settings:

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/yard_sync"

# Auth
NEXTAUTH_SECRET="your-secret-key-change-in-production"
NEXTAUTH_URL="http://localhost:3000"

# File Storage (pick one)
# Option A: Supabase
NEXT_PUBLIC_SUPABASE_URL="https://your-project.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"

# Option B: AWS S3
AWS_ACCESS_KEY_ID="your-key"
AWS_SECRET_ACCESS_KEY="your-secret"
AWS_S3_BUCKET="yard-sync-dev"
```

### Step 3: Database Setup

#### Option A: Local PostgreSQL

```bash
# macOS with Homebrew
brew install postgresql
brew services start postgresql

# Create database
createdb yard_sync

# Update DATABASE_URL in .env.local
DATABASE_URL="postgresql://localhost:5432/yard_sync"
```

#### Option B: Docker PostgreSQL

```bash
docker run -d \
  --name yard-sync-db \
  -e POSTGRES_DB=yard_sync \
  -e POSTGRES_PASSWORD=yourpassword \
  -p 5432:5432 \
  postgres:15

# Update DATABASE_URL
DATABASE_URL="postgresql://postgres:yourpassword@localhost:5432/yard_sync"
```

#### Option C: Cloud Database

Use Supabase PostgreSQL (free tier):
1. Create project at https://supabase.com
2. Get DATABASE_URL from project settings
3. Paste into `.env.local`

### Step 4: Run Prisma Migrations

```bash
# Generate Prisma client
pnpm run db:generate

# Run migrations (creates tables)
pnpm run db:migrate

# Optional: Seed test data
pnpm run db:seed
```

### Step 5: Start Development Server

```bash
pnpm run dev
```

Visit http://localhost:3000

---

## 🧪 Development Commands

```bash
# Run dev server
pnpm run dev

# Build for production
pnpm run build

# Start production server
pnpm run start

# Type checking
pnpm run type-check

# Linting
pnpm run lint

# Database studio (visual editor)
pnpm run db:studio

# Database seed script
pnpm run db:seed

# Interactive migrations
pnpm run db:migrate
```

---

## 📦 Production Deployment

### Option A: Deploy to Vercel (Recommended)

**Prerequisites:**
- GitHub account with repo
- Supabase or RDS PostgreSQL database
- Domain (optional)

**Steps:**

1. Push code to GitHub

2. Connect Vercel:
   - Go to https://vercel.com
   - Click "New Project"
   - Select your GitHub repo
   - Click "Import"

3. Configure environment variables:
   - `DATABASE_URL` - Your PostgreSQL connection string
   - `NEXTAUTH_SECRET` - Generate: `openssl rand -base64 32`
   - `NEXTAUTH_URL` - Your production URL
   - `NEXT_PUBLIC_SUPABASE_URL` - If using Supabase
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - etc.

4. Click "Deploy"

5. Run database migrations:
   ```bash
   # From Vercel dashboard > Settings > Functions
   # Or run locally:
   pnpm run db:push --skip-generate
   ```

6. Add custom domain (if needed):
   - Vercel > Settings > Domains
   - Add your domain
   - Update DNS records

---

### Option B: Deploy to Railway

**Steps:**

1. Create account at https://railway.app

2. Create new project > Provision PostgreSQL

3. Connect GitHub repo

4. Set environment variables in Railway dashboard

5. Auto-deploys on push to main

---

### Option C: Self-Hosted (AWS, DigitalOcean, etc)

**Prerequisites:**
- VPS with Node.js
- PostgreSQL database
- SSL certificate (Let's Encrypt)
- Reverse proxy (Nginx)

**Setup Nginx:**

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Start Application:**

```bash
# Install PM2 (process manager)
npm install -g pm2

# Start app
pm2 start "pnpm start" --name "yard-sync"

# Auto-restart on reboot
pm2 startup
pm2 save
```

---

## 🔒 Security Checklist

Before production:

- [ ] `NEXTAUTH_SECRET` is strong (32+ chars)
- [ ] Environment variables removed from code
- [ ] Database has backups configured
- [ ] HTTPS/SSL enabled
- [ ] Rate limiting configured
- [ ] CORS properly set
- [ ] Sensitive endpoints protected with auth
- [ ] Database credentials rotated
- [ ] Monitoring/logging enabled (Sentry)

---

## 📊 Monitoring & Logging

### Application Monitoring

Add Sentry for error tracking:

```bash
pnpm install @sentry/nextjs
```

Update `next.config.js`:

```javascript
const withSentry = require("@sentry/nextjs/withSentry");

module.exports = withSentry({
  // ... existing next.config
}, {
  org: "your-org",
  project: "yard-sync",
});
```

### Database Backups

**Supabase:** Automatic daily backups (free tier)

**AWS RDS:** Configure automated backups in console

**Self-Hosted:** Use pgBackRest or WAL archiving

---

## 🐛 Troubleshooting

### Database Connection Issues

```bash
# Test connection
psql $DATABASE_URL

# View Prisma logs
DEBUG="prisma:*" pnpm run db:migrate
```

### Build Failures

```bash
# Clear Next.js cache
rm -rf .next

# Reinstall dependencies
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Performance Issues

```bash
# Analyze bundle size
pnpm add -D @next/bundle-analyzer

# Check database query performance
pnpm run db:studio
# Look for slow queries
```

---

## 📈 Scaling Checklist

As users grow:

- [ ] Database query optimization (add indexes)
- [ ] Implement caching (Redis)
- [ ] CDN for static assets
- [ ] API rate limiting
- [ ] Database replication for redundancy
- [ ] Load balancer setup
- [ ] Horizontal scaling strategy

---

## 🔄 Continuous Integration/Deployment

### GitHub Actions (Optional)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm run build
      - run: pnpm run type-check
      
      - name: Deploy to Vercel
        run: vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## 📞 Support

For issues:
1. Check error logs: `pnpm run dev` (dev) or Vercel dashboard (prod)
2. Review [troubleshooting guide](#-troubleshooting)
3. Check database schema: `pnpm run db:studio`
4. Review API responses: Browser DevTools Network tab

