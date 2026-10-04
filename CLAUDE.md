# Kingdom Spin — Design Spec

Wheel-based sibling of Kingdom Clash. Same engine, same outcomes, same dual-window
setup — but instead of picking a numbered tile, a player stops a spinning wheel with
a physical buzzer.

**Before writing any code, read the reference implementation:**
`~/Desktop/Kingdom-Clash-main/index.html` and its `CLAUDE.md` (architecture + ten
hard-won gotchas: CSS cascade, hidden-tab rAF, sync design, Pages caching, etc.).
Reuse its patterns wholesale: outcome logic, pickers, popups, settings, score
count-ups, team/shield state, SFX pipeline. All sound/image assets are already
copied into this folder.

## Core loop

1. Wheel is divided into wedges — one per deck entry, same deck as Kingdom Clash:
   money (100/200/500/1000), BANKRUPT, x2, SPIN AGAIN (rename of PICK_AGAIN),
   STEAL (🐉, with skip option), FACE OFF (⚔️, 20s timer + winner picker),
   SHIELD (🛡️). Wedge counts configurable like board sizes: 12 / 16 / 20.
2. Operator starts the spin (button and/or spacebar). Wheel spins at constant
   speed INDEFINITELY — players need time to walk up to the buzzer.
3. The buzzer is a USB device that types the character "1". On buzz, the wheel
   stops INSTANTLY on whatever wedge is under the pointer. NO deceleration coast —
   host explicitly rejected it. Sell the stop with juice instead: pointer flick,
   landed-wedge flash/pulse, a hard stop sound, micro screen-shake.
4. The outcome applies to the active team, good or bad — landing on BANKRUPT on
   your own buzz is part of the game.
5. The consumed wedge is removed (shatter/fade animation) and the remaining wedges
   ANIMATE smoothly to proportionally larger angles filling 360°. This re-layout
   animation is a centerpiece moment — never snap it. Late-game surviving BANKRUPT
   wedges growing huge is a core drama feature.
6. Turn auto-rotates as in Kingdom Clash, with the manual Toggle button kept
   prominent — the host sometimes lets a team go twice in a row.
7. LAST wedge: still spin + buzz for ceremony (never auto-apply).
8. All wedges consumed → winner sequence. Keep Kingdom Clash's timing fix: let the
   final outcome popup play fully (popupAutoCloseMs + 700) before the winner popup.
9. New wheel = fresh game: zero scores, shields, turn — nothing carries over.

## Buzzer input (critical details)

- Listen for keydown of "1" in BOTH windows and broadcast a BUZZ event over the
  sync channel — keystrokes only reach the focused window, and focus may be on
  either one. The buzz must work regardless of focus.
- Debounce hard (kids hammer it): ignore unless a spin is live, dedup within
  ~500ms, ignore while any popup/picker is open.
- Operator needs a manual Stop button as a buzzer-failure fallback.
- Operator also keeps a Spin button; spacebar as shortcut.

## Wheel sync & rendering

- Do NOT stream rotation frames between windows. Broadcast spin EVENTS with
  parameters — spinStart(startAngle, angularVelocity, timestamp) and
  stop(finalAngle) — and let each window animate deterministically from the same
  numbers. The landing wedge is computed analytically from timestamps, so a
  hidden/backgrounded window still ends correct (rAF doesn't run there — see
  Kingdom Clash CLAUDE.md gotcha #3).
- SVG wedges rotated via requestAnimationFrame; 20 wedges is trivial. Labels:
  plain numbers for money, emoji for specials, color-coded by outcome type,
  readable from the back of a room on the presenter.
- Fixed pointer/flapper at 12 o'clock; the wedge under it at stop is the result.
- Presenter (?present): giant wheel + team HUD. Operator: wheel + controls +
  scores + log. Same broadcast trio as Kingdom Clash (BroadcastChannel +
  localStorage storage events + polling fallback).

## Everything else

Teams (2–4), names, scores, shields, steal/face-off pickers, settings panel with
Apply & Start Fresh Wheel, fluid clamp() sizing, SFX wiring (files in this folder;
note `face off.mp3` and `pick again.mp3` contain spaces), synthesized countdown
tick — all identical to Kingdom Clash. Single self-contained index.html, no build
step. Deploy: own fresh GitHub repo (this folder), GitHub Pages serving from main.
