# Daily Repo Opportunity Scan: 2026-08-17

_No prior review found — this is the baseline scan. Future runs will diff against this file._

## 1. Net-New Opportunities (High Priority)

1. **Dead code left behind by the tab removal (`src/panel/components/AITab.tsx`, `TokensTab.tsx`, `tokens.css`, `ai.css`)** — Commit `e39c7ac` ("Remove Tokens and AI tabs") deleted the tab wiring in `App.tsx` but left the components and styles in place. `AITab.tsx` (546 lines) and `TokensTab.tsx` (172 lines) are imported by nothing anywhere in `src/`. `tokens.css` (125 lines) is only imported by the now-dead `TokensTab.tsx`. `ai.css` (483 lines) is still pulled into `src/design-system/index.tsx` even though the component whose classes it styles is gone. ~1,326 lines of unreachable code/styles. Value unlock: delete all four files and drop the stray `ai.css` import — shrinks the panel bundle and removes a trap for future edits (e.g. token-binding work in `be921e6` touching `TokensTab.tsx`-adjacent concepts without anyone noticing the tab itself is dead).

2. **Click-outside-to-close logic hand-duplicated 7 times** — `App.tsx` (new `sendMenuRef`, added in `fa85782`), `components/TypographyTab.tsx`, `sections/StrokeSection.tsx`, `sections/SpacingSection.tsx`, `controls/FontPicker.tsx`, `controls/ColorPicker.tsx`, and `controls/UnitInput.tsx` each hand-roll the same `mousedown` + `ref.contains(e.target)` + `document.addEventListener`/cleanup pattern to close a popover/menu. The Send Changes menu rework just added an 7th copy instead of reusing one. Value unlock: extract a `useClickOutside(ref, onOutside, active)` hook into `src/panel/hooks/` (alongside the existing `useElementData.ts`/`useStyleChange.ts`) and swap all 7 call sites over — removes ~60 duplicated lines and gives future popovers (menus, pickers) one hardened implementation (including the `Escape`-key handling that only `UnitInput.tsx` currently has).

## 2. Design System & UI Consistency

- The `useClickOutside` gap above is really a design-system-drift issue: the panel has no shared "dismissable overlay" primitive, so every new menu/popover (gear popovers, font picker, color picker, and now the Send menu) reimplements dismissal from scratch with small behavioral differences (only `UnitInput` handles `Escape`; only `TypographyTab`/section popovers track a second "trigger button" ref to avoid re-opening on the toggle click). Consolidating into one hook would also make that Escape-key behavior consistent everywhere for free.
- No other new hardcoded-style regressions found in the files touched by the last 3 commits (`fa85782`, `14b53a8`, `e39c7ac`) — the new Send Changes menu CSS in `panel.css` correctly uses existing `--pd-*` tokens.

## 3. Status of Previous Flags

N/A — first run, no previous report to compare against.

## 4. Suggested Action/Execution Plan

`claude -p "Delete src/panel/components/AITab.tsx, TokensTab.tsx, tokens.css, and remove the now-dangling 'import \"./ai.css\"' equivalent reference from src/design-system/index.tsx (src/panel/components/ai.css); then extract the duplicated click-outside-close logic in App.tsx, TypographyTab.tsx, StrokeSection.tsx, SpacingSection.tsx, FontPicker.tsx, ColorPicker.tsx, and UnitInput.tsx into a shared src/panel/hooks/useClickOutside.ts hook and use it in all seven call sites"`
