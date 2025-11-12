---
name: visual-style
description: When you are tasked with creating a CLI visual interface, read these visual guidelines   
---


## Visual Style - Cyberpunk Terminal Theme

When creating command-line interfaces (CLI), always apply the following retro-futuristic hacker aesthetic:

### Color Scheme
- **Primary color:** Bright neon green (`#00FF41`, RGB: 0, 255, 65) - classic terminal phosphor glow
- **Secondary color:** Dark green (`#008F11`, RGB: 0, 143, 17) - for less emphasized elements
- **Background:** Black (`#000000`) or very dark gray (`#0D0208`) - deep void aesthetic
- **Accent colors:** Neon green (`#39FF14`) for highlights and important information
- **Text dimmed:** Dark green/gray (`#003B00`) for secondary text

### Typography & Characters
- Use monospace fonts exclusively (simulating old terminal displays)
- Incorporate these cyberpunk glyphs for decorative elements: `║ ═ ╔ ╗ ╚ ╝ ░ ▒ ▓ █ ▀ ▄ ▌ ▐`


### Animation Effects
- **Cascading character streams:** Vertical columns of random ASCII/Unicode characters falling down the screen (as startup/loading animation)
- **Text materialization:** Characters appearing one by one, as if being "compiled" in real-time
- **Glitch effects:** Occasional random character flickers in text (simulating data transmission)
- **Progressive reveal:** Text emerging from streams of code, solidifying into readable content
- **Pulse effects:** Subtle brightness variations in borders and key elements (like a terminal phosphor glow)

### UI Components (using Rich library)
- **Panels:** Green borders with shadow effects, titles in bright neon green
- **Progress bars:** Green bars with pulsing effect, use block characters as fill (█▓▒░)
- **Tables:** Green grid lines, alternating row shading with dark/darker green
- **Menus:** Arrow-key navigation (using `questionary` or `inquirer`), selected items in bright green with `►` indicator
- **Spinners:** Use cyberpunk-themed spinners (cascading characters, rotating symbols, data stream effects)
- **Status indicators:** `[●]` for active, `[○]` for inactive, in green shades

### Structural Elements
```
Example header:
╔═══════════════════════════════════════╗
║   ░▒▓ CYBERPUNK RECRUITMENT ▓▒░       ║
║  Agentic Hiring System v1.0           ║
╚═══════════════════════════════════════╝
```

### Startup Sequence
Every CLI app should start with:
1. Brief cascading character animation (1-2 seconds) - streams of falling ASCII
2. Logo/title materializing from the character streams
3. System initialization messages in neon green text
4. Ready state with pulsing cursor: `█`

### Interactive Elements
- **Input prompts:** `► ` (right-pointing triangle)
- **Success messages:** `✓` in bright green
- **Error messages:** `✗` in red (exception to green rule)
- **Loading states:** Animated sequences like `[▓▒░]` or `[█▓▒░]`
- **Waiting states:** Pulsing ellipsis `...` or rotating characters `|/-\`

### Implementation Notes
- Use Python's `rich` library for panels, colors, and formatting
- Use `questionary` or `inquirer` for menu navigation (arrow keys only)
- Add subtle delays (0.01-0.05s) between character prints for "typing" effect
- Keep animations smooth but not distracting from functionality
- Ensure all text remains readable - style should enhance, not hinder usability

