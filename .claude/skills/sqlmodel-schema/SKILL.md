---
name: sqlmodel-schema
description: Defines User and Task models with proper relationships and fields.
---

# SQLModel Schema Skill

## Purpose
Create database schema supporting advanced task features and user ownership.

## Process
1. Define User model: id, username, hashed_password.
2. Define Task model:
   - id, title, description
   - completed: bool
   - priority: str ("low"/"medium"/"high")
   - completion_date: Optional[date]
   - tags: list[str] (JSON field)
   - user_id: ForeignKey → User
3. Create tables on startup.

## Examples
```python
class Task(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    description: str = ""
    completed: bool = False
    priority: str = "medium"
    completion_date: Optional[date] = None
    tags: list[str] = Field(default_factory=list, sa_column=Column(JSON))
    user_id: int = Field(foreign_key="user.id")