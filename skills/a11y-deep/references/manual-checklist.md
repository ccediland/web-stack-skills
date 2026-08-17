---
title: The manual layers — per-release smoke and the WCAG-EM 2.0 audit
summary: The codified per-release manual smoke (keyboard, screen readers, reflow, form semantics, and the six WCAG 2.2 A/AA deltas as literal checklist lines) and the WCAG-EM 2.0 structure reserved for public conformance claims.
last_updated: 2026-08-17
applies_to: WCAG 2.2 (Rec) · WCAG-EM 2.0 (W3C Group Note 2026-07-23) · NVDA · VoiceOver
---

# The manual layers

> Automation ends at roughly half the real issues; these two layers are the other half. Tier 1 is a smoke pass any release can afford; tier 2 is the structured audit a public claim requires. Record findings as issues with the WCAG criterion number — findings without criteria numbers rot.

## Contents

- Tier 1 — per-release manual smoke
- The WCAG 2.2 A/AA deltas (literal lines)
- Tier 2 — the WCAG-EM 2.0 audit
- Recording and fixing

## Tier 1 — per-release manual smoke

Run on the release candidate (the live preview, not localhost), on the key templates plus any page the release touched. Roughly 30-45 minutes for a marketing-site surface.

Keyboard, no mouse:

- Tab through every page — every interactive element reachable, in a sensible order, no traps.
- Focus VISIBLE at every stop (and not clipped under sticky headers — see the 2.2 deltas).
- Escape closes anything that opened; nothing opens on focus alone.
- Skip link present and functional on pages with repeated nav.

Screen reader smoke (two engines, ~10 minutes each):

- NVDA + Firefox or Chrome on Windows; VoiceOver + Safari on macOS or iOS.
- Landmarks announce sensibly (banner, nav, main, contentinfo); headings outline the page; images announce alt or are silent when decorative.
- Forms: every field announces its label; submitting with errors announces WHAT failed and moves focus or aria-live announces it; success states announce.
- Any client-side navigation announces the new page (see `astro-seams.md` for the ClientRouter rule).

Zoom and reflow (WCAG 1.4.10):

- 400% browser zoom on a 1280px window (= 320px reflow): no two-dimensional scrolling for content, nothing clipped or overlapped, all functions still reachable.
- Text-spacing bookmarklet (1.4.12) on text-heavy templates.

Media and motion:

- prefers-reduced-motion ON: no autoplaying video, animations reduced per the motion-system posture, WebGL fallback static.
- Any audio/video content has captions/transcript per its kind.

## The WCAG 2.2 A/AA deltas (literal lines)

The six new criteria — five have ZERO automation; check them by hand every time:

- 3.2.6 Consistent Help (A) — help mechanisms (contact link, WhatsApp button, chat) appear in the SAME relative place on every page that has them.
- 3.3.7 Redundant Entry (A) — no form asks for the same information twice in one process (or it auto-fills/offers the previous value). Multi-step forms are the trap.
- 2.4.11 Focus Not Obscured minimum (AA) — with sticky headers/banners present, tab through the page: the focused element is never ENTIRELY hidden behind the sticky layer.
- 2.5.7 Dragging Movements (AA) — anything draggable (sliders, carousels, maps) has a single-pointer alternative (buttons, taps).
- 2.5.8 Target Size minimum (AA) — interactive targets ≥24×24 CSS px or adequately spaced; the axe/Lighthouse target-size rules cover most of this — the manual check is for targets automation cannot classify (inline links are exempt; icon buttons are not).
- 3.3.8 Accessible Authentication minimum (AA) — no cognitive test (memorize, transcribe, solve) to log in; password-manager paste allowed; email links or OTP paste count as alternatives. Relevant the day a site gains any login.

Tombstone: 4.1.1 Parsing was REMOVED in WCAG 2.2 — do not spend manual time on it and do not let old checklists resurrect it.

## Tier 2 — the WCAG-EM 2.0 audit

Reserved for a public conformance or accessibility statement. Follow the current methodology (WCAG-EM 2.0, Group Note 2026-07-23) — five steps:

1. Define the evaluation scope — which pages/processes, which conformance target (WCAG 2.2 AA), which assistive-tech baseline.
2. Explore the site — identify page types, templates, key functionality, required technologies.
3. Select the sample — representative pages (every template, every process) PLUS a random sample; complete processes must be included end-to-end.
4. Evaluate the sample — every applicable success criterion per sampled page, using the WCAG quick-reference as the per-criterion source; record pass/fail/not-applicable per criterion per page.
5. Report — aggregated findings, per-criterion results, the sample list, and the evaluation statement. The W3C WCAG-EM Report Tool structures exactly this output.

The public statement then quotes THIS audit (date, scope, methodology, known exceptions) — never the CI badge. CI proves regressions are caught; the audit proves conformance was evaluated.

## Recording and fixing

- Every finding = one issue: WCAG criterion number, page, assistive tech/config, expected vs observed, severity by user impact.
- Fixes land behind the same gates as any change (the axe job catches regressions on the automated subset; re-run the manual line that found it).
- Findings that trace to the brand canon (contrast pairs, motion identity) are OWNER decisions — file them as such per the brand-canon-ingest promotion path, do not patch tokens site-side.
