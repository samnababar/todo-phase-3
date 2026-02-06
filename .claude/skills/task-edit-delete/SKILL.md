---
name: task-edit-delete
description: Inline edit and delete actions on task cards.
---

# Task Edit Delete Skill

## Purpose
Allow quick modification or removal of tasks.

## Process
1. Icons visible on card hover
2. Edit: opens add-task-box pre-filled
3. Delete: confirmation modal (violet "Confirm" button)
4. Success toast notification

## Examples
Icons: opacity-0 group-hover:opacity-100 transition-opacity text-[#A78BFA] hover:text-[#8B5CF6]

## Guidelines
- Use obsidian-theme-skill
- Prevent accidental delete
- Optimistic UI updates