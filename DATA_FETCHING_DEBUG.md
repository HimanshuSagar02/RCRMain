# Data Fetching Debugging Guide

If data (courses, assignments, authentication) is not being fetched after deployment, follow these steps:

## 🔍 Step 1: Test Database Connection

```
GET https://your-backend-url.onrender.com/api/test/db
```

**Expected:** `{"success": true, "status": "connected"}`

**If failed:** Check `MONGODB_URL` in Render environment variables.

---

## 🔍 Step 2: Test Data Queries

### Test Courses:
```
GET https://your-backend-url.onrender.com/api/test/courses
```

**Expected:** List of courses with count.

### Test Assignments:
```
GET https://your-backend-url.onrender.com/api/test/assignments
```

**Expected:** List of assignments with count.

### Test Notifications:
```
GET https://your-backend-url.onrender.com/api/test/notifications
```

**Expected:** List of notifications with count.

### Test Shared Notes:
```
GET https://your-backend-url.onrender.com/api/test/shared-notes
```

**Expected:** List of shared notes with count.

### Test Attendance:
```
GET https://your-backend-url.onrender.com/api/test/attendance
```

**Expected:** List of attendance records with count.

### Test Live Classes:
```
GET https://your-backend-url.onrender.com/api/test/live-classes
```

**Expected:** List of live classes with count.

### Test Users:
```
GET https://your-backend-url.onrender.com/api/test/users
```

**Expected:** List of users with count.

---

## 🔍 Step 3: Test Authentication

```
GET https://your-backend-url.onrender.com/api/test/auth-test
```

**This checks:**
- If token exists in cookies
- If token is valid
- If JWT_SECRET is configured

**Expected:** `{"success": true, "message": "Token is valid", "userId": "..."}`

---

## 🔍 Step 4: Check Backend Logs (Render)

When making API requests, check Render logs for:

### Courses:
- `[GetAllCourses] Fetching all courses...`
- `[GetAllCourses] Found X courses`
- `[GetPublishedCourses] Found X published courses`
- `[GetCourseById] Course found: ...`

### Assignments:
- `[GetAssignments] Fetching assignments for course: ...`
- `[GetAssignments] Found X assignments`

### Notifications:
- `[GetMyNotifications] Fetching notifications for user: ...`
- `[GetMyNotifications] Found X notifications`

### Shared Notes:
- `[ListCourseNotes] Fetching notes for user: ...`
- `[ListCourseNotes] Found X notes`

### Attendance:
- `[ListMyAttendance] Fetching attendance for user: ...`
- `[ListMyAttendance] Found X attendance records`

### Live Classes:
- `[GetMyLiveClasses] Fetching live classes for user: ...`
- `[GetMyLiveClasses] Found X live classes`

### Authentication:
- `[isAuth] Token verified for user: ...`
- `[GetCurrentUser] User found: ...`

### Errors:
- `[GetAllCourses] Error: ...`
- `[isAuth] Database not connected`
- `[isAuth] Token verification failed`

---

## 🐛 Common Issues and Fixes

### Issue 1: "Database connection unavailable"

**Symptoms:**
- All API calls return 503
- Logs show: `Database not connected. State: 0`

**Fix:**
1. Check `MONGODB_URL` in Render environment variables
2. Verify MongoDB Atlas Network Access (allow `0.0.0.0/0`)
3. Check MongoDB Atlas cluster is running
4. Restart Render service

### Issue 2: "Authentication required" (but user is logged in)

**Symptoms:**
- API calls return 401
- Logs show: `[isAuth] No token found in cookies`

**Fix:**
1. Check cookie settings in production:
   - `secure: true` (HTTPS only)
   - `sameSite: "None"` (for cross-site)
2. Check browser DevTools → Application → Cookies
3. Verify `VITE_SERVER_URL` in Netlify matches Render backend URL
4. Check CORS settings include your Netlify URL

### Issue 3: "Token expired" or "Invalid token"

**Symptoms:**
- API calls return 401
- Logs show: `[isAuth] Token verification failed`

**Fix:**
1. User needs to login again
2. Check `JWT_SECRET` is set in Render
3. Verify token is being sent in cookies

### Issue 4: Data returns empty arrays

**Symptoms:**
- API calls succeed but return `[]`
- Logs show: `Found 0 courses/assignments/notifications/etc`

**Fix:**
1. Check if data exists in database:
   - `/api/test/courses` - shows total count
   - `/api/test/assignments` - shows total count
   - `/api/test/notifications` - shows total count
   - `/api/test/shared-notes` - shows total count
   - `/api/test/attendance` - shows total count
   - `/api/test/live-classes` - shows total count
2. If count is 0, data doesn't exist (need to create)
3. If count > 0 but API returns empty, check:
   - Query filters (e.g., `isPublished: true`, `isActive: true`)
   - User permissions (students only see enrolled courses)
   - Course/assignment associations
   - User role-based filtering

### Issue 5: "User not found" errors

**Symptoms:**
- `[GetCurrentUser] User not found`
- `[GetEnrolledCourses] User not found`

**Fix:**
1. Verify user exists: `/api/test/user/email@example.com`
2. Check `req.userId` is correct (from token)
3. Verify token contains correct `userId`

---

## ✅ Verification Checklist

After fixing issues:

- [ ] Database connected (`/api/test/db`)
- [ ] Courses exist (`/api/test/courses` shows count > 0)
- [ ] Assignments exist (`/api/test/assignments` shows count > 0)
- [ ] Notifications exist (`/api/test/notifications` shows count > 0)
- [ ] Shared Notes exist (`/api/test/shared-notes` shows count > 0)
- [ ] Attendance records exist (`/api/test/attendance` shows count > 0)
- [ ] Live Classes exist (`/api/test/live-classes` shows count > 0)
- [ ] Authentication works (`/api/test/auth-test`)
- [ ] Backend logs show successful queries
- [ ] Frontend can make API calls (check Network tab)
- [ ] Cookies are set (check Application → Cookies)
- [ ] CORS allows frontend URL

---

## 📊 Debugging Workflow

1. **Test database:** `/api/test/db`
2. **Test data exists:** `/api/test/courses`, `/api/test/assignments`
3. **Test authentication:** `/api/test/auth-test`
4. **Check Render logs** when making actual API calls
5. **Check browser console** for frontend errors
6. **Check Network tab** for API request/response details

---

## 🔧 Quick Fixes

### If database not connected:
```bash
# Check Render environment variables
MONGODB_URL=mongodb+srv://...
```

### If authentication fails:
```bash
# Check Render environment variables
JWT_SECRET=your-secret-key
```

### If CORS errors:
- Add your Netlify URL to backend CORS origins
- Check `FRONTEND_URL` in Render environment variables

### If data empty:
- Verify data exists in MongoDB Atlas
- Check query filters
- Verify user has access to data

---

## 📝 Notes

- All controllers now have detailed logging with `[ControllerName]` prefixes
- Database connection is checked in `isAuth` middleware
- Empty arrays are returned instead of errors (prevents frontend crashes)
- All errors include detailed messages for debugging

