
### 3. `user-isolation-skill.md`
```markdown
---
name: user-isolation
description: Ensures every database query filters tasks by authenticated user_id only.
---

# User Isolation Skill

## Purpose
Prevent cross-user data access – zero trust security for multi-user ObsidianList.

## Process
1. In every FastAPI route, use current_user from jwt-security-skill.
2. Filter all Task queries: WHERE user_id = current_user.id
3. Apply to GET, POST, PUT, DELETE, PATCH operations.
4. Never expose or accept user_id from client payload.

## Examples
```python
tasks = session.exec(select(Task).where(Task.user_id == current_user.id)).all()