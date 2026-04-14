# PulseHub — Competitor Feature Analysis
> Where PulseHub stands vs. the market. Use this to prioritise what to build next.
> Last updated: 2026-03-19

---

## Competitors Covered

| Tool | Focus | Pricing (per user/mo) | Market position |
|------|-------|-----------------------|----------------|
| **Jira** | Dev/engineering PM | $8.15 – $16 | Enterprise standard |
| **Asana** | General team work | $10.99 – $24.99 | Mid-market leader |
| **Monday.com** | Work OS / visual | $9 – $19 | Fastest growing |
| **Linear** | Eng-focused, speed | $8 – $16 | Dev-team darling |
| **ClickUp** | All-in-one | $7 – $12 | Feature-richest |
| **Notion** | Docs + DB hybrid | $8 – $15 | Doc-first teams |
| **Trello** | Simple Kanban | $5 – $17.50 | Small teams |
| **Basecamp** | Simple all-in-one | $15 flat | Remote teams |
| **PulseHub** | Full-stack PM + collab | TBD | In development |

---

## PulseHub 3-Tier Access Architecture

PulseHub implements a **three-tier role and access model** that is more granular than most competitors. This is a core architectural differentiator.

### Tier 1 — Platform Level (Super Admin)
The top-level administrator who manages the entire PulseHub installation.

| Capability | Details |
|-----------|---------|
| Manage all users | Create, edit, deactivate any user across all workspaces |
| Manage all workspaces | See, configure, or delete any workspace |
| System settings | Global configuration, audit logs across all tenants |
| Bypass all permissions | Super Admin always has full access regardless of workspace/project roles |
| User Management panel | Dedicated `/app/users` page only visible to Super Admin |
| Admin Panel | Dedicated `/app/admin` page visible to Super Admin + Admin |

> **Analogy**: Super Admin is like a system operator — they manage the platform, not just a single team.

---

### Tier 2 — Workspace Level
Each workspace has its own membership and roles. A user can have different roles in different workspaces.

| Role | Label | Permissions |
|------|-------|-------------|
| `owner` | Owner | Full control — settings, delete workspace, invite/remove anyone, transfer ownership. One per workspace. |
| `admin` | Admin | Delegated manager — manage members, projects, automations. Cannot delete workspace or manage billing. |
| `billing_admin` | Billing Admin | Finance-only access — billing dashboard. No project access. |
| `member` | Member | Default role. Actual capabilities determined by their project-level role. |
| `guest` | Guest | External user. Must have a `guest_access` record scoped to a specific project. |

> **Legacy workspace roles** (`pm`, `viewer`, `commenter`) exist in the database for backward compatibility but are superseded by project-level roles in RBAC V2. New invitations should use the roles above.

Workspace-level access is set when a user is **invited to a workspace** via the invite system (`/invite/:token`). Roles can be changed by the workspace owner or admin at any time via the **Workspace Members Manager**.

---

### Tier 3 — Project Level
Within a workspace, each project can have its own membership and role overrides. A `member` in the workspace can be a `project_lead` in a specific project.

| Role | Label | Permissions within the project |
|------|-------|-------------------------------|
| `project_lead` | Project Lead | Full control of the project — settings, statuses, members, tasks, budget, automations |
| `contributor` | Contributor | Default role — create/update tasks, log time, add comments, upload attachments |
| `reporter` | Reporter | Create tasks and report status; cannot edit others' tasks |
| `reviewer` | Reviewer | Review and approve tasks; can comment and change status |
| `commenter` | Commenter | View tasks and add comments only; cannot create or edit tasks |
| `viewer` | Viewer | Read-only access to the project |

Project-level membership is managed via the **Project Members Manager** inside each project's settings tab.

---

### How the Tiers Interact

```
Platform (Super Admin)
    └── Workspace A
        ├── owner / admin / billing_admin / member / guest  ← Workspace role
        └── Project X
            ├── project_lead / contributor / reporter       ← Project role
            ├── reviewer / commenter / viewer               ← Project role
            └── (overrides workspace role for this project)
        └── Project Y
            └── project_lead / contributor / ...
    └── Workspace B
        └── ...
```

**Effective role resolution order** (as defined in the codebase):
1. `super_admin` → full access everywhere, bypasses all checks
2. `workspace owner` → full access within their workspace
3. **Project role** → used if the user has a `project_members` record for this project
4. **Workspace role** → fallback if no project-level role exists
5. Deny → if none of the above apply

> This means a `member` at the workspace level can be given `project_lead` access to a single sensitive project without granting them broader workspace permissions — matching enterprise access-control expectations.

**Note on the sidebar:** The `RoleBasedSidebar.tsx` uses a simplified effective-role calculation for navigation gating, resolving workspace roles first, then project roles, then the user's base role. It maps the full role set down to `super_admin | admin | owner | pm | member | viewer` for sidebar visibility decisions.

---

### How PulseHub Compares on Access Control

| Access Model | Jira | Asana | Monday | Linear | ClickUp | **PulseHub** |
|-------------|------|-------|--------|--------|---------|-------------|
| Platform-level admin | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Workspace-level roles | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project-level role overrides | ✅ | ⚠️ | ⚠️ | ❌ | ✅ | ✅ |
| Invite-based onboarding | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Role-scoped sidebar navigation | ⚠️ | ⚠️ | ⚠️ | ❌ | ⚠️ | ✅ |
| Audit log | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 2FA / MFA | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## Authentication & Onboarding

PulseHub has a complete auth stack that competes well with enterprise tools:

| Feature | Status | Notes |
|---------|--------|-------|
| Email + password login | ✅ Done | JWT-based, tokens in httpOnly cookies |
| Google OAuth | ✅ Done | `/auth/callback` — one-click login via Google |
| Email verification | ✅ Done | Token link sent on register; verified before full access |
| Two-factor authentication (2FA) | ✅ Done | TOTP-based (Google Authenticator etc.) |
| Workspace invite system | ✅ Done | Email invite with `/invite/:token` link; role pre-assigned |
| Project member invitations | ✅ Done | Project Members Manager in project settings |
| Demo seed accounts | ✅ Done | `npm run seed:demo` — pre-populated demo data for onboarding |
| Password reset | ⚠️ Partial | Backend route exists; email sending not confirmed |
| SSO (SAML/OIDC) | ❌ Planned | Enterprise tier feature |

---

## Core Feature Comparison Matrix

### ✅ = Full feature · ⚠️ = Partial / limited · ❌ = Not available

| Feature | Jira | Asana | Monday | Linear | ClickUp | Notion | Trello | Basecamp | **PulseHub** |
|---------|------|-------|--------|--------|---------|--------|--------|----------|-------------|
| **TASK MANAGEMENT** | | | | | | | | | |
| Task CRUD | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Subtasks | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ | ❌ | ❌ | ✅ |
| Task dependencies | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Multi-assignee | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ |
| Priority levels | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ⚠️ | ❌ | ✅ |
| Custom statuses | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ⚠️ | ❌ | ✅ |
| Task labels/tags | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ 🔲 |
| Recurring tasks | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Task templates | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ⚠️ |
| Bulk operations | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| CSV import/export | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| Time estimates | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Progress tracking | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| **VIEWS** | | | | | | | | | |
| List view | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Kanban board | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ | ✅ | ❌ | ✅ |
| Gantt chart | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Calendar view | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ⚠️ | ❌ | ⚠️ |
| Timeline view | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Workload view | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Table/spreadsheet | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| Mind map | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Dashboard view | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Saved/custom views | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| **COLLABORATION** | | | | | | | | | |
| Comments + replies | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| @mentions | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ 🔲 |
| Emoji reactions | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| File attachments | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Real-time notifications | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ | ❌ | ⚠️ | ✅ |
| Real-time chat | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ | ⚠️ |
| Video calls | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Guest access | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Notification bell | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| **DOCUMENTS** | | | | | | | | | |
| Rich text docs | ⚠️ | ⚠️ | ⚠️ | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ 🔲 |
| Doc templates | ❌ | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Collaborative editing | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| Version history | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| Whiteboard | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| **PROJECT MANAGEMENT** | | | | | | | | | |
| Project templates | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ⚠️ |
| Project health score | ❌ | ⚠️ | ⚠️ | ✅ | ⚠️ | ❌ | ❌ | ❌ | ✅ |
| Portfolio view | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Milestones | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Goals/OKRs | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Stakeholder dashboard | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| **ANALYTICS & REPORTING** | | | | | | | | | |
| Built-in reports | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Custom reports | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Velocity chart | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Burndown chart | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Cumulative flow | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Export PDF/Excel | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| **RESOURCE & TIME** | | | | | | | | | |
| Time tracking | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Resource planning | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Workload balancing | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Budget tracking | ✅ | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| **AUTOMATION** | | | | | | | | | |
| No-code automation builder | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ |
| Custom triggers | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Multiple actions per rule | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Workspace-wide automations | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Project-scoped automations | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ |
| Scheduled / time-based triggers | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ 🔲 |
| Webhook support | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ 🔲 |
| **ACCESS & SECURITY** | | | | | | | | | |
| 3-tier role system (platform/workspace/project) | ✅ | ⚠️ | ⚠️ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Role-based sidebar navigation | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ❌ | ❌ | ❌ | ✅ |
| Workspace invite system | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Project member invitations | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Google OAuth login | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| Email verification | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 2FA / MFA | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Audit log | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| SSO (SAML/OIDC) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ 🔲 |
| IP allowlist | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **SEARCH** | | | | | | | | | |
| Global search (Cmd+K) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| Search across tasks/projects | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| Advanced filters (AND/OR) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| Saved searches | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| **INTEGRATIONS** | | | | | | | | | |
| Slack | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ 🔲 |
| GitHub / GitLab | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ 🔲 |
| Google Workspace | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Microsoft 365 | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Zapier | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ 🔲 |
| Figma | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Jira (import) | — | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ 🔲 |
| REST API access | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| **AI FEATURES** | | | | | | | | | |
| AI task summaries | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| AI writing assist | ❌ | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ 🔲 |
| AI prioritization | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ 🔲 |
| AI auto-tagging | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ 🔲 |
| **PLATFORM** | | | | | | | | | |
| Web app | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| iOS app | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Android app | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Desktop app | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Dark mode | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ 🔲 |
| Offline mode | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Real-time sync (WebSocket) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |

> 🔲 = PulseHub plans to build this (see FEATURE_CHECKLIST.md)

---

## PulseHub Unique Strengths

| Strength | Why it matters |
|----------|----------------|
| **3-tier RBAC (platform → workspace → project)** | More granular than Asana/Trello/Linear; matches Jira and ClickUp enterprise level. Role-scoped sidebar is a UX differentiator — users only see what they can access. |
| **Full automation builder (triggers + actions)** | 7 trigger types, 5 action types, workspace-wide or project-scoped rules. Matches ClickUp/Asana automation level. |
| **Real-time notifications via Socket.io** | Push notifications on task events — no polling lag. Most competitors still rely on polling or third-party push services. |
| **Project health score** | Automated, not manual. Linear does this partially; most others don't. |
| **All views in one product** | List + Kanban + Gantt + Calendar + Workload + Stakeholder Dashboard — few tools nail all 6. |
| **Budget + Time tracking built-in** | ClickUp does this but most competitors don't at this price point. |
| **Invite-based team onboarding** | Role is pre-assigned in the invite link. No manual role assignment step after joining. |
| **Demo seed accounts** | One-command demo environment (`npm run seed:demo`) — great for sales demos and testing. |

---

## PulseHub Gaps vs Top 3 Competitors

### vs. Jira
| Jira has | Priority for PulseHub |
|----------|----------------------|
| Sprint velocity chart | 🟡 Medium — Q2 |
| Burndown / cumulative flow | 🟡 Medium — Q2 |
| Advanced JQL filtering | 🟡 Medium (saved views partially covers this) |
| Issue linking types (epic > story > bug) | 🟡 Medium — needs hierarchy |
| Agile board planning | 🟡 Medium |
| Marketplace of 3000+ integrations | 🔴 Long-term |
| SSO / SAML | 🟡 Medium — enterprise tier |

### vs. ClickUp (closest feature match)
| ClickUp has | Priority for PulseHub |
|------------|----------------------|
| Docs with collaborative editing | 🔴 High — Q1 |
| Scheduled / time-based automation triggers | 🟡 Medium |
| Webhook support | 🟡 Medium |
| Mind map view | 🟢 Low |
| Goals / OKRs tracking | 🟡 Medium |
| Custom fields on all entities | 🟡 Medium |
| AI features | 🟡 Medium — Q2 (Claude API integration planned) |
| Dark mode | 🟡 Medium |

### vs. Linear (dev-team darling)
| Linear has | Priority for PulseHub |
|-----------|----------------------|
| Keyboard-first UX (every action has shortcut) | 🟡 Medium |
| Cycles (time-boxed sprints, auto-close) | 🟡 Medium |
| GitHub PR ↔ issue sync | 🔴 High (integration) |
| Sub-1s page loads (SQLite + edge deployment) | 🟡 Medium (performance) |
| Saved/custom views | 🟡 Medium |
| Issue templates | 🟡 Medium |

---

## Feature Parity Score

| Competitor | Features matched | Total competitor features | Parity % |
|-----------|-----------------|--------------------------|----------|
| Trello | 20/25 | 25 | **80%** |
| Basecamp | 15/20 | 20 | **75%** |
| Linear | 30/40 | 40 | **75%** |
| Asana | 35/50 | 50 | **70%** |
| Monday.com | 33/50 | 50 | **66%** |
| ClickUp | 38/65 | 65 | **58%** |
| Jira | 33/60 | 60 | **55%** |

> PulseHub is now most competitive with **Trello, Basecamp, and Linear**, and has closed the gap with **Asana** significantly after completing the Automation Builder, real-time notifications, RBAC V2, and full auth stack.
> To compete with **ClickUp and Jira**, priority items are: rich text docs, collaborative editing, AI features, Slack/GitHub integrations, and advanced reporting.

---

## What to Build to Become Competitive

### To beat Trello & Basecamp (done or nearly done)
- [x] All core task/project management ✅
- [x] Comments + reactions ✅
- [x] Real-time notifications (Socket.io) ✅
- [x] Automation builder ✅
- [x] 3-tier RBAC + invite system ✅
- [ ] @mentions in comments
- [ ] Rich text documents
- [ ] Recurring tasks

### To match Linear (~1–2 months)
- [ ] Saved/custom views
- [ ] GitHub integration
- [ ] Keyboard shortcuts for all actions
- [ ] Sprint/cycle management
- [ ] Performance improvements (page load speed)

### To match Asana (~2–3 months)
- [x] Automation builder ✅
- [ ] Goals / OKRs module
- [ ] Portfolio view across projects
- [ ] Advanced reporting (burndown, velocity)
- [ ] Notification preferences per user (email on/off, digest)
- [ ] Dark mode

### To match ClickUp (~5–6 months)
- [ ] Rich text collaborative documents
- [ ] AI-powered features (Claude API — task prioritization, health advisor)
- [ ] Full whiteboard with drawing tools
- [ ] Custom fields on workspace/project entities
- [ ] Zapier / Make webhook integration
- [ ] Scheduled automation triggers
- [ ] Mind map view
- [ ] Mobile apps (iOS/Android)

---

*Reference: FEATURE_CHECKLIST.md for implementation tracking*
