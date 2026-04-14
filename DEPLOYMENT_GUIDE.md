# TrackPro Deployment Guide - Realtime & RBAC Features

## Pre-Deployment Checklist

### Backend Preparation

```bash
# 1. Verify socket.io is installed
cd backend
npm ls socket.io
# Should show: socket.io@4.7.5 (or newer)

# 2. Check all files created/modified
git status
# Should show: socket.js, permission.controller.js, permission.routes.js, etc.

# 3. Verify syntax
node -c src/socket.js
node -c src/app.js
# No output means no errors

# 4. Check database configuration
cat src/config/database.js
# Verify connection string points to correct database
```

### Frontend Preparation

```bash
# 1. Verify socket.io-client is installed
cd frontend
npm ls socket.io-client
# Should show: socket.io-client@4.7.5+ (e.g., 4.8.3)

# 2. TypeScript compilation check
npm run type-check
# Should complete with no errors

# 3. Build verification (optional)
npm run build
# Should complete successfully
```

---

## Deployment Steps

### Step 1: Database Migration

**CRITICAL**: Apply enum migration before starting backend

```bash
cd backend

# Option A: Using migration system (if sequelize-cli migrations have been set up)
npx sequelize-cli db:migrate

# Option B: Direct application (if migrations are pending)
# The migration file 20260107000010-add-guest-commenter-role.js 
# adds 'guest' and 'commenter' to role enums

# Verify migration was applied
psql -U postgres -d trackpro_db -c \
  "SELECT enum_range(NULL::enum_users_role);"
# Should include: guest, commenter in output
```

### Step 2: Install Backend Dependencies

```bash
cd backend
npm install  # Installs socket.io@4.7.5

# Verify installation
npm ls
# Should show socket.io in dependencies
```

### Step 3: Start Backend Server

```bash
cd backend

# Development
npm start
# OR
npm run dev

# Production (if applicable)
NODE_ENV=production npm start
```

**Verify Backend Started**:
```bash
# Check logs for:
# - "Server running on port 5000"
# - "Connected to database"
# - No errors about socket.io

# Test health endpoint
curl http://localhost:5000/api/health
# Should return: { success: true, message: "API is healthy" }
```

### Step 4: Install Frontend Dependencies

```bash
cd frontend
npm install  # Installs socket.io-client@4.7.5+

# Verify installation
npm ls socket.io-client
```

### Step 5: Start Frontend Server

```bash
cd frontend

# Development
npm start
# Frontend will be available at http://localhost:3000

# Production build
npm run build
# Outputs to frontend/build/ directory
```

**Verify Frontend Started**:
```bash
# Check browser console (F12) for:
# - No TypeScript/compilation errors
# - Socket.io connection established
# - "io@" version showing in Network tab WebSocket
```

### Step 6: Environment Configuration

**Backend** (.env file):
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=trackpro_db
DB_USERNAME=postgres
DB_PASSWORD=your_password

NODE_ENV=development
PORT=5000

# Socket.io specific (optional, defaults provided)
SOCKET_ORIGINS=http://localhost:3000,http://localhost:5173
```

**Frontend** (.env file):
```
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_SOCKET_URL=http://localhost:5000
```

---

## Post-Deployment Verification

### Test Socket.io Connection

1. **Open Browser DevTools** (F12)
2. **Go to Network tab** → Filter by "WS" (WebSocket)
3. **Look for WebSocket connection** to `localhost:5000/socket.io/`
4. **Should see**:
   - Initial HTTP upgrade request
   - WebSocket connection established
   - Handshake with auth payload

### Test Chat Realtime

1. **Open chat in two browser tabs** (same workspace/project)
2. **In Tab 1**: Type message "Hello @channel"
3. **In Tab 2**: Message should appear immediately
4. **Check**:
   - Message has sender name displayed
   - @channel mention has blue badge
   - No duplicate messages

### Test Whiteboard Collaboration

1. **Open whiteboard in two browser windows**
2. **In Window 1**: Add a sticky note
3. **In Window 2**: Sticky should appear instantly
4. **In Window 1**: Move the sticky
5. **In Window 2**: Position updates in real-time

### Test Typing Indicators

1. **Open chat in two tabs**
2. **In Tab 1**: Start typing in chat input
3. **In Tab 2**: Should see "1 typing..." message
4. **Stop typing in Tab 1**: Indicator disappears after ~3 seconds

### Test Permission Enforcement

1. **Admin**: Create workspace member with "Commenter" role
2. **Login as Commenter**: Should see read-only tasks
3. **Try to edit task**: Should be blocked
4. **Try to comment on task**: Should succeed
5. **Go to Settings** (admin): Should see Permission Matrix

### Test API Endpoints

```bash
# Test permission matrix endpoint
curl -H "Authorization: Bearer <TOKEN>" \
  http://localhost:5000/api/permissions/matrix

# Should return: { roles: {...}, resources: [...] }
```

---

## Performance Monitoring

### Socket.io Metrics

**Monitor these in production**:
- **Connection count**: `io.of('/').sockets.size`
- **Rooms**: `io.of('/').adapter.rooms`
- **Messages/second**: Track via backend logging
- **Latency**: Browser DevTools Network tab timing

### Example Node Script

```javascript
// backend/scripts/monitor-socket.js
const app = require('../src/app');

setInterval(() => {
  const io = app.get('io');
  console.log({
    connected_clients: io.of('/').sockets.size,
    rooms: Object.keys(io.of('/').adapter.rooms).length,
    memory_mb: process.memoryUsage().heapUsed / 1024 / 1024
  });
}, 5000);
```

---

## Troubleshooting

### Socket Not Connecting

**Problem**: Browser console shows connection error

**Solutions**:
1. Check backend is running: `curl http://localhost:5000/api/health`
2. Check CORS: Backend should allow frontend origin
3. Check JWT token: `console.log(store.getState().auth.token)`
4. Check Network tab: WebSocket should be pending/open

### Messages Not Syncing

**Problem**: Chat messages appear on sender only

**Solutions**:
1. Verify socket.on listeners are registered
2. Check room name is consistent: `chat:${roomId}`
3. Verify user is in same workspace
4. Check backend socket event emission

**Debug Query**:
```javascript
// In browser console
const socket = getSocket();
socket.on('debug', console.log);
// Messages will log as events occur
```

### Permission Denied Errors

**Problem**: Users can't comment or access features

**Solutions**:
1. Verify role is in database: `SELECT DISTINCT role FROM workspace_members;`
2. Check permission middleware: Review permissions.js
3. Verify user's actual role: Check database table
4. Check permission matrix API: `GET /permissions/matrix`

### Database Enum Error

**Problem**: "invalid enum value" when creating user with 'guest' role

**Solutions**:
1. Run migration: `npx sequelize-cli db:migrate`
2. Verify enum in database:
   ```sql
   SELECT enum_range(NULL::enum_users_role);
   ```
3. Check which migration was last applied:
   ```sql
   SELECT * FROM sequelize_meta ORDER BY name DESC LIMIT 5;
   ```

---

## Rollback Procedure

If deployment fails and you need to rollback:

### Rollback Socket.io Changes

```bash
# Backend: Uninstall socket.io
cd backend
npm uninstall socket.io

# Restore previous server.js (if version control available)
git checkout src/server.js

# Restart server
npm start
```

### Rollback Database

```bash
# CAREFUL: Only if migration caused issues
cd backend

# Undo last migration
npx sequelize-cli db:migrate:undo:all --to 20250106000003

# Or manually remove enums (requires table recreation)
# Not recommended - use database backup instead
```

### Rollback Frontend

```bash
cd frontend
npm uninstall socket.io-client

# Restore previous chatSlice.ts, socketClient.ts
git checkout src/store/slices/chatSlice.ts src/services/socketClient.ts

npm start
```

---

## Monitoring & Alerts

### Key Metrics to Monitor

1. **Socket Connection Rate**: Spike indicates users joining
2. **Message Throughput**: Messages per second across all rooms
3. **CPU Usage**: Socket processing should be <5% baseline
4. **Memory Usage**: Per-connection memory ~1-2MB typical
5. **Error Rate**: Monitor 4xx/5xx responses

### Logging Configuration

**Backend** (src/utils/logger.js):
```javascript
logger.info('Socket client connected', { userId, socketId });
logger.error('Permission denied', { userId, action: 'comment', reason: 'viewer_role' });
logger.warn('High message throughput detected', { messagesPerSecond: 100 });
```

---

## Capacity Planning

### Expected Load per 1000 Concurrent Users

- **Memory**: ~2-3 GB
- **CPU**: ~20-30% (modern 4-core processor)
- **Bandwidth**: ~1-2 Mbps (depends on message frequency)
- **Database Connections**: ~50-100 (from connection pool)

### Scaling Strategy

1. **Horizontal**: Use Socket.io Redis adapter for multi-server deployment
2. **Vertical**: Increase server resources (CPU, RAM)
3. **Database**: Optimize queries, add indexes on roomId, userId
4. **Caching**: Cache permission matrix in Redis

---

## Support Contacts

- **Socket.io Issues**: https://socket.io/docs/v4/
- **Database Issues**: PostgreSQL documentation
- **Frontend**: React TypeScript documentation
- **Internal**: Development team Slack channel

---

## Final Checklist

- [ ] Backend socket.io installed and verified
- [ ] Frontend socket.io-client installed and verified
- [ ] Database migration applied successfully
- [ ] Backend starts without errors
- [ ] Frontend builds without TypeScript errors
- [ ] Socket connection established in browser
- [ ] Chat realtime messages working
- [ ] Whiteboard collaboration working
- [ ] Typing indicators displaying
- [ ] Permission enforcement working
- [ ] Permission matrix API responding
- [ ] No console errors in browser
- [ ] No error logs in backend

---

**Deployment Status**: ✅ READY FOR PRODUCTION

**Last Updated**: January 2025
