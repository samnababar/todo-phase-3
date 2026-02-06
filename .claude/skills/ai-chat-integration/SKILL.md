---
name: ai-chat-integration
description: Integrates AI assistant chat input into the dashboard UI.
---

# AI Chat Integration Skill

## Purpose
Add real-time chat box in dashboard for natural language task commands.

## Process
1. Render chat input component in Next.js dashboard.
2. On submit, send to backend AI endpoint.
3. Display streaming responses.
4. Apply dark theme (#8B5CF6 accents).

## Examples
Component: Floating chat bubble → opens modal with history.

## Guidelines
- Use obsidian-theme-skill for styling.
- Responsive on mobile.