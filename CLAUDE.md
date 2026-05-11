# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Single-page HTML deck (`index.html`) introducing 4 internal AI systems at 마인드리퍼블릭 — Slack 봇 3종(에스더 / 마리 / 이하은) + Apps Script 시트 자동화. Deployed to GitHub Pages at https://mindrepublic2018.github.io/ai-bots-presentation/. No build step, no framework, no dependencies — vanilla HTML/CSS/JS in one file.

Live demo videos (Tella embeds) are integrated for the Apps Script flows. Both desktop and mobile (iPhone notch / Android) are first-class.

## Deploy

Push to `main` and GitHub Pages auto-builds (~30–60s). Verify with:
```bash
gh api repos/mindrepublic2018/ai-bots-presentation/pages/builds/latest --jq '"\(.commit[:8]) \(.status)"'
```
Wait for `built`. The site URL stays the same across deploys; no per-PR previews.

There's only ever one source of truth — `index.html` at repo root. No `gh-pages` branch, no `/docs` folder. GitHub Pages serves from `main:/`.

## Architecture (single-file deck)

`index.html` contains inline `<style>`, slide markup, and a vanilla JS slide controller. The slide system is intentionally simple:

- Each slide is a `<div class="slide">` inside `<div class="slide-container">`. Slide order = DOM order. The JS auto-counts (`document.querySelectorAll('.slide')`) — adding/removing slides updates the counter automatically.
- `.slide.active` is the only visible slide; the rest are `opacity: 0; pointer-events: none`. Transitions handled by CSS.
- Navigation: ←/→/Space/PageUp/PageDown/Home/End/F (fullscreen) + touch swipe (guarded — see below) + bottom nav buttons. Progress bar reflects `idx / total`.
- HTML comments like `<!-- 4-2. Mari scenarios -->` are the only outline — there is no separate slide manifest. Use them when locating slides. Sub-slides use dot-suffix (`6-3-1`, `6-3-2`, `6-3-3` for the three Apps Script demo videos inserted between `6-3` and `6-4`).

### Scroll-safe centering (important — don't break)

`.slide` uses `display: flex; flex-direction: column;` with **no** `align-items/justify-content: center`. Vertical centering is delegated to `.slide-inner { margin: auto }`. This is deliberate: a flex container with `justify-content: center` + tall content + `overflow-y: auto` cuts off the top of overflowing content. The `margin: auto` pattern collapses to 0 on overflow, allowing natural scrolling.

If you change the slide layout, preserve this pattern.

### Bot color system

Each of the 4 systems has one accent color, used consistently across overview cards, detail slides, scenario chat bubbles, and section labels. Don't introduce new accents — reuse:

| 봇 | 변수 | Hex |
|---|---|---|
| 에스더 (AI 변호사) | `--accent-1` | `#6366f1` (indigo) |
| 마리 (경영지원) | `--accent-2` | `#8b5cf6` (violet) |
| 이하은 (비서실장) | `--accent-3` | `#ec4899` (pink) |
| 시트 자동화 (Apps Script) | `--accent-4` | `#06b6d4` (cyan) |

In a per-bot slide, set `style="--bot-color: <hex>;"` on `.slide-inner`. All scoped descendants (`.detail-icon-wrap`, `.feature-list li`, `.example`, `.chat-msg.bot`, `.character-avatar.size-detail`, video iframe wrapper border) read `--bot-color` for borders/glows. This is how the same scenario CSS adapts per bot.

### Reusable patterns inside slides

Three repeating component shapes — copy-paste from an existing slide of the same bot, then change the text:

1. **Detail slide** (per-bot intro) — `.detail-layout` 2-column grid: left has avatar + name + tagline + summary + `.feature-list`, right has heading + `.examples` (command list) + tip box.
2. **Scenario slide** — `.scenarios-grid` (3 columns) of `.chat-mockup` cards. Each mockup is a stack of `.chat-msg.user` (right-aligned) and `.chat-msg.bot` (left-aligned, accent-colored). Use this for showing real Slack-style conversations.
3. **Catalog/grid slide** — `.bot-grid`, `.apps-grid`, or ad-hoc CSS Grid for at-a-glance feature lists.
4. **Video slide** — see "Video embeds" below.

When adding a new feature scenario, the cheapest path is: copy a `.chat-mockup` block from the same bot's existing scenario slide (so the `--bot-color` cascade still works), swap the chat lines.

### Video embeds (Tella) — lazy-load pattern is mandatory

All Tella videos use **lazy-load** — the iframe loads only when its slide becomes active. This is required because hidden iframes with autoPlay would otherwise consume bandwidth and (in some browsers) leak muted audio.

**Pattern** (don't deviate):
```html
<iframe
  data-src="https://www.tella.tv/video/<vid_id>/embed?b=1&title=1&a=1&loop=0&autoPlay=true&t=0&muted=1&wt=1&o=1"
  src="about:blank"
  width="800" height="450"
  style="position:absolute;inset:0;width:100%;height:100%;border:0;"
  allow="autoplay; fullscreen"
  allowtransparency
  allowfullscreen>
</iframe>
```

The JS `show(i)` function handles the swap:
- **Entering** a slide: copies `data-src` → `src` (video loads + autoplays muted).
- **Leaving**: sets `src = 'about:blank'` (video stops + iframe content unloaded + network released).

If you add a new video, always set `data-src` (not `src`) and initialize `src="about:blank"`. The video slide layout wraps the iframe in a `position:relative; aspect-ratio:16/9` container so it scales naturally on any width.

Current video slides: `6-3-1` (매출현황·세금발행), `6-3-2` (신규 거래처 등록), `6-3-3` (입금요청 처리) — all in the Apps Script section, all autoPlay muted.

### Character avatars (`assets/`)

`.character-avatar` has 3 sizes: `size-card` (64px in overview), `size-detail` (140px on detail slides, with bot-color glow), `size-mini` (28px, currently unused but reserved for inline name+face combos). Character images are circular via `border-radius: 50%`.

**Asset pipeline** — large originals (`*.png` 12–18MB) live at repo root; web-optimized JPGs are committed under `assets/`. To add or replace:
```bash
sips -Z 800 -s format jpeg -s formatOptions 85 <원본> --out assets/<name>.jpg
```
The HTML always references `assets/*.jpg`. Do NOT commit the multi-MB originals into HTML `<img src>` paths — Pages serves them but page load suffers.

The unused `다니엘.jpg` and `벤자민.png` in repo root are placeholder personas not yet wired into any slide.

## Mobile considerations (≤600px)

Mobile got specific treatment after real-device testing — don't undo without re-testing on iPhone:

- `<meta name="viewport" content="...viewport-fit=cover">` + `env(safe-area-inset-*)` on `.nav` (bottom) and on `.slide` padding-bottom. Without this, iPhone home bar hides the nav buttons.
- `.slide { touch-action: pan-y; overscroll-behavior: contain; }` — vertical scroll is **always native** (instant), only horizontal swipes go to the JS handler. Without `pan-y`, scroll was unreliable on iOS.
- Touch swipe handler (`document.addEventListener('touchend', ...)`) requires `|dx| > 60 && |dx| > |dy| * 1.5` — prevents vertical scrolls from accidentally triggering slide navigation.
- `.slide-label` (top-left "SLIDE 03" overlay) is `display: none` on mobile — it was overlapping content. The bottom nav counter (`5 / 25`) covers the same need.
- `padding-bottom: calc(env(safe-area-inset-bottom, 0px) + 100px)` on `.slide` — guarantees content never hides under the fixed nav.
- The `@media (max-width: 600px)` block also flattens any inline `grid-template-columns:repeat(3,1fr)` / `1fr 1fr` to single column via attribute selectors — this catches the ad-hoc inline grids inside slide markup without needing class refactoring.

There's also a softer `@media (max-width: 900px)` breakpoint that stacks the main grids (`.detail-layout`, `.bot-grid`, `.apps-grid`, `.scenarios-grid`) but keeps font sizes intact. Tablet portrait sits in that range.

## When making changes

- **Slide insertion**: add the new `<div class="slide">` between existing ones. No JS update needed unless it's a video slide (see "Video embeds").
- **New bot/section**: introduce a new `--accent-*` variable + matching `--bot-color` cascade. The 4 existing accents are claimed; don't reassign.
- **Touch the inline `<style>` block carefully** — there's no CSS modules / scoping, so generic class names (`.feature-list`, `.examples`, `.chat-msg`) affect all slides using them.
- **After editing, test 3 viewports**: 1280px+ fullscreen (`F`), narrow window (~700px to hit the 900px breakpoint), and mobile (~375px to hit the 600px breakpoint). The mobile rules are strict — if you add an inline grid with a different column count (`repeat(4, 1fr)` etc.), it won't auto-flatten and you need to extend the attribute selector.
- **Commit messages** are descriptive but loose — e.g. `Add character portraits for Esther, Mari, Hahn`, `Mobile fix: native vertical scroll`. One line of what changed, optional details below.

## Notes for future edits

- The deck mixes character-avatar overview cards (에스더/마리/이하은) and an emoji card (Apps Script `📊`). If you give Apps Script a character image, update both the overview card (slide 2) and consider whether the section eyebrow / detail slide should use the avatar pattern.
- Slide numbering in HTML comments uses dot-suffix for sub-slides. When inserting between e.g. slide `6-3` and `6-4`, name it `6-3-N` so the existing order doesn't shift.
- The progress bar, slide counter, and slide label all derive from `slides.length` at script start — adding/removing slides updates them automatically.
