---
name: completion-date
description: Date picker shown conditionally when task marked completed.
---

# Completion Date Skill

## Purpose
Track exact completion date for completed tasks.

## Process
1. In add/edit form: visible only when status = Completed
2. Modern dark date picker (react-datepicker or native)
3. Format: YYYY-MM-DD
4. Display in task card when present

## Examples
Picker theme: bg-[#1A1A1A] text-[#E0E0E0] border-[#8B5CF6]

## Guidelines
- Use obsidian-theme-skill
- Optional field
- Auto-fill today on complete toggle