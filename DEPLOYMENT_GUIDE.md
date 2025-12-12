# Deployment Guide for RCR Platform

This guide will help you deploy the RCR platform with backend on Render and frontend on Netlify.

## 📋 Prerequisites

1. **MongoDB Database**: 
   - Create a MongoDB Atlas account at https://www.mongodb.com/cloud/atlas
   - Create a new cluster (free tier is fine)
   - Get your connection string (MONGODB_URL)

2. **Render Account**: For backend hosting
3. **Netlify Account**: For frontend hosting

---

## 🔧 Backend Deployment on Render

### Step 1: Prepare Backend

1. Make sure your `backend` folder has a `package.json` with a `start` script:
   ```json
   {
     "scripts": {
       "start": "node index.js",
       "dev": "nodemon index.js"
     }
   }
   ```

### Step 2: Create Render Web Service

1. Go to [Render Dashboard](https://dashboard.render.com/)
2. Click **"New +"** → **"Web Service"**
3. Connect your GitHub repository
4. Configure:
   - **Name**: `rcr-backend` (or your preferred name)
   - **Environment**: `Node`
   - **Build Command**: `cd backend && npm install`
   - **Start Command**: `cd backend && npm start`
   - **Root Directory**: Leave empty (or set to `backend` if your repo root is different)

### Step 3: Set Environment Variables in Render

Go to **Environment** tab and add these variables:

```env
# Database
MONGODB_URL=your-mongodb-atlas-connection-string

# Server
PORT=10000
NODE_ENV=production

# JWT
JWT_SECRET=your-strong-random-secret-key-here

# Email (for forgot password)
EMAIL=your-email@gmail.com
EMAIL_PASS=your-gmail-app-password

# Cloudinary (for file uploads)
CLOUDINARY_CLOUD_NAME=your-cloudinary-cloud-name
CLOUDINARY_API_KEY=your-cloudinary-api-key
CLOUDINARY_API_SECRET=your-cloudinary-api-secret

# AI (Gemini)
GEMINI_API_KEY=your-gemini-api-key

# LiveKit (for live classes)
LIVEKIT_API_KEY=your-livekit-api-key
LIVEKIT_API_SECRET=your-livekit-api-secret
LIVEKIT_URL=wss://your-livekit-url.livekit.cloud

# Payment (Razorpay)
RAZORPAY_KEY_ID=your-razorpay-key-id
RAZORPAY_KEY_SECRET=your-razorpay-key-secret
```

**Important Notes:**
- Replace all `your-*` values with your actual credentials
- For `MONGODB_URL`, use your MongoDB Atlas connection string (format: `mongodb+srv://username:password@cluster.mongodb.net/dbname?retryWrites=true&w=majority`)
- Make sure to whitelist Render's IP (or use `0.0.0.0/0` for all IPs) in MongoDB Atlas Network Access

### Step 4: Deploy

1. Click **"Create Web Service"**
2. Render will automatically build and deploy your backend
3. Wait for deployment to complete
4. Copy your backend URL (e.g., `https://rcr-backend.onrender.com`)

---

## 🎨 Frontend Deployment on Netlify

### Step 1: Prepare Frontend

1. Create a `.env.production` file in your `frontend` folder (or use Netlify environment variables):
   ```env
   VITE_SERVER_URL=https://your-backend-url.onrender.com
   VITE_FIREBASE_APIKEY=your-firebase-api-key
   ```

### Step 2: Create Netlify Site

1. Go to [Netlify Dashboard](https://app.netlify.com/)
2. Click **"Add new site"** → **"Import an existing project"**
3. Connect your GitHub repository
4. Configure build settings:
   - **Base directory**: `frontend`
   - **Build command**: `npm run build`
   - **Publish directory**: `frontend/dist`

### Step 3: Set Environment Variables in Netlify

1. Go to **Site settings** → **Environment variables**
2. Add:
   ```env
   VITE_SERVER_URL=https://your-backend-url.onrender.com
   VITE_FIREBASE_APIKEY=your-firebase-api-key
   ```

**Important:** Replace `https://your-backend-url.onrender.com` with your actual Render backend URL.

### Step 4: Update CORS in Backend

Make sure your backend CORS includes your Netlify URL. Update `backend/index.js`:

```javascript
app.use(cors({
    origin: [
        "https://rajchemreactor.netlify.app",
        "https://your-netlify-site.netlify.app", // Add your actual Netlify URL
        "http://localhost:5173" // For local development
    ],
    credentials: true
}))
```

### Step 5: Deploy

1. Click **"Deploy site"**
2. Netlify will build and deploy your frontend
3. Your site will be live at `https://your-site.netlify.app`

---

## ✅ Post-Deployment Checklist

### Backend (Render)
- [ ] Backend is accessible (check Render logs)
- [ ] Database connection successful (check Render logs for "✅ DB connected successfully")
- [ ] All environment variables are set
- [ ] CORS includes your Netlify frontend URL

### Frontend (Netlify)
- [ ] Frontend builds successfully
- [ ] `VITE_SERVER_URL` points to your Render backend
- [ ] Site is accessible
- [ ] Can make API calls to backend (check browser console)

### Testing
- [ ] Try to sign up - should work
- [ ] Try to log in - should work
- [ ] Check browser console for errors
- [ ] Check Network tab - API calls should go to Render backend

---

## 🐛 Troubleshooting

### "Login/Signup Failed" or Database Not Working

1. **Test Database Connection**:
   - Visit: `https://your-backend-url.onrender.com/api/test/db`
   - Should return: `{"success": true, "status": "connected"}`
   - If not connected, check `MONGODB_URL` in Render environment variables

2. **Test User Query**:
   - Visit: `https://your-backend-url.onrender.com/api/test/users`
   - Should return list of users
   - If empty, your database might be empty or connection is wrong

3. **Test Specific User**:
   - Visit: `https://your-backend-url.onrender.com/api/test/user/your-email@example.com`
   - Should return user details if user exists
   - Use this to verify if your user exists in the database

4. **Check Backend Logs (Render)**:
   - Go to Render dashboard → Your service → Logs
   - Look for database connection errors
   - Look for `[Login]` messages when trying to login
   - Check if `MONGODB_URL` is set correctly
   - Look for: `✅ DB connected successfully`

5. **Check MongoDB Atlas**:
   - Verify your connection string is correct
   - Check Network Access - whitelist `0.0.0.0/0` (all IPs) or Render's IP
   - Verify database user has correct permissions
   - Check if your database/collection exists

6. **Check CORS**:
   - Verify your Netlify URL is in the CORS origins list
   - Check browser console for CORS errors
   - Cookie settings: In production, `sameSite: "None"` and `secure: true` are required

7. **Check Environment Variables**:
   - Verify all required variables are set in Render:
     - `MONGODB_URL` (most important!)
     - `JWT_SECRET`
     - `NODE_ENV=production`
   - Check `VITE_SERVER_URL` in Netlify matches your Render backend URL

8. **Check Network Requests**:
   - Open browser DevTools → Network tab
   - Try to login/signup
   - Check if requests are going to the correct backend URL
   - Check response status codes (should be 200, not 500 or 401)
   - Check response body for error messages

9. **Common Issues**:
   - **"User does not exist"**: User might not be in database, or email case mismatch
   - **"Database not connected"**: Check `MONGODB_URL` and MongoDB Atlas network access
   - **"Login error"**: Check Render logs for detailed error messages (now improved)
   - **Cookies not working**: In production, need `secure: true` and `sameSite: "None"`

### Common Errors

**Error: "Failed to fetch"**
- Backend URL is incorrect
- CORS is blocking the request
- Backend is not running

**Error: "Database connection failed"**
- `MONGODB_URL` is incorrect
- MongoDB Atlas network access not configured
- Database credentials are wrong

**Error: "CORS policy"**
- Frontend URL not in backend CORS origins
- Missing `credentials: true` in CORS config

---

## 📝 Additional Notes

1. **Render Free Tier**:
   - Services spin down after 15 minutes of inactivity
   - First request after spin-down may take 30-60 seconds
   - Consider upgrading for production use

2. **Netlify**:
   - Free tier is generous and suitable for production
   - Automatic deployments on git push
   - Custom domain support

3. **MongoDB Atlas**:
   - Free tier (M0) is perfect for development
   - Consider upgrading for production with more data

4. **Environment Variables**:
   - Never commit `.env` files to Git
   - Always set variables in deployment platforms
   - Use different values for production vs development

---

## 🔄 Updating Your Deployment

### Backend (Render)
- Push changes to your GitHub repository
- Render will automatically detect and deploy changes

### Frontend (Netlify)
- Push changes to your GitHub repository
- Netlify will automatically build and deploy

---

## 📞 Support

If you encounter issues:
1. Check Render logs for backend errors
2. Check Netlify build logs for frontend errors
3. Check browser console for client-side errors
4. Verify all environment variables are set correctly

