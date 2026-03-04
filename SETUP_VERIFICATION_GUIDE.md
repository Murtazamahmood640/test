# Setup & Verification Guide

## Pre-Setup Checklist

- [ ] Node.js 16+ installed
- [ ] MongoDB running locally or connected via URI
- [ ] Git repository initialized
- [ ] All dependencies installed (`npm install`)

---

## Step 1: Install Dependencies

```bash
cd /path/to/project
npm install
```

**Expected:** No errors, all packages installed

---

## Step 2: Environment Setup

### Backend Configuration
Create/Update `backend/.env`:
```env
MONGODB_URI=mongodb://localhost:27017/luxe-detail-booker
JWT_SECRET=your_jwt_secret_here
ADMIN_EMAIL=admin@example.com
NODE_ENV=development
```

### Frontend Configuration
Create/Update `.env.local` (optional):
```env
VITE_API_URL=http://localhost:3000
```

---

## Step 3: Start Development Environment

### Option A: Start Both Frontend & Backend (Recommended)
```bash
npm run dev:all
```

This will:
- Start Vite dev server on default port (usually 5173)
- Start Express backend on port 3000
- Enable hot module replacement (HMR)

### Option B: Start Separately

Terminal 1 - Backend:
```bash
npm run dev:backend
# or
node --loader ts-node/esm backend/server.ts
```

Terminal 2 - Frontend:
```bash
npm run dev
```

---

## Step 4: Verify Backend is Running

1. Open new terminal
2. Test API endpoint:
```bash
curl -X GET http://localhost:3000/api/appointments
```

**Expected Response:**
```json
{
  "success": true,
  "data": [...]
}
```

If you get "Connection refused", backend is not running.

---

## Step 5: Verify Frontend is Running

1. Open browser
2. Navigate to `http://localhost:5173` (or port shown in terminal)
3. You should see login page

---

## Step 6: Create Admin User (First Time Only)

### Using MongoDB Compass or mongo shell:

```javascript
db.users.insertOne({
  email: "admin@example.com",
  password: "hashedPassword", // Use bcrypt in production
  fullName: "Admin User",
  role: "admin",
  isActive: true,
  createdAt: new Date(),
  updatedAt: new Date()
})
```

Or use the API to create:
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "Password123",
    "fullName": "Admin User",
    "role": "admin"
  }'
```

---

## Step 7: Admin Login

1. Open http://localhost:5173/admin/login
2. Enter credentials:
   - Email: `admin@example.com`
   - Password: `Password123`
3. Click Login

**Expected:** Redirect to admin dashboard with statistics

---

## Feature Verification Checklist

### Dashboard Load
- [ ] Statistics cards display (Total Bookings, Pending, Confirmed, Revenue)
- [ ] Appointments table loads with data
- [ ] No console errors
- [ ] No loading spinners stuck

### Appointments Tab
- [ ] Can view appointment details (click eye icon)
- [ ] Can edit appointment status (click pencil)
- [ ] Confirmation dialog appears for changes
- [ ] Can delete appointment (trash icon → confirm)
- [ ] Can search appointments
- [ ] Can filter by status
- [ ] Can export to CSV
- [ ] Statistics update after changes

### Users Tab
- [ ] User list loads
- [ ] Can toggle user status (on/off switch)
- [ ] Status changes immediately and persists
- [ ] Can edit user (name, role)
- [ ] Can delete user (with confirmation)
- [ ] Can add new user
- [ ] Can search by email/name
- [ ] Can filter by role and status

### Coupons Tab
- [ ] Coupons display as cards
- [ ] Can toggle status
- [ ] Can edit discount percentage and expiry
- [ ] Can delete coupon (with confirmation)
- [ ] Can add new coupon
- [ ] Can search by code
- [ ] Can filter by status

---

## Error Scenarios Testing

### Scenario 1: Backend Down
1. Stop backend server
2. Try to perform any action
3. **Expected:** Error toast: "Error updating appointment"

**Fix:** Restart backend with `npm run dev:backend`

### Scenario 2: Invalid Token
1. Delete token from localStorage manually
2. Refresh page
3. **Expected:** Redirect to login

### Scenario 3: Duplicate Email (Add User)
1. Try to add user with existing email
2. **Expected:** Error message: "User with this email already exists"

### Scenario 4: Delete Confirmation Cancel
1. Click trash icon on any item
2. Click "Cancel" in confirmation dialog
3. **Expected:** Dialog closes, item remains

### Scenario 5: API Validation Error
1. Try to create coupon with discount > 100%
2. **Expected:** Error from backend: "Discount must be between 1 and 100"

---

## Database Verification

### Check MongoDB Collections

```javascript
// Connect to MongoDB
use luxe-detail-booker

// Check Appointments
db.appointments.find().limit(1)

// Check Users
db.users.find().limit(1)

// Check Coupons
db.coupons.find().limit(1)
```

**Expected:** Each collection should have documents with `_id` field

---

## Browser Developer Tools Checklist

### Console Tab
- [ ] No red errors (warnings OK)
- [ ] No 404 errors for assets
- [ ] No CORS errors

### Network Tab
- [ ] GET /api/appointments: 200 status
- [ ] GET /api/users: 200 status
- [ ] GET /api/coupons: 200 status
- [ ] PUT requests return 200
- [ ] DELETE requests return 200
- [ ] All requests have Authorization header

### Application Tab
- [ ] Token stored in localStorage under `adminToken` key
- [ ] No sensitive data in localStorage

---

## Performance Testing

### Load Time
1. Open DevTools → Lighthouse
2. Run performance audit
3. Expected: > 80 score

### API Response Time
```bash
# Measure response time
time curl http://localhost:3000/api/appointments
```
Expected: < 100ms

### Search Performance
1. Type in search box
2. Results filter immediately (< 50ms)

---

## Security Verification

### Authentication
- [ ] Can't access admin without login
- [ ] Token expires after set time
- [ ] Can logout and login again
- [ ] Different users see different data

### Authorization
- [ ] Regular user can't access admin panel
- [ ] User can't create/delete other users
- [ ] User can't modify their own role

### Data Protection
- [ ] Passwords not visible in frontend
- [ ] Sensitive data not in LocalStorage
- [ ] All API calls use HTTPS (in production)

---

## Production Deployment Checklist

- [ ] All environment variables set
- [ ] MongoDB production URI configured
- [ ] JWT secret set to strong value
- [ ] Frontend built: `npm run build`
- [ ] Backend process manager set up (PM2, systemd)
- [ ] HTTPS enabled
- [ ] CORS configured for allowed domains
- [ ] Rate limiting enabled
- [ ] Database backups configured
- [ ] Error logging configured
- [ ] Monitoring set up (optional but recommended)

---

## Troubleshooting Quick Guide

| Issue | Symptom | Solution |
|-------|---------|----------|
| Backend connection error | "Connection refused" | Start backend: `npm run dev:backend` |
| Auth errors | 401/403 errors | Check token is being sent in header |
| No data displayed | Empty tables | Check MongoDB connection |
| CORS error | "No 'Access-Control-Allow-Origin'" | Backend might not be running |
| 404 on API | GET /api/appointments fails | Verify backend routes exist |
| Confirmation dialog not appearing | Can't confirm delete | Check React rendering |
| Can't login | Always redirected to login | Check credentials in database |
| Search not working | Filter doesn't filter | Check useMemo calculation |

---

## Rollback Plan

If something goes wrong:

### Revert Changes
```bash
git log --oneline  # Find previous commit
git revert <commit-hash>  # Revert to previous state
npm install  # Reinstall dependencies
```

### Clear Cache
```bash
# Clear browser cache
# In DevTools: Application → Clear site data

# Clear npm cache
npm cache clean --force

# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Reset Database (if needed)
```bash
# WARNING: This deletes all data!
mongo
use luxe-detail-booker
db.dropDatabase()
```

---

## Performance Optimization Tips

### For Development
- Use DevTools to identify slow functions
- Check Network tab for slow API calls
- Use React DevTools to check unnecessary re-renders
- Profile with Lighthouse

### For Production
- Enable caching headers
- Use CDN for static assets
- Optimize images
- Minify CSS/JS
- Enable gzip compression
- Set up database indexes
- Use connection pooling

---

## Logging & Debugging

### Frontend Logs
```javascript
// Enable debug logging
localStorage.setItem('DEBUG', 'admin:*')

// Check logs
console.log("[v0] Message here")
```

### Backend Logs
Check Express server console output for:
- Request details
- Database queries
- Errors and stack traces

### MongoDB Logging
Enable slow query logging:
```javascript
db.setProfilingLevel(1)
db.system.profile.find().limit(5).sort({ts:-1}).pretty()
```

---

## Health Check Endpoint

Create a health check to monitor system:

```bash
curl -X GET http://localhost:3000/health
```

Expected:
```json
{
  "status": "ok",
  "database": "connected",
  "uptime": 12345
}
```

---

## Support Resources

1. **Documentation**
   - PORTAL_UPDATE_SUMMARY.md - Feature overview
   - IMPLEMENTATION_GUIDE.md - Technical details
   - UPDATED_FILES_LIST.md - Files changed

2. **Backend API**
   - Check `/api/*.ts` files for endpoint details
   - Review `/backend/models/*.ts` for database schemas

3. **Frontend Code**
   - Check AdminDashboard.tsx for component structure
   - Review context files for state management

4. **External Resources**
   - MongoDB docs: docs.mongodb.com
   - Express docs: expressjs.com
   - React docs: react.dev
   - TailwindCSS: tailwindcss.com

---

## Final Sign-Off

- [ ] All tests passed
- [ ] No console errors
- [ ] All features working
- [ ] Database connected
- [ ] Admin can CRUD all items
- [ ] Users can login/logout
- [ ] Confirmations working
- [ ] Notifications showing
- [ ] Performance acceptable
- [ ] Ready for production

---

**Setup Complete!** 🎉

Your admin portal is now fully functional with complete CRUD operations for appointments, users, and coupons. Users can be toggled active/inactive, confirmations prevent accidental deletions, and all changes persist in the database.

For questions or issues, refer to the documentation files or check the browser console for error messages.
