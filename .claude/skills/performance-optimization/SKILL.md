---
name: performance-optimization
description: Optimizes loading and responsiveness.
---

# Performance Optimization Skill

## Purpose
Make ObsidianList fast and smooth (lazy loading, code splitting).

## Process
1. Lazy load task list components.
2. Use Next.js dynamic imports.
3. Optimize images/queries.
4. Add skeleton loaders for tasks.

## Examples
`dynamic(() => import('./TaskList'))` for dashboard.

## Guidelines
- Target <2s load time.
- Mobile-first responsive.
- Integrate with task-list-skill.