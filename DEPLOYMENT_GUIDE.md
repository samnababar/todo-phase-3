# Complete Deployment Guide: ObsidianList

This guide walks you through deploying the AI-Powered Todo Chatbot to production.

**Architecture:**
- **Frontend**: Vercel (Next.js)
- **Backend**: Render (FastAPI)
- **Database**: Neon (PostgreSQL)

---

## Prerequisites

Before starting, you need accounts on:
1. **GitHub** - https://github.com (to host your code)
2. **Vercel** - https://vercel.com (free tier available)
3. **Render** - https://render.com (free tier available)
4. **Neon** - https://neon.tech (free tier: 0.5GB storage)
5. **OpenAI** - https://platform.openai.com (for AI chat)
6. **Resend** - https://resend.com (for reminder emails, free tier: 100 emails/day)

---

## Step 1: Push Code to GitHub

### 1.1 Create a New GitHub Repository

1. Go to https://github.com/new
2. Repository name: `obsidianlist` (or your preferred name)
3. Make it **Public** or **Private**
4. Click **Create repository**

### 1.2 Push Your Code

Open terminal in your project folder and run:

```bash
# Initialize git (if not already)
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: ObsidianList AI Todo Chatbot"

# Add remote origin (replace with YOUR repository URL)
git remote add origin https://github.com/YOUR_USERNAME/obsidianlist.git

# Push to main branch
git branch -M main
git push -u origin main
```

---

## Step 2: Set Up Neon Database

### 2.1 Create Neon Account & Project

1. Go to https://neon.tech
2. Click **Sign Up** (use GitHub for easy login)
3. Click **Create Project**
4. Project name: `obsidianlist`
5. Region: Choose closest to your users (e.g., `US East`)
6. Click **Create Project**

### 2.2 Get Database Connection String

1. After project creation, you'll see the **Connection Details**
2. Copy the **Connection string** (looks like):
   ```
   postgresql://username:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
   ```
3. **SAVE THIS** - you'll need it for backend deployment

### 2.3 Create Tables

Option A: **Using Neon Console**
1. Click **SQL Editor** in Neon dashboard
2. Run the following SQL:

```sql
-- Users table
CREATE TABLE IF NOT EXISTS "user" (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tasks table
CREATE TABLE IF NOT EXISTS task (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES "user"(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    priority VARCHAR(10) DEFAULT 'medium',
    tags JSONB DEFAULT '[]',
    completed BOOLEAN DEFAULT FALSE,
    completion_date TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Reminders table
CREATE TABLE IF NOT EXISTS reminder (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID NOT NULL REFERENCES task(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES "user"(id) ON DELETE CASCADE,
    reminder_date DATE NOT NULL,
    reminder_day VARCHAR(20) NOT NULL,
    reminder_time TIME NOT NULL,
    sent BOOLEAN DEFAULT FALSE,
    sent_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Conversations table
CREATE TABLE IF NOT EXISTS conversation (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES "user"(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Messages table
CREATE TABLE IF NOT EXISTS message (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID NOT NULL REFERENCES conversation(id) ON DELETE CASCADE,
    role VARCHAR(20) NOT NULL,
    content TEXT NOT NULL,
    tool_calls JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_task_user_id ON task(user_id);
CREATE INDEX IF NOT EXISTS idx_reminder_task_id ON reminder(task_id);
CREATE INDEX IF NOT EXISTS idx_reminder_user_id ON reminder(user_id);
CREATE INDEX IF NOT EXISTS idx_reminder_sent ON reminder(sent);
CREATE INDEX IF NOT EXISTS idx_conversation_user_id ON conversation(user_id);
CREATE INDEX IF NOT EXISTS idx_message_conversation_id ON message(conversation_id);
```

---

## Step 3: Get API Keys

### 3.1 OpenAI API Key

1. Go to https://platform.openai.com/api-keys
2. Click **Create new secret key**
3. Name: `obsidianlist`
4. Copy the key (starts with `sk-`)
5. **SAVE THIS** - you won't see it again!

**Note**: You need to add credits to your OpenAI account for the API to work.

### 3.2 Resend API Key (for Email Reminders)

1. Go to https://resend.com
2. Sign up / Log in
3. Go to **API Keys** in dashboard
4. Click **Create API Key**
5. Name: `obsidianlist`
6. Permission: `Sending access`
7. Copy the key (starts with `re_`)

**Important**: For production emails, you need to:
1. Go to **Domains** in Resend
2. Add and verify your domain
3. Use that domain in `EMAIL_FROM_ADDRESS`

For testing, use: `onboarding@resend.dev` (limited)

### 3.3 Generate JWT Secret

Generate a secure random string (32+ characters):

**Option A**: Use this online generator: https://generate-secret.vercel.app/32

**Option B**: Run in terminal:
```bash
openssl rand -hex 32
```

**Option C**: Use any password generator

Example: `a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6`

---

## Step 4: Deploy Backend to Render

### 4.1 Create Render Account

1. Go to https://render.com
2. Click **Get Started for Free**
3. Sign up with **GitHub** (recommended - easier to connect repos)

### 4.2 Create New Web Service

1. Click **New +** → **Web Service**
2. Connect your GitHub repository
3. Select your `obsidianlist` repo
4. Configure the service:

| Setting | Value |
|---------|-------|
| **Name** | `obsidianlist-api` |
| **Region** | Oregon (US West) or closest to you |
| **Branch** | `main` |
| **Root Directory** | `backend` |
| **Runtime** | `Python 3` |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
| **Instance Type** | Free |

### 4.3 Add Environment Variables

Scroll down to **Environment Variables** and add:

| Key | Value |
|-----|-------|
| `DATABASE_URL` | Your Neon connection string |
| `JWT_SECRET` | Your generated secret (32+ chars) |
| `OPENAI_API_KEY` | Your OpenAI API key (sk-...) |
| `RESEND_API_KEY` | Your Resend API key (re_...) |
| `EMAIL_FROM_ADDRESS` | `onboarding@resend.dev` (or your verified domain) |
| `FRONTEND_URL` | `https://your-app.vercel.app` (update after Vercel deploy) |
| `ALLOWED_ORIGINS` | `https://your-app.vercel.app` (update after Vercel deploy) |
| `PYTHON_VERSION` | `3.11.0` |

### 4.4 Deploy

1. Click **Create Web Service**
2. Wait for build and deployment (5-10 minutes for first deploy)
3. Once deployed, you'll get a URL like: `https://obsidianlist-api.onrender.com`
4. **COPY THIS URL** - you need it for frontend

### 4.5 Test Backend

Open in browser: `https://YOUR-RENDER-URL.onrender.com/health`

You should see:
```json
{"status":"healthy","timestamp":"..."}
```

---

## Step 5: Deploy Frontend to Vercel

### 5.1 Create Vercel Account

1. Go to https://vercel.com
2. Click **Sign Up**
3. Sign up with **GitHub** (recommended)

### 5.2 Import Project

1. Click **Add New...** → **Project**
2. Click **Import** next to your `obsidianlist` repo
3. Configure:

| Setting | Value |
|---------|-------|
| **Project Name** | `obsidianlist` |
| **Framework Preset** | Next.js (auto-detected) |
| **Root Directory** | `frontend` |

### 5.3 Add Environment Variables

Before deploying, add environment variable:

| Key | Value |
|-----|-------|
| `NEXT_PUBLIC_API_URL` | `https://YOUR-RENDER-URL.onrender.com` |

Replace with your actual Render backend URL!

### 5.4 Deploy

1. Click **Deploy**
2. Wait for build (2-3 minutes)
3. Once deployed, you'll get a URL like: `https://obsidianlist.vercel.app`

---

## Step 6: Update Backend with Frontend URL

Now go back to Render and update the environment variables:

1. Go to your Render dashboard
2. Click on `obsidianlist-api` service
3. Go to **Environment** tab
4. Update these variables:

| Key | New Value |
|-----|-----------|
| `FRONTEND_URL` | `https://obsidianlist.vercel.app` (your Vercel URL) |
| `ALLOWED_ORIGINS` | `https://obsidianlist.vercel.app` |

5. Click **Save Changes**
6. Service will automatically redeploy

---

## Step 7: Update Frontend vercel.json (Optional)

If you want to use API rewrites (proxying), update `frontend/vercel.json`:

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://YOUR-RENDER-URL.onrender.com/api/:path*"
    }
  ]
}
```

Then redeploy on Vercel or push to GitHub.

---

## Step 8: Test Your Deployment

### 8.1 Test Backend Health
```
https://YOUR-RENDER-URL.onrender.com/health
```

### 8.2 Test Frontend
1. Open `https://YOUR-VERCEL-URL.vercel.app`
2. You should see the landing page

### 8.3 Test Full Flow
1. Click **Get Started**
2. Sign up with a new account
3. Log in
4. Create a task
5. Try the AI Assistant
6. Set a reminder

---

## Troubleshooting

### Backend won't start
- Check Render logs for errors
- Verify all environment variables are set
- Make sure `DATABASE_URL` has `?sslmode=require`

### CORS Errors
- Make sure `ALLOWED_ORIGINS` includes your Vercel URL (with https://)
- No trailing slash in URLs

### Database Connection Failed
- Verify Neon database is running
- Check connection string format
- Ensure tables are created

### AI Chat Not Working
- Check OpenAI API key is valid
- Verify you have credits in OpenAI account
- Check Render logs for errors

### Emails Not Sending
- Verify Resend API key
- Check `EMAIL_FROM_ADDRESS` is valid
- For production, verify your domain in Resend

### Free Tier Limitations

**Render Free Tier:**
- Service spins down after 15 mins of inactivity
- First request after spin-down takes ~30 seconds
- 750 hours/month

**Neon Free Tier:**
- 0.5 GB storage
- Auto-suspend after 5 mins inactivity

**Vercel Free Tier:**
- 100 GB bandwidth/month
- Serverless function limits

---

## Custom Domain (Optional)

### Vercel Custom Domain
1. Go to Project Settings → Domains
2. Add your domain
3. Update DNS as instructed

### Render Custom Domain
1. Go to Service Settings → Custom Domains
2. Add your domain
3. Update DNS as instructed

**Remember to update:**
- `FRONTEND_URL` on Render
- `ALLOWED_ORIGINS` on Render
- `NEXT_PUBLIC_API_URL` on Vercel (if using custom backend domain)

---

## Summary of URLs Needed

| Service | Purpose | Example |
|---------|---------|---------|
| Neon | Database | `postgresql://...@neon.tech/db` |
| Render | Backend API | `https://obsidianlist-api.onrender.com` |
| Vercel | Frontend | `https://obsidianlist.vercel.app` |

## Environment Variables Checklist

### Backend (Render)
- [ ] `DATABASE_URL`
- [ ] `JWT_SECRET`
- [ ] `OPENAI_API_KEY`
- [ ] `RESEND_API_KEY`
- [ ] `EMAIL_FROM_ADDRESS`
- [ ] `FRONTEND_URL`
- [ ] `ALLOWED_ORIGINS`
- [ ] `PYTHON_VERSION` = `3.11.0`

### Frontend (Vercel)
- [ ] `NEXT_PUBLIC_API_URL`

---

Congratulations! Your AI-Powered Todo Chatbot is now live! 🎉
