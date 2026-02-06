---
name: task-stats
description: Dashboard counters for pending, completed, and high-priority tasks.
---

# Task Stats Skill

## Purpose
Display real-time productivity overview at top of dashboard.

## Process
1. Three dark stat cards in row/grid
2. Card 1: "Pending" – count + violet icon
3. Card 2: "Completed" – count + subtle glow
4. Card 3: "High Priority" – count + warning accent
5. Hover: slight lift + violet shadow

## Examples
Stat card: bg-[#1A1A1A] rounded-xl p-6 text-center shadow-lg hover:shadow-[#8B5CF6]/30

## Guidelines
- Use obsidian-theme-skill
- Update live via state/store
- Responsive stack on mobile