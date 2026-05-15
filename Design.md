# Design Direction

## Brand in one sentence
Dark, precise, technical — the aesthetic of a SOC analyst who writes real detection systems, not a generic "cybersecurity" template.

## Mood
- Deep space / naval ops — not hacker green, not neon purple
- Calm authority, not aggressive
- Glass and light over heavy solid blocks
- Intelligence over decoration

## Color palette

| Role | Token | Value |
|------|-------|-------|
| Page background | `--base` | `#0c1e3a` — deep navy |
| Card surface | `--surface` | `rgba(14,34,68,0.65)` — translucent navy |
| Primary accent | `--sky` | `#5bc8ff` — electric blue |
| Highlight accent | `--ice` | `#e4f9ff` — near-white blue |
| Secondary accent | `--steel` | `#96d0ff` — soft steel blue |
| Body text | `--text` | `#8fc8e8` — muted blue-grey |
| Secondary text | `--text-light` | `#c8e6f8` |
| High emphasis | `--text-bright` | `#edf8ff` — near white |
| Borders | `--border` | `rgba(100,200,255,0.14)` |

**Never use:** pure white, pure black, saturated red, green terminal palette, purple.

## Typography

| Use | Font | Weight | Notes |
|-----|------|--------|-------|
| Page titles, hero name | Space Grotesk | 700 | Negative letter-spacing (-0.03em) |
| Section headings | Space Grotesk | 600–700 | |
| Body text, descriptions | Inter | 400 | Line-height 1.6–1.7 |
| Labels, tags, nav links, code | Fira Code | 400–500 | Uppercase, tracked (+0.06–0.15em) |
| Numbers / data | Fira Code | 400 | Monospace alignment |

## Spacing system
Use Tailwind's default scale. Prefer generous whitespace — sections need 80–120px vertical padding. Cards get 24–32px internal padding.

## Component feel

### Cards
- Glass effect: `backdrop-filter: blur(20px)` + `rgba(12,30,60,0.6)` background
- Border: `1px solid rgba(79,195,247,0.12)` at rest → `0.3` on hover
- Hover: `translateY(-6px)` + soft sky glow shadow
- Border-radius: 16px (cards), 8–12px (smaller elements)

### Buttons
- Primary: gradient fill `rgba(79,195,247,0.18→0.28)` + pulse-glow animation
- Outline: transparent + sky border, fills lightly on hover
- Both use Fira Code, 0.75rem, tracked

### Tags / Pills
- Fira Code, 0.65rem, all-caps or lowercase — consistent per page
- `rgba(79,195,247,0.07)` background, `rgba(79,195,247,0.2)` border
- Border-radius: 100px (pill)

### Section headings
Pattern: `[number].  Title  ————————`
- Number in Fira Code + sky color
- Title in Space Grotesk 700
- Trailing line fades right

## Animation philosophy
- **Reveal on scroll:** subtle — 8px translateY + fade, 0.4s. Not dramatic.
- **Stagger groups:** 50ms between items — feels responsive, not mechanical
- **Ambient motion:** nebula orbs drift slowly (38–55s cycles). Barely perceptible.
- **Canvas particles:** low opacity (0.5), connected graph of dots — technical feel
- **Hover:** quick (0.3s), purposeful — lift + glow, not scale
- **No bounce, no spring physics on UI elements** — reserved for occasional hero moments

## Background layers (bottom to top)
1. `#0c1e3a` base
2. Radial gradient nebula (CSS, fixed)
3. Ambient orbs (CSS animation, fixed, blur 100px)
4. Canvas particle graph (JS, fixed, opacity 0.5)
5. Page content (position relative, z-index 1+)

## What to avoid
- Bright, saturated colors anywhere except tiny accents
- Hard edges / heavy shadows (use glow instead)
- Text walls without visual breaks
- Cluttered layouts — each section should breathe
- Animations that fight for attention simultaneously
- Comic/playful iconography — use Heroicons or Lucide (line style, consistent stroke)

## Inspiration references
- Linear.app (dark, precise, technical)
- Vercel dashboard (glassmorphism, clean data)
- Josh W Comeau's site (personality + depth)
- Brittany Chiang portfolio (sidebar layout, section numbering — direct influence)
