# Setup Guide for Google Sign-in and Forgot Password

## 🔐 Google Sign-in Setup

### Step 1: Get Firebase API Key
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project (or create a new one)
3. Go to Project Settings (gear icon)
4. Under "Your apps", find your web app
5. Copy the `apiKey` value

### Step 2: Configure Frontend
Create or update `frontend/.env` file:
```env
VITE_FIREBASE_APIKEY=your-firebase-api-key-here
```

### Step 3: Enable Google Sign-in in Firebase
1. In Firebase Console, go to **Authentication**
2. Click **Get Started** (if not already enabled)
3. Go to **Sign-in method** tab
4. Enable **Google** provider
5. Add your authorized domains (e.g., `localhost`)

### Step 4: Test
- Restart your frontend dev server
- Try Google sign-in on Login/SignUp page
- Check browser console for any errors

---

## 📧 Forgot Password (Email) Setup

### Step 1: Get Gmail App Password
1. Go to your Google Account: https://myaccount.google.com/
2. Enable **2-Step Verification** (required for App Passwords)
3. Go to **Security** → **2-Step Verification** → **App passwords**
4. Select app: **Mail**
5. Select device: **Other (Custom name)** → Enter "RCR"
6. Click **Generate**
7. Copy the 16-character password (no spaces)

### Step 2: Configure Backend
Create or update `backend/.env` file:
```env
EMAIL=your-email@gmail.com
EMAIL_PASS=your-16-char-app-password
```

**Important:** 
- Use your Gmail address for `EMAIL`
- Use the App Password (not your regular Gmail password) for `EMAIL_PASS`
- The App Password will look like: `abcd efgh ijkl mnop` (remove spaces)

### Step 3: Test Email Configuration
After starting your backend server, you can test email:
```bash
GET http://localhost:8000/api/test/test-email?email=your-test@email.com
```

Or use curl:
```bash
curl "http://localhost:8000/api/test/test-email?email=your-test@email.com"
```

### Step 4: Test Forgot Password Flow
1. Go to Login page
2. Click "Forget your password?"
3. Enter your email
4. Check your email inbox (and spam folder) for OTP
5. Enter OTP and reset password

---

## 🐛 Troubleshooting

### Google Sign-in Issues:
- **Error: "Google sign-in is not configured"**
  - Check if `VITE_FIREBASE_APIKEY` is set in `frontend/.env`
  - Restart frontend dev server after adding env variable

- **Error: "Popup blocked"**
  - Allow popups for localhost in your browser
  - Try a different browser

- **Error: "auth/popup-closed-by-user"**
  - User closed the popup - this is normal, just try again

### Email Issues:
- **Error: "Email configuration missing"**
  - Check if `EMAIL` and `EMAIL_PASS` are set in `backend/.env`
  - Restart backend server after adding env variables

- **Error: "EAUTH" or "Authentication failed"**
  - Make sure you're using App Password, not regular password
  - Verify 2-Step Verification is enabled
  - Generate a new App Password

- **OTP not received**
  - Check spam/junk folder
  - Verify email address is correct
  - Check backend console for error messages
  - Test email endpoint: `/api/test/test-email?email=your@email.com`

---

## ✅ Quick Checklist

### Google Sign-in:
- [ ] Firebase project created
- [ ] Google provider enabled in Firebase
- [ ] `VITE_FIREBASE_APIKEY` in `frontend/.env`
- [ ] Frontend server restarted
- [ ] Tested Google sign-in

### Forgot Password:
- [ ] 2-Step Verification enabled on Google Account
- [ ] App Password generated
- [ ] `EMAIL` and `EMAIL_PASS` in `backend/.env`
- [ ] Backend server restarted
- [ ] Email test endpoint works
- [ ] Tested forgot password flow

---

## 📝 Example .env Files

### `frontend/.env`:
```env
VITE_FIREBASE_APIKEY=AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

### `backend/.env`:
```env
EMAIL=youremail@gmail.com
EMAIL_PASS=abcdefghijklmnop
PORT=8000
MONGODB_URI=your-mongodb-connection-string
JWT_SECRET=your-jwt-secret
GEMINI_API_KEY=your-gemini-api-key
CLOUDINARY_CLOUD_NAME=your-cloudinary-name
CLOUDINARY_API_KEY=your-cloudinary-key
CLOUDINARY_API_SECRET=your-cloudinary-secret
LIVEKIT_API_KEY=APIgxiUFATz9e75
LIVEKIT_API_SECRET=your-livekit-api-secret
LIVEKIT_URL=wss://rcrtech-kla69owt.livekit.cloud
```

---

## 🆘 Still Having Issues?

1. Check browser console for frontend errors
2. Check backend terminal for server errors
3. Verify all environment variables are set correctly
4. Make sure servers are restarted after .env changes
5. Test email endpoint to verify email configuration
6. Check Firebase Console for authentication errors

