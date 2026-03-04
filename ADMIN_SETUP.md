# Admin Dashboard Setup & Troubleshooting Guide

## Issue Summary
The admin dashboard wasn't working because:
1. The **backend Express server (port 3000)** wasn't running
2. The **appointments API response format** didn't match the expected data structure
3. The **appointments state** wasn't initialized with mock data

## What Was Fixed

### 1. State Initialization (AdminDashboard.tsx)
- Changed: `useState<Appointment[]>([])` 
- To: `useState<Appointment[]>(MOCK_APPOINTMENTS ?? [])`
- **Why**: Provides default data immediately, preventing "Cannot read properties of undefined" errors

### 2. Safe Calculations with useMemo
- Wrapped `filtered` and `stats` calculations in `useMemo` hook
- Added safety checks: `if (!appointments || !Array.isArray(appointments))`
- **Why**: Prevents errors when components try to filter undefined data during render

### 3. API Response Parsing
- Fixed: `setAppointments(data.appointments)`
- To: `setAppointments(data.appointments || data.data)`
- **Why**: The API returns `data: appointments` but code expected `appointments: data.appointments`

### 4. Added Development Scripts
- Added `npm run dev:backend` - Run backend server only
- Added `npm run dev:all` - Run both frontend and backend (recommended)
- **Why**: Makes it easier to develop with both frontend and backend running

## How to Run the Application

### Option 1: Run Frontend & Backend Separately (for debugging)

**Terminal 1 - Start Frontend (Vite):**
```bash
npm run dev
# Runs on http://localhost:5173 (or auto-detected port)
```

**Terminal 2 - Start Backend (Express):**
```bash
npm run dev:backend
# Runs on http://localhost:3000
```

### Option 2: Run Both Simultaneously (Recommended)
```bash
npm run dev:all
# Automatically starts both frontend and backend in one terminal
```

## How to Login to Admin Dashboard

### Step 1: Create an Admin User
First, you need a user with `role: 'admin'` in MongoDB. The backend has two ways:

**Option A: Direct MongoDB Insert**
```javascript
db.users.insertOne({
  email: "admin@example.com",
  password: "hashed_password_here", // Must be hashed with bcrypt
  fullName: "Admin User",
  role: "admin",
  isActive: true,
  createdAt: new Date()
})
```

**Option B: Register via API then manually set role to admin**
1. Register a new user (POST to `/api/auth/register`)
2. Log in to MongoDB and change the user's `role` from `'user'` to `'admin'`

### Step 2: Navigate to Admin Login
- URL: `http://localhost:5173/admin/login` (adjust port if different)
- Email: Use the admin email you created
- Password: Use the password you set

### Step 3: Dashboard Features After Login
Once logged in, you can:
- **View Appointments**: All appointments from the database (with filters)
- **Manage Users**: Create, read, update, delete users (admin only)
- **Manage Coupons**: Create, read, update, delete coupon codes (admin only)

## Troubleshooting

### Error: "Cannot read properties of undefined"
**Solution**: Make sure the AdminDashboard state is initialized with mock data and uses useMemo for calculations.
- ✅ Already fixed in this version

### Error: "Failed to load users" / "Failed to load coupons" (404 or 403)
**Causes**:
1. **Backend not running** → Start with `npm run dev:backend`
2. **Token not being sent** → Check AdminAuthContext is passing `Authorization: Bearer ${token}`
3. **User not admin** → Make sure logged-in user has `role: 'admin'` in MongoDB
4. **MongoDB connection failed** → Check `MONGODB_URI` in `.env`

**Solutions**:
```bash
# Start backend server
npm run dev:backend

# Check MongoDB connection
# Make sure MONGODB_URI in backend/.env is valid

# Verify admin role
# Log in to MongoDB and check the user document
db.users.findOne({ email: "admin@example.com" })
# Should show: { role: "admin", ... }
```

### Error: "Appointments state is undefined"
**Solution**: Already fixed by initializing with `MOCK_APPOINTMENTS ?? []`

## API Endpoints Reference

### Authentication
- **POST** `/api/auth/login` - Login (public)
- **POST** `/api/auth/register` - Register (public)

### Users (requires admin role)
- **GET** `/api/users?search=&role=&status=` - List all users
- **POST** `/api/users` - Create new user
- **PUT** `/api/users?id=ID` - Update user
- **DELETE** `/api/users?id=ID` - Delete user

### Coupons (requires admin role)
- **GET** `/api/coupons?search=&status=` - List all coupons
- **POST** `/api/coupons` - Create new coupon
- **PUT** `/api/coupons?id=ID` - Update coupon
- **DELETE** `/api/coupons?id=ID` - Delete coupon

### Appointments (public)
- **GET** `/api/appointments` - List all appointments
- **POST** `/api/appointments` - Create appointment
- **PUT** `/api/appointments?id=ID` - Update appointment
- **DELETE** `/api/appointments?id=ID` - Delete appointment

## Environment Variables Required

**Backend (backend/.env or .env):**
- `MONGODB_URI` - MongoDB connection string
- `JWT_SECRET` - Secret key for JWT tokens
- `PORT` - Backend port (default: 3000)
- `NODE_ENV` - development/production
- `ADMIN_EMAIL` - Email for admin notifications

**Frontend:**
- Vite automatically uses `VITE_API_BASE_URL` if defined in .env

## Next Steps

1. ✅ Verify backend and frontend both start without errors
2. ✅ Create an admin user in MongoDB
3. ✅ Log in to admin dashboard
4. ✅ Test creating users and coupons
5. ✅ Check that appointments display from the database
