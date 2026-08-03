# Daily Repo Opportunity Scan: 2026-08-03

No `.ai/previous_review.md` existed prior to this run — this is the baseline scan. No commits landed in the last 24h (HEAD `fa85782` is from 2026-07-12), so this covers the current repo state in full rather than a 24h diff.

## 1. Net-New Opportunities (High Priority)

1. **Orphaned "Tokens"/"AI" tab code was never deleted.** `e39c7ac` ("Remove Tokens and AI tabs...") ripped the tabs out of `src/panel/App.tsx` but left the implementation files behind: `src/panel/components/AITab.tsx` (546 lines), `src/panel/components/TokensTab.tsx` (172 lines), `src/panel/components/ai.css` (483 lines), `src/panel/components/tokens.css` (125 lines) — 1,326 lines, zero references anywhere in `src/`. CHANGELOG.md even documents the removal intent. Value unlock: delete the four files; shrinks the panel bundle and removes a maintenance trap where a future contributor edits dead code.
2. **Design tokens exist but aren't fully adopted in `controls.css`.** `panel.css` defines a proper `--pd-*` token set (colors, spacing) including a light-mode override block, but `src/panel/controls/controls.css` still hardcodes raw hex (`#fff`, `#000`, `#ccc`) in ~12 places (lines 355-1088) for swatch borders and checkerboard fills. Most are legitimately theme-invariant (transparency-grid, pure-white swatch preview), but worth a pass to confirm none should track `--pd-text`/`--pd-border` for the light-mode override to apply correctly.

## 2. Design System & UI Consistency

- `src/design-system/SidebarPreview.tsx` still exists as a standalone "design system preview" surface but its only prior consumer (the Tokens tab) is the dead code from item 1 above — confirm it's still reachable/used before treating it as live.
- No other duplicate-component drift found this pass (controls in `src/panel/controls/` — `ColorPicker`, `FontPicker`, `SelectDropdown`, etc. — each have distinct, non-overlapping responsibilities).

## 3. Status of Previous Flags

No prior report exists — nothing to carry forward. This file is now the baseline for tomorrow's diff.

## 4. Suggested Action/Execution Plan

```bash
git rm src/panel/components/AITab.tsx src/panel/components/TokensTab.tsx src/panel/components/ai.css src/panel/components/tokens.css && npm run typecheck
```
