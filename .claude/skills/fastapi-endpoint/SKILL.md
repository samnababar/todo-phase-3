
### 5. `fastapi-endpoint-skill.md`
```markdown
---
name: fastapi-endpoint
description: Generates secure CRUD API endpoints matching hackathon spec.
---

# FastAPI Endpoint Skill

## Purpose
Implement all required REST endpoints with full security and validation.

## Process
1. Create routes exactly as PDF table:
   - GET/POST /api/{user_id}/tasks
   - GET/PUT/DELETE /api/{user_id}/tasks/{id}
   - PATCH /api/{user_id}/tasks/{id}/complete
2. Apply JWT dependency and user isolation.
3. Use Pydantic models for request/response.
4. Return proper HTTP status codes.

## Examples
POST /tasks → creates task with current_user.id as owner.

## Guidelines
- Use Depends() for current_user injection.
- Combine with jwt-security-skill and user-isolation-skill.