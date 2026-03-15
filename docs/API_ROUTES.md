# API Route Structure

This documents all REST endpoints for the Horse & Yard Management MVP.

## Authentication Endpoints

### POST /api/auth/register
Register a new user (manager only)
```json
{
  "email": "manager@example.com",
  "password": "hashed",
  "name": "John Manager",
  "yardId": "yard_123"
}
```
Response: `{ userId, token, role }`

### POST /api/auth/login
```json
{
  "email": "user@example.com",
  "password": "hashed"
}
```
Response: `{ userId, token, role, yardId }`

### POST /api/auth/logout
Invalidate session token

### GET /api/auth/me
Get current authenticated user profile

---

## Horse Management Endpoints

### GET /api/horses
List all horses in yard (filter by role)
**Query params:**
- `yardId`: Yard ID
- `assignedGroom`: Filter by groom (groom view)
- `search`: Search by name

**Response:**
```json
{
  "horses": [
    {
      "id": "horse_123",
      "name": "Equestrian",
      "age": 5,
      "breed": "Thoroughbred",
      "passportNumber": "123456",
      "assignedGroom": "user_456",
      "owner": { "id": "user_789", "name": "Jane Owner" },
      "feedPlan": "2 scoops morning, 3 evening",
      "supplements": ["Biotin", "Omega 3"],
      "activeMedication": "Phenylbutazone (4 weeks)",
      "workSchedule": "Mon, Wed, Fri - 1hr",
      "lastNoteAt": "2026-03-01T10:30:00Z",
      "currentAlertLevel": "GREEN"
    }
  ],
  "total": 7
}
```

### GET /api/horses/:horseId
Get single horse profile + full history
Returns: basic info + vaccination history + recent notes + medical history

### POST /api/horses
Create new horse (MANAGER only)
```json
{
  "name": "New Horse",
  "age": 3,
  "breed": "Arabian",
  "passportNumber": "987654",
  "assignedGroom": "user_456"
}
```

### PATCH /api/horses/:horseId
Update horse metadata (MANAGER only)
```json
{
  "feedPlan": "New plan",
  "supplements": ["Biotin"],
  "activeMedication": "Phenylbutazone"
}
```

### DELETE /api/horses/:horseId
Soft delete horse (MANAGER only, sets `isDraft = true`)

### POST /api/horses/:horseId/vaccinations
Add vaccination record
```json
{
  "name": "Tetanus",
  "nextDueDate": "2027-03-01",
  "lastDate": "2023-03-01",
  "notes": "Given by Dr. Smith"
}
```

### GET /api/horses/:horseId/medical-history
Get medical history timeline

### POST /api/horses/:horseId/medical-history
Add medical history entry
```json
{
  "date": "2026-03-01",
  "condition": "Slight lameness",
  "treatment": "Rest + ice",
  "vetNotes": "Return to work in 3 days"
}
```

---

## Task Management Endpoints

### GET /api/tasks
List tasks (filtered by role + date)
**Query params:**
- `yardId`: Yard ID
- `horseId`: Filter by horse (optional)
- `date`: Single date or date range
- `status`: PENDING, COMPLETED, MISSED, OVERDUE
- `assignedGroom`: Filter by groom

**Response:**
```json
{
  "tasks": [
    {
      "id": "task_123",
      "name": "AM Feed",
      "category": "FEEDING",
      "dueDate": "2026-03-01T08:00:00Z",
      "horseId": "horse_123",
      "horseName": "Equestrian",
      "status": "COMPLETED",
      "completedAt": "2026-03-01T08:15:00Z",
      "completedBy": { "id": "user_456", "name": "Sarah Groom" }
    }
  ],
  "stats": {
    "total": 42,
    "completed": 38,
    "pending": 4,
    "overdue": 0
  }
}
```

### GET /api/tasks/today
Get today's tasks for a groom
Simplified response for mobile/offline

### POST /api/tasks
Create task (MANAGER only)
```json
{
  "name": "AM Feed",
  "description": "2 scoops main feed",
  "category": "FEEDING",
  "frequency": "DAILY",
  "dueDate": "2026-03-01",
  "horseId": "horse_123"
}
```

### PATCH /api/tasks/:taskId
Update task (MANAGER only)
```json
{
  "name": "Updated name",
  "dueDate": "2026-03-02"
}
```

### DELETE /api/tasks/:taskId
Delete task (MANAGER only)

### POST /api/tasks/:taskId/complete
Mark task as complete (GROOM or MANAGER)
```json
{
  "completedBy": "user_456",
  "notes": "Horse ate well"
}
```

### POST /api/tasks/generate-daily
**Cron endpoint (RUN AT 12 AM)**
Generates today's tasks from yard templates
- Called via Vercel cron job
- Creates Task instances for each recurring template
- Returns: `{ created: 42, errors: [] }`

### GET /api/task-templates
List all task templates (MANAGER view)

### POST /api/task-templates
Create task template (MANAGER only)
```json
{
  "name": "Daily Feed Schedule",
  "category": "FEEDING",
  "frequency": "DAILY",
  "isYardWide": false,
  "assignedHorses": ["horse_123", "horse_456"]
}
```

### DELETE /api/task-templates/:templateId
Delete template (MANAGER only)
- Does NOT delete past task instances
- Only stops future generation

---

## Notes Endpoints

### GET /api/notes
Get notes feed (filtered by horse + tag)
**Query params:**
- `horseId`: Horse ID (required)
- `tag`: HEALTH, TRAINING, BEHAVIOR, etc (optional)
- `limit`: 50 (default)
- `offset`: 0 (for pagination)

**Response:**
```json
{
  "notes": [
    {
      "id": "note_123",
      "text": "Slight swelling in LF",
      "tag": "INJURY",
      "horseId": "horse_123",
      "createdBy": { "id": "user_456", "name": "Sarah Groom" },
      "attachments": [
        {
          "id": "attach_789",
          "filename": "swelling.jpg",
          "url": "https://s3.../swelling.jpg",
          "mimeType": "image/jpeg"
        }
      ],
      "createdAt": "2026-03-01T14:30:00Z"
    }
  ],
  "total": 156,
  "hasMore": true
}
```

### POST /api/notes
Create new note (GROOM or MANAGER)
```json
{
  "horseId": "horse_123",
  "text": "Slight swelling in LF",
  "tag": "INJURY"
}
```
Response: Created note object with ID + timestamp

### POST /api/notes/:noteId/attachments
Upload attachment to note
- Multipart form: `file` + `filetype`
- Returns: Attachment object with signed S3 URL

### GET /api/notes/search
Search notes across yard
**Query params:**
- `q`: Search query
- `tag`: Filter by tag
- `horseId`: Filter by horse
- `startDate`, `endDate`: Date range

Returns: Matching notes sorted by relevance + date

---

## Dashboard Endpoints

### GET /api/dashboard/overview
Manager dashboard summary
```json
{
  "todayCompliance": {
    "total": 42,
    "completed": 38,
    "percentage": 90.5
  },
  "alertHorses": [
    {
      "id": "horse_123",
      "name": "Equestrian",
      "alertLevel": "RED",
      "reasons": [
        "2 missed tasks",
        "Medication overdue"
      ]
    }
  ],
  "horsesOnMedication": 3,
  "recentHealthNotes": [
    { "id": "note_123", "horseName": "Equestrian", "text": "...", "createdAt": "..." }
  ],
  "missedTasks": [
    { "id": "task_123", "name": "AM Feed", "horseName": "Equestrian", "dueDate": "..." }
  ]
}
```

### GET /api/dashboard/compliance/:horseId
Detailed compliance for single horse
```json
{
  "horse": { ... },
  "thisWeek": {
    "tasksCompleted": 35,
    "tasksTotal": 40,
    "percentage": 87.5
  },
  "thisMonth": { ... },
  "missedTasks": [ ... ],
  "upcomingVaccinations": [ ... ]
}
```

### GET /api/dashboard/health-alerts
List all health-tagged notes from last 7 days

---

## Expense Endpoints

### GET /api/expenses
List expenses (MANAGER only)
**Query params:**
- `yardId`: Yard ID
- `startDate`, `endDate`: Date range
- `category`: FEED, VET, FARRIER, etc
- `horseId`: Filter by horse (optional)

**Response:**
```json
{
  "expenses": [
    {
      "id": "exp_123",
      "date": "2026-03-01",
      "vendor": "Equine Supplies Ltd",
      "category": "FEED",
      "amount": 250.50,
      "notes": "50kg mixed feed + supplements",
      "horseId": "horse_123",
      "createdBy": "user_456"
    }
  ],
  "total": 1500.75,
  "byCategory": {
    "FEED": 800,
    "VET": 400,
    "FARRIER": 300
  }
}
```

### POST /api/expenses
Create expense (MANAGER only)
```json
{
  "date": "2026-03-01",
  "vendor": "Equine Supplies",
  "category": "FEED",
  "amount": 250.50,
  "notes": "Monthly feed order",
  "horseId": "horse_123" // optional
}
```

### PATCH /api/expenses/:expenseId
Update expense (MANAGER only)

### DELETE /api/expenses/:expenseId
Delete expense (MANAGER only)

### POST /api/expenses/export
Export expenses to Excel/PDF
**Query params:**
- `format`: "xlsx" or "pdf"
- `startDate`, `endDate`: Date range

Returns: File download

---

## Attachment Endpoints

### POST /api/attachments/upload
Upload file to S3/Supabase
- Accepts multipart form: `file`
- Returns: `{ url, filename, mimeType, size }`

### DELETE /api/attachments/:attachmentId
Delete file (MANAGER only, or creator)
- Deletes from S3 + database

---

## Middleware & Error Handling

### Authentication Middleware
```typescript
export function withAuth(handler) {
  return async (req, res) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'Unauthorized' });
    
    const user = await verifyToken(token);
    req.user = user;
    return handler(req, res);
  };
}
```

### Authorization Middleware
```typescript
export function requireRole(...roles: Role[]) {
  return (handler) => withAuth(async (req, res) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    return handler(req, res);
  });
}
```

### CORS
- Configured for frontend domain(s)
- Credentials included

### Rate Limiting
- 100 requests per minute (general)
- 1000 requests per day (API tier)

---

## Offline Sync Queue

When offline:
1. Client stores requests in IndexedDB
2. POST requests get temporary IDs (offline_123)
3. On reconnect, client retries all queued requests
4. Server merges conflicts (latest timestamp wins)
5. Client reconciles IDs

Endpoints involved:
- `POST /api/tasks/:taskId/complete`
- `POST /api/notes`
- Local storage in browser

See frontend docs for detailed implementation.

