# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clickable demo of RCMS (Registration and Capital Monitoring System, MOIC + APB, Laos). It is a single-page app in vanilla JS with no framework and no backend. All data is mock, kept in memory, and reset on reload. The UI is bilingual: Lao (the default) and English. `README.md` (in Thai) documents every screen, role and business rule. The `SPEC-*.md` / `RCMS_Phase1_Spec.md` files are the requirement notes the demo follows.

## Commands

```
npm install
npm run dev      # Vite dev server on src/ (http://localhost:5173)
npm run build    # writes the single self-contained index.html at the repo root
npm run preview  # serve the built file
```

There are no tests, linter or formatter. To verify a change, run it in a browser against `npm run dev`.

## Source vs build

- **Edit only `src/index.html`.** The root `index.html` is generated output. `vite-plugin-singlefile` inlines all JS/CSS, and images become data URIs, so the file works over `file://` and on GitHub Pages (served from `main`, repo root).
- After changing `src/index.html`, run `npm run build` and commit **both** files. Pages serves the root file, so an unbuilt change never ships.
- `vite.config.js` sets `root: 'src'` and `emptyOutDir: false`, so the build overwrites only `index.html`.

## Code style

- ES5 throughout: `var`, `function(){}`, string concatenation `'a'+b`. No arrow functions, template literals or optional chaining (`SPEC-payment-fix.md` requires this).
- HTML is built as concatenated strings. Escape user text with `esc()`.
- UI text and docs use `-`, never an em-dash (README design rule).

## Architecture (all in `src/index.html`)

Section comments split the file: dictionary → helpers → accounts → seed data → state → app shell → views → render/navigation → `bind()`.

- **i18n**
  - `D.lo` / `D.en` hold every UI string; `t('key')` looks one up.
  - Data fields carry `{lo,en}` objects and are read with `L(obj)`.
  - Any new string needs both languages.
- **State and render loop**
  - One global `S` (built by `reset()`).
  - `render()` rebuilds the whole page from `S.view` (and the role), then `bind()` re-attaches handlers.
  - Handlers are found by `data-*` attributes (`data-nav`, `data-open`, `data-task`, `data-subtab`, ...). They mutate `S`, then call `render()` or `go(view)`.
  - To add an interaction: add a `data-*` attribute in the view, then a handler in `bind()`.
  - Text inputs that re-render while typing must restore focus/caret (see the existing handlers).
- **Roles**
  - `ACCTS` lists the demo accounts: owner, maker, checker, approver, valfirm, external, admin, newuser.
  - `S.role` drives what is shown:
    - `menus()` builds the sidebar. Entries with a 5th element are sub-menus, selected via `S.dashTab` or `S.finTab` / `S.setTab`.
    - `roleHome()` picks the home view.
    - `firstTab()` picks the default sub-menu.
    - `vDash()` dispatches to the per-role dashboard (`vMakerDash`, `vCheckerDash`, `vApproverDash`, `vAdminDash`, `vOwnerList` / `vOwnerDash`).
- **Login and Create Account** (`vLogin()`, steps in `S.loginStep`: `cred` → `otp`, or `signup` → `signupDone`)
  - Login runs credentials, then OTP. Accounts with `tempPw` then get the forced set-password modal (`S.modal={type:'forgot',step:'new',first:true}`).
  - Create Account keeps its form in `S.signup`, with `S.signup.step` 1-3:
    1. Applicant details: email, Citizen ID or Passport (`idType` `cid`/`pp`), Role applied for (`SU_ROLES`), Lao and English names.
    2. Verify email: OTP via `otpWire()`.
    3. Documents, chosen by `suDocsFor(su)`. Owner gets Enterprise details plus the documents for the type (`SU_ETYPES` / `SU_DOC`). Other roles upload ID/Passport plus an employee ID card.
  - Uploads are real `<input type="file">` (`data-sufile`, drop zone `data-sudrop`). `suFileErr()` allows PDF/JPG/PNG up to 10 MB.
  - Submitting shows a "pending document review" page. The demo skips the review: the `newuser` account becomes the applicant and can log in at once.
  - English UI says "Log in" / "Log out" and "Create Account", never "Sign in" / "Sign up".
- **Lists, search and paging** (use these instead of hand-rolled tables)
  - `listCard(o)` renders the search card (`colFilterBar` → `filterBar` with Search / Clear) plus a table card with count and pager.
    - Fields: `[col, label, options?, type?, match?]`. `options` makes a dropdown (exact match), `type:'date'` a date picker, `match(row, value)` a custom rule (`listMatch()`).
    - Typing edits `S.colDraft[key]`. Search / Enter copies it to `S.colf[key]`. Filtering happens in render, never by hiding DOM rows.
  - `paginate(key, list, sig, n0)` keeps page state in `S.pg[key]` and resets to page 1 when `sig` changes. Use it alone for tables without search.
  - `optsOf(list, getter)` builds dropdown options from values present in the data (Submit by / Verified by / Returned by).
  - Tables show Enterprise and Registration no. as separate columns, and the search splits them into separate fields too.
- **Summary cards**: `sumCard({title, prog, metrics, notes})` with `metric(tone, label, value, sub)` is the shared overview pattern.
- **Mock data**
  - Enterprises: `seedEnts()` (6 hand-written demo enterprises ELN-2025-1001..1006, used by the Owner/Maker walkthroughs) plus `seedMockEnts()` from `MOCK_ENTS` (24 more, ELN-2025-1101..1124, so lists page).
  - Pending declarations: `seedTasks()` plus `seedMockTasks()`. `stage` is `checker` or `approve`.
  - Valuation jobs: `VALJOBS` plus `MOCK_VAL`, with `mockCollateral()` on the enterprise.
  - The owner's enterprises: `OWNER_ENTS`.
  - "Today" is fixed at 04/08/2026 (`daysLeft()`).
  - Keep the six original enterprises' states intact; demo flows depend on them.
- **Dead code**: `vStaffDash` and `vQueue` (views `inspect` / `approveQ`) are unreachable from any role or menu.
