# EPOCH — Project Rules
# Read this entire file before writing any code.

## WHAT YOU ARE BUILDING
Single-file scroll website for EPOCH, a watch and wall-clock brand.
The entire site is one scroll-scrubbed video of a single life, told
across seven stages — a clock moving from wall, to wrist, through
every stage of a life, and back to the wall. Scroll down = forward
through the life. Scroll up = backward. Everything lives in
index.html. No npm. No build tools.
Served locally with: python3 -m http.server 8080

## HARD RULES — NEVER BREAK
1. ONE FILE ONLY — everything in index.html
2. NO CANVAS EVER
3. VIDEO SCRUB uses getBoundingClientRect() only — never window.scrollY alone
4. Never add will-change:transform or translateZ(0) to any <video>
5. The hard scroll-seam jump (see LOOP LOGIC) must use an INSTANT
   scrollTo — never smooth-scroll the jump itself
6. loopCount is a plain JS variable, never persisted to storage —
   it resets naturally on page reload, which is intentional
7. Respect prefers-reduced-motion — see ACCESSIBILITY FALLBACK.
   Not optional polish, a hard requirement before launch
8. The seven stage cuts are intentional hard cuts, not morphs.
   Do not attempt to crossfade between stages — a hard cut is
   correct here, do not soften it

## CDN IMPORTS — exact order, always
<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.23/dist/lenis.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
<script src="https://unpkg.com/lenis@1.3.23/dist/lenis.min.js"></script>

<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;1,9..144,400&family=Inter:wght@300;400;500&display=swap" rel="stylesheet">

## LOOP LOGIC — the core mechanism
State variables (module scope, top of script):
  let loopCount = 1;              // shown as GENERATION 01
  let breakTriggered = false;
  let breakRewindFloor = 0;

Main video (epoch-life.mp4) scrub: standard getBoundingClientRect
pattern — progress = clamp(-rect.top, 0, total) / total,
video.currentTime = progress * duration.

SEAM — scrolling down past the end:
  if scrollTop >= maxScrollTop AND scrolling down AND !breakTriggered:
    loopCount += 1
    update the GENERATION counter label
    if loopCount === 2: trigger BREAK instead of looping
    else: window.scrollTo(0, 1)

SEAM — scrolling up past the start:
  if scrollTop <= breakRewindFloor AND scrolling up:
    if breakTriggered: do nothing further, this is the hard floor
    else if loopCount > 1: loopCount -= 1, scrollTo(0, maxScrollTop - 1)
    else: do nothing, loopCount cannot go below 1

BREAK trigger (fires once, when loopCount first reaches 2):
  breakTriggered = true
  swap the active scrub video source to epoch-break.mp4
  set up a new local scroll track sized for the break video's duration
  breakRewindFloor = the scrollTop at the start of this new track
  near the end of this track's progress (roughly past 85%), fade in
    the closing text overlay — see RESOLUTION COPY below
  on reaching the end: hold on the final frame, do not loop or jump
    anywhere — the static Resolution / product spotlight section
    follows directly below in normal document flow

## THE GENERATION COUNTER UI
Fixed top-right, small wall-clock icon (simple SVG circle + two
hands), z-index 100, always visible, a separate render layer from
the scrubbed video, never inside the pinned scrub container. The
second hand ticks via setInterval(1000ms) on real wall-clock time,
fully decoupled from scroll. Label beneath: "GENERATION 0" +
loopCount, Inter 10px uppercase, letter-spacing 2px, color #C9A877.

## RESOLUTION COPY
Fades in near the end of the break sequence, then remains as the
opening of the static Resolution section below:
  Line 1 (italic, larger): "punarapi jananam, punarapi maranam."
  Line 2 (smaller, muted): "Again birth. Again death."
  Line 3 (smaller, muted): "Until something doesn't go back on the wall."

## ACCESSIBILITY FALLBACK — prefers-reduced-motion
At the very top of the script, before any scroll listeners attach:
  const reduced = window.matchMedia(
    '(prefers-reduced-motion: reduce)'
  ).matches;
  if (reduced) { initStaticFallback(); return; }
initStaticFallback() renders the seven stage images as seven normal
stacked 100vh sections in order, each with a simple CSS fade-in on
scroll into view (IntersectionObserver, no GSAP pin, no scroll
hijacking). The GENERATION counter, the seam jump, and the break
video are NOT shown in this mode. After the seventh section, show
one static section with epoch-suspended-still.png and the same
RESOLUTION COPY used above. Full normal page scroll — no scrollTo
calls anywhere in this code path.

## COLORS
#0A0A0A   near-black        Body background throughout
#F0EAE0   warm cream        All text, headings, nav
#C9A877   brass gold        Accent lines, counter, CTA borders
#6E6657   muted warm gray   Captions, secondary labels

## FONTS
Fraunces — all display/headings
Inter    — all body/labels/UI
All Sanskrit text appears as Latin transliteration only (e.g.
"punarapi jananam"), never as Devanagari script — keeps font
loading simple and avoids rendering issues across browsers.

## ASSETS
All image files are .png. Both videos must be re-encoded per
Steps 06 and 07 before this build starts.

## ALWAYS END THE SCRIPT WITH
ScrollTrigger.refresh();
