---
name: task-list
description: Responsive card-based list of tasks with integrated filters.
---

# Task List Skill

## Purpose
Core display of all user tasks in beautiful dark cards.

## Process
1. Grid or flex layout of task cards
2. Above list: toolbar with search, filter dropdowns, sort
3. Integrate task-card-skill for each item
4. Infinite scroll or pagination
5. Skeleton loaders while fetching

## Examples
Container: grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6

## Guidelines
- Use obsidian-theme-skill
- Combine with task-search-skill, task-sort-skill, priority-filter-skill
- Empty state fallback