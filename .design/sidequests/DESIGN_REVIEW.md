# Design Review: Sidequests ("my desk")

Reviewed against: your live aesthetic (index / blog / spaces / about) + reference you cited — [Rocky's Matcha · Matcha Guide](https://www.rockysmatcha.com/blogs/matcha-guide)
Philosophy: warm editorial gallery — EB Garamond, cream `#f4efe4`, gold `#8a7355`, generous whitespace, mono micro-labels, italic discipline, soft imagery, restraint.
Date: 2026-06-06

## Screenshots Captured (preview)

| State | What it shows |
| --- | --- |
| landing | "welcome to my desk" over **black** — desk photo is broken/missing |
| desk view (lit) | photoreal warm wooden desk room — but **empty of the hotspot objects** |
| desk view (night) | near-total black; only the torch cursor reveals anything |
| popup (dfa) | cream/serif card — **this part is on-brand** |

## Summary

The concept — an explorable photoreal "desk" full of side projects — is ambitious and the *content* inside it (unsent letters, fortune cookie, places-to-go boarding pass, working photobooth) is genuinely charming and very you. But the page is the single biggest aesthetic outlier on your site: it's **dark, photoreal, gamified, and hidden**, while everything else (and the Rocky's reference you love) is **light, editorial, calm, and legible**. On top of that, it's currently **broken on entry** and the core interaction doesn't actually work, because the desk photo doesn't contain the objects the hotspots point to.

## Must Fix

1. **The landing image is broken.** `sidequests.html` line 251 loads `sidequests/zoomout.png`, but the file on disk is `sidequests/zoom-out.png` (hyphen). The entire first impression is a black screen with a broken-image icon. _Fix: rename the reference to `zoom-out.png` (or rename the file)._

2. **The hotspots point at objects that aren't in the photo.** The desk render is a clean, mostly-empty wooden desk. There's no DFA blueprint, no laptop, no playbill pile, no boarding pass, no fortune cookie, no drawer. So the experience is: hover blindly across an empty desk hoping to hit invisible zones whose labels name things you can't see ("the playbill pile" floating over a bare shelf). The metaphor collapses. _Fix: either (a) use/generate a desk image that actually contains those props so each hotspot sits on a real object, or (b) drop the literal-room metaphor (see "How to move forward")._

3. **Default state is barely visible.** After 8pm night mode dims the desk to ~black, and even in day mode the room leans dark and relies on the torch cursor to reveal anything. With no mouse movement (or on touch — there's no torch on tap) it reads as a broken black page. _Fix: raise base brightness, make night mode a subtle accent not a blackout, and give hotspots a faint always-on shimmer so they're discoverable without hunting._

## Should Fix

1. **Aesthetic mismatch with the rest of the site.** Dark `#0a0905`, photoreal imagery, torch/flashlight mechanic, mystery-box discovery — none of this vocabulary appears on index, blog, spaces, or about, which are cream, typographic, and open. A visitor moving from those pages to this one feels like they left your site. _Fix: see "How to move forward."_

2. **Performance.** `zoomin.png` is **17 MB** and `zoom-out.png` is **16 MB**. That's a multi-second load on good wifi, far worse on mobile. _Fix: export at ~1920px wide, compress to WebP — should land each under ~400 KB._

3. **No mobile / touch story.** The torch cursor, `cursor:none`, and hover-only tooltips don't exist on touch devices; hotspot positions are tuned to a 1536×1024 desktop crop. On a phone this is likely an empty dark image you can't navigate. _Fix: a tap-friendly fallback, or a different layout on small screens._

4. **Accessibility.** Hotspots are `<div onclick>` (not keyboard-reachable, no roles), nav text is ~15% opacity over a busy photo (fails contrast), there's no `prefers-reduced-motion`, and `cursor:none` removes the system cursor. _Fix: real `<button>`s/links, legible nav, reduced-motion guard._

## Could Improve

1. The popup close affordance and veil are nice; the cream popup system is the strongest, most on-brand piece here — lean into it.
2. The unsent-letters, fortunes, and places content is excellent writing — it deserves to be *found*, not buried behind a flashlight.

## What Works Well

- **The popup content system is fully on-brand** — cream card, serif italic headers, gold mono tags, stat rows, progress bars. It already matches index/blog/spaces. This is your bridge.
- **The writing and ideas are distinctive and warm** — "emails i wrote after talks… never sent them, kept the lessons," the fortune cookie, the boarding-pass wishlist. This is the good stuff.
- **The working photobooth** is a real "wow" interaction and very shareable.

---

## How to move forward — 3 options

**Option A — Fix the room as-is.** Rename the image, swap in a desk render that actually contains the props, compress everything, lighten the base, add subtle always-on hotspot glows, build a touch fallback. Keeps the immersive idea but it stays the aesthetic outlier and is the most fragile (depends on a perfect photo).

**Option B — Recommended: rebuild sidequests as a cream editorial "index of side quests"** — exactly the structure of the Rocky's matcha guide you like, and consistent with your spaces/blog pages. A warm cream page, big serif title ("sidequests — things in progress"), a short italic intro, then a **grid of guide-cards** (dfa · afterhours · unsent letters · fortune · places i want to go · photobooth · breakfast · playbill). Each card opens the **same cream popups you already built** — so you keep 100% of the charming content and the photobooth, lose the broken parts, and the page finally feels like the rest of your site. Soft imagery + whitespace + mono labels, like Rocky's grid.

**Option C — Hybrid.** Do Option B as the real page, and keep the dark "desk room" as one optional easter-egg card inside it ("peek into my actual desk →") — fixed and lightweight — so the immersive idea survives without being the front door.

My recommendation: **B, optionally keeping the room as a C-style easter egg.** It's the smallest amount of new work for the biggest jump in coherence, it directly matches the Rocky's reference, and it rescues all the writing/interactions that are currently hidden.
