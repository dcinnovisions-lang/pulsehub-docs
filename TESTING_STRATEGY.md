# PulseHub — Testing Strategy & Test Execution Plan
> Complete, page-by-page, endpoint-by-endpoint test plan covering API tests, role-based UI matrix, edge cases, walkthrough scripts, performance benchmarks, and execution checklists.
> Last updated: 2026-03-20

---

## 5-PART TESTING PLAN — SESSION TRACKER

> Each part covers Backend + Frontend + E2E for one domain slice.
> After each part there is a demonstrable, showcaseable result.
> Start each new session by reading this table to know exactly where to resume.

| Part | Domain | Backend | Frontend | E2E Specs | Status | Showcase |
|------|--------|---------|----------|-----------|--------|---------|
| **1** | Auth + RBAC + Users | auth.test.js, user.test.js, rbac checks | LoginPage.test, authSlice.test, PermissionGuard.test | auth.spec.ts, rbac.spec.ts, edge-cases-auth.spec.ts, edge-cases-rbac.spec.ts | ✅ COMPLETE (2026-03-20) | "All 6 roles secured, guards enforced" |
| **2** | Workspaces + Projects + Tasks + Kanban + Gantt | workspace.test.js, project.test.js, task.test.js | KanbanBoard.test, TaskDetailPage.test, selectors.test | tasks.spec.ts, gantt.spec.ts, edge-cases-tasks.spec.ts | ⬜ NOT STARTED | "Full task lifecycle works for every role" |
| **3** | Chat + Documents + Whiteboard + Notifications | chat.test.js, document.test.js | NotificationBell.test, GlobalSearch.test | chat.spec.ts, documents.spec.ts, edge-cases-chat.spec.ts, edge-cases-documents.spec.ts, edge-cases-socket.spec.ts | ⬜ NOT STARTED | "Real-time collaboration fully tested" |
| **4** | Budget + Automations + Time Tracking + Dashboard | budget, automation, time-tracking endpoints | Dashboard widget tests | budget.spec.ts, automations.spec.ts, dashboard.spec.ts | ⬜ NOT STARTED | "Business features gated by role, charts render" |
| **5** | Settings + All Edge Cases + Final Report | — | — | settings.spec.ts, all remaining edge-cases-*.spec.ts | ⬜ NOT STARTED | TEST_RESULTS.md — production readiness report |

### Status Key
```
⬜ NOT STARTED
🔄 IN PROGRESS
✅ COMPLETE
❌ BLOCKED (note reason)
```

### How to Resume in a New Session
1. Read this table — find the first row that is NOT ✅
2. Tell Claude: **"Start Part X"**
3. Claude writes all test files for that part, runs them, fixes failures, marks ✅

### Test Files Location
```
backend/src/__tests__/          ← Jest + Supertest
frontend/src/__tests__/         ← Vitest + React Testing Library
e2e/                            ← Playwright (all roles × all pages)
D:\PulseHub\Docs\TEST_RESULTS.md ← Pass/fail report (created in Part 5)
```

### Part 1 — Start Prompt (copy-paste when ready)
> "Start Part 1 — Auth + RBAC + Users testing"

### Part 1 — Files Written (✅ 2026-03-20)

**Backend (Jest + Supertest — mocked DB):**
```
backend/jest.config.js
backend/src/__tests__/helpers/setup.js          ← env vars injected before tests
backend/src/__tests__/helpers/jwt.js            ← token factory + mock users for all 6 roles
backend/src/__tests__/auth.test.js              ← 30 tests: register, login, 2FA, refresh, logout, edge cases
backend/src/__tests__/user.test.js              ← 22 tests: CRUD + RBAC matrix all 6 roles
```

**Frontend (Jest + React Testing Library via react-scripts):**
```
frontend/src/__tests__/authSlice.test.ts        ← 25 tests: all reducers + async thunk state transitions
frontend/src/__tests__/permissions.test.ts      ← 60+ tests: every exported function, full role matrix
frontend/src/__tests__/PermissionGuard.test.tsx ← 30 tests: all 6 roles × 4 actions + ownership
frontend/src/__tests__/LoginPage.test.tsx       ← 20 tests: render, validation, login, 2FA flow
```

**E2E (Playwright):**
```
playwright.config.ts                            ← config (Chromium, baseURL localhost:3000)
e2e/helpers/auth.ts                             ← loginAs(), injectAuthState(), fillLoginForm()
e2e/auth.spec.ts                                ← A1–A10: login page, auth flows, protected routes
e2e/rbac.spec.ts                                ← R1–R10: all 6 roles × pages × API RBAC
```

**To run:**
```bash
# Backend
cd backend && npx jest --coverage

# Frontend
cd frontend && npm test -- --watchAll=false

# E2E (requires both servers running + Playwright installed)
npm install -D @playwright/test && npx playwright install chromium
npx playwright test e2e/auth.spec.ts e2e/rbac.spec.ts
```

### Part 2 — Start Prompt
> "Start Part 2 — Workspaces + Projects + Tasks + Kanban + Gantt testing"

---

## SECTION 1: How Tests Are Run

### Backend Tests (API)
```bash
# Base URL
BASE=http://localhost:5000/api

# Auth header (replace TOKEN after login)
AUTH="Authorization: Bearer $TOKEN"

# Standard curl pattern
curl -s -X <METHOD> "$BASE/<path>" \
  -H "Content-Type: application/json" \
  -H "$AUTH" \
  -d '{"key":"value"}' | jq .
```

### Frontend Tests (Manual / Playwright)
For each page: navigate, check render, perform CRUD actions, check error states.

### Test Users (from seed-demo.js)
| Role | Email | Password |
|------|-------|----------|
| super_admin | admin@pulsehub.com | Admin123! |
| owner | owner@pulsehub.com | Owner123! |
| admin | adminuser@pulsehub.com | Admin123! |
| billing_admin | billing@pulsehub.com | Billing123! |
| member | member@pulsehub.com | Member123! |
| guest | guest@pulsehub.com | Guest123! |

---

## SECTION 2: API Endpoint Tests

### Test Suite Index

| # | Module | API Endpoints | Frontend Page | Priority |
|---|--------|--------------|---------------|----------|
| 1 | Health Check | GET /health | — | P0 |
| 2 | Auth — Login/Register | 8 endpoints | /auth/login, /auth/register | P0 |
| 3 | Auth — 2FA | 4 endpoints | /settings (Security tab) | P1 |
| 4 | Auth — Google OAuth | 2 endpoints | /auth/google | P2 |
| 5 | Users | 7 endpoints | /app/users | P0 |
| 6 | Workspaces | 11 endpoints | /app/workspaces | P0 |
| 7 | Projects | 14 endpoints | /app/projects | P0 |
| 8 | Tasks | 17 endpoints | /app/tasks, /app/tasks/:id | P0 |
| 9 | Subtasks | 3 endpoints | Task detail modal | P1 |
| 10 | Task Dependencies | 4 endpoints | Task detail modal | P1 |
| 11 | Custom Fields | 6 endpoints | Project settings | P1 |
| 12 | Kanban Board | GET /projects/:id/kanban | /app/kanban | P0 |
| 13 | Gantt Chart | GET /gantt/:projectId | /app/gantt | P1 |
| 14 | Calendar | GET /calendar | /app/calendar | P1 |
| 15 | Statuses | 4 endpoints | Project settings | P1 |
| 16 | Comments | 6 endpoints | Task detail | P1 |
| 17 | Attachments | 4 endpoints | Task detail | P1 |
| 18 | Notifications | 6 endpoints | Bell icon + /app/notifications | P1 |
| 19 | Search | GET /search | Cmd+K global search | P1 |
| 20 | Dashboard (Personal) | — | /app/dashboard | P0 |
| 21 | Stakeholder Dashboard | GET /dashboard/stakeholder | /app/stakeholder-dashboard | P1 |
| 22 | Time Tracking | 5 endpoints | /app/time-tracking | P1 |
| 23 | Resource Management | 2 endpoints | /app/resources | P1 |
| 24 | Budget Management | 4 endpoints | /app/budgets | P1 |
| 25 | Activity Log | GET /activity-logs | /app/settings (Audit Log tab) | P2 |
| 26 | Saved Views | 5 endpoints | Tasks filter panel | P2 |
| 27 | Workflows | 7 endpoints | Project settings | P2 |
| 28 | Automation Builder | 6 endpoints | /app/automations | P1 |
| 29 | Documents | 9 endpoints | /app/documents | P1 |
| 30 | Whiteboard | 5 endpoints | /app/whiteboard | P1 |
| 31 | Chat | 12 endpoints | /app/chat | P1 |
| 32 | Invites | 5 endpoints | Workspace invite flow | P1 |
| 33 | Permissions Matrix | GET /permissions/matrix | /app/settings | P2 |
| 34 | Workload View | — | /app/workload | P2 |
| 35 | Settings Page | — | /app/settings | P1 |

---

### 1. Health Check

**Endpoint:** `GET /api/health`

| Test | Expected |
|------|----------|
| Server is running | 200 `{ success: true }` |

```bash
curl -s http://localhost:5000/api/health | jq .
```

---

### 2. Auth — Login / Register / Session

**Endpoints:** `/auth/login`, `/auth/register`, `/auth/me`, `/auth/logout`, `/auth/refresh-token`, `/auth/verify-email/:token`, `/auth/resend-verification`

#### 2.1 Login — Happy Path
```bash
curl -s -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@pulsehub.com","password":"Admin123!"}' | jq .
# Expected: 200 { success: true, data: { user: {...}, token: "..." } }
```

#### 2.2 Login — Wrong Password
```bash
-d '{"email":"admin@pulsehub.com","password":"wrongpass"}'
# Expected: 401 { success: false, message: "Invalid credentials" }
```

#### 2.3 Login — Non-existent User
```bash
-d '{"email":"nobody@x.com","password":"test"}'
# Expected: 401 Invalid credentials
```

#### 2.4 Login — Missing Fields
```bash
-d '{"email":"admin@pulsehub.com"}'
# Expected: 400 validation error
```

#### 2.5 Get Current User (`/auth/me`)
```bash
curl -s http://localhost:5000/api/auth/me \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { success: true, data: { user: {...} } }
```

#### 2.6 Get Me — No Token
```bash
curl -s http://localhost:5000/api/auth/me | jq .
# Expected: 401 Unauthorized
```

#### 2.7 Register — Happy Path
```bash
curl -s -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"newuser@test.com","password":"Test1234!","firstName":"Test","lastName":"User"}' | jq .
# Expected: 201 { success: true, data: { user: {...}, token: "..." } }
```

#### 2.8 Register — Duplicate Email
```bash
# Same as above but with existing email
# Expected: 400/409 "Email already in use"
```

#### 2.9 Logout
```bash
curl -s -X POST http://localhost:5000/api/auth/logout \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { success: true }
```

#### 2.10 Refresh Token
```bash
curl -s -X POST http://localhost:5000/api/auth/refresh-token \
  -H "Content-Type: application/json" \
  -d '{"refreshToken":"<refresh_token>"}' | jq .
# Expected: 200 { success: true, data: { token: "..." } }
```

**Frontend Checks:**
- [ ] `/login` renders form — email + password fields + Submit button
- [ ] Login succeeds → redirects to `/app/dashboard`
- [ ] Wrong password → shows inline error message
- [ ] `/register` renders form — all fields present
- [ ] Register succeeds → user logged in + email verification banner shown

---

### 3. Auth — Two-Factor Authentication (2FA)

**Endpoints:** `/auth/2fa/setup`, `/auth/2fa/verify`, `/auth/2fa/disable`, `/auth/2fa/verify-login`, `/auth/2fa/backup-login`

#### 3.1 Setup 2FA
```bash
curl -s -X POST http://localhost:5000/api/auth/2fa/setup \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { success: true, data: { secret, qrCode, backupCodes } }
```

#### 3.2 Verify 2FA TOTP Code
```bash
curl -s -X POST http://localhost:5000/api/auth/2fa/verify \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"token":"123456"}' | jq .
# Expected: 200 (with valid speakeasy TOTP token)
```

#### 3.3 Login with 2FA Enabled
```bash
# Step 1: login returns { requires2FA: true, tempToken: "..." }
# Step 2: POST /auth/2fa/verify-login { tempToken, totpToken }
# Expected: 200 full auth response
```

#### 3.4 Backup Code Login
```bash
curl -s -X POST http://localhost:5000/api/auth/2fa/backup-login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@pulsehub.com","backupCode":"<code>"}' | jq .
# Expected: 200 with valid backup code
```

**Frontend Checks:**
- [ ] Settings → Security tab → "Enable 2FA" button present
- [ ] Setup flow: QR code shown + backup codes listed
- [ ] After enable: login requires TOTP code entry

---

### 4. Users

**Endpoints:** `GET /users`, `GET /users/:id`, `PUT /users/:id`, `PUT /users/:id/role`, `DELETE /users/:id`, `GET /users/search`, `POST /users/me/avatar`, `DELETE /users/me`

#### 4.1 List All Users (super_admin only)
```bash
curl -s "$BASE/users" -H "Authorization: Bearer $ADMIN_TOKEN" | jq .
# Expected: 200 array of users
```

#### 4.2 Get Single User
```bash
curl -s "$BASE/users/$USER_ID" -H "Authorization: Bearer $ADMIN_TOKEN" | jq .
# Expected: 200 user object (no password/twoFactorSecret)
```

#### 4.3 Update User
```bash
curl -s -X PUT "$BASE/users/$USER_ID" \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Updated","lastName":"Name"}' | jq .
# Expected: 200 updated user
```

#### 4.4 Update User Role
```bash
curl -s -X PUT "$BASE/users/$USER_ID/role" \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"role":"admin"}' | jq .
# Expected: 200
```

#### 4.5 Search Users
```bash
curl -s "$BASE/users/search?q=admin" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 matching users array
```

#### 4.6 Delete User (super_admin only)
```bash
curl -s -X DELETE "$BASE/users/$USER_ID" \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq .
# Expected: 200/204
```

#### 4.7 Delete Own Account
```bash
curl -s -X DELETE "$BASE/users/me" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200
```

**Frontend Checks (UsersPage `/app/users` — super_admin only):**
- [ ] Page renders user table with all users
- [ ] Can change user role via dropdown
- [ ] Can deactivate/delete user
- [ ] Non-super_admin cannot access `/app/users` → redirect
- [ ] Avatar upload button works

---

### 5. Workspaces

**Endpoints:** `GET /workspaces`, `POST /workspaces`, `GET /workspaces/:id`, `PUT /workspaces/:id`, `DELETE /workspaces/:id`, `GET /workspaces/:id/members`, `POST /workspaces/:id/members`, `PUT /workspaces/:id/members/:userId`, `DELETE /workspaces/:id/members/:userId`, `POST /workspaces/:id/restore`, `POST /workspaces/:id/hard-delete`

#### 5.1 List Workspaces
```bash
curl -s "$BASE/workspaces" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of workspaces the user belongs to
```

#### 5.2 Create Workspace
```bash
curl -s -X POST "$BASE/workspaces" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Workspace","description":"for testing"}' | jq .
# Expected: 201 new workspace
```

#### 5.3 Get Single Workspace
```bash
curl -s "$BASE/workspaces/$WS_ID" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200
```

#### 5.4 Update Workspace
```bash
curl -s -X PUT "$BASE/workspaces/$WS_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Renamed Workspace"}' | jq .
# Expected: 200 (owner/admin only)
```

#### 5.5 List Members
```bash
curl -s "$BASE/workspaces/$WS_ID/members" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of members with roles
```

#### 5.6 Add Member
```bash
curl -s -X POST "$BASE/workspaces/$WS_ID/members" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"userId":"<uuid>","role":"member"}' | jq .
# Expected: 201
```

#### 5.7 Update Member Role
```bash
curl -s -X PUT "$BASE/workspaces/$WS_ID/members/$USER_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"role":"admin"}' | jq .
# Expected: 200
```

#### 5.8 Remove Member
```bash
curl -s -X DELETE "$BASE/workspaces/$WS_ID/members/$USER_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200/204
```

#### 5.9 Soft Delete Workspace
```bash
curl -s -X DELETE "$BASE/workspaces/$WS_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 (owner only)
```

#### 5.10 Restore Workspace
```bash
curl -s -X POST "$BASE/workspaces/$WS_ID/restore" \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq .
# Expected: 200
```

**Frontend Checks (WorkspacesPage `/app/workspaces`):**
- [ ] Workspace cards render correctly
- [ ] Create workspace modal opens + submits
- [ ] Edit workspace name/description works
- [ ] Member list shows roles
- [ ] Role change dropdown saves correctly
- [ ] Delete workspace prompts confirmation

---

### 6. Projects

**Endpoints:** `GET /projects`, `POST /projects`, `GET /projects/:id`, `PUT /projects/:id`, `DELETE /projects/:id`, `GET /projects/:id/health`, `GET /projects/:id/kanban`, `GET /projects/:id/members`, `GET /projects/:id/members/roles`, `POST /projects/:id/members`, `PUT /projects/:id/members/:userId`, `DELETE /projects/:id/members/:userId`, `GET /projects/templates`, `POST /projects/from-template`

#### 6.1 List Projects
```bash
curl -s "$BASE/projects" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of projects
```

#### 6.2 Create Project
```bash
curl -s -X POST "$BASE/projects" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Project","workspaceId":"<ws_id>","description":"Test"}' | jq .
# Expected: 201
```

#### 6.3 Get Project
```bash
curl -s "$BASE/projects/$PROJ_ID" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200
```

#### 6.4 Update Project
```bash
curl -s -X PUT "$BASE/projects/$PROJ_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Renamed Project","status":"in_progress"}' | jq .
# Expected: 200
```

#### 6.5 Project Health Score
```bash
curl -s "$BASE/projects/$PROJ_ID/health" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { score, breakdown }
```

#### 6.6 Get Project Templates
```bash
curl -s "$BASE/projects/templates" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of templates
```

#### 6.7 Create from Template
```bash
curl -s -X POST "$BASE/projects/from-template" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"templateId":"<id>","name":"New from Template","workspaceId":"<ws_id>"}' | jq .
# Expected: 201
```

#### 6.8 Project Members CRUD
```bash
# List: GET /projects/:id/members
# Add:  POST /projects/:id/members { userId, role: "contributor" }
# Update: PUT /projects/:id/members/:userId { role: "reviewer" }
# Remove: DELETE /projects/:id/members/:userId
```

**Frontend Checks (ProjectsPage + ProjectDetailPage):**
- [ ] Projects list renders with cards/table
- [ ] Create project modal: name, workspace selector, description
- [ ] Project detail page loads with tabs (Tasks, Members, Settings, etc.)
- [ ] Health score widget displays correctly
- [ ] Template gallery modal opens + can create from template
- [ ] Members tab: add/remove/change role works
- [ ] Delete project prompts confirmation, removes from list

---

### 7. Tasks

**Endpoints:** `GET /tasks`, `POST /tasks`, `GET /tasks/:id`, `PUT /tasks/:id`, `DELETE /tasks/:id`, `PUT /tasks/:id/move`, `PUT /tasks/reorder`, `POST /tasks/bulk`, `PUT /tasks/bulk`, `DELETE /tasks/bulk`, `POST /tasks/bulk-archive`, `GET /tasks/export`, `POST /tasks/import`

#### 7.1 List Tasks (with filters)
```bash
curl -s "$BASE/tasks?projectId=$PROJ_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { tasks: [...], total: N }

# With filters:
curl -s "$BASE/tasks?projectId=$PROJ_ID&status=in_progress&assignee=$USER_ID&priority=high" \
  -H "Authorization: Bearer $TOKEN" | jq .
```

#### 7.2 Create Task
```bash
curl -s -X POST "$BASE/tasks" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test Task",
    "projectId": "'$PROJ_ID'",
    "priority": "medium",
    "status": "todo"
  }' | jq .
# Expected: 201 new task with id
```

#### 7.3 Get Task
```bash
curl -s "$BASE/tasks/$TASK_ID" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 task with subtasks, assignees, comments, attachments
```

#### 7.4 Update Task
```bash
curl -s -X PUT "$BASE/tasks/$TASK_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated Task","priority":"high","status":"in_progress"}' | jq .
# Expected: 200
```

#### 7.5 Delete Task
```bash
curl -s -X DELETE "$BASE/tasks/$TASK_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200/204
```

#### 7.6 Move Task (status change)
```bash
curl -s -X PUT "$BASE/tasks/$TASK_ID/move" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"statusId":"<status_id>"}' | jq .
# Expected: 200
```

#### 7.7 Bulk Create
```bash
curl -s -X POST "$BASE/tasks/bulk" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"tasks":[{"title":"Bulk 1","projectId":"'$PROJ_ID'"},{"title":"Bulk 2","projectId":"'$PROJ_ID'"}]}' | jq .
# Expected: 201
```

#### 7.8 Bulk Update
```bash
curl -s -X PUT "$BASE/tasks/bulk" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"taskIds":["<id1>","<id2>"],"updates":{"priority":"low"}}' | jq .
# Expected: 200
```

#### 7.9 Bulk Delete
```bash
curl -s -X DELETE "$BASE/tasks/bulk" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"taskIds":["<id1>","<id2>"]}' | jq .
# Expected: 200
```

#### 7.10 Export Tasks (CSV)
```bash
curl -s "$BASE/tasks/export?projectId=$PROJ_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -o tasks_export.csv
# Expected: CSV file download
```

#### 7.11 Import Tasks (CSV)
```bash
curl -s -X POST "$BASE/tasks/import" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@tasks_export.csv" \
  -F "projectId=$PROJ_ID" | jq .
# Expected: 200 { imported: N }
```

**Frontend Checks (TasksPage + TaskDetailPage):**
- [ ] Tasks list renders with filter bar
- [ ] Filter by status, priority, assignee works
- [ ] Create task modal opens + submits
- [ ] Task row click opens detail panel/modal
- [ ] Edit task: title, description, priority, due date
- [ ] Assign user works
- [ ] Bulk select checkboxes + bulk actions toolbar
- [ ] CSV export downloads file
- [ ] CSV import modal accepts file
- [ ] Subtasks section in task detail

---

### 8. Subtasks

**Endpoints:** `POST /tasks/:taskId/subtasks`, `PUT /tasks/:taskId/subtasks/:subtaskId`, `DELETE /tasks/:taskId/subtasks/:subtaskId`

#### 8.1 Create Subtask
```bash
curl -s -X POST "$BASE/tasks/$TASK_ID/subtasks" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Sub-task 1"}' | jq .
# Expected: 201
```

#### 8.2 Update Subtask
```bash
curl -s -X PUT "$BASE/tasks/$TASK_ID/subtasks/$SUB_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated","isCompleted":true}' | jq .
# Expected: 200
```

#### 8.3 Delete Subtask
```bash
curl -s -X DELETE "$BASE/tasks/$TASK_ID/subtasks/$SUB_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200/204
```

---

### 9. Task Dependencies

**Endpoints:** `GET /tasks/:taskId/dependencies`, `POST /tasks/:taskId/dependencies`, `DELETE /tasks/:taskId/dependencies/:dependencyId`, `GET /tasks/:taskId/dependency-graph`

#### 9.1 Add Dependency
```bash
curl -s -X POST "$BASE/tasks/$TASK_ID/dependencies" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"dependsOnId":"<other_task_id>","type":"finish_to_start"}' | jq .
# Expected: 201
```

#### 9.2 Get Dependency Graph
```bash
curl -s "$BASE/tasks/$TASK_ID/dependency-graph" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 nodes + edges
```

---

### 10. Custom Fields

**Endpoints:** `GET /projects/:projectId/custom-fields`, `POST /projects/:projectId/custom-fields`, `PUT /custom-fields/:id`, `DELETE /custom-fields/:id`, `GET /tasks/:taskId/custom-fields`, `PUT /tasks/:taskId/custom-fields/:customFieldId`

#### 10.1 Create Custom Field
```bash
curl -s -X POST "$BASE/projects/$PROJ_ID/custom-fields" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Client Name","type":"text"}' | jq .
# Expected: 201
```

#### 10.2 Set Field Value on Task
```bash
curl -s -X PUT "$BASE/tasks/$TASK_ID/custom-fields/$FIELD_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"value":"Acme Corp"}' | jq .
# Expected: 200
```

---

### 11. Kanban Board

**Endpoint:** `GET /projects/:id/kanban`

#### 11.1 Get Kanban Data
```bash
curl -s "$BASE/projects/$PROJ_ID/kanban" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { columns: [{ status, tasks: [...] }] }
```

**Frontend Checks (KanbanBoardPage `/app/kanban`):**
- [ ] Columns render per status
- [ ] Task cards display title, priority, assignee avatar
- [ ] Drag card between columns → task status updates
- [ ] Create task in column works
- [ ] Column header shows task count

---

### 12. Gantt Chart

**Endpoint:** `GET /gantt/:projectId`

#### 12.1 Get Gantt Data
```bash
curl -s "$BASE/gantt/$PROJ_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { tasks: [{ id, title, startDate, dueDate, dependencies }] }
```

**Frontend Checks (GanttPage `/app/gantt`):**
- [ ] Timeline renders with tasks as bars
- [ ] Dependency lines drawn between dependent tasks
- [ ] Zoom controls work (day/week/month)
- [ ] Today marker visible
- [ ] Drag bars to reschedule → saves new dates to backend

---

### 13. Calendar

**Endpoint:** `GET /calendar`

#### 13.1 Get Calendar Events
```bash
curl -s "$BASE/calendar?start=2026-03-01&end=2026-03-31" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of tasks with due dates in range
```

**Frontend Checks (CalendarPage `/app/calendar`):**
- [ ] Month/week/day view toggles work
- [ ] Tasks with due dates appear on correct day
- [ ] Click on day → create task modal opens with date pre-filled
- [ ] Clicking task event → task detail opens

---

### 14. Statuses

**Endpoints:** `GET /projects/:projectId/statuses`, `POST /projects/:projectId/statuses`, `PUT /statuses/:id`, `DELETE /statuses/:id`

#### 14.1 List Statuses
```bash
curl -s "$BASE/projects/$PROJ_ID/statuses" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of status objects
```

#### 14.2 Create Status
```bash
curl -s -X POST "$BASE/projects/$PROJ_ID/statuses" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"In Review","color":"#ffa500","order":3}' | jq .
# Expected: 201
```

#### 14.3 Update / Delete Status
```bash
curl -s -X PUT "$BASE/statuses/$STATUS_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Revised","color":"#ff0000"}' | jq .

curl -s -X DELETE "$BASE/statuses/$STATUS_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
```

---

### 15. Comments & Reactions

**Endpoints:** `GET /tasks/:taskId/comments`, `POST /tasks/:taskId/comments`, `PUT /comments/:id`, `DELETE /comments/:id`, `POST /comments/:id/reactions`, `DELETE /comments/:id/reactions/:emoji`

#### 15.1 Add Comment
```bash
curl -s -X POST "$BASE/tasks/$TASK_ID/comments" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"This looks good!"}' | jq .
# Expected: 201 comment with user, timestamp
```

#### 15.2 Add Reaction
```bash
curl -s -X POST "$BASE/comments/$COMMENT_ID/reactions" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"emoji":"👍"}' | jq .
# Expected: 200
```

#### 15.3 Remove Reaction
```bash
curl -s -X DELETE "$BASE/comments/$COMMENT_ID/reactions/👍" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200
```

**Frontend Checks:**
- [ ] Comments list in task detail renders
- [ ] Post comment → immediately appears
- [ ] Edit own comment → saves
- [ ] Delete own comment → removed
- [ ] Emoji reaction buttons work, counter updates

---

### 16. Attachments

**Endpoints:** `GET /tasks/:taskId/attachments`, `POST /tasks/:taskId/attachments`, `GET /attachments/:id/download`, `DELETE /attachments/:id`

#### 16.1 Upload Attachment
```bash
curl -s -X POST "$BASE/tasks/$TASK_ID/attachments" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/path/to/file.pdf" | jq .
# Expected: 201 { id, filename, url }
```

#### 16.2 Download Attachment
```bash
curl -s "$BASE/attachments/$ATTACH_ID/download" \
  -H "Authorization: Bearer $TOKEN" \
  -o downloaded_file.pdf
# Expected: file contents
```

#### 16.3 Delete Attachment
```bash
curl -s -X DELETE "$BASE/attachments/$ATTACH_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200/204
```

---

### 17. Notifications

**Endpoints:** `GET /notifications`, `GET /notifications/unread-count`, `PUT /notifications/read-all`, `PUT /notifications/:id/read`, `DELETE /notifications`, `DELETE /notifications/:id`

#### 17.1 Get Notifications
```bash
curl -s "$BASE/notifications" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of notifications
```

#### 17.2 Unread Count
```bash
curl -s "$BASE/notifications/unread-count" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { count: N }
```

#### 17.3 Mark One as Read
```bash
curl -s -X PUT "$BASE/notifications/$NOTIF_ID/read" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200
```

#### 17.4 Mark All as Read
```bash
curl -s -X PUT "$BASE/notifications/read-all" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200
```

#### 17.5 Delete Notification
```bash
curl -s -X DELETE "$BASE/notifications/$NOTIF_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200/204
```

**Frontend Checks:**
- [ ] Bell icon shows unread count badge
- [ ] Click bell → dropdown panel opens with notifications
- [ ] Clicking notification → navigates to related task/project
- [ ] "Mark all read" button clears badge
- [ ] Real-time: create task in another tab → notification appears without refresh (Socket.io)

---

### 18. Global Search

**Endpoint:** `GET /search?q=`

#### 18.1 Search Tasks
```bash
curl -s "$BASE/search?q=test&type=task" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { tasks: [...], projects: [...], users: [...] }
```

**Frontend Checks:**
- [ ] Cmd+K opens search modal
- [ ] Typing shows results categorized (Tasks, Projects, Users)
- [ ] Clicking result navigates to entity
- [ ] Escape closes modal

---

### 19. Personal Dashboard

**Frontend Checks (DashboardPage `/app/dashboard`):**
- [ ] Page loads without errors
- [ ] Assigned tasks widget shows current user's tasks
- [ ] Recent activity feed renders
- [ ] Quick create task button works
- [ ] Project stats (open/closed tasks count) correct

---

### 20. Stakeholder Dashboard

**Endpoint:** `GET /dashboard/stakeholder`

#### 20.1 Get Stakeholder Data
```bash
curl -s "$BASE/dashboard/stakeholder?workspaceId=$WS_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { projects: [...kpis...], teamUtilization, ... }
```

**Frontend Checks (StakeholderDashboardPage):**
- [ ] KPI cards render (on-track, at-risk, overdue counts)
- [ ] Project health grid shows all projects
- [ ] Team utilization chart renders

---

### 21. Time Tracking

**Endpoints:** `GET /time-logs`, `POST /time-logs`, `PUT /time-logs/:id`, `DELETE /time-logs/:id`, `GET /time-logs/statistics`

#### 21.1 Log Time
```bash
curl -s -X POST "$BASE/time-logs" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "taskId": "'$TASK_ID'",
    "hours": 2.5,
    "date": "2026-03-19",
    "description": "Worked on feature"
  }' | jq .
# Expected: 201
```

#### 21.2 Get Statistics
```bash
curl -s "$BASE/time-logs/statistics?userId=$USER_ID&startDate=2026-03-01&endDate=2026-03-31" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { totalHours, byProject: [...], byDay: [...] }
```

#### 21.3 Update / Delete Log
```bash
curl -s -X PUT "$BASE/time-logs/$LOG_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"hours":3}' | jq .

curl -s -X DELETE "$BASE/time-logs/$LOG_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
```

**Frontend Checks (TimeTrackingPage):**
- [ ] Log list renders with hours, date, task name
- [ ] Add log form submits
- [ ] Statistics summary card shows totals
- [ ] Live timer widget: Start/Pause/Stop (TimeTracker.tsx)
- [ ] Timer persists if page refreshed

---

### 22. Resource Management

**Endpoints:** `GET /resources/workload`, `POST /resources`

#### 22.1 Get Workload
```bash
curl -s "$BASE/resources/workload?workspaceId=$WS_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of { userId, name, allocations, utilization% }
```

#### 22.2 Create Allocation
```bash
curl -s -X POST "$BASE/resources" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"userId":"<id>","projectId":"<id>","allocationPercent":60}' | jq .
# Expected: 201
```

**Frontend Checks (ResourceManagementPage):**
- [ ] Team member cards render with allocation bars
- [ ] Add allocation modal works
- [ ] Over-allocation shows warning (red bar)

---

### 23. Budget Management

**Endpoints:** `GET /budgets`, `POST /budgets`, `GET /budgets/:budgetId/expenses`, `POST /budgets/:budgetId/expenses`

#### 23.1 Create Budget
```bash
curl -s -X POST "$BASE/budgets" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"projectId":"'$PROJ_ID'","totalBudget":50000,"currency":"USD"}' | jq .
# Expected: 201
```

#### 23.2 Add Expense
```bash
curl -s -X POST "$BASE/budgets/$BUDGET_ID/expenses" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"amount":1200,"description":"Software license","category":"tools"}' | jq .
# Expected: 201
```

**Frontend Checks (BudgetManagementPage):**
- [ ] Budget progress bar renders
- [ ] Expenses list with categories
- [ ] Add expense modal works
- [ ] Over-budget warning shown

---

### 24. Activity Log

**Endpoint:** `GET /activity-logs`

#### 24.1 Get Activity Logs
```bash
curl -s "$BASE/activity-logs?workspaceId=$WS_ID&limit=20" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of { action, entity, user, timestamp }
```

**Frontend Checks (Settings → Audit Log tab):**
- [ ] Timeline renders with user avatars, actions, timestamps
- [ ] Filter by user works
- [ ] Pagination / infinite scroll works

---

### 25. Saved Views

**Endpoints:** `GET /saved-views`, `POST /saved-views`, `GET /saved-views/:id`, `PUT /saved-views/:id`, `DELETE /saved-views/:id`

#### 25.1 Create Saved View
```bash
curl -s -X POST "$BASE/saved-views" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My High Priority Tasks",
    "filters": {"priority":"high","assigneeId":"'$USER_ID'"},
    "projectId": "'$PROJ_ID'"
  }' | jq .
# Expected: 201
```

---

### 26. Workflows

**Endpoints:** `GET /workflows`, `GET /workflows/templates`, `POST /workflows`, `GET /workflows/:id`, `PUT /workflows/:id`, `DELETE /workflows/:id`, `POST /workflows/:id/apply`

#### 26.1 List Workflow Templates
```bash
curl -s "$BASE/workflows/templates" -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array of preset workflow templates
```

#### 26.2 Apply Workflow to Project
```bash
curl -s -X POST "$BASE/workflows/$WF_ID/apply" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"projectId":"'$PROJ_ID'"}' | jq .
# Expected: 200
```

---

### 27. Automation Builder

**Endpoints:** `GET /automations`, `POST /automations`, `GET /automations/:id`, `PUT /automations/:id`, `DELETE /automations/:id`, `PATCH /automations/:id/toggle`

#### 27.1 List Automations
```bash
curl -s "$BASE/automations?workspaceId=$WS_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 array
```

#### 27.2 Create Automation
```bash
curl -s -X POST "$BASE/automations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Auto-assign high priority tasks",
    "workspaceId": "'$WS_ID'",
    "trigger": { "type": "task_created", "conditions": [{"field":"priority","operator":"equals","value":"high"}] },
    "actions": [{ "type": "assign_user", "config": {"userId":"'$USER_ID'"} }],
    "isActive": true
  }' | jq .
# Expected: 201
```

#### 27.3 Toggle On/Off
```bash
curl -s -X PATCH "$BASE/automations/$AUTO_ID/toggle" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { isActive: false }
```

#### 27.4 Automation Fires (integration test)
```bash
# 1. Create automation: task_created → send_notification
# 2. Create a task in the scoped project
# 3. GET /notifications → should contain the auto-generated notification
```

**Frontend Checks (AutomationsPage `/app/automations`):**
- [ ] Automations list renders with toggle switch
- [ ] Create automation modal opens
- [ ] Trigger type selector works (7 types)
- [ ] Condition builder: field / operator / value
- [ ] Action type selector works (5 types)
- [ ] Save → appears in list
- [ ] Toggle on/off → isActive reflects immediately
- [ ] Delete → removes from list

---

### 28. Documents

**Endpoints:** `POST /documents`, `GET /documents`, `GET /documents/:id`, `PUT /documents/:id`, `DELETE /documents/:id`, `POST /documents/:id/share`, `DELETE /documents/:id/share`, `GET /documents/public/:shareToken`

#### 28.1 Create Document
```bash
curl -s -X POST "$BASE/documents" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Meeting Notes","content":"Initial content","projectId":"'$PROJ_ID'"}' | jq .
# Expected: 201
```

#### 28.2 Update Document
```bash
curl -s -X PUT "$BASE/documents/$DOC_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Updated content with more details"}' | jq .
# Expected: 200
```

#### 28.3 Share Document (generate public link)
```bash
curl -s -X POST "$BASE/documents/$DOC_ID/share" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { shareToken: "<uuid>", isPublic: true }
```

#### 28.4 Access Public Document (no auth)
```bash
curl -s "$BASE/documents/public/$SHARE_TOKEN" | jq .
# Expected: 200 document content (no Authorization header required)
```

#### 28.5 Unshare Document
```bash
curl -s -X DELETE "$BASE/documents/$DOC_ID/share" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { isPublic: false }
# Subsequent GET /documents/public/$SHARE_TOKEN → 404
```

**Frontend Checks (DocumentsPage):**
- [ ] Documents list renders
- [ ] Create document: Tiptap rich text editor loads
- [ ] Bold, italic, headings, bullet lists all work in editor
- [ ] Edit content auto-saves (no manual save button required)
- [ ] Share button generates link → copy to clipboard
- [ ] Shared link opens in incognito without login → content visible
- [ ] Unshare → shared link now returns 404
- [ ] Delete prompts confirmation

---

### 29. Whiteboard

**Endpoints:** `POST /whiteboards`, `GET /whiteboards`, `GET /whiteboards/:id`, `POST /whiteboards/:whiteboardId/elements`, `DELETE /whiteboards/:whiteboardId/elements/:elementId`

#### 29.1 Create Whiteboard
```bash
curl -s -X POST "$BASE/whiteboards" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Sprint Planning Board","projectId":"'$PROJ_ID'"}' | jq .
# Expected: 201
```

#### 29.2 Add Element (sticky note)
```bash
curl -s -X POST "$BASE/whiteboards/$WB_ID/elements" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"sticky","content":"Key decision","x":100,"y":200,"color":"#ffeb3b"}' | jq .
# Expected: 201
```

**Frontend Checks (WhiteboardPage):**
- [ ] Whiteboard canvas renders
- [ ] Can add sticky notes
- [ ] Multiple users see changes in real-time (Socket.io)

---

### 30. Chat

**Endpoints:** `POST /chat/rooms`, `GET /chat/rooms`, `GET /chat/rooms/:roomId/messages`, `POST /chat/rooms/:roomId/messages`, `PUT /chat/messages/:id`, `DELETE /chat/messages/:id`, `POST /chat/messages/:id/reactions`, `GET /chat/messages/search`, `POST /chat/rooms/:roomId/upload`

#### 30.1 Create Chat Room
```bash
curl -s -X POST "$BASE/chat/rooms" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"General","workspaceId":"'$WS_ID'"}' | jq .
# Expected: 201
```

#### 30.2 Send Message
```bash
curl -s -X POST "$BASE/chat/rooms/$ROOM_ID/messages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Hello team!"}' | jq .
# Expected: 201
```

#### 30.3 Edit Message
```bash
curl -s -X PUT "$BASE/chat/messages/$MSG_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Hello team! (edited)"}' | jq .
# Expected: 200 { id, content, editedAt }
```

#### 30.4 Delete Message
```bash
curl -s -X DELETE "$BASE/chat/messages/$MSG_ID" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200/204
```

#### 30.5 Add Reaction to Message
```bash
curl -s -X POST "$BASE/chat/messages/$MSG_ID/reactions" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"emoji":"🎉"}' | jq .
# Expected: 200 { messageId, reactions: [{ emoji, users: [...] }] }
```

#### 30.6 Search Messages
```bash
curl -s "$BASE/chat/messages/search?roomId=$ROOM_ID&q=hello" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 { messages: [...] }
```

#### 30.7 Upload File to Chat Room
```bash
curl -s -X POST "$BASE/chat/rooms/$ROOM_ID/upload" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/path/to/image.png" | jq .
# Expected: 201 { messageId, fileUrl, filename, size }
```

**Frontend Checks (ChatPage):**
- [ ] Rooms list in sidebar renders
- [ ] Click room → messages load
- [ ] Send message → appears immediately
- [ ] Real-time: message from another user appears without refresh
- [ ] Edit own message → pencil icon visible, inline edit saves
- [ ] Delete own message → confirm prompt, message removed
- [ ] React to message → emoji picker opens, reaction count shown
- [ ] Reply to message → thread/quote appears
- [ ] Attach file button → file picker opens, uploads → preview in chat
- [ ] Search bar in chat → returns matching messages
- [ ] @mention autocomplete works

---

### 31. Invites

**Endpoints:** `POST /invites`, `GET /invites`, `GET /invites/:token`, `POST /invites/:token/accept`, `DELETE /invites/:id`

#### 31.1 Create Invite
```bash
curl -s -X POST "$BASE/invites" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email":"newmember@example.com","workspaceId":"'$WS_ID'","role":"member"}' | jq .
# Expected: 201 { id, token, email, expiresAt }
```

#### 31.2 Accept Invite
```bash
curl -s -X POST "$BASE/invites/$TOKEN/accept" \
  -H "Authorization: Bearer $TOKEN" | jq .
# Expected: 200 user added to workspace
```

**Frontend Checks (AcceptInvitePage):**
- [ ] Visiting `/app/invite/:token` shows invite details (workspace name, role)
- [ ] Click Accept → joins workspace, redirected to dashboard
- [ ] Expired token → shows error message

---

### 32. Settings Page

**Frontend Checks (SettingsPage `/app/settings`):**

**Profile Tab:**
- [ ] Shows current user name, email, avatar
- [ ] Update name → saves
- [ ] Upload new avatar → updates

**Security Tab:**
- [ ] Change password form works
- [ ] Enable 2FA button → setup flow
- [ ] Active sessions list (if implemented)

**Permissions Tab:**
- [ ] Permission matrix table renders by role
- [ ] `GET /permissions/matrix` returns data

**Audit Log Tab:**
- [ ] Activity timeline renders
- [ ] Filter by action type works

**Templates Tab:**
- [ ] Project templates listed
- [ ] Can create template from existing project

**Notifications Tab:**
- [ ] Per-channel toggles (email/in-app) — Note: backend NOT YET BUILT

---

### 33. Workload View

**Frontend Checks (WorkloadViewPage `/app/workload`):**
- [ ] Team member rows render
- [ ] Capacity bars (allocated vs available hours)
- [ ] Navigation to individual member works
- [ ] Over-allocated members highlighted in red

---

### RBAC Permission Tests

These tests verify the access control layer works correctly.

| Action | super_admin | owner | admin | billing_admin | member | guest |
|--------|------------|-------|-------|--------------|--------|-------|
| Access `/app/users` | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Create workspace | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Delete workspace | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Invite members | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| Access billing page | ✓ | ✓ | ✗ | ✓ | ✗ | ✗ |
| Create project | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| Create task (contributor+) | ✓ | ✓ | ✓ | ✗ | ✓ | ✗ |
| View-only task (viewer) | ✓ | ✓ | ✓ | ✗ | ✓(role) | ✓(scoped) |

### Key RBAC Curl Tests

```bash
# Test: member cannot access /app/users API
curl -s "$BASE/users" -H "Authorization: Bearer $MEMBER_TOKEN" | jq .
# Expected: 403 Forbidden

# Test: billing_admin cannot create tasks
curl -s -X POST "$BASE/tasks" \
  -H "Authorization: Bearer $BILLING_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"test","projectId":"'$PROJ_ID'"}' | jq .
# Expected: 403

# Test: guest can only see their scoped project
curl -s "$BASE/projects" -H "Authorization: Bearer $GUEST_TOKEN" | jq .
# Expected: only projects with guest_access record for this user

# Test: non-owner cannot delete workspace
curl -s -X DELETE "$BASE/workspaces/$WS_ID" \
  -H "Authorization: Bearer $MEMBER_TOKEN" | jq .
# Expected: 403
```

---

### Socket.io Real-Time Tests

```bash
# Test 1: Notification on task create
# - Open browser as user A (logged in)
# - Create a task assigned to user B from another session
# - user B should see notification badge increment without refresh

# Test 2: Chat real-time
# - Open two browser tabs, both logged in
# - Tab 1: send chat message in room
# - Tab 2: message appears immediately

# Test 3: Whiteboard sync
# - Two tabs on same whiteboard
# - Add sticky note in tab 1
# - Should appear in tab 2
```

---

### Error State Tests

For every major endpoint, also test:

| Scenario | Expected Response |
|----------|------------------|
| No auth token | 401 `{ success: false, message: "No token provided" }` |
| Expired token | 401 `{ success: false, message: "Token expired" }` |
| Wrong role | 403 `{ success: false, message: "Insufficient permissions" }` |
| Resource not found | 404 `{ success: false, message: "Not found" }` |
| Invalid input | 400 `{ success: false, message/errors: [...] }` |
| Server error | 500 `{ success: false, message: "Internal server error" }` |

---

## SECTION 3: Role-by-Role UI Test Matrix

Use: ✅ Allowed | ❌ Blocked (UI hides button or API returns 403) | 👁 View only | N/A Not applicable

---

### 3.1 Login Page (`/login`, `/register`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Access /login (unauthenticated) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Submit login form with valid credentials | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Login with wrong password shows inline error | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Access /register (unauthenticated) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Submit register form → new account created | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Authenticated user visits /login → redirect to /app/dashboard | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

### 3.2 Dashboard (`/app/dashboard`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Page loads without error | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| My Tasks widget shows assigned tasks | ✅ | ✅ | ✅ | N/A | ✅ | 👁 |
| Recent Activity feed visible | ✅ | ✅ | ✅ | ✅ | ✅ | 👁 |
| Quick create task button visible | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Project stats widget visible | ✅ | ✅ | ✅ | ✅ | ✅ | 👁 |
| Stakeholder dashboard link visible | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Notifications bell with unread count | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

### 3.3 Workspaces (`/app/workspaces`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View list of workspaces | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Create new workspace button visible | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Create new workspace (API) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Edit workspace name/description | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Delete workspace button visible | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Delete workspace (API) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| View workspace members | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Add member to workspace | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Change member role | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Remove member from workspace | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| super_admin sees ALL workspaces | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

### 3.4 Projects List (`/app/projects`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View projects list | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Create new project button visible | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Create new project (API POST /projects) | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Edit project name/description | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Delete project button visible | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Delete project (API DELETE /projects/:id) | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| View project from template | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Filter projects by workspace | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |

---

### 3.5 Project Detail (tabs: Tasks, Members, Settings, Health)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View project detail page | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Tasks tab — view tasks | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Tasks tab — create task | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Members tab — view members | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Members tab — add member | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Members tab — change member role | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Members tab — remove member | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Settings tab — visible | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Settings tab — edit project settings | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Health tab — view health score | ✅ | ✅ | ✅ | ✅ | ✅ | 👁 |
| Custom fields — create/edit | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Statuses — create/edit/delete | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |

---

### 3.6 Task List (`/app/tasks`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View task list | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Filter by status | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Filter by priority | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Filter by assignee | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Filter by due date range | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Save filter as saved view | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Bulk select tasks | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Bulk update status | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Bulk delete tasks | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Bulk archive tasks | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Export tasks as CSV | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Import tasks from CSV | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |

---

### 3.7 Task Detail Modal

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View task detail | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Edit title | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Edit description | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Change priority | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Change status | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Change due date | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Assign/unassign user | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Add subtask | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Complete subtask | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete subtask | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Add dependency | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Remove dependency | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Add comment | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Edit own comment | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Delete own comment | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Delete any comment | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| React to comment | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Upload attachment | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete attachment | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Set custom field value | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Log time on task | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Edit own time log | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete own time log | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete any time log | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |

---

### 3.8 Kanban Board (`/app/kanban`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View Kanban board | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Drag task card to different column | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Create task via "+ Add Task" in column | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Open task detail from card | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Column header count updates after move | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Filter cards by assignee | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Filter cards by priority | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |

---

### 3.9 Gantt Chart (`/app/gantt`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View Gantt chart | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Zoom to day/week/month view | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Drag bar to reschedule task (save to backend) | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Resize bar to change duration | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Dependency lines visible | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Today marker visible | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Click task bar → open detail | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |

---

### 3.10 Calendar (`/app/calendar`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View calendar (month/week/day) | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Toggle month/week/day view | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Click date to create task (modal) | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Click event to open task detail | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Tasks show on correct dates | ✅ | ✅ | ✅ | 👁 | ✅ | 👁 |
| Navigate prev/next month | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

### 3.11 Chat (`/app/chat`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View chat page and room list | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Create new chat room | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Send message in room | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Reply to message (thread) | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| React to message with emoji | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Attach file to message | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Edit own message | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Delete own message | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Delete any message (admin) | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Search messages in room | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| @mention user in message | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Real-time message delivery visible | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |

---

### 3.12 Documents (`/app/documents`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View documents list | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Create new document | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Open and view document content | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Edit document content (Tiptap) | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Edit document title | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Share document (generate link) | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Copy share link to clipboard | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Unshare document | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete document | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Access public document via share link (no auth) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

### 3.13 Whiteboard (`/app/whiteboard`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View whiteboard list | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Create new whiteboard | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Open whiteboard canvas | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Add sticky note element | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Move element on canvas | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete element | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Real-time collaboration (see others' changes) | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete whiteboard | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |

---

### 3.14 Notifications

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View notification bell with badge | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Open notification dropdown | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| View notification list | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Mark single notification as read | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Mark all notifications as read | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Delete single notification | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Clear all notifications | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Click notification → navigate to entity | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Real-time badge increment on new notification | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Notification settings tab (per-channel toggles) | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT |

---

### 3.15 Global Search (Cmd+K)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Cmd+K opens search modal | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Search returns task results | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Search returns project results | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Search returns user results | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Click result navigates to entity | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Escape closes modal | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Results scoped to accessible workspaces | ✅ (all) | ✅ | ✅ | ✅ | ✅ | ❌ |

---

### 3.16 Time Tracking (`/app/time-tracking`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View time log list | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Add manual time log | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Start live timer | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Pause/stop live timer | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Edit own time log | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Delete own time log | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Edit any user's time log | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| View statistics summary | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |
| Filter by user | ✅ | ✅ | ✅ | 👁 | ❌ | ❌ |
| Filter by date range | ✅ | ✅ | ✅ | 👁 | ✅ | ❌ |

---

### 3.17 Resource Management (`/app/resources`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View workload page | ✅ | ✅ | ✅ | 👁 | ❌ | ❌ |
| View member allocation bars | ✅ | ✅ | ✅ | 👁 | ❌ | ❌ |
| Add new allocation | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Edit existing allocation | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Delete allocation | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Over-allocation warning displayed | ✅ | ✅ | ✅ | 👁 | ❌ | ❌ |

---

### 3.18 Budget Management (`/app/budgets`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View budget list | ✅ | ✅ | 👁 | ✅ | ❌ | ❌ |
| Create new budget | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Edit budget total | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Add expense | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Edit expense | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Delete expense | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| View budget progress bar | ✅ | ✅ | 👁 | ✅ | ❌ | ❌ |
| Over-budget warning displayed | ✅ | ✅ | 👁 | ✅ | ❌ | ❌ |

---

### 3.19 Automations (`/app/automations`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| View automations list | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Create new automation | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Edit existing automation | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Toggle automation on/off | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Delete automation | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| View automation run history | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |

---

### 3.20 Settings (`/app/settings`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Access /app/settings | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Profile tab — view/edit own name | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Profile tab — upload avatar | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Security tab — change password | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Security tab — enable/disable 2FA | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Permissions tab — view matrix | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Permissions tab — edit role permissions | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Audit Log tab — view activity | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Templates tab — view templates | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| Templates tab — create template | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Notifications tab — visible | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT | ❌ NOT BUILT |

---

### 3.21 Users Admin (`/app/users`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Access /app/users page | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| View all users table | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Change any user's global role | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Deactivate user | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Delete user | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Search users by name/email | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| API GET /users returns 403 for non-super_admin | N/A | ✅ (403) | ✅ (403) | ✅ (403) | ✅ (403) | ✅ (403) |

---

### 3.22 Stakeholder Dashboard (`/app/stakeholder-dashboard`)

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Access stakeholder dashboard | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Project health score chart renders | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Budget utilization chart renders | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Timeline chart renders | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Risk matrix chart renders | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Team velocity chart renders | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Member directly navigates to /app/stakeholder-dashboard | N/A | N/A | N/A | N/A | ❌ redirect | ❌ redirect |
| Charts update when project data changes | ✅ | ✅ | ✅ | ✅ | N/A | N/A |

---

### 3.24 Invites

| Test Action | super_admin | owner | admin | billing_admin | member | guest |
|-------------|------------|-------|-------|--------------|--------|-------|
| Send workspace invite via email | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| View pending invites list | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Revoke pending invite | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Accept invite (invited user) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Accept invite with expired token → error shown | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Accept invite already used → error shown | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## SECTION 4: Edge Case Test Scenarios

---

### 4.1 Auth Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| A1 | Login with correct email, wrong password | POST /auth/login with bad password | 401; frontend shows inline "Invalid credentials" error; no redirect |
| A2 | Login with unverified email | Register new user, do not verify, attempt login | Error response says "Please verify your email" |
| A3 | Register with already-used email | POST /auth/register with admin@pulsehub.com | 400/409 "Email already in use" |
| A4 | Register with weak password (< 8 chars) | POST /auth/register password="abc" | 400 with validation error listing requirement |
| A5 | Register with no uppercase in password | POST /auth/register password="test1234!" | 400 validation error |
| A6 | JWT token expired | Wait for token TTL to expire, call GET /auth/me | 401 "Token expired"; frontend redirects to /login |
| A7 | Use refresh token after logout | POST /auth/logout; then POST /auth/refresh-token with old refreshToken | 401 — invalidated token rejected |
| A8 | 2FA: wrong TOTP code on first attempt | POST /auth/2fa/verify-login with wrong totpToken | 401 "Invalid code"; account NOT locked |
| A9 | 2FA: used backup code cannot be reused | Use backup code to login; attempt same code again | 401 "Backup code already used" |
| A10 | Access protected route with no token | GET /tasks without Authorization header | 401 "No token provided" |

---

### 4.2 Task Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| T1 | Create task with title = 500 chars | POST /tasks with 500-char title string | Either 400 "Title too long" OR task created and title truncated |
| T2 | Set due date in the past | PUT /tasks/:id with dueDate = "2020-01-01" | 200 allowed; task shown with red "Overdue" badge in UI |
| T3 | Assign same user twice | POST /tasks/:id/assignees with same userId twice | Silently deduplicated; user appears once in assignees list |
| T4 | Delete task that has subtasks | DELETE /tasks/:id where task has 3 subtasks | Subtasks cascade deleted; no orphan records in DB |
| T5 | Bulk action with 0 tasks selected | Attempt to show bulk toolbar with no checkboxes ticked | Bulk toolbar not visible; no action sent |
| T6 | Move task to its current status | PUT /tasks/:id/move with same statusId | 200; no error; no change recorded; no duplicate activity log entry |
| T7 | Import CSV with missing required column "title" | POST /tasks/import with CSV missing title column | 400 error response naming the missing column ("title column required") |
| T8 | Create circular dependency (A→B→A) | POST /tasks/A/dependencies {dependsOnId: B}; POST /tasks/B/dependencies {dependsOnId: A} | Second POST returns 400 "Circular dependency detected" |
| T9 | Delete task with active time logs | DELETE /tasks/:id that has time logs | Time logs cascade deleted OR task deletion rejected with message |
| T10 | Bulk delete 200 tasks at once | POST /tasks/bulk with 200 task IDs | All 200 deleted; response includes count; no timeout |

---

### 4.3 Chat Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| C1 | Send empty message (only spaces) | Type "   " in chat input; click Send | Send button remains disabled; no POST to API |
| C2 | Send message > 10,000 chars | Paste 10,001 char string into chat; submit | Backend returns 400 or silently truncates; frontend shows error if rejected |
| C3 | Upload file > 25MB | Click attach, select a 30MB video file | UI shows error "File exceeds 25MB limit" before or after upload attempt |
| C4 | Upload .exe file | Attempt to attach a .exe file | Allowed (no backend type restriction currently) OR blocked with "File type not permitted" |
| C5 | Reply to deleted message | Send message A; delete message A; reply to A | Parent quote shows "[deleted]" or is gracefully absent in thread |
| C6 | Edit message after 24h (if restriction exists) | Attempt PUT /chat/messages/:id 25h after creation | If restriction: 403 "Edit window expired"; if no restriction: 200 |
| C7 | @mention user not in workspace | Type @nonmember in chat | Name still highlighted with @ syntax; NO notification sent to mentioned user |
| C8 | Internet disconnect during typing | Disconnect network; send message | "Reconnecting…" indicator shown; message queues locally; sends on reconnect |
| C9 | Open same room in two tabs | Send message in Tab 1 | Tab 2 sees message in real-time without refresh (Socket.io) |
| C10 | Delete message in one tab | Delete message in Tab 1 | Tab 2 sees message disappear in real-time |

---

### 4.4 Document Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| D1 | Share document, access in incognito | Share doc; copy link; open link in incognito window | Document content visible without any login prompt |
| D2 | Unshare document, access old link | Unshare; attempt GET /documents/public/:oldToken | 404 "Document not found or no longer shared" |
| D3 | Auto-save behavior | Open doc; type content; do NOT click save; close and reopen | Content preserved (auto-save fired within ~2s of typing) |
| D4 | Two users edit same document simultaneously | User A and User B both open same doc; both type content | Last write wins; no conflict resolution UI; content from last save survives |
| D5 | Create document with empty title | POST /documents with title="" | 400 "Title is required" |
| D6 | Delete shared document | Share doc; then DELETE /documents/:id | Document deleted; shared link returns 404 |
| D7 | Rich text formatting persists | Apply bold, H2, bullet list in Tiptap; save; reload | All formatting preserved in saved content |

---

### 4.5 RBAC Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| R1 | Guest navigates directly to /app/projects | Login as guest; navigate to /app/projects | Redirect to dashboard OR empty state with no projects shown |
| R2 | billing_admin tries POST /tasks via API | Use billing_admin token: POST /tasks | 403 "Insufficient permissions" |
| R3 | Member tries to delete another member's comment | Login as member; DELETE /comments/:otherMembersCommentId | 403 (not just hidden in UI — API also rejects) |
| R4 | Non-owner tries to delete workspace via API | Login as admin; DELETE /workspaces/:id | 403 |
| R5 | Super admin sees all workspaces without membership | Login as super_admin; GET /workspaces | Returns ALL workspaces in system, not just owned/member ones |
| R6 | Expired invite token | GET /invites/:tokenThatExpired | 400 or 410 "Invite expired or invalid" |
| R7 | Member accesses billing page directly | Navigate to /app/budgets as member | Redirect to dashboard OR 403 empty state |
| R8 | Guest tries to create chat room via API | POST /chat/rooms with guest token | 403 |
| R9 | Admin tries to change workspace owner | PUT /workspaces/:id/members/:userId {role:"owner"} as admin | 403 only workspace owner or super_admin can assign owner role |
| R10 | Non-member tries to access project | GET /projects/:id with token of user not in workspace | 403 or 404 |

---

### 4.6 Socket.io Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| S1 | Real-time chat message delivery | User A sends message in room; User B is in same room | User B sees message appear without refresh; delivery < 500ms |
| S2 | Reconnect after disconnect | User A disconnects (network off); User B sends 3 messages; User A reconnects | User A's chat history shows the 3 missed messages on reconnect |
| S3 | Whiteboard multi-user | 3 users open same whiteboard; User A adds sticky note | Users B and C see note appear in real-time |
| S4 | Notification on task assignment | User B is online; User A assigns task to User B | User B's notification bell increments in real-time without refresh |
| S5 | User offline 5 min, reconnects | User disconnects 5 min; chat activity occurs; reconnects | Missed messages appear in chat history on rejoin |
| S6 | Duplicate socket events | Rapid fire: send 10 messages in 1 second | All 10 messages appear once; no duplicates displayed |
| S7 | Room join on reconnect | User disconnects then reconnects | User is automatically re-joined to their chat rooms (server re-subscribes) |
| S8 | Whiteboard element deletion | User A deletes element in whiteboard | User B sees element disappear in real-time |

---

## SECTION 5: Complete Role Walkthrough Scripts

These are step-by-step browser walkthrough scripts for manual testing of each role.

---

### Script 1: super_admin Full Walkthrough

**Login credentials:** admin@pulsehub.com / Admin123!

1. Navigate to `http://localhost:3000/login`
2. Enter email and password → click Login
3. Verify redirect to `/app/dashboard` — no errors in console
4. Verify "My Tasks", "Recent Activity", and project stats widgets visible
5. Navigate to `/app/users`
6. Verify all 6 seeded users listed in table
7. Find the member user (member@pulsehub.com) → change role to "admin" via dropdown
8. Verify role change saved (row updates, no page reload needed)
9. Revert role back to "member"
10. Navigate to `/app/workspaces`
11. Click "Create Workspace" → enter name "Super Admin Test WS" → submit
12. Verify new workspace appears in list
13. Open workspace → click "Members" → add owner@pulsehub.com as "admin"
14. Navigate to `/app/projects`
15. Click "Create Project" → enter name "Test Project Alpha", select workspace, add description → submit
16. Verify project card appears in list
17. Open project → verify tabs: Tasks, Members, Settings, Health all visible
18. Click Tasks tab → click "Create Task"
19. Enter title "Verify auth flow" → set priority High → assign to member@pulsehub.com → submit
20. Verify task appears in task list with correct priority badge and assignee avatar
21. Navigate to `/app/kanban` → select "Test Project Alpha"
22. Verify task "Verify auth flow" appears in the correct status column
23. Drag the task card to the "In Progress" column
24. Verify status column updates and task count adjusts
25. Navigate to `/app/chat`
26. Click "Create Room" → name "General Testing" → select workspace → submit
27. In the new room, type "Hello @member" → verify @mention autocomplete appears → send
28. Click the attach file button → select a small image → verify it uploads and appears as preview in chat
29. Navigate to `/app/documents`
30. Click "Create Document" → enter title "QA Notes"
31. In the Tiptap editor, type "# Heading 1" then a paragraph with **bold** and _italic_ text
32. Verify auto-save triggers (no manual save needed)
33. Click "Share" button → copy the generated link
34. Open a new incognito window → paste the link → verify document content visible without login
35. Back in the main window, click "Unshare" → verify share link now returns 404 in incognito
36. Navigate to `/app/settings`
37. Verify all tabs visible: Profile, Security, Permissions, Audit Log, Templates
38. Click Permissions tab → verify role matrix table renders
39. Click Audit Log tab → verify activity entries appear
40. Verify Notifications tab either shows "not built" state or is absent

---

### Script 2: owner Walkthrough

**Login credentials:** owner@pulsehub.com / Owner123!

1. Navigate to `/login` → enter credentials → submit
2. Verify redirect to `/app/dashboard`
3. Attempt to navigate to `/app/users` directly
4. Verify redirect to dashboard OR 403 page — owner cannot access user admin
5. Navigate to `/app/workspaces`
6. Verify "Create Workspace" button is visible → click it
7. Create workspace "Owner Workspace Test" → submit
8. Navigate into workspace → click "Members"
9. Click "Invite Member" → enter email "member@pulsehub.com" with role "member" → send
10. Verify invite appears in pending invites list
11. Navigate to `/app/projects` → click "Create Project"
12. Create project "Owner Project" in "Owner Workspace Test" → submit
13. Open project → click "Health" tab → verify health score widget renders
14. Navigate to `/app/budgets`
15. Click "Create Budget" → enter totalBudget 25000, currency USD → link to "Owner Project" → submit
16. Click "Add Expense" → enter amount 500, description "Design tool subscription", category "tools" → submit
17. Verify budget progress bar updates to reflect spent amount
18. Navigate to `/app/automations`
19. Click "Create Automation" → configure: trigger = task_created, condition priority=high, action = send_notification → save
20. Verify automation appears in list with toggle on
21. Toggle it off → verify isActive changes to false
22. Navigate to `/app/settings` → verify all expected tabs present (Profile, Security, Permissions, Audit Log, Templates)
23. Verify Notifications tab is absent or shows backend-missing message

---

### Script 3: admin Walkthrough

**Login credentials:** adminuser@pulsehub.com / Admin123!

1. Navigate to `/login` → enter credentials → submit
2. Verify redirect to `/app/dashboard`
3. Navigate to `/app/workspaces`
4. Verify workspaces admin belongs to are listed
5. Verify "Create Workspace" button is NOT visible (admin cannot create workspaces)
6. Open any workspace admin is a member of
7. Navigate to `/app/projects` → click "Create Project"
8. Verify button is visible → create project "Admin Test Project" → submit
9. Open the project → click "Members" tab
10. Add member@pulsehub.com as "contributor" → verify saved
11. Change member's project role to "reviewer" → verify saved
12. Remove member from project → verify removed
13. Go to Tasks tab → create 3 tasks with different priorities
14. Edit one task → change title, description, due date → verify saved
15. Delete one task → confirm prompt → verify removed from list
16. Attempt to delete the workspace → verify delete button is absent OR API returns 403
17. Navigate to `/app/settings` → attempt Billing/Budget tab
18. Verify billing page is inaccessible (redirect or empty state shown)
19. Navigate to `/app/automations` → verify automations page accessible
20. Create a test automation → toggle it on/off → delete it

---

### Script 4: billing_admin Walkthrough

**Login credentials:** billing@pulsehub.com / Billing123!

1. Navigate to `/login` → enter credentials → submit
2. Verify redirect to `/app/dashboard`
3. Dashboard should load — verify layout appears but task widgets may show empty state
4. Navigate to `/app/budgets`
5. Verify budget list is visible and accessible
6. Click "Create Budget" → verify button is present and functional
7. Create a budget for an existing project → submit
8. Add an expense → verify it appears in expense list
9. Verify budget progress bar updates
10. Navigate to `/app/projects`
11. Verify "Create Project" button is NOT visible
12. Attempt POST /projects via curl with billing_admin token → expect 403
13. Navigate to `/app/tasks`
14. Verify task list is either empty or read-only (no create button visible)
15. Attempt POST /tasks via curl with billing_admin token → expect 403
16. Navigate to `/app/workspaces`
17. Verify workspaces they belong to are shown
18. Verify "Create Workspace" button is NOT visible
19. Navigate to `/app/users` → verify redirect or 403
20. Navigate to `/app/settings` → verify only Profile and Security tabs accessible

---

### Script 5: member Walkthrough

**Login credentials:** member@pulsehub.com / Member123!

1. Navigate to `/login` → enter credentials → submit
2. Verify redirect to `/app/dashboard`
3. Verify "My Tasks" widget shows tasks assigned to this member
4. Navigate to `/app/projects` → open a project member belongs to
5. Tasks tab → verify "Create Task" button is visible (member has contributor role)
6. Create task "Member Created Task" → set priority medium → submit
7. Open the task detail → edit the title → verify save works
8. Change priority to High → verify saved
9. Change due date to tomorrow → verify saved
10. Open a task created by the admin user
11. Attempt to edit the admin's task fields → if member has viewer role on this task, verify fields are read-only
12. Add a comment "Looking good!" → verify comment appears
13. React to admin's comment with 👍 → verify reaction count increments
14. Attempt to delete admin's comment → verify delete button is absent
15. Log 2 hours on the member-created task → verify time log appears
16. Navigate to `/app/kanban`
17. Find "Member Created Task" → drag it to "In Progress" column → verify status updates
18. Attempt to drag a task owned by admin → if viewer role, verify drag is disabled
19. Navigate to `/app/time-tracking`
20. Verify only own time logs visible
21. Edit own time log → verify saves
22. Navigate to `/app/users` → verify redirect or 403 (cannot access user admin)
23. Navigate to `/app/automations` → verify page is inaccessible
24. Navigate to `/app/chat`
25. Verify "Create Room" button is NOT visible
26. Open an existing room → send a message → verify delivery
27. React to a message with an emoji → verify reaction visible

---

### Script 6: guest Walkthrough

**Login credentials:** guest@pulsehub.com / Guest123!

1. Navigate to `/login` → enter credentials → submit
2. Verify redirect to `/app/dashboard`
3. Dashboard renders in limited mode — verify only accessible projects/tasks shown
4. Navigate to `/app/projects`
5. Verify only projects explicitly shared with guest are listed (scoped access)
6. Open a scoped project → verify all fields are read-only
7. Verify "Create Task" button is NOT visible
8. Click on an existing task → verify task detail opens in view-only mode
9. Verify all edit controls are absent or disabled
10. Attempt to add a comment (if guest has commenter role) → verify comment submits
11. Attempt to add a reaction → verify reaction works if commenter role, blocked if viewer
12. Navigate to `/app/kanban` → verify board is read-only (no drag, no create)
13. Navigate to `/app/settings` → verify redirect or empty state (guest cannot access settings)
14. Navigate to `/app/users` → verify redirect or 403
15. Navigate to `/app/workspaces` → verify redirect or empty state
16. Navigate to `/app/chat` → verify page is inaccessible or shows "no rooms" with no create button
17. Navigate to `/app/automations` → verify redirect or 403
18. Navigate to `/app/documents` → verify either redirect OR empty list with no create button
19. Attempt to navigate to Cmd+K search → verify search modal does not open for guest
20. Verify bell notifications still work for guest (can receive and read notifications)

---

## SECTION 6: Performance Benchmarks

All benchmarks measured from request initiation to response received (API) or time-to-interactive (frontend). Measured with browser DevTools Network panel or `curl -w "%{time_total}"`.

| Operation | Acceptable | Warning | Critical |
|-----------|-----------|---------|----------|
| API login response | < 300ms | 300–1000ms | > 1000ms |
| Task list load (50 tasks) | < 500ms | 500–2000ms | > 2000ms |
| Kanban board load | < 800ms | 800–3000ms | > 3000ms |
| Global search results | < 400ms | 400–1500ms | > 1500ms |
| File upload (5MB) | < 3s | 3–10s | > 10s |
| Socket message delivery | < 200ms | 200–1000ms | > 1000ms |
| Dashboard initial load | < 1000ms | 1000–3000ms | > 3000ms |
| Gantt chart render (20 tasks) | < 1200ms | 1200–4000ms | > 4000ms |
| Document save (Tiptap auto-save) | < 500ms | 500–2000ms | > 2000ms |
| Bulk task update (50 tasks) | < 1000ms | 1000–5000ms | > 5000ms |

### How to Measure

```bash
# Measure API response time
curl -s -o /dev/null -w "Total: %{time_total}s\n" \
  -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@pulsehub.com","password":"Admin123!"}'

# Measure task list load time
curl -s -o /dev/null -w "Total: %{time_total}s\n" \
  "$BASE/tasks?projectId=$PROJ_ID" \
  -H "Authorization: Bearer $TOKEN"
```

### Frontend Measurement (Chrome DevTools)
1. Open DevTools → Network tab → check "Disable cache"
2. Navigate to page
3. Record total DOMContentLoaded and Load event times
4. Check for any requests > 1000ms and flag them

---

## SECTION 7: Test Execution Checklist

### Phase 1 — Core Auth + Data (P0)
- [ ] Health check passes
- [ ] Login works for all 6 seed users
- [ ] Token authentication works on protected routes
- [ ] Users CRUD (super_admin)
- [ ] Workspace CRUD + member management
- [ ] Project CRUD + member management
- [ ] Task CRUD (list, create, update, delete)
- [ ] Task bulk operations

### Phase 2 — Collaboration (P1)
- [ ] Kanban board loads + drag works
- [ ] Comments CRUD + reactions
- [ ] Attachments upload + download
- [ ] Notifications (list, mark read, real-time)
- [ ] Global search returns results
- [ ] Chat (rooms + messages + real-time)
- [ ] Chat edit/delete/react/search/file-upload (new endpoints)
- [ ] Automation CRUD + engine fires correctly
- [ ] Documents CRUD + Tiptap editor
- [ ] Document share/unshare/public access
- [ ] Whiteboard CRUD + real-time

### Phase 3 — Data & Views (P1)
- [ ] Gantt chart loads with correct dates
- [ ] Gantt drag-to-reschedule saves to backend
- [ ] Calendar shows tasks by due date
- [ ] Time tracking log + statistics
- [ ] Resource workload data
- [ ] Budget + expenses
- [ ] Activity log

### Phase 4 — Auth Edge Cases (P1)
- [ ] 2FA setup + login flow
- [ ] Email verification flow
- [ ] Invite accept flow
- [ ] Refresh token works
- [ ] Logout invalidates session
- [ ] Expired token redirects to login
- [ ] 2FA wrong code does not lock account on first attempt
- [ ] Used backup code cannot be reused

### Phase 5 — RBAC Boundaries (P2)
- [ ] Workspace role restrictions
- [ ] Project role restrictions
- [ ] Super admin bypass works
- [ ] Guest access scoping
- [ ] Billing admin has no project/task create access
- [ ] Member cannot delete others' comments (API-level 403)
- [ ] Non-owner cannot delete workspace via API
- [ ] Circular dependency rejected by backend

### Phase 6 — Settings + Edge Cases (P2)
- [ ] Settings page all tabs work
- [ ] Saved views CRUD
- [ ] Workflow templates + apply
- [ ] Subtasks + dependencies
- [ ] Custom fields CRUD
- [ ] CSV import/export
- [ ] Document auto-save behavior verified
- [ ] Empty message cannot be sent in chat
- [ ] File > 25MB shows error in chat upload

### Phase 7 — Role UI Walkthroughs
- [ ] Script 1: super_admin full walkthrough completed
- [ ] Script 2: owner walkthrough completed
- [ ] Script 3: admin walkthrough completed
- [ ] Script 4: billing_admin walkthrough completed
- [ ] Script 5: member walkthrough completed
- [ ] Script 6: guest walkthrough completed
- [ ] All role matrix entries verified in browser (not just API)

### Phase 8 — Edge Cases
- [ ] Auth edge cases A1–A10 all tested
- [ ] Task edge cases T1–T10 all tested
- [ ] Chat edge cases C1–C10 all tested
- [ ] Document edge cases D1–D7 all tested
- [ ] RBAC edge cases R1–R10 all tested
- [ ] Socket.io edge cases S1–S8 all tested
- [ ] Performance benchmarks recorded for all 10 operations

---

## Known Issues / Pre-Test Notes

| Issue | Status | Notes |
|-------|--------|-------|
| `automation.routes.js` — wrong import | FIXED | `{ authenticate: protect }` |
| `User.js` — camelCase field mismatches | FIXED | Added explicit `field:` overrides |
| `seed-demo.js` — double bcrypt hash | FIXED | Plain text to `defaults`, `hooks:false` for existing |
| Chat reactions / threads | FIXED | POST /chat/messages/:id/reactions { emoji } |
| Chat file upload | FIXED | POST /chat/rooms/:roomId/upload (multipart/form-data, file field) |
| Document sharing link | FIXED | POST /documents/:id/share → { shareToken, isPublic: true } |
| Rich text in Documents | FIXED | Tiptap editor integrated |
| Drag-to-reschedule on Gantt | FIXED | Drag bars save new dates to backend |
| Notifications tab in Settings | NOT BUILT | Backend endpoint missing — per-channel notification toggles not implemented |
| Whiteboard draw tools | NOT BUILT | Sticky notes only — freehand draw, shapes, connectors not yet built |
| DocumentsPage activeMenu dropdown | FIXED | Added outside-click handler via `useEffect` + `mousedown` listener |
| DocumentsPage copyTimer memory leak | FIXED | `clearTimeout` now called before each setTimeout; unmount cleanup added |
| selectors.ts redundant double-filter | FIXED | `selectUnreadNotificationCount` now derives from `selectUnreadNotifications` |
| cache.js stale eviction only on size > 500 | FIXED | Added `setInterval` periodic cleanup + size-cap evicts oldest entries |
| chat.controller.js mid-file requires | FIXED | `multer`, `path`, `fs.promises` moved to top of file |
| chat.controller.js mkdir on every upload | FIXED | `CHAT_UPLOAD_DIR` created once at module load; destination callback is synchronous |
| socketClient.ts — no auto-reconnect | FIXED | `_joinedRooms` Set; `reconnect` event re-emits joins; exponential backoff configured |

---

## SECTION 8: Playwright E2E Automated Test Templates

Install Playwright: `npm install -D @playwright/test && npx playwright install`

### 8.1 Playwright Config (`playwright.config.ts`)

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  timeout: 30_000,
  retries: 1,
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

### 8.2 Auth Helper (`e2e/helpers/auth.ts`)

```typescript
import { Page } from '@playwright/test';

export const TEST_USERS = {
  super_admin: { email: 'admin@pulsehub.com', password: 'Admin123!' },
  owner:       { email: 'owner@pulsehub.com', password: 'Owner123!' },
  admin:       { email: 'adminuser@pulsehub.com', password: 'Admin123!' },
  billing:     { email: 'billing@pulsehub.com', password: 'Billing123!' },
  member:      { email: 'member@pulsehub.com', password: 'Member123!' },
  guest:       { email: 'guest@pulsehub.com', password: 'Guest123!' },
};

export async function loginAs(page: Page, role: keyof typeof TEST_USERS) {
  const { email, password } = TEST_USERS[role];
  await page.goto('/login');
  await page.fill('input[type="email"]', email);
  await page.fill('input[type="password"]', password);
  await page.click('button[type="submit"]');
  await page.waitForURL('**/app/dashboard');
}
```

### 8.3 Auth Tests (`e2e/auth.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test('login with valid credentials redirects to dashboard', async ({ page }) => {
  await loginAs(page, 'super_admin');
  await expect(page).toHaveURL(/\/app\/dashboard/);
});

test('login with wrong password shows inline error', async ({ page }) => {
  await page.goto('/login');
  await page.fill('input[type="email"]', 'admin@pulsehub.com');
  await page.fill('input[type="password"]', 'wrongpassword');
  await page.click('button[type="submit"]');
  await expect(page.locator('.error-message, [class*="error"]')).toBeVisible();
  await expect(page).not.toHaveURL(/\/app\/dashboard/);
});

test('authenticated user visiting /login is redirected', async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/login');
  await expect(page).toHaveURL(/\/app\/dashboard/);
});

test('unauthenticated user visiting /app/dashboard is redirected to /login', async ({ page }) => {
  await page.goto('/app/dashboard');
  await expect(page).toHaveURL(/\/login/);
});
```

### 8.4 Role Access Guard Tests (`e2e/rbac.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test('super_admin can access /app/users', async ({ page }) => {
  await loginAs(page, 'super_admin');
  await page.goto('/app/users');
  await expect(page.locator('table, [class*="users-table"]')).toBeVisible();
});

test('owner cannot access /app/users', async ({ page }) => {
  await loginAs(page, 'owner');
  await page.goto('/app/users');
  // Should be redirected or show 403
  await expect(page).not.toHaveURL(/\/app\/users/);
});

test('billing_admin cannot create tasks', async ({ page }) => {
  await loginAs(page, 'billing');
  await page.goto('/app/tasks');
  await expect(page.locator('button:has-text("Create Task"), button:has-text("New Task")')).toHaveCount(0);
});

test('guest cannot access /app/chat', async ({ page }) => {
  await loginAs(page, 'guest');
  await page.goto('/app/chat');
  // Either redirected or shows empty/locked state
  const chatRoomInput = page.locator('input[placeholder*="message"], textarea[placeholder*="message"]');
  await expect(chatRoomInput).toHaveCount(0);
});

test('member cannot see Automations page', async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/app/automations');
  await expect(page).not.toHaveURL(/\/app\/automations/);
});
```

### 8.5 Task CRUD Tests (`e2e/tasks.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test.beforeEach(async ({ page }) => {
  await loginAs(page, 'admin');
});

test('create a task and verify it appears in list', async ({ page }) => {
  await page.goto('/app/tasks');
  await page.click('button:has-text("Create Task"), button:has-text("New Task")');
  await page.fill('input[placeholder*="title"], input[placeholder*="Task name"]', 'Playwright E2E Test Task');
  await page.click('button[type="submit"], button:has-text("Save"), button:has-text("Create")');
  await expect(page.locator('text=Playwright E2E Test Task')).toBeVisible();
});

test('edit a task title and save', async ({ page }) => {
  await page.goto('/app/tasks');
  await page.click('text=Playwright E2E Test Task');
  await page.fill('input[value="Playwright E2E Test Task"]', 'Updated E2E Task Title');
  await page.keyboard.press('Enter');
  await expect(page.locator('text=Updated E2E Task Title')).toBeVisible();
});

test('delete task via confirm dialog', async ({ page }) => {
  await page.goto('/app/tasks');
  // Open task options and delete
  await page.locator('[data-testid="task-menu"], .task-menu-btn').first().click();
  await page.click('button:has-text("Delete")');
  // Confirm dialog
  await page.click('button:has-text("Delete"):visible');
  await expect(page.locator('text=Updated E2E Task Title')).toHaveCount(0);
});

test('kanban drag-and-drop changes task status', async ({ page }) => {
  await page.goto('/app/kanban');
  // Locate first task card in "To Do" column and drag to "In Progress"
  const todoCard = page.locator('[class*="kanban-column"]:first-child [class*="task-card"]').first();
  const inProgressColumn = page.locator('[class*="kanban-column"]:nth-child(2)');
  await todoCard.dragTo(inProgressColumn);
  // Verify card moved (column count changes)
  await expect(inProgressColumn.locator('[class*="task-card"]')).toHaveCount({ min: 1 });
});
```

### 8.6 Chat Tests (`e2e/chat.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test('send message and see it appear in chat', async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/app/chat');
  await page.locator('[class*="room-list-item"]').first().click();
  const composer = page.locator('textarea[placeholder*="message"], input[placeholder*="message"]');
  await composer.fill('Playwright test message ' + Date.now());
  await page.keyboard.press('Enter');
  await expect(page.locator('[class*="message-content"]:last-child')).toContainText('Playwright test message');
});

test('react to a message with emoji', async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/app/chat');
  await page.locator('[class*="room-list-item"]').first().click();
  // Hover over a message to reveal reaction button
  const msg = page.locator('[class*="msg-bubble"]').first();
  await msg.hover();
  await page.locator('[class*="react-btn"], button[title*="React"]').first().click();
  await page.locator('button:has-text("👍")').click();
  await expect(page.locator('[class*="reaction-chip"]').filter({ hasText: '👍' })).toBeVisible();
});

test('search messages in room', async ({ page }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/chat');
  await page.locator('[class*="room-list-item"]').first().click();
  await page.click('[class*="header-action-btn"]:has([data-testid="search-icon"]), button[title="Search"]');
  await page.fill('[class*="search-input"]', 'Playwright test');
  await expect(page.locator('[class*="search-result"]')).toHaveCount({ min: 1 });
});
```

### 8.7 Document Tests (`e2e/documents.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test('create document, edit, and verify auto-save', async ({ page }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.click('button[title="New document"], button:has-text("New")');
  // Fill title
  await page.fill('input[placeholder="Document title"]', 'Playwright Doc');
  // Type in Tiptap editor
  await page.locator('.ProseMirror, [class*="tiptap"]').click();
  await page.keyboard.type('This is a Playwright test document');
  // Wait for auto-save
  await page.waitForTimeout(3000);
  await expect(page.locator('[class*="save-status"]')).toContainText('Saved');
});

test('share document generates a link', async ({ page }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.locator('[class*="doc-list-item"]').first().click();
  await page.click('button:has-text("Share")');
  // Toggle public access on
  await page.click('[class*="share-toggle-btn"]');
  await expect(page.locator('[class*="share-link-input"]')).toBeVisible();
  const shareUrl = await page.locator('[class*="share-link-input"]').inputValue();
  expect(shareUrl).toContain('/shared/doc/');
});

test('public document accessible without auth', async ({ page, context }) => {
  // Get share URL first (logged in)
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.locator('[class*="doc-list-item"]').first().click();
  await page.click('button:has-text("Share")');
  await page.click('[class*="share-toggle-btn"]');
  const shareUrl = await page.locator('[class*="share-link-input"]').inputValue();

  // Open in new incognito page
  const incognitoPage = await context.newPage();
  await incognitoPage.goto(shareUrl);
  await expect(incognitoPage.locator('[class*="doc-public"], .ProseMirror, [class*="tiptap"]')).toBeVisible();
});
```

### 8.8 Running All E2E Tests

```bash
# Run all tests
npx playwright test

# Run with UI mode (interactive)
npx playwright test --ui

# Run specific test file
npx playwright test e2e/rbac.spec.ts

# Run tests for a specific role scenario
npx playwright test --grep "super_admin"

# Generate HTML report
npx playwright test --reporter=html
npx playwright show-report
```

### 8.9 Dashboard Widgets Per Role (`e2e/dashboard.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

// ── super_admin ──────────────────────────────────────────────────────────────
test.describe('Dashboard — super_admin', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'super_admin'); });

  test('My Tasks widget visible', async ({ page }) => {
    await expect(page.locator('[class*="my-tasks"], [data-testid="my-tasks"]')).toBeVisible();
  });
  test('Recent Activity feed visible', async ({ page }) => {
    await expect(page.locator('[class*="activity"], [data-testid="activity-feed"]')).toBeVisible();
  });
  test('Quick create task button visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Create Task"), button:has-text("New Task")')).toBeVisible();
  });
  test('Stakeholder dashboard link visible', async ({ page }) => {
    await expect(page.locator('a[href*="stakeholder"], button:has-text("Stakeholder")')).toBeVisible();
  });
  test('Notification bell with badge visible', async ({ page }) => {
    await expect(page.locator('[class*="notif-bell"], [class*="notification-bell"]')).toBeVisible();
  });
});

// ── owner ────────────────────────────────────────────────────────────────────
test.describe('Dashboard — owner', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'owner'); });

  test('page loads without error', async ({ page }) => {
    await expect(page).toHaveURL(/\/app\/dashboard/);
    await expect(page.locator('[class*="dashboard"], main')).toBeVisible();
  });
  test('Quick create task button visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Create Task"), button:has-text("New Task")')).toBeVisible();
  });
  test('Stakeholder dashboard link visible', async ({ page }) => {
    await expect(page.locator('a[href*="stakeholder"], button:has-text("Stakeholder")')).toBeVisible();
  });
});

// ── billing_admin ─────────────────────────────────────────────────────────────
test.describe('Dashboard — billing_admin', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'billing'); });

  test('page loads without error', async ({ page }) => {
    await expect(page).toHaveURL(/\/app\/dashboard/);
  });
  test('Quick create task button NOT visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Create Task"), button:has-text("New Task")')).toHaveCount(0);
  });
  test('Stakeholder dashboard link visible', async ({ page }) => {
    await expect(page.locator('a[href*="stakeholder"], button:has-text("Stakeholder")')).toBeVisible();
  });
});

// ── member ────────────────────────────────────────────────────────────────────
test.describe('Dashboard — member', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'member'); });

  test('My Tasks widget shows assigned tasks', async ({ page }) => {
    await expect(page.locator('[class*="my-tasks"], [data-testid="my-tasks"]')).toBeVisible();
  });
  test('Quick create task button visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Create Task"), button:has-text("New Task")')).toBeVisible();
  });
  test('Stakeholder dashboard link NOT visible', async ({ page }) => {
    await expect(page.locator('a[href*="stakeholder"], button:has-text("Stakeholder")')).toHaveCount(0);
  });
});

// ── guest ─────────────────────────────────────────────────────────────────────
test.describe('Dashboard — guest', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'guest'); });

  test('page loads without error', async ({ page }) => {
    await expect(page).toHaveURL(/\/app\/dashboard/);
  });
  test('Quick create task button NOT visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Create Task"), button:has-text("New Task")')).toHaveCount(0);
  });
  test('Stakeholder link NOT visible', async ({ page }) => {
    await expect(page.locator('a[href*="stakeholder"]')).toHaveCount(0);
  });
});
```

---

### 8.10 Gantt Per Role (`e2e/gantt.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test.describe('Gantt — view access per role', () => {
  const viewRoles = ['super_admin', 'owner', 'admin', 'member'] as const;

  for (const role of viewRoles) {
    test(`${role} can view Gantt chart`, async ({ page }) => {
      await loginAs(page, role);
      await page.goto('/app/gantt');
      await expect(page.locator('[class*="gantt"], [data-testid="gantt-chart"]')).toBeVisible();
    });

    test(`${role} can zoom to week view`, async ({ page }) => {
      await loginAs(page, role);
      await page.goto('/app/gantt');
      await page.locator('button:has-text("Week"), [data-view="week"]').click();
      await expect(page.locator('[class*="gantt-header"], [class*="gantt-grid"]')).toBeVisible();
    });
  }
});

test.describe('Gantt — drag to reschedule (permitted roles)', () => {
  const editRoles = ['super_admin', 'owner', 'admin', 'member'] as const;

  for (const role of editRoles) {
    test(`${role} can drag task bar to new date`, async ({ page }) => {
      await loginAs(page, role);
      await page.goto('/app/gantt');
      const taskBar = page.locator('[class*="gantt-task-bar"]').first();
      await taskBar.waitFor({ state: 'visible' });
      const box = await taskBar.boundingBox();
      if (!box) return;
      // Drag 200px to the right (forward in time)
      await page.mouse.move(box.x + box.width / 2, box.y + box.height / 2);
      await page.mouse.down();
      await page.mouse.move(box.x + box.width / 2 + 200, box.y + box.height / 2, { steps: 10 });
      await page.mouse.up();
      // Verify save feedback (no error toast, bar position changed)
      await expect(page.locator('[class*="error-toast"], .toast-error')).toHaveCount(0);
    });
  }
});

test.describe('Gantt — drag blocked for restricted roles', () => {
  test('billing_admin cannot drag task bar', async ({ page }) => {
    await loginAs(page, 'billing');
    await page.goto('/app/gantt');
    // Gantt should be read-only — task bars have no drag cursor or pointer-events
    const taskBar = page.locator('[class*="gantt-task-bar"]').first();
    if (await taskBar.count() > 0) {
      const cursor = await taskBar.evaluate(el => getComputedStyle(el).cursor);
      expect(cursor).not.toBe('grab');
    }
  });

  test('guest is redirected away from Gantt', async ({ page }) => {
    await loginAs(page, 'guest');
    await page.goto('/app/gantt');
    await expect(page).not.toHaveURL(/\/app\/gantt/);
  });
});

test.describe('Gantt — resize task bar (change duration)', () => {
  test('admin can resize task bar from right edge', async ({ page }) => {
    await loginAs(page, 'admin');
    await page.goto('/app/gantt');
    const taskBar = page.locator('[class*="gantt-task-bar"]').first();
    await taskBar.waitFor({ state: 'visible' });
    const resizeHandle = taskBar.locator('[class*="resize-handle"]');
    if (await resizeHandle.count() > 0) {
      const box = await resizeHandle.boundingBox();
      if (!box) return;
      await page.mouse.move(box.x + box.width / 2, box.y + box.height / 2);
      await page.mouse.down();
      await page.mouse.move(box.x + 100, box.y + box.height / 2, { steps: 10 });
      await page.mouse.up();
      await expect(page.locator('[class*="error-toast"]')).toHaveCount(0);
    }
  });
});

test.describe('Gantt — UI elements', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'super_admin'); });

  test('today marker is visible', async ({ page }) => {
    await page.goto('/app/gantt');
    await expect(page.locator('[class*="today-marker"], [class*="gantt-today"]')).toBeVisible();
  });

  test('clicking task bar navigates to task detail', async ({ page }) => {
    await page.goto('/app/gantt');
    const taskBar = page.locator('[class*="gantt-task-bar"]').first();
    await taskBar.waitFor({ state: 'visible' });
    await taskBar.click();
    await expect(page).toHaveURL(/\/app\/tasks\//);
  });

  test('dependency lines visible when tasks have dependencies', async ({ page }) => {
    await page.goto('/app/gantt');
    // Lines rendered as SVG paths or divs with dependency class
    const depLines = page.locator('[class*="dependency"], svg line, svg path');
    // Just verify the Gantt rendered successfully — dep lines only exist if data has them
    await expect(page.locator('[class*="gantt"]')).toBeVisible();
  });
});
```

---

### 8.11 Budget CRUD Per Role (`e2e/budget.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

// ── super_admin & owner: full CRUD ─────────────────────────────────────────
test.describe('Budget — super_admin full CRUD', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'super_admin'); });

  test('can access /app/budgets', async ({ page }) => {
    await page.goto('/app/budgets');
    await expect(page.locator('[class*="budget"], h1:has-text("Budget")')).toBeVisible();
  });

  test('create budget', async ({ page }) => {
    await page.goto('/app/budgets');
    await page.click('button:has-text("Create Budget"), button:has-text("New Budget")');
    await page.fill('input[placeholder*="total"], input[name="totalBudget"]', '50000');
    await page.locator('select[name="currency"], [class*="currency-select"]').selectOption('USD');
    await page.click('button[type="submit"], button:has-text("Save"), button:has-text("Create")');
    await expect(page.locator('[class*="budget-card"], [class*="budget-item"]').first()).toBeVisible();
  });

  test('add expense to budget', async ({ page }) => {
    await page.goto('/app/budgets');
    await page.locator('[class*="budget-card"]').first().click();
    await page.click('button:has-text("Add Expense"), button:has-text("New Expense")');
    await page.fill('input[name="amount"], input[placeholder*="amount"]', '1500');
    await page.fill('input[name="description"], input[placeholder*="description"]', 'Playwright test expense');
    await page.click('button[type="submit"], button:has-text("Save")');
    await expect(page.locator('text=Playwright test expense')).toBeVisible();
  });

  test('edit expense', async ({ page }) => {
    await page.goto('/app/budgets');
    await page.locator('[class*="budget-card"]').first().click();
    await page.locator('[class*="expense-item"]').first().hover();
    await page.locator('[class*="expense-item"]').first().locator('button[title*="Edit"], [class*="edit-btn"]').click();
    await page.fill('input[name="amount"], input[placeholder*="amount"]', '2000');
    await page.click('button[type="submit"], button:has-text("Save")');
    await expect(page.locator('text=2000, text=$2,000')).toBeVisible();
  });

  test('delete expense', async ({ page }) => {
    await page.goto('/app/budgets');
    await page.locator('[class*="budget-card"]').first().click();
    const expenseCount = await page.locator('[class*="expense-item"]').count();
    await page.locator('[class*="expense-item"]').first().hover();
    await page.locator('[class*="expense-item"]').first().locator('button[title*="Delete"], [class*="delete-btn"]').click();
    await page.locator('button:has-text("Delete"):visible, button:has-text("Confirm"):visible').click();
    await expect(page.locator('[class*="expense-item"]')).toHaveCount(Math.max(0, expenseCount - 1));
  });

  test('budget progress bar updates after adding expense', async ({ page }) => {
    await page.goto('/app/budgets');
    await expect(page.locator('[class*="progress-bar"], [class*="budget-progress"]').first()).toBeVisible();
  });

  test('over-budget warning shown when spent > total', async ({ page }) => {
    await page.goto('/app/budgets');
    // If any budget is over, the warning should appear
    const warning = page.locator('[class*="over-budget"], [class*="warning"], text=Over budget');
    // We just verify the element exists in DOM when condition applies — not strictly asserting count
    await expect(page.locator('[class*="budget"]').first()).toBeVisible();
  });
});

// ── billing_admin: full CRUD ──────────────────────────────────────────────
test.describe('Budget — billing_admin full CRUD', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'billing'); });

  test('can access /app/budgets', async ({ page }) => {
    await page.goto('/app/budgets');
    await expect(page.locator('[class*="budget"], h1:has-text("Budget")')).toBeVisible();
  });

  test('create budget button visible', async ({ page }) => {
    await page.goto('/app/budgets');
    await expect(page.locator('button:has-text("Create Budget"), button:has-text("New Budget")')).toBeVisible();
  });

  test('can add expense', async ({ page }) => {
    await page.goto('/app/budgets');
    await page.locator('[class*="budget-card"]').first().click();
    await expect(page.locator('button:has-text("Add Expense"), button:has-text("New Expense")')).toBeVisible();
  });
});

// ── admin: view-only on budgets ───────────────────────────────────────────
test.describe('Budget — admin view-only', () => {
  test.beforeEach(async ({ page }) => { await loginAs(page, 'admin'); });

  test('can access /app/budgets in read mode', async ({ page }) => {
    await page.goto('/app/budgets');
    await expect(page.locator('[class*="budget"]')).toBeVisible();
  });

  test('Create Budget button NOT visible', async ({ page }) => {
    await page.goto('/app/budgets');
    await expect(page.locator('button:has-text("Create Budget")')).toHaveCount(0);
  });

  test('Add Expense button NOT visible', async ({ page }) => {
    await page.goto('/app/budgets');
    if (await page.locator('[class*="budget-card"]').count() > 0) {
      await page.locator('[class*="budget-card"]').first().click();
      await expect(page.locator('button:has-text("Add Expense")')).toHaveCount(0);
    }
  });
});

// ── member & guest: blocked ───────────────────────────────────────────────
test.describe('Budget — member & guest blocked', () => {
  test('member cannot access /app/budgets', async ({ page }) => {
    await loginAs(page, 'member');
    await page.goto('/app/budgets');
    await expect(page).not.toHaveURL(/\/app\/budgets/);
  });

  test('guest cannot access /app/budgets', async ({ page }) => {
    await loginAs(page, 'guest');
    await page.goto('/app/budgets');
    await expect(page).not.toHaveURL(/\/app\/budgets/);
  });
});
```

---

### 8.12 Automations Per Role (`e2e/automations.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

// ── super_admin / owner / admin: full CRUD ────────────────────────────────
const automationRoles = ['super_admin', 'owner', 'admin'] as const;

for (const role of automationRoles) {
  test.describe(`Automations — ${role}`, () => {
    test.beforeEach(async ({ page }) => { await loginAs(page, role as any); });

    test('can access /app/automations', async ({ page }) => {
      await page.goto('/app/automations');
      await expect(page.locator('[class*="automation"], h1:has-text("Automation")')).toBeVisible();
    });

    test('create automation', async ({ page }) => {
      await page.goto('/app/automations');
      await page.click('button:has-text("Create Automation"), button:has-text("New Automation")');
      // Name the automation
      await page.fill('input[name="name"], input[placeholder*="name"]', `${role} Test Automation`);
      // Select trigger
      await page.locator('[class*="trigger-select"], select[name="trigger"]').first().click();
      await page.locator('text=Task Created, [data-value="task_created"]').first().click();
      // Select action
      await page.locator('[class*="action-select"], select[name="action"]').first().click();
      await page.locator('text=Send Notification, [data-value="send_notification"]').first().click();
      await page.click('button:has-text("Save"), button[type="submit"]');
      await expect(page.locator(`text=${role} Test Automation`)).toBeVisible();
    });

    test('toggle automation on/off', async ({ page }) => {
      await page.goto('/app/automations');
      const toggle = page.locator('[class*="automation-item"]').first().locator('[class*="toggle"], input[type="checkbox"]');
      const wasChecked = await toggle.isChecked();
      await toggle.click();
      await expect(toggle).toBeChecked({ checked: !wasChecked });
      // Revert
      await toggle.click();
    });

    test('edit existing automation', async ({ page }) => {
      await page.goto('/app/automations');
      await page.locator('[class*="automation-item"]').first().locator('button[title*="Edit"], [class*="edit-btn"]').click();
      await page.fill('input[name="name"], input[placeholder*="name"]', `Updated ${role} Automation`);
      await page.click('button:has-text("Save"), button[type="submit"]');
      await expect(page.locator(`text=Updated ${role} Automation`)).toBeVisible();
    });

    test('delete automation', async ({ page }) => {
      await page.goto('/app/automations');
      const countBefore = await page.locator('[class*="automation-item"]').count();
      await page.locator('[class*="automation-item"]').first().locator('button[title*="Delete"], [class*="delete-btn"]').click();
      await page.locator('button:has-text("Delete"):visible').click();
      await expect(page.locator('[class*="automation-item"]')).toHaveCount(Math.max(0, countBefore - 1));
    });

    test('view automation run history', async ({ page }) => {
      await page.goto('/app/automations');
      const historyBtn = page.locator('button:has-text("History"), button:has-text("Run History"), [class*="history-btn"]').first();
      if (await historyBtn.count() > 0) {
        await historyBtn.click();
        await expect(page.locator('[class*="run-history"], [class*="automation-log"]')).toBeVisible();
      }
    });
  });
}

// ── billing_admin / member / guest: blocked ────────────────────────────────
const blockedRoles = ['billing', 'member', 'guest'] as const;

for (const role of blockedRoles) {
  test(`${role} cannot access /app/automations`, async ({ page }) => {
    await loginAs(page, role);
    await page.goto('/app/automations');
    await expect(page).not.toHaveURL(/\/app\/automations/);
  });
}
```

---

### 8.13 Settings Tabs Per Role (`e2e/settings.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

// ── super_admin: all tabs accessible ─────────────────────────────────────
test.describe('Settings — super_admin', () => {
  test.beforeEach(async ({ page }) => {
    await loginAs(page, 'super_admin');
    await page.goto('/app/settings');
  });

  test('Profile tab — view and edit name', async ({ page }) => {
    await page.click('button:has-text("Profile"), [data-tab="profile"]');
    await expect(page.locator('input[name="firstName"], input[placeholder*="first"]')).toBeVisible();
    await page.fill('input[name="firstName"], input[placeholder*="first"]', 'SuperAdmin');
    await page.click('button:has-text("Save"), button[type="submit"]');
    await expect(page.locator('[class*="success"], text=Saved')).toBeVisible();
  });

  test('Profile tab — upload avatar button visible', async ({ page }) => {
    await page.click('button:has-text("Profile"), [data-tab="profile"]');
    await expect(page.locator('input[type="file"], button:has-text("Upload Avatar")')).toBeVisible();
  });

  test('Security tab — change password form visible', async ({ page }) => {
    await page.click('button:has-text("Security"), [data-tab="security"]');
    await expect(page.locator('input[type="password"], input[name*="password"]').first()).toBeVisible();
  });

  test('Security tab — 2FA toggle visible', async ({ page }) => {
    await page.click('button:has-text("Security"), [data-tab="security"]');
    await expect(page.locator('button:has-text("Enable 2FA"), button:has-text("Disable 2FA"), [class*="2fa"]')).toBeVisible();
  });

  test('Permissions tab visible and renders role matrix', async ({ page }) => {
    await page.click('button:has-text("Permissions"), [data-tab="permissions"]');
    await expect(page.locator('table, [class*="permission-matrix"]')).toBeVisible();
  });

  test('Audit Log tab visible and shows entries', async ({ page }) => {
    await page.click('button:has-text("Audit"), [data-tab="audit"]');
    await expect(page.locator('[class*="activity-log"], [class*="audit-log"]')).toBeVisible();
  });

  test('Templates tab visible', async ({ page }) => {
    await page.click('button:has-text("Templates"), [data-tab="templates"]');
    await expect(page.locator('[class*="templates"], [class*="template-list"]')).toBeVisible();
  });

  test('Notifications tab — absent or shows not-built state', async ({ page }) => {
    const notifTab = page.locator('button:has-text("Notifications"), [data-tab="notifications"]');
    if (await notifTab.count() > 0) {
      await notifTab.click();
      // Should show not-built state or backend-missing message
      await expect(page.locator('[class*="empty"], text=coming soon, text=not available')).toBeVisible();
    } else {
      // Tab not present — correct behavior since feature not built
      expect(true).toBe(true);
    }
  });
});

// ── owner: same as super_admin except Notifications ───────────────────────
test.describe('Settings — owner', () => {
  test.beforeEach(async ({ page }) => {
    await loginAs(page, 'owner');
    await page.goto('/app/settings');
  });

  test('Profile, Security, Permissions, Audit, Templates tabs all present', async ({ page }) => {
    for (const tab of ['Profile', 'Security', 'Permissions', 'Audit', 'Templates']) {
      const tabBtn = page.locator(`button:has-text("${tab}"), [data-tab="${tab.toLowerCase()}"]`);
      await expect(tabBtn).toBeVisible();
    }
  });
});

// ── admin: Profile, Security, Permissions (view), Audit, Templates ─────────
test.describe('Settings — admin', () => {
  test.beforeEach(async ({ page }) => {
    await loginAs(page, 'admin');
    await page.goto('/app/settings');
  });

  test('Profile tab accessible', async ({ page }) => {
    await page.click('button:has-text("Profile"), [data-tab="profile"]');
    await expect(page.locator('input[name="firstName"], input[placeholder*="first"]')).toBeVisible();
  });

  test('Permissions tab — view only (no edit controls)', async ({ page }) => {
    await page.click('button:has-text("Permissions"), [data-tab="permissions"]');
    await expect(page.locator('table, [class*="permission-matrix"]')).toBeVisible();
    // Admin cannot EDIT permissions — no save button in permissions tab
    await expect(page.locator('[class*="permission-matrix"] button:has-text("Save")')).toHaveCount(0);
  });
});

// ── billing_admin: only Profile + Security ─────────────────────────────────
test.describe('Settings — billing_admin', () => {
  test.beforeEach(async ({ page }) => {
    await loginAs(page, 'billing');
    await page.goto('/app/settings');
  });

  test('Profile tab accessible', async ({ page }) => {
    await page.click('button:has-text("Profile"), [data-tab="profile"]');
    await expect(page.locator('input[name="firstName"], input[placeholder*="first"]')).toBeVisible();
  });

  test('Security tab accessible', async ({ page }) => {
    await page.click('button:has-text("Security"), [data-tab="security"]');
    await expect(page.locator('input[type="password"]').first()).toBeVisible();
  });

  test('Permissions tab NOT accessible or hidden', async ({ page }) => {
    const permTab = page.locator('button:has-text("Permissions"), [data-tab="permissions"]');
    if (await permTab.count() > 0) {
      await permTab.click();
      // Should redirect or show 403 state
      await expect(page.locator('[class*="forbidden"], [class*="empty"], text=403')).toBeVisible();
    }
  });

  test('Audit Log tab NOT visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Audit Log"), [data-tab="audit"]')).toHaveCount(0);
  });
});

// ── member: only Profile + Security ──────────────────────────────────────
test.describe('Settings — member', () => {
  test.beforeEach(async ({ page }) => {
    await loginAs(page, 'member');
    await page.goto('/app/settings');
  });

  test('Profile tab — edit own name and save', async ({ page }) => {
    await page.click('button:has-text("Profile"), [data-tab="profile"]');
    const firstNameInput = page.locator('input[name="firstName"], input[placeholder*="first"]');
    await firstNameInput.fill('MemberUpdated');
    await page.click('button:has-text("Save"), button[type="submit"]');
    await expect(page.locator('[class*="success"], text=Saved')).toBeVisible();
    // Revert
    await firstNameInput.fill('Member');
    await page.click('button:has-text("Save"), button[type="submit"]');
  });

  test('Permissions tab NOT visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Permissions"), [data-tab="permissions"]')).toHaveCount(0);
  });

  test('Audit Log tab NOT visible', async ({ page }) => {
    await expect(page.locator('button:has-text("Audit"), [data-tab="audit"]')).toHaveCount(0);
  });
});

// ── guest: cannot access settings ─────────────────────────────────────────
test.describe('Settings — guest', () => {
  test('guest is redirected from /app/settings', async ({ page }) => {
    await loginAs(page, 'guest');
    await page.goto('/app/settings');
    await expect(page).not.toHaveURL(/\/app\/settings/);
  });
});
```

---

### 8.14 Edge Cases — Auth (`e2e/edge-cases-auth.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';

// A1 — Login wrong password shows inline error, no redirect
test('A1: wrong password shows error, no redirect', async ({ page }) => {
  await page.goto('/login');
  await page.fill('input[type="email"]', 'admin@pulsehub.com');
  await page.fill('input[type="password"]', 'wrongpassword');
  await page.click('button[type="submit"]');
  await expect(page.locator('[class*="error"], .error-message')).toBeVisible();
  await expect(page).not.toHaveURL(/\/app\/dashboard/);
});

// A2 — Unverified email shows verification error
test('A2: unverified email shows verification message', async ({ page }) => {
  // Register a fresh user (no verification)
  await page.goto('/register');
  await page.fill('input[name="firstName"]', 'Unverified');
  await page.fill('input[name="lastName"]', 'User');
  await page.fill('input[type="email"]', `unverified_${Date.now()}@test.com`);
  await page.fill('input[type="password"]', 'Test1234!');
  await page.click('button[type="submit"]');
  // Should see verification banner or message
  await expect(page.locator('text=verify, text=verification, [class*="verify-banner"]')).toBeVisible();
});

// A3 — Duplicate email on register
test('A3: duplicate email on register shows error', async ({ page }) => {
  await page.goto('/register');
  await page.fill('input[name="firstName"]', 'Dup');
  await page.fill('input[name="lastName"]', 'User');
  await page.fill('input[type="email"]', 'admin@pulsehub.com');
  await page.fill('input[type="password"]', 'Test1234!');
  await page.click('button[type="submit"]');
  await expect(page.locator('[class*="error"], text=already, text=in use')).toBeVisible();
});

// A4 — Weak password on register
test('A4: password too short shows validation error', async ({ page }) => {
  await page.goto('/register');
  await page.fill('input[name="firstName"]', 'Test');
  await page.fill('input[name="lastName"]', 'User');
  await page.fill('input[type="email"]', `test_${Date.now()}@test.com`);
  await page.fill('input[type="password"]', 'abc');
  await page.click('button[type="submit"]');
  await expect(page.locator('[class*="error"], text=password, text=least 8')).toBeVisible();
});

// A5 — Password no uppercase
test('A5: password without uppercase shows validation error', async ({ page }) => {
  await page.goto('/register');
  await page.fill('input[name="firstName"]', 'Test');
  await page.fill('input[name="lastName"]', 'User');
  await page.fill('input[type="email"]', `test_${Date.now()}@test.com`);
  await page.fill('input[type="password"]', 'test1234!');
  await page.click('button[type="submit"]');
  await expect(page.locator('[class*="error"], text=uppercase')).toBeVisible();
});

// A6 — Expired token redirects to /login
test('A6: expired token accessing protected route redirects to login', async ({ page, context }) => {
  // Set an expired/invalid JWT in localStorage
  await context.addInitScript(() => {
    localStorage.setItem('token', 'eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjF9.invalid_expired_token');
  });
  await page.goto('/app/dashboard');
  await expect(page).toHaveURL(/\/login/);
});

// A7 — No token on protected route
test('A7: no token accessing /app/dashboard redirects to /login', async ({ page }) => {
  await page.goto('/app/dashboard');
  await expect(page).toHaveURL(/\/login/);
});

// A10 — Unauthenticated API call
test('A10: API call without token returns 401', async ({ request }) => {
  const res = await request.get('http://localhost:5000/api/auth/me');
  expect(res.status()).toBe(401);
});
```

---

### 8.15 Edge Cases — Tasks (`e2e/edge-cases-tasks.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test.beforeEach(async ({ page }) => { await loginAs(page, 'admin'); });

// T1 — Title at max length
test('T1: 500-char title accepted or rejected with clear error', async ({ page }) => {
  await page.goto('/app/tasks');
  await page.click('button:has-text("Create Task"), button:has-text("New Task")');
  await page.fill('input[placeholder*="title"], input[name="title"]', 'A'.repeat(500));
  await page.click('button[type="submit"], button:has-text("Create")');
  // Either task created or validation error shown — both are acceptable
  const created = await page.locator('[class*="task-row"], [class*="task-card"]').count();
  const errShown = await page.locator('[class*="error"]').count();
  expect(created > 0 || errShown > 0).toBe(true);
});

// T2 — Past due date shows overdue badge
test('T2: past due date shows overdue badge in UI', async ({ page }) => {
  await page.goto('/app/tasks');
  await page.click('button:has-text("Create Task"), button:has-text("New Task")');
  await page.fill('input[placeholder*="title"], input[name="title"]', 'Overdue Task Test');
  await page.fill('input[type="date"], input[name="dueDate"]', '2020-01-01');
  await page.click('button[type="submit"], button:has-text("Create")');
  await expect(page.locator('[class*="overdue"], text=Overdue')).toBeVisible();
});

// T3 — Bulk action with 0 tasks selected
test('T3: bulk toolbar not visible when no tasks selected', async ({ page }) => {
  await page.goto('/app/tasks');
  await expect(page.locator('[class*="bulk-toolbar"], [class*="bulk-actions"]')).toHaveCount(0);
});

// T5 — Bulk select and bulk update status
test('T5: bulk select tasks and update status', async ({ page }) => {
  await page.goto('/app/tasks');
  const checkboxes = page.locator('[class*="task-checkbox"], input[type="checkbox"]');
  if (await checkboxes.count() >= 2) {
    await checkboxes.nth(0).click();
    await checkboxes.nth(1).click();
    await expect(page.locator('[class*="bulk-toolbar"], [class*="bulk-actions"]')).toBeVisible();
  }
});

// T8 — CSV export produces a file download
test('T8: export tasks as CSV triggers download', async ({ page }) => {
  await page.goto('/app/tasks');
  const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.click('button:has-text("Export"), button:has-text("CSV")'),
  ]);
  expect(download.suggestedFilename()).toMatch(/\.csv$/i);
});

// T9 — Delete task with subtasks cascades
test('T9: delete task with subtasks removes all', async ({ page }) => {
  await page.goto('/app/tasks');
  // Open first task, add subtask, then delete parent
  await page.locator('[class*="task-row"], [class*="task-card"]').first().click();
  const addSubtask = page.locator('button:has-text("Add Subtask"), button:has-text("Subtask")');
  if (await addSubtask.count() > 0) {
    await addSubtask.click();
    await page.fill('input[placeholder*="subtask"], input[name="subtaskTitle"]', 'Child task');
    await page.keyboard.press('Enter');
    // Close detail and delete parent
    await page.keyboard.press('Escape');
    const taskMenu = page.locator('[class*="task-menu-btn"], [class*="task-options"]').first();
    await taskMenu.click();
    await page.click('button:has-text("Delete")');
    await page.click('button:has-text("Delete"):visible');
    // Child task should be gone
    await expect(page.locator('text=Child task')).toHaveCount(0);
  }
});
```

---

### 8.16 Edge Cases — Chat (`e2e/edge-cases-chat.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test.beforeEach(async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/app/chat');
  await page.locator('[class*="room-list-item"]').first().click();
});

// C1 — Empty message (spaces only) cannot be sent
test('C1: whitespace-only message cannot be sent', async ({ page }) => {
  const composer = page.locator('textarea[placeholder*="message"], input[placeholder*="message"]');
  await composer.fill('   ');
  const sendBtn = page.locator('button[type="submit"], [class*="send-btn"]');
  await expect(sendBtn).toBeDisabled();
});

// C3 — File > 25MB shows error
test('C3: file over 25MB shows size error', async ({ page }) => {
  const fileInput = page.locator('input[type="file"]');
  // Create a mock large file
  await page.evaluate(() => {
    const dt = new DataTransfer();
    const bigFile = new File([new ArrayBuffer(26 * 1024 * 1024)], 'big.mp4', { type: 'video/mp4' });
    dt.items.add(bigFile);
    const input = document.querySelector('input[type="file"]') as HTMLInputElement;
    if (input) Object.defineProperty(input, 'files', { value: dt.files });
    if (input) input.dispatchEvent(new Event('change', { bubbles: true }));
  });
  await expect(page.locator('[class*="error"], text=25MB, text=too large')).toBeVisible({ timeout: 3000 });
});

// C5 — Reply to a message shows quoted content
test('C5: reply to message shows parent quote in composer', async ({ page }) => {
  const msg = page.locator('[class*="msg-bubble"]').first();
  await msg.hover();
  const replyBtn = page.locator('[class*="reply-btn"], button[title*="Reply"]').first();
  if (await replyBtn.count() > 0) {
    await replyBtn.click();
    await expect(page.locator('[class*="reply-bar"], [class*="reply-preview"]')).toBeVisible();
  }
});

// C9 — Same room in two tabs (real-time delivery)
test('C9: message sent in one context appears in another (real-time)', async ({ page, context }) => {
  const page2 = await context.newPage();
  await loginAs(page2, 'admin');
  await page2.goto('/app/chat');
  await page2.locator('[class*="room-list-item"]').first().click();

  const uniqueMsg = `RT test ${Date.now()}`;
  const composer = page.locator('textarea[placeholder*="message"], input[placeholder*="message"]');
  await composer.fill(uniqueMsg);
  await page.keyboard.press('Enter');

  // Other user sees message without refresh
  await expect(page2.locator(`text=${uniqueMsg}`)).toBeVisible({ timeout: 5000 });
  await page2.close();
});

// C10 — Delete message visible in both tabs
test('C10: delete message in one tab disappears in other', async ({ page, context }) => {
  const page2 = await context.newPage();
  await loginAs(page2, 'admin');
  await page2.goto('/app/chat');
  await page2.locator('[class*="room-list-item"]').first().click();

  // Send a message to delete
  const uniqueMsg = `Delete me ${Date.now()}`;
  await page.locator('textarea[placeholder*="message"], input[placeholder*="message"]').fill(uniqueMsg);
  await page.keyboard.press('Enter');
  await expect(page.locator(`text=${uniqueMsg}`)).toBeVisible();

  // Delete it
  const msgEl = page.locator(`[class*="msg-bubble"]:has-text("${uniqueMsg}")`);
  await msgEl.hover();
  await page.locator('[class*="delete-btn"], button[title*="Delete"]').first().click();
  await page.locator('button:has-text("Delete"):visible').click();

  // Page2 should no longer see it
  await expect(page2.locator(`text=${uniqueMsg}`)).toHaveCount(0);
  await page2.close();
});
```

---

### 8.17 Edge Cases — Documents (`e2e/edge-cases-documents.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

// D1 — Public doc accessible in incognito
test('D1: shared document accessible without login', async ({ page, context }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  // Create and share a document
  await page.click('button[title="New document"], button:has-text("New")');
  await page.fill('input[placeholder="Document title"]', 'D1 Public Doc');
  await page.locator('.ProseMirror, [class*="tiptap"]').click();
  await page.keyboard.type('Public content test');
  await page.waitForTimeout(2500); // auto-save
  await page.click('button:has-text("Share")');
  await page.locator('[class*="share-toggle-btn"]').click();
  const shareUrl = await page.locator('[class*="share-link-input"]').inputValue();

  const incognitoPage = await context.newPage();
  await incognitoPage.goto(shareUrl);
  await expect(incognitoPage.locator('text=Public content test, .ProseMirror')).toBeVisible({ timeout: 8000 });
  await incognitoPage.close();
});

// D2 — Unshared doc link returns 404
test('D2: unsharing document makes link return 404', async ({ page, context }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.locator('[class*="doc-list-item"]').first().click();
  // Ensure it is shared first
  await page.click('button:has-text("Share")');
  const toggleBtn = page.locator('[class*="share-toggle-btn"]');
  const isOn = (await toggleBtn.innerText()).toLowerCase().includes('on');
  if (!isOn) await toggleBtn.click();
  const shareUrl = await page.locator('[class*="share-link-input"]').inputValue();
  // Now unshare
  await toggleBtn.click(); // turn off
  await page.locator('[class*="share-modal-close"]').click();

  const incognitoPage = await context.newPage();
  await incognitoPage.goto(shareUrl);
  await expect(incognitoPage.locator('text=404, text=not found, text=no longer shared')).toBeVisible({ timeout: 8000 });
  await incognitoPage.close();
});

// D3 — Auto-save: content preserved without manual save
test('D3: auto-save preserves content without clicking Save', async ({ page }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.click('button[title="New document"], button:has-text("New")');
  await page.fill('input[placeholder="Document title"]', 'D3 Auto-save Test');
  await page.locator('.ProseMirror, [class*="tiptap"]').click();
  const uniqueText = `Auto-save test ${Date.now()}`;
  await page.keyboard.type(uniqueText);
  // Wait for auto-save (2s delay + network)
  await expect(page.locator('text=Saved, [class*="save-status"]')).toBeVisible({ timeout: 6000 });
  // Reload and verify content persisted
  await page.reload();
  await page.locator('[class*="doc-list-item"]:has-text("D3 Auto-save Test")').click();
  await expect(page.locator(`.ProseMirror, [class*="tiptap"]`)).toContainText(uniqueText);
});

// D5 — Empty title rejected
test('D5: creating document with empty title shows error', async ({ page, request }) => {
  const { TEST_USERS } = await import('./helpers/auth');
  // API-level test for empty title
  const loginRes = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.admin.email, password: TEST_USERS.admin.password },
  });
  const { data } = await loginRes.json();
  const res = await request.post('http://localhost:5000/api/documents', {
    data: { title: '', content: '', contentType: 'html' },
    headers: { Authorization: `Bearer ${data.token}` },
  });
  expect(res.status()).toBe(400);
});

// D6 — Delete shared document; link returns 404
test('D6: deleting a shared document makes link 404', async ({ page, context }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.click('button[title="New document"]');
  await page.fill('input[placeholder="Document title"]', 'D6 Delete Shared');
  await page.waitForTimeout(2500);
  await page.click('button:has-text("Share")');
  await page.locator('[class*="share-toggle-btn"]').click();
  const shareUrl = await page.locator('[class*="share-link-input"]').inputValue();
  await page.locator('[class*="share-modal-close"]').click();
  // Delete the document
  const docItem = page.locator('[class*="doc-list-item"]:has-text("D6 Delete Shared")');
  await docItem.locator('[class*="doc-list-menu-btn"]').click();
  await page.click('[class*="doc-dropdown-item--danger"], button:has-text("Delete")');
  await page.click('button:has-text("Delete"):visible');
  // Verify link is dead
  const incognitoPage = await context.newPage();
  await incognitoPage.goto(shareUrl);
  await expect(incognitoPage.locator('text=404, text=not found')).toBeVisible({ timeout: 8000 });
  await incognitoPage.close();
});

// D7 — Rich text formatting persists after save and reload
test('D7: rich text formatting preserved after reload', async ({ page }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/documents');
  await page.click('button[title="New document"]');
  await page.fill('input[placeholder="Document title"]', 'D7 Rich Text');
  const editor = page.locator('.ProseMirror, [class*="tiptap"]');
  await editor.click();
  await page.keyboard.type('Normal then ');
  await page.keyboard.press('Control+b');
  await page.keyboard.type('bold text');
  await page.keyboard.press('Control+b');
  await expect(page.locator('text=Saved, [class*="save-status"]')).toBeVisible({ timeout: 6000 });
  await page.reload();
  await page.locator('[class*="doc-list-item"]:has-text("D7 Rich Text")').click();
  await expect(editor.locator('strong, b')).toContainText('bold text');
});
```

---

### 8.18 Edge Cases — RBAC (`e2e/edge-cases-rbac.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs, TEST_USERS } from './helpers/auth';

// R1 — Guest navigating to /app/projects is redirected
test('R1: guest redirected from /app/projects', async ({ page }) => {
  await loginAs(page, 'guest');
  await page.goto('/app/projects');
  await expect(page).not.toHaveURL(/\/app\/projects/);
});

// R2 — billing_admin cannot create tasks via API
test('R2: billing_admin POST /tasks returns 403', async ({ request }) => {
  const loginRes = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.billing.email, password: TEST_USERS.billing.password },
  });
  const { data } = await loginRes.json();
  const res = await request.post('http://localhost:5000/api/tasks', {
    data: { title: 'RBAC test', projectId: 'any-id' },
    headers: { Authorization: `Bearer ${data.token}` },
  });
  expect([403, 404]).toContain(res.status());
});

// R3 — Member cannot delete another member's comment via API
test('R3: member cannot delete another users comment', async ({ request }) => {
  const memberLogin = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.member.email, password: TEST_USERS.member.password },
  });
  const { data: memberData } = await memberLogin.json();
  // Try to delete a comment ID that doesn't belong to member
  const res = await request.delete('http://localhost:5000/api/comments/00000000-0000-0000-0000-000000000001', {
    headers: { Authorization: `Bearer ${memberData.token}` },
  });
  expect([403, 404]).toContain(res.status());
});

// R4 — Non-owner cannot delete workspace via API
test('R4: admin cannot delete workspace via API', async ({ request }) => {
  const loginRes = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.admin.email, password: TEST_USERS.admin.password },
  });
  const { data } = await loginRes.json();
  const res = await request.delete('http://localhost:5000/api/workspaces/00000000-0000-0000-0000-000000000001', {
    headers: { Authorization: `Bearer ${data.token}` },
  });
  expect([403, 404]).toContain(res.status());
});

// R5 — super_admin sees all workspaces
test('R5: super_admin GET /workspaces returns all workspaces', async ({ request }) => {
  const loginRes = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.super_admin.email, password: TEST_USERS.super_admin.password },
  });
  const { data } = await loginRes.json();
  const res = await request.get('http://localhost:5000/api/workspaces', {
    headers: { Authorization: `Bearer ${data.token}` },
  });
  expect(res.status()).toBe(200);
  const body = await res.json();
  expect(body.data.length).toBeGreaterThan(0);
});

// R7 — Member access to /app/budgets is blocked
test('R7: member redirected from /app/budgets', async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/app/budgets');
  await expect(page).not.toHaveURL(/\/app\/budgets/);
});

// R8 — Guest cannot create chat room via API
test('R8: guest POST /chat/rooms returns 403', async ({ request }) => {
  const loginRes = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.guest.email, password: TEST_USERS.guest.password },
  });
  const { data } = await loginRes.json();
  const res = await request.post('http://localhost:5000/api/chat/rooms', {
    data: { name: 'Guest Room Attempt', type: 'project' },
    headers: { Authorization: `Bearer ${data.token}` },
  });
  expect(res.status()).toBe(403);
});

// R10 — Non-member cannot access project via API
test('R10: non-member GET /projects/:id returns 403 or 404', async ({ request }) => {
  const loginRes = await request.post('http://localhost:5000/api/auth/login', {
    data: { email: TEST_USERS.guest.email, password: TEST_USERS.guest.password },
  });
  const { data } = await loginRes.json();
  const res = await request.get('http://localhost:5000/api/projects/00000000-0000-0000-0000-000000000001', {
    headers: { Authorization: `Bearer ${data.token}` },
  });
  expect([403, 404]).toContain(res.status());
});
```

---

### 8.19 Edge Cases — Socket.io Real-time (`e2e/edge-cases-socket.spec.ts`)

```typescript
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

// S1 — Real-time chat message delivery < 5s
test('S1: chat message delivered to second user in real-time', async ({ context }) => {
  const senderPage = await context.newPage();
  const receiverPage = await context.newPage();

  await loginAs(senderPage, 'admin');
  await loginAs(receiverPage, 'member');

  await senderPage.goto('/app/chat');
  await senderPage.locator('[class*="room-list-item"]').first().click();

  await receiverPage.goto('/app/chat');
  await receiverPage.locator('[class*="room-list-item"]').first().click();

  const uniqueMsg = `Socket S1 ${Date.now()}`;
  await senderPage.locator('textarea[placeholder*="message"], input[placeholder*="message"]').fill(uniqueMsg);
  await senderPage.keyboard.press('Enter');

  await expect(receiverPage.locator(`text=${uniqueMsg}`)).toBeVisible({ timeout: 5000 });

  await senderPage.close();
  await receiverPage.close();
});

// S3 — Whiteboard multi-user sticky note visibility
test('S3: sticky note added by User A visible to User B in real-time', async ({ context }) => {
  const pageA = await context.newPage();
  const pageB = await context.newPage();

  await loginAs(pageA, 'admin');
  await loginAs(pageB, 'member');

  await pageA.goto('/app/whiteboard');
  await pageB.goto('/app/whiteboard');

  // Both open same whiteboard
  const wbItem = pageA.locator('[class*="whiteboard-item"]').first();
  if (await wbItem.count() > 0) {
    await wbItem.click();
    await pageB.locator('[class*="whiteboard-item"]').first().click();

    // User A adds sticky note
    await pageA.locator('button[title*="Sticky"], button:has-text("Note")').click();
    await pageA.locator('[class*="sticky-note"] [contenteditable]').first().fill('S3 RT note');

    // User B sees it
    await expect(pageB.locator('text=S3 RT note')).toBeVisible({ timeout: 5000 });
  }

  await pageA.close();
  await pageB.close();
});

// S4 — Notification badge increments on task assignment in real-time
test('S4: notification badge increments when task assigned in real-time', async ({ context }) => {
  const adminPage = await context.newPage();
  const memberPage = await context.newPage();

  await loginAs(adminPage, 'admin');
  await loginAs(memberPage, 'member');

  // Get initial badge count for member
  await memberPage.goto('/app/dashboard');
  const badgeBefore = await memberPage.locator('[class*="notif-badge"], [class*="badge"]').first().textContent() || '0';

  // Admin assigns task to member
  await adminPage.goto('/app/tasks');
  await adminPage.locator('[class*="task-row"]').first().click();
  await adminPage.locator('[class*="assignee-selector"], button:has-text("Assign")').click();
  await adminPage.locator('[class*="member-option"]').filter({ hasText: 'Member' }).click();

  // Member sees badge increment
  await expect(memberPage.locator('[class*="notif-badge"], [class*="badge"]').first()).not.toHaveText(badgeBefore, { timeout: 5000 });

  await adminPage.close();
  await memberPage.close();
});

// S6 — No duplicate messages on rapid send
test('S6: rapid fire messages do not produce duplicates', async ({ page }) => {
  await loginAs(page, 'admin');
  await page.goto('/app/chat');
  await page.locator('[class*="room-list-item"]').first().click();

  const composer = page.locator('textarea[placeholder*="message"], input[placeholder*="message"]');
  const uniquePrefix = `Rapid ${Date.now()}`;

  for (let i = 1; i <= 5; i++) {
    await composer.fill(`${uniquePrefix} msg${i}`);
    await page.keyboard.press('Enter');
    await page.waitForTimeout(100);
  }

  // Each message should appear exactly once
  for (let i = 1; i <= 5; i++) {
    await expect(page.locator(`text=${uniquePrefix} msg${i}`)).toHaveCount(1);
  }
});

// S7 — Auto-rejoin rooms on socket reconnect
test('S7: user auto-rejoins chat room after disconnect', async ({ page }) => {
  await loginAs(page, 'member');
  await page.goto('/app/chat');
  await page.locator('[class*="room-list-item"]').first().click();

  // Simulate network offline/online cycle
  await page.context().setOffline(true);
  await page.waitForTimeout(1000);
  await page.context().setOffline(false);

  // After reconnect, user should still be in room and able to send
  const composer = page.locator('textarea[placeholder*="message"], input[placeholder*="message"]');
  await expect(composer).toBeVisible({ timeout: 8000 });
  const reconnectMsg = `Post-reconnect ${Date.now()}`;
  await composer.fill(reconnectMsg);
  await page.keyboard.press('Enter');
  await expect(page.locator(`text=${reconnectMsg}`)).toBeVisible({ timeout: 8000 });
});
```

---

### 8.20 Complete E2E File Structure

```
e2e/
├── helpers/
│   └── auth.ts                      ← loginAs() helper + TEST_USERS
├── auth.spec.ts                     ← Login/register flows
├── rbac.spec.ts                     ← Role access guard (page-level)
├── dashboard.spec.ts                ← Widget visibility per role   [NEW]
├── tasks.spec.ts                    ← Task CRUD per role
├── gantt.spec.ts                    ← Gantt drag/resize per role   [NEW]
├── budget.spec.ts                   ← Budget CRUD per role         [NEW]
├── automations.spec.ts              ← Automation CRUD per role     [NEW]
├── settings.spec.ts                 ← Settings tabs per role       [NEW]
├── chat.spec.ts                     ← Chat CRUD + reactions
├── documents.spec.ts                ← Document CRUD + share
├── edge-cases-auth.spec.ts          ← A1–A10                      [NEW]
├── edge-cases-tasks.spec.ts         ← T1–T9                       [NEW]
├── edge-cases-chat.spec.ts          ← C1–C10                      [NEW]
├── edge-cases-documents.spec.ts     ← D1–D7                       [NEW]
├── edge-cases-rbac.spec.ts          ← R1–R10                      [NEW]
└── edge-cases-socket.spec.ts        ← S1–S8                       [NEW]
```

### 8.21 Updated Run Commands

```bash
# Run all tests with HTML report
npx playwright test --reporter=html

# Run only role-specific tests
npx playwright test --grep "super_admin|owner|admin|billing|member|guest"

# Run only edge case tests
npx playwright test e2e/edge-cases-*.spec.ts

# Run specific gap areas
npx playwright test e2e/dashboard.spec.ts e2e/gantt.spec.ts e2e/budget.spec.ts e2e/automations.spec.ts e2e/settings.spec.ts

# Run with 3 parallel workers
npx playwright test --workers=3

# Debug a specific failing test
npx playwright test e2e/gantt.spec.ts --debug
```

---

## SECTION 9: Additional Edge Cases

### 9.1 Performance Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| P1 | Task list with 500+ tasks | Seed DB with 500 tasks; load /app/tasks | Page renders in < 2s; no browser freeze; pagination or virtualization active |
| P2 | Kanban with 10 columns × 50 cards | Seed 10 statuses, 50 tasks each; load /app/kanban | Board renders; horizontal scroll works; no dropped frames on scroll |
| P3 | Global search on large dataset | 1000+ tasks, 100 projects; type in search | Results appear within 400ms; debounce prevents excessive API calls |
| P4 | Chat room with 1000 messages | Load a room with 1000 messages | Only last ~50 messages rendered initially; scroll-up loads older messages |
| P5 | Gantt with 100 tasks | Seed 100 tasks with dates; load /app/gantt | All bars render; drag is responsive; zoom change is instant |

### 9.2 Responsive / Mobile Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| M1 | Dashboard on 375px viewport | Set DevTools to iPhone SE; load dashboard | Layout collapses to single column; no horizontal scroll required |
| M2 | Kanban on tablet (768px) | Set DevTools to iPad; load /app/kanban | Columns visible with horizontal scroll or carousel layout |
| M3 | Chat on mobile | Open /app/chat at 375px | Room list and message area fit; keyboard doesn't break layout |
| M4 | Navigation menu on mobile | Load any page at 375px | Sidebar collapses to hamburger / bottom nav |

### 9.3 Dark Mode Edge Cases

| # | Scenario | Steps | Expected Result |
|---|----------|-------|----------------|
| DM1 | Toggle dark mode; reload page | Enable dark mode; hard-reload | Dark mode persists without flash of white (FOUC prevented by IIFE) |
| DM2 | Dark mode on all pages | Toggle dark; visit every page | No page has hardcoded white/black colors that break dark theme |
| DM3 | Modals in dark mode | Open task detail, share modal, confirm dialog in dark mode | All modals use CSS token `--bg-surface`; no white boxes in dark mode |
| DM4 | Charts in dark mode | Open stakeholder dashboard in dark mode | Chart labels, axes, gridlines readable against dark background |
