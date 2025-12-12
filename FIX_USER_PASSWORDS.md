# Fix Users Without Passwords

If you have users in the database who cannot login because they don't have passwords set, follow these steps:

## 🔍 Step 1: Check User Password Status

Test if a user has a password:

```
GET https://your-backend-url.onrender.com/api/test/user/email@example.com
```

Response will show:
```json
{
  "success": true,
  "user": { ... },
  "passwordInfo": {
    "hasPassword": false,
    "passwordLength": 0,
    "isHashed": false
  }
}
```

## 🔧 Step 2: Set Password for User

### Option A: Using Test Endpoint (Quick Fix)

```
POST https://your-backend-url.onrender.com/api/test/user/email@example.com/set-password
Content-Type: application/json

{
  "password": "NewPassword123"
}
```

### Option B: Using Forgot Password Flow (Recommended)

1. Go to login page
2. Click "Forgot Password"
3. Enter user's email
4. Check email for OTP
5. Enter OTP and set new password

**Note:** The reset password function now allows setting password even if user has no password (first time setup).

### Option C: Admin Can Set Password

If you're logged in as admin:
```
PATCH https://your-backend-url.onrender.com/api/admin/users/:userId/password
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "password": "NewPassword123"
}
```

## 🐛 Debugging Login Issues

### Check Backend Logs

When trying to login, check Render logs for:
- `[Login] Attempting login for email: ...`
- `[Login] User found - ID: ..., Status: ..., Role: ...`
- `[Login] User has no password set: ...` ← This means user needs password
- `[Login] Password comparison result: true/false`
- `[Login] Login successful` or error messages

### Common Issues

1. **"User has no password set"**
   - Solution: Use one of the methods above to set password

2. **"Incorrect password"**
   - Password might be wrong
   - Password might not be hashed correctly in database
   - Check if password was set using the correct method

3. **"User does not exist"**
   - User might not be in database
   - Email might be case-sensitive (now fixed to be case-insensitive)

## ✅ Verification

After setting password, test login:
1. Try to login with email and new password
2. Check Render logs for `[Login] Login successful`
3. Should return user data and set authentication cookie

## 📝 Notes

- Passwords must be at least 6 characters
- Passwords are hashed using bcrypt before storing
- Users created via Google sign-in don't have passwords by default
- Users created by admin should have passwords set during creation

