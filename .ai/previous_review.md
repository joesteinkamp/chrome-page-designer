# Daily Repo Opportunity Scan: 2026-08-10

_First run — no prior review existed, so this is a baseline audit of the full current state rather than a 24h diff. Last commit predates this scan by ~4 weeks (`fa85782`, 2026-07-12), so nothing here reflects "new" code in the literal 24h sense — it's the standing opportunity backlog as of today._

## 1. Net-New Opportunities (High Priority)

1. **No `useClickOutside` hook — reimplemented 7 times.** `src/panel/App.tsx:227-236`, `ColorPicker.tsx:308-318`, `FontPicker.tsx:18-28`, `UnitInput.tsx:68-76`, `SpacingSection.tsx:66-76`, `StrokeSection.tsx:104-114`, `TypographyTab.tsx:124-135` each hand-roll a ref-contains + mousedown-listener popover-close. `src/panel/hooks/` only has `useElementData.ts` and `useStyleChange.ts`. A shared hook removes ~60 duplicated lines and closes a divergence risk (the ref-guard logic already differs slightly between copies).
2. **`layers-panel.ts` does a full-tree teardown/rebuild on every relevant DOM mutation.** `src/content/layers-panel.ts:448-464` observes `childList`/`subtree`/`class`/`id` mutations across the entire host document; `scheduleRender()` (572-578, rAF-debounced) → `renderTree()` (580-592) does `treeContainer.textContent = ""` then a full recursive `renderNode()` from `document.documentElement`. On a dynamic SPA host page this is a full rebuild per animation frame with churn — a real perf risk since the extension's job is to sit on arbitrary live pages, some of which mutate constantly.
3. **Style-change property is unvalidated.** `StyleChange.property` is a bare `string` (`src/shared/types.ts:124`) and `APPLY_STYLE`/`APPLY_STYLES` (`src/shared/messages.ts:30-32`) carry `property: string; value: string` with no validation against real CSS properties. A typo'd property (plausible from AI-tab natural-language edits in `AITab.tsx`) applies silently as a no-op and still exports into the changeset — a developer or coding agent consuming the handoff has no signal it didn't do anything.

## 2. Design System & UI Consistency

`src/design-system/` is a component *showcase* (`index.tsx`, `SidebarPreview.tsx`), not the actual token/primitive source — real tokens live in `src/panel/panel.css` (e.g. `--pd-accent: #4f9eff`). The accent hex is hardcoded instead of referencing the CSS var in five places: `src/panel/sections/StrokeSection.tsx:172`, `src/panel/sections/FillSection.tsx:72`, `src/design-system/index.tsx:79,162,166`, `src/design-system/SidebarPreview.tsx:77`. Refactor: replace literal `#4f9eff` with `var(--pd-accent)` (or a shared JS token export) at each site so a future rebrand is a one-line change instead of a grep-and-pray.

## 3. Status of Previous Flags

None — this is the first scan, so there is no prior report to compare against. Future runs should diff against this file and only report items that are new or have worsened.

## 4. Suggested Action/Execution Plan

`claude -p "Add a useClickOutside(ref, onClose) hook in src/panel/hooks/ and replace the 7 duplicated ref+mousedown popover-close implementations in App.tsx, ColorPicker.tsx, FontPicker.tsx, UnitInput.tsx, SpacingSection.tsx, StrokeSection.tsx, and TypographyTab.tsx with it."`
