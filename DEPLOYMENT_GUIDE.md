# DEPLOYMENT GUIDE: Phase III Todo AI Chatbot

Complete guide to deploy the full-stack Todo AI chatbot with MCP, OpenAI Agents SDK, and ChatKit.

## 📋 Overview

You have 3 things to deploy:
1. **Database** (Neon PostgreSQL)
2. **Backend** (FastAPI on Railway)
3. **Frontend** (ChatKit/Next.js on Vercel)

**Architecture:**
- **Frontend**: Vercel (Next.js with ChatKit)
- **Backend**: Railway (FastAPI with MCP + OpenAI Agents SDK)
- **Database**: Neon (PostgreSQL)

## Prerequisites

Before starting, you need accounts on:
1. **GitHub** - https://github.com (already done ✅)
2. **Vercel** - https://vercel.com (free tier available)
3. **Railway** - https://railway.app (free tier available)
4. **Neon** - https://neon.tech (free tier: 0.5GB storage)
5. **OpenAI** - https://platform.openai.com (for AI chat)

---

## 1️⃣ DATABASE DEPLOYMENT (Neon PostgreSQL)

### Step 1: Create Neon Database

1. Go to [Neon Console](https://console.neon.tech)
2. Click **"Create a new project"**
3. Choose PostgreSQL version (15 or latest)
4. Create project
5. Go to **"Connection string"** tab
6. Copy the **DATABASE_URL** (looks like):
   ```
   postgresql://user:password@ep-xyz.neon.tech/dbname
   ```

### Step 2: Update Environment Variables

In your local `.env` file (or Railway/Vercel later):

```env
DATABASE_URL=postgresql://user:password@ep-xyz.neon.tech/dbname
```

### Step 3: Run Database Migrations

**Run this locally ONCE before deploying:**

```bash
cd backend
python -m alembic upgrade head
```

Or if using custom migration script:

```bash
python reset_db.py
```

**✅ Database is ready**

---

## 2️⃣ BACKEND DEPLOYMENT (FastAPI)

### Recommended: Railway.app

Railway integrates perfectly with FastAPI + Neon PostgreSQL.

### Step 1: Push Backend to GitHub

Already done ✅ (commit: 9b8f7e7)

### Step 2: Railway Setup

1. Go to [railway.app](https://railway.app)
2. Click **"New Project"** → **"Deploy from GitHub"**
3. Select your repository: `samnababar/todo-phase-3`
4. Set **Root Directory** = `backend`
5. Click **"Deploy"**

### Step 3: Add Environment Variables (⚠️ CRITICAL)

In Railway Dashboard → **Variables**:

```env
OPENAI_API_KEY=sk-xxxx... (your OpenAI API key)
DATABASE_URL=postgresql://user:password@ep-xyz.neon.tech/dbname
BETTER_AUTH_SECRET=your-secure-random-secret-here
ENVIRONMENT=production
```

**How to get OPENAI_API_KEY:**
- Go to [OpenAI Platform](https://platform.openai.com/api-keys)
- Create new API key
- Copy and paste into Railway Variables

**How to generate BETTER_AUTH_SECRET:**
```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

### Step 4: Start Command

Railway auto-detects FastAPI, but ensure your Procfile or start command is:

```
uvicorn main:app --host 0.0.0.0 --port $PORT
```

Check `backend/main.py` has:
```python
if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000)
```

### Step 5: Deploy

1. Click **"Deploy"** in Railway
2. Wait for build to complete (2-3 minutes)
3. Railway will assign URL: `https://todo-phase-3-production.up.railway.app`

### Step 6: Verify Backend Deployment

Test health endpoint:

```bash
curl https://your-railway-url/health
```

Expected response:
```json
{"status": "ok", "message": "Backend is running"}
```

**✅ Backend deployed**

---

## 3️⃣ FRONTEND DEPLOYMENT (Next.js on Vercel)

### Step 1: Deploy to Vercel

1. Go to [vercel.com](https://vercel.com)
2. Click **"New Project"**
3. Click **"Import Git Repository"**
4. Select: `samnababar/todo-phase-3`
5. Set **Root Directory** = `frontend`
6. Click **"Deploy"**

Vercel auto-builds Next.js apps. Wait 2-3 minutes for deployment.

### Step 2: Add Environment Variables

In Vercel → Your Project → **Settings** → **Environment Variables**:

```env
NEXT_PUBLIC_API_URL=https://your-backend-railway-url
NEXT_PUBLIC_OPENAI_DOMAIN_KEY=xxxx (see Step 3)
```

⚠️ **IMPORTANT**: Use `NEXT_PUBLIC_` prefix for frontend-accessible env vars.

### Step 3: Redeploy

After adding env vars, click **"Redeploy"** in Vercel.

**✅ Frontend deployed**

---

## 4️⃣ OPENAI DOMAIN ALLOWLIST (⚠️ DON'T SKIP)

### ⚠️ CRITICAL: ChatKit Requires Domain Allowlist

ChatKit will NOT work without this configuration. It's a security requirement from OpenAI.

### Step 1: Get Your Vercel Domain

After Vercel deployment, you'll have:
```
https://your-project.vercel.app
```

### Step 2: Add Domain to OpenAI Allowlist

1. Go to [OpenAI Security Settings](https://platform.openai.com/settings/organization/security/domain-allowlist)
2. Click **"Add Domain"**
3. Enter your Vercel domain:
   ```
   your-project.vercel.app
   ```
4. Click **"Add"**

### Step 3: Get Domain Key

1. In OpenAI Security Settings, find **"Domain Key"** section
2. Click **"Create Key"** or **"View Key"**
3. Copy the domain key (looks like): `sk-domain-xxxx...`

### Step 4: Add Domain Key to Vercel

In Vercel → Environment Variables:

```env
NEXT_PUBLIC_OPENAI_DOMAIN_KEY=sk-domain-xxxx...
```

### Step 5: Redeploy Frontend

Click **"Redeploy"** in Vercel to apply the domain key.

**✅ OpenAI domain allowlist configured**

---

## 5️⃣ FINAL INTEGRATION TEST

### Test Checklist

After all 3 are deployed, test:

| Action | Expected Behavior | Endpoint |
|--------|-------------------|----------|
| Open frontend | Loads without errors | https://your-app.vercel.app |
| Add task | Creates task via MCP add_task | POST /api/{user_id}/chat |
| List tasks | Shows all tasks via MCP list_tasks | GET /api/{user_id}/tasks |
| Complete task | Marks complete via MCP complete_task | POST /api/{user_id}/chat |
| Restart backend | Chat still works (stateless ✅) | /health endpoint |
| Refresh page | Conversation resumes | Previous messages load |

### Test Commands (Local Terminal)

```bash
# Health check
curl https://your-railway-url/health

# Create task via API
curl -X POST https://your-railway-url/api/1/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"message": "Add task: Buy groceries by tomorrow"}'

# List tasks
curl https://your-railway-url/api/1/tasks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### If Tests Fail

**Backend returns 500:**
- Check Railway logs: Railway Dashboard → Deployments → View logs
- Verify DATABASE_URL is correct
- Verify OPENAI_API_KEY is valid

**Frontend loads but chat doesn't work:**
- Check browser console (F12) for errors
- Verify NEXT_PUBLIC_API_URL is correct (no trailing slash)
- Verify domain is in OpenAI allowlist

**ChatKit shows domain error:**
- Verify domain is in OpenAI Security Settings
- Verify NEXT_PUBLIC_OPENAI_DOMAIN_KEY is set in Vercel
- Redeploy frontend after adding key

**✅ All tests pass = Phase III fully successful**

---

## 6️⃣ SUBMISSION CHECKLIST

Before submitting, verify:

- [ ] Database (Neon) created and accessible
- [ ] Backend (Railway) deployed and responding to /health
- [ ] Frontend (Vercel) deployed and loads without errors
- [ ] OPENAI_API_KEY set in Railway
- [ ] Domain in OpenAI allowlist
- [ ] NEXT_PUBLIC_OPENAI_DOMAIN_KEY set in Vercel
- [ ] Can add task via chat
- [ ] Can list tasks
- [ ] Can complete task
- [ ] Conversation persists on refresh
- [ ] Backend restart doesn't lose chat (stateless ✅)

---

## 7️⃣ WHAT TO MENTION IN SUBMISSION

You can confidently say:

> "Deployed full-stack AI Todo chatbot using MCP, OpenAI Agents SDK, stateless FastAPI backend, Neon PostgreSQL, and ChatKit frontend. Implemented 5 MCP tools (add_task, list_tasks, update_task, complete_task, delete_task) with stateless request-response architecture for horizontal scalability. Database migrations via Alembic, JWT authentication, real-time conversation persistence, and OpenAI domain allowlist integration for ChatKit security."

**That sentence alone is 🔥**

---

## 8️⃣ ENVIRONMENT VARIABLES REFERENCE

### Backend (.env)

```env
DATABASE_URL=postgresql://user:password@ep-xyz.neon.tech/dbname
OPENAI_API_KEY=sk-proj-xxxxx
OPENAI_MODEL=gpt-4
BETTER_AUTH_SECRET=your-secret-here
ENVIRONMENT=production
JWT_SECRET=your-jwt-secret
JWT_EXPIRATION_HOURS=168
```

### Frontend (.env.local)

```env
NEXT_PUBLIC_API_URL=https://your-railway-url
NEXT_PUBLIC_OPENAI_DOMAIN_KEY=sk-domain-xxxxx
```

### Railway Dashboard Variables

Same as Backend .env

### Vercel Environment Variables

Same as Frontend .env.local (with NEXT_PUBLIC_ prefix)

---

## 9️⃣ TROUBLESHOOTING

### Issue: "Cannot connect to database"
**Solution:**
- Verify DATABASE_URL in Railway Variables
- Test locally: `psql <DATABASE_URL>`
- Check Neon dashboard for active connections

### Issue: "OpenAI API key invalid"
**Solution:**
- Go to https://platform.openai.com/api-keys
- Create new API key
- Update OPENAI_API_KEY in Railway
- Redeploy backend

### Issue: "ChatKit domain not allowed"
**Solution:**
- Go to https://platform.openai.com/settings/organization/security/domain-allowlist
- Add your Vercel domain
- Wait 2 minutes for propagation
- Refresh frontend

### Issue: "Chat endpoint returns 401"
**Solution:**
- Verify JWT token is valid
- Check Authorization header format: `Bearer <token>`
- Verify user_id in URL matches token

### Issue: "Conversation doesn't persist on refresh"
**Solution:**
- Check browser console for errors
- Verify conversation_id is stored in localStorage
- Check database: `SELECT * FROM conversations;`
- Verify message history is being loaded

---

## 🔟 PERFORMANCE OPTIMIZATION (Optional)

### Backend Performance Targets

- Chat endpoint: **< 3 seconds** (P95)
- MCP tool execution: **< 500ms**
- Database queries: **< 100ms**

### Monitor Performance

Railway Dashboard → Metrics:
- Response time
- CPU usage
- Memory usage
- Error rate

### If Performance Issues

1. Check database indexes
2. Add caching (Redis on Railway)
3. Implement rate limiting
4. Optimize agent prompt (fewer tokens)

---

## Support & Troubleshooting

If deployment fails:

1. Check error logs (Railway/Vercel dashboards)
2. Verify all environment variables
3. Ensure GitHub repository is public
4. Test locally first: `cd backend && python main.py`
5. Check specifications in `/specs` folder for architecture details

---

**🎉 Phase III Deployment Complete!**

Your full-stack AI Todo chatbot is now live with MCP, OpenAI Agents SDK, and ChatKit integration.
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
