# Daily Repo Opportunity Scan: 2026-08-31

*Note: no `.ai/previous_review.md` existed before this run — no commit activity in the last 24h either (repo's last commit is `fa85782`, 2026-07-12). This is a baseline scan of the current tree.*

## 1. Net-New Opportunities (High Priority)

1. **Orphaned AI feature: dead UI + live backend surface.** Commit `e39c7ac` ("Remove Tokens and AI tabs...") deleted the `AITab`/`TokensTab` entries from `src/panel/App.tsx` but left the components and their backing message handlers in place:
   - `src/panel/components/AITab.tsx` (546 lines) and `src/panel/components/TokensTab.tsx` + `src/panel/components/tokens.css` are no longer imported anywhere.
   - `src/background/service-worker.ts` still handles `AI_CRITIQUE_REQUEST`/`AI_NL_EDIT_REQUEST` (`runDesignCritique`, `runNLEdit`, ~lines 339-370) and `src/shared/messages.ts` still declares `AI_NL_EDIT_REQUEST`/`AI_CRITIQUE_REQUEST`/`AI_CRITIQUE_RESPONSE`/`AI_ERROR`/`GET_PAGE_TOKENS`/`PAGE_TOKENS_RESPONSE`/`APPLY_TOKEN`.
   - Value unlock: ~550+ lines of unreachable React UI plus an unused API-key-handling code path in the service worker is dead weight and a latent security surface (it still accepts and forwards `apiKey` over messaging with no caller). Either restore the entry points or delete the whole chain (components, CSS, message types, service-worker handlers) to shrink the bundle and remove an orphaned credential-handling path.

2. **`src/content/main.ts` (1149 lines) is becoming a god-file.** It mixes overlay bootstrapping, element picking wiring, drag/resize orchestration, and message routing in one entry. No single new addition caused this, but it's now the largest content-script file and the natural next place bugs will hide. Splitting message-routing (`chrome.runtime.onMessage` handling) into its own module (mirroring the pattern already used for `layers-panel.ts`, `change-tracker.ts`, `style-bridge.ts`) would keep the entry point thin and testable.

3. **`ColorPicker.tsx` (695 lines) is the largest control for the smallest concern.** It's more than double the size of the next-largest control (`GradientEditor.tsx`, ~unlisted but smaller) and carries 9 of the panel's 20 total inline `style={{...}}` usages. Given `GradientEditor` and `FillSection` both re-implement swatch/preview rendering adjacent to it, there's a likely extraction: a shared `ColorSwatch`/`ColorField` primitive to remove duplication between `ColorPicker` and `GradientEditor`.

## 2. Design System & UI Consistency

- No new hardcoded-style regressions found in the last commit set — the panel continues to route through `src/panel/controls/*` primitives and `src/shared/css-mapping.ts` for Figma-native terminology, which is the intended pattern per `CLAUDE.md`.
- The dead `TokensTab.tsx` + `tokens.css` pairing (see §1.1) is itself a design-system inconsistency: it's the only panel tab with its own dedicated CSS file rather than using the shared control styling — one more reason to remove rather than resurrect it if the token feature isn't coming back soon.

## 3. Status of Previous Flags

No previous review exists — this is the first scan. Nothing to carry forward.

## 4. Suggested Action/Execution Plan

```bash
claude -p "Delete the orphaned AI/Tokens feature: remove src/panel/components/AITab.tsx, TokensTab.tsx, tokens.css, the AI_* and *_TOKEN* message types in src/shared/messages.ts, and the runDesignCritique/runNLEdit handlers + apiKey plumbing in src/background/service-worker.ts. Verify with npm run typecheck and npm run build."
```
