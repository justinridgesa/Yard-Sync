# Horse & Yard Management - Database Schema

## Overview

The database is designed around these key entities:

- **Users** (managers, grooms, owners, professionals)
- **Horses** (core entity with profiles)
- **Tasks** (recurring + daily instances)
- **Notes** (activity feed)
- **Expenses** (cost tracking)
- **Attachments** (files linked to horses/tasks)

---

## Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============= ENUMS =============

enum Role {
  YARD_MANAGER
  GROOM
  OWNER
  VET
  FARRIER
}

enum TaskStatus {
  PENDING
  COMPLETED
  MISSED
  OVERDUE
}

enum TaskFrequency {
  DAILY
  WEEKLY
  MONTHLY
  ONCE
}

enum TaskCategory {
  FEEDING
  TURNOUT
  MUCKING
  SUPPLEMENTS
  MEDICATION
  EXERCISE
  GROOMING
  HEALTH_CHECK
  YARD_MAINTENANCE
  OTHER
}

enum NoteTag {
  HEALTH
  TRAINING
  BEHAVIOR
  INJURY
  FEED
  EQUIPMENT
  BEHAVIOR_CHANGE
  OTHER
}

enum ExpenseCategory {
  FEED
  VET
  FARRIER
  BEDDING
  LABOUR
  EQUIPMENT
  OTHER
}

enum AlertLevel {
  GREEN      // Compliant
  AMBER      // Needs attention
  RED        // Urgent
}

// ============= CORE MODELS =============

model User {
  id            String     @id @default(cuid())
  email         String     @unique
  name          String
  password      String     // Hashed (NextAuth)
  role          Role
  yardId        String
  yard          Yard       @relation(fields: [yardId], references: [id])
  
  // Groom assignments
  assignedHorses Horse[]    @relation("GroomAssignment")
  
  // Ownership
  ownedHorses    Horse[]    @relation("OwnerAssignment")
  
  // Activity tracking
  createdTasks   Task[]
  completedTasks TaskCompletion[]
  createdNotes   Note[]
  createdExpenses Expense[]
  attachments    Attachment[]
  
  // Session management
  sessions       Session[]
  
  createdAt      DateTime   @default(now())
  updatedAt      DateTime   @updatedAt

  @@index([yardId])
  @@index([role])
}

model Session {
  id        String     @id @default(cuid())
  userId    String
  user      User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String     @unique
  expiresAt DateTime
  
  createdAt DateTime   @default(now())

  @@index([userId])
}

model Yard {
  id            String     @id @default(cuid())
  name          String
  location      String?
  users         User[]
  horses        Horse[]
  tasks         Task[]
  taskTemplates TaskTemplate[]
  notes         Note[]
  expenses      Expense[]
  
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
}

// ============= HORSE MANAGEMENT =============

model Horse {
  id                String      @id @default(cuid())
  yardId            String
  yard              Yard        @relation(fields: [yardId], references: [id])
  
  // Basic Info
  name              String
  age               Int
  breed             String?
  passportNumber    String?     @unique
  
  // Assignments
  assignedGroom     String?
  groom             User?       @relation("GroomAssignment", fields: [assignedGroom], references: [id])
  owner             User?       @relation("OwnerAssignment", fields: [owner], references: [id])
  ownerId           String?
  
  // Operational Data
  feedPlan          String?     // Rich text description
  supplements       String?     // Current supplements list
  activeMedication  String?     // Current medication list
  workSchedule      String?     // Training schedule
  
  // Health Status
  vaccinations      Vaccination[]
  medicalHistory    MedicalHistory[]
  
  // Activity
  tasks             Task[]
  taskCompletions   TaskCompletion[]
  notes             Note[]
  expenses          Expense[]
  attachments       Attachment[]
  
  // Dashboard flags
  currentAlertLevel AlertLevel  @default(GREEN)
  lastNoteAt        DateTime?   // For "no notes in 24h" check
  medicationOverdue Boolean     @default(false)
  
  isDraft           Boolean     @default(false) // Soft delete for testing
  
  createdAt         DateTime    @default(now())
  updatedAt         DateTime    @updatedAt

  @@index([yardId])
  @@index([assignedGroom])
  @@index([ownerId])
}

model Vaccination {
  id            String    @id @default(cuid())
  horseId       String
  horse         Horse     @relation(fields: [horseId], references: [id], onDelete: Cascade)
  
  name          String    // e.g., "Tetanus", "Flu"
  nextDueDate   DateTime
  lastDate      DateTime?
  notes         String?
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([horseId])
}

model MedicalHistory {
  id            String    @id @default(cuid())
  horseId       String
  horse         Horse     @relation(fields: [horseId], references: [id], onDelete: Cascade)
  
  date          DateTime
  condition     String
  treatment     String?
  vetNotes      String?
  attachments   Attachment[]
  
  createdAt     DateTime  @default(now())

  @@index([horseId])
}

// ============= TASK MANAGEMENT =============

model TaskTemplate {
  id            String    @id @default(cuid())
  yardId        String
  yard          Yard      @relation(fields: [yardId], references: [id])
  
  name          String    // e.g., "Daily Feed Schedule"
  category      TaskCategory
  frequency     TaskFrequency // DAILY, WEEKLY, MONTHLY
  
  // For recurring: which day(s)/time(s)
  dayOfWeek     Int?      // 0=Mon, 6=Sun (for weekly)
  dayOfMonth    Int?      // 1-31 (for monthly)
  
  assignedHorses Horse[]   // Many-to-many via TaskTemplateAssignment
  isYardWide    Boolean   @default(false)
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([yardId])
}

model Task {
  id            String    @id @default(cuid())
  yardId        String
  yard          Yard      @relation(fields: [yardId], references: [id])
  
  // What
  name          String    // "AM Feed"
  description   String?
  category      TaskCategory
  
  // When
  frequency     TaskFrequency // DAILY, WEEKLY, MONTHLY, or ONCE
  dueDate       DateTime
  completedAt   DateTime?
  
  // Who
  horseId       String?   // NULL if yard-wide task
  horse         Horse?    @relation(fields: [horseId], references: [id], onDelete: SetNull)
  
  // Status
  status        TaskStatus @default(PENDING)
  completedBy   String?   // User ID who completed it
  
  // Tracking
  createdById   String    // User who created this task
  createdBy     User      @relation(fields: [createdById], references: [id])
  
  completions   TaskCompletion[]
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([horseId])
  @@index([yardId])
  @@index([dueDate])
  @@index([status])
}

model TaskCompletion {
  id            String    @id @default(cuid())
  taskId        String
  task          Task      @relation(fields: [taskId], references: [id], onDelete: Cascade)
  
  horseId       String?
  horse         Horse?    @relation(fields: [horseId], references: [id], onDelete: SetNull)
  
  completedBy   String
  user          User      @relation(fields: [completedBy], references: [id])
  
  completedAt   DateTime  @default(now())
  notes         String?   // Free-form notes when completing
  
  createdAt     DateTime  @default(now())

  @@index([taskId])
  @@index([horseId])
  @@index([completedBy])
}

// ============= NOTES & ACTIVITY FEED =============

model Note {
  id            String    @id @default(cuid())
  
  yardId        String
  yard          Yard      @relation(fields: [yardId], references: [id])
  
  horseId       String
  horse         Horse     @relation(fields: [horseId], references: [id], onDelete: Cascade)
  
  // Content
  text          String
  tag           NoteTag
  
  // Who & When
  createdBy     String
  author        User      @relation(fields: [createdBy], references: [id])
  
  // Files
  attachments   Attachment[]
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([horseId])
  @@index([yardId])
  @@index([createdAt]) // For feed ordering
  @@index([tag])
}

// ============= EXPENSES =============

model Expense {
  id            String    @id @default(cuid())
  
  yardId        String
  yard          Yard      @relation(fields: [yardId], references: [id])
  
  date          DateTime
  vendor        String
  category      ExpenseCategory
  amount        Float
  notes         String?
  
  // Optional horse assignment
  horseId       String?
  horse         Horse?    @relation(fields: [horseId], references: [id], onDelete: SetNull)
  
  createdBy     String
  user          User      @relation(fields: [createdBy], references: [id])
  
  attachments   Attachment[]
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@index([yardId])
  @@index([date])
  @@index([horseId])
}

// ============= FILES =============

model Attachment {
  id            String    @id @default(cuid())
  
  // Polymorphic: attached to Horse, Note, Expense, or MedicalHistory
  horseId       String?
  horse         Horse?    @relation(fields: [horseId], references: [id], onDelete: Cascade)
  
  noteId        String?
  note          Note?     @relation(fields: [noteId], references: [id], onDelete: Cascade)
  
  expenseId     String?
  expense       Expense?  @relation(fields: [expenseId], references: [id], onDelete: Cascade)
  
  medicalHistoryId String?
  medicalHistory MedicalHistory? @relation(fields: [medicalHistoryId], references: [id], onDelete: Cascade)
  
  // File metadata
  filename      String
  mimeType      String
  size          Int       // bytes
  url           String    // S3 or Supabase URL
  
  uploadedBy    String
  uploader      User      @relation(fields: [uploadedBy], references: [id])
  
  createdAt     DateTime  @default(now())

  @@index([horseId])
  @@index([noteId])
  @@index([expenseId])
}
```

---

## Key Design Decisions

### 1. **Task Model Design**
- `Task` = instance of work (e.g., "Tuesday AM Feed for Horse X")
- `TaskTemplate` = recurring template at yard level
- `TaskCompletion` = historical record of who completed what when
- Allows full audit trail + completion history per horse

### 2. **Polymorphic Attachments**
- Single `Attachment` table, multiple parent entities
- Avoids duplication of file storage logic
- Supports photos on notes, vet reports on medical history, invoices on expenses

### 3. **Alert Level Tracking**
- `currentAlertLevel` on horse (GREEN/AMBER/RED)
- Computed programmatically based on:
  - Missed tasks
  - Overdue medications
  - Overdue vaccinations
  - No notes in 24h
- Dashboard queries filter by this flag

### 4. **Soft Delete Pattern**
- `isDraft` on Horse (for testing without deleting)
- Proper `onDelete: Cascade` for dependent records
- Clean data model

### 5. **Indexes**
- All foreign keys indexed
- Commonly filtered fields indexed (status, date, tag)
- Improves dashboard queries significantly

---

## Dashboard Query Examples

```sql
-- Horses with alerts today
SELECT h.* FROM Horse h
WHERE h.yardId = $yardId
  AND h.currentAlertLevel IN ('AMBER', 'RED')
ORDER BY h.currentAlertLevel DESC;

-- Task compliance for today
SELECT 
  COUNT(*) as total,
  COUNT(CASE WHEN status = 'COMPLETED' THEN 1 END) as completed
FROM Task t
WHERE t.yardId = $yardId
  AND DATE(t.dueDate) = CURRENT_DATE;

-- Horses on medication
SELECT DISTINCT h.* FROM Horse h
WHERE h.yardId = $yardId
  AND h.activeMedication IS NOT NULL;

-- Recent activity feed
SELECT n.*, h.name FROM Note n
JOIN Horse h ON n.horseId = h.id
WHERE n.yardId = $yardId
ORDER BY n.createdAt DESC
LIMIT 50;
```

