---
name: better-auth
description: Handles signup and login using Better Auth integrated with Neon PostgreSQL via SQLModel.
---

# Better Auth Skill

## Purpose
Implement secure user authentication for ObsidianList using Better Auth library with persistent storage in Neon DB.

## Process
1. Install and configure Better Auth in Next.js frontend with JWT plugin enabled.
2. Set shared BETTER_AUTH_SECRET in .env (same for frontend and backend).
3. Create signup and login pages/routes that store user data in Neon via SQLModel.
4. On successful auth, issue JWT and store in httpOnly cookie.
5. Provide protected route wrapper for dashboard.

## Examples
Signup form → POST /api/auth/sign-up → creates User row in DB → returns JWT cookie.
Login → verifies password → returns JWT.

## Guidelines
- Use SQLModel User model with hashed_password.
- Ensure password hashing with Better Auth defaults.
- Integrate with neon-db-connection-skill for DB access.
- Dark theme styling on auth pages.