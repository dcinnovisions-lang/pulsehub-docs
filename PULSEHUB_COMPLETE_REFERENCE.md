# PulseHub — Complete Technical Reference
> A-to-Z guide covering architecture, feature status, coding conventions, data models, API contracts, and roadmap.
> Last updated: 2026-03-18

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Repository Structure](#3-repository-structure)
4. [Architecture Overview](#4-architecture-overview)
5. [Backend — Deep Dive](#5-backend--deep-dive)
   - 5.1 Server & App Bootstrap
   - 5.2 Database Models (32)
   - 5.3 Controllers & Endpoints
   - 5.4 Middleware
   - 5.5 RBAC V2 Permission System
   - 5.6 Real-time (Socket.io)
   - 5.7 Email (Nodemailer)
   - 5.8 Migrations
   - 5.9 Seed Scripts
6. [Frontend — Deep Dive](#6-frontend--deep-dive)
   - 6.1 App Bootstrap & Routing
   - 6.2 Pages (24)
   - 6.3 Components (35+)
   - 6.4 Redux Store & Slices (28)
   - 6.5 API Services (31)
   - 6.6 Styling Conventions
7. [Coding Style Guide](#7-coding-style-guide)
8. [Feature Status Matrix](#8-feature-status-matrix)
9. [What Is Fully Working](#9-what-is-fully-working)
10. [What Is Partially Built](#10-what-is-partially-built)
11. [What Is Missing / Not Started](#11-what-is-missing--not-started)
12. [API Reference Summary](#12-api-reference-summary)
13. [Data Flow Diagrams](#13-data-flow-diagrams)
14. [Environment & Configuration](#14-environment--configuration)
15. [Demo Accounts](#15-demo-accounts)
16. [Known Bugs & Tech Debt](#16-known-bugs--tech-debt)
17. [Roadmap — Prioritized](#17-roadmap--prioritized)

---

## 1. Project Overview

**PulseHub** is a full-stack, SaaS-style project management and team collaboration platform. It targets mid-size engineering and product teams who need a single place for planning, execution, and communication.

| Property | Value |
|---|---|
| Working name | PulseHub (backend package still says `trackpro-server`) |
| Stage | Feature-complete MVP — ~80% polished, ~20% rough edges |
| Users | Multi-tenant: many workspaces, each with projects, each with members |
| Auth | Email/password + Google OAuth + TOTP-based 2FA |
| Roles | 3-level RBAC (platform → workspace → project) |
| Real-time | Socket.io for chat + notification scaffolding |
| Deployment | Not yet deployed — local dev only |

---

## 2. Tech Stack

### Backend
| Layer | Technology | Version |
|---|---|---|
| Runtime | Node.js | 18+ |
| Framework | Express | ^4.18.2 |
| ORM | Sequelize | ^6.35.2 |
| Database | PostgreSQL | 14+ |
| Auth | jsonwebtoken + bcryptjs | ^9.0.2 / ^2.4.3 |
| 2FA | speakeasy + qrcode | ^2.0.0 / ^1.5.4 |
| OAuth | passport-google-oauth20 | ^2.0.0 |
| Real-time | socket.io | ^4.7.5 |
| Email | nodemailer | ^7.0.12 |
| File upload | multer | ^1.4.5-lts.1 |
| Logging | winston | ^3.11.0 |
| Validation | express-validator | ^7.0.1 |
| Rate limiting | express-rate-limit | ^7.1.5 |
| Security | helmet | ^7.1.0 |
| Excel | xlsx | ^0.18.5 |

### Frontend
| Layer | Technology | Version |
|---|---|---|
| UI library | React | 18.2.0 |
| Language | TypeScript | 4.9.5 |
| Build | Vite | 5.0.8 |
| Routing | react-router-dom | 6.21.0 |
| State | Redux Toolkit + react-redux | ^2.11.2 / ^9.2.0 |
| Persistence | redux-persist | 6.0.0 |
| Alt state | Zustand | 4.4.7 |
| Data fetching | Axios | 1.6.2 |
| Server state | @tanstack/react-query | 5.90.15 |
| Icons | react-icons (Feather) + lucide-react | 4.12.0 / 0.562.0 |
| Drag & drop | react-beautiful-dnd | 13.1.1 |
| Forms | react-hook-form | 7.49.2 |
| Dates | date-fns | 3.0.6 |
| WebSocket | socket.io-client | 4.7.5 |
| Animations | aos | 2.3.4 |

---

## 3. Repository Structure

```
d:/PulseHub/
├── backend/
│   ├── src/
│   │   ├── app.js                     # Express app (CORS, middleware, routes)
│   │   ├── server.js                  # HTTP + Socket.io server bootstrap
│   │   ├── socket.js                  # Socket.io event handlers
│   │   ├── config/
│   │   │   └── database.js            # Sequelize connection config
│   │   ├── controllers/               # 30 controller files
│   │   ├── models/                    # 32 Sequelize model files + index.js
│   │   ├── routes/                    # 28 route files + index.js
│   │   ├── middleware/
│   │   │   ├── auth.js                # JWT authenticate + authorize
│   │   │   ├── permissions.js         # RBAC V2 role matrix enforcement
│   │   │   ├── errorHandler.js
│   │   │   └── notFound.js
│   │   ├── utils/
│   │   │   ├── logger.js              # Winston logger
│   │   │   └── permissions.js
│   │   └── validators/
│   │       └── auth.validator.js
│   ├── migrations/                    # 12 Sequelize migration files
│   ├── scripts/                       # Seed and utility scripts
│   ├── .env                           # Environment variables
│   ├── .sequelizerc
│   └── package.json
│
├── frontend/
│   ├── public/
│   │   └── images/                    # Landing page section images
│   ├── src/
│   │   ├── App.tsx                    # Root component + all routes
│   │   ├── index.tsx                  # ReactDOM render + Providers
│   │   ├── pages/                     # 24 page components
│   │   ├── components/                # 35+ feature components
│   │   ├── store/
│   │   │   ├── index.ts               # configureStore + persistor
│   │   │   ├── hooks.ts               # useAppDispatch, useAppSelector
│   │   │   ├── authStore.ts           # Zustand store (auth alt)
│   │   │   └── slices/                # 28 Redux slice files
│   │   ├── services/                  # 31 Axios service files
│   │   ├── types/
│   │   │   └── index.ts               # All TypeScript interfaces
│   │   ├── styles/                    # Global and shared CSS
│   │   └── index.css                  # Global reset + tokens
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── package.json
│
└── Docs/
    ├── RBAC_ROLE_TESTING_GUIDE.md
    ├── SMTP_SETUP_GUIDE.md
    └── PULSEHUB_COMPLETE_REFERENCE.md  ← this file
```

---

## 4. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        BROWSER                              │
│  React 18 + Redux Toolkit + TypeScript                      │
│  ┌──────────┐  ┌─────────────┐  ┌───────────────────────┐  │
│  │  Pages   │  │  Components  │  │  Redux Store (28 slices│  │
│  │  (24)    │──│   (35+)     │──│  + redux-persist)     │  │
│  └──────────┘  └─────────────┘  └───────────────────────┘  │
│        │               │                    │               │
│        └───────────────┴────────────────────┘               │
│                         │                                   │
│              Axios (api.ts) + socket.io-client              │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/WS
┌──────────────────────────▼──────────────────────────────────┐
│                    NODE.JS / EXPRESS                         │
│  app.js ─ helmet ─ cors ─ morgan ─ rateLimit ─ routes       │
│                                                              │
│  Middleware chain:                                           │
│  authenticate() → authorize() → permissions.js → handler    │
│                                                              │
│  Controllers (30) → Models (32) → PostgreSQL                 │
│                                                              │
│  Socket.io server ← → socket.js (chat, notifications)       │
│                                                              │
│  Nodemailer → SMTP (invite emails, 2FA)                      │
└──────────────────────────────────────────────────────────────┘
```

### Request Lifecycle
1. Request hits Express router
2. `authenticate()` middleware verifies Bearer JWT, attaches `req.user`
3. `authorize(...roles)` checks `req.user.role` against allowed platform roles
4. Route-specific `permissions.js` guard checks workspace/project role
5. Controller executes business logic
6. Sequelize ORM queries PostgreSQL
7. Controller sends JSON response
8. On data-mutating operations, `createNotification()` is called (non-fatal, wrapped in try/catch)
9. On create/update of chat messages, Socket.io broadcasts to room

---

## 5. Backend — Deep Dive

### 5.1 Server & App Bootstrap

**`server.js`**
- Creates HTTP server from Express app
- Attaches Socket.io with CORS config
- Calls `sequelize.authenticate()` then `sequelize.sync({ alter: false })`
- Listens on `PORT` (default 5000)
- Imports `socket.js` for event registration

**`app.js`**
- `helmet()` — security headers
- `compression()` — gzip responses
- `cors({ origin: 'http://localhost:3000', credentials: true })`
- `morgan('dev')` — request logging
- `express-rate-limit` — 100 req/15 min (production)
- `express.json({ limit: '10mb' })`
- `passport.initialize()` — Google OAuth
- Mounts all routes under `/api/v1`
- `errorHandler` and `notFound` middleware at the bottom

---

### 5.2 Database Models (32)

#### Core Identity
```
User
  id          UUID (PK)
  email       STRING UNIQUE NOT NULL
  password    STRING (bcrypt hashed)
  firstName   STRING
  lastName    STRING
  role        ENUM: super_admin | admin | owner | billing_admin | pm | member | commenter | guest | viewer
  avatar      STRING (file path)
  isActive    BOOLEAN default true
  twoFactorEnabled  BOOLEAN default false
  twoFactorSecret   STRING (encrypted TOTP secret)
  googleId    STRING (OAuth)
  lastLogin   DATE
```

#### Workspace Layer
```
Workspace
  id          UUID (PK)
  name        STRING NOT NULL
  description TEXT
  ownerId     UUID → User
  status      ENUM: active | archived

WorkspaceMembers (junction)
  userId      UUID → User
  workspaceId UUID → Workspace
  role        ENUM: owner | admin | pm | member | commenter | viewer | guest
  joinedAt    DATE
```

#### Project Layer
```
Project
  id          UUID (PK)
  name        STRING NOT NULL
  description TEXT
  workspaceId UUID → Workspace
  status      ENUM: active | archived | completed
  color       STRING (hex)
  isTemplate  BOOLEAN default false
  templateId  UUID → Project (null if not from template)
  ownerId     UUID → User

ProjectMembers (junction)
  userId      UUID → User
  projectId   UUID → Project
  role        ENUM: project_lead | contributor | reporter | reviewer | commenter | viewer
```

#### Task Layer
```
Task
  id              UUID (PK)
  title           STRING NOT NULL
  description     TEXT
  projectId       UUID → Project
  listId          UUID → List
  statusId        UUID → Status
  priority        ENUM: urgent | high | medium | low
  dueDate         DATE
  startDate       DATE
  estimatedHours  FLOAT
  position        INTEGER (for ordering)
  createdBy       UUID → User
  isArchived      BOOLEAN default false
  progress        INTEGER 0-100

TaskAssignees (junction)
  taskId      UUID → Task
  userId      UUID → User

Subtask
  id              UUID (PK)
  taskId          UUID → Task
  title           STRING NOT NULL
  isCompleted     BOOLEAN default false
  assigneeId      UUID → User (nullable)
  dueDate         DATE
  startDate       DATE
  estimatedHours  FLOAT

TaskDependency
  id              UUID (PK)
  taskId          UUID → Task
  dependsOnTaskId UUID → Task
  type            ENUM: blocks | blocked_by | relates_to

TaskCustomField
  id            UUID (PK)
  taskId        UUID → Task
  customFieldId UUID → CustomField
  value         TEXT
```

#### Project Metadata
```
Status
  id        UUID (PK)
  name      STRING NOT NULL
  projectId UUID → Project
  color     STRING (hex)
  position  INTEGER
  isDefault BOOLEAN

List
  id        UUID (PK)
  name      STRING NOT NULL
  projectId UUID → Project
  position  INTEGER

CustomField
  id        UUID (PK)
  name      STRING NOT NULL
  projectId UUID → Project
  type      ENUM: text | number | select | date | checkbox
  options   JSON (for select type)
```

#### Collaboration
```
Comment
  id        UUID (PK)
  content   TEXT NOT NULL
  taskId    UUID → Task
  createdBy UUID → User
  reactions JSON (map of emoji → [userId])
  updatedAt DATE

Attachment
  id         UUID (PK)
  fileName   STRING
  filePath   STRING
  fileSize   INTEGER
  mimeType   STRING
  taskId     UUID → Task
  uploadedBy UUID → User
```

#### Communication
```
ChatRoom
  id        UUID (PK)
  name      STRING NOT NULL
  projectId UUID → Project
  createdBy UUID → User

ChatMessage
  id        UUID (PK)
  roomId    UUID → ChatRoom
  userId    UUID → User
  content   TEXT
  createdAt DATE

Document
  id          UUID (PK)
  title       STRING NOT NULL
  content     TEXT (rich text/HTML)
  workspaceId UUID → Workspace (nullable)
  projectId   UUID → Project (nullable)
  createdBy   UUID → User
  updatedAt   DATE

Whiteboard
  id        UUID (PK)
  title     STRING NOT NULL
  projectId UUID → Project
  createdBy UUID → User

WhiteboardElement
  id            UUID (PK)
  whiteboardId  UUID → Whiteboard
  type          STRING (rect | circle | text | arrow)
  data          JSON
  position      JSON {x, y}
```

#### Notifications & Audit
```
Notification
  id          UUID (PK)
  recipientId UUID → User
  type        ENUM: task_assigned | task_comment | project_member_added |
                    project_member_removed | workspace_member_added |
                    workspace_member_removed | invite_accepted
  title       STRING
  body        TEXT
  entityType  STRING (task | project | workspace)
  entityId    UUID
  actorId     UUID → User
  isRead      BOOLEAN default false
  metadata    JSON (url, projectId, etc.)
  createdAt   DATE

ActivityLog
  id          UUID (PK)
  entityType  STRING
  entityId    UUID
  action      STRING (created | updated | deleted | etc.)
  userId      UUID → User
  changes     JSON (before/after)
  metadata    JSON
  createdAt   DATE
```

#### Project Analytics
```
Budget
  id        UUID (PK)
  name      STRING NOT NULL
  projectId UUID → Project
  amount    FLOAT (total budget)
  spent     FLOAT (calculated)
  status    ENUM: active | exceeded | completed

BudgetExpense
  id          UUID (PK)
  budgetId    UUID → Budget
  description STRING
  amount      FLOAT
  date        DATE

TimeTracking
  id          UUID (PK)
  taskId      UUID → Task
  userId      UUID → User
  hours       FLOAT
  date        DATE
  description TEXT

TimeLog
  id        UUID (PK)
  taskId    UUID → Task
  userId    UUID → User
  startTime DATE
  endTime   DATE
  duration  INTEGER (minutes)

Resource
  id         UUID (PK)
  userId     UUID → User
  projectId  UUID → Project
  allocation INTEGER (percentage)
  startDate  DATE
  endDate    DATE
```

#### Access Control
```
Invite
  id          UUID (PK)
  email       STRING NOT NULL
  workspaceId UUID → Workspace (nullable)
  projectId   UUID → Project (nullable)
  role        STRING
  token       STRING UNIQUE (UUID, used in accept link)
  invitedBy   UUID → User
  status      ENUM: pending | accepted | cancelled | expired
  expiresAt   DATE (7 days from creation)

GuestAccess
  id          UUID (PK)
  guestId     UUID → User
  workspaceId UUID → Workspace
  projectId   UUID → Project
  permissions JSON
```

#### Views & Workflows
```
SavedView
  id        UUID (PK)
  name      STRING NOT NULL
  projectId UUID → Project
  filters   JSON (status, priority, assignee, etc.)
  sortBy    JSON
  createdBy UUID → User

Workflow
  id          UUID (PK)
  name        STRING NOT NULL
  projectId   UUID → Project
  isTemplate  BOOLEAN default false
  stages      JSON (ordered list of stage definitions)
```

---

### 5.3 Controllers & Endpoints

All routes are prefixed with `/api/v1`.

#### Auth — `auth.routes.js`
| Method | Path | Controller | Auth |
|--------|------|-----------|------|
| POST | /auth/register | `register` | Public |
| POST | /auth/login | `login` | Public |
| POST | /auth/refresh | `refreshToken` | Public |
| POST | /auth/logout | `logout` | Bearer |
| GET  | /auth/me | `getMe` | Bearer |
| POST | /auth/2fa/setup | `setup2FA` | Bearer |
| POST | /auth/2fa/verify | `verify2FA` | Bearer |
| POST | /auth/2fa/disable | `disable2FA` | Bearer |
| POST | /auth/2fa/login | `verify2FALogin` | Public |
| GET  | /auth/google | `googleAuth` | Public |
| GET  | /auth/google/callback | `googleCallback` | Public |

#### Users — `user.routes.js`
| Method | Path | Controller | Auth |
|--------|------|-----------|------|
| GET | /users | `getAllUsers` | admin+ |
| GET | /users/:id | `getUserById` | Bearer |
| PUT | /users/:id | `updateUser` | Bearer (own or admin) |
| DELETE | /users/:id | `deleteUser` | admin+ |
| PATCH | /users/:id/role | `updateUserRole` | super_admin |

#### Workspaces — `workspace.routes.js`
| Method | Path | Controller |
|--------|------|-----------|
| GET | /workspaces | `getWorkspaces` |
| POST | /workspaces | `createWorkspace` |
| GET | /workspaces/:id | `getWorkspaceById` |
| PUT | /workspaces/:id | `updateWorkspace` |
| DELETE | /workspaces/:id | `deleteWorkspace` |
| GET | /workspaces/:id/members | `getWorkspaceMembers` |
| POST | /workspaces/:id/members | `addWorkspaceMember` |
| PUT | /workspaces/:id/members/:userId | `updateWorkspaceMemberRole` |
| DELETE | /workspaces/:id/members/:userId | `removeWorkspaceMember` |

#### Projects — `project.routes.js`
| Method | Path | Controller |
|--------|------|-----------|
| GET | /projects | `getProjects` |
| POST | /projects | `createProject` |
| GET | /projects/:id | `getProjectById` |
| PUT | /projects/:id | `updateProject` |
| DELETE | /projects/:id | `deleteProject` |
| GET | /projects/templates | `getTemplates` |
| POST | /projects/from-template | `createProjectFromTemplate` |

#### Tasks — `task.routes.js`
| Method | Path | Controller |
|--------|------|-----------|
| GET | /tasks | `getTasks` (query: projectId, listId, statusId, assigneeId) |
| POST | /tasks | `createTask` |
| GET | /tasks/:id | `getTaskById` |
| PUT | /tasks/:id | `updateTask` |
| DELETE | /tasks/:id | `deleteTask` |
| POST | /tasks/bulk | `bulkCreateTasks` |
| PUT | /tasks/bulk | `bulkUpdateTasks` |
| DELETE | /tasks/bulk | `bulkDeleteTasks` |
| POST | /tasks/bulk/archive | `bulkArchiveTasks` |
| GET | /tasks/export | `exportTasksToCSV` |
| POST | /tasks/import | `importTasksFromCSV` |
| POST | /tasks/reorder | `reorderTasks` |
| POST | /tasks/:id/subtasks | `createSubtask` |
| PUT | /tasks/:id/subtasks/:sid | `updateSubtask` |
| DELETE | /tasks/:id/subtasks/:sid | `deleteSubtask` |

#### Project Members — `projectMembers` embedded in project routes
| Method | Path | Controller |
|--------|------|-----------|
| GET | /projects/:id/members | `getProjectMembers` |
| POST | /projects/:id/members | `addProjectMember` |
| PUT | /projects/:id/members/:userId | `updateProjectMemberRole` |
| DELETE | /projects/:id/members/:userId | `removeProjectMember` |

#### Notifications — `notification.routes.js`
| Method | Path | Controller |
|--------|------|-----------|
| GET | /notifications | `getNotifications` |
| GET | /notifications/unread-count | `getUnreadCount` |
| PUT | /notifications/:id/read | `markAsRead` |
| PUT | /notifications/read-all | `markAllAsRead` |
| DELETE | /notifications/:id | `deleteNotification` |
| DELETE | /notifications/clear-read | `clearReadNotifications` |

#### Invites — `invite.routes.js`
| Method | Path | Controller |
|--------|------|-----------|
| POST | /invites | `sendInvite` |
| GET | /invites/:token | `getInviteByToken` |
| POST | /invites/:token/accept | `acceptInvite` |
| GET | /invites | `listInvites` |
| DELETE | /invites/:id | `cancelInvite` |

#### Search — `search.routes.js`
| Method | Path | Controller |
|--------|------|-----------|
| GET | /search?q=&types= | `globalSearch` |

Search scopes: tasks, projects, workspaces, users, comments, documents

#### Remaining Feature Endpoints (all authenticated, Bearer required)

| Route file | Endpoints |
|-----------|-----------|
| `status.routes.js` | CRUD for project statuses (5 endpoints) |
| `list.routes.js` | CRUD for lists/boards (5 endpoints) |
| `customField.routes.js` | CRUD + task field values (6 endpoints) |
| `taskDependency.routes.js` | Add/remove dependencies (3 endpoints) |
| `comment.routes.js` | CRUD + reactions (6 endpoints) |
| `attachment.routes.js` | Upload/download/delete (4 endpoints) |
| `activityLog.routes.js` | Get logs by entity (2 endpoints) |
| `calendar.routes.js` | Get calendar data for workspace (1 endpoint) |
| `gantt.routes.js` | Gantt chart data (2 endpoints) |
| `kanban.routes.js` | Board + move task (2 endpoints) |
| `budget.routes.js` | Budget + expenses (4 endpoints) |
| `resource.routes.js` | Workload + upsert (2 endpoints) |
| `timeTracking.routes.js` | Log + get time (5 endpoints) |
| `savedView.routes.js` | CRUD saved views (5 endpoints) |
| `workflow.routes.js` | CRUD workflows (7 endpoints) |
| `document.routes.js` | CRUD documents (5 endpoints) |
| `whiteboard.routes.js` | CRUD whiteboards + elements (5 endpoints) |
| `chat.routes.js` | Rooms + messages (4 endpoints) |
| `permission.routes.js` | Get permission matrix (1 endpoint) |
| `dashboard.routes.js` | Stakeholder dashboard (1 endpoint) |

---

### 5.4 Middleware

#### `auth.js`
```javascript
// authenticate() — extracts Bearer token, verifies JWT, attaches req.user
// authorize(...roles) — checks req.user.role against allowed platform roles
// Example: router.delete('/:id', authenticate, authorize('admin','super_admin'), handler)
```

#### `permissions.js`
Implements RBAC V2 — checks workspace-level and project-level roles via DB lookup. Called after `authenticate`. Guards are defined per route:
- `requireWorkspaceRole(minRole)` — checks WorkspaceMembers.role
- `requireProjectRole(minRole)` — checks ProjectMembers.role
- Roles are ordered; min role means "this role or above"

#### `errorHandler.js`
- Catches all errors thrown by controllers
- Returns standardized `{ error, message, stack? }` JSON
- 4xx for validation/permission errors, 5xx for server errors

---

### 5.5 RBAC V2 Permission System

Three independent role axes:

```
Platform Role (User.role)
  super_admin > admin > owner > billing_admin > pm > member > commenter > guest > viewer

Workspace Role (WorkspaceMembers.role)
  owner > admin > pm > member > commenter > viewer > guest

Project Role (ProjectMembers.role)
  project_lead > contributor > reporter > reviewer > commenter > viewer
```

**Permission Matrix (key rules):**
| Action | Min Platform | Min Workspace | Min Project |
|--------|-------------|---------------|-------------|
| Create workspace | member | — | — |
| Delete workspace | admin | owner | — |
| Add workspace member | — | admin | — |
| Create project | — | pm | — |
| Delete project | — | admin | project_lead |
| Create task | — | member | contributor |
| Edit any task | — | pm | project_lead |
| Comment on task | — | member | commenter |
| View-only | — | viewer | viewer |
| Access billing | billing_admin | — | — |
| View audit log | admin | — | — |
| Manage 2FA | — (own) | — | — |

---

### 5.6 Real-time (Socket.io)

**`socket.js`** handles:
- `connection` — associates socket with `userId` (from handshake auth)
- `chat:join_room` — joins a project chat room
- `chat:send_message` — persists ChatMessage, broadcasts to room
- `chat:leave_room` — leaves room

**NOT YET WIRED (gap):**
- `notification:new` — server emits this when `createNotification()` is called, but the frontend does not listen for it. Frontend only polls every 60 seconds.

---

### 5.7 Email (Nodemailer)

Used for:
- Workspace/project invite emails (sends link with token)
- 2FA setup confirmation (optional)

Config loaded from `.env`: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM`.

---

### 5.8 Migrations (12 files, in order)

| File | Purpose |
|------|---------|
| `20251230134429-create-initial-tables.js` | All core tables (Users, Workspaces, Projects, Tasks, etc.) |
| `20251231094800-add-assignee-id-to-subtasks.js` | assigneeId on Subtask |
| `20251231120000-create-invites-table.js` | Invite model |
| `20250101000000-add-super-admin-role.js` | ENUM expansion for User.role |
| `20250101000001-create-activity-logs.js` | ActivityLog table |
| `20250101000002-create-resources-budgets.js` | Resource + Budget + BudgetExpense |
| `20250106000001-add-subtask-dates-estimates.js` | startDate + estimatedHours on Subtask |
| `20250106000002-create-saved-views.js` | SavedView table |
| `20250106000003-add-is-template-to-workflows.js` | isTemplate flag on Workflow |
| `20260107000010-add-guest-commenter-role.js` | Expand role ENUMs |
| `20260318000001-rbac-v2.js` | ProjectMembers.role ENUM expansion |
| `20260318000002-create-notifications.js` | Notification table |

Run: `cd backend && npm run db:migrate`

---

### 5.9 Seed Scripts

| Script | Command | Purpose |
|--------|---------|---------|
| `scripts/seed-demo.js` | `npm run seed:demo` | Creates 9 demo accounts (all platform roles), 2 workspaces, 3 projects, members, statuses, lists, 5 tasks |
| `scripts/create-super-admin.js` | `node scripts/create-super-admin.js` | One-off super admin creation |
| `scripts/create-test-users.js` | `node scripts/create-test-users.js` | Batch test user creation |
| `scripts/test-smtp.js` | `npm run test:smtp` | Verify SMTP config works |

---

## 6. Frontend — Deep Dive

### 6.1 App Bootstrap & Routing

**`index.tsx`**
```tsx
<Provider store={store}>
  <PersistGate loading={null} persistor={persistor}>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </PersistGate>
</Provider>
```

**`App.tsx`** — all routes:
```
/                    → LandingPage (via LandingRoute — redirects to /dashboard if logged in)
/login               → LoginPage (PublicRoute)
/register            → RegisterPage (PublicRoute)
/oauth/callback      → OAuthCallbackPage
/invite/:token       → AcceptInvitePage
/dashboard           → DashboardPage (PrivateRoute)
/workspaces          → WorkspacesPage (PrivateRoute)
/projects            → ProjectsPage (PrivateRoute)
/projects/:id        → ProjectDetailPage (PrivateRoute)
/projects/:id/kanban → KanbanBoardPage (PrivateRoute)
/projects/:id/gantt  → GanttPage (PrivateRoute)
/tasks               → TasksPage (PrivateRoute)
/tasks/:id           → TaskDetailPage (PrivateRoute)
/calendar            → CalendarPage (PrivateRoute)
/budgets             → BudgetManagementPage (PrivateRoute)
/resources           → ResourceManagementPage (PrivateRoute)
/time-tracking       → TimeTrackingPage (PrivateRoute)
/documents           → DocumentsPage (PrivateRoute)
/chat                → ChatPage (PrivateRoute)
/whiteboard          → WhiteboardPage (PrivateRoute)
/workload            → WorkloadViewPage (PrivateRoute)
/users               → UsersPage (PrivateRoute — admin only)
/settings            → SettingsPage (PrivateRoute)
/dashboard/stakeholder → StakeholderDashboardPage (PrivateRoute)
```

**Route guards:**
- `PrivateRoute` — checks `authSlice.isAuthenticated`, redirects to `/login`
- `PublicRoute` — redirects authenticated users to `/dashboard`
- `LandingRoute` — redirects authenticated users away from landing page

---

### 6.2 Pages (24)

| Page | Location | Status | Notes |
|------|----------|--------|-------|
| LandingPage | `pages/LandingPage.tsx` | ✅ Complete | Full redesign with `.lp-*` CSS |
| LoginPage | `pages/auth/LoginPage.tsx` | ✅ Complete | Email/pass + Google OAuth |
| RegisterPage | `pages/auth/RegisterPage.tsx` | ✅ Complete | |
| OAuthCallbackPage | `pages/auth/OAuthCallbackPage.tsx` | ✅ Complete | |
| AcceptInvitePage | `pages/invite/AcceptInvitePage.tsx` | ✅ Complete | Token-based invite flow |
| DashboardPage | `pages/DashboardPage.tsx` | ✅ Complete | Overview stats + recent activity |
| WorkspacesPage | `pages/workspaces/WorkspacesPage.tsx` | ✅ Complete | Create/list workspaces |
| ProjectsPage | `pages/projects/ProjectsPage.tsx` | ✅ Complete | Grid view + template modal w/ gallery |
| ProjectDetailPage | `pages/projects/ProjectDetailPage.tsx` | ✅ Complete | Task list, members, settings |
| KanbanBoardPage | `pages/tasks/KanbanBoardPage.tsx` | ✅ Complete | Drag-and-drop board |
| GanttPage | `pages/gantt/GanttPage.tsx` | ⚠️ Partial | UI rendered, some data gaps |
| TasksPage | `pages/tasks/TasksPage.tsx` | ✅ Complete | List view with filters |
| TaskDetailPage | `pages/tasks/TaskDetailPage.tsx` | ✅ Complete | Full task view, comments, attachments |
| CalendarPage | `pages/calendar/CalendarPage.tsx` | ⚠️ Partial | Shows tasks by due date |
| BudgetManagementPage | `pages/budgets/BudgetManagementPage.tsx` | ⚠️ Partial | Budget CRUD, no charts |
| ResourceManagementPage | `pages/resources/ResourceManagementPage.tsx` | ⚠️ Partial | Allocation table |
| TimeTrackingPage | `pages/timeTracking/TimeTrackingPage.tsx` | ⚠️ Partial | Manual log + list view |
| DocumentsPage | `pages/documents/DocumentsPage.tsx` | ⚠️ Partial | Plain textarea editor |
| ChatPage | `pages/chat/ChatPage.tsx` | ⚠️ Partial | Room list + messages (no typing indicators) |
| WhiteboardPage | `pages/whiteboard/WhiteboardPage.tsx` | ⚠️ Partial | Basic canvas only |
| WorkloadViewPage | `pages/workload/WorkloadViewPage.tsx` | ⚠️ Partial | Table view, no chart |
| UsersPage | `pages/users/UsersPage.tsx` | ✅ Complete | Admin user management |
| SettingsPage | `pages/settings/SettingsPage.tsx` | ✅ Complete | 4 tabs: Security, Permissions, Audit, Templates |
| StakeholderDashboardPage | `pages/dashboard/StakeholderDashboardPage.tsx` | ⚠️ Partial | KPI cards only |

---

### 6.3 Components (35+)

#### Layout
| Component | Purpose |
|-----------|---------|
| `Layout.tsx` | Shell with Sidebar + Header + main content area |
| `Header.tsx` | Top bar: GlobalSearch, NotificationBell, user menu |
| `Sidebar.tsx` | Navigation: workspaces, projects, views |
| `RoleBasedSidebar.tsx` | Sidebar that hides items based on user role |

#### Common / Shared
| Component | Purpose |
|-----------|---------|
| `AssigneeSelector.tsx` | Searchable user picker with avatars |
| `StatusSelector.tsx` | Status dropdown per project |
| `UserSelector.tsx` | Generic user search/select |
| `Toast.tsx` / `ToastContainer.tsx` | In-app toast notification system |
| `PermissionGuard.tsx` | Conditional render based on role |
| `ConfirmReasonModal.tsx` | Confirmation dialog with reason input |
| `AppBar.tsx` | Secondary navigation bar (page-level) |

#### Auth
| Component | Purpose |
|-----------|---------|
| `PrivateRoute.tsx` | Route-level auth guard |
| `PublicRoute.tsx` | Redirect authenticated users |
| `LandingRoute.tsx` | Landing-specific redirect |

#### Feature Components
| Component | Purpose | Status |
|-----------|---------|--------|
| `NotificationBell.tsx` | Badge + dropdown with notifications | ✅ Complete |
| `GlobalSearch.tsx` | Cmd+K modal, searches all entities | ✅ Complete |
| `TwoFactorSetup.tsx` | QR code display, TOTP verify, disable | ✅ Complete |
| `PermissionMatrix.tsx` | Visual role-permission table | ✅ Complete |
| `WorkspaceMembersManager.tsx` | Member list + invite + role change | ✅ Complete |
| `ProjectMembersManager.tsx` | Project member CRUD | ✅ Complete |
| `ProjectTemplateGallery.tsx` | Visual template card picker | ✅ Complete |
| `ActivityTimeline.tsx` | Feed of ActivityLog events | ✅ Complete |
| `ProjectHealthScore.tsx` | Score ring + breakdown | ✅ Complete |
| `CommentList.tsx` | Threaded comments + reactions | ✅ Complete |
| `AttachmentList.tsx` | File upload/download list | ✅ Complete |
| `CustomFieldConfig.tsx` | Define project custom fields | ✅ Complete |
| `CustomFieldInput.tsx` | Render input for a custom field type | ✅ Complete |
| `TaskDependencyManager.tsx` | Add/view task dependencies | ✅ Complete |
| `TimeTracker.tsx` | Start/stop timer widget | ⚠️ Partial |
| `ResourceManagementDashboard.tsx` | Allocation overview | ⚠️ Partial |
| `TimeTrackingDashboard.tsx` | Hours summary | ⚠️ Partial |

---

### 6.4 Redux Store & Slices (28)

**Store configuration** (`store/index.ts`):
```typescript
configureStore({
  reducer: {
    auth, workspace, project, task, subtask,
    status, list, customField, taskDependency,
    comment, attachment, invite, notification,
    permission, user, projectHealth, kanban,
    activityLog, calendar, budget, gantt,
    document, whiteboard, chat, timeTracking,
    resource, savedView
  },
  middleware: getDefaultMiddleware({ serializableCheck: { ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER] } })
})
```

Redux Persist config: persists `auth` slice only (tokens + user).

#### Key Slice Shapes

**`authSlice`**
```typescript
{
  user: User | null,
  token: string | null,
  refreshToken: string | null,
  isAuthenticated: boolean,
  isLoading: boolean,
  error: string | null,
  twoFactorRequired: boolean,
  twoFactorQRCode: string | null,
}
```
Actions: `login`, `register`, `logout`, `refreshToken`, `getMe`, `setup2FA`, `verify2FA`, `disable2FA`

**`notificationSlice`**
```typescript
{
  notifications: Notification[],
  unreadCount: number,
  isLoading: boolean,
  error: string | null,
}
```
Actions: `fetchNotifications`, `fetchUnreadCount`, `markAsRead`, `markAllAsRead`, `deleteNotification`, `addNotification` (for real-time push — reducer ready, Socket.io not wired on frontend)

**`taskSlice`**
```typescript
{
  tasks: Task[],
  currentTask: Task | null,
  isLoading: boolean,
  error: string | null,
  filters: { statusId?, assigneeId?, priority?, search? }
}
```
Actions: `fetchTasks`, `fetchTaskById`, `createTask`, `updateTask`, `deleteTask`, `bulkUpdate`, `bulkDelete`, `reorderTasks`, `importTasks`, `exportTasks`

**`projectSlice`**
```typescript
{
  projects: Project[],
  currentProject: Project | null,
  templates: Project[],
  isLoading: boolean,
  error: string | null,
}
```
Actions: `fetchProjects`, `fetchProjectById`, `createProject`, `updateProject`, `deleteProject`, `fetchTemplates`, `createFromTemplate`

**`workspaceSlice`**
```typescript
{
  workspaces: Workspace[],
  currentWorkspace: Workspace | null,
  members: WorkspaceMember[],
  isLoading: boolean,
}
```

---

### 6.5 API Services (31)

All services use a shared `api.ts` Axios instance:
```typescript
// api.ts
const api = axios.create({ baseURL: 'http://localhost:5000/api/v1', withCredentials: true })

// Request interceptor: attaches Bearer token from Redux store
// Response interceptor: on 401 → tries refreshToken → retries request → on fail logs out
```

Each service file exports functions that call `api.get/post/put/delete`:
```typescript
// Example: taskService.ts
export const getTasks = (params) => api.get('/tasks', { params })
export const createTask = (data) => api.post('/tasks', data)
export const updateTask = (id, data) => api.put(`/tasks/${id}`, data)
// ...
```

**Note:** There are duplicate service/slice files for two features:
- `timeTracking.service.ts` + `timeTrackingService.ts`
- `resource.service.ts` + `resourceService.ts`
- `timeTracking.slice.ts` + `timeTrackingSlice.ts`
- `resource.slice.ts` + `resourceSlice.ts`

These are **tech debt** — only one of each pair is imported into the store. The underscore-named files are likely dead code.

---

### 6.6 Styling Conventions

PulseHub uses **plain CSS** — no CSS-in-JS, no Tailwind.

**File organization:**
- Each component has a co-located `.css` file: `ComponentName.tsx` + `ComponentName.css`
- Page-level CSS is in the same folder as the page
- Global tokens are in `index.css`

**Design tokens (from `index.css` and `LandingPage.css`):**
```css
/* Colors */
--primary: #2563eb         /* Blue */
--primary-dark: #1d4ed8
--secondary: #7c3aed       /* Purple */
--success: #10b981         /* Green */
--warning: #f59e0b         /* Amber */
--danger: #ef4444          /* Red */
--text-primary: #0f172a
--text-secondary: #475569
--text-muted: #94a3b8
--border: #e2e8f0
--bg-white: #ffffff
--bg-subtle: #f8fafc
--bg-dark: #0f172a

/* Spacing (common values used) */
4px, 8px, 12px, 16px, 20px, 24px, 32px, 40px, 48px, 64px, 96px

/* Radius */
4px (small), 8px (button/input), 12px (card), 16px (large card), 999px (pill)

/* Shadows */
0 2px 8px rgba(0,0,0,0.06)        /* subtle */
0 4px 16px rgba(0,0,0,0.1)        /* card */
0 8px 32px rgba(0,0,0,0.12)       /* elevated */
0 4px 14px rgba(37,99,235,0.35)   /* primary button glow */
```

**CSS class naming pattern:**
- Feature prefix: `.landing-*`, `.lp-*`, `.ws-*`, `.project-*`, `.task-*`
- BEM-style for new components: `.block__element--modifier`
- No CSS class names starting with a digit (avoid `.2fa-*` — use `.tfa-*`)

---

## 7. Coding Style Guide

### Backend (JavaScript / Node.js)

**File organization:**
- One controller function per exported name
- Controllers are "fat" — business logic lives there, not in routes
- Models use Sequelize associations defined in each model file, called from `models/index.js`

**Async pattern:** All controller functions are `async/await`:
```javascript
exports.createTask = async (req, res) => {
  try {
    const { title, projectId, ...rest } = req.body;
    const task = await Task.create({ title, projectId, createdBy: req.user.id, ...rest });
    // ... associate assignees ...
    // Notification (non-fatal)
    try {
      await Promise.allSettled(assigneeIds.map(uid => createNotification({ ... })));
    } catch (notifErr) {
      logger.warn('Notification failed:', notifErr.message);
    }
    res.status(201).json({ success: true, task });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

**Error responses:**
- `400` — validation error: `{ error: 'Validation failed', details: [...] }`
- `401` — unauthenticated: `{ error: 'Unauthorized' }`
- `403` — insufficient role: `{ error: 'Forbidden' }`
- `404` — not found: `{ error: 'Resource not found' }`
- `500` — server error: `{ error: error.message }`

**Notification pattern (all triggers follow this):**
```javascript
// Always non-fatal — never let notification failure break the main response
await Promise.allSettled(
  recipientIds.map(uid => createNotification({
    userId: uid,
    type: 'task_assigned',
    title: `You were assigned to "${task.title}"`,
    body: `${actorName} assigned you to this task`,
    entityType: 'task', entityId: task.id, actorId: req.user.id,
    metadata: { url: `/app/projects/${task.projectId}/tasks/${task.id}` }
  }))
);
```

### Frontend (TypeScript / React)

**Component pattern:**
```tsx
// Functional components only — no class components
const MyComponent: React.FC<Props> = ({ prop1, prop2 }) => {
  const dispatch = useAppDispatch();
  const data = useAppSelector(state => state.slice.data);

  useEffect(() => { /* side effects */ }, [dep]);

  const handleAction = async () => {
    try {
      await dispatch(someThunk(payload)).unwrap();
      // success
    } catch (err) {
      // error
    }
  };

  return <div className="component-root">...</div>;
};

export default MyComponent;
```

**Redux Thunk pattern:**
```typescript
// In slice file
export const fetchItems = createAsyncThunk(
  'slice/fetchItems',
  async (params, { rejectWithValue }) => {
    try {
      const response = await itemService.getItems(params);
      return response.data;
    } catch (err: any) {
      return rejectWithValue(err.response?.data?.error || 'Failed');
    }
  }
);

// Builder pattern in createSlice
builder
  .addCase(fetchItems.pending, state => { state.isLoading = true; state.error = null; })
  .addCase(fetchItems.fulfilled, (state, action) => { state.isLoading = false; state.items = action.payload; })
  .addCase(fetchItems.rejected, (state, action) => { state.isLoading = false; state.error = action.payload as string; });
```

**TypeScript conventions:**
- All component props typed as interfaces (not `type`)
- Redux state typed in each slice via `interface SliceState`
- Service functions return `Promise<AxiosResponse<T>>`
- Use `as` casts only when type narrowing is proven safe (e.g., `selectedRole as WorkspaceInviteRole`)
- Avoid `any` — use `unknown` + type guards instead

**Icon usage:**
```tsx
// Always import from react-icons/fi (Feather Icons)
import { FiPlus, FiTrash2, FiEdit } from 'react-icons/fi';
// Size via prop: <FiPlus size={16} />
// Or via className: <FiPlus className="icon-sm" />
```

---

## 8. Feature Status Matrix

| Feature | Backend | Frontend | Notes |
|---------|---------|----------|-------|
| **Auth — Email/password** | ✅ | ✅ | JWT + refresh token |
| **Auth — Google OAuth** | ✅ | ✅ | Passport.js |
| **Auth — 2FA (TOTP)** | ✅ | ✅ | QR code setup, verify, disable |
| **Workspace CRUD** | ✅ | ✅ | |
| **Workspace Members** | ✅ | ✅ | Invite + role management |
| **Project CRUD** | ✅ | ✅ | |
| **Project Templates** | ✅ | ✅ | Visual gallery picker |
| **Project Members (RBAC V2)** | ✅ | ✅ | 6 project roles |
| **Permission Matrix UI** | ✅ | ✅ | Read-only table |
| **Task CRUD** | ✅ | ✅ | Full feature set |
| **Subtasks** | ✅ | ✅ | |
| **Task Dependencies** | ✅ | ✅ | blocks/blocked_by/relates_to |
| **Task Assignees** | ✅ | ✅ | Multi-assignee |
| **Task Priority** | ✅ | ✅ | urgent/high/medium/low |
| **Task Custom Fields** | ✅ | ✅ | text/number/select/date/checkbox |
| **Task Status** | ✅ | ✅ | Per-project custom statuses |
| **Task Lists/Boards** | ✅ | ✅ | |
| **Bulk Task Operations** | ✅ | ✅ | Create/update/delete/archive |
| **CSV Import/Export** | ✅ | ✅ | |
| **Kanban Board** | ✅ | ✅ | Drag-and-drop |
| **Gantt Chart** | ✅ | ⚠️ | UI exists, data binding partial |
| **Calendar View** | ✅ | ⚠️ | Displays due dates, no drag |
| **Comments + Reactions** | ✅ | ✅ | Emoji reactions |
| **File Attachments** | ✅ | ✅ | Upload/download/delete |
| **Activity Log (backend)** | ✅ | ✅ | Surfaces in Settings > Audit tab |
| **Global Search** | ✅ | ✅ | Cmd+K, multi-entity |
| **Notifications (backend)** | ✅ | ✅ | All 7 triggers wired |
| **Notifications (real-time)** | ⚠️ | ❌ | Server emits but frontend doesn't listen |
| **Notifications (polling)** | ✅ | ✅ | 60s interval fallback |
| **Invite System** | ✅ | ✅ | Email invite + accept page |
| **Chat** | ✅ | ⚠️ | Backend + Socket.io done, UI basic |
| **Documents** | ✅ | ⚠️ | Textarea editor, no rich text |
| **Whiteboard** | ✅ | ⚠️ | Basic canvas, no real-time sync |
| **Budget Management** | ✅ | ⚠️ | CRUD works, no charts |
| **Time Tracking** | ✅ | ⚠️ | Manual log, no timer widget |
| **Resource Management** | ✅ | ⚠️ | Allocation table, no Gantt view |
| **Project Health Score** | ✅ | ✅ | Automated score + breakdown |
| **Stakeholder Dashboard** | ✅ | ⚠️ | KPI cards, no drill-down |
| **Workload View** | ✅ | ⚠️ | Table only, no bar chart |
| **Workflows** | ✅ | ❌ | Backend done, no UI |
| **Saved Views** | ✅ | ❌ | Backend done, no UI |
| **Automation Builder** | ❌ | ❌ | Not started |
| **AI Features** | ❌ | ❌ | Not started |
| **Slack Integration** | ❌ | ❌ | Not started |
| **GitHub Integration** | ❌ | ❌ | Not started |
| **Test Suite** | ❌ | ❌ | Zero test files |
| **Mobile Responsive** | — | ⚠️ | Basic, needs work |
| **Dark Mode** | — | ❌ | Not started |
| **i18n** | — | ❌ | Not started |

**Legend:** ✅ Done and polished · ⚠️ Partial (exists but incomplete) · ❌ Not started

---

## 9. What Is Fully Working

These features can be used end-to-end with no known gaps:

1. **Authentication** — Register, login, logout, JWT refresh, Google OAuth, TOTP 2FA (setup, verify, disable), remember device
2. **Workspace management** — Create, read, update, delete workspaces; add/remove/change-role members; invite via email
3. **Project management** — Full CRUD; create from template; visual template gallery; project-specific member roles
4. **Task management** — Full CRUD; multi-assignee; priority; due dates; subtasks; dependencies; bulk operations; CSV import/export; position reordering
5. **Kanban board** — Drag-and-drop tasks across statuses with real-time position persistence
6. **Custom fields** — Define (text/number/select/date/checkbox) per project; set values per task
7. **Comments** — Create/edit/delete with emoji reactions; scoped to task
8. **Attachments** — Upload files to tasks; download; delete; file size/mime validation
9. **Notifications** — In-app bell with badge; dropdown list; mark read/unread; 7 wired triggers; 60s poll fallback
10. **Global search** — Cmd+K modal; searches tasks, projects, workspaces, users, comments, documents; keyboard navigation
11. **Invite system** — Send email invite with token link; accept page; auto-add to workspace/project with correct role
12. **Activity log** — Every CRUD operation creates a log entry; surfaced in Settings > Audit Log tab
13. **RBAC V2** — 3-level role system fully enforced on every API endpoint; permission matrix UI
14. **Settings page** — 4 tabs: Security (2FA), Permissions (matrix), Audit Log, Project Templates
15. **Project health score** — Automated score calculation + visual breakdown component
16. **Landing page** — Full redesign with kanban mockup, stats, features, capabilities panels, pricing, testimonials, integrations, CTA, footer

---

## 10. What Is Partially Built

These features have both backend and frontend code but are incomplete or unpolished:

| Feature | What Works | What's Missing |
|---------|-----------|----------------|
| **Gantt chart** | Chart renders with bar rows | Dependency lines, drag to reschedule, real data binding |
| **Calendar view** | Tasks appear on due date | Drag-to-reschedule, multi-day spans, recurring events |
| **Chat** | Send/receive messages, Socket.io room join | Typing indicators, message reactions, file attachments in chat, read receipts |
| **Documents** | Create/edit/delete with plain textarea | Rich text editor (Notion-like blocks), collaboration, version history |
| **Whiteboard** | Canvas with basic shapes | Real-time multi-user sync, sticky notes, images, undo/redo |
| **Budget** | Create budget, add expenses, see total | Charts/graphs, overspend alerts, export |
| **Time tracking** | Manual log entries, view logs per task | Start/stop timer widget, weekly totals dashboard, Gantt integration |
| **Resource management** | Allocation percentage per user/project | Visual workload bar chart, conflict detection |
| **Workload view** | Table of assignments | Capacity visualization, drag-to-reassign |
| **Stakeholder dashboard** | KPI cards (task counts, velocity) | Charts, trend lines, export to PDF |
| **Notifications — real-time** | Backend emits `notification:new` on Socket.io | Frontend socket client does not listen; only polls every 60s |

---

## 11. What Is Missing / Not Started

### High Priority (P0 — core product)

**1. Test Suite — zero coverage**
- Backend: no Jest tests. `jest` and `supertest` are installed but no `__tests__` folder exists.
- Frontend: no React Testing Library tests. `@testing-library/react` is listed in `react-scripts` deps but no test files exist.
- Risk: every refactor is blind; no CI pipeline.

**2. Real-time Notification Push**
- `notificationSlice` has `addNotification` reducer ready.
- `socket.js` emits `notification:new` — but nothing connects.
- Frontend `socketClient.ts` exists but is not mounted anywhere.
- Fix needed: in `Layout.tsx` on mount, connect socket, listen for `notification:new`, dispatch `addNotification`.

**3. Workflows UI (Automation Builder)**
- Backend: `workflow.controller.js` + `Workflow` model fully exists (7 endpoints).
- Frontend: no UI whatsoever. This is the most requested P0 feature.
- Should be: visual trigger/condition/action builder (e.g., "When task status → Done, notify assignees").

**4. Saved Views UI**
- Backend: `savedView.controller.js` + `SavedView` model fully exists (5 endpoints).
- Frontend: no UI. Should be a "Save current filter" button in task list that persists the view.

### Medium Priority (P1)

**5. Rich Text Editor for Documents**
- Currently: `<textarea>` — plain text only.
- Needed: block-based editor (Notion-style). Consider: `@tiptap/react`, `slate.js`, or `quill`.

**6. Notification Preferences**
- No per-user settings for: email notifications on/off, digest vs. instant, do-not-disturb hours.
- Backend model would need a new `NotificationPreferences` table.

**7. Dark Mode**
- CSS custom properties are defined (`--bg-*`, `--text-*`) but no `[data-theme="dark"]` overrides exist.
- Requires: theme toggle in Settings, `prefers-color-scheme` media query.

**8. Advanced Search Filters**
- Current search is basic: string match across fields.
- Missing: AND/OR/NOT operators, filter by status/priority/date range, search history, saved searches.

**9. Duplicate Slice/Service Files**
- `timeTracking.slice.ts` + `timeTrackingSlice.ts` (one is dead code)
- `resource.slice.ts` + `resourceSlice.ts` (one is dead code)
- `timeTracking.service.ts` + `timeTrackingService.ts`
- `resource.service.ts` + `resourceService.ts`
- Should consolidate to one each and remove orphans.

### Low Priority / Future (P2)

**10. Integrations**
- Slack: post notifications to channels when tasks are updated
- GitHub: link commits/PRs to tasks
- Zapier/Make: webhook triggers
- Google Calendar: sync due dates

**11. AI Features**
- Task prioritization advisor (Claude API)
- Project health advisor ("you have 3 overdue tasks, risk is high")
- Auto-tagging tasks by description
- Sprint planning assistant

**12. Mobile Apps**
- No native iOS/Android app.
- The web app is not fully responsive below 600px.

**13. Multi-language (i18n)**
- No translation layer exists.
- Consider: `react-i18next`.

**14. Advanced Reporting**
- Custom report builder
- PDF/Excel export of any view
- Scheduled report emails

**15. Templates Marketplace**
- Pre-seeded templates: Agile Sprint, Marketing Campaign, Product Launch, Bug Triage
- Currently users create their own templates; no curated library

---

## 12. API Reference Summary

**Base URL:** `http://localhost:5000/api/v1`

**Headers required (authenticated):**
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**Standard response envelopes:**
```json
// Success (single)
{ "success": true, "data": { ... } }

// Success (list)
{ "success": true, "data": [ ... ], "total": 42, "page": 1 }

// Error
{ "error": "Human readable message", "details": [...] }
```

**Pagination:** `?page=1&limit=20` on list endpoints

**Filtering tasks:** `GET /tasks?projectId=X&statusId=Y&priority=high&assigneeId=Z&search=keyword`

**Global search:** `GET /search?q=keyword&types=tasks,projects,users`

---

## 13. Data Flow Diagrams

### Task Creation Flow
```
User fills "New Task" form
    ↓
dispatch(createTask({ title, projectId, listId, assigneeIds, ... }))
    ↓
taskSlice thunk → taskService.createTask(data)
    ↓
POST /api/v1/tasks
    ↓
authenticate() → task.controller.js::createTask()
    ↓
Task.create() + TaskAssignees.bulkCreate()
    ↓
[notification] Promise.allSettled → createNotification() per assignee
    ↓
res.json({ success: true, task })
    ↓
Redux: tasks array updated, UI re-renders
    ↓
Each assignee: NotificationBell polls → badge increments
```

### Invite Accept Flow
```
User clicks email link → /invite/:token
    ↓
AcceptInvitePage: GET /invites/:token (validates token, shows workspace/project info)
    ↓
User clicks "Accept"
    ↓
POST /invites/:token/accept (with Bearer token if logged in, else redirect to login first)
    ↓
invite.controller.js::acceptInvite()
    ↓
Invite.update({ status: 'accepted' })
WorkspaceMembers.create() or ProjectMembers.create()
createNotification({ userId: invite.invitedBy, type: 'invite_accepted' })
    ↓
Redirect to /dashboard or /projects/:id
```

### Notification Real-time Gap
```
Current (broken):
  createNotification() saved to DB
  Backend emits socket.io 'notification:new'  ← emitted but nobody listens
  Frontend polls every 60s                     ← only way user sees notification

Target (fix needed in Layout.tsx):
  Layout.tsx mounts → socketClient.connect()
  socketClient.on('notification:new', (notif) => dispatch(addNotification(notif)))
  NotificationBell badge updates instantly
```

---

## 14. Environment & Configuration

### Backend `.env`
```
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/pulsehub_dev
DB_HOST=localhost
DB_PORT=5432
DB_NAME=pulsehub_dev
DB_USER=postgres
DB_PASSWORD=yourpassword

# JWT
JWT_SECRET=your_very_long_random_secret
JWT_EXPIRES_IN=7d
JWT_REFRESH_SECRET=another_very_long_secret
JWT_REFRESH_EXPIRES_IN=30d

# Google OAuth
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
GOOGLE_CALLBACK_URL=http://localhost:5000/api/v1/auth/google/callback

# SMTP (for invite emails)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@gmail.com
SMTP_PASS=your_app_password
SMTP_FROM=PulseHub <noreply@pulsehub.dev>

# App
NODE_ENV=development
PORT=5000
FRONTEND_URL=http://localhost:3000
```

### Frontend (Vite)
Configured via `vite.config.ts`. API base URL is hardcoded in `services/api.ts` as `http://localhost:5000/api/v1`.

---

## 15. Demo Accounts

Run `cd backend && npm run seed:demo` to create all accounts.

| Role | Email | Password | Access Level |
|------|-------|----------|-------------|
| super_admin | super@pulsehub.dev | Super@123 | Everything — platform-wide |
| admin | admin@pulsehub.dev | Admin@123 | Manage all workspaces/users |
| owner | owner@pulsehub.dev | Owner@123 | Owns Demo Workspace Alpha |
| billing_admin | billing@pulsehub.dev | Billing@123 | Finance only |
| pm | pm@pulsehub.dev | Pm@123456 | Create projects in workspace |
| member | member@pulsehub.dev | Member@123 | Standard team member |
| commenter | commenter@pulsehub.dev | Comment@123 | Comment-only |
| guest | guest@pulsehub.dev | Guest@123 | Limited guest access |
| viewer | viewer@pulsehub.dev | Viewer@123 | Read-only |

Seed is **idempotent** — safe to run multiple times (`findOrCreate` throughout).

---

## 16. Known Bugs & Tech Debt

| # | Severity | Description | File(s) |
|---|---------|-------------|---------|
| 1 | High | Real-time notifications not pushed to frontend — only polled every 60s | `frontend/src/App.tsx` or `Layout.tsx` — needs socket listener |
| 2 | Medium | Duplicate slice/service files (resource, timeTracking) — one in each pair is dead code | `store/slices/`, `services/` |
| 3 | Medium | Gantt chart data binding partial — timeline bars render but dependency lines and real project data not always connected | `pages/gantt/GanttPage.tsx` |
| 4 | Medium | `backend/package.json` name still says `trackpro-server` | `backend/package.json` |
| 5 | Medium | Document editor is a plain `<textarea>` — not a rich text editor | `pages/documents/DocumentsPage.tsx` |
| 6 | Low | `AboutUsPage.tsx` and `ContactPage.tsx` exist as routes but have placeholder content | `pages/AboutUsPage.tsx` |
| 7 | Low | No loading skeleton states — most pages show blank while fetching | All pages |
| 8 | Low | No 404 page | `App.tsx` — add a catch-all route |
| 9 | Low | `redux-persist` persists only `auth` but token expiry isn't checked on app load | `store/index.ts` |
| 10 | Low | `express-session` is installed but not used — leftover from OAuth exploration | `backend/package.json` |

---

## 17. Roadmap — Prioritized

### Sprint 1 — Critical Gaps (Now)
1. **Socket.io frontend listener** — Wire `notification:new` in `Layout.tsx` (~30 lines)
2. **Saved Views UI** — "Save filter" button + view picker in task list
3. **404 page** — Catch-all route with friendly UI
4. **Loading skeletons** — Replace blank states with skeleton loaders
5. **Consolidate duplicate files** — Remove dead resource/timeTracking duplicates

### Sprint 2 — Core Product Completion
6. **Workflow Builder UI** — Visual trigger → condition → action editor (backend already done)
7. **Rich Text Editor** — Replace textarea in Documents with Tiptap or Quill
8. **Gantt chart** — Complete data binding + drag-to-reschedule
9. **Calendar** — Drag task to change due date
10. **Dark Mode** — CSS variable overrides + toggle in Settings

### Sprint 3 — Quality & Growth
11. **Test Suite** — Jest + Supertest for auth/task/search controllers; RTL for NotificationBell + GlobalSearch
12. **Notification preferences** — Per-user email/in-app settings
13. **Advanced Search** — Filters, operators, history
14. **Whiteboard real-time** — Socket.io sync for whiteboard elements
15. **Mobile responsive** — Fix all pages below 768px

### Sprint 4 — Integrations & AI
16. **Slack integration** — Webhook → post to channel on task update
17. **GitHub integration** — Link commits to tasks via PR description
18. **Claude API integration** — Task prioritization advisor, project health insights
19. **PDF/Excel export** — Any view → download
20. **Templates marketplace** — Pre-seeded Agile, marketing, product launch templates

### Sprint 5 — Scale
21. **CI/CD pipeline** — GitHub Actions: lint → test → build → deploy
22. **Multi-language (i18n)** — react-i18next scaffolding
23. **Native mobile apps** — React Native reuse of API layer
24. **Multi-tenant billing** — Stripe integration for Pro/Enterprise plans
25. **Custom domains** — Per-workspace subdomain routing

---

*This document is the single source of truth for PulseHub's technical state as of 2026-03-18.*
*Update this file whenever a feature is completed, a new pattern is established, or architecture changes.*
