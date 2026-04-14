# Projva — Setup Guide

Complete installation guide for developers setting up Projva for the first time.

---

## Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| Node.js | 18+ | Check with `node -v` |
| npm | 9+ | Comes with Node.js |
| PostgreSQL | 14+ | Must be running on port 5432 (or configured in `.env`) |
| psql (PostgreSQL client) | Match PG version | Required for `setup.js` to create the database |

### Installing PostgreSQL

**macOS (Homebrew):**
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
```

**Windows:**
Download from https://www.postgresql.org/download/windows/

**Docker (alternative to local install):**
```bash
docker run -d \
  --name projva-db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_DB=projva \
  -p 5432:5432 \
  postgres:15-alpine
```

---

## Step-by-Step Setup

### Option A: Interactive Setup (Recommended)

```bash
git clone <your-repo-url>
cd projva
npm run setup
```

Follow the prompts. The wizard will:

1. Check Node.js and PostgreSQL
2. Ask for database credentials
3. Create `backend/.env` and `frontend/.env`
4. Install backend and frontend dependencies
5. Create the `projva` database
6. Run all 23 migrations
7. Seed demo accounts
8. Verify the installation

### Option B: Manual Setup

```bash
# 1. Clone
git clone <your-repo-url>
cd projva

# 2. Backend dependencies
cd backend
npm install

# 3. Configure environment
cp .env.example .env
# Edit .env and fill in DB_PASSWORD and JWT_SECRET

# 4. Create database
createdb -U postgres projva
# (or: psql -U postgres -c "CREATE DATABASE projva;")

# 5. Run migrations
npx sequelize-cli db:migrate --env development

# 6. Seed demo accounts
node scripts/seed-demo.js

# 7. Frontend dependencies
cd ../frontend
npm install
cp .env.example .env

# 8. Start
cd ..
npm run dev
```

---

## Configuration

### Backend (`backend/.env`)

```env
# Required
DB_HOST=localhost
DB_PORT=5432
DB_NAME=projva
DB_USERNAME=postgres
DB_PASSWORD=your_postgres_password

JWT_SECRET=generate_a_random_64_char_string
JWT_REFRESH_SECRET=another_random_64_char_string
SESSION_SECRET=another_random_string

# Optional (for email features)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_gmail_app_password
SMTP_FROM="Projva <noreply@projva.io>"

# Optional (for Google login)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
```

**Generating secrets:**
```bash
# macOS / Linux
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"

# Windows PowerShell
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### Frontend (`frontend/.env`)

```env
REACT_APP_API_URL=http://localhost:5000/api/v1
REACT_APP_SOCKET_URL=http://localhost:5000
```

---

## Available Commands

| Command | Description |
|---------|-------------|
| `npm run setup` | Full setup wizard |
| `npm run dev` | Start both backend + frontend |
| `npm run start:backend` | Backend only (port 5000) |
| `npm run build:frontend` | Production build (outputs to `frontend/build/`) |
| `npm run db:migrate` | Run pending migrations |
| `npm run db:migrate:undo` | Rollback last migration |
| `npm run db:seed` | Re-seed demo accounts |
| `npm run db:reset` | Full reset: rollback migrations + re-setup |

---

## Demo Accounts

After seeding, these accounts are available:

| Role | Email | Password |
|------|-------|----------|
| Super Admin | superadmin@projva.com | SuperAdmin@2025 |
| Admin | admin@projva.com | Admin@2025 |
| Owner | owner@projva.com | Owner@2025 |
| Project Manager | pm@projva.com | PM@2025 |
| Member | member@projva.com | Member@2025 |
| Commenter | commenter@projva.com | Commenter@2025 |
| Guest | guest@projva.com | Guest@2025 |
| Viewer | viewer@projva.com | Viewer@2025 |

---

## Database Schema

Projva uses **23 Sequelize migrations** to build its schema:

| # | Migration | What it does |
|---|-----------|-------------|
| 1 | Initial tables | users, workspaces, projects, lists, tasks, subtasks, comments, attachments, time_logs, etc. |
| 2 | Subtask assignee | Adds `assigneeId` to subtasks |
| 3 | Invites table | `invites` table |
| 4 | Super admin role | Adds `super_admin` to user role ENUM |
| 5 | Activity logs | `activity_logs` table |
| 6 | Resources & budgets | `resources`, `budgets`, `budget_expenses` tables |
| 7 | Subtask dates | Adds `startDate`, `estimatedTime`, `actualTime` to subtasks |
| 8 | Workflow templates | `isTemplate` field on workflows |
| 9 | Saved views | `saved_views` table |
| 10 | RBAC V2 | Project-level roles, new ENUMs, `project_members`, `guest_access` |
| 11 | Notifications | `notifications` table |
| 12 | Extended user fields | `isEmailVerified`, verification tokens, profile fields |
| 13 | Task labels | `labels` JSONB column on tasks |
| 14 | Automations | `automations`, `automation_logs` tables |
| 15 | Chat fields | `reactions` JSONB, `editedAt` on chat_messages |
| 16 | Billable time logs | `isBillable`, `billingRate` on time_logs |
| 17 | Chat parent message | `parentId` for thread replies in chat |
| 18 | Recurring tasks | `recurrence` JSONB on tasks |
| 19 | Chat attachments | `attachmentUrl`, `attachmentType` on chat_messages |
| 20 | Document sharing | `Document`, `document_sharing` table |

All migrations are **additive** — they only add columns/tables and new ENUM values. No data is destroyed on upgrade.

---

## Troubleshooting

### "psql: command not found"

PostgreSQL client is not in your PATH.

**Fix:** Install PostgreSQL and ensure the `bin` directory is in PATH, or use Docker:
```bash
docker run -d --name projva-db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=projva \
  -p 5432:5432 \
  postgres:15-alpine
```

---

### "Database does not exist"

The `projva` database hasn't been created yet.

**Fix:**
```bash
# With psql
psql -U postgres -c "CREATE DATABASE projva;"

# Or let setup.js handle it
npm run setup
```

---

### "Cannot connect to database" / "Connection refused"

PostgreSQL is not running or credentials are wrong.

**Fix:**
```bash
# Check PostgreSQL is running
pg_isready -h localhost -p 5432

# macOS
brew services start postgresql@15

# Ubuntu
sudo systemctl start postgresql

# Windows
# Start the PostgreSQL service via Services app
```

Then verify credentials in `backend/.env`:
```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=projva
DB_USERNAME=postgres
DB_PASSWORD=your_password
```

---

### Migration fails with "relation does not exist"

The database was created but migrations haven't run, or migrations ran out of order.

**Fix:**
```bash
npm run db:migrate
```

If it fails on a specific migration, check that the previous migration created the referenced table.

---

### "Permission denied for table X"

The database user doesn't have access to the database.

**Fix:**
```sql
-- Connect as postgres superuser
psql -U postgres -d projva

-- Grant all permissions
GRANT ALL PRIVILEGES ON DATABASE projva TO your_username;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO your_username;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO your_username;
ALTER DATABASE projva OWNER TO your_username;
```

---

### Frontend shows "API Error" or blank page

Backend is not running, or frontend `.env` points to wrong URL.

**Fix:**
1. Start backend: `npm run start:backend`
2. Check `frontend/.env`:
   ```env
   REACT_APP_API_URL=http://localhost:5000/api/v1
   ```
3. Verify backend health: `curl http://localhost:5000/api/v1/health`

---

### Port 3000 or 5000 already in use

Another process is using the port.

```bash
# Find what's using port 3000 (frontend)
# Windows:
netstat -ano | findstr :3000
# macOS/Linux:
lsof -i :3000

# Find what's using port 5000 (backend)
lsof -i :5000

# Kill the process or change the port in .env
```

To use a different backend port, update:
```env
# backend/.env
PORT=3001
# frontend/.env
REACT_APP_API_URL=http://localhost:3001/api/v1
REACT_APP_SOCKET_URL=http://localhost:3001
```

---

### "Module not found" errors

`node_modules` is missing or corrupted.

**Fix:**
```bash
# Remove and reinstall
cd backend && rm -rf node_modules && npm install
cd ../frontend && rm -rf node_modules && npm install
```

---

### Database migration out of sync after pulling new code

New migrations were added by another developer.

**Fix:**
```bash
# Check pending migrations
cd backend
npx sequelize-cli db:migrate:status --env development

# Apply pending migrations
npm run db:migrate
```

---

## Switching to Docker (Optional)

If you prefer Docker over local PostgreSQL:

```bash
# Create the database container
docker run -d \
  --name projva-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_DB=projva \
  -p 5432:5432 \
  -v projva-data:/var/lib/postgresql/data \
  postgres:15-alpine

# Then run setup with non-interactive mode
npm run setup -- --non-interactive
```

When prompted for DB host, enter `localhost` (Docker exposes port 5432).

---

## Resetting Everything

To start from scratch:

```bash
# 1. Drop and recreate the database
psql -U postgres -c "DROP DATABASE IF EXISTS projva;"
psql -U postgres -c "CREATE DATABASE projva;"

# 2. Run migrations fresh
npm run db:migrate

# 3. Seed demo accounts
npm run db:seed

# Or use the one-liner:
npm run db:reset
```

---

## Getting Help

- **Setup issues** — Check this guide first
- **Feature questions** — See docs/FEATURE_CHECKLIST.md
- **Testing** — See docs/TESTING_STRATEGY.md
- **Deployment** — See docs/DEPLOYMENT_GUIDE.md
