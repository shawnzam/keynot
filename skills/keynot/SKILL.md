---
name: keynot
description: Create polished, self-contained HTML slide presentations — no PowerPoint or Keynote needed. Use this skill whenever the user asks for a presentation, slide deck, slides, or one-pager that will be shown in a browser. Also trigger when the user says things like "make it a deck", "turn this into slides", "build a presentation from this content", or "I need something to present". Prefer this over PPTX when the user has not explicitly asked for a .pptx file — HTML decks are more flexible, more visually rich, and easier to iterate on. Always use this skill when the user provides a brand style guide (PDF, URL, or description) alongside a presentation request. Trigger even if the user only says "make me a deck" with no further detail — start with content questions.
---
 
# Keynot
 
A skill for building browser-based slide presentations as single, self-contained HTML files. Keyboard navigation, swipe, fullscreen, animated reveals, embedded images, brand-accurate design. No external dependencies required at runtime.
 
---
 
## When to Use This Skill
 
- User asks for a "presentation", "deck", "slides", "one-pager to present"
- User has a brand style guide and wants slides that match it
- User wants to iterate on slides quickly in the browser
- User wants a file they can open anywhere without software
 
**Prefer HTML over PPTX unless:**
- User explicitly asks for `.pptx`
- User needs to edit slides in PowerPoint
- User needs to share with someone who must edit in Office
 
---
 
## Step 1: Extract the Brand Guide
 
If the user provides a brand PDF, URL, or describes their brand, extract these before writing any code. Be explicit and precise.
 
### What to Extract
 
**Colors**
- Primary (dominant background/text color)
- Accent (calls to action, rules, highlights)
- Neutrals (background, cream, gray)
- Document exact hex codes — never approximate
 
**Typography**
- Serif font (display headings, pull quotes)
- Sans-serif font (body, labels, UI)
- If proprietary fonts are specified (e.g., ITC Stone Serif, Acumin), find the closest Google Fonts equivalent
- Document the pairing explicitly
 
**Layout Language**
- Dominant color usage ratio (e.g., "blue should always dominate, red as accent only")
- Typical layout patterns visible in the guide (split panels, rules, dividers, margins)
- Tone: formal/editorial, warm/approachable, minimal/dense
 
**Logo and Mark Rules**
- Whether the logo/wordmark may be reproduced
- Clear space requirements
- Approved color variants (full color, white, black)
- Note: for institutional brands (universities, corporations), do NOT reproduce trademarked logos — use a text-based wordmark instead
 
### Example Brand Record
 
Once extracted, record the brand parameters in a compact block at the top of the deck's `<style>` so they're easy to audit and swap:
 
```
Primary Colors:
  Primary:  #HEX  (dominant — backgrounds, headings)
  Accent:   #HEX  (rules, CTAs, status — used sparingly)
  Cream:    #HEX  (light slide backgrounds)
 
Usage Rule: One color dominates. Accent appears sparingly —
  rules, call-to-action bars, status indicators.
  Never use the accent color as a full background.
 
Typography:
  Serif:      <Brand serif>      → <closest Google Font>
  Sans-serif: <Brand sans-serif> → <closest Google Font>
 
Layout Language:
  - Which colors own which regions (dark panels vs. light panels)
  - Any spine bars, rules, or dividers the brand uses
  - Layout motifs (split panels, stat columns, card grids)
  - Tone: editorial / warm / minimal / dense
 
Logo Rule: For institutional or trademarked brands, do NOT
  reproduce the logo. Use a text-based wordmark instead:
  <div class="wordmark">
    <div class="w-line-1">Organization</div>
    <div class="w-line-2">Subtitle or parent entity</div>
  </div>
```
 
---
 
## Step 2: Deck Architecture
 
Every deck is a single HTML file. All assets (fonts via CDN, images via base64) are embedded.
 
### Shell Structure
 
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Google Fonts import -->
  <!-- All CSS in <style> tag -->
</head>
<body>
  <div class="deck" id="deck">
    <div class="slide active" id="s1">...</div>
    <div class="slide" id="s2">...</div>
    <!-- more slides -->
  </div>
  <!-- Navigation -->
  <div class="nav">
    <button id="prev">&#8592;</button>
    <div class="dots" id="dots"></div>
    <button id="next">&#8594;</button>
    <span class="counter" id="counter">1 / N</span>
    <button id="fsBtn" onclick="toggleFS()">&#x26F6;</button>
  </div>
  <script><!-- All JS inline --></script>
</body>
</html>
```
 
### Core CSS Patterns
 
```css
:root {
  /* Always use CSS variables for brand colors — swap these per brand */
  --primary: #1a1a2e;   /* dominant: dark panels, headings */
  --accent:  #e94560;   /* sparingly: rules, CTAs, status */
  --white:   #ffffff;
  --cream:   #f7f4ee;   /* light slide backgrounds */
  --serif:   'Cormorant Garamond', Georgia, serif;
  --sans:    'DM Sans', system-ui, sans-serif;
}
 
/* Deck shell */
html, body { width: 100%; height: 100%; overflow: hidden; background: #000; }
.deck { width: 100vw; height: 100vh; position: relative; }
 
/* Slides: opacity transition, stacked absolutely */
.slide {
  position: absolute; inset: 0;
  opacity: 0; pointer-events: none;
  transition: opacity .55s cubic-bezier(.4,0,.2,1);
}
.slide.active { opacity: 1; pointer-events: all; }
 
/* Staggered reveal animation on active slide children */
.slide.active .reveal { animation: fadeUp .6s both; }
.slide.active .reveal:nth-child(2) { animation-delay: .1s; }
.slide.active .reveal:nth-child(3) { animation-delay: .2s; }
/* ...continue for as many children as needed */
 
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(22px); }
  to   { opacity: 1; transform: translateY(0); }
}
```
 
### Navigation CSS
 
```css
.nav {
  position: fixed; bottom: 28px; left: 50%;
  transform: translateX(-50%);
  z-index: 999;
  display: flex; align-items: center; gap: 14px;
  background: rgba(26,26,46,.75);  /* tint from --primary */
  backdrop-filter: blur(10px);
  padding: 9px 20px; border-radius: 40px;
  border: 1px solid rgba(255,255,255,.14);
  opacity: 1;
  transition: opacity .4s ease;   /* required for auto-hide */
}
.nav-hidden { opacity: 0; pointer-events: none; }  /* auto-hide state */
.nav button {
  background: none; border: none; cursor: pointer;
  color: rgba(255,255,255,.65); font-size: 16px;
  width: 30px; height: 30px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%;
  transition: color .2s, background .2s;
}
.nav button:hover { color: #fff; background: rgba(255,255,255,.12); }
.dot {
  width: 7px; height: 7px; border-radius: 50%;
  background: rgba(255,255,255,.3); cursor: pointer;
  transition: background .2s, transform .2s;
}
.dot.active { background: var(--accent); transform: scale(1.35); }
```
 
### JavaScript (complete — do not modify arithmetic)
 
```javascript
// Fullscreen
function toggleFS() {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen().catch(() => {});
    document.getElementById('fsBtn').innerHTML = '&#x2715;';
  } else {
    document.exitFullscreen();
    document.getElementById('fsBtn').innerHTML = '&#x26F6;';
  }
}
document.addEventListener('fullscreenchange', () => {
  if (!document.fullscreenElement)
    document.getElementById('fsBtn').innerHTML = '&#x26F6;';
});
 
// Navigation
const slides = document.querySelectorAll('.slide');  // REQUIRED — do not omit
const dotsEl = document.getElementById('dots');
const counter = document.getElementById('counter');
let cur = 0;
 
slides.forEach((_, i) => {
  const d = document.createElement('div');
  d.className = 'dot' + (i === 0 ? ' active' : '');
  d.addEventListener('click', () => go(i));
  dotsEl.appendChild(d);
});
 
function go(n) {
  slides[cur].classList.remove('active');
  dotsEl.children[cur].classList.remove('active');
  cur = (n + slides.length) % slides.length;
  slides[cur].classList.add('active');
  dotsEl.children[cur].classList.add('active');
  counter.textContent = `${cur + 1} / ${slides.length}`;
}
 
document.getElementById('next').addEventListener('click', () => go(cur + 1));
document.getElementById('prev').addEventListener('click', () => go(cur - 1));
 
document.addEventListener('keydown', e => {
  if (e.key === 'ArrowRight' || e.key === 'ArrowDown' || e.key === ' ') go(cur + 1);
  if (e.key === 'ArrowLeft'  || e.key === 'ArrowUp')                     go(cur - 1);
  if (e.key === 'f' || e.key === 'F') toggleFS();
});
 
// Auto-hide nav after 1.8s of inactivity; reappear on any interaction
const nav = document.querySelector('.nav');
let hideTimer;
function showNav() {
  nav.classList.remove('nav-hidden');
  clearTimeout(hideTimer);
  hideTimer = setTimeout(() => nav.classList.add('nav-hidden'), 1800);
}
document.addEventListener('mousemove', showNav);
document.addEventListener('touchstart', showNav, { passive: true });
document.addEventListener('keydown', showNav);
showNav(); // start timer on load
 
// Swipe
let tx = 0;
document.addEventListener('touchstart', e => { tx = e.touches[0].clientX; }, { passive: true });
document.addEventListener('touchend', e => {
  const dx = e.changedTouches[0].clientX - tx;
  if (Math.abs(dx) > 50) go(dx < 0 ? cur + 1 : cur - 1);
}, { passive: true });
```
 
---
 
## Step 3: Layout Patterns
 
Use these as building blocks. Mix and match per slide.
 
### Split Panel (most versatile)
Dark panel left (brand primary color), light panel right (white or cream). Diagonal slice or hard edge between them.
 
```html
<div class="slide" id="sN" style="display:flex;">
  <div class="left-panel"><!-- headline, statement, quote --></div>
  <div class="right-panel"><!-- list, stats, details --></div>
</div>
```
 
```css
.left-panel {
  width: 50%; background: var(--primary);
  display: flex; flex-direction: column; justify-content: center;
  padding: 72px 64px; position: relative; overflow: hidden;
}
.right-panel {
  flex: 1;
  display: flex; flex-direction: column; justify-content: center;
  padding: 72px 56px 72px 64px; background: var(--cream);
}
```
 
### Stat Column (credentials, metrics)
Right-side column of large serif numbers/words with small uppercase labels. Works over photo backgrounds.
 
```html
<div class="stats-col">
  <div class="stat">
    <div class="stat-n">13+</div>
    <div class="stat-l">Years at institution</div>
  </div>
  <!-- divider between stats: border-top: 1px solid rgba(255,255,255,.09) -->
</div>
```
 
### Value Cards (principles, pillars)
Left colored bar accent, title + description. One bar color per value for visual differentiation.
 
```html
<div class="value-card">
  <div class="value-bar" style="background:#2e7d32"></div>
  <div>
    <div class="value-title">Principle Name</div>
    <div class="value-text">Description of the principle.</div>
  </div>
</div>
```
 
### Approach Rows (process, steps)
Large ghost number, vertical rule, then body content. Good for 3-step processes.
 
```html
<div class="approach">
  <div class="approach-num">01</div>
  <div class="approach-body">
    <div class="approach-title">Step Title</div>
    <div class="approach-text">Detail text.</div>
    <div class="approach-tags"><span>Tag 1</span><span>Tag 2</span></div>
  </div>
</div>
```
 
### Tool/Status Cards
Name + status badge + description. Status badge variants: live (green), in-progress (amber), planned (neutral).
 
```html
<div class="tool-card">
  <div class="tool-body">
    <div class="tool-name">Tool Name</div>
    <span class="status status-live">Live</span>
    <div class="tool-desc">What it does and who it's for.</div>
  </div>
</div>
```
 
### Photo Background Panel
Embed image as base64, apply gradient overlay so text stays readable.
 
```html
<div class="photo-panel" style="
  position:absolute; top:0; right:0; bottom:0; width:42%;
  background: url('data:image/png;base64,...') center/cover no-repeat;
  z-index:1;
">
  <div style="position:absolute;inset:0;
    background:linear-gradient(to right, var(--primary) 0%, rgba(0,0,0,.55) 40%, rgba(0,0,0,.1) 100%);
  "></div>
</div>
```
 
To embed an image as base64:
```python
import base64
data = 'data:image/png;base64,' + base64.b64encode(open('image.png','rb').read()).decode()
```
 
---
 
## Step 4: Content Slide Sequence
 
A strong 5-slide deck structure for an institutional introduction:
 
| Slide | Purpose | Layout |
|-------|---------|--------|
| 1 | Who you are + elevator pitch | Full bleed dark + photo + stat column |
| 2 | Values / principles | Split panel + value cards |
| 3 | Approach / framework | Light background + approach rows |
| 4 | Tools / enablement | Dark background + two columns |
| 5 | Close + discussion opener | Split panel + photo background |
 
Adapt freely. The key is each slide has ONE job and ONE dominant visual idea.
 
---
 
## Step 5: Typography Rules
 
```css
/* Display heading — large, light weight, letter-spaced */
.display {
  font-family: var(--serif);
  font-size: clamp(52px, 7vw, 88px);
  font-weight: 300; line-height: .95;
  letter-spacing: -.01em;
}
 
/* Section heading */
.heading {
  font-family: var(--serif);
  font-size: clamp(32px, 4vw, 50px);
  font-weight: 400; line-height: 1.1;
}
 
/* Eyebrow label */
.label {
  font-family: var(--sans);
  font-size: 10px; font-weight: 600;
  letter-spacing: .2em; text-transform: uppercase;
}
 
/* Body */
.body {
  font-family: var(--sans);
  font-size: 13px; font-weight: 300; line-height: 1.65;
}
 
/* Use clamp() for all font sizes to scale with viewport */
```
 
---
 
## Critical Pitfalls
 
### NEVER use blanket string/regex replacement on files containing JavaScript
 
This will corrupt arithmetic operators. Example of what goes wrong:
 
```python
# DANGEROUS — corrupts JS
content = content.replace(' - ', ', ')
# cur - 1  becomes  cur, 1     (prev button breaks)
# clientX - tx  becomes  clientX, tx  (swipe breaks)
```
 
**Safe approach:** Use targeted `str_replace` with unique, specific strings. Always view the JS section after any automated replacement to verify arithmetic operators are intact.
 
### Always declare `const slides` before using it
 
If inserting a new function before the navigation block, check that `const slides = document.querySelectorAll('.slide')` is still present and comes before `slides.forEach(...)`.
 
### `str_replace` requires unique strings
 
Before using str_replace, grep for the target string to confirm it appears exactly once. If it appears multiple times, add more context lines to make it unique.
 
### Base64 images make files large but self-contained
 
A single high-res PNG can add 150-200KB to the file. This is fine for a presentation deck. Use `cover` sizing and `center center` positioning. Always add a gradient overlay div for text legibility.
 
### Emoji in content
 
Avoid leading emoji in list items and card titles — they read as AI-generated and date quickly. If the user asks to remove emoji, use Python unicode replacement rather than sed:
 
```python
import re
content = re.sub(r'[\U00002600-\U0001FAFF]+', '', content)
content = content.replace('\ufe0f', '')  # variation selector
```
 
---
 
## Iterative Editing Workflow
 
When making changes after initial build:
 
1. **View the relevant section first** — `view` tool with line range
2. **Use `str_replace`** for surgical edits — never rewrite the whole file for small changes
3. **Verify JS after any automated text processing** — check lines with `cur`, `dx`, `+`, `-` operators
4. **Present fresh file after changes** — user may be viewing cached version
 
When the user says things like "slide 3: change X" — make ONLY that change. Resist the urge to improve adjacent content.
 
---
 
## Codex / API Usage Notes
 
This skill is compatible with Claude API (Codex) usage. Key considerations:
 
- The complete JS block (navigation + fullscreen) should be treated as a unit — include it verbatim and do not modify arithmetic
- When generating via API, stream the file to disk and open in browser for preview
- The base64 embedding approach works identically in API context
- CSS variables (`--primary`, `--accent`, etc.) make theming programmatic — pass brand colors as parameters and substitute into the template
 
---
 
## Quick Reference: Palette Slots
 
Every deck fills these slots. Populate from the brand guide, then theme programmatically via CSS variables.
 
```
--primary     Dominant backgrounds and dark panels
--accent      Rules, spine bars, CTAs, status dots, hover states
--cream       Light slide backgrounds (warmer than pure white)
--stone       Dividers, subtle borders (one tint above cream)
--ink         Body text on light backgrounds (softer than black)
--muted       Labels, metadata, secondary text
--go          Positive / live / green status
--warn        Review / caution / amber status
--alt         Fifth accent for value/pillar differentiation
```
 
Fill with real hex codes at the top of `<style>` and reference everywhere via `var(--name)`.