# Portfolio — Claude Instructions

## Project overview
Static HTML portfolio for Abdelkrim Zouaki, SOC Analyst based in Montreal. Deployed on GitHub Pages at https://karim871.github.io/Portfolio/.

## Stack — critical constraints
- **No npm, no build step, no framework.** Plain HTML files + CDN scripts only.
- Tailwind CSS via CDN (`<script src="https://cdn.tailwindcss.com">`) with inline `tailwind.config`
- GSAP 3.12.5 + ScrollTrigger via CDN
- Lenis 1.0.42 via CDN (smooth scroll)
- All JS lives in `<script>` tags at bottom of each HTML file (or `stylesheets/script.js` for trivial one-liners)
- CSS lives in `<style>` tags in `<head>` for page-specific styles; shared base in `stylesheets/style.css`

## File structure
```
Portfolio/
├── index.html              # Main page (hero, projects, skills, certs, contact)
├── secondarypages/         # All sub-pages (blog posts, project deep-dives)
├── stylesheets/
│   ├── style.css           # Legacy base styles (mostly replaced by inline Tailwind)
│   └── script.js           # Tiny shared JS (name-tag toggle)
├── images/                 # All images and screenshots
├── brand-assets/           # Logos, fonts, exported brand files
│   ├── fonts/
│   ├── logos/
│   └── inspiration/
└── Abdelkrim_Zouaki_CV.pdf
```

## Design tokens — always use these, never hardcode raw colors
```css
--base:         #0c1e3a       /* page background */
--surface:      rgba(14,34,68,0.65) /* glass card background */
--border:       rgba(100,200,255,0.14)
--border-hover: rgba(100,200,255,0.45)
--text:         #8fc8e8       /* default body text */
--text-light:   #c8e6f8       /* secondary text */
--text-bright:  #edf8ff       /* headings, high emphasis */
--sky:          #5bc8ff       /* primary accent */
--ice:          #e4f9ff       /* highlight accent */
--steel:        #96d0ff       /* secondary accent */
--glow-sky:     rgba(91,200,255,0.22)
--glow-ice:     rgba(228,249,255,0.12)
```

Tailwind color aliases (from inline config):
- `bg-base`, `text-sky`, `text-ice`, `text-steel`, `bg-surface`

## Typography — never use other fonts
- **Display / headings:** `Space Grotesk` — weights 300, 400, 500, 600, 700
- **Body / prose:** `Inter` — weights 300, 400, 500, 600
- **Code / mono / labels / tags:** `Fira Code` — weights 400, 500

## Animation rules
- Reveal on scroll: `.reveal` class + `IntersectionObserver` or ScrollTrigger — `opacity: 0 → 1`, `translateY(8px → 0)`, duration 0.4s, cubic-bezier(0.22,1,0.36,1)
- Stagger delay classes: `.reveal-delay-1` (0.05s), `.reveal-delay-2` (0.1s), `.reveal-delay-3` (0.15s)
- Hover lifts: `translateY(-6px)` on project cards, `translateY(-2px)` on skill cards
- Never use `animation-duration` faster than 0.3s for UI transitions
- Lenis handles smooth scroll — do NOT set `scroll-behavior: smooth` on `html`

## Reusable component patterns
```html
<!-- Glass card -->
<div class="glass p-6 rounded-2xl">...</div>

<!-- Section heading -->
<h2 class="section-heading">
  <span class="section-num">01.</span>
  Section Title
  <span class="section-line"></span>
</h2>

<!-- Tag pill -->
<span class="project-tag">SIEM</span>

<!-- Primary button -->
<button class="btn-primary">Label →</button>

<!-- Outline button -->
<button class="btn-outline">Label</button>

<!-- Nav link (sidebar) -->
<a href="#section" class="nav-link">label</a>
```

## Adding a new secondary page
1. Copy an existing page in `secondarypages/` as a template
2. Keep the same `<head>` (fonts, GSAP, Lenis, Tailwind CDN + config)
3. Keep the same CSS variables block in `<style>`
4. Keep cursor glow, sidebar nav, and Lenis init in `<script>`
5. Link back to `../index.html` in the logo

## Screenshot loop (two-pass iteration)
The puppeteer MCP is configured in `.claude/settings.json`. When it loads:
1. Start a local server: `! python -m http.server 3000` (in Claude Code terminal via `!`)
2. Ask Claude to take a screenshot of `http://localhost:3000`
3. Claude will see the site visually, identify issues, edit the HTML/CSS, then screenshot again
4. This removes the guessing — iterate until it looks right visually

If puppeteer MCP doesn't connect, run `! npx @modelcontextprotocol/server-puppeteer` first to install it.

## Deployment
GitHub Pages — push to `main` branch, site auto-deploys. No build step needed.

## Tone / content voice
Professional but human. Not corporate. Cybersecurity practitioner who builds real systems. Avoid buzzwords ("passionate", "results-driven"). Use specific technical details.
