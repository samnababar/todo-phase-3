---
name: obsidian-theme
description: Core dark theme generator for ObsidianList with pure black base and violet accents.
---

# Obsidian Theme Skill

## Purpose
Define and apply consistent premium dark aesthetic across all pages and components.

## Process
1. Primary background: #000000 (pure black)
2. Card/surface: #1A1A1A
3. Primary accent: #8B5CF6 (violet buttons/links)
4. Hover/active: #A78BFA
5. Text body: #E0E0E0
6. Headings: #FFFFFF
7. Borders: #262626
8. Generate Tailwind config and utility classes
9. Add subtle glow/shadow on interactive elements

## Examples
Button: bg-[#8B5CF6] hover:bg-[#A78BFA] text-white rounded-lg shadow-lg shadow-[#8B5CF6]/20
Hero text: bg-gradient-to-r from-[#8B5CF6] to-[#A78BFA] bg-clip-text text-transparent

## Guidelines
- OLED-friendly black
- High contrast for accessibility
- Smooth transitions (duration-300)
- Use with all UI skills