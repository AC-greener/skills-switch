# SkillSync Design System

## Platform
macOS desktop app, targeting native macOS feel.
Sidebar + main content layout, similar to Finder or VS Code.

## Color palette
- Background primary: #1E1E1E (dark)
- Background secondary: #252526
- Sidebar: #2D2D2D
- Accent: #5B8AF5 (blue, for active states and CTAs)
- Success: #4CAF82
- Text primary: #E8E8E8
- Text secondary: #9D9D9D
- Border: #3A3A3A

## Typography
- Font: SF Pro (system font)
- Body: 13px regular
- Label: 12px secondary color
- Code/monospace: SF Mono 12px for skill content preview

## Layout rules
- Left sidebar: 240px fixed width
- Always use macOS-style toolbar at top
- Cards use 8px border radius
- Use tight spacing (8px/12px gaps) — this is a pro tool, not a consumer app

## Component patterns
- Skill items in sidebar list: icon + name + agent badge
- Status indicators: colored dot (green=synced, yellow=modified, red=conflict)
- Panels use subtle dividers, not heavy borders