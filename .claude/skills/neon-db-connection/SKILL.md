
### 4. `neon-db-connection-skill.md`
```markdown
---
name: neon-db-connection
description: Sets up SQLModel engine connection to Neon Serverless PostgreSQL.
---

# Neon DB Connection Skill

## Purpose
Establish secure, persistent database connection for ObsidianList using Neon.tech.

## Process
1. Read DATABASE_URL from .env (Neon connection string).
2. Create SQLModel engine with async support.
3. Implement session maker for dependency injection.
4. Handle connection pooling and errors.

## Examples
```python
engine = create_engine(DATABASE_URL, echo=False, future=True)
SessionLocal = sessionmaker(bind=engine, class_=Session, expire_on_commit=False)