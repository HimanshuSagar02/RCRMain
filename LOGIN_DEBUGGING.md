# Login Debugging Guide

This guide will help you identify and fix login issues.

## 🔍 Step 1: Test Database Connection

```
GET https://your-backend-url.onrender.com/api/test/db
```

**Expected Response:**
```json
{
  "success": true,
  "status": "connected",
  "message": "Database is connected"
}
```

**If failed:** Check `MONGODB_URL` in Render environment variables.

---

## 🔍 Step 2: Check if User Exists

```
GET https://your-backend-url.onrender.com/api/test/user/email@example.com
```

**Response shows:**
- User exists: `"success": true`
- Password status: `"passwordInfo": { "hasPassword": true/false }`
- User details (without password)

**If user not found:** User doesn't exist in database.

**If `hasPassword: false`:** User needs password set (see Step 4).

---

## 🔍 Step 3: Test Login (Without Setting Cookie)

```
POST https://your-backend-url.onrender.com/api/test/login-test
Content-Type: application/json

{
  "email": "email@example.com",
  "password": "your-password"
}
```

**This simulates login and shows:**
- User found/not found
- Account status (pending/approved/rejected)
- Password correct/incorrect
- All checks without setting cookies

**Use this to identify the exact issue!**

---

## 🔧 Step 4: Fix Common Issues

### Issue 1: User Has No Password

**Solution A - Quick Fix:**
```
POST https://your-backend-url.onrender.com/api/test/user/email@example.com/set-password
Content-Type: application/json

{
  "password": "NewPassword123"
}
```

**Solution B - Forgot Password Flow:**
1. Go to login page → Click "Forgot Password"
2. Enter email → Get OTP
3. Enter OTP → Set new password
4. Now works even if user had no password!

### Issue 2: Account Status is "pending"

**Solution:** Admin needs to approve the account:
- Login as admin
- Go to Admin Users
- Approve the user

### Issue 3: Incorrect Password

**Solution:** 
- Use forgot password to reset
- Or admin can set new password

### Issue 4: Frontend Can't Connect to Backend

**Check:**
1. `VITE_SERVER_URL` in Netlify environment variables
2. Should be: `https://your-backend-url.onrender.com`
3. Check browser console for CORS errors
4. Check Network tab - requests should go to Render backend

---

## 📊 Step 5: Check Backend Logs (Render)

When trying to login, check Render logs for:

### Success Flow:
```
[Login] Attempting login for email: ...
[Login] User found - ID: ..., Status: ..., Role: ...
[Login] Comparing password for: ...
[Login] Password comparison result: true
[Login] Token generated successfully for: ...
[Login] Cookie set - secure: ..., sameSite: ...
[Login] Login successful for: ..., Role: ..., UserID: ...
```

### Error Flow:
```
[Login] User not found: ...
OR
[Login] User has no password set: ...
OR
[Login] Incorrect password for: ...
OR
[Login] Token generation failed: ...
```

**These logs tell you exactly what's wrong!**

---

## 🐛 Common Errors and Solutions

### Error: "User does not exist"
- **Cause:** User not in database or email mismatch
- **Fix:** Check email spelling, verify user exists in database

### Error: "This account does not have a password set"
- **Cause:** User created without password (Google sign-in or admin creation)
- **Fix:** Use Step 4, Solution A or B

### Error: "Account pending approval"
- **Cause:** User account not approved by admin
- **Fix:** Admin needs to approve account

### Error: "Incorrect password"
- **Cause:** Wrong password or password not hashed correctly
- **Fix:** Use forgot password to reset

### Error: "Failed to generate authentication token"
- **Cause:** `JWT_SECRET` not set in Render
- **Fix:** Add `JWT_SECRET` to Render environment variables

### Error: "Cannot connect to server"
- **Cause:** Frontend can't reach backend
- **Fix:** 
  - Check `VITE_SERVER_URL` in Netlify
  - Check backend is running (Render dashboard)
  - Check CORS settings

### Error: CORS error in browser
- **Cause:** Frontend URL not in backend CORS origins
- **Fix:** Add your Netlify URL to backend CORS in `backend/index.js`

---

## ✅ Verification Checklist

After fixing issues, verify:

- [ ] Database connected (`/api/test/db`)
- [ ] User exists (`/api/test/user/:email`)
- [ ] User has password (`passwordInfo.hasPassword: true`)
- [ ] User status is "approved"
- [ ] Test login works (`/api/test/login-test`)
- [ ] Frontend `VITE_SERVER_URL` is correct
- [ ] Backend logs show successful login
- [ ] Browser console shows no errors
- [ ] Cookie is set (check Application → Cookies in DevTools)

---

## 🔄 Quick Fix Workflow

1. **Test database:** `/api/test/db`
2. **Check user:** `/api/test/user/email@example.com`
3. **If no password:** `/api/test/user/email@example.com/set-password`
4. **Test login:** `/api/test/login-test`
5. **Try actual login** on frontend
6. **Check logs** if still failing

---

## 📝 Notes

- All test endpoints are available at `/api/test/*`
- Test endpoints don't set cookies (safe to use)
- Backend logs show detailed `[Login]` messages
- Frontend now uses `serverUrl` from App.jsx (uses `VITE_SERVER_URL`)
- Cookies work in production with `secure: true` and `sameSite: "None"`

