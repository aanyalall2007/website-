---
name: cool-interactions
description: Add specific high-quality interactive UI effects inspired by top design portfolios (sundays.rsvp, Jackie Zhang, Joshua Wolk, etc). Effects include silver/chrome metallic hover, magnetic buttons, custom spring cursor, image-follow-cursor, text scramble, marquee ticker, card 3D tilt, stagger reveal, noise grain overlay, link underline draw, split-text animation, scroll progress bar, and more. Use when the user wants to make something feel interactive, alive, or "cool" — or when they name a specific effect like "silver hover", "magnetic button", "cursor trailer", "text scramble", etc.
---

You are implementing a specific high-quality interactive effect. The catalog below defines each effect precisely — its name, the reference it comes from, and its exact implementation. Pick the best match for what the user asked for, or let them choose from the catalog if unclear.

---

## Aanya's Portfolio — Known Context

When working on `aanya-portfolio-v14.html` or `sidequests.html`, these facts are already established:

**CSS variables (`:root`):**
- `--cream: #f4efe4` — background
- `--ink: #0d0d0d` — primary text
- `--gold: #8a7355` — accent

**Typography:** `EB Garamond` serif only. `text-transform: lowercase` is set globally on `body` — text scramble must account for this (the scrambled chars will auto-lowercase, which is fine).

**Custom cursor already exists:**
- `<div id="cursor">` — 9px white circle, `mix-blend-mode: difference`, `z-index: 9999`, snaps directly to mouse position
- `cursor: none` is already set globally
- **For Effect 03 (Spring Cursor Trailer):** Do NOT add a new element. Instead, upgrade `#cursor` by replacing the snap-to-position JS with the lerp loop. The element is already positioned correctly.

**Z-index stack:**
- `9999` — `#cursor`
- `300` — dropdowns / connect popup
- `200` — nav
- `10` — split line
- New overlays (grain, spotlight, progress bar) should use `9000`–`9998` to stay below the cursor.

**Layout:** Split-panel — `.craft-side` (left, cream bg) and `.work-side` (right, dark bg). Nav links on the left use `color: var(--ink)` and on the right use `rgba(255,255,255,...)`. Silver hover on right-side nav links should start from the white rgba value, not from `--ink`.

**What NOT to touch:**
- Don't change `cursor: none` — it's intentional
- Don't add `font-family` or `text-transform` to effect CSS — it'll conflict with global rules
- Don't use `z-index` values above 9999

---

## Before Writing Code

1. **Read the existing file** — understand the markup and CSS already present.
2. **Check for existing dependencies** — if GSAP/anime.js/Framer Motion is already loaded, use it. Otherwise use vanilla JS + CSS only unless a library is clearly the right tool.
3. **Ask one question if the target is ambiguous** — "Which element should this apply to?" Do not ask anything else.
4. **Pick the effect from the catalog below.** If the user didn't name one, recommend the best fit and show the catalog entry name so they can redirect.
5. **Implement it.** Drop into the actual file. Don't scaffold new files unless unavoidable.

---

## Effect Catalog

### 01 — SILVER CHROME HOVER
**Inspired by:** sundays.rsvp nav links, invite-page aesthetics
**What it does:** Text turns silver/metallic on hover with a shimmer sweep across it.
**How to implement:**
```css
.silver-hover {
  position: relative;
  background: linear-gradient(
    110deg,
    #888 0%,
    #fff 30%,
    #c0c0c0 50%,
    #fff 70%,
    #888 100%
  );
  background-size: 250% auto;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  color: transparent;
  background-position: 200% center;
  transition: background-position 0s;
}
.silver-hover:hover {
  animation: silver-shimmer 0.6s ease forwards;
}
@keyframes silver-shimmer {
  0%   { background-position: 200% center; }
  100% { background-position: -50% center; }
}
```
**Notes:** Works on `<a>`, `<h1>`–`<h6>`, `<span>`. For a more subtle version, set start color to the existing text color and only reveal silver on hover. Can also apply to the full element at all times (not just hover) for a permanently metallic look — just set `background-position: center` and animate on hover.

---

### 02 — MAGNETIC BUTTON
**Inspired by:** Portfolio CTAs (Josh Wolk, Belle Duffner, common in award-winning portfoios)
**What it does:** Button slightly follows the cursor when hovered, snaps back on leave.
**How to implement (vanilla JS):**
```html
<button class="magnetic-btn" data-strength="0.3">Let's talk</button>
```
```js
document.querySelectorAll('.magnetic-btn').forEach(btn => {
  const strength = parseFloat(btn.dataset.strength) || 0.3;
  btn.addEventListener('mousemove', e => {
    const rect = btn.getBoundingClientRect();
    const cx = rect.left + rect.width / 2;
    const cy = rect.top + rect.height / 2;
    const dx = (e.clientX - cx) * strength;
    const dy = (e.clientY - cy) * strength;
    btn.style.transform = `translate(${dx}px, ${dy}px)`;
  });
  btn.addEventListener('mouseleave', () => {
    btn.style.transform = 'translate(0,0)';
    btn.style.transition = 'transform 0.4s cubic-bezier(0.23, 1, 0.32, 1)';
    setTimeout(() => btn.style.transition = '', 400);
  });
  btn.addEventListener('mouseenter', () => {
    btn.style.transition = 'transform 0.1s ease';
  });
});
```
**Notes:** `data-strength` controls pull intensity (0.2 = subtle, 0.5 = strong). Wrap the button's inner text in a `<span>` and apply a stronger translate to the text than the button itself for a two-layer depth effect.

---

### 03 — SPRING CURSOR TRAILER
**Inspired by:** Common in portfolio sites (liumichelle.com, masonjwang.com)
**What it does:** A small circle or dot trails the real cursor with spring/lag physics.
**How to implement:**
```html
<div class="cursor-trailer" aria-hidden="true"></div>
```
```css
.cursor-trailer {
  position: fixed;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: currentColor;
  pointer-events: none;
  z-index: 9999;
  mix-blend-mode: difference;
  transition: width 0.2s, height 0.2s, opacity 0.2s;
  transform: translate(-50%, -50%);
}
body:not(:hover) .cursor-trailer { opacity: 0; }
```
```js
const trailer = document.querySelector('.cursor-trailer');
let tx = 0, ty = 0, cx = 0, cy = 0;
document.addEventListener('mousemove', e => { tx = e.clientX; ty = e.clientY; });
(function loop() {
  cx += (tx - cx) * 0.12;
  cy += (ty - cy) * 0.12;
  trailer.style.left = cx + 'px';
  trailer.style.top  = cy + 'px';
  requestAnimationFrame(loop);
})();
// Expand on interactive elements
document.querySelectorAll('a, button').forEach(el => {
  el.addEventListener('mouseenter', () => {
    trailer.style.width = '40px';
    trailer.style.height = '40px';
    trailer.style.opacity = '0.4';
  });
  el.addEventListener('mouseleave', () => {
    trailer.style.width = '12px';
    trailer.style.height = '12px';
    trailer.style.opacity = '1';
  });
});
```
**Notes:** `0.12` is the lerp factor — lower = more lag/spring, higher = tighter follow. `mix-blend-mode: difference` creates the inverted-color-over-content look. For a border-only ring version: set `background: transparent; border: 1.5px solid currentColor`.

---

### 04 — IMAGE FOLLOW CURSOR (PROJECT PREVIEW)
**Inspired by:** rachelchen.tech, masonjwang.com project cards
**What it does:** Hovering over a list item or link makes a preview image appear and follow the cursor.
**How to implement:**
```html
<li class="preview-link" data-img="project-thumb.jpg">Project Name →</li>
<div class="preview-img-wrap" aria-hidden="true">
  <img class="preview-img" src="" alt="">
</div>
```
```css
.preview-img-wrap {
  position: fixed;
  pointer-events: none;
  z-index: 1000;
  opacity: 0;
  transition: opacity 0.25s ease;
  transform: translate(-50%, -60%) rotate(-3deg);
  width: 280px;
}
.preview-img-wrap.visible { opacity: 1; }
.preview-img-wrap img { width: 100%; border-radius: 4px; }
```
```js
const wrap = document.querySelector('.preview-img-wrap');
const img  = wrap.querySelector('.preview-img');
let mx = 0, my = 0, px = 0, py = 0;
document.addEventListener('mousemove', e => { mx = e.clientX; my = e.clientY; });
document.querySelectorAll('.preview-link').forEach(link => {
  link.addEventListener('mouseenter', () => {
    img.src = link.dataset.img;
    wrap.classList.add('visible');
  });
  link.addEventListener('mouseleave', () => wrap.classList.remove('visible'));
});
(function loop() {
  px += (mx - px) * 0.1;
  py += (my - py) * 0.1;
  wrap.style.left = px + 'px';
  wrap.style.top  = py + 'px';
  requestAnimationFrame(loop);
})();
```
**Notes:** Adjust `rotate(-3deg)` and lerp factor for personality. Add a slight `scale` transform on `.visible` (1.0 vs 0.92) for an entrance pop.

---

### 05 — TEXT SCRAMBLE
**Inspired by:** Tech/hacker aesthetic, used by Jackie Zhang, glitch-style portfolios
**What it does:** On hover (or page load), letters randomize rapidly before settling on the real text.
**How to implement:**
```js
const CHARS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*';
function scramble(el) {
  const original = el.dataset.text || el.textContent;
  el.dataset.text = original;
  let iteration = 0;
  const interval = setInterval(() => {
    el.textContent = original
      .split('')
      .map((char, i) => {
        if (char === ' ') return ' ';
        if (i < iteration) return original[i];
        return CHARS[Math.floor(Math.random() * CHARS.length)];
      })
      .join('');
    if (iteration >= original.length) clearInterval(interval);
    iteration += 0.4; // lower = slower reveal
  }, 30);
}
document.querySelectorAll('[data-scramble]').forEach(el => {
  el.addEventListener('mouseenter', () => scramble(el));
});
```
```html
<h1 data-scramble>Aanya Lall</h1>
```
**Notes:** Run on page load for a dramatic entrance: call `scramble(el)` directly in DOMContentLoaded. `iteration += 0.4` controls how fast letters lock in. For a "decode on scroll" version, trigger inside an IntersectionObserver callback.

---

### 06 — MARQUEE / INFINITE TICKER
**Inspired by:** sundays.rsvp, many fashion/event sites
**What it does:** A line of text (or logos/tags) scrolls horizontally and loops seamlessly.
**How to implement:**
```html
<div class="marquee" aria-hidden="true">
  <div class="marquee__track">
    <span>Available for work &nbsp;·&nbsp; </span>
    <span>Available for work &nbsp;·&nbsp; </span>
    <span>Available for work &nbsp;·&nbsp; </span>
    <!-- duplicate enough copies to fill width × 2 -->
  </div>
</div>
```
```css
.marquee { overflow: hidden; white-space: nowrap; }
.marquee__track {
  display: inline-block;
  animation: marquee-scroll 18s linear infinite;
}
.marquee:hover .marquee__track { animation-play-state: paused; }
@keyframes marquee-scroll {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-50%); } /* -50% because track is 2× content width */
}
```
**Notes:** Duplicate the content (not wrap it) so the loop is seamless. Speed: 12s = fast, 24s = slow drift. Use `gap` with flexbox on `marquee__track` for spacing. Add `direction: reverse` for right-to-left.

---

### 07 — 3D CARD TILT
**Inspired by:** joshuawolk.com project cards, general portfolio card interactions
**What it does:** Card rotates in 3D space following the cursor while hovered.
**How to implement:**
```css
.tilt-card {
  transform-style: preserve-3d;
  transition: transform 0.1s ease;
  will-change: transform;
}
.tilt-card__inner {
  transform: translateZ(20px); /* lifts content toward viewer */
}
```
```js
document.querySelectorAll('.tilt-card').forEach(card => {
  card.addEventListener('mousemove', e => {
    const rect = card.getBoundingClientRect();
    const x = (e.clientX - rect.left) / rect.width  - 0.5; // -0.5 to 0.5
    const y = (e.clientY - rect.top)  / rect.height - 0.5;
    card.style.transform = `perspective(600px) rotateX(${-y * 12}deg) rotateY(${x * 12}deg) scale(1.02)`;
  });
  card.addEventListener('mouseleave', () => {
    card.style.transform = 'perspective(600px) rotateX(0) rotateY(0) scale(1)';
    card.style.transition = 'transform 0.5s cubic-bezier(0.23,1,0.32,1)';
    setTimeout(() => card.style.transition = 'transform 0.1s ease', 500);
  });
});
```
**Notes:** `12` in rotateX/Y is max degrees — reduce to 6 for subtle, increase to 18 for dramatic. Add a `radial-gradient` highlight that also tracks cursor position for a specular reflection effect (overlay div with `mix-blend-mode: overlay`).

---

### 08 — STAGGER REVEAL ON SCROLL
**Inspired by:** masonjwang.com sections, rachenlchen.tech project grid
**What it does:** Elements enter sequentially as they scroll into view (fade + rise).
**How to implement:**
```css
.stagger-item {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.stagger-item.visible {
  opacity: 1;
  transform: translateY(0);
}
```
```js
const observer = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const items = entry.target.querySelectorAll('.stagger-item');
      items.forEach((item, i) => {
        setTimeout(() => item.classList.add('visible'), i * 80);
      });
      observer.unobserve(entry.target);
    }
  });
}, { threshold: 0.15 });
document.querySelectorAll('.stagger-group').forEach(g => observer.observe(g));
```
```html
<section class="stagger-group">
  <div class="stagger-item">Card 1</div>
  <div class="stagger-item">Card 2</div>
  <div class="stagger-item">Card 3</div>
</section>
```
**Notes:** `80ms` stagger delay is comfortable. Go as low as 40ms for fast cascades or up to 150ms for dramatic. Swap translateY for translateX for a horizontal slide-in. `threshold: 0.15` means trigger when 15% of the container is visible.

---

### 09 — NOISE / GRAIN TEXTURE OVERLAY
**Inspired by:** sundays.rsvp, bridgit-demoinvite.space — gives photos/backgrounds a film grain feel
**What it does:** Adds a subtle animated noise/grain texture over a section or the whole page.
**How to implement (pure CSS + SVG filter):**
```css
body::after {
  content: '';
  position: fixed;
  inset: -200%;
  width: 400%;
  height: 400%;
  pointer-events: none;
  z-index: 9998;
  opacity: 0.035;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
  animation: grain-shift 0.5s steps(1) infinite;
}
@keyframes grain-shift {
  0%   { transform: translate(0, 0); }
  10%  { transform: translate(-2%, -3%); }
  20%  { transform: translate(3%, 1%); }
  30%  { transform: translate(-1%, 4%); }
  40%  { transform: translate(4%, -2%); }
  50%  { transform: translate(-3%, 3%); }
  60%  { transform: translate(1%, -4%); }
  70%  { transform: translate(-4%, 1%); }
  80%  { transform: translate(2%, 3%); }
  90%  { transform: translate(-1%, -1%); }
  100% { transform: translate(3%, -2%); }
}
```
**Notes:** `opacity: 0.035` is barely perceptible but adds tactile texture. Increase to 0.08 for stronger film grain. The animated `transform` shifts randomize which part of the noise tile shows, creating flickering grain. Use `::before` on a specific section instead of `body::after` to scope it.

---

### 10 — LINK UNDERLINE DRAW
**Inspired by:** Jackie Zhang, belle duffner — elegant link hover
**What it does:** A thin underline draws from left to right on hover (not the default browser underline).
**How to implement:**
```css
.draw-link {
  position: relative;
  text-decoration: none;
}
.draw-link::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 0;
  height: 1px;
  background: currentColor;
  transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.draw-link:hover::after { width: 100%; }
```
**Variation — draw out, erase from left:**
```css
.draw-link::after {
  left: auto;
  right: 0;
  transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.draw-link:hover::after { width: 100%; left: 0; right: auto; }
```
**Notes:** For a "wipe out" on mouse-leave, use two pseudo-elements: `::before` draws in on hover, `::after` erases on leave by expanding from the right with a slight delay.

---

### 11 — SPLIT TEXT CHARACTER ANIMATION
**Inspired by:** Jackie Zhang (scattered letterforms), kinetic type portfolios
**What it does:** Wraps each character in a `<span>` and animates them with staggered delays — on load, on hover, or triggered by scroll.
**How to implement:**
```js
function splitText(el) {
  const text = el.textContent;
  el.innerHTML = text
    .split('')
    .map((char, i) => char === ' '
      ? '<span aria-hidden="true">&nbsp;</span>'
      : `<span class="char" style="--i:${i}" aria-hidden="true">${char}</span>`)
    .join('');
  el.setAttribute('aria-label', text); // keep accessible
}
document.querySelectorAll('[data-split]').forEach(splitText);
```
```css
[data-split] .char {
  display: inline-block;
  opacity: 0;
  transform: translateY(20px) rotate(4deg);
  animation: char-in 0.4s cubic-bezier(0.23, 1, 0.32, 1) forwards;
  animation-delay: calc(var(--i) * 30ms);
}
@keyframes char-in {
  to { opacity: 1; transform: translateY(0) rotate(0deg); }
}
```
```html
<h1 data-split>Hello</h1>
```
**Notes:** `30ms` per character is punchy; use `50ms` for slow-motion drama. Remove `rotate(4deg)` for clean vertical reveal. For hover-triggered version: add/remove a `.active` class on mouseenter/leave and scope the animation to `.active .char`.

---

### 12 — SCROLL PROGRESS BAR
**Inspired by:** Common editorial sites, sang han journal style
**What it does:** A thin bar at the top of the viewport fills as the user scrolls down the page.
**How to implement:**
```html
<div class="scroll-progress" aria-hidden="true"></div>
```
```css
.scroll-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 2px;
  width: 0%;
  background: currentColor;
  z-index: 10000;
  transition: width 0.05s linear;
  mix-blend-mode: difference;
}
```
```js
window.addEventListener('scroll', () => {
  const scrolled = window.scrollY;
  const total = document.documentElement.scrollHeight - window.innerHeight;
  document.querySelector('.scroll-progress').style.width = (scrolled / total * 100) + '%';
});
```
**Notes:** `mix-blend-mode: difference` makes it visible on any background. Use a specific color instead for a branded accent. Height 2px is standard; 1px is very minimal, 3px is bolder.

---

### 13 — HOVER COLOR INVERT / NEGATIVE
**Inspired by:** bridgit-demoinvite.space, invite-card aesthetics
**What it does:** On hover, a section or button flips to inverted colors — dark bg becomes light, text swaps.
**How to implement:**
```css
.invert-hover {
  background: var(--bg, #fff);
  color: var(--fg, #111);
  transition: background 0.25s ease, color 0.25s ease;
}
.invert-hover:hover {
  background: var(--fg, #111);
  color: var(--bg, #fff);
}
```
**Notes:** Use CSS variables so both states are defined in one place. For a "fill sweep" variant instead of instant swap, use a `::before` pseudo-element that scales from `scaleX(0)` to `scaleX(1)` with `transform-origin: left`, sitting behind the text with `z-index: -1`.

---

### 14 — AMBIENT GLOW / SPOTLIGHT FOLLOW
**Inspired by:** sundays.rsvp, invite-style landing pages
**What it does:** A radial gradient "spotlight" or ambient glow follows the cursor, lighting up the section it's in.
**How to implement:**
```html
<section class="glow-section">...</section>
```
```css
.glow-section {
  position: relative;
  overflow: hidden;
}
.glow-section::before {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(600px circle at var(--mx, 50%) var(--my, 50%),
    rgba(200, 200, 220, 0.08), transparent 60%);
  pointer-events: none;
  z-index: 0;
  transition: opacity 0.3s;
  opacity: 0;
}
.glow-section:hover::before { opacity: 1; }
```
```js
document.querySelectorAll('.glow-section').forEach(section => {
  section.addEventListener('mousemove', e => {
    const rect = section.getBoundingClientRect();
    const x = ((e.clientX - rect.left) / rect.width  * 100).toFixed(1) + '%';
    const y = ((e.clientY - rect.top)  / rect.height * 100).toFixed(1) + '%';
    section.style.setProperty('--mx', x);
    section.style.setProperty('--my', y);
  });
});
```
**Notes:** Adjust the `rgba` color to match your palette — warm white for cozy, cool blue for tech, gold for luxury. For a page-wide version that always shows, move the `::before` to `body` and drive `--mx`/`--my` from a global `mousemove` listener.

---

### 15 — TYPEWRITER / CYCLING TEXT
**Inspired by:** portfolio taglines (megan yap, michelle liu)
**What it does:** Cycles through an array of strings with a typewriter type-and-erase effect.
**How to implement:**
```html
<span id="typewriter" data-words='["designer","builder","thinker","friend"]'></span>
```
```js
const el = document.getElementById('typewriter');
const words = JSON.parse(el.dataset.words);
let wi = 0, ci = 0, deleting = false;
function tick() {
  const word = words[wi];
  if (!deleting) {
    el.textContent = word.slice(0, ++ci);
    if (ci === word.length) { deleting = true; return setTimeout(tick, 1200); }
  } else {
    el.textContent = word.slice(0, --ci);
    if (ci === 0) { deleting = false; wi = (wi + 1) % words.length; }
  }
  setTimeout(tick, deleting ? 40 : 80);
}
tick();
```
**Notes:** 80ms per character = normal typing speed; 50ms = fast typist. 1200ms pause at end of word. Add a blinking cursor: `el.parentElement` gets `::after { content: '|'; animation: blink 1s step-end infinite; }`.

---

## Combining Effects

Effects are designed to compose without conflict:
- **Silver hover + Draw link underline**: Apply both to the same nav links (`.silver-hover.draw-link`).
- **Stagger reveal + Split text**: Wrap stagger groups in split-text elements. Run split first, then stagger the `.char` spans instead of the item directly.
- **Noise grain + Ambient glow**: Both use `::before`/`::after` — scope grain to `body::after` and glow to `section::before`.
- **Magnetic button + Spring cursor**: The cursor trailer expands on hover (already in Effect 03), pairs naturally with magnetic button pull.
- **3D tilt + Noise grain**: Add the grain overlay inside the card so it tilts with the 3D transform.

## Accessibility

Every effect must respect reduced motion:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```
Add this block if it doesn't already exist. Custom cursors should still move (functional), but the spring lag can be zeroed out under reduced-motion by setting the lerp factor to 1.

## Code Quality Rules

- **No libraries unless already present.** All effects work in vanilla JS + CSS.
- **No global style pollution.** Use specific class names. Never override browser defaults globally.
- **No `!important` except in the reduced-motion block.**
- **Inline event listeners go in a `<script>` at the bottom of `<body>` or in the existing JS file.** Never in `onclick=` HTML attributes.
- **Remove any effect cleanly** — each effect is self-contained and can be removed by deleting its CSS block and JS function without breaking anything else.
