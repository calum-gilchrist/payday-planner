# Payday Planner – notes for Claude

Personal money planner for one user (UK, £). Live at https://calum-gilchrist.github.io/payday-planner/ via GitHub Pages from `main` (root folder). Installed as an app on Android and used on a PC.

## How it's built
- No build step. Everything is in `index.html`: CSS tokens at the top (light + dark), then one IIFE with the app logic.
- `manifest.webmanifest` + `icons/` make it installable. `sw.js` caches the app for offline use.
- `vendor/` holds the Firebase JS SDK 10.14.1 compat build (app, auth, firestore). It's loaded locally, not from a CDN.
- `FIREBASE_CONFIG` near the top of `index.html` is the project config (public, not a secret). With it set to `null` the app runs local-only with example figures, which is handy for testing.

## Data
- Sign-in is Firebase email + password.
- All of a user's figures are one Firestore document, `users/{uid}/data/ledger` = `{state, updatedAt}`, written whole with `set()` (debounced in `save()`/`flush()`).
- A copy is also kept in `localStorage` (`pp-state`).
- `state` holds `bills[]`, `debts[]`, `goals[]`, `car{}` and `months{}`.
  - `months` is keyed by **pay month** `YYYY-MM` and holds per-month ticks, overrides, one-offs, adjustments and balance.
- `normalise()` fills in defaults for older data. Never rename or remove fields without migrating them there, or the user's saved figures break.
- Firestore rules (also in README): only `request.auth.uid == uid` can read or write `users/{uid}/**`.
- Backup/Restore (Settings) uses JSON `{app:'payday-planner', version:1, savedAt, state}`.

## Rules the app depends on
- **Payday** is the Thursday before the last Friday of the month. In December it's the Thursday before the last Friday *before* 25 Dec. See `payday(k)`.
  - A pay month runs from that payday to the next one.
  - Bill dates come from `itemDate(k, day)`, which takes the first matching day on or after payday.
- **Left for you** = already in account + money in/out + salary − house transfer − bills-account total − savings − car money.
- **Total owed** is shown but never taken off Left for you.
- **Left to spend** (from an entered balance) = balance − anything not yet ticked as moved out.

## User's preferences
- Plain UK English, £, no jargon. Keep the home screen short.
- Bills tab starts with the Bills account card (to cover, gone out, should be in account). Settings page shows no pay split.
- Manifest/meta theme colour matches the page background so Android's nav bar doesn't show a blue strip.
- Header: brand left, gear icon always top right on the same line; the sync status sits on its own line underneath.
- Settings (gear icon, top right): Auto/Light/Dark appearance (per device, `pp-theme`, default Auto), who the house money goes to, backup/restore, log out.
- Salary-split bar colours must stay clearly different hues (raspberry, blue, cyan, orange, green). The balance box stays subtle (no accent fill).
- Home screen ("This month"):
  - An **Overview** card sits at the top of every tab except Settings: Available, Spent (both from the entered balance) and Bills remaining (= should be in bills account), plus Update balance. The next-pay countdown ("3w 4d") is in the line under the month title.
  - Tabs: This month · Pay split · Bills · Owed · Car. The salary/transfer/car split lives on the **Pay split** tab.
  - Then the unticked **Bills to come off** in the user's own order (set with arrows on the Bills tab), then unticked **Owed to come off**, then Savings.
  - Ticked items move to a collapsible **Came off** section at the bottom.
- Bills can be:
  - a set day;
  - no set date (single tick);
  - several times a month (numbered tick boxes, no dates; "+ 1 extra" adds one just for that pay month via `months[k].times[id]`);
  - "carry unspent money over" budgets.
- Owed can be:
  - split into N payments (Klarna pay in 3, 12 months, etc.);
  - a fixed monthly amount;
  - pay what I can (credit cards, with an optional card name).

  Each can be marked "paid next one early", or a payment marked **pending** (`months[k].pend[id]`): it counts off the debt balance but stays in the bills account and in "Owed to come off" until ticked as gone out. Paid-off debts have **Undo last payment** and **Remove**.
- Savings goals have a target, where the money is kept and notes. You add money any time.
- Car mileage shows miles over, the total owed and the amount put aside. Money is added any time; it is not spread over the rest of the lease.

## Releasing a change
1. Edit the files.
2. Bump `VERSION` in `sw.js` (e.g. `pp-v2`) so installed apps pick up the update.
3. Commit to `main` and push. Pages redeploys in about a minute.
4. The user may need to close and reopen the app to see it.

GitHub settings (e.g. Pages) can't be changed through the API from Claude sessions. Ask the user to do those in GitHub.

## Testing
- Serve the folder with `python3 -m http.server`.
- Test in Playwright against a temporary copy with `FIREBASE_CONFIG = null`. Delete the copy afterwards and never commit it.
- Check:
  - no console errors;
  - paydays Oct 2026 → Thu 29 Oct, Dec 2026 → Thu 17 Dec, Jan 2027 → Thu 28 Jan;
  - restoring a backup works;
  - the page works at phone width (~420px) in light and dark.
