---
name: tag-management
description: Tag input in add form and tag-based filtering.
---

# Tag Management Skill

## Purpose
Enable flexible organization via tags.

## Process
1. In add-task-box: chip-style input (type → Enter → violet tag pill)
2. In task card: display tags as removable pills
3. In toolbar: tag filter dropdown (multi-select)
4. Click tag in card → filter to that tag

## Examples
Tag pill: bg-[#8B5CF6]/20 text-[#A78BFA] rounded-full px-3 py-1

## Guidelines
- Use obsidian-theme-skill
- Store as JSON array in DB
- Max 10 tags per task