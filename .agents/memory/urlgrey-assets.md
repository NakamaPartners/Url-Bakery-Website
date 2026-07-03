---
name: urlgrey vanilla-HTML asset wiring
description: How to reference attached_assets from the urlgrey artifact's hand-written index.html (React never mounts)
---

# Referencing brand assets in urlgrey/index.html

`artifacts/urlgrey/index.html` is a vanilla HTML/CSS/JS page that fully replaces the React entry (React never mounts). Vite still serves/builds it.

**Rule:** to use files from `attached_assets/`, import them in a separate `<script type="module">` via the `@assets` alias and assign `.src` at runtime (e.g. `import x from '@assets/…jpg'; el.src = x`).

**Why:** the `@assets` alias points OUTSIDE the Vite root (`../../attached_assets`) and `server.fs.strict` is true, so plain `<img src="attached_assets/…">` won't resolve. Copying into `public/` is awkward because `base` is a dynamic `BASE_PATH`. Module imports resolve in both dev (`/@fs/…?import`) and production build (hashed emit) and keep the HTML close to the template.

**How to apply:** give media elements ids, keep their containers FIXED height (card `.art` = 230px, founder photo = 300px) so async media load causes no reflow — this protects the event-driven zero-lag scroll loop whose geometry is cached in `measure()`.

# Smooth transform-driven media galleries
The zero-lag scroll uses horizontal tracks translated via CSS transform on vertical scroll (metrics cached in `measure()`, writes gated on change in a single rAF `frame()`). To add another such section, mirror the existing track's metric+shift pattern — don't add layout reads inside `frame()`.

For video reels inside a transform-driven / sticky+overflow-hidden track: IntersectionObserver DOES respect post-transform geometry, so use one IO to (a) lazy-set each video `.src` only on first near-viewport intersect (`rootMargin` ~200px) and (b) play/pause by visibility. This keeps many videos from all buffering at once — the main smoothness lever. Skip autoplay under `prefers-reduced-motion`.

Planet parallax trick: put the JS floater parallax transform on the OUTER `.floater`, and a continuous CSS keyframe (bob/rotate) on the INNER `<img>`, so the two transforms compose without clobbering each other.

# Fonts / direction
Keep template fonts: Anton (display) / Homemade Apple (accents) / Poppins (body). Do NOT switch to Fredoka — user chose "follow the template, not the moodboard."
