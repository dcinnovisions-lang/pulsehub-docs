# PulseHub — Master Feature Checklist
> Every feature broken down to the lowest actionable level.
> Mark `[x]` when done. **Last updated: 2026-03-19 — Full codebase audit performed.**
>
> **Legend:** ✅ `[x]` Done · 🔲 `[ ]` Not built · ⚠️ Partial (noted inline)

---

## Progress Tracker

| Domain | Total | Done | Remaining |
|--------|-------|------|-----------|
| Authentication | 18 | 18 | 0 |
| User Profile | 8 | 8 | 0 |
| Workspace | 16 | 15 | 1 |
| Projects | 21 | 20 | 1 |
| Tasks | 38 | 37 | 1 |
| Kanban Board | 10 | 9 | 1 |
| Gantt Chart | 12 | 11 | 1 |
| Calendar | 10 | 7 | 3 |
| Notifications | 12 | 11 | 1 |
| Global Search | 8 | 7 | 1 |
| Comments & Reactions | 10 | 9 | 1 |
| Attachments | 8 | 7 | 1 |
| Custom Fields | 8 | 8 | 0 |
| Task Dependencies | 6 | 6 | 0 |
| RBAC & Permissions | 16 | 15 | 1 |
| Invites | 10 | 10 | 0 |
| Activity Log & Audit | 8 | 7 | 1 |
| Settings Page | 16 | 10 | 6 |
| Chat | 18 | 16 | 2 |
| Documents | 12 | 12 | 0 |
| Whiteboard | 14 | 5 | 9 |
| Budget Management | 12 | 8 | 4 |
| Time Tracking | 12 | 9 | 3 |
| Resource Management | 10 | 5 | 5 |
| Workload View | 8 | 5 | 3 |
| Stakeholder Dashboard | 10 | 5 | 5 |
| Main Dashboard | 10 | 6 | 4 |
| Automation Builder | 20 | 16 | 4 |
| Saved Views | 8 | 2 | 6 |
| Landing Page | 14 | 14 | 0 |
| Navigation & Layout | 12 | 12 | 0 |
| Real-time (Socket.io) | 14 | 11 | 3 |
| Dark Mode | 8 | 6 | 2 |
| Error Handling & UX | 10 | 3 | 7 |
| Performance & Polish | 8 | 1 | 7 |
| Test Suite | 20 | 0 | 20 |
| DevOps / Deployment | 10 | 7 | 3 |
| Tech Debt | 10 | 1 | 9 |
| **TOTAL** | **433** | **333** | **100** |

---

## 1. Authentication

### 1.1 Registration
- [x] Email + password register endpoint (`POST /api/v1/auth/register`)
- [x] Input validation (email format, password min 8 chars)
- [x] Password hashing with bcrypt (10 rounds via beforeCreate hook)
- [x] Duplicate email check with proper error message
- [x] RegisterPage UI with form + validation feedback
- [x] Auto-login after successful registration (token returned)
- [x] Email verification on register — token generated, verification email sent (SMTP), non-fatal if SMTP not configured
- [x] Resend verification email button — banner in Layout + resend endpoint

### 1.2 Login
- [x] Email + password login endpoint (`POST /api/v1/auth/login`)
- [x] JWT access token (7d) + refresh token (30d) returned
- [x] LoginPage UI with error handling
- [x] Redirect to `/app/dashboard` after login

### 1.3 Token Refresh
- [x] `POST /api/v1/auth/refresh-token` endpoint
- [x] Axios interceptor auto-refreshes on 401
- [x] On refresh failure → dispatch logout → redirect to `/login`

### 1.4 Google OAuth
- [x] `GET /api/v1/auth/google` + callback endpoint
- [x] Passport.js strategy configured
- [x] OAuthCallbackPage handles token storage
- [x] Google OAuth button on LoginPage

### 1.5 Two-Factor Authentication (2FA)
- [x] `POST /api/v1/auth/2fa/setup` — generates TOTP secret + QR code
- [x] `POST /api/v1/auth/2fa/verify` — verifies TOTP code and enables 2FA
- [x] `POST /api/v1/auth/2fa/disable` — disables 2FA with password confirmation
- [x] `POST /api/v1/auth/2fa/verify-login` — second step for users with 2FA enabled
- [x] TwoFactorSetup.tsx component with QR code display
- [x] Backup codes generation — 8 codes shown once in modal after 2FA enabled, copy-all button
- [x] Backup code recovery login endpoint (`POST /api/v1/auth/backup-login`)

---

## 2. User Profile

### 2.1 Profile Management
- [x] `GET /api/v1/users/:id` endpoint
- [x] `PUT /api/v1/users/:id` endpoint
- [x] Settings → Profile tab (ProfileSettings.tsx)
- [x] Avatar upload — file picker, 2MB limit, preview, saves to `/uploads/avatars/`
- [x] Display name editable (firstName + lastName)
- [x] Change password (currentPassword + newPassword + confirmNewPassword)
- [x] Timezone setting — select dropdown, 21 IANA zones
- [x] Delete my account — danger zone, password confirmation, soft-delete

---

## 3. Workspace

### 3.1 Workspace CRUD
- [x] Create workspace (`POST /api/v1/workspaces`)
- [x] List my workspaces (`GET /api/v1/workspaces`)
- [x] View workspace (`GET /api/v1/workspaces/:id`)
- [x] Update workspace (`PUT /api/v1/workspaces/:id`)
- [x] Delete workspace (`DELETE /api/v1/workspaces/:id`)
- [x] WorkspacesPage with grid of workspace cards
- [x] Workspace settings — edit modal with name/description
- [x] Workspace logo/avatar upload

### 3.2 Workspace Members
- [x] List members (`GET /api/v1/workspaces/:id/members`)
- [x] Add member (`POST /api/v1/workspaces/:id/members`)
- [x] Change member role (`PUT /api/v1/workspaces/:id/members/:userId`)
- [x] Remove member (`DELETE /api/v1/workspaces/:id/members/:userId`)
- [x] WorkspaceMembersManager.tsx UI
- [x] Role dropdown per member
- [x] Notification on member add/remove

---

## 4. Projects

### 4.1 Project CRUD
- [x] Create project (`POST /api/v1/projects`)
- [x] List projects (`GET /api/v1/projects`)
- [x] View project detail (`GET /api/v1/projects/:id`)
- [x] Update project (`PUT /api/v1/projects/:id`)
- [x] Delete project (`DELETE /api/v1/projects/:id`)
- [x] ProjectsPage with card grid
- [x] Project color picker
- [x] Project status (active / archived / completed)
- [x] Archive/Unarchive from card kebab menu
- [ ] Project description rich text (currently plain text input)

### 4.2 Project Templates
- [x] `GET /api/v1/projects/templates` — list all templates
- [x] `POST /api/v1/projects/from-template` — clone template
- [x] Mark project as template (`isTemplate` flag)
- [x] ProjectTemplateGallery.tsx visual card picker
- [x] Pre-seeded default templates (Agile Sprint, Bug Tracker, Marketing Campaign, Product Launch) via `seed-demo.js`

### 4.3 Project Members
- [x] `GET /api/v1/projects/:id/members`
- [x] `POST /api/v1/projects/:id/members`
- [x] `PUT /api/v1/projects/:id/members/:userId`
- [x] `DELETE /api/v1/projects/:id/members/:userId`
- [x] ProjectMembersManager.tsx UI
- [x] 6 project roles (project_lead → viewer)
- [x] Notification on member add/remove
- [x] Bulk-invite multiple emails (tag-input, Enter/comma to add)

### 4.4 Project Health Score
- [x] `calculateHealthScore` controller
- [x] `GET /api/v1/projects/:id/health` endpoint
- [x] ProjectHealthScore.tsx ring chart component

---

## 5. Tasks

### 5.1 Task CRUD
- [x] Create task (`POST /api/v1/tasks`)
- [x] List tasks with filters (`GET /api/v1/tasks?projectId&statusId&priority&assigneeId&search`)
- [x] View task detail (`GET /api/v1/tasks/:id`)
- [x] Update task (`PUT /api/v1/tasks/:id`)
- [x] Delete task (`DELETE /api/v1/tasks/:id`)
- [x] TasksPage list view
- [x] TaskDetailPage full detail view

### 5.2 Task Fields
- [x] Title + description
- [x] Priority (urgent / high / medium / low) with colour badges
- [x] Status (per-project custom statuses)
- [x] Due date + start date
- [x] Estimated hours
- [x] Progress percentage (0–100)
- [x] Multi-assignee with AssigneeSelector
- [x] Position (drag-to-reorder in list)
- [x] Task labels / tags — JSONB column, colour-coded tag input
- [ ] Recurring tasks (repeat: daily / weekly / monthly)

### 5.3 Subtasks
- [x] Create/Update/Delete subtask endpoints
- [x] Subtask assignee + due date + estimated hours
- [x] Subtask completion toggle in UI
- [x] Convert subtask to full task (↗ button → createTask + deleteSubtask)
- [x] Subtask progress rolled up to parent task % (Auto-calculate button)

### 5.4 Bulk Operations
- [x] `POST /api/v1/tasks/bulk` — bulk create
- [x] `PUT /api/v1/tasks/bulk` — bulk update (status, priority, assignee)
- [x] `DELETE /api/v1/tasks/bulk` — bulk delete
- [x] `POST /api/v1/tasks/bulk/archive` — bulk archive
- [x] Checkbox selection UI in task list
- [ ] Bulk move to different project/list

### 5.5 Import / Export
- [x] `GET /api/v1/tasks/export` — export to CSV
- [x] `POST /api/v1/tasks/import` — import from CSV
- [x] Export button + Import modal with file picker in TasksPage

### 5.6 Task Reordering
- [x] `POST /api/v1/tasks/reorder` — update positions
- [x] Drag-to-reorder in list view

---

## 6. Kanban Board

- [x] `GET /api/v1/kanban/:projectId` — board data grouped by status
- [x] `POST /api/v1/kanban/move` — move task to new status + position
- [x] KanbanBoardPage with drag-and-drop (react-beautiful-dnd)
- [x] Column headers with task count badges
- [x] Task cards with priority, assignee avatar, due date
- [x] Optimistic UI update on drag
- [x] Create task inline in column (+ button)
- [x] Column colour coded by status
- [x] Automations fire on status change (via task.controller.js)
- [x] WIP limits per column (inline settings gear + warning badge + toast on exceed)

---

## 7. Gantt Chart

- [x] `GET /api/v1/gantt/:projectId` — timeline data
- [x] GanttPage renders timeline with task bars
- [x] Multi-level zoom (Day / Week / Month / Year)
- [x] Date navigation (prev / next / today)
- [x] Priority colour coding on bars
- [x] Assignee avatars in sidebar
- [x] Dependency lines (SVG arrows)
- [x] Progress bars within task bars
- [x] Drag-to-move task bar (change start/due date)
- [ ] Drag bar edge to resize (change duration)
- [x] Milestone markers (diamond shape)
- [x] Export Gantt as PNG / PDF

---

## 8. Calendar View

- [x] `GET /api/v1/calendar` — fetch tasks by date range
- [x] CalendarPage with Month / Week / Day views
- [x] Event cards with priority colours
- [x] Navigation (prev / next / today)
- [x] Project filter dropdown
- [x] Multi-day task span display (start → due date)
- [x] Create task by clicking empty date cell *(confirmed in code — newTaskModal state + createTask dispatch)*
- [ ] Edit task due date by dragging event card
- [ ] Recurring event display
- [ ] iCal / Google Calendar export

---

## 9. Notifications

### 9.1 Backend Triggers
- [x] `task_assigned` — task created or updated with assignees
- [x] `task_comment` — new comment on a task
- [x] `project_member_added` / `project_member_removed`
- [x] `workspace_member_added` / `workspace_member_removed`
- [x] `invite_accepted`
- [x] `task_due_soon` — cron fires 24h before due date (server.js cron, runs every hour)
- [x] `task_overdue` — cron fires when task passes due date

### 9.2 Real-time Delivery
- [x] `createNotification()` emits `notification:new` via Socket.io (socket.js line ~49)
- [x] Layout.tsx connects to socket on login, listens for `notification:new`, dispatches `addNotification`
- [x] Socket disconnects on logout

### 9.3 Frontend UI
- [x] NotificationBell.tsx with unread count badge
- [x] Dropdown list (title, body, time ago, actor avatar, type icon)
- [x] Mark single as read / Mark all as read
- [x] Delete single / Clear all read
- [x] Click notification → navigate to entity (via `notif.metadata.url`)
- [x] 60-second polling fallback
- [ ] Per-user notification preferences (email on/off, in-app on/off, digest)

---

## 10. Global Search

- [x] `GET /api/v1/search?q=&types=` — multi-entity search
- [x] Searches: tasks, projects, workspaces, users, comments, documents
- [x] GlobalSearch.tsx modal (Cmd+K / Ctrl+K)
- [x] Keyboard navigation (↑↓ arrows, Enter)
- [x] Result groups with entity type icons
- [x] Click result → navigate to entity
- [x] Loading state and empty state
- [ ] Recent searches history (localStorage)

---

## 11. Comments & Reactions

- [x] `GET/POST /api/v1/tasks/:id/comments`
- [x] `PUT/DELETE /api/v1/tasks/:id/comments/:cid`
- [x] `POST/DELETE /api/v1/tasks/:id/comments/:cid/reactions`
- [x] CommentList.tsx with avatar, timestamp, edit/delete
- [x] Emoji reaction display + toggle
- [x] Comment notification trigger
- [ ] @mention autocomplete in comment textarea

---

## 12. Attachments

- [x] `POST /api/v1/tasks/:id/attachments` — upload (multer)
- [x] `GET /api/v1/tasks/:id/attachments`
- [x] `GET /api/v1/attachments/:id/download`
- [x] `DELETE /api/v1/attachments/:id`
- [x] AttachmentList.tsx — file icon, size, uploader name, download button
- [x] File type icons (pdf, image, doc, etc.)
- [ ] Image preview thumbnail in attachment list
- [ ] File size limit warning (currently silent fail on oversized files)

---

## 13. Custom Fields

- [x] `GET /api/v1/custom-fields?projectId=`
- [x] `POST / PUT / DELETE /api/v1/custom-fields`
- [x] `POST /api/v1/task-custom-fields` — set value
- [x] `GET /api/v1/task-custom-fields/:taskId`
- [x] CustomFieldConfig.tsx — define fields per project
- [x] CustomFieldInput.tsx — renders correct input type (text / number / select / date / checkbox)

---

## 14. Task Dependencies

- [x] `POST /api/v1/task-dependencies`
- [x] `DELETE /api/v1/task-dependencies/:id`
- [x] `GET /api/v1/tasks/:id/dependencies`
- [x] Dependency types: `blocks` / `blocked_by` / `relates_to`
- [x] TaskDependencyManager.tsx UI
- [x] Dependency lines in Gantt chart (SVG)

---

## 15. RBAC & Permissions

### 15.1 Platform Roles (users.role)
- [x] `super_admin` — bypasses all checks, `/app/users` + `/app/admin`
- [x] `admin`, `owner`, `billing_admin`, `pm`, `member`, `commenter`, `guest`, `viewer`
- [x] `authorize(...roles)` middleware on protected endpoints
- [x] `PATCH /api/v1/users/:id/role` — super_admin only

### 15.2 Workspace Roles (workspace_members.role)
- [x] `owner | admin | billing_admin | member | guest` (+ legacy pm/viewer/commenter)
- [x] WorkspaceMembers junction table with role enforcement

### 15.3 Project Roles (project_members.role)
- [x] `project_lead | contributor | reporter | reviewer | commenter | viewer`
- [x] ProjectMembers junction table with role enforcement

### 15.4 Frontend RBAC
- [x] PermissionMatrix.tsx — visual role-permission table
- [x] PermissionGuard.tsx — conditional render by role
- [x] RoleBasedSidebar.tsx — hides nav items by role
- [x] Settings tabs gated by role (admin+ only for RBAC/audit/templates)
- [ ] GuestAccess UI — create/manage scoped guest links (backend model exists, no UI)

---

## 16. Invite System

- [x] `POST /api/v1/invites` — send invite email with token
- [x] `GET /api/v1/invites/:token` — validate token, return invite details
- [x] `POST /api/v1/invites/:token/accept` — accept invite
- [x] `GET /api/v1/invites` — list pending invites
- [x] `DELETE /api/v1/invites/:id` — cancel invite
- [x] AcceptInvitePage.tsx with workspace/project info
- [x] Invite email via Nodemailer (HTML template)
- [x] 7-day token expiry; statuses: `pending | accepted | cancelled | expired`
- [x] `invite_accepted` notification to inviter

---

## 17. Activity Log & Audit

- [x] ActivityLog model (entityType, entityId, action, userId, changes, metadata)
- [x] `GET /api/v1/activity-logs` endpoint
- [x] `createActivityLog()` utility called in controllers
- [x] ActivityTimeline.tsx component (feed of events with icon + actor + description)
- [x] Settings → Audit Log tab shows timeline with workspace filter
- [x] Limits to 100 entries per fetch
- [x] Event icons + relative timestamps
- [ ] Pagination / "Load more" in audit timeline
- [ ] Filter by entity type (task / project / member)
- [ ] Export audit log as CSV

---

## 18. Settings Page

### 18.1 Tab: Profile *(confirmed present — ProfileSettings.tsx)*
- [x] Edit first name / last name
- [x] Avatar upload with preview
- [x] Bio text area
- [x] Timezone dropdown
- [x] Change password form
- [ ] Delete account from settings (currently only from profile page directly)

### 18.2 Tab: Security
- [x] TwoFactorSetup.tsx embedded
- [x] 2FA status (enabled / disabled) with QR + verify form
- [x] Disable 2FA button
- [ ] Active sessions list (which devices are logged in)
- [ ] Session revoke button

### 18.3 Tab: Permissions (admin only)
- [x] PermissionMatrix.tsx — read-only role-permission table
- [ ] "Save changes" flow for custom permission overrides (P2)

### 18.4 Tab: Audit Log (admin only)
- [x] Workspace selector dropdown
- [x] ActivityTimeline renders audit events
- [ ] Date range filter
- [ ] Entity type filter (member / task / project)
- [ ] Export audit log CSV

### 18.5 Tab: Templates (admin only)
- [x] Lists workspace templates with colour + name
- [ ] Delete template
- [ ] Edit template name/colour
- [ ] Create blank template from settings

### 18.6 Missing Tabs
- [x] **Notifications tab** — per-type on/off toggles (email + in-app)
- [x] **Workspace tab** — workspace name, logo, timezone, danger zone
- [x] **API Keys tab** — generate/revoke personal access tokens
- [x] **Integrations tab** — connect Slack, GitHub (stubs)

---

## 19. Chat

### 19.1 Backend
- [x] ChatRoom model + `POST/GET /api/v1/chat/rooms`
- [x] ChatMessage model + `POST/GET /api/v1/chat/rooms/:id/messages`
- [x] Socket.io: `chat:join`, `chat:leave`, `chat:send_message`, `chat:typing`

### 19.2 Frontend (ChatPage.tsx)
- [x] Room list sidebar
- [x] Create room modal
- [x] Message display (sender, timestamp)
- [x] Message composer textarea + send
- [x] Typing indicator emit
- [x] Typing indicator display ("Sarah is typing…" UI)
- [x] Emoji reactions on messages
- [x] Edit message
- [x] Delete message
- [ ] Reply/thread on a message
- [ ] File/image attachment in message
- [ ] @mention autocomplete dropdown
- [ ] Read receipts ("Seen by N")
- [x] Unread message count badge on rooms
- [x] Scroll-to-bottom button
- [x] Message history pagination (infinite scroll up)
- [x] Search messages within a room

---

## 20. Documents

### 20.1 Backend
- [x] Document model (title, content, projectId, workspaceId)
- [x] Full CRUD endpoints (`GET/POST/PUT/DELETE /api/v1/documents`)

### 20.2 Frontend (DocumentsPage.tsx — plain textarea)
- [x] Create / update / delete document
- [x] Document list sidebar
- [x] **Replace textarea with rich text editor** (Tiptap or Quill)
  - [x] Bold / Italic / Underline / Strikethrough
  - [x] Headings (H1, H2, H3)
  - [x] Bullet list + Numbered list
  - [x] Blockquote + Code block (syntax highlighted)
  - [x] Hyperlinks
  - [x] Image embed (upload or URL)
  - [x] Tables
- [x] Auto-save on change (debounced 2s)
- [x] Version history (list saves, restore)
- [ ] Document sharing link (public/private toggle)
- [ ] Collaborative editing (socket.io content sync)

---

## 21. Whiteboard

### 21.1 Backend
- [x] Whiteboard + WhiteboardElement models
- [x] CRUD endpoints + element upsert
- [x] Socket.io: `whiteboard:join`, `whiteboard:leave`, `whiteboard:update`, `whiteboard:delete`, `whiteboard:cursor`

### 21.2 Frontend (WhiteboardPage.tsx — full rewrite)
- [x] Create whiteboards
- [x] Sticky note, shape (rect/circle), text element creation
- [x] Socket.io join/leave room
- [x] Real-time element sync (`whiteboard:update` listener)
- [x] Real-time cursor tracking (`whiteboard:cursor` listener + display labels)
- [x] Drag-and-drop to move elements
- [x] Resize elements (8-handle resize corners)
- [x] Edit text inline (double-click)
- [x] Delete element (× button + Delete key)
- [ ] Draw tool (pencil / pen path)
- [x] Shape tools (sticky, rectangle, circle, text)
- [x] Color picker (8-color predefined palette)
- [x] Undo / Redo (Ctrl+Z / Ctrl+Y, 50-level history)
- [x] Collaborator cursor display (dot + username label)
- [x] Zoom in / out / fit-to-screen (wheel + buttons, 0.2×–3×)
- [x] Export whiteboard as PNG (Canvas 2D, 2× HiDPI)

---

## 22. Budget Management

### 22.1 Backend
- [x] Budget + BudgetExpense models
- [x] CRUD endpoints (create budget, add expense, list, get by project)

### 22.2 Frontend (BudgetManagementPage.tsx)
- [x] Create budget modal (name, amount, currency, dates)
- [x] Add expense modal (category, amount, description, date)
- [x] Budget cards with progress bar (red if exceeded)
- [x] Summary cards (Total Budget, Total Spent, Remaining, Over Budget count)
- [x] Expense list per budget (expandable)
- [x] Status badge (active / exceeded / closed)
- [ ] Edit budget (update name, total amount, dates)
- [ ] Edit/update individual expense
- [ ] Budget spend-over-time line chart
- [x] Expense breakdown by category (pie/donut chart)
- [x] Budget threshold alert (notify at 80% spent)
- [x] Export budget report as CSV

---

## 23. Time Tracking

### 23.1 Backend
- [x] TimeTracking + TimeLog models
- [x] Manual log endpoints (`GET/POST/PUT/DELETE /api/v1/time-logs`)

### 23.2 Frontend (TimeTrackingPage.tsx + TimeTracker.tsx)
- [x] Manual log form (task picker, hours, minutes, description, date)
- [x] Time log table with delete
- [x] Statistics cards (Total Time, Entries, Active Users, Projects)
- [x] **Live timer widget (TimeTracker.tsx)** — Play / Pause / Stop, HH:MM:SS display
- [x] Auto-submit log on Stop
- [x] Time stats by project
- [x] Weekly hours bar chart (hours logged per day)
- [x] Export timesheet as CSV
- [ ] Billable vs non-billable flag per entry
- [x] Filter logs by project, date range

---

## 24. Resource Management

### 24.1 Backend
- [x] Resource model (userId, projectId, allocation %, dates)
- [x] `GET /api/v1/resources` (getWorkload) + `POST` (createOrUpdateResource)

### 24.2 Frontend (ResourceManagementPage.tsx)
- [x] Resource cards with avatar, utilization bar, assigned tasks
- [x] Availability badge (available / busy / overallocated / on_leave)
- [x] Utilization % with colour coding
- [ ] Edit resource allocation % inline
- [ ] Gantt-style timeline view of allocations
- [ ] Capacity planning — available hours per week
- [ ] Conflict detection — highlight double-booked users
- [ ] Export resource plan as CSV

---

## 25. Workload View

### 25.1 Frontend (WorkloadViewPage.tsx)
- [x] Resource cards with capacity / logged / estimated hours
- [x] Week / Month view toggle
- [x] Navigation (prev / next period)
- [x] Summary cards (Total Resources, Avg Utilization, Over Capacity)
- [x] Utilization bar per resource
- [ ] Gantt-style capacity bars per day/week
- [ ] Drag-to-reassign tasks between team members
- [ ] Team availability heatmap (dark = overloaded, light = free)

---

## 26. Stakeholder Dashboard

### 26.1 Backend
- [x] `GET /api/v1/dashboard/stakeholder` — aggregated KPI data

### 26.2 Frontend (StakeholderDashboardPage.tsx)
- [x] 4 KPI summary cards (Total / On Track / At Risk / Off Track projects)
- [x] Performance metrics (completion rate bar, overdue count, health avg)
- [x] Projects overview grid with health score + task stats
- [x] Click project → navigate to ProjectDetailPage
- [ ] Health score trend chart (line chart over last 30 days)
- [ ] Budget vs actual spending chart
- [ ] Resource utilization chart
- [ ] Custom date range selector
- [ ] Export dashboard as PDF

---

## 27. Main Dashboard

### 27.1 Frontend (DashboardPage.tsx)
- [x] Welcome greeting with user name
- [x] 4 stat cards: Total Tasks, Completed, In Progress, Overdue
- [x] My Tasks section (5 most recent assigned to me)
- [x] Quick Actions (Create Workspace / Project / Task)
- [x] Recent Activity section
- [x] Task completion trend chart (sparkline SVG — created/completed per day)
- [ ] Workspace-level filter (stats for selected workspace)
- [x] Upcoming deadlines widget (tasks due in next 7 days)
- [ ] Team activity feed (not just mine)

---

## 28. Automation Builder

### 28.1 Backend
- [x] Automation model (trigger JSONB, actions JSONB, isActive, runCount, lastRunAt)
- [x] Migration `20260319000001-create-automations.js`
- [x] `automationEngine.js` — evaluates triggers, executes actions
- [x] Full CRUD + toggle endpoints (`GET/POST/PUT/DELETE/PATCH /api/v1/automations`)
- [x] Hooked into task.controller.js — fires on `task_created`, `task_status_changed`, `task_assigned`, `task_priority_changed`

### 28.2 Trigger Types (7)
- [x] `task_created`
- [x] `task_status_changed`
- [x] `task_assigned`
- [x] `task_priority_changed`
- [x] `task_overdue`
- [x] `task_due_soon`
- [x] `comment_added`

### 28.3 Action Types (5)
- [x] `change_status`
- [x] `set_priority`
- [x] `assign_user`
- [x] `send_notification`
- [x] `post_comment`

### 28.4 Frontend
- [x] `automationService.ts` — full typed API client
- [x] `automationSlice.ts` — Redux state (fetch / create / update / delete / toggle)
- [x] `AutomationsPage.tsx` — list with stats bar, skeleton loading, empty state
- [x] `AutomationBuilder.tsx` — modal builder (trigger card grid + action list + validation)
- [x] Sidebar link at `/app/automations` (visible to owner/admin/pm/super_admin)

### 28.5 Not Yet Built
- [x] Automation execution log (UI to see what ran, when, result)
- [x] Trigger: "When comment added" hook in comment.controller.js
- [x] Action: `create_subtask` type
- [x] Conditions on triggers (e.g. "only if priority = urgent")

---

## 29. Saved Views

### 29.1 Backend
- [x] SavedView model (name, filters JSON, sortBy JSON, userId, projectId)
- [x] 5 CRUD endpoints (`GET/POST/PUT/DELETE /api/v1/saved-views`)

### 29.2 Frontend
- [x] "Save current view" button in TasksPage filter bar
- [x] Save View modal (name field + share toggle)
- [x] Saved Views dropdown/sidebar in TasksPage
- [x] Apply saved view (restore filters + sort)
- [ ] Rename saved view
- [x] Share saved view with team (make public to project)

---

## 30. Landing Page

- [x] Sticky frosted-glass nav (scroll state shadow)
- [x] Mobile hamburger menu with animated open/close
- [x] Hero section — copy left + Kanban mockup right
- [x] Kanban mockup with real columns (To Do / In Progress / Done)
- [x] Stats bar (10k+ teams, 98% satisfaction, 3×, 99.9% uptime)
- [x] Features grid — 6 emoji tiles (3×2)
- [x] Capabilities section — 3 alternating rows with UI panels
- [x] Pricing — 3 plan cards (Starter / Pro / Enterprise)
- [x] Testimonials — 3 cards with 5-star ratings
- [x] Integrations — 8 tool logos (Slack, GitHub, Zapier, Notion…)
- [x] Final CTA section (dark gradient)
- [x] Footer — 4-column links
- [x] AboutUsPage (`/about`)
- [x] ContactPage (`/contact`)

---

## 31. Navigation & Layout

- [x] `Layout.tsx` — shell (Sidebar + Header + main outlet)
- [x] `Header.tsx` — GlobalSearch + NotificationBell + user menu
- [x] `RoleBasedSidebar.tsx` — role-gated nav items
- [x] `PrivateRoute` / `PublicRoute` / `LandingRoute` guards
- [x] Active route highlight in sidebar
- [x] Sidebar collapsible on mobile
- [x] Email verification banner (unverified users)
- [x] Socket.io connected in Layout.tsx on login
- [x] Socket disconnects on logout
- [x] 404 Not Found page (currently redirects to `/` — no friendly error page)
- [x] Loading skeleton states (replace blank screens during fetch)
- [x] Error boundary component (catch render crashes, show fallback)

---

## 32. Real-time (Socket.io)

### 32.1 Backend (socket.js)
- [x] Socket.io server attached to HTTP server
- [x] `authenticate` event — maps userId → socketId
- [x] Emits `notification:new` via `io.to(socketId).emit(...)` when createNotification is called
- [x] Chat: `chat:join`, `chat:leave`, `chat:send_message`, `chat:typing`
- [x] Whiteboard: `whiteboard:join`, `whiteboard:leave`, `whiteboard:update`, `whiteboard:delete`, `whiteboard:cursor`
- [x] Emit `task:updated` when task is mutated (for real-time board refresh)
- [ ] Emit `member:added` when workspace/project member changes

### 32.2 Frontend (Layout.tsx + socketClient.ts)
- [x] Connect socket on login (Layout.tsx), disconnect on logout
- [x] Listen for `notification:new` → dispatch `addNotification`
- [x] Chat: `joinChatRoom`, `sendChatMessage`, `onChatMessage`, `onTyping`
- [x] Whiteboard: `joinWhiteboard`, `onWhiteboardUpdate`, `onCursorMove`
- [x] Connection status indicator in Header (green/red dot)
- [ ] Auto-reconnect on network restore
- [x] Listen for `task:updated` → refresh Kanban / task list in real time

---

## 33. Dark Mode

- [x] CSS variables defined in `index.css` (primary, secondary, gray scale, shadows, transitions)
- [x] `@media (prefers-color-scheme: dark)` base override in index.css
- [x] Full dark token overrides for all components (cards, sidebar, modals, tables, inputs)
- [x] Toggle button in Settings → Profile tab
- [x] Persist theme in `localStorage`
- [x] Apply on load before render (prevent flash of wrong theme)
- [ ] Dark mode for LandingPage
- [ ] Dark mode for all modal overlays

---

## 34. Error Handling & UX Polish

- [x] Global `errorHandler.js` middleware (backend)
- [x] Toast system (Toast.tsx + ToastContainer.tsx)
- [ ] Show toast on every API error (currently some errors are silent)
- [ ] Form field inline validation errors (red text under input)
- [ ] Empty states for all list pages (illustration + CTA)
- [ ] Loading skeletons: Tasks list, Project cards, Members list, Notifications
- [ ] Confirm dialogs before destructive actions (delete task, remove member, delete project)
- [ ] Unsaved changes warning on navigating away from edit forms
- [x] Offline banner ("You're offline — changes won't be saved")
- [ ] Dynamic page titles per route (`document.title = "Tasks — PulseHub"`)

---

## 35. Performance & Polish

- [x] Lazy-load page components (`React.lazy` + `Suspense` in App.tsx)
- [ ] Memoize expensive selectors (`createSelector` / reselect)
- [ ] Virtualize long task lists (`react-window`) for 500+ tasks
- [ ] Image optimization — compress/resize avatar uploads on backend
- [ ] HTTP response compression (verify `compression` middleware is active)
- [ ] Bundle analysis + tree-shake unused imports
- [ ] API response caching with React Query for non-Redux pages
- [ ] Debounce all search/filter inputs consistently

---

## 36. Test Suite

### 36.1 Backend (Jest + Supertest)
- [ ] Test setup — `jest.config.js`, test DB (`.env.test`)
- [ ] Auth: register, login, JWT, 2FA flow, backup codes
- [ ] User: CRUD, role change
- [ ] Workspace: CRUD, member add/remove
- [ ] Project: CRUD, template clone
- [ ] Task: CRUD, bulk ops, reorder, automations fire
- [ ] Notification: trigger fires, mark read, socket emit
- [ ] Search: returns correct entity types
- [ ] Permissions: 403 on unauthorized access
- [ ] Invites: send, accept, expiry
- [ ] Integration: full task lifecycle (create → assign → comment → complete)

### 36.2 Frontend (Vitest + React Testing Library)
- [ ] Test setup — Vitest config, mock store, mock Axios
- [ ] LoginPage — form submit, error display, 2FA flow
- [ ] NotificationBell — renders badge, marks read, click navigates
- [ ] GlobalSearch — Cmd+K opens, search returns results
- [ ] KanbanBoard — drag card to new column
- [ ] TaskDetailPage — renders task, posts comment
- [ ] PermissionGuard — hides/shows by role
- [ ] AuthSlice — login thunk, logout, token refresh
- [ ] AutomationBuilder — create automation, toggle, delete

---

## 37. DevOps / Deployment

- [x] `Dockerfile` for backend (Node 18 Alpine)
- [x] `Dockerfile` for frontend (build → Nginx)
- [x] `docker-compose.yml` (backend + frontend + PostgreSQL)
- [ ] `.env.example` for backend + frontend with all required keys
- [x] GitHub Actions CI — lint → test → build
- [x] GitHub Actions CD — build Docker → push to registry
- [x] Nginx config for SPA (catch-all to `index.html`)
- [ ] Database backup strategy (`pg_dump` cron)
- [x] Health check endpoint `GET /api/v1/health` → `{ status: "ok" }`
- [ ] Environment configs (development / staging / production)

---

## 38. Tech Debt

- [x] Rename `backend/package.json` name from `trackpro-server` → `pulsehub-server`
- [x] Remove duplicate `resource.slice.ts` (keep `resourceSlice.ts`)
- [x] Remove duplicate `timeTracking.slice.ts` (keep `timeTrackingSlice.ts`)
- [x] Remove duplicate `resource.service.ts` (keep `resourceService.ts`)
- [x] Remove duplicate `timeTracking.service.ts` (keep `timeTrackingService.ts`)
- [ ] Remove unused `express-session` dependency from backend
- [x] Audit and remove dead `console.log` statements
- [ ] Standardize API response envelope (`{ success, data }` is current standard — fix any outliers)
- [x] Add `.env.example` to both backend/ and frontend/
- [x] Backend `.env` exists with working DB credentials

---

## Quick Reference — Top 20 Remaining Items (by impact)

| # | Item | Effort | Impact |
|---|------|--------|--------|
| 1 | 404 page + Error boundary | S (1h) | 🔴 High |
| 2 | Rich text editor — Documents (Tiptap) | L (1–2 days) | 🔴 High |
| 3 | Dark mode — full implementation | L (1–2 days) | 🔴 High |
| 4 | Loading skeletons on all list pages | M (4h) | 🔴 High |
| 5 | Saved Views UI (save/apply filters) | M (1 day) | 🟡 Medium |
| 6 | Settings → Notifications tab | M (3h) | 🟡 Medium |
| 7 | Settings → Workspace tab | M (2h) | 🟡 Medium |
| 8 | Whiteboard: drag, resize, delete, color | L (1–2 days) | 🟡 Medium |
| 9 | Gantt drag-to-reschedule | L (2 days) | 🟡 Medium |
| 10 | Chat: reactions, edit, delete, unread count | M (1 day) | 🟡 Medium |
| 11 | Budget: edit budget + expense + charts | M (4h) | 🟡 Medium |
| 12 | Time tracking: export CSV + billable flag | S (2h) | 🟡 Medium |
| 13 | Automation: execution log UI | M (4h) | 🟡 Medium |
| 14 | Automation: comment_added hook in controller | S (30 min) | 🟡 Medium |
| 15 | Socket: emit `task:updated` for real-time board | S (1h) | 🟡 Medium |
| 16 | Notification preferences UI | M (3h) | 🟡 Medium |
| 17 | Per-user API Keys (personal access tokens) | M (1 day) | 🟢 Low |
| 18 | GuestAccess UI | M (4h) | 🟢 Low |
| 19 | Workspace logo upload | S (2h) | 🟢 Low |
| 20 | Tech debt cleanup (duplicates, rename) | S (1h) | 🟢 Low |

---

*Update `[ ]` → `[x]` as each item is completed.*
*Every item = one discrete, testable deliverable.*
