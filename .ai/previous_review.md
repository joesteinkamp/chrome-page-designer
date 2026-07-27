# Daily Repo Opportunity Scan: 2026-07-27

*First run — no prior review file existed in `.ai/`. Repo's last commit is `fa85782` (2026-07-12); no commits in the last 24h, so this scan covers the current HEAD in full rather than a rolling diff.*

## 1. Net-New Opportunities (High Priority)

1. **Dead tab components left behind after removal — `src/panel/components/TokensTab.tsx` (172 lines) and `src/panel/components/AITab.tsx` (546 lines).** Commit `e39c7ac` ("Remove Tokens and AI tabs...") stripped both from `App.tsx`'s `Tab` union and render tree (`App.tsx:13` now only has `"design" | "changes"`), but never deleted the files. Neither is imported anywhere in `src/`. Token binding functionality was independently rebuilt inline in swatch controls (CHANGELOG "Design-token swatches bind var(--name)..."), so `TokensTab.tsx` isn't just unreferenced, it's superseded — safe to delete outright. 718 lines of dead weight in the panel bundle and a false trail for anyone grepping for "how does the AI tab work."
   - Fix: `git rm src/panel/components/TokensTab.tsx src/panel/components/AITab.tsx` and grep for any leftover message types in `src/shared/messages.ts` that existed solely to feed them.

## 2. Design System & UI Consistency

No new deviations found. Hardcoded hex colors in panel components (`ColorPicker.tsx`, `FillSection.tsx`, `StrokeSection.tsx`, `TypographyTab.tsx`, icons) are limited (6 occurrences) and consistent with prior patterns — not a new drift. No duplicate component patterns detected among sections/controls this pass.

## 3. Status of Previous Flags

N/A — this is the baseline review. Nothing to carry forward.

## 4. Suggested Action/Execution Plan

`claude -p "Delete src/panel/components/TokensTab.tsx and AITab.tsx, remove any now-unused message types they exclusively fed in src/shared/messages.ts, and run npm run typecheck"`
