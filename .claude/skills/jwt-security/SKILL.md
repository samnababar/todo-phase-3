---
name: jwt-security
description: Implements JWT verification middleware for FastAPI backend.
---

# JWT Security Skill

## Purpose
Secure all API routes by verifying JWT tokens and extracting authenticated user_id.

## Process
1. Create FastAPI dependency that reads Authorization: Bearer <token> header.
2. Verify token signature using shared BETTER_AUTH_SECRET.
3. Decode payload to extract user_id.
4. Raise 401 if invalid/expired/missing.
5. Inject current_user into route dependencies.

## Examples
```python
async def get_current_user(token: str = Depends(oauth2_scheme)):
    payload = jwt.decode(token, SECRET, algorithms=["HS256"])
    return payload.get("sub")  # user_id