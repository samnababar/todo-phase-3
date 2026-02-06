---
name: openai-agents-assistant
description: Handles natural language task creation/update using OpenAI Agents SDK on backend.
---

# OpenAI Agents Assistant Skill

## Purpose
Enable AI-powered natural language interaction for adding/editing tasks in ObsidianList dashboard.

## Process
1. Parse user natural language input (e.g., "Add high priority meeting tomorrow").
2. Use OpenAI Agents SDK to extract task details (name, desc, priority, status, date, tags).
3. Call backend FastAPI endpoint with JWT to create/update task.
4. Return confirmation or suggestions.

## Examples
User: "Remind me to buy groceries high priority" → Create task with priority high, tags ["groceries"].

## Guidelines
- Secure with JWT.
- Integrate with dashboard chat input.
- Fallback to manual form if ambiguous.