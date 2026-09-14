# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static portfolio site for Zachary Lu-Ming Fan (UI/UX designer). Plain HTML/CSS, zero JavaScript, no build tooling, no package manager. Deployed as-is via GitHub Pages — pushing to `main` on `origin` (`https://github.com/zachfan-96/zachfan-96.github.io`) publishes directly to `https://zachfan-96.github.io/`, with no build/CI step in between.

## Running locally

There's no dev server built into the repo (no `package.json`, no Python/Node guaranteed on the host). Serve the directory with whatever static server is available, e.g.:
- `npx serve .` or `python -m http.server 8000` if Node/Python are present, **or**
- a minimal `System.Net.HttpListener`-based static file server in PowerShell if neither is installed (this has been needed before on Windows hosts here — check for `python`/`node` first before assuming they exist).

Then open `index.html` (or any `*_case_study.html`) through that server rather than via `file://`, so relative asset paths resolve correctly.

## Architecture

Four standalone HTML pages, each self-contained:
- `index.html` — homepage (hero, selected work list, about, footer)
- `cov_case_study.html`, `ctr_case_study.html`, `homescope_case_study.html` — one case study per project

**CSS is split two ways:**
- `styles.css` — rules that are byte-identical across multiple pages: reset, CSS custom properties (`--ink`, `--paper`, etc.), nav, the shared `.case-hero`/`.case-content`/`.case-tags`/`.case-bottom-nav` case-study scaffolding, the `fadeUp` keyframes, and the parts of the mobile breakpoint that are common to all pages.
- Each page's inline `<style>` block — rules unique to that page only (e.g. `index.html` keeps its own `.hero`/`.work`/`.about`/`footer` rules; each case study keeps its own content-grid/feature-row/component-grid rules).

When editing shared visual elements (nav, case-study hero band, tag pills, bottom nav), change `styles.css` once rather than editing each page — that's the reason the split exists. When editing something page-specific, it stays inline in that page.

**Known CSS gotcha — do not remove:** `styles.css` has `img, video { height: auto; }` near the top. Every `<img>` (and the three GIF-replacement `<video>` elements on the HomeScope page) carries explicit `width`/`height` HTML attributes (added to prevent layout shift), but none of the page CSS declares an explicit `height` for these elements — sizing instead relies on `aspect-ratio`/`object-fit`/percentage `width`. Without `img, video { height: auto; }` overriding it, browsers use the `height` HTML attribute as a literal pixel value, which nullifies `aspect-ratio` (it only applies when a dimension is `auto`) and badly distorts the element. If you add a new image/video sizing rule with explicit `width`/`height` attributes, either give it its own explicit `height` in CSS or keep relying on this global override — don't remove it without checking every sized `<img>`/`<video>` still renders correctly.

**Assets** live at the repo root (not in a subfolder) and are named kebab-case (e.g. `cov-slide-1-opening-slide.png`, `portfolio-cov.png`, `homescope-app-demo-1.mp4`). Match that convention for anything new — the repo previously had spaces in filenames and they were all renamed for URL/CLI safety.

**No JavaScript, no framework.** Interactivity (hover states, animations, mobile nav collapse) is CSS-only (`:hover`, `@keyframes`, `@media (max-width: 768px)`). The 3 HomeScope interaction demos autoplay/loop via plain `<video autoplay loop muted playsinline>` — no JS needed. On narrow screens `index.html` hides `.nav-link` entirely (`display: none` in the mobile breakpoint) rather than collapsing into a hamburger menu — there is no mobile nav menu in this site. Don't introduce a JS dependency for something that can stay CSS/HTML-only, to keep the site's zero-build-step deployment intact.

**Tap targets.** `.nav-back`, `.nav-link`, and `.back-link` in `styles.css` carry `padding: 0.75rem 0` (and `.nav-link` is `display: inline-block`) specifically to keep their clickable area comfortable on touch devices even though the visible text is small — keep that padding if you touch these rules.

**Click-to-enlarge lightbox (all 3 case study write-ups).** Every content image inside `.case-content` on `cov_case_study.html`, `ctr_case_study.html`, and `homescope_case_study.html` (screenshots, diagrams, wireframes, component closeups — not hero banners or the autoplay demo videos) opens a full-screen overlay on click, built with the CSS-only `:target` technique (no JS): the image is wrapped in `<a href="#lb-N" class="lightbox-link">`, followed by a sibling `<div id="lb-N" class="lightbox">` containing a `.lightbox-backdrop` (click to close), a `.lightbox-close` × button (click to close), and the enlarged `<img>`. Shared CSS lives in `styles.css` under "Lightbox". IDs (`lb-1`, `lb-2`, …) only need to be unique within each page. The close/backdrop links use `href="#!"`, not `href="#"` — an empty fragment (`#`) is defined by the HTML spec to scroll to the top of the document, which yanked the page to the top on every close; `#!` matches no element, so it clears `:target` without moving the scroll position. The enlarged `<img>` is sized by `.lightbox:target img`, and the `:target` qualifier there is **load-bearing specificity, not decoration** — most lightboxes sit inside a card wrapper (`.wf-card`, `.component-card`, `.component-card-wide`) whose page-inline rule sizes images to fill the card (`width: 100%`). That rule ties `.lightbox img` on specificity (0,1,1) and wins on source order, stretching the enlarged image's layout box far past the visible picture; clicks on that transparent overhang then land on the `<img>` (which has no close handler) instead of the backdrop, so clicking beside the image silently fails to close. Keep `width`/`height` at `auto` there so the box hugs the picture exactly, and don't lower that selector's specificity.

Follow this same pattern for any new content image added to a case study write-up.

**Media optimization.** `ctr_demo.mp4` and `passive_cooling_prototype.mp4` are H.264 (libx264, CRF 27, faststart) re-encodes of the original captures — re-encode with the same approach (`ffmpeg -i in.mp4 -vf "scale='min(1600,iw)':-2" -c:v libx264 -preset slow -crf 27 -pix_fmt yuv420p -c:a aac -b:a 96k -movflags +faststart out.mp4`) if replacing them. The three HomeScope interaction demos were originally GIFs and are now silent looping MP4s (`homescope-app-demo-{1,2,3}.mp4`) — GIF is a very inefficient format for this kind of content (the three files dropped from 4.6–8.1MB each to under 260KB by switching to H.264), so don't reintroduce GIFs for new demos; convert to a muted autoplay/loop `<video>` instead (`ffmpeg -i in.gif -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" -c:v libx264 -preset slow -crf 28 -pix_fmt yuv420p -movflags +faststart -an out.mp4` — the scale filter forces even dimensions, which libx264 requires). There's no automated compression pipeline in this repo, so this has to be done by hand before committing new media.
