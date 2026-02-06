---
name: accessibility
description: Ensures high contrast and keyboard navigation in dark theme.
---

# Accessibility Skill

## Purpose
Make ObsidianList WCAG-compliant for dark mode users.

## Process
1. Check contrast ratios (violet accents on black > 4.5:1).
2. Add ARIA labels to buttons/forms.
3. Ensure keyboard nav (tab order, focus styles).
4. Test screen reader compatibility.

## Examples
Button: `<button aria-label="Add Task">` + visible focus ring (#A78BFA).

## Guidelines
- Use with obsidian-theme-skill.
- Prioritize for dashboard interactions.