---
name: add-task-box
description: Modal or inline box for creating tasks with all advanced fields.
---

# Add Task Box Skill

## Purpose
Provide premium dark modal/form for adding tasks with full feature set.

## Process
1. Trigger: Floating violet "+" button or "Create Task" in header
2. Modal overlay with dark background
3. Fields:
   - Task Name (required)
   - Description (textarea)
   - Priority dropdown (Low/Medium/High)
   - Status dropdown (Pending/Completed/Marketing)
   - Completion Date (date picker – shown only if Completed)
   - Tags (chip input – add with Enter/Comma)
4. Buttons: "Add Task" (violet primary), "Cancel" (gray)

## Examples
Priority select: bg-[#1A1A1A] text-[#E0E0E0] border-[#8B5CF6]
Tags: violet pills with remove X

## Guidelines
- Use obsidian-theme-skill
- Conditional date field visibility
- Client-side validation
- Smooth open/close animation