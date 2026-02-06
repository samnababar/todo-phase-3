---
name: task-suggestion
description: Generates AI-powered task recommendations based on user history.
---

# Task Suggestion Skill

## Purpose
Suggest new tasks intelligently (e.g., recurring or based on tags).

## Process
1. Analyze user's existing tasks (via DB query).
2. Use OpenAI to generate 3-5 relevant suggestions.
3. Display in dashboard "Suggested Tasks" section.

## Examples
User has "Gym" tag weekly → Suggest "Gym session this Friday?"

## Guidelines
- Privacy: Only use user's own data.
- Optional accept button to add suggestion.