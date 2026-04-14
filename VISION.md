# PulseHub — Product Vision & Architecture Overview
> The single source of truth for what we are building, who it is for, and how it works.
> Last updated: 2026-03-19

---

## What Is PulseHub?

**PulseHub** is a full-stack, multi-tenant SaaS project management and team collaboration platform.

It is a direct competitor to tools like **ClickUp, Asana, Linear, and Monday.com** — but built from scratch, fully owned, and designed to be extended with AI and custom integrations.

The goal is a **single platform** where a team can:
- Plan and track work (tasks, projects, sprints)
- Communicate (chat, comments, notifications)
- Visualize progress (Kanban, Gantt, Calendar, Workload, Dashboard)
- Automate repetitive actions (no-code Automation Builder)
- Manage people and permissions (3-tier RBAC)
- Track time, budgets, and resources

---

## Who Is It For?

| Audience | Use Case |
|----------|----------|
| **Engineering teams** | Sprint planning, bug tracking, PR-linked tasks |
| **Product teams** | Roadmaps, milestones, stakeholder reporting |
| **Marketing teams** | Campaign planning, content calendars |
| **Agencies** | Client project management, guest access for clients |
| **Any mid-size team** | Daily task management, team communication, progress visibility |

**Primary target:** Teams of 5–200 people who currently juggle multiple tools (Slack + Jira + Notion + spreadsheets) and want one integrated platform.

---

## The 3-Tier Access Architecture

This is the most important architectural concept in PulseHub. Everything about who can do what flows through three layers.

---

### Tier 1 — Platform (Super Admin)

The top-level system administrator. Manages the entire PulseHub installation.

- Only one or a small number of users have this role (`super_admin`)
- Can see, edit, and delete anything across all workspaces
- Has a dedicated **User Management** page (`/app/users`) and **Admin Panel** (`/app/admin`)
- Bypasses all workspace and project permission checks
- Set in the `users.role` database column
- Assigned via script (`npm run create:super-admin`) — not through the UI

> Think of it like a system operator or SaaS owner. They don't belong to any specific workspace — they manage the platform.

---

### Tier 2 — Workspace

A **workspace** is a company, team, or organisation's space inside PulseHub. Each workspace has its own projects, members, and settings.

Users are invited to a workspace and assigned a **workspace role**:

| Role | What they can do |
|------|-----------------|
| `owner` | Full control — settings, billing, delete workspace, transfer ownership. One per workspace. |
| `admin` | Delegated manager — manage members, projects, automations. Cannot delete workspace. |
| `billing_admin` | Finance-only — billing dashboard access. No project access. |
| `member` | Default role. What they can actually do is controlled by their **project role** (Tier 3). |
| `guest` | External/client user. Must have a scoped `guest_access` record per project. |

> Legacy roles (`pm`, `viewer`, `commenter`) still exist in the database for backward compatibility but are superseded by project roles in the current RBAC V2 system.

Workspace roles are assigned at the time of **invite** — the invite email pre-assigns the role, so the user has the right access the moment they join.

**Key pages:** WorkspacesPage (`/app/workspaces`), Workspace Members Manager (inside workspace settings)

---

### Tier 3 — Project

Within a workspace, each project has its own membership list and role assignments. A user can have **different roles in different projects** — even if their workspace role is just `member`.

| Role | Label | What they can do |
|------|-------|-----------------|
| `project_lead` | Project Lead | Full project control — settings, statuses, members, tasks, budget, automations |
| `contributor` | Contributor | Default — create/update tasks, log time, add comments, upload files |
| `reporter` | Reporter | Create tasks and report status; cannot edit others' work |
| `reviewer` | Reviewer | Review and approve tasks; can comment and change status |
| `commenter` | Commenter | View tasks, add comments only. Cannot create or edit tasks. |
| `viewer` | Viewer | Read-only access to the project |

Project roles are managed via the **Project Members Manager** inside each project's settings tab.

**Key pages:** ProjectDetailPage (`/app/projects/:id`), Project Members Manager (settings tab inside project)

---

### How the Three Tiers Work Together

```
Platform Level
└── Super Admin (users.role = 'super_admin')
    └── bypasses everything

Workspace Level  (workspace_members.role)
└── Workspace A
    ├── owner  → full workspace control
    ├── admin  → manage workspace (no delete/billing)
    ├── billing_admin → billing only
    ├── member → access determined by project role
    └── guest  → scoped to specific projects

    Project Level  (project_members.role)
    └── Project X
        ├── project_lead  → full project control
        ├── contributor   → create & update tasks
        ├── reporter      → create tasks, report status
        ├── reviewer      → review & approve tasks
        ├── commenter     → comment only
        └── viewer        → read only
    └── Project Y
        └── (different roles for same user — independent)
```

**Permission resolution order** (how the system decides what a user can do):

1. **Super Admin** → always allowed
2. **Workspace owner** → always allowed within their workspace
3. **Project role** → checked if user has a `project_members` record
4. **Workspace role** → fallback if no project-level record exists
5. **Deny** → if none of the above grant access

**The sidebar** (`RoleBasedSidebar.tsx`) uses a simplified version of this for navigation visibility — it maps the full role set to a display-level check so users only see menu items they have access to.

---

## What Is Built Today

### Core Platform (100% done)
- Multi-tenant workspace and project system
- Full task management (CRUD, subtasks, dependencies, bulk ops, CSV import/export)
- 3-tier RBAC with invite system and permission guards
- Authentication: Email/password + Google OAuth + TOTP 2FA + email verification
- Real-time notifications via Socket.io (push on task events)
- Global search (Cmd+K) across tasks, projects, users, documents
- Audit log with ActivityTimeline UI
- Demo seed accounts (`npm run seed:demo`)

### Views & Visualization (mostly done)
- List view (tasks page with filters)
- Kanban board (drag and drop, per-project)
- Gantt chart (timeline, dependency lines, multi-zoom)
- Calendar view (month/week/day)
- Workload view (team capacity)
- Stakeholder dashboard (KPIs, project health grid)
- Main dashboard (personal stats, quick actions)

### Collaboration (mostly done)
- Comments + emoji reactions on tasks
- File attachments on tasks
- Chat (rooms, messages, Socket.io)
- Whiteboard (sticky notes, Socket.io sync)
- Documents (basic, no rich text yet)

### Management Tools (mostly done)
- Time tracking (manual log)
- Resource management (allocation, utilization)
- Budget management (budget + expenses, progress bar)
- Project health score (automated, not manual)
- Project templates gallery

### Automation Builder (100% done — just built)
- Backend: `Automation` model, migration, engine (`automationEngine.js`), controller, routes
- Engine hooks into task events: fires on create, status change, assignment, priority change
- 7 trigger types: `task_created`, `task_status_changed`, `task_assigned`, `task_priority_changed`, `task_overdue`, `task_due_soon`, `comment_added`
- 5 action types: `change_status`, `set_priority`, `assign_user`, `send_notification`, `post_comment`
- Scope: workspace-wide or scoped to a single project
- Frontend: `AutomationsPage` (`/app/automations`), `AutomationBuilder` modal
- Accessible to: `owner`, `admin`, `pm`, `super_admin`

---

## What Is Partially Built (needs more work)

| Feature | What works | What's missing |
|---------|-----------|----------------|
| Gantt | View, zoom, dependency lines | Drag to reschedule, milestone markers, export |
| Calendar | Month/week/day view | Click to create task, drag to reschedule |
| Chat | Rooms, messages, Socket.io | Reactions, edit/delete, threads, read receipts |
| Whiteboard | Sticky notes, socket sync | Drag, draw tools, shapes, zoom, export |
| Documents | CRUD, list | Rich text editor, auto-save, version history |
| Time tracking | Manual log, stats | Live timer widget, charts, export |
| Notifications | Bell, unread count, mark read | Click → navigate to entity, per-user preferences |
| Settings page | Security, Permissions, Audit Log, Templates | Profile tab, Notifications tab, Workspace tab |
| Workload | Capacity bars, navigation | Gantt-style allocation view, heatmap |

---

## What Is Not Built Yet (Roadmap)

### High Priority — next to build
| Item | Why |
|------|-----|
| **Test suite** | Zero coverage on backend controllers and frontend components. Needed before any real deployment. |
| **Rich text documents** | Tiptap or Quill editor — biggest missing feature vs Notion/ClickUp |
| **AI features** | Claude API integration — task prioritization, health advisor, writing assist |
| **Dark mode** | Expected by modern users; 0% implemented |
| **Notification preferences** | Per-user email on/off, in-app on/off, digest mode |

### Medium Priority
| Item | Why |
|------|-----|
| **Saved Views** | Backend model done; no frontend UI |
| **Slack / GitHub integrations** | Extends the Automation Builder we just built |
| **Advanced reporting** | Burndown charts, velocity, PDF export |
| **Live timer** (Time Tracking) | Start/pause/stop stopwatch widget |
| **@mentions** in comments | Expected in any collaborative tool |
| **Goals / OKRs module** | Needed to match Asana |

### Long-term
| Item | Notes |
|------|-------|
| Mobile apps (iOS/Android) | After web is stable |
| SSO (SAML/OIDC) | Enterprise tier |
| Zapier / webhook support | Integration layer |
| Templates marketplace | Pre-seeded Agile, marketing, etc. templates |
| Multi-language (i18n) | International market |

---

## Tech Stack at a Glance

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TypeScript + Vite |
| State management | Redux Toolkit + Redux Persist (auth only) |
| Routing | React Router v6 (nested under `/app`) |
| Styling | Plain CSS with CSS variables (no Tailwind) |
| HTTP client | Axios (with interceptor for token refresh) |
| Backend | Node.js + Express |
| Database | PostgreSQL 14+ via Sequelize ORM |
| Auth | JWT (access 7d + refresh 30d) + bcryptjs + speakeasy (2FA) + Passport (OAuth) |
| Real-time | Socket.io (notifications, chat, whiteboard) |
| Email | Nodemailer (SMTP) |
| File uploads | Multer → `/uploads/` directory |
| Migrations | Sequelize CLI |

---

## Current State Summary

| Metric | Value |
|--------|-------|
| Total planned features | 416 tasks (FEATURE_CHECKLIST.md) |
| Completed | ~285 tasks (~69%) |
| Remaining | ~131 tasks (~31%) |
| Backend coverage | ~85% complete |
| Frontend coverage | ~65% complete (views exist but many are shallow) |
| Test coverage | 0% |
| Deployment | Local dev only — not yet containerised or deployed |

---

## Suggested Build Order (Next Steps)

1. **Test suite** — Jest + Supertest for backend; Vitest + React Testing Library for frontend
2. **Rich text editor** for Documents (Tiptap) — closes biggest gap vs Notion/ClickUp
3. **AI features** — integrate Claude API for task prioritization and project health advisor
4. **Dark mode** — high user expectation, zero implementation
5. **Slack / GitHub integrations** — natural extension of the Automation Builder
6. **Advanced reporting** — burndown, velocity, PDF export
7. **Deployment** — Dockerfile + docker-compose + CI/CD pipeline

---

## Key Files to Know

| File | Purpose |
|------|---------|
| `backend/src/server.js` | Express + Socket.io bootstrap |
| `backend/src/routes/index.js` | All API routes registered here |
| `backend/src/utils/automationEngine.js` | Automation trigger/action execution engine |
| `backend/src/utils/permissions.js` | Workspace permission helpers |
| `backend/src/models/ProjectMembers.js` | Project-level roles (the 6 real role names) |
| `backend/src/models/WorkspaceMembers.js` | Workspace-level roles |
| `frontend/src/App.tsx` | All React routes |
| `frontend/src/store/index.ts` | Redux store (all slices registered here) |
| `frontend/src/components/layout/RoleBasedSidebar.tsx` | Role-gated navigation |
| `frontend/src/components/layout/Layout.tsx` | App shell + Socket.io connection |

---

## Reference Documents

| Document | Purpose |
|----------|---------|
| `FEATURE_CHECKLIST.md` | Every feature broken to task level, with done/todo status |
| `COMPETITOR_ANALYSIS.md` | How PulseHub compares to Jira, ClickUp, Asana, Linear etc. |
| `PULSEHUB_COMPLETE_REFERENCE.md` | Deep technical reference (models, API, architecture) |
| `RBAC_ROLE_TESTING_GUIDE.md` | How to test each role's access |
| `DEPLOYMENT_GUIDE.md` | Steps to deploy (when ready) |
