# 🧪 RBAC Role-Based Testing Guide

## 📋 Overview

This guide provides step-by-step testing procedures for each role to verify proper access control across all features.

---

## 👥 Test Users Setup

Create one user for each role:

| Role | Email | Password | Notes |
|------|-------|----------|-------|
| Super Admin | superadmin@test.com | Test@123 | Full system access |
| Admin | admin@test.com | Test@123 | Organization admin |
| Owner | owner@test.com | Test@123 | Workspace owner |
| PM | pm@test.com | Test@123 | Project manager |
| Member | member@test.com | Test@123 | Team member |
| Viewer | viewer@test.com | Test@123 | Read-only access |

---

## 🔐 Super Admin Testing

### ✅ What Super Admin CAN Do:

1. **User Management** (`/app/users`)
   - ✅ View all users in system
   - ✅ Create new users
   - ✅ Edit any user's role (including making admins)
   - ✅ Delete users
   - ✅ Deactivate users

2. **Workspaces**
   - ✅ View ALL workspaces (even not assigned)
   - ✅ Create workspaces
   - ✅ Assign workspace owners
   - ✅ Edit any workspace
   - ✅ Delete any workspace
   - ✅ Manage members in any workspace

3. **Projects**
   - ✅ View ALL projects
   - ✅ Create projects in any workspace
   - ✅ Edit any project
   - ✅ Delete any project

4. **Tasks**
   - ✅ View ALL tasks
   - ✅ Create tasks in any project
   - ✅ Edit any task
   - ✅ Delete any task
   - ✅ Assign tasks to anyone

5. **Time Tracking**
   - ✅ View ALL time logs
   - ✅ View statistics for all users/projects
   - ✅ Create/edit/delete any time log

6. **Resources**
   - ✅ View all resources
   - ✅ Add/edit resources
   - ✅ View all workload data

7. **Budgets**
   - ✅ View all budgets
   - ✅ Create budgets for any project
   - ✅ Add expenses to any budget

8. **Calendar**
   - ✅ View all tasks on calendar
   - ✅ Filter by any project/user

9. **Kanban**
   - ✅ View any project's Kanban board
   - ✅ Move any task

### ❌ What Super Admin CANNOT Do:
- Nothing! Super Admin has full access.

---

## 👔 Admin Testing

### ✅ What Admin CAN Do:

1. **Workspaces**
   - ✅ View all workspaces
   - ✅ Create workspaces
   - ✅ Edit workspaces
   - ✅ Delete workspaces
   - ✅ Manage workspace members
   - ❌ Cannot assign workspace owners (Super Admin only)

2. **Projects**
   - ✅ View all projects
   - ✅ Create projects
   - ✅ Edit projects
   - ✅ Delete projects

3. **Tasks**
   - ✅ View all tasks
   - ✅ Create/edit/delete tasks
   - ✅ Assign tasks

4. **Time Tracking**
   - ✅ View all time logs
   - ✅ View all statistics
   - ✅ Create/edit/delete time logs

5. **Resources**
   - ✅ View all resources
   - ✅ Add/edit resources

6. **Budgets**
   - ✅ View all budgets
   - ✅ Create budgets
   - ✅ Add expenses

7. **Calendar & Kanban**
   - ✅ Full access

### ❌ What Admin CANNOT Do:
- ❌ Access User Management page
- ❌ Edit user roles
- ❌ Assign workspace owners
- ❌ Manage super admins

---

## 👑 Owner Testing

### ✅ What Owner CAN Do:

1. **Workspaces**
   - ✅ View OWN workspaces only
   - ✅ Create workspaces
   - ✅ Edit OWN workspaces
   - ✅ Delete OWN workspaces
   - ✅ Manage members in OWN workspaces
   - ❌ Cannot see other owners' workspaces

2. **Projects**
   - ✅ View projects in OWN workspaces
   - ✅ Create projects in OWN workspaces
   - ✅ Edit projects in OWN workspaces
   - ✅ Delete projects in OWN workspaces

3. **Tasks**
   - ✅ View tasks in OWN workspaces
   - ✅ Create/edit/delete tasks
   - ✅ Assign tasks

4. **Time Tracking**
   - ✅ View time logs for OWN workspace
   - ✅ Create/edit/delete time logs

5. **Resources**
   - ✅ View resources in OWN workspace
   - ❌ Cannot add/edit resources (Admin only)

6. **Budgets**
   - ✅ View budgets in OWN workspace
   - ✅ Create budgets
   - ✅ Add expenses

### ❌ What Owner CANNOT Do:
- ❌ Access User Management
- ❌ View other owners' workspaces
- ❌ Add/edit resources
- ❌ Assign workspace owners

---

## 📊 PM (Project Manager) Testing

### ✅ What PM CAN Do:

1. **Workspaces**
   - ✅ View assigned workspaces (read-only)
   - ❌ Cannot create workspaces
   - ❌ Cannot edit workspaces
   - ❌ Cannot manage members

2. **Projects**
   - ✅ View assigned projects
   - ✅ Create projects (in assigned workspace)
   - ✅ Edit assigned projects
   - ❌ Cannot delete projects

3. **Tasks**
   - ✅ View tasks in assigned projects
   - ✅ Create tasks
   - ✅ Edit tasks
   - ✅ Assign tasks
   - ❌ Cannot delete tasks

4. **Subtasks**
   - ✅ Create/edit/delete subtasks

5. **Time Tracking**
   - ✅ View time logs for assigned projects
   - ✅ Create time logs
   - ✅ View project statistics

6. **Resources**
   - ✅ View workload for assigned workspace
   - ❌ Cannot add/edit resources

7. **Budgets**
   - ✅ View budgets for assigned projects
   - ✅ Create budgets
   - ✅ Add expenses

8. **Calendar & Kanban**
   - ✅ View assigned projects
   - ✅ Move tasks in Kanban

### ❌ What PM CANNOT Do:
- ❌ Create workspaces
- ❌ Delete projects
- ❌ Delete tasks
- ❌ Manage workspace members
- ❌ Add/edit resources
- ❌ Access User Management

---

## 👤 Member Testing

### ✅ What Member CAN Do:

1. **Workspaces**
   - ✅ View assigned workspaces (read-only)

2. **Projects**
   - ✅ View assigned projects (read-only)

3. **Tasks**
   - ✅ View assigned tasks
   - ✅ Edit OWN tasks only
   - ✅ Change status of OWN tasks
   - ❌ Cannot create tasks
   - ❌ Cannot delete tasks
   - ❌ Cannot assign tasks

4. **Subtasks**
   - ✅ View subtasks
   - ✅ Edit OWN subtasks only
   - ❌ Cannot create subtasks
   - ❌ Cannot delete subtasks

5. **Comments**
   - ✅ View comments
   - ✅ Add comments
   - ✅ Edit/delete OWN comments

6. **Attachments**
   - ✅ View attachments
   - ✅ Upload to OWN tasks
   - ✅ Delete OWN attachments

7. **Time Tracking**
   - ✅ View OWN time logs only
   - ✅ Create OWN time logs
   - ✅ Edit/delete OWN time logs
   - ✅ View OWN statistics

8. **Calendar**
   - ✅ View assigned tasks on calendar
   - ❌ Cannot create events

9. **Kanban**
   - ✅ View Kanban board
   - ✅ Move OWN tasks only

### ❌ What Member CANNOT Do:
- ❌ Create workspaces/projects/tasks
- ❌ Delete anything
- ❌ Edit other users' tasks
- ❌ Access Resources page
- ❌ Access Budgets page
- ❌ View other users' time logs
- ❌ Assign tasks

---

## 👁️ Viewer Testing

### ✅ What Viewer CAN Do:

1. **Workspaces**
   - ✅ View assigned workspaces (read-only)

2. **Projects**
   - ✅ View assigned projects (read-only)

3. **Tasks**
   - ✅ View assigned tasks (read-only)
   - ✅ View task details
   - ❌ Cannot edit/delete/create

4. **Comments & Attachments**
   - ✅ View only
   - ❌ Cannot add/edit/delete

5. **Calendar**
   - ✅ View assigned tasks on calendar
   - ❌ Cannot create/edit events

6. **Kanban**
   - ✅ View Kanban board (read-only)
   - ❌ Cannot move tasks (drag disabled)

7. **Activity Timeline**
   - ✅ View activity history

### ❌ What Viewer CANNOT Do:
- ❌ Create anything
- ❌ Edit anything
- ❌ Delete anything
- ❌ Add comments
- ❌ Upload attachments
- ❌ Track time
- ❌ Access Time Tracking page
- ❌ Access Resources page
- ❌ Access Budgets page
- ❌ Move tasks in Kanban

---

## 🧪 Testing Scenarios

### Scenario 1: Workspace Creation
1. **Super Admin**: ✅ Can create and assign owner
2. **Admin**: ✅ Can create (but cannot assign owner)
3. **Owner**: ✅ Can create
4. **PM/Member/Viewer**: ❌ Button hidden

### Scenario 2: Task Creation
1. **Super Admin/Admin/Owner/PM**: ✅ Can create
2. **Member/Viewer**: ❌ Button hidden

### Scenario 3: Task Editing
1. **Super Admin/Admin/Owner/PM**: ✅ Can edit any task
2. **Member**: ✅ Can edit own tasks only
3. **Viewer**: ❌ Cannot edit

### Scenario 4: Time Tracking
1. **Super Admin/Admin/Owner/PM**: ✅ Can view all logs
2. **Member**: ✅ Can view own logs only
3. **Viewer**: ❌ Cannot access page

### Scenario 5: Resource Management
1. **Super Admin/Admin**: ✅ Can add/edit resources
2. **Owner/PM**: ✅ Can view only
3. **Member/Viewer**: ❌ Cannot access page

### Scenario 6: Budget Management
1. **Super Admin/Admin/Owner/PM**: ✅ Full access
2. **Member/Viewer**: ❌ Cannot access page

### Scenario 7: Kanban Drag-Drop
1. **Super Admin/Admin/Owner/PM**: ✅ Can move any task
2. **Member**: ✅ Can move own tasks
3. **Viewer**: ❌ Drag disabled

---

## ✅ Quick Test Checklist

### For Each Role, Test:

- [ ] Login and verify sidebar shows correct menu items
- [ ] Try to access restricted pages (should redirect or show 403)
- [ ] Verify create buttons are hidden for restricted roles
- [ ] Verify edit/delete buttons are hidden appropriately
- [ ] Test ownership checks (member can only edit own items)
- [ ] Verify backend API returns 403 for unauthorized actions
- [ ] Test workspace/project context filtering
- [ ] Verify calendar shows only accessible tasks
- [ ] Verify Kanban drag-drop respects permissions
- [ ] Test time tracking access and ownership

---

## 📍 Where to Test Each Feature

### Dashboard
- **URL**: `/app/dashboard`
- **Test**: Verify data filtering by role

### Workspaces
- **URL**: `/app/workspaces`
- **Test**: Create button visibility, member management

### Projects
- **URL**: `/app/projects`
- **Test**: Create button, project list filtering

### Tasks
- **URL**: `/app/tasks`
- **Test**: Create button, task list filtering

### Task Detail
- **URL**: `/app/tasks/:id`
- **Test**: Edit/Delete buttons, subtask actions, comments, attachments

### Calendar
- **URL**: `/app/calendar`
- **Test**: Task visibility, edit permissions

### Time Tracking
- **URL**: `/app/time-tracking`
- **Test**: Page access, log creation, statistics visibility

### Resources
- **URL**: `/app/resources`
- **Test**: Page access, add resource button

### Budgets
- **URL**: `/app/budgets`
- **Test**: Page access, create budget, add expense

### Kanban
- **URL**: `/app/projects/:id/kanban`
- **Test**: Drag-drop permissions, task visibility

### User Management
- **URL**: `/app/users`
- **Test**: Super Admin only access

---

## 🐛 Common Issues to Check

1. **UI Buttons Visible but API Fails**
   - Fix: Add PermissionGuard to buttons

2. **Backend Allows Unauthorized Access**
   - Fix: Add permission middleware to routes

3. **Members Can Edit Others' Tasks**
   - Fix: Add ownership check in PermissionGuard

4. **Viewers Can Drag Tasks in Kanban**
   - Fix: Disable drag for viewers

5. **Wrong Data Shown (e.g., all workspaces for member)**
   - Fix: Add filtering in backend controllers

---

## 📊 Test Results Template

```
Role: [Role Name]
Date: [Date]
Tester: [Name]

✅ Passed Tests:
- [List passed tests]

❌ Failed Tests:
- [List failed tests with details]

⚠️ Issues Found:
- [List issues]

Overall Status: [Pass/Fail/Partial]
```

---

**Last Updated**: [Current Date]
**Status**: Ready for Testing

