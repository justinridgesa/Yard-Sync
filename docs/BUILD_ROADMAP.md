# Horse & Yard Management - Build Roadmap

## Phase 1: Foundation (Week 1-2)

### 1.1 Project Setup
- [ ] Initialize Next.js 14 project
- [ ] Set up PostgreSQL + Prisma
- [ ] Configure environment variables
- [ ] Set up NextAuth.js authentication
- [ ] Configure TypeScript strict mode
- [ ] Set up TailwindCSS + component library

### 1.2 Database Infrastructure
- [ ] Write Prisma schema
- [ ] Run initial migrations
- [ ] Create seed script for test data (7 horses, 2 grooms, 2 managers)
- [ ] Set up database backups

### 1.3 Authentication & RBAC
- [ ] Implement user registration (manager creates accounts)
- [ ] Implement login/logout
- [ ] Session management with NextAuth
- [ ] Role-based middleware
- [ ] Protected API routes
- [ ] Protected frontend pages

### 1.4 Basic UI Shell
- [ ] Create responsive layout (desktop/tablet/mobile)
- [ ] Navigation structure per role
- [ ] Authentication pages (login)

**Deliverable:** Working deployment with auth

---

## Phase 2: Core Horse Management (Week 3-4)

### 2.1 Horse Module
- [ ] Create horse list page (manager + groom view)
- [ ] Horse detail page (profile + history)
- [ ] Edit horse basic info (manager only)
- [ ] Add vaccination records
- [ ] Add medical history entries
- [ ] Display attachments (photos, vet reports)

### 2.2 Groom View
- [ ] "My Horses" page (filtered to assigned horses)
- [ ] Simple horse card UI
- [ ] Basic info display

### 2.3 Owner View (Read-Only)
- [ ] Owner can see assigned horse(s)
- [ ] View health updates
- [ ] View training notes (no editing)

**Deliverable:** Full horse lifecycle visible in app

---

## Phase 3: Task System (Week 5-6)

### 3.1 Daily Task Generation
- [ ] Create task template system (manager creates recurring patterns)
- [ ] Implement daily task generator (cron job at midnight)
- [ ] Assign multiple horses to templates
- [ ] Display today's tasks per horse

### 3.2 Task Completion
- [ ] Groom marks tasks complete
- [ ] Auto-timestamp + user logging
- [ ] Task completion history
- [ ] Missed task detection

### 3.3 Task Dashboard
- [ ] Manager sees compliance % per day
- [ ] Missed tasks highlighted
- [ ] Per-horse task history

### 3.4 Task Offline Support
- [ ] Cache today's tasks on login
- [ ] Allow offline task completion
- [ ] Queue syncing on reconnect

**Deliverable:** Grooms can complete tasks offline

---

## Phase 4: Notes & Activity Feed (Week 7-8)

### 4.1 Notes Feed
- [ ] Create note for each horse
- [ ] Tag notes (HEALTH, TRAINING, BEHAVIOR, INJURY, FEED)
- [ ] Chronological feed display
- [ ] Photo attachment to notes
- [ ] Search + filter notes by tag

### 4.2 Activity Intelligence
- [ ] Flag: "No notes in 24h"
- [ ] Show recent activity on horse profile
- [ ] Feed page shows all yard notes (searchable)

### 4.3 Offline Sync
- [ ] Queue note creation when offline
- [ ] Sync on reconnect
- [ ] Conflict resolution (manual review if needed)

**Deliverable:** Replace WhatsApp with structured notes

---

## Phase 5: Manager Dashboard & Alerts (Week 9-10)

### 5.1 Dashboard Overview
- [ ] Today's compliance % (simple metric)
- [ ] Traffic light system (🟢 🟠 🔴 per horse)
- [ ] Alert horses list
- [ ] Horses on medication
- [ ] Recent health notes widget
- [ ] Missed tasks widget

### 5.2 Alert Logic
- [ ] Missed task → AMBER alert
- [ ] 2+ missed tasks → RED alert
- [ ] Medication overdue → RED alert
- [ ] Vaccination overdue → AMBER alert
- [ ] No notes in 24h → soft prompt

### 5.3 Compliance Tracking
- [ ] Per-horse compliance detail page
- [ ] Weekly/monthly compliance trends
- [ ] Filter horses by alert level

**Deliverable:** Managers have real-time visibility

---

## Phase 6: Expense Logging (Week 11)

### 6.1 Expense Module (Manager Only)
- [ ] Create expense form (date, vendor, category, amount)
- [ ] Optional horse assignment
- [ ] Expense list with filtering
- [ ] Category breakdown chart

### 6.2 Export Functionality
- [ ] Export to Excel (date range + category)
- [ ] Export to PDF (formatted report)

**Deliverable:** Cost tracking integrated

---

## Phase 7: External Professionals (Week 12)

### 7.1 Temporary Access
- [ ] Manager can grant temporary vet/farrier access
- [ ] Link expires after X days or access revoked manually
- [ ] Professional can upload reports
- [ ] Professional can read horse health data (limited)

### 7.2 Professional Dashboard
- [ ] Simple view of assigned horses
- [ ] Attachment upload
- [ ] Cannot see notes/tasks

**Deliverable:** Vets + farriers integrated securely

---

## Phase 8: Polishing & Testing (Week 13-14)

### 8.1 Mobile Optimization
- [ ] Test on iOS + Android
- [ ] Touch-friendly buttons
- [ ] Offline functionality verified
- [ ] Service worker tested

### 8.2 Performance
- [ ] Optimize images
- [ ] Lazy load lists
- [ ] API response caching
- [ ] Bundle size audit

### 8.3 Testing
- [ ] Unit tests for core logic
- [ ] E2E tests for critical flows (login, task completion, note creation)
- [ ] Accessibility audit (WCAG AA)
- [ ] Security audit (OWASP)

### 8.4 Deployment
- [ ] Production environment setup
- [ ] SSL/HTTPS enforced
- [ ] Database backups automated
- [ ] Error logging (Sentry)
- [ ] Monitoring (Vercel/New Relic)

**Deliverable:** Production-ready MVP

---

## Phase 9: Launch & Monitoring (Week 15)

### 9.1 Pre-Launch
- [ ] Load testing (simulate 7 horses, 4 users)
- [ ] Data migration scripts ready
- [ ] Rollback plan documented
- [ ] Staff training materials

### 9.2 Launch
- [ ] Deploy to production
- [ ] Initial user onboarding
- [ ] Monitor error logs
- [ ] Quick bug fixes

### 9.3 Post-Launch (Week 1-4)
- [ ] User feedback collection
- [ ] Bug fixes
- [ ] Performance tuning
- [ ] Small feature iterations

---

## Post-MVP Roadmap (Future Phases)

### Phase 10: Real-Time Collaboration (Week 16-17)
- Socket.io for live note updates
- Real-time task completion notifications
- Multi-user editing

### Phase 11: Advanced Analytics
- 30/60/90 day compliance trends
- Health pattern detection
- Feed cost per horse
- Training performance metrics

### Phase 12: Mobile App
- React Native or Flutter
- Better offline support
- Push notifications
- Biometric login

### Phase 13: Integrations
- Calendar integrations (Google Calendar, Outlook)
- Email notifications
- SMS alerts for critical items
- Slack/Teams bot for notifications

### Phase 14: Advanced Reporting
- Custom report builder
- Automated email reports
- League tables (horse rankings)
- KPI dashboards

---

## Success Metrics

### MVP Success
- ✅ All 7 horses visible in app
- ✅ Grooms can complete tasks offline
- ✅ Manager dashboard shows real-time status
- ✅ No WhatsApp required for task updates
- ✅ Full audit trail for compliance

### User Adoption
- 80% groom-initiated tasks completed via app (not reminders)
- 0 missed medication alerts
- Manager dashboards checked daily
- Notes replaces 90% of WhatsApp usage

### System Reliability
- 99.5% uptime
- < 2 second page load time
- < 5 second offline sync time
- Zero data loss incidents

---

## Resource Requirements

### Team
- 1 Full-Stack Developer (2 weeks)
- 1 Frontend Developer (2 weeks UI/UX polish)
- 1 QA/Tester (concurrent with development)
- 1 Project Manager (part-time)

### Infrastructure
- PostgreSQL database: $25-50/month
- Vercel deployment: $0-20/month
- File storage: $5-10/month
- **Total: ~$40-80/month**

### Development Tools
- GitHub (free)
- Vercel (free tier sufficient)
- Supabase/Railway (free tier for dev)

